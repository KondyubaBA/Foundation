```cs
public interface IConfiguration
{
    string? this[string key] { get; set; }

    IConfigurationSection GetSection(string key);

    IEnumerable<IConfigurationSection> GetChildren();

    IChangeToken GetReloadToken();
}


 public interface IConfigurationSection : IConfiguration
 {
     string Key { get; }
     string Path { get; }
     string? Value { get; set; }
 }
```
