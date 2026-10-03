# Guía Onion — Simple Stock Flow

## Corrección documental

La documentación original del spec describe una arquitectura **hexagonal**. Este proyecto no afirma que el spec original fuera Onion. Esta edición traduce esas decisiones a una arquitectura **Onion de cuatro anillos + Bootstrap** para Laravel y React.

## Anillos

1. **Domain** — negocio puro.
2. **Application** — casos de uso y puertos.
3. **Infrastructure** — implementaciones técnicas.
4. **Presentation** — HTTP/UI.
5. **Bootstrap** — composición, no anillo.

## Dependencias

```text
Presentation ─────► Application ─────► Domain
Infrastructure ───► Application ─────► Domain
Bootstrap ────────► Infrastructure + Application
```

Presentation nunca importa Infrastructure. Domain no importa nada del proyecto ni de terceros.
