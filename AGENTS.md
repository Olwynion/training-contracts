# Инструкции по разработке микросервисов

## Общие правила
- Не менять версию .NET, csproj или зависимости без согласования
- `Nullable` reference types: `<Nullable>enable</Nullable>`
- 3 официальных гайдлайна (Microsoft + Google style)
- Архитектура: Onion (Hexagonal)
- Все изменения через feature branch + Pull Request в main
- В main должен находиться только каркас сервиса

## Архитектура
- **Domain (Core)** — сущности, VO, интерфейсы репозиториев, интерфейсы сервисов
- **Services** — proto-файлы, DI, внешние интеграции
- **Infrastructure** — реализации репозиториев, БД, реализации сервисов (BCrypt, JWT)
- **Presentation (API)** — Program.cs, gRPC сервисы, Handlers (CQRS), конфиги
- Зависимости внутрь, Domain без внешних зависимостей
- Интерфейсы в Domain, реализации в Infrastructure

## DDD
- `Entities/`, `ValueObjects/`, `Repositories/`, `Enums/` — в Domain
- `Services/` — интерфейсы сервисов в Domain (IAuthTokenService, IPasswordHasher)
- `Mappings/` — в Infrastructure

## Proto
- Proto-файлы находятся в слое Services
- RPC именуются по шаблону: `Verb(VerbRequest) returns (VerbResponse)`
- Request и Response лежат в том же proto-файле, что и сервис
- **Не использовать** `common.proto`, `google.protobuf.Empty`, `IdRequest`
- Каждое RPC имеет собственную пару Request/Response (даже для void-операций)

## Handlers (CQRS)
- Все Handlers/Queries/Results находятся в папке `Handlers/` на слое API (Presentation)
- Commands и Queries records лежат в том же `.cs` файле, что и Handler
- Shared результаты (например AuthResult) выносятся в отдельный файл

## gRPC
- gRPC сервисы находятся в папке `Grpc/` на слое API (Presentation)
- Наследуются от `*ServiceBase` (генерируется из proto)

## Tests
- Единый тестовый проект `tests/<Service>.Tests/` без разделения по слоям
- Внутри — папки по функциональным группам: `Handlers/`, `GrpcServices/`, `Domain/`, `Services/`
- Покрытие: handlers, gRPC сервисы, обычные сервисы, domain

## Именование
- `PascalCase` для классов/методов/свойств
- `_camelCase` для приватных полей
- `I` для интерфейсов: `IUserRepository`

## Организация кода
- Каждый класс в отдельном файле, имя = класс

## Асинхронность
- `async/await`, без `.Result/.Wait()`, `ConfigureAwait(false)`

## БД
- Dapper, async методы, строки не в коде

## Тесты
- xUnit + Moq
- `<ClassName>Tests`
- Минимум 60% покрытия (Domain + Services, не репозитории)

## Proto генерация
- `platform grpc restore --proto-source vendor.protogen`

## Docker
- PostgreSQL поднимается через `docker compose up -d` из корня сервиса
- Порт PostgreSQL: 5432, база: `training_auth`
- Инициализация таблиц через `db/init.sql`

## Run
- `dotnet run --project src/<Service>/<Service>.csproj`
- Сервисы слушают на портах: auth — 5002, training — 5003, ai — 5004, gateway — 5000
- gRPC reflection включена (Postman может получить proto через server reflection)

## Before commit
- `dotnet build`
- `dotnet test`

## AppSettings
- `appsettings.Local.json` — локальные настройки (gitignored)
- `appsettings.Local.json` загружается в `Program.cs` через `AddJsonFile("appsettings.Local.json", optional: true)`

## Фичи C#
- File-scoped namespaces (C# 10+)
- primary constructors (C# 12+)
