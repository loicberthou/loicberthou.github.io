---
layout: post
title:  "Integration Tests in Dotnet"
date:   2024-11-05 17:55:44 -0500
categories: dotnet testing integration
---
## Problem
When writing integration tests, you may want to replace some services with mocks or remove some services from the service collection.

## Solution
```csharp
public class CustomWebApplicationFactory<TProgram>
    : WebApplicationFactory<TProgram> where TProgram : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Here you will be able to override the services registered in the application
            // For example, you can remove the background service
            var backgroundService = services.Single(d => d.ImplementationType == typeof(JobsCleanupBackgroundService));
            services.Remove(backgroundService);

            // Or you can replace the implementation of a service
            var serviceDescriptor = services.Single(d => d.ServiceType == typeof(IAmazonS3));
            services.Remove(serviceDescriptor);
            services.AddSingleton(A.Fake<IAmazonS3>());
        });

        builder.UseEnvironment("Development");
    }
}
```