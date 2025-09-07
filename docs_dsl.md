# Definición del DSL (Music Language)

## Objetivo
Sintaxis legible para representar música de forma textual.

## Elementos Básicos
- Tempo: `tempo 120`
- Compás: `compas 4/4`
- Nota: `<nota><octava> <figura>`
  - Ej: `do4 negra`
- Figuras: redonda, blanca, negra, corchea, semicorchea, fusa (TODO)
- Alteraciones: `#` sostenido, `b` bemol.

## Acordes
`acorde doM7`
> TODO: lista soportada.

## Repeticiones
```
repetir 2 {
  do4 negra
  re4 negra
}
```

## Comentarios
`# Esto es un comentario`

## Errores Comunes
- Nota inválida: `h4`
- Figura desconocida: `doblecorchea` (si no existe)
- Desfase de duración en compás.

## Extensiones Futuras
- Dinámicas: `p`, `mf`, `f`
- Articulaciones: staccato, legato