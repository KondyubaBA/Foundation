<details>
  <summary>WebApplication</summary>

```cs
public sealed class WebApplication : IHost, IApplicationBuilder, IEndpointRouteBuilder, IAsyncDisposable
{
    internal const string GlobalEndpointRouteBuilderKey = "__GlobalEndpointRouteBuilder";

    private readonly IHost _host;
    private readonly List<EndpointDataSource> _dataSources = new();

    internal WebApplication(IHost host)
    {
        _host = host;
        ApplicationBuilder = new ApplicationBuilder(host.Services, ServerFeatures);
        Logger = host.Services.GetRequiredService<ILoggerFactory>().CreateLogger(Environment.ApplicationName ?? nameof(WebApplication));

        Properties[GlobalEndpointRouteBuilderKey] = this;
    }

    public IServiceProvider Services => _host.Services;

    public IConfiguration Configuration => _host.Services.GetRequiredService<IConfiguration>();

    public IWebHostEnvironment Environment => _host.Services.GetRequiredService<IWebHostEnvironment>();

    public IHostApplicationLifetime Lifetime => _host.Services.GetRequiredService<IHostApplicationLifetime>();

    public ILogger Logger { get; }

    public ICollection<string> Urls => ServerFeatures.GetRequiredFeature<IServerAddressesFeature>().Addresses;

    IServiceProvider IApplicationBuilder.ApplicationServices
    {
        get => ApplicationBuilder.ApplicationServices;
        set => ApplicationBuilder.ApplicationServices = value;
    }

    internal IFeatureCollection ServerFeatures => _host.Services.GetRequiredService<IServer>().Features;
    IFeatureCollection IApplicationBuilder.ServerFeatures => ServerFeatures;

    internal IDictionary<string, object?> Properties => ApplicationBuilder.Properties;
    IDictionary<string, object?> IApplicationBuilder.Properties => Properties;

    internal ICollection<EndpointDataSource> DataSources => _dataSources;
    ICollection<EndpointDataSource> IEndpointRouteBuilder.DataSources => DataSources;

    internal ApplicationBuilder ApplicationBuilder { get; }

    IServiceProvider IEndpointRouteBuilder.ServiceProvider => Services;

    public static WebApplication Create(string[]? args = null) =>
        new WebApplicationBuilder(new() { Args = args }).Build();

    public static WebApplicationBuilder CreateBuilder() =>
        new(new());

    public static WebApplicationBuilder CreateBuilder(string[] args) =>
        new(new() { Args = args });

    public static WebApplicationBuilder CreateBuilder(WebApplicationOptions options) =>
        new(options);

    public Task StartAsync(CancellationToken cancellationToken = default) =>
        _host.StartAsync(cancellationToken);

    public Task StopAsync(CancellationToken cancellationToken = default) =>
        _host.StopAsync(cancellationToken);

    public Task RunAsync(string? url = null)
    {
        Listen(url);
        return HostingAbstractionsHostExtensions.RunAsync(this);
    }

    public void Run(string? url = null)
    {
        Listen(url);
        HostingAbstractionsHostExtensions.Run(this);
    }

    void IDisposable.Dispose() => _host.Dispose();

    public ValueTask DisposeAsync() => ((IAsyncDisposable)_host).DisposeAsync();

    internal RequestDelegate BuildRequestDelegate() => ApplicationBuilder.Build();
    RequestDelegate IApplicationBuilder.Build() => BuildRequestDelegate();

    // REVIEW: Should this be wrapping another type?
    IApplicationBuilder IApplicationBuilder.New()
    {
        var newBuilder = ApplicationBuilder.New();
        // Remove the route builder so branched pipelines have their own routing world
        newBuilder.Properties.Remove(GlobalEndpointRouteBuilderKey);
        return newBuilder;
    }

    public IApplicationBuilder Use(Func<RequestDelegate, RequestDelegate> middleware)
    {
        ApplicationBuilder.Use(middleware);
        return this;
    }

    IApplicationBuilder IEndpointRouteBuilder.CreateApplicationBuilder() => ((IApplicationBuilder)this).New();

    private void Listen(string? url)
    {
        if (url is null)
        {
            return;
        }

        var addresses = ServerFeatures.Get<IServerAddressesFeature>()?.Addresses;
        if (addresses is null)
        {
            throw new InvalidOperationException($"Changing the URL is not supported because no valid {nameof(IServerAddressesFeature)} was found.");
        }
        if (addresses.IsReadOnly)
        {
            throw new InvalidOperationException($"Changing the URL is not supported because {nameof(IServerAddressesFeature.Addresses)} {nameof(ICollection<string>.IsReadOnly)}.");
        }

        addresses.Clear();
        addresses.Add(url);
    }
}
```  
</details>
