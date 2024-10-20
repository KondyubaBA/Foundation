<details>
  <summary>IHostBuilder</summary>

```cs
public interface IHostBuilder
{
    IDictionary<object, object> Properties { get; }

    IHostBuilder ConfigureHostConfiguration(Action<IConfigurationBuilder> configureDelegate);
    IHostBuilder ConfigureAppConfiguration(Action<HostBuilderContext, IConfigurationBuilder> configureDelegate);
    IHostBuilder ConfigureServices(Action<HostBuilderContext, IServiceCollection> configureDelegate);
    IHostBuilder UseServiceProviderFactory<TContainerBuilder>(IServiceProviderFactory<TContainerBuilder> factory) where TContainerBuilder : notnull;
    IHostBuilder UseServiceProviderFactory<TContainerBuilder>(Func<HostBuilderContext, IServiceProviderFactory<TContainerBuilder>> factory) where TContainerBuilder : notnull;
    IHostBuilder ConfigureContainer<TContainerBuilder>(Action<HostBuilderContext, TContainerBuilder> configureDelegate);
    IHost Build();
}
```  
</details>

#### Методы расширения
<details>
  <summary>UseApplicationMetadata</summary>

```cs
public static class ApplicationMetadataHostBuilderExtensions
{
    private const string DefaultSectionName = "ambientmetadata:application";

    public static IHostBuilder UseApplicationMetadata(this IHostBuilder builder, string sectionName = DefaultSectionName)
    {
        _ = Throw.IfNull(builder);
        _ = Throw.IfNullOrWhitespace(sectionName);

        return builder
            .ConfigureAppConfiguration((hostBuilderContext, configurationBuilder) => configurationBuilder.AddApplicationMetadata(hostBuilderContext.HostingEnvironment, sectionName))
            .ConfigureServices((hostBuilderContext, serviceCollection) => serviceCollection.AddApplicationMetadata(hostBuilderContext.Configuration.GetSection(sectionName)));
    }
}
```
</details>
