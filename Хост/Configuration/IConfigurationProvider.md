```cs
public interface IConfigurationProvider
{
    bool TryGet(string key, out string? value);
    void Set(string key, string? value);
    IChangeToken GetReloadToken();
    void Load();
    IEnumerable<string> GetChildKeys(IEnumerable<string> earlierKeys, string? parentPath);
}
```

bool TryGet(string key, out string? value);: Этот метод используется для извлечения значения конфигурации по его ключу. Если ключ найден, значение устанавливается и возвращается true. Если ключ не найден, значение устанавливается в null и возвращается false.

void Set(string key, string? value);: Этот метод позволяет установить значение конфигурации для определенного ключа. Это может быть полезно для динамического изменения конфигурации во время выполнения.

IChangeToken GetReloadToken();: Этот метод возвращает токен изменений, который может быть использован для отслеживания изменений в конфигурации. Если конфигурация изменяется, токен сгенерирует уведомление об изменении.

void Load();: Этот метод инициирует перезагрузку данных конфигурации из источника. Это может быть полезно при работе с источниками конфигурации, которые могут меняться во время выполнения.

IEnumerable<string> GetChildKeys(IEnumerable<string> earlierKeys, string? parentPath);: Этот метод используется для получения всех ключей, которые являются дочерними для указанного родительского пути. Это полезно при работе с иерархическими структурами конфигурации.
