# Contrato DTO congelado — Laravel

Este archivo fija la traducción del contrato HTTP a Laravel. El contrato original sigue en `spec-python/api-contract.md`.

## Convenciones

- JSON: `camelCase`.
- Identificadores: UUID como string.
- Money: JSON `number`, precisión de dos decimales; internamente `BigDecimal`.
- Fechas: ISO 8601 con desplazamiento explícito UTC (`+00:00`).
- Campos nulos declarados: se serializan como `null`.
- `currency`: siempre `COP` en vistas monetarias y reporte.
- `total` y `subtotal`: calculados, nunca persistidos.
- `totalPages`: debe aparecer en todo `PagedResult` serializado.

## Request DTO / Command

### Login

`username: string`, `password: string`.

### Register

`username: string`, `password: string`, `role: string`.

El puerto solo crea vendedores. `admin` se rechaza por la decisión DP-04.

### Product create/update

`name: string`, `price: number`, `stock: integer`, `categoryId: uuid`.

La imagen se maneja por endpoint separado.

### Place sale

El cuerpo contiene las líneas de la venta. El vendedor se obtiene del token; no se acepta como campo del cliente.

## Response DTO / View

### ProductView

```text
id: string
name: string
price: number
currency: string
stock: integer
categoryId: string
categoryName: string
imageUrl: string|null
```

### SaleItemView

```text
productId: string
productName: string
quantity: integer
unitPrice: number
subtotal: number
```

### SaleView

```text
id: string
soldAt: string
soldBy: string
total: number
currency: string
items: SaleItemView[]
```

### SalesReportRow

```text
productId: string
productName: string
categoryName: string
unitsSold: integer
revenue: number
```

### SalesReport

```text
from: string
to: string
salesCount: integer
grandTotal: number
currency: string
rows: SalesReportRow[]
```

### AuthResult

```text
accessToken: string
expiresAt: string
username: string
role: string
```

### PagedResult<T>

```text
items: T[]
page: integer
size: integer
totalItems: integer
totalPages: integer
```

## Errores

### Problem Details

Para 422/409/500: `application/problem+json` con `detail` en español.

### Validation Error

Para 400:

```json
{
  "title": "...",
  "status": 400,
  "detail": "...",
  "errors": {
    "campo": ["..." ]
  }
}
```

### Empty Error

Para 401/403/404/405: cuerpo vacío (`Content-Length: 0`), conservando `WWW-Authenticate` y `Allow` cuando correspondan.
