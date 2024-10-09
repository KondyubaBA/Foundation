#### ConfigureDefault
```cs
public static IHostBuilder CreateDefaultBuilder(string[]? args)
{
    HostBuilder builder = new();
    return builder.ConfigureDefaults(args);
}
```

```cs
//public static class HostingHostBuilderExtensions
public static IHostBuilder ConfigureDefaults(this IHostBuilder builder, string[]? args)
{
    return builder.ConfigureHostConfiguration(config => ApplyDefaultHostConfiguration(config, args))
                  .ConfigureAppConfiguration((hostingContext, config) => ApplyDefaultAppConfiguration(hostingContext, config, args))
                  .ConfigureServices(AddDefaultServices)
                  .UseServiceProviderFactory(context => new DefaultServiceProviderFactory(CreateDefaultServiceProviderOptions(context)));
}
```

#### Настройка ConfigureHostConfiguration
```cs
//public static class HostingHostBuilderExtensions
private static void ApplyDefaultHostConfiguration(IConfigurationBuilder hostConfigBuilder, string[]? args)
{
    SetDefaultContentRoot(hostConfigBuilder);

    hostConfigBuilder.AddEnvironmentVariables(prefix: "DOTNET_");
    AddCommandLineConfig(hostConfigBuilder, args);
}
```
