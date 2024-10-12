#### Абстракция
```cs
 public interface IConfigurationRoot : IConfiguration
 {
     void Reload();
     IEnumerable<IConfigurationProvider> Providers { get; }
 }
```

#### Итог
```cs
public interface IConfigurationRoot
{
      void Reload();
      IEnumerable<IConfigurationProvider> Providers { get; }
      
      //IConfiguration
      string? this[string key] { get; set; }
      IConfigurationSection GetSection(string key);
      IEnumerable<IConfigurationSection> GetChildren();
      IChangeToken GetReloadToken(); 
}
```
