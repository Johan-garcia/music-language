# Music Language

Descripción breve del proyecto (una o dos líneas).
> TODO: Explica en una frase qué es este proyecto.  
Ejemplo: “Music Language es un lenguaje específico de dominio (DSL) escrito en Python para describir, analizar y generar estructuras musicales de forma programática.”

## Tabla de Contenidos
- [Descripción General](#descripción-general)
- [Características](#características)
- [Arquitectura](#arquitectura)
- [Requisitos del Sistema](#requisitos-del-sistema)
- [Instalación (desde cero)](#instalación-desde-cero)
  - [Opción A: Entorno local](#opción-a-entorno-local)
  - [Opción B: Docker](#opción-b-docker)
- [Uso Rápido](#uso-rápido)
- [Lenguaje / DSL (Sintaxis)](#lenguaje--dsl-sintaxis)
- [Ejemplos](#ejemplos)
- [Ejecutar Pruebas](#ejecutar-pruebas)
- [Configuración Avanzada](#configuración-avanzada)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Buenas Prácticas](#buenas-prácticas)
- [Roadmap](#roadmap)
- [Contribuir](#contribuir)
- [Licencia](#licencia)
- [Créditos](#créditos)

## Descripción General
> TODO: Explica el problema que resuelve y para quién está pensado.  
Ejemplo: “Permite a compositores, educadores y desarrolladores representar música como código para análisis, transformación y exportación a formatos estándar.”

## Características
- Definición declarativa de piezas musicales.
- Posible exportación a MIDI / MusicXML. (TODO: Confirmar)
- API en Python para manipulación programática.
- CLI para procesar archivos del lenguaje. (TODO: Confirmar si existe)
- Soporte de escalas, acordes y compases. (TODO)

## Arquitectura
> Resumen breve; el detalle completo en `docs/arquitectura.md`.
- Núcleo: Parser del lenguaje → AST (Árbol Sintáctico).
- Módulo de análisis / transformación.
- Módulo de generación de salida (ej. MIDI). (TODO)
- CLI (entrypoint) en `...` (TODO: ruta)
- Posible integración con Docker para entorno reproducible.

## Requisitos del Sistema
- Python >= 3.10 (TODO: confirmar versión mínima).
- pip / venv.
- (Opcional) Docker y Docker Compose.
- Sistema operativo: Linux, macOS o Windows 10/11.

## Instalación (desde cero)

### Opción A: Entorno local

1. Clonar el repositorio:
   ```bash
   git clone https://github.com/Johan-garcia/music-language.git
   cd music-language
   ```

2. Crear y activar entorno virtual (recomendado):

   Linux / macOS:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

   Windows (PowerShell):
   ```powershell
   python -m venv .venv
   .venv\Scripts\Activate.ps1
   ```

3. Instalar dependencias:
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```
   > TODO: Si usas Poetry o pipenv, reemplazar instrucciones.

4. (Opcional) Instalar en editable:
   ```bash
   pip install -e .
   ```

### Opción B: Docker
```bash
docker build -t music-language:latest .
docker run --rm -it -v $(pwd):/app music-language:latest bash
```
> TODO: Si el Dockerfile ya ejecuta algo por defecto, documentarlo.

## Uso Rápido
Ejemplo mínimo (Python):
```python
from music_language import parse, compile_score   # TODO: nombres reales

code = """
tempo 120
compas 4/4
do4 negra
re4 negra
mi4 blanca
"""
ast = parse(code)
result = compile_score(ast)   # genera objeto musical / archivo
```

Ejemplo CLI (si existe):
```bash
musiclang compile ejemplos/tema1.mus -o salida.mid
```
> TODO: Confirmar nombre del comando.

## Lenguaje / DSL (Sintaxis)
> TODO: Documenta elementos del lenguaje.
- Declaración de tempo: `tempo 120`
- Tonalidad: `tonalidad do menor`
- Notas: `do4 negra`, `re#5 corchea`
- Acordes: `acorde doM7`
- Repeticiones: `repetir 2 { ... }`
- Comentarios: `# Esto es un comentario`

## Ejemplos
Directorio sugerido: `examples/`
- `escala_basica.mus`
- `acordes_progresion.mus`
- `melodia_simple.mus`

Ejemplo:
```text
tempo 90
compas 3/4
sol4 negra
la4 negra
si4 negra
do5 blanca
si4 negra
```

## Ejecutar Pruebas
```bash
pytest -v
```
> TODO: Si usas otra herramienta (unittest, nose, etc.), ajustar.

Cobertura:
```bash
pytest --cov=music_language --cov-report=term-missing
```

## Configuración Avanzada
Archivo de configuración (si existe): `config.toml` / `.env`
Variables posibles:
- OUTPUT_FORMAT=midi|musicxml
- LOG_LEVEL=info|debug

## Estructura del Proyecto
> Sugerencia (ajusta a lo real):
```
music-language/
├── music_language/           # Paquete principal
│   ├── __init__.py
│   ├── parser.py
│   ├── lexer.py
│   ├── ast.py
│   ├── compiler.py
│   ├── midi_exporter.py
│   └── cli.py
├── examples/
├── tests/
├── docs/
│   ├── arquitectura.md
│   ├── dsl.md
│   └── contribucion.md
├── requirements.txt
├── setup.py / pyproject.toml (TODO)
├── Dockerfile
└── README.md
```

## Buenas Prácticas
- Ejecutar `pytest` antes de hacer commit.
- Nombrar archivos `.mus` para el DSL.
- Mantener ejemplos simples y cubriendo casos edge.

## Roadmap
- [ ] Exportación a MusicXML (TODO)
- [ ] Soporte para dinámicas (p, mf, f)
- [ ] Integración con librería de síntesis
- [ ] Visualización de pentagrama
> Edita este listado según tus planes.

## Contribuir
Ver `docs/contribucion.md`.
Pasos rápidos:
1. Haz fork.
2. Crea rama: `git checkout -b feature/nueva-funcionalidad`
3. Commit: `git commit -m "Agrega X"`
4. Push: `git push origin feature/nueva-funcionalidad`
5. Abre un Pull Request.

## Licencia
> TODO: Indica la licencia (MIT recomendada si no has decidido).
Ejemplo:
Este proyecto se distribuye bajo la licencia MIT. Ver [LICENSE](LICENSE).

## Créditos
Autor: @Johan-garcia  
Inspiraciones / Librerías:
- mido (TODO)
- music21 (TODO)

---
¿Falta algo? Abre un issue o mejora este README.