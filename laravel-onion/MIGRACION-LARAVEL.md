# Guía de migración a Laravel + Onion

## Alcance

La migración parte del contrato y modelo existentes en `spec-python/`. No convierte el sistema en un CRUD Laravel convencional. La estructura Onion es obligatoria y las decisiones funcionales del spec permanecen intactas.

## Mapeo por capas

### Domain

Crear:

- `Product`, `Sale`, `SaleItem`, `Category`, `User`.
- `Money`, `Quantity`, `ProductId`, `SaleId`, `CategoryId`, `UserId`, `Username`, `Role`.
- Las excepciones de negocio descritas en `ARQUITECTURA-ONION.md`.

Prohibido:

- `Illuminate\\*`.
- Eloquent.
- `DB`, `config()`, `now()` y facades.
- `version`, `deleted_at`, `total`, `subtotal` como estado del dominio.

### Application

Crear los cinco puertos inbound y los diez outbound definidos en la arquitectura. Los casos de uso son exactamente cinco: `PlaceSaleService`, `ProductCatalogService`, `GetSalesService`, `SalesReportService`, `AuthenticationService`.

`UnitOfWork` mantiene exactamente esta forma:

```php
interface UnitOfWork
{
    public function run(callable $operation): mixed;
}
```

### Infrastructure

Eloquent solo aquí. Los mappers traducen entre modelos Eloquent y objetos del dominio. La implementación de `UnitOfWork` es el único lugar donde aparece `DB::transaction()`.

La concurrencia usa `product.version` en persistencia y `UPDATE ... WHERE id = ? AND version = ?`. Cero filas afectadas se convierte en `ConcurrencyConflict`.

### Presentation

Los controladores solo reciben puertos inbound. Las FormRequests validan presencia, tipo y formato. Las reglas de negocio no se trasladan a `gt`, `ge`, `min`, etc., porque eso cambiaría respuestas de 422 a 400.

Los errores se renderizan mediante los tres renderers definidos en la arquitectura.

## Contrato HTTP que no debe cambiar

- `POST /api/auth/login`
- `POST /api/auth/register`
- `GET /api/products`
- `GET /api/products/{id}`
- `POST /api/products`
- `PUT /api/products/{id}`
- `DELETE /api/products/{id}`
- `POST /api/products/{id}/image`
- `GET /api/categories`
- `POST /api/sales`
- `GET /api/sales`
- `GET /api/sales/{id}`
- `GET /api/reports/sales?from=&to=`
- `GET /health`
- `GET /media/{key}`

El cable usa camelCase. Los campos nulos declarados deben viajar como `null`. Los errores 401/403/404/405 son cuerpos vacíos. Las reglas de negocio siguen saliendo por 422.

## Checklist antes de declarar migrado

- [ ] `grep -R "Illuminate\\\\" app/Domain` devuelve 0.
- [ ] `grep -R "DB::" app/Application` devuelve 0.
- [ ] `grep -R "Infrastructure" app/Presentation` devuelve 0.
- [ ] Solo `app/Bootstrap` instancia implementaciones de Infrastructure.
- [ ] Los tests de Application funcionan sin boot de Laravel.
- [ ] Las migraciones crean las cinco tablas en singular.
- [ ] Existen los CHECK exigidos por `data-model.md`.
- [ ] Existen índices y FKs del modelo físico.
- [ ] `seed_categories` usa UUID fijos.
- [ ] No se guarda `total` ni `subtotal`.
- [ ] El administrador inicial se crea mediante el puerto de hash.
- [ ] El reporte agrega en SQL y usa los valores congelados de `sale_item`.
- [ ] `verify.sh` ejecuta dentro del contenedor.
