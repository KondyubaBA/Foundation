Microsoft.Extensions.Hosting.Internal.Host
<details>
  <summary>IHost</summary>

```cs
public interface IHost : IDisposable
{
    IServiceProvider Services { get; }
    Task StartAsync(CancellationToken cancellationToken = default);
    Task StopAsync(CancellationToken cancellationToken = default);
}
```
</details>


<details>
  <summary>StartAsync</summary>

Запуск логгера
```cs
_logger.Starting();
```
Логика для ограничения по времени выполнения задачи
```cs
CancellationTokenSource? cts = null;
CancellationTokenSource linkedCts;
if (_options.StartupTimeout != Timeout.InfiniteTimeSpan)
{
    cts = new CancellationTokenSource(_options.StartupTimeout);
    linkedCts = CancellationTokenSource.CreateLinkedTokenSource(cts.Token, cancellationToken, _applicationLifetime.ApplicationStopping);
}
else
{
    linkedCts = CancellationTokenSource.CreateLinkedTokenSource(cancellationToken, _applicationLifetime.ApplicationStopping);
}
```

  
</details>
