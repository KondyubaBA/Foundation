#### Структура

Строитель хоста (IHostBuilder)
  - ConfigureDefaults
      - ConfigureHostConfiguration (настройка хоста)
        - SetDefaultContentRoot (указать путь приложения)
        - AddEnvironmentVariables (добавить переменные среды)
        - AddCommandLineConfig (добавить параметры командной строки)
      - ConfigureAppConfiguration (настойка приложения)
