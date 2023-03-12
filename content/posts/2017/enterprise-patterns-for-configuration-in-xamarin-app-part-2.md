---
categories:
- mobile
date: "2017-12-06T00:00:00Z"
tags:
- design
- SOLID
- IoC
title: 'Enterprise patterns for configuration in Xamarin app. Part 2: Managing dependencies'
---

In [last post](/blog/enterprise-patterns-for-configuration-in-xamarin-app-part-1) I described how to cope, in some extent, with different pace of delivering mobile app versus it's supporting backend. The article also provided a simple hint for speeding up the mobile app development by introducing mocks instead of the external network services.

In this one I'm gonna give you some advice on managing the dependencies as your app gets more of them in time.


## Managing mocks

Let's take a look at the example of registering dependencies from the [previous post](/blog/enterprise-patterns-for-configuration-in-xamarin-app-part-1):

```csharp
public void RegisterDependencies(Autofac.ContainerBuilder builder)
{
  #if OFFLINE_MODE
  builder.RegisterType<MockBackendService>().As<IMyBackendService>().SingleInstance();
  #else
  builder.RegisterType<HttpBackendService>().As<IMyBackendService>().SingleInstance();
  #endif
}
```

So far so good. Initially the `IMyBackendService` could have only few methods. However it will start to grow as the functionality offered by the app and it's supporting backend extends. Even if the app communicates with an single endpoint the new methods (in RPC style) or entities (in RESTful style) will be introduced. Eventually at some point the service will have to be broken down into smaller services. This is the direct consequence of [Single Responsibility](https://en.wikipedia.org/wiki/Single_responsibility_principle) and [Interface Segregation](https://en.wikipedia.org/wiki/Interface_segregation_principle) principles. It's probably easier to spot when using the RESTful API.

To picture how this can look like see the interface that seems to represent to many responsibilities:

```csharp
public interface IMyBackendService
{
  Task<IEnumerable<Employee>> GetEmployees();
  Task Update(Employee employee);

  Task<IEnumerable<Department>> GetDepartments();
  Task Update(Department department);

  Task<IEnumerable<Product>> GetProducts();
  Task Update(Product department);
}
```

The implementing class will have to deal with different kinds of entities: `Employee`, `Department` and `Product` (so as the mock). All `IMyBackendService` implementations will probably grow in time to hundreds lines or more depending of the corresponding additional logic.

On the consumer side the view models (or other services) will have access to all those kinds of data no matter if they need it or not. This greatly reduces traceability of the real logical connections inside the application. It also often introduces coupling that is hard maintain (even in short term). Of course the example above is still quite simple but you get the idea.

The remedy is to split the interface (and corresponding implementations):

```csharp
public interface IEmployeesService
{
  Task<IEnumerable<Employee>> GetEmployees();
  Task Update(Employee employee);
}

public interface IDepartmentsService
{
  Task<IEnumerable<Department>> GetDepartments();
  Task Update(Department department);
}

public interface IProductsService
{
  Task<IEnumerable<Product>> GetProducts();
  Task Update(Product department);
}
```

All relevant view models and other consumers would have to be updated as well.

Now let's get back to the dependency registration from the beginning:

```csharp
public void RegisterDependencies(Autofac.ContainerBuilder builder)
{
  #if OFFLINE_MODE
  builder.RegisterType<MockEmployeesService>().As<IEmployeesService>().SingleInstance();
  builder.RegisterType<MockDepartmentsService>().As<IDepartmentsService>().SingleInstance();
  builder.RegisterType<MockProductsService>().As<IProductsService>().SingleInstance();
  #else
  builder.RegisterType<HttpEmployeesService>().As<IEmployeesService>().SingleInstance();
  builder.RegisterType<HttpDepartmentsService>().As<IDepartmentsService>().SingleInstance();
  builder.RegisterType<HttpProductsService>().As<IProductsService>().SingleInstance();
  #endif
}
```

This begins to look pretty awkward. It's not hard to imagine how this will look like after adding few more services. How can we improve the readability of this method? Maybe method-level decomposition will do the trick:

```csharp
public void RegisterDependencies(Autofac.ContainerBuilder builder)
{
  #if OFFLINE_MODE
  RegisterMockServices(builder);
  #else
  RegisterHttpServices(builder);
  #endif
}

private void RegisterMockServices(Autofac.ContainerBuilder builder)
{
  builder.RegisterType<MockEmployeesService>().As<IEmployeesService>().SingleInstance();
  builder.RegisterType<MockDepartmentsService>().As<IDepartmentsService>().SingleInstance();
  builder.RegisterType<MockProductsService>().As<IProductsService>().SingleInstance();
}

private void RegisterHttpServices(Autofac.ContainerBuilder builder)
{
  builder.RegisterType<HttpEmployeesService>().As<IEmployeesService>().SingleInstance();
  builder.RegisterType<HttpDepartmentsService>().As<IDepartmentsService>().SingleInstance();
  builder.RegisterType<HttpProductsService>().As<IProductsService>().SingleInstance();
}
```

This solution has still some drawbacks. First - both cases are still handled within the same class which smells like breaking SRP rule again. Each new dependency will increase the number of lines in this class +2 (or +N if you have more such cases).

From the less puristic perspective - if you're developing with any modern IDE it will probably suggest that one of the two private methods should be removed since only one is being called in the current build configuration. I cannot count how many times I had accidentally removed the living code because of automatic clean-ups...

## Modules to the rescue

An alternative for registering everything in one place are modules. This concept is probably well known for many backend developers since the number of dependencies tends to grow much faster in those applications. Unfortunately, as far as I know, it have not been implemented in [Xamarin.Forms DependencyService](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/dependency-service/introduction/). From the DI containers I have worked with it's available in [Ninject](https://github.com/ninject/Ninject/wiki/Modules-and-the-Kernel) and [Autofac](http://autofaccn.readthedocs.io/en/latest/configuration/modules.html).

There are many interesting things you can do with modules but for now let's see how they can help to overcome the issues mentioned above:

```csharp
public class MockServicesModule : Autofac.Module
{
    protected override void Load(Autofac.ContainerBuilder builder)
    {
      builder.RegisterType<MockEmployeesService>().As<IEmployeesService>().SingleInstance();
      builder.RegisterType<MockDepartmentsService>().As<IDepartmentsService>().SingleInstance();
      builder.RegisterType<MockProductsService>().As<IProductsService>().SingleInstance();
    }
}

public class HttpServicesModule : Autofac.Module
{
    protected override void Load(Autofac.ContainerBuilder builder)
    {
      builder.RegisterType<HttpEmployeesService>().As<IEmployeesService>().SingleInstance();
      builder.RegisterType<HttpDepartmentsService>().As<IDepartmentsService>().SingleInstance();
      builder.RegisterType<HttpProductsService>().As<IProductsService>().SingleInstance();
    }
}
```

The registration entry point doesn't change much comparing to the previous one:

```csharp
public void RegisterDependencies(Autofac.ContainerBuilder builder)
{
  #if OFFLINE_MODE
  builder.RegisterModule<MockServicesModule>();
  #else
  builder.RegisterModule<HttpServicesModule>();
  #endif
}
```

As you can see this eliminates the issues of the previous approach. All the registration cases have their own type (SRP). IDE or ReSharper will not remove your registrations or their usings by accident. I like this approach also because it makes the main registration point short and clean.

## Wrapping up

The benefits of decomposing app IoC configuration with modules are not limited to the mocking use-case. You can easily create other environment-dependent configurations like `UiTestsModule`, `LocalDevModule` etc. Instead of using static compilation directives you can create dynamic runtime configuration (with couple more details like dependencies unregistration). I encourage you to experiment. IoC configuration is often much easier to maintain than tens of flags of switch-case logic in your services or view models.

In next articles I'll continue with the configuration topics since there are quite a few more to cover. Dealing with sensitive mocked data or managing version-dependent configurations are just the examples.

Happy coding!
