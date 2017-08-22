---
layout: "post"
title: "Mocking HTTP response in Simple.OData client"
date: "2017-08-22 17:00 +0100"
tags: [unit tests, odata]
categories: [.net]
---

3rd party libraries never seems to be documented enough. It's the old truth that every software developer learns sooner or later. In most cases after dozens of hours spent on trying to figure out "what the hell is wrong with my/that code!?". This post is about one of such "hidden features" in Microsoft oData client - [Simple.OData](https://github.com/object/Simple.OData.Client).<!-- more -->

Well written communication libraries in .NET allow for replacing their `HttpMessageHandler`. Some might want to add diagnostic logging or low-level error handling to the HTTP processing pipeline. Another reason is writing unit tests. This was my case.

I was writing unit tests for the OData proxy setup component. My first attempt was simple - derive from the  `HttpMessageHandler`, allow for mocking the response content and headers and finally implement `SendAsync`:

```csharp
private class MockHttpMessageHandler : HttpMessageHandler
{
    private const string DefaultResponseContent = "...";
    private const string DefaultResponseMimeType = "application/json";

    public HttpRequestMessage Request { get; private set; }
    public HttpResponseMessage Response { get; private set; }

    public void SetupResponse(HttpStatusCode statusCode,
        string mimeType = DefaultResponseMimeType,
        string content = DefaultResponseContent)
    {
        Response = new HttpResponseMessage(statusCode)
        {
            Content = new StringContent(content)
            {
                Headers = { ContentType = MediaTypeHeaderValue.Parse(mimeType) }
            }
        };
    }

    protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
    {
        Request = request;
        Response.RequestMessage = request;
        return await Task.FromResult(Response);
    }
}
```

Noting surprising here. The test code was also quite simple:

```csharp
[Test]
public async Task ShouldCreateClientAddingAuthorizationCookie()
{
    _messageHandler.SetupResponse(HttpStatusCode.OK);

    var client = _target.Create();
    await Query(client);

    var requestHeaders = _messageHandler.Request.Headers;
    requestHeaders.Should().Contain(header => IsCookieHeader(header));

    var cookieHeader = requestHeaders.First(IsCookieHeader);
    cookieHeader.Value.Should().Contain(value => IsAuthorizationCookie(value));
}
```

The surprise came right after running it:

```
Microsoft.OData.Core.ODataException : An unexpected 'EndOfInput' node was found when reading from the JSON reader. A 'StartObject' node was expected. Error found near:  <---
  at Microsoft.OData.Core.Json.JsonReaderExtensions.ValidateNodeType (Microsoft.OData.Core.Json.JsonReader jsonReader, Microsoft.OData.Core.Json.JsonNodeType expectedNodeType)
  at Microsoft.OData.Core.Json.JsonReaderExtensions.ReadNext (Microsoft.OData.Core.Json.JsonReader jsonReader, Microsoft.OData.Core.Json.JsonNodeType expectedNodeType)
  ...
  at Microsoft.OData.Core.ODataReaderCore.ReadImplementation ()
  at Microsoft.OData.Core.ODataReaderCore.ReadSynchronously ()
  at Microsoft.OData.Core.ODataReaderCore.InterceptException[T] (System.Func`1[TResult] action)
  at Microsoft.OData.Core.ODataReaderCore.Read ()
  at Simple.OData.Client.V4.Adapter.ResponseReader.ReadResponse (Microsoft.OData.Core.ODataReader odataReader, System.Boolean includeAnnotationsInResults)
  at Simple.OData.Client.V4.Adapter.ResponseReader+<GetResponseAsync>d__11.MoveNext ()
```

Reading the code of lower-level parsing API did not help much. After few hours of digging in the Simple.OData codebase I found the first clue - `ResponseReader` `GetResponseAsync()` method:

```csharp
public override Task<ODataResponse> GetResponseAsync(HttpResponseMessage responseMessage, bool includeAnnotationsInResults = false)
{
    return this.GetResponseAsync((IODataResponseMessageAsync) new ODataResponseMessage(responseMessage), includeAnnotationsInResults);
}
```

It turned out that the original `HttpResponseMessage` is wrapped with the internal implementation. One look on the `ODataResponseMessage` class helped me to understand what's happening:

```csharp
public Task<Stream> GetStreamAsync()
{
    StreamContent content = this._response.Content as StreamContent;
    if (content != null)
        return content.ReadAsStreamAsync();

    TaskCompletionSource<Stream> completionSource = new TaskCompletionSource<Stream>();
    completionSource.SetResult(Stream.Null);
    return completionSource.Task;
}
```

Obviously the `StringContent` type in `HttpResponseMessage` was not expected by the author.

Here is the final version of `SetupResponse()` method in the mock class:

```csharp
public void SetupResponse(HttpStatusCode statusCode,
    string mimeType = DefaultResponseMimeType,
    string content = DefaultResponseContent)
{
    var stream = new MemoryStream();
    using (var writer = new StreamWriter(stream, Encoding.UTF8, 1024, true))
    {
        writer.Write(content);
        writer.Flush();
    }
    stream.Position = 0;

    Response = new HttpResponseMessage(statusCode)
    {
        Content = new StreamContent(stream)
        {
            Headers = { ContentType = MediaTypeHeaderValue.Parse(mimeType) }
        }
    };
}
```

Remember to set the `MemoryStream.Position` property to the start of the buffer. Otherwise you will get exactly the same error as before but for the different reason.

Happy coding!
