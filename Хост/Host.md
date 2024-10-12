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

```cs
using (cts)
using (linkedCts)
{
    CancellationToken token = linkedCts.Token;
```
  - Используются операторы using для автоматического освобождения ресурсов cts и linkedCts после выполнения блока.
  - CancellationToken token = linkedCts.Token; — создается токен отмены, который будет использоваться для всех асинхронных операций.

#### Ожидание запуска приложения
```cs
await _hostLifetime.WaitForStartAsync(token).ConfigureAwait(false);
token.ThrowIfCancellationRequested();
```
  - _hostLifetime.WaitForStartAsync(token) — ожидает, пока приложение начнет запускаться. Если токен отмены будет отменен, метод выбросит OperationCanceledException.
  - token.ThrowIfCancellationRequested(); — проверяет, был ли токен отменен, и выбрасывает исключение, если да.

#### Инициализация служб
```cs
List<Exception> exceptions = new();
_hostedServices ??= Services.GetRequiredService<IEnumerable<IHostedService>>();
_hostedLifecycleServices = GetHostLifecycles(_hostedServices);
_hostStarting = true;
bool concurrent = _options.ServicesStartConcurrently;
bool abortOnFirstException = !concurrent;
```
  - List<Exception> exceptions = new(); — создается список для хранения исключений, которые могут возникнуть во время запуска.
  - _hostedServices ??= Services.GetRequiredService<IEnumerable<IHostedService>>(); — получает все зарегистрированные службы типа IHostedService.
  - _hostedLifecycleServices = GetHostLifecycles(_hostedServices); — получает службы, реализующие интерфейс IHostedLifecycle.
  - _hostStarting = true; — устанавливает флаг, указывающий, что процесс запуска начался.
  - bool concurrent = _options.ServicesStartConcurrently; — определяет, должны ли службы запускаться параллельно.
  - bool abortOnFirstException = !concurrent; — определяет, должен ли процесс запуска прерываться при первом исключении (если службы запускаются последовательно).

#### Вызов валидаторов
```cs
IStartupValidator? validator = Services.GetService<IStartupValidator>();
if (validator is not null)
{
    try
    {
        validator.Validate();
    }
    catch (Exception ex)
    {
        exceptions.Add(ex);
        LogAndRethrow();
    }
}
```
  - IStartupValidator? validator = Services.GetService<IStartupValidator>(); — получает валидатор, если он зарегистрирован.
  - validator.Validate(); — вызывает метод валидации, который проверяет корректность настройки приложения.
  - Если возникает исключение, оно добавляется в список exceptions и вызывается метод LogAndRethrow() для логирования и прерывания процесса запуска.

#### Вызов методов жизненного цикла служб
```cs
if (_hostedLifecycleServices is not null)
{
    await ForeachService(_hostedLifecycleServices, token, concurrent, abortOnFirstException, exceptions,
        (service, token) => service.StartingAsync(token)).ConfigureAwait(false);
    LogAndRethrow();
}
```
  - ForeachService — это вспомогательный метод, который вызывает метод StartingAsync для каждой службы.
  - LogAndRethrow(); — вызывается для логирования и прерывания процесса запуска, если возникли исключения.





  
</details>
