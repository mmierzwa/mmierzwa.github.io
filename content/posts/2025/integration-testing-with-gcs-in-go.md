---
categories:
- testing
date: "2025-02-15T00:00:00Z"
tags:
- golang
- acceptance tests
- gcp
title: Testing integration with GCS in Go
---

As I wrote [in my previous post](/blog/do-we-still-need-testing-pyramid), I have a strong preference towards automated acceptance testing on a service-level. So, when I recently faced a need of creating a Go API related to file management that uses GCS as a storage, I started looking for some way to test it this way. Unfortunately, Google doesn't provide any simulator for GCS similar and there is also no the-one-to-rule-them-all tool such as Localstack, which I've been successfully using for years working with AWS services. I found [`fsouza/fake-gcs-server`](https://github.com/fsouza/fake-gcs-server) good enough and quite popular, so I started with that. Although, there are [some samples](https://github.com/fsouza/fake-gcs-server/blob/c4a13e7a656207272288bc913bcc99f3784e2bb1/examples/go/main.go) in the project itself and a couple of blog posts how to start using it, I had to solve a couple of issues, which I think can be interesting, if you're working with this tool in your Go application.

## Containers

Hopefully, your app is already running locally in container. If so, even if you don't have your Docker compose file yet, it will be easy to create one. Running all services - both locally and on CI - will greatly help you to test your app with all required dependencies deterministically and fast. Here is a minimalistic example of such file.

```yaml
services:
  api:
    image: golang:1.23.5-alpine
    working_dir: /app
    depends_on:
      gcs-storage:
        condition: service_healthy
    volumes:
      - ./:/app
      - pkg:/go/pkg
    env:
      - TEST_GCP_GOOGLE_ACCESS_ID=...
      - TEST_GCP_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----..."
      - GSC_ENDPOINT_URL=http://gcs-storage:8083/storage/v1/
      - STORAGE_EMULATOR_HOST=http://gcs-storage:8083
      - ...
    ports:
      - "8080:8080"
    healthcheck:
      test: "nc -z -w 1 api 8080"
      interval: 30s
      timeout: 30s
      retries: 15
      start_period: 1s
    command: ["go", "run", "./path-to-your-main-cmd-binary"]

  gcs-storage:
    image: fsouza/fake-gcs-server:1.52.1
    volumes:
      - ./testdata/gcs:/data
    ports:
      - "8083:8083"
    command: [ "-scheme", "http", "-port", "8083", "-external-url", "http://[::]:8083", "-backend", "memory" ]
    healthcheck:
      test: ["CMD-SHELL", "wget --no-verbose --tries=1 --spider http://gcs-storage:8083/storage/v1/b || exit 1"]
      interval: 10s
      timeout: 3s
      retries: 5
      start_period: 1s

volumes:
  pkg:
```

Obviously, you will need to adjust it to your needs, i.e. the actual port numbers, networking etc. Also, remember to upgrade Docker images versions; I have the static version numbers to control dependencies (using `@latest` is also not recommended from the security standpoint). 

Important parts here are:

* `STORAGE_EMULATOR_HOST` - this env is used in the official GCP SDK from Google. Once SDK detects it, it will know that you're using a non-cloud development API, which will save your time debugging strange errors, as I had to.
* `TEST_GCP_GOOGLE_ACCESS_ID`, `TEST_GCP_PRIVATE_KEY`, `GSC_ENDPOINT_URL` - those env variables will be further used in the app code, so I will explain them later.
* `command: [ "-scheme", "http", "-port", "8083", "-external-url", "http://[::]:8083", "-backend", "memory" ]` runs the fake-gcs-server as non-SSL HTTP server, accepting connections under all hostnames (so you can possibly set some network aliases as well), and with in-memory storage resetting with the process (which will also make your tests more predictable).
* `healthcheck` elements are always a good idea in compose file. Once you have them properly configured with dependencies set, your test runs will be much more stable and deterministic. Experiment with different values to tune for the best speed/stability ratio on your local machine and CI agent.

## Application

Now, within your app, you will need to add the [Google GCP SDK](https://pkg.go.dev/cloud.google.com/go/storage):

```cli
go get cloud.google.com/go/storage@latest
```

Now, the way for initializing the client heavily depends on the authorization method you use which in turn depends on the way you deploy your app. If your app is hosted in the GCP CloudRun or AppEngine you will probably already have the Application Default Credentials env and config file in your target environment. Otherwise, follow [this instruction](https://cloud.google.com/docs/authentication/application-default-credentials) to set it up.

Locally, accessing the `fsouza/fake-gcs-server` API doesn't require any credentials, so you won't have to bother with that on local or CI environments. All you should do is to point the local API (the `GSC_ENDPOINT_URL` env in the compose example above):

```go
import (
	"context"
	"os"
	
	"cloud.google.com/go/storage"
	"google.golang.org/api/option"
)

func NewGCSClient() (*storage.Client, error) {
	ctx := context.Background()

	opts := make([]option.ClientOption, 0, 1)
	// non-local environments don't need this setting
    endpointURL, exists := os.LookupEnv("GSC_ENDPOINT_URL")
	if exists {
		opts = append(opts, option.WithEndpoint(endpointURL))
	}

	client, err := storage.NewClient(ctx, opts...)
	if err != nil {
		return nil, err
	}
	return client, nil
}
```

There is an exception, however, when it comes to [generating pre-signed URLs](https://pkg.go.dev/cloud.google.com/go/storage#hdr-Credential_requirements_for_signing). If your app must handle such scenario, there are some additional setup you need to run on calling this part of the SDK. 

Here is function that implements generating pre-signed URLs:

```go
import (
    "time"
	"os"
    
    "cloud.google.com/go/storage"
)

var (
    // normally, you would pass those as configuration
    testGCPGoogleAccessID = os.GetEnv("TEST_GCP_GOOGLE_ACCESS_ID")
    testGCPPrivateKey = os.GetEnv("TEST_GCP_PRIVATE_KEY")
	urlExpirationTime = 10 * time.Second
)

func GenerateAuthenticatedFileURL(client *storage.Client, bucketName, bucketKey string) (string, error) {
    opts := &storage.SignedURLOptions{
		Scheme:  storage.SigningSchemeV4,
		Method:  "GET",
		Expires: time.Now().Add(urlExpirationTime),
	}

	// those options are used only in the local testing
	if testGCPGoogleAccessID != "" && testGCPPrivateKey != "" {
		opts.GoogleAccessID = testGCPGoogleAccessID
		opts.PrivateKey = []byte(testGCPPrivateKey)
	}

	return client.Bucket(bucketName).SignedURL(bucketKey, opts)
}
```

Note the envs already mentioned earlier in the compose file - `TEST_GCP_GOOGLE_ACCESS_ID` and `TEST_GCP_PRIVATE_KEY`. Normally, in your production environment with properly configured Default Application Credentials, those are set for you. However, because we set the `STORAGE_EMULATOR_HOST` env, they are not loaded and must be provided: 

* The Google Access ID is in email format, i.e. `<12 random digits>-compute@developer.gserviceaccount.com`. You can use an arbitrary 12-digit-string here.
* Private key is an asymmetric PKI private key in PKCS12 PEM format. The format must be correct, but as above, it doesn't matter what it will be.

To ensure that the digits are truly random I typically use this command:

```cli
LC_ALL=C tr -dc 0-9 </dev/urandom | head -c 12; echo
```

You can generate the private key using OpenSSL:

```cli
openssl pkcs12 -in key.p12 -out key.pem -nodes
```

and then copy the `key.pem` to your compose file (or .env file if you manage local envs this way). It will look like this:

```txt
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQC2vUwdenqOofUP
takIWeRSTfziHGHI/BfHUxYK/1sDToBucuoUz8RBBvJQQvsAOfGARB7KVgtvuRi8
...
GTJdRfjU2P+zChri+23+/NMqkyAVcvfXMaFbnwizdoO7mZrRizu1P3BgEBlgUEeK
NU1/ivyYVJxX/zpMhoaxuCHQt7CgYTSVorAyT14Wo9WqUQy7T0b2TW7LC2RWPvCf
y9ifuqzLQfshxD0dli1eB6Aq
-----END PRIVATE KEY-----
```

And that's more or less all. I typically prepare also some test helpers for clearing the bucket, listing its content, etc. Let me know if you're interested in adding those samples here. Other operations are quite well described [in the pkg docs](https://pkg.go.dev/cloud.google.com/go/storage#section-readme). 

I hope this will help you somehow in building your testable and a bit more maintainable application in Go that uses GCS.

Happy coding!
