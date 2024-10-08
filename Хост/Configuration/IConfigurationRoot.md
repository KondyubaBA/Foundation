#### Абстракция
```cs
 public interface IConfigurationRoot : IConfiguration
 {
     void Reload();
     IEnumerable<IConfigurationProvider> Providers { get; }
 }
```

#### Реализация
```cs
[DebuggerDisplay("{DebuggerToString(),nq}")]
[DebuggerTypeProxy(typeof(ConfigurationRootDebugView))]
public class ConfigurationRoot : IConfigurationRoot, IDisposable
{
    private readonly IList<IConfigurationProvider> _providers;
    private readonly List<IDisposable> _changeTokenRegistrations;
    private ConfigurationReloadToken _changeToken = new ConfigurationReloadToken();

    public ConfigurationRoot(IList<IConfigurationProvider> providers)
    {
        ThrowHelper.ThrowIfNull(providers);

        _providers = providers;
        _changeTokenRegistrations = new List<IDisposable>(providers.Count);
        foreach (IConfigurationProvider p in providers)
        {
            p.Load();
            _changeTokenRegistrations.Add(ChangeToken.OnChange(p.GetReloadToken, RaiseChanged));
        }
    }

    public IEnumerable<IConfigurationProvider> Providers => _providers;

    public string? this[string key]
    {
        get => GetConfiguration(_providers, key);
        set => SetConfiguration(_providers, key, value);
    }

    public IEnumerable<IConfigurationSection> GetChildren() => this.GetChildrenImplementation(null);

    public IChangeToken GetReloadToken() => _changeToken;

    public IConfigurationSection GetSection(string key)
        => new ConfigurationSection(this, key);

    public void Reload()
    {
        foreach (IConfigurationProvider provider in _providers)
        {
            provider.Load();
        }
        RaiseChanged();
    }

    private void RaiseChanged()
    {
        ConfigurationReloadToken previousToken = Interlocked.Exchange(ref _changeToken, new ConfigurationReloadToken());
        previousToken.OnReload();
    }

    /// <inheritdoc />
    public void Dispose()
    {
        // dispose change token registrations
        foreach (IDisposable registration in _changeTokenRegistrations)
        {
            registration.Dispose();
        }

        // dispose providers
        foreach (IConfigurationProvider provider in _providers)
        {
            (provider as IDisposable)?.Dispose();
        }
    }

    internal static string? GetConfiguration(IList<IConfigurationProvider> providers, string key)
    {
        for (int i = providers.Count - 1; i >= 0; i--)
        {
            IConfigurationProvider provider = providers[i];

            if (provider.TryGet(key, out string? value))
            {
                return value;
            }
        }

        return null;
    }

    internal static void SetConfiguration(IList<IConfigurationProvider> providers, string key, string? value)
    {
        if (providers.Count == 0)
        {
            throw new InvalidOperationException(SR.Error_NoSources);
        }

        foreach (IConfigurationProvider provider in providers)
        {
            provider.Set(key, value);
        }
    }

    private string DebuggerToString()
    {
        return $"Sections = {ConfigurationSectionDebugView.FromConfiguration(this, this).Count}";
    }

    private sealed class ConfigurationRootDebugView
    {
        private readonly ConfigurationRoot _current;

        public ConfigurationRootDebugView(ConfigurationRoot current)
        {
            _current = current;
        }

        [DebuggerBrowsable(DebuggerBrowsableState.RootHidden)]
        public ConfigurationSectionDebugView[] Items => ConfigurationSectionDebugView.FromConfiguration(_current, _current).ToArray();
    }
}
```
