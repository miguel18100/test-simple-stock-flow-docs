# Plan Laravel + React — migración a Onion

## 1. Propósito

Este documento adapta el spec existente a Laravel + React sin sustituirlo. La fuente de verdad funcional sigue siendo `spec-python/`; este directorio contiene únicamente decisiones de traducción tecnológica y su trazabilidad.

## 2. Arquitectura objetivo

Backend Laravel con 4 anillos + Bootstrap:

- `app/Domain`: entidades, value objects, invariantes y excepciones de negocio. No importa Laravel.
- `app/Application`: casos de uso, puertos inbound/outbound, DTOs de aplicación y modelos de aplicación.
- `app/Infrastructure`: Eloquent, repositorios, mappers, JWT, almacenamiento, transacciones y configuración.
- `app/Presentation`: HTTP, requests de forma, resources, serialización, errores y middleware.
- `app/Bootstrap`: único punto de ensamblaje de implementaciones.

Frontend React:

- `src/domain`
- `src/application`
- `src/infrastructure`
- `src/features`

## 3. Estructura backend

```text
app/
├── Domain/
│   ├── Model/
│   ├── ValueObject/
│   ├── Exception/
│   └── Service/.gitkeep
├── Application/
│   ├── Ports/Inbound/
│   ├── Ports/Outbound/
│   ├── UseCase/
│   ├── Model/
│   └── Exception/
├── Infrastructure/
│   ├── Persistence/Model/
│   ├── Persistence/Mapper/
│   ├── Persistence/Repository/
│   ├── Security/
│   ├── Storage/
│   ├── Configuration/
│   └── Logging/
├── Presentation/
│   ├── Http/Controller/
│   ├── Http/Request/
│   ├── Http/Resource/
│   ├── Http/Serialization/
│   ├── Http/ProblemDetails/
│   └── Middleware/
└── Bootstrap/
    └── PortBindingsServiceProvider.php

database/migrations/
database/seeders/
routes/api.php
```

## 4. Traducción de tecnología

| Spec original | Laravel | Regla |
|---|---|---|
| Pydantic DTO | FormRequest + DTO de Application + Resource | Request valida forma; dominio valida negocio |
| SQLAlchemy model | Eloquent Model en Infrastructure | Nunca Eloquent en Domain |
| Repository | Interface en Application + implementación Eloquent | El controlador nunca recibe repositorio |
| UnitOfWork | `UnitOfWork::run(callable)` + `DB::transaction()` | `DB::transaction()` solo Infrastructure |
| Decimal | `Brick\\Math\\BigDecimal` | `Money` en Domain; escala 2, HALF_UP |
| JWT | Adaptador en Infrastructure | El puerto es `TokenGenerator` |
| PasswordHasher | Argon2 en Infrastructure | El dominio no conoce el algoritmo |
| Alembic migration | Laravel migration | El esquema es propiedad de `api` |
| Testcontainers | Docker Compose | La base del compose empieza vacía |
| pytest | PHPUnit 11 | Domain/Application sin boot de Laravel cuando corresponda |
| import-linter | Deptrac | Verifica R-01…R-06 |
| mypy --strict | PHPStan max + Larastan | Tipado estático |

## 5. Migraciones de base de datos

El modelo físico del spec se conserva: tablas singulares `category`, `product`, `user`, `sale`, `sale_item`.

La migración Laravel debe producir, como mínimo:

1. `category`: UUID `CHAR(36)`, `name`, PK, `uq_category_name`, `ck_category_name_not_blank`.
2. `product`: UUID, `name`, `DECIMAL(12,2)`, `stock`, `category_id`, `image_key`, `deleted_at`, `version`, índices y CHECK definidos por el spec.
3. `user`: UUID, `username`, `password_hash`, `role`, unicidad y CHECK de normalización/rol/hash.
4. `sale`: UUID, `sold_at`, `sold_by_username`, `sold_by_user_id`, índices y FK restrictiva.
5. `sale_item`: UUID, `sale_id`, `product_id`, `product_name`, `category_name`, `quantity`, `unit_price`, unicidad por `(sale_id, product_id)`, FKs y CHECK.
6. `seed_categories`: las cinco categorías con UUID fijos del `data-model.md`.

No se crean columnas `total` ni `subtotal`. Ambos son valores derivados.

`product.version` y `product.deleted_at` existen únicamente en persistencia; no se añaden a `Domain\\Model\\Product`.

El administrador inicial NO se inserta mediante SQL ni Seeder. Se crea mediante el flujo de aplicación usando `ADMIN_EMAIL` y `ADMIN_PASSWORD`, pasando por `PasswordHasher`.

## 6. Orden de ejecución

```text
P0 docs/onion-decision
  ↓
P1 Domain + barreras de arquitectura
  ↓
P2 migrations + seed_categories + verify.sh
  ↓
P3 casos de uso
  ↓
P4 concurrencia, baja lógica, congelados, invariantes
  ↓
P5 React contra API real
  ↓
P6 page/tool/logging
  ↓
P7 documentación y verificación final
```

## 7. Regla de oro para la migración

No se migra archivo por archivo desde Python/.NET. Se migra decisión por decisión:

1. localizar la decisión en `spec-python/`;
2. registrar la traducción Laravel en este directorio;
3. implementar en la capa correspondiente;
4. probar la regla;
5. verificar que la dependencia sigue apuntando hacia dentro.

Si una traducción necesita cambiar una regla funcional o un campo del contrato, se detiene la implementación y se registra primero la decisión documental.
