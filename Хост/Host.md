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
  
</details>
