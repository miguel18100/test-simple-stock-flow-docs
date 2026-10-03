# ADR-005 — Traducción de hexagonal a Onion

## Estado
Aceptado.

## Decisión
El spec original conserva sus decisiones funcionales y contractuales, pero la implementación Laravel adopta cuatro anillos Onion: Domain, Application, Infrastructure y Presentation, más `Bootstrap` como punto de ensamblaje.

## Consecuencia
La dependencia entre capas queda verificable con Deptrac y tests de reflexión. `Bootstrap` es el único lugar autorizado para enlazar interfaces con implementaciones.
