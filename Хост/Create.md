#### 1. CreateDefaultBuilder
```cs
public static IHostBuilder CreateDefaultBuilder(string[]? args)
{
    HostBuilder builder = new();
    return builder.ConfigureDefaults(args);
}
```

#### 2. ConfigureDefault - конфигурация по умолчанию
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

#### 2.1 ConfigureHostConfiguration - конфигурация хоста
```cs
//public static class HostingHostBuilderExtensions
private static void ApplyDefaultHostConfiguration(IConfigurationBuilder hostConfigBuilder, string[]? args)
{
    //устанавливает путь к корневому каталогу приложения
    SetDefaultContentRoot(hostConfigBuilder);
    //добавляет все переменные среды в конфигурацию приложения, которые начинаются с префикса "DOTNET_"
    hostConfigBuilder.AddEnvironmentVariables(prefix: "DOTNET_");
    //добавляет аргументы командной строки в конфигурацию приложения
    AddCommandLineConfig(hostConfigBuilder, args);
}
```


<details>
  <summary>SetDefaultContentRoot</summary>

```cs
        internal static void SetDefaultContentRoot(IConfigurationBuilder hostConfigBuilder)
        {
            // If we're running anywhere other than C:\Windows\system32, we default to using the CWD for the ContentRoot.
            // However, since many things like Windows services and MSIX installers have C:\Windows\system32 as there CWD which is not likely
            // to really be the home for things like appsettings.json, we skip changing the ContentRoot in that case. The non-"default" initial
            // value for ContentRoot is AppContext.BaseDirectory (e.g. the executable path) which probably makes more sense than the system32.

            // In my testing, both Environment.CurrentDirectory and Environment.SystemDirectory return the path without
            // any trailing directory separator characters. I'm not even sure the casing can ever be different from these APIs, but I think it makes sense to
            // ignore case for Windows path comparisons given the file system is usually (always?) going to be case insensitive for the system path.
            string cwd = Environment.CurrentDirectory;
            if (
#if NETFRAMEWORK
                Environment.OSVersion.Platform != PlatformID.Win32NT ||
#else
                !RuntimeInformation.IsOSPlatform(OSPlatform.Windows) ||
#endif
                !string.Equals(cwd, Environment.SystemDirectory, StringComparison.OrdinalIgnoreCase))
            {
                hostConfigBuilder.AddInMemoryCollection(new[]
                {
                    new KeyValuePair<string, string?>(HostDefaults.ContentRootKey, cwd),
                });
            }
        }
```
</details>

<details>
  <summary>HostDefaults</summary>

```cs
//Microsoft.Extensions.Hosting
public static class HostDefaults
{
    public static readonly string ApplicationKey = "applicationName";
    public static readonly string EnvironmentKey = "environment";
    public static readonly string ContentRootKey = "contentRoot";
}
```
</details>

