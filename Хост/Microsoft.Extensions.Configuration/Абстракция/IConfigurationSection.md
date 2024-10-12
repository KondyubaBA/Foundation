```cs
 public interface IConfigurationSection : IConfiguration
 {
     string Key { get; }
     string Path { get; }
     string? Value { get; set; }
 }
```