```cs
public static class Host
{
    public static IHostBuilder CreateDefaultBuilder() =>
        CreateDefaultBuilder(args: null);

    public static IHostBuilder CreateDefaultBuilder(string[]? args)
    {
        HostBuilder builder = new();
        return builder.ConfigureDefaults(args);
    }
    public static HostApplicationBuilder CreateApplicationBuilder() => new HostApplicationBuilder();

    public static HostApplicationBuilder CreateApplicationBuilder(string[]? args) => new HostApplicationBuilder(args);

    public static HostApplicationBuilder CreateApplicationBuilder(HostApplicationBuilderSettings? settings)
        => new HostApplicationBuilder(settings);

    public static HostApplicationBuilder CreateEmptyApplicationBuilder(HostApplicationBuilderSettings? settings)
        => new HostApplicationBuilder(settings, empty: true);
}
```
