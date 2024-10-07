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

```cs
public interface IServiceProviderFactory<TContainerBuilder> where TContainerBuilder : notnull
{
    /// <summary>
    /// Creates a container builder from an <see cref="IServiceCollection"/>.
    /// </summary>
    /// <param name="services">The collection of services</param>
    /// <returns>A container builder that can be used to create an <see cref="IServiceProvider"/>.</returns>
    TContainerBuilder CreateBuilder(IServiceCollection services);

    /// <summary>
    /// Creates an <see cref="IServiceProvider"/> from the container builder.
    /// </summary>
    /// <param name="containerBuilder">The container builder</param>
    /// <returns>An <see cref="IServiceProvider"/></returns>
    IServiceProvider CreateServiceProvider(TContainerBuilder containerBuilder);
}
```

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
