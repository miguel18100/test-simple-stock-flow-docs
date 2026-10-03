# ADR-006 — Puertos en Application

## Estado
Aceptado.

## Decisión
Los puertos inbound y outbound viven en `app/Application/Ports/` y no en Domain.

## Consecuencia
Application expresa directamente sus necesidades externas. Domain permanece completamente aislado de repositorios, persistencia, seguridad y framework.
