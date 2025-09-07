# Arquitectura Técnica

## Visión General
> TODO: Explica el flujo: Código fuente (.mus) → Lexer → Parser → AST → Transformaciones → Exportador (MIDI / otros).

## Componentes
1. Lexer:
   - Responsabilidad: tokenizar.
   - Biblioteca usada: (TODO) ¿handmade / ply / lark?
2. Parser:
   - Convierte tokens a AST.
3. AST:
   - Nodos: Nota, Acorde, Compás, Repetición, etc. (TODO)
4. Motor Semántico:
   - Validaciones: duración total por compás, rango tonal, etc.
5. Exportadores:
   - MIDI: `midi_exporter.py`
   - MusicXML: (futuro)
6. CLI:
   - Entrada: archivo `.mus`
   - Salida: `.mid`

## Flujo
```mermaid
flowchart LR
  A[Fichero .mus] --> B[Lexer]
  B --> C[Parser]
  C --> D[AST]
  D --> E[Validaciones]
  E --> F[Transformaciones opcionales]
  F --> G[Exportador MIDI]
  G --> H[Archivo .mid]
```

## Decisiones Técnicas
- Python por rapidez de iteración.
- Diseño modular para permitir nuevos exportadores.

## Rendimiento
> TODO: Notas sobre complejidad y tamaño máximo esperado.

## Posibles Mejoras
- Cacheo de AST.
- LSP para editores.