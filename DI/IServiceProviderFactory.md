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
