```cs
public class HostBuilderContext
{
    public HostBuilderContext(IDictionary<object, object> properties)
    {
        ThrowHelper.ThrowIfNull(properties);

        Properties = properties;
    }

    public IHostEnvironment HostingEnvironment { get; set; } = null!;

    public IConfiguration Configuration { get; set; } = null!;

    public IDictionary<object, object> Properties { get; }
}
```
