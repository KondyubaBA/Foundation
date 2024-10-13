```cs
public class HostOptions
{
    public TimeSpan ShutdownTimeout { get; set; } = TimeSpan.FromSeconds(30);
    public TimeSpan StartupTimeout { get; set; } = Timeout.InfiniteTimeSpan;
    public bool ServicesStartConcurrently { get; set; }
    public bool ServicesStopConcurrently { get; set; }

    public BackgroundServiceExceptionBehavior BackgroundServiceExceptionBehavior { get; set; } =
        BackgroundServiceExceptionBehavior.StopHost;

    internal void Initialize(IConfiguration configuration)
    {
        var timeoutSeconds = configuration["shutdownTimeoutSeconds"];
        if (!string.IsNullOrEmpty(timeoutSeconds)
            && int.TryParse(timeoutSeconds, NumberStyles.None, CultureInfo.InvariantCulture, out var seconds))
        {
            ShutdownTimeout = TimeSpan.FromSeconds(seconds);
        }

        timeoutSeconds = configuration["startupTimeoutSeconds"];
        if (!string.IsNullOrEmpty(timeoutSeconds)
            && int.TryParse(timeoutSeconds, NumberStyles.None, CultureInfo.InvariantCulture, out seconds))
        {
            StartupTimeout = TimeSpan.FromSeconds(seconds);
        }

        var servicesStartConcurrently = configuration["servicesStartConcurrently"];
        if (!string.IsNullOrEmpty(servicesStartConcurrently)
            && bool.TryParse(servicesStartConcurrently, out bool startBehavior))
        {
            ServicesStartConcurrently = startBehavior;
        }

        var servicesStopConcurrently = configuration["servicesStopConcurrently"];
        if (!string.IsNullOrEmpty(servicesStopConcurrently)
            && bool.TryParse(servicesStopConcurrently, out bool stopBehavior))
        {
            ServicesStopConcurrently = stopBehavior;
        }
    }
}
```

- **ShutdownTimeout** Время, в течение которого хост будет ждать завершения всех сервисов при остановке. Если сервисы не завершатся за это время, хост будет завершен принудительно.
- **StartupTimeout** Время, в течение которого хост будет ждать запуска всех сервисов. Если значение установлено на Timeout.InfiniteTimeSpan, хост будет ждать неограниченное время.
- **ServicesStartConcurrently** Указывает, должны ли сервисы запускаться параллельно (true) или последовательно (false).
- **ServicesStopConcurrently** Указывает, должны ли сервисы останавливаться параллельно (true) или последовательно (false).
- **BackgroundServiceExceptionBehavior** Определяет поведение хоста при возникновении исключения в фоновом сервисе. Возможные значения:
    - StopHost: Хост останавливается при возникновении исключения.  
    - Ignore: Исключения игнорируются, и хост продолжает работать.  
