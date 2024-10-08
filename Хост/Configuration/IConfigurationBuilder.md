#### Абстракция
```cs
public interface IConfigurationBuilder
{
    IDictionary<string, object> Properties { get; }
    IList<IConfigurationSource> Sources { get; }
    IConfigurationBuilder Add(IConfigurationSource source);
    IConfigurationRoot Build();
}
```

#### Реализация
```cs
public class ConfigurationBuilder : IConfigurationBuilder
{
    private readonly List<IConfigurationSource> _sources = new();
    public IList<IConfigurationSource> Sources => _sources;
    public IDictionary<string, object> Properties { get; } = new Dictionary<string, object>();

    public IConfigurationBuilder Add(IConfigurationSource source)
    {
        ThrowHelper.ThrowIfNull(source);

        _sources.Add(source);
        return this;
    }

    public IConfigurationRoot Build()
    {
        var providers = new List<IConfigurationProvider>();
        foreach (IConfigurationSource source in _sources)
        {
            IConfigurationProvider provider = source.Build(this);
            providers.Add(provider);
        }
        return new ConfigurationRoot(providers);
    }
}
```
