<details>
  <summary>ConfigurationManager</summary>

```cs
[DebuggerDisplay("{DebuggerToString(),nq}")]
[DebuggerTypeProxy(typeof(ConfigurationManagerDebugView))]
public sealed class ConfigurationManager : IConfigurationManager, IConfigurationRoot, IDisposable
{
    private readonly ConfigurationSources _sources;
    private readonly ConfigurationBuilderProperties _properties;

    private readonly ReferenceCountedProviderManager _providerManager = new();

    private readonly List<IDisposable> _changeTokenRegistrations = new();
    private ConfigurationReloadToken _changeToken = new();

    public ConfigurationManager()
    {
        _sources = new ConfigurationSources(this);
        _properties = new ConfigurationBuilderProperties(this);

        // Make sure there's some default storage since there are no default providers.
        _sources.Add(new MemoryConfigurationSource());
    }

    public string? this[string key]
    {
        get
        {
            using ReferenceCountedProviders reference = _providerManager.GetReference();
            return ConfigurationRoot.GetConfiguration(reference.Providers, key);
        }
        set
        {
            using ReferenceCountedProviders reference = _providerManager.GetReference();
            ConfigurationRoot.SetConfiguration(reference.Providers, key, value);
        }
    }

    public IConfigurationSection GetSection(string key) => new ConfigurationSection(this, key);

    public IEnumerable<IConfigurationSection> GetChildren() => this.GetChildrenImplementation(null);

    IDictionary<string, object> IConfigurationBuilder.Properties => _properties;

    public IList<IConfigurationSource> Sources => _sources;

    IEnumerable<IConfigurationProvider> IConfigurationRoot.Providers => _providerManager.NonReferenceCountedProviders;

    /// <inheritdoc/>
    public void Dispose()
    {
        DisposeRegistrations();
        _providerManager.Dispose();
    }

    IConfigurationBuilder IConfigurationBuilder.Add(IConfigurationSource source)
    {
        ThrowHelper.ThrowIfNull(source);

        _sources.Add(source);
        return this;
    }

    IConfigurationRoot IConfigurationBuilder.Build() => this;

    IChangeToken IConfiguration.GetReloadToken() => _changeToken;

    void IConfigurationRoot.Reload()
    {
        using (ReferenceCountedProviders reference = _providerManager.GetReference())
        {
            foreach (IConfigurationProvider provider in reference.Providers)
            {
                provider.Load();
            }
        }

        RaiseChanged();
    }

    internal ReferenceCountedProviders GetProvidersReference() => _providerManager.GetReference();

    private void RaiseChanged()
    {
        var previousToken = Interlocked.Exchange(ref _changeToken, new ConfigurationReloadToken());
        previousToken.OnReload();
    }

    // Don't rebuild and reload all providers in the common case when a source is simply added to the IList.
    private void AddSource(IConfigurationSource source)
    {
        IConfigurationProvider provider = source.Build(this);

        provider.Load();
        _changeTokenRegistrations.Add(ChangeToken.OnChange(provider.GetReloadToken, RaiseChanged));

        _providerManager.AddProvider(provider);
        RaiseChanged();
    }

    // Something other than Add was called on IConfigurationBuilder.Sources or IConfigurationBuilder.Properties has changed.
    private void ReloadSources()
    {
        DisposeRegistrations();

        _changeTokenRegistrations.Clear();

        var newProvidersList = new List<IConfigurationProvider>();

        foreach (IConfigurationSource source in _sources)
        {
            newProvidersList.Add(source.Build(this));
        }

        foreach (IConfigurationProvider p in newProvidersList)
        {
            p.Load();
            _changeTokenRegistrations.Add(ChangeToken.OnChange(p.GetReloadToken, RaiseChanged));
        }

        _providerManager.ReplaceProviders(newProvidersList);
        RaiseChanged();
    }

    private void DisposeRegistrations()
    {
        // dispose change token registrations
        foreach (IDisposable registration in _changeTokenRegistrations)
        {
            registration.Dispose();
        }
    }

    private string DebuggerToString()
    {
        return $"Sections = {ConfigurationSectionDebugView.FromConfiguration(this, this).Count}";
    }

    private sealed class ConfigurationManagerDebugView
    {
        private readonly ConfigurationManager _current;

        public ConfigurationManagerDebugView(ConfigurationManager current)
        {
            _current = current;
        }

        [DebuggerBrowsable(DebuggerBrowsableState.RootHidden)]
        public ConfigurationSectionDebugView[] Items => ConfigurationSectionDebugView.FromConfiguration(_current, _current).ToArray();
    }

    private sealed class ConfigurationSources : IList<IConfigurationSource>
    {
        private readonly List<IConfigurationSource> _sources = new();
        private readonly ConfigurationManager _config;

        public ConfigurationSources(ConfigurationManager config)
        {
            _config = config;
        }

        public IConfigurationSource this[int index]
        {
            get => _sources[index];
            set
            {
                _sources[index] = value;
                _config.ReloadSources();
            }
        }

        public int Count => _sources.Count;

        public bool IsReadOnly => false;

        public void Add(IConfigurationSource source)
        {
            _sources.Add(source);
            _config.AddSource(source);
        }

        public void Clear()
        {
            _sources.Clear();
            _config.ReloadSources();
        }

        public bool Contains(IConfigurationSource source)
        {
            return _sources.Contains(source);
        }

        public void CopyTo(IConfigurationSource[] array, int arrayIndex)
        {
            _sources.CopyTo(array, arrayIndex);
        }

        public List<IConfigurationSource>.Enumerator GetEnumerator() => _sources.GetEnumerator();

        public int IndexOf(IConfigurationSource source)
        {
            return _sources.IndexOf(source);
        }

        public void Insert(int index, IConfigurationSource source)
        {
            _sources.Insert(index, source);
            _config.ReloadSources();
        }

        public bool Remove(IConfigurationSource source)
        {
            var removed = _sources.Remove(source);
            _config.ReloadSources();
            return removed;
        }

        public void RemoveAt(int index)
        {
            _sources.RemoveAt(index);
            _config.ReloadSources();
        }

        IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();

        IEnumerator<IConfigurationSource> IEnumerable<IConfigurationSource>.GetEnumerator() => GetEnumerator();
    }

    private sealed class ConfigurationBuilderProperties : IDictionary<string, object>
    {
        private readonly Dictionary<string, object> _properties = new();
        private readonly ConfigurationManager _config;

        public ConfigurationBuilderProperties(ConfigurationManager config)
        {
            _config = config;
        }

        public object this[string key]
        {
            get => _properties[key];
            set
            {
                _properties[key] = value;
                _config.ReloadSources();
            }
        }

        public ICollection<string> Keys => _properties.Keys;

        public ICollection<object> Values => _properties.Values;

        public int Count => _properties.Count;

        public bool IsReadOnly => false;

        public void Add(string key, object value)
        {
            _properties.Add(key, value);
            _config.ReloadSources();
        }

        public void Add(KeyValuePair<string, object> item)
        {
            ((IDictionary<string, object>)_properties).Add(item);
            _config.ReloadSources();
        }

        public void Clear()
        {
            _properties.Clear();
            _config.ReloadSources();
        }

        public bool Contains(KeyValuePair<string, object> item)
        {
            return _properties.Contains(item);
        }

        public bool ContainsKey(string key)
        {
            return _properties.ContainsKey(key);
        }

        public void CopyTo(KeyValuePair<string, object>[] array, int arrayIndex)
        {
            ((IDictionary<string, object>)_properties).CopyTo(array, arrayIndex);
        }

        public IEnumerator<KeyValuePair<string, object>> GetEnumerator()
        {
            return _properties.GetEnumerator();
        }

        public bool Remove(string key)
        {
            var wasRemoved = _properties.Remove(key);
            _config.ReloadSources();
            return wasRemoved;
        }

        public bool Remove(KeyValuePair<string, object> item)
        {
            var wasRemoved = ((IDictionary<string, object>)_properties).Remove(item);
            _config.ReloadSources();
            return wasRemoved;
        }

        public bool TryGetValue(string key, [NotNullWhen(true)] out object? value)
        {
            return _properties.TryGetValue(key, out value);
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return _properties.GetEnumerator();
        }
    }
}
```  
</details>
