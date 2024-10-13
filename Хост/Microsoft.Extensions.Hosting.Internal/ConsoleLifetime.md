<details>
    <summary>interface</summary>
    
```cs
 public interface IDisposable
 {
     void Dispose();
 }
    
public interface IHostLifetime
{
    Task WaitForStartAsync(CancellationToken cancellationToken);
    Task StopAsync(CancellationToken cancellationToken);
}
```    
</details>

<details>
    <summary>partial ConsoleLifetime</summary>

```cs
[UnsupportedOSPlatform("android")]
[UnsupportedOSPlatform("browser")]
[UnsupportedOSPlatform("ios")]
[UnsupportedOSPlatform("tvos")]
public partial class ConsoleLifetime : IHostLifetime, IDisposable
{
    private CancellationTokenRegistration _applicationStartedRegistration;
    private CancellationTokenRegistration _applicationStoppingRegistration;

    public ConsoleLifetime(IOptions<ConsoleLifetimeOptions> options, IHostEnvironment environment, IHostApplicationLifetime applicationLifetime, IOptions<HostOptions> hostOptions)
        : this(options, environment, applicationLifetime, hostOptions, NullLoggerFactory.Instance) { }

    public ConsoleLifetime(IOptions<ConsoleLifetimeOptions> options, IHostEnvironment environment, IHostApplicationLifetime applicationLifetime, IOptions<HostOptions> hostOptions, ILoggerFactory loggerFactory)
    {
        ThrowHelper.ThrowIfNull(options?.Value, nameof(options));
        ThrowHelper.ThrowIfNull(applicationLifetime);
        ThrowHelper.ThrowIfNull(environment);
        ThrowHelper.ThrowIfNull(hostOptions?.Value, nameof(hostOptions));
        ThrowHelper.ThrowIfNull(loggerFactory);

        Options = options.Value;
        Environment = environment;
        ApplicationLifetime = applicationLifetime;
        HostOptions = hostOptions.Value;
        Logger = loggerFactory.CreateLogger("Microsoft.Hosting.Lifetime");
    }

    private ConsoleLifetimeOptions Options { get; }

    private IHostEnvironment Environment { get; }

    private IHostApplicationLifetime ApplicationLifetime { get; }

    private HostOptions HostOptions { get; }

    private ILogger Logger { get; }

    public Task WaitForStartAsync(CancellationToken cancellationToken)
    {
        if (!Options.SuppressStatusMessages)
        {
            _applicationStartedRegistration = ApplicationLifetime.ApplicationStarted.Register(state =>
            {
                ((ConsoleLifetime)state!).OnApplicationStarted();
            },
            this);
            _applicationStoppingRegistration = ApplicationLifetime.ApplicationStopping.Register(state =>
            {
                ((ConsoleLifetime)state!).OnApplicationStopping();
            },
            this);
        }

        RegisterShutdownHandlers();

        // Console applications start immediately.
        return Task.CompletedTask;
    }

    private partial void RegisterShutdownHandlers();

    private void OnApplicationStarted()
    {
        Logger.LogInformation("Application started. Press Ctrl+C to shut down.");
        Logger.LogInformation("Hosting environment: {EnvName}", Environment.EnvironmentName);
        Logger.LogInformation("Content root path: {ContentRoot}", Environment.ContentRootPath);
    }

    private void OnApplicationStopping()
    {
        Logger.LogInformation("Application is shutting down...");
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        // There's nothing to do here
        return Task.CompletedTask;
    }

    public void Dispose()
    {
        UnregisterShutdownHandlers();

        _applicationStartedRegistration.Dispose();
        _applicationStoppingRegistration.Dispose();
    }

    private partial void UnregisterShutdownHandlers();
}
```
</details>

<details>
    <summary>partial ConsoleLifetime</summary>

```cs
public partial class ConsoleLifetime : IHostLifetime
{
    private PosixSignalRegistration? _sigIntRegistration;
    private PosixSignalRegistration? _sigQuitRegistration;
    private PosixSignalRegistration? _sigTermRegistration;

    private partial void RegisterShutdownHandlers()
    {
        Action<PosixSignalContext> handler = HandlePosixSignal;
        _sigIntRegistration = PosixSignalRegistration.Create(PosixSignal.SIGINT, handler);
        _sigQuitRegistration = PosixSignalRegistration.Create(PosixSignal.SIGQUIT, handler);
        _sigTermRegistration = PosixSignalRegistration.Create(PosixSignal.SIGTERM, handler);
    }

    private void HandlePosixSignal(PosixSignalContext context)
    {
        Debug.Assert(context.Signal == PosixSignal.SIGINT || context.Signal == PosixSignal.SIGQUIT || context.Signal == PosixSignal.SIGTERM);

        context.Cancel = true;
        ApplicationLifetime.StopApplication();
    }

    private partial void UnregisterShutdownHandlers()
    {
        _sigIntRegistration?.Dispose();
        _sigQuitRegistration?.Dispose();
        _sigTermRegistration?.Dispose();
    }
}
```
</details>

<details>
    <summary>Описание</summary>

```cs
public Task WaitForStartAsync(CancellationToken cancellationToken)
{
    if (!Options.SuppressStatusMessages)
    {
        _applicationStartedRegistration = ApplicationLifetime.ApplicationStarted.Register(state =>
        {
            ((ConsoleLifetime)state!).OnApplicationStarted();
        },
        this);
        _applicationStoppingRegistration = ApplicationLifetime.ApplicationStopping.Register(state =>
        {
            ((ConsoleLifetime)state!).OnApplicationStopping();
        },
        this);
    }

    RegisterShutdownHandlers();

    // Console applications start immediately.
    return Task.CompletedTask;
}

```
</details>
