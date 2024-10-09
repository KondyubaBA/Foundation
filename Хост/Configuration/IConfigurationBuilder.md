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

IConfigurationBuilder в .NET является интерфейсом, который используется для конструирования объектов конфигурации. Это ключевой компонент в системе конфигурации .NET, который позволяет вам гибко и динамично управлять конфигурационными данными вашего приложения.

Вот разбор каждого члена этого интерфейса:

IDictionary<string, object> Properties { get; }: Это свойство предоставляет коллекцию дополнительных свойств, которые можно использовать для хранения пользовательских данных. Это может быть полезно для хранения дополнительной информации, которую вы хотите связать с конфигурацией.

IList<IConfigurationSource> Sources { get; }: Это свойство представляет коллекцию источников конфигурации, которые ConfigurationBuilder будет использовать для загрузки данных конфигурации. Каждый IConfigurationSource представляет источник данных, такой как файл, база данных, служба, и т.д.

IConfigurationBuilder Add(IConfigurationSource source);: Этот метод используется для добавления нового источника конфигурации в список источников. Это позволяет вам динамически добавлять различные источники данных для конфигурации вашего приложения.

IConfigurationRoot Build();: Этот метод используется для построения конечного объекта конфигурации (IConfigurationRoot). Он объединяет все источники конфигурации и возвращает объект, который можно использовать для доступа к конфигурационным данным.


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
