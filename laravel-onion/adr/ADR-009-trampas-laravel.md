# ADR-009 — Trampas de Laravel

## Estado
Aceptado.

## Decisión
Se prohíbe que facades, Eloquent, auto-wiring o helpers de Laravel atraviesen hacia Domain/Application.

## Consecuencia
Los controladores reciben solo puertos inbound. Eloquent queda detrás de repositorios y mappers. Las transacciones quedan detrás de `UnitOfWork`.
