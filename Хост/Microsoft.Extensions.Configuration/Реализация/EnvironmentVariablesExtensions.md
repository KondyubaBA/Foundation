```cs
public static class EnvironmentVariablesExtensions
{
    public static IConfigurationBuilder AddEnvironmentVariables(this IConfigurationBuilder configurationBuilder)
    {
        configurationBuilder.Add(new EnvironmentVariablesConfigurationSource());
        return configurationBuilder;
    }

    public static IConfigurationBuilder AddEnvironmentVariables(this IConfigurationBuilder configurationBuilder, string? prefix)
    {
        configurationBuilder.Add(new EnvironmentVariablesConfigurationSource
        {
            Prefix = prefix
        });
        return configurationBuilder;
    }

    public static IConfigurationBuilder AddEnvironmentVariables(this IConfigurationBuilder builder, Action<EnvironmentVariablesConfigurationSource>? configureSource)
    {
        return builder.Add(configureSource);
    }
```
