# Analytics Service

Сервис аналитики платформы CorporationX. Он собирает события пользовательской активности, сохраняет их в PostgreSQL и предоставляет внутреннюю логику для получения аналитики по получателю, типу события и периоду.

Сервис запускается как Spring Boot-приложение и работает на порту `8086`.

## Функциональность

- принимает события о создании комментариев из Kafka topic `comment_events`;
- преобразует событие комментария в аналитическое событие:
  - `postId` используется как `receiverId`;
  - `authorId` используется как `actorId`;
  - тип события устанавливается в `POST_COMMENT`;
  - `createdAt` используется как время получения события;
- валидирует события перед сохранением: идентификаторы должны быть положительными, тип и время обязательны, время события не может быть в будущем;
- сохраняет события в таблицу `analytics_event` в PostgreSQL;
- фильтрует аналитику по `receiverId` и `EventType`;
- поддерживает готовые интервалы `LAST_HOUR`, `LAST_DAY`, `LAST_WEEK`, `LAST_MONTH` и произвольный диапазон `from`–`to`;
- возвращает найденные события в порядке от новых к старым;
- применяет Liquibase-миграции при запуске приложения.

Поддерживаемые типы аналитических событий описаны в `EventType`. Сейчас обработчик Kafka подключен к событиям комментариев.

## Технологии

- Java 17
- Spring Boot
- Spring Data JPA
- Apache Kafka
- PostgreSQL
- Redis
- Liquibase
- Gradle
- MapStruct и Lombok
- JUnit 5, Mockito и Testcontainers

## Требования

- JDK 17;
- PostgreSQL;
- Kafka;
- Redis.

При локальном запуске сервис ожидает следующие значения по умолчанию:

| Компонент | Адрес по умолчанию |
|---|---|
| PostgreSQL | `localhost:5432`, база `postgres`, пользователь `user`, пароль `password` |
| Kafka | `localhost:9094` |
| Redis | `localhost:6379` |
| Project Service | `localhost:8082` |

## Запуск локально

1. Запустите PostgreSQL, Kafka и Redis. В общей инфраструктуре проекта зависимости можно поднять из директории `infra`.
2. Соберите приложение:

```shell
./gradlew clean bootJar
```

Для Windows:

```shell
gradlew.bat clean bootJar
```

3. Запустите собранный JAR:

```shell
java -jar build/libs/service.jar
```

Для подключения к сервисам с другими адресами передайте переменные окружения:

```shell
DB_HOST=localhost
DB_PORT=5432
DB_NAME=postgres
DB_USERNAME=user
DB_PASSWORD=password
REDIS_HOST=localhost
REDIS_PORT=6379
KAFKA_BOOTSTRAP_SERVERS=localhost:9094
PROJECT_SERVICE_HOST=localhost
PROJECT_SERVICE_PORT=8082
```

Указанные переменные имеют значения по умолчанию и могут быть переопределены при запуске приложения.

## Запуск в Docker

Docker-образ собирается из уже созданного JAR-файла:

```shell
gradlew.bat clean bootJar
docker build -t analytics-service .
```

Запустите контейнер в сети, где доступны PostgreSQL, Kafka и Redis:

```shell
docker run --rm --name analytics-service --network <network-name> -p 8086:8086 `
  -e DB_HOST=postgres `
  -e DB_PORT=5432 `
  -e DB_NAME=postgres `
  -e DB_USERNAME=user `
  -e DB_PASSWORD=password `
  -e REDIS_HOST=redis `
  -e REDIS_PORT=6379 `
  -e KAFKA_BOOTSTRAP_SERVERS=kafka:9092 `
  analytics-service
```

Параметр `--network <network-name>` подключает контейнер к сети, в которой уже запущены зависимости. Значения `DB_HOST`, `REDIS_HOST` и `KAFKA_BOOTSTRAP_SERVERS` должны соответствовать именам сервисов и портам внутри Docker-сети. Dockerfile открывает порт `8086`.

## Тесты

Запуск тестов:

```shell
gradlew.bat test
```

Тесты используют JUnit 5, Mockito и Testcontainers для проверки работы с PostgreSQL и Redis.
