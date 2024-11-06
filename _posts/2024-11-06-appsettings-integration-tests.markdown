---
layout: post
title:  "Appsettings and Integration Tests"
date:   2024-11-06 11:34:44 -0500
categories: dotnet testing appsettings
---
## Problem
When writing integration tests, you may need to use different configurations than the ones you use in your application.
For example, you may want to use a different database connection string or a different API key.
In this post, I will show you how to use different appsettings.json files for your integration tests.

As explained in a previous post, you can use the `WebApplicationFactory` class to create a test server for your integration tests.
This class allows you to configure the test server in the same way you would configure the application.

I initially thought that I could specify the appsettings file to use in the `ConfigureWebHost` method of the `WebApplicationFactory` class.
This would load the test file on top of the other appsettings files loaded by the app.

Here is what it looked like:

{% highlight csharp mark_lines="6 7 8" %}
public class CustomWebApplicationFactory<TProgram>
    : WebApplicationFactory<TProgram> where TProgram : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureAppConfiguration((_, configurationBuilder) =>
        {
            configurationBuilder.AddJsonFile("appsettings.tests.json");
        });

        builder.ConfigureServices(services =>
        {
            // [...]
        });
    }
}
{% endhighlight %}

However, this did not work as expected. This method worked fine for simple configurations, but it did not work when the appsettings file was used to configure services.

If any of the services dependency injection (DI) initialization were reliant on the configuration loaded from the appsettings, the test would fail.

That is because the code for the services DI initialization is executed before the `ConfigureWebHost` method from `CustomWebApplicationFactory` is called.

The execution order is as follows:
- appsettings are loaded
- services are initialized
- test appsettings are loaded

## Solution

To fix this issue, the simple solution was to explicitly set the Environment to 'test' in the `ConfigureWebHost` method as follows:

{% highlight csharp mark_lines="6" %}
public class CustomWebApplicationFactory<TProgram>
    : WebApplicationFactory<TProgram> where TProgram : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.UseEnvironment("test");

        builder.ConfigureServices(services =>
        {
            // [...]
        }
    }
}
{% endhighlight %}