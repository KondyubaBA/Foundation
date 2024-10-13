<details>
  <summary>Interface</summary>

```cs
public interface IHostApplicationLifetime
{
    CancellationToken ApplicationStarted { get; }

    CancellationToken ApplicationStopping { get; }

    CancellationToken ApplicationStopped { get; }

    void StopApplication();
}
``` 
</details>


<details>
  <summary>ApplicationLifetime</summary>

```cs
[DebuggerDisplay("ApplicationStarted = {ApplicationStarted.IsCancellationRequested}, " +
        "ApplicationStopping = {ApplicationStopping.IsCancellationRequested}, " +
        "ApplicationStopped = {ApplicationStopped.IsCancellationRequested}")]
#pragma warning disable CS0618 // Type or member is obsolete
    public class ApplicationLifetime : IApplicationLifetime, IHostApplicationLifetime
#pragma warning restore CS0618 // Type or member is obsolete
{
    private readonly CancellationTokenSource _startedSource = new CancellationTokenSource();
    private readonly CancellationTokenSource _stoppingSource = new CancellationTokenSource();
    private readonly CancellationTokenSource _stoppedSource = new CancellationTokenSource();
    private readonly ILogger<ApplicationLifetime> _logger;

    public ApplicationLifetime(ILogger<ApplicationLifetime> logger)
    {
        _logger = logger;
    }

    public CancellationToken ApplicationStarted => _startedSource.Token;

    public CancellationToken ApplicationStopping => _stoppingSource.Token;

    public CancellationToken ApplicationStopped => _stoppedSource.Token;

    public void StopApplication()
    {
        lock (_stoppingSource)
        {
            try
            {
                _stoppingSource.Cancel();
            }
            catch (Exception ex)
            {
                _logger.ApplicationError(LoggerEventIds.ApplicationStoppingException,
                                         "An error occurred stopping the application",
                                         ex);
            }
        }
    }

    public void NotifyStarted()
    {
        try
        {
            _startedSource.Cancel();
        }
        catch (Exception ex)
        {
            _logger.ApplicationError(LoggerEventIds.ApplicationStartupException,
                                     "An error occurred starting the application",
                                     ex);
        }
    }

    public void NotifyStopped()
    {
        try
        {
            _stoppedSource.Cancel();
        }
        catch (Exception ex)
        {
            _logger.ApplicationError(LoggerEventIds.ApplicationStoppedException,
                                     "An error occurred stopping the application",
                                     ex);
        }
    }
}
```
</details>
