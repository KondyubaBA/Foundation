#### Структура

Строитель хоста (IHostBuilder)
  - ConfigureDefaults
      - ConfigureHostConfiguration (настройка хоста)
        - ApplyDefaultHostConfiguration
          - SetDefaultContentRoot (указать путь приложения)
          - AddEnvironmentVariables (добавить переменные среды)
          - AddCommandLineConfig (добавить параметры командной строки)
      - ConfigureAppConfiguration (настойка приложения)
        - ApplyDefaultAppConfiguration
          - AddJsonFile (добавляет appsettings.json)
          - AddJsonFile (добавялет appsettings.Development.json или appsettings.Production.json)
          - AddUserSecrets (добавляет серкреты пользователя)
          - AddEnvironmentVariables (добавляет все переменные среды)
          - AddCommandLineConfig (добавляет аргументы командной строки)
      - ConfigureServices (добавляет сервисы)
        - AddDefaultServices
          - AddLogging (добавялет логгер)
            - AddConfiguration (настраивает логгер секция Logging)
          - AddMetrics (добавялет метрики)
            - AddConfiguration (настраивает метрики секция Metrics)
      - UseServiceProviderFactory (создает фабрику)
        - DefaultServiceProviderFactory(создает фабрику)
          - CreateDefaultServiceProviderOptions (конфигурация опций провайдера сервисов для фабрики)
         
#### Build
  - InitializeHostConfiguration();
    - IConfigurationBuilder (создает тип для конфигурации)
    - 



