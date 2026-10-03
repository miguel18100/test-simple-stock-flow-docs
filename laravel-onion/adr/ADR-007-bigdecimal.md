# ADR-007 — BigDecimal para Money

## Estado
Aceptado.

## Decisión
PHP no aporta un tipo decimal de dominio adecuado. `Money` utiliza `Brick\\Math\\BigDecimal`, escala 2 y redondeo `HALF_UP`.

## Consecuencia
No se utiliza `float` para importes. El mapper rechaza valores con más de dos decimales cuando el contrato/modelo así lo exige; el comportamiento exacto del endpoint se mantiene según `api-contract.md`.
