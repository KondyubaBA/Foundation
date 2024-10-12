```cs
public static class JsonConfigurationExtensions
{
    public static IConfigurationBuilder AddJsonFile(this IConfigurationBuilder builder, string path)
    {
        return AddJsonFile(builder, provider: null, path: path, optional: false, reloadOnChange: false);
    }

    public static IConfigurationBuilder AddJsonFile(this IConfigurationBuilder builder, string path, bool optional)
    {
        return AddJsonFile(builder, provider: null, path: path, optional: optional, reloadOnChange: false);
    }

    public static IConfigurationBuilder AddJsonFile(this IConfigurationBuilder builder, string path, bool optional, bool reloadOnChange)
    {
        return AddJsonFile(builder, provider: null, path: path, optional: optional, reloadOnChange: reloadOnChange);
    }

    public static IConfigurationBuilder AddJsonFile(this IConfigurationBuilder builder, IFileProvider? provider, string path, bool optional, bool reloadOnChange)
    {
        ThrowHelper.ThrowIfNull(builder);

        if (string.IsNullOrEmpty(path))
        {
            throw new ArgumentException(SR.Error_InvalidFilePath, nameof(path));
        }

        return builder.AddJsonFile(s =>
        {
            s.FileProvider = provider;
            s.Path = path;
            s.Optional = optional;
            s.ReloadOnChange = reloadOnChange;
            s.ResolveFileProvider();
        });
    }

    public static IConfigurationBuilder AddJsonFile(this IConfigurationBuilder builder, Action<JsonConfigurationSource>? configureSource)
        => builder.Add(configureSource);

    public static IConfigurationBuilder AddJsonStream(this IConfigurationBuilder builder, Stream stream)
    {
        ThrowHelper.ThrowIfNull(builder);

        return builder.Add<JsonStreamConfigurationSource>(s => s.Stream = stream);
    }
}
```
