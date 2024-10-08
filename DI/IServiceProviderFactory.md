```cs
public class DefaultServiceProviderFactory : IServiceProviderFactory<IServiceCollection>
{
    private readonly ServiceProviderOptions _options;

    public DefaultServiceProviderFactory() : this(ServiceProviderOptions.Default)
    {
    }

    public DefaultServiceProviderFactory(ServiceProviderOptions options)
    {
        _options = options ?? throw new ArgumentNullException(nameof(options));
    }

    public IServiceCollection CreateBuilder(IServiceCollection services)
    {
        return services;
    }

    public IServiceProvider CreateServiceProvider(IServiceCollection containerBuilder)
    {
        return containerBuilder.BuildServiceProvider(_options);
    }
}
```


<details>
    <summary>IServiceProviderFactory</summary>
    
```cs
public interface IServiceProviderFactory<TContainerBuilder> where TContainerBuilder : notnull
{
    TContainerBuilder CreateBuilder(IServiceCollection services);
    IServiceProvider CreateServiceProvider(TContainerBuilder containerBuilder);
}
```
</details>


<details>
    <summary>ServiceProviderOptions</summary>

```cs
public class ServiceProviderOptions
{
    internal static readonly ServiceProviderOptions Default = new ServiceProviderOptions();
    public bool ValidateScopes { get; set; }
    public bool ValidateOnBuild { get; set; }
}
```
</details>


```cs
public interface IServiceCollection : IList<ServiceDescriptor>
{
}
```

```cs
public interface IServiceProvider
  {
      object? GetService(Type serviceType);
  }
```

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
```
</details>
