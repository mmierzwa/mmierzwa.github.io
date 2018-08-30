---
layout: post
title: "Scheduling in ASP.NET Core with Quartz.NET"
date: 2017-04-13 17:57 +0200
tags: [.net, scheduling]
---

Running certain tasks in a scheduled manner may be an easy solution for many problems. One might be refreshing the application cache when the data needs to be fresh and warm no matter what the actual traffic is. Other could be the synchronization or periodical clean-up. There are obviously few good architectural patterns to do it in more elegant and efficient way - distributed queues, publish-subscribe models, enterprise service buses etc. But the simplicity of scheduling still might be an important decision variable.

I was using [Quartz.NET](https://www.quartz-scheduler.net/) (2.x) for this purpose for quite a while in ASP.NET MVC5. It's simple to use in simple scenarios but powerful enough to handle more complex when there is such a need. Recently I needed to implement a scheduling in an ASP.NET Core Web API service. I wanted to have all the Core features on board - importantly dependency injection, strongly-typed configuration and standard logging. It turned out not to be as straight forward as it seemed on the beginning. But was not so hard either. Here is how I did it.
<!-- more -->

There's one thing worth to mention before I start. My ASP.NET Core service was configured to use .NET Framework 4.6.1 (`net461` in `project.json`), not the .NET Core. I'm not sure if there is any .NET Standard- or PCL-compatible version of Quartz.NET. At least in the latest stable release I was using - 2.5. This is something to be checked before you start the development. Especially if running the service on non-Windows environments (like Docker containers) is your requirement. Read the [ASP.NET Core documentation](https://docs.microsoft.com/en-gb/dotnet/articles/standard/choosing-core-framework-server) for more details on that.  
Now let's move on to the main part.

The first step is to create a custom implementation of Quartz `IJobFactory`. The default one does not allow to use DI on jobs creation. Passing in the `IServiceProvider` reference enables this scenario in `NewJob()` method:

```csharp
public class QuartzJonFactory : IJobFactory
{
    private readonly IServiceProvider _serviceProvider;

    public QuartzJonFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public IJob NewJob(TriggerFiredBundle bundle, IScheduler scheduler)
    {
        var jobDetail = bundle.JobDetail;

        var job = (IJob)_serviceProvider.GetService(jobDetail.JobType);
        return job;
    }

    public void ReturnJob(IJob job) { }
}
```

The above example does not utilize job pooling. If you would like to apply this strategy you need to change `NewJob()` method and implement `ReturnJob()`. I would recommend this if jobs creation is time consuming or you notice that GC hits the performance doing an intensive work on your `IJob` objects. This might depend on your triggers configuration.

Next step is setting-up the dependencies. I did it in ASP.NET Core fashion which is just an extension method on `IServiceCollection`:

```csharp
public static void UseQuartz(this IServiceCollection services, params Type[] jobs)
{
    services.AddSingleton<IJobFactory, QuartzJonFactory>();
    services.Add(jobs.Select(jobType => new ServiceDescriptor(jobType, jobType, ServiceLifetime.Singleton)));

    services.AddSingleton(provider =>
    {
        var schedulerFactory = new StdSchedulerFactory();
        var scheduler = schedulerFactory.GetScheduler();
        scheduler.JobFactory = provider.GetService<IJobFactory>();
        scheduler.Start();
        return scheduler;
    });
}
```

Here is a utility method that configures the job and its trigger:

```csharp
public static class QuartzServicesUtilities
{
    public static void StartJob<TJob>(IScheduler scheduler, TimeSpan runInterval)
        where TJob : IJob
    {
        var jobName = typeof(TJob).FullName;

        var job = JobBuilder.Create<TJob>()
            .WithIdentity(jobName)
            .Build();

        var trigger = TriggerBuilder.Create()
            .WithIdentity($"{jobName}.trigger")
            .StartNow()
            .WithSimpleSchedule(scheduleBuilder =>
                scheduleBuilder
                    .WithInterval(runInterval)
                    .RepeatForever())
            .Build();

        scheduler.ScheduleJob(job, trigger);
    }
}
```

It is very basic and specific to my recent project. You may want to add more flexibility here - for example by exposing a configuration builder with a fluent syntax.

Now it's time code the actual `IJob` implementation. Here is an example:

```csharp
public class SomeJob : IJob
{
    private readonly ILogger<SomeJob> _log;
    private readonly JobConfiguration _configuration;
    // other dependencies will probably go here

    public SomeJob(IOptions<JobConfiguration> configuration, ILogger<SomeJob> log)
    {
        _log = log;
        _configuration = configuration.Value;
    }

    public void Execute(IJobExecutionContext context)
    {
        Task.Run(Execute);
    }

    private async Task Execute()
    {
        try
        {
            // implement your scheduled job logic
        }
        catch (Exception ex)
        {
            _log.LogError(1, ex, "An error occurred during execution of scheduled job");
        }
    }
}
```

I've included two standard ASP.NET Core dependencies as an example. First is a [strongly-typed configuration](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration). I prefer it over the standard Quartz.NET `JobDataMap` because of consistency with rest of the Web API and clean design. Second is the ASP.NET Core [logging infrastructure](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging). If you plan to use it instead of `Common.Logging` features delivered with Quartz.NET keep in mind to create your scheduled jobs after the logging is configured.

Here is an example code that you can use in `Startup` class to wire-up all those pieces:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // here goes all the IoC configuration

    services.UseQuartz(typeof(SomeJob));
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    // here you might want to include other config like logging
    // and fetch the schedule interval probably from configuration

    var scheduler = app.ApplicationServices.GetService<IScheduler>();
    QuartzServicesUtilities.StartJob<SomeJob>(scheduler, someInterval);
}
```

And voilà - that's practically all the code you need for the basic stuff. The rest may be as complex as you can imagine in your own `IJob` classes.

Happy coding!
