# mApps API Contracts

Централизованный репозиторий gRPC proto-контрактов для микросервисной платформы **mApps** — мобильного приложения.

---

## Содержание

- [О проекте](#о-проекте)
- [Структура репозитория](#структура-репозитория)
- [Сервисы и API](#сервисы-и-api)
  - [Auth — Аутентификация](#auth--аутентификация)
  - [Products — Продукты](#products--продукты)
  - [AssetManager — Медиафайлы](#assetmanager--медиафайлы)
- [Быстрый старт](#быстрый-старт)
- [Инструменты](#инструменты)

---

## О проекте

Данный репозиторий хранит единый источник истины для всех gRPC-контрактов платформы mApps. Каждый микросервис подключает этот репозиторий как зависимость и генерирует код на своей стороне.

**Технологический стек:**
- [Protocol Buffers v3](https://protobuf.dev/)
- [Buf](https://buf.build/) — линтинг, управление зависимостями
- [Google API Annotations](https://github.com/googleapis/googleapis) — HTTP/REST-транскодинг поверх gRPC

---

## Структура репозитория

```
proto/
├── auth/
│   └── v1/
│       └── auth.proto              # Аутентификация пользователей
├── products/
│   └── v1/
│       └── products.proto          # Каталог продуктов
└── asset_manager/
    └── v1/
        └── asset_manager.proto     # Управление медиафайлами (изображения, 3D модели)
buf.yaml                            # Конфигурация Buf
buf.lock                            # Lockfile зависимостей
Makefile                            # Команды для работы с зависимостями
```

---

## Сервисы и API

### Auth — Аутентификация

**Package:** `mapps.auth.v1`

| RPC | HTTP | Описание |
|-----|------|----------|
| `Register` | `POST /api/v1/mobile/auth/register` | Регистрация нового пользователя |
| `Login` | `POST /api/v1/mobile/auth/login` | Вход в систему |

---

### Products — Продукты

**Package:** `mapps.products.v1`

| RPC | HTTP | Описание |
|-----|------|----------|
| `GetProduct` | `GET /api/v1/mobile/products/{product_id}` | Получить продукт по ID |
| `ListProducts` | `GET /api/v1/mobile/products` | Список продуктов с фильтрацией и пагинацией |
| `CreateProduct` | `POST /api/v1/mobile/products` | Создать продукт |
| `UpdateProduct` | `PUT /api/v1/mobile/products/{product_id}` | Обновить продукт |

Продукт ссылается на медиафайлы через `virtual_image_id` и `model_id` из сервиса AssetManager.

---

### AssetManager — Медиафайлы

**Package:** `mapps.asset_manager.v1`

Сервис управляет загрузкой файлов в S3/MinIO через presigned URL — клиент получает временную ссылку и грузит файл напрямую в хранилище, минуя бэкенд.

| RPC | HTTP | Описание |
|-----|------|----------|
| `GetAssetUploadURL` | `POST /api/v1/assets/textures/upload-url` | Получить presigned URL для загрузки изображения/текстуры |
| `GetAsset` | `GET /api/v1/assets/textures/{asset_id}` | Получить изображение/текстуру по ID |
| `GetModelUploadURL` | `POST /api/v1/assets/models/upload-url` | Получить presigned URL для загрузки 3D модели |
| `GetModel` | `GET /api/v1/assets/models/{model_id}` | Получить 3D модель по ID |

**Связь сущностей:**
```
Product
  virtual_image_id ──→ Asset
  model_id         ──→ Model3D
                           thumbnail_id ──→ Asset
                           texture_ids[] ──→ Asset[]
```

---

## Быстрый старт

### Требования

- [Buf CLI](https://buf.build/docs/installation) `>= 1.x`

### Обновление зависимостей

```bash
make depup
```

### Линтинг

```bash
buf lint
```

---

## Инструменты

| Инструмент | Назначение |
|------------|------------|
| [Buf](https://buf.build/) | Линтинг, форматирование, управление зависимостями proto |
| [Google API Annotations](https://buf.build/googleapis/googleapis) | HTTP-транскодинг gRPC (REST gateway) |
| [grpcurl](https://github.com/fullstorydev/grpcurl) | Ручное тестирование gRPC-эндпоинтов |
