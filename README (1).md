# music-language (Backend)

Plataforma backend (FastAPI) orientada a:
- Gestión de usuarios.
- Integración con Spotify (top tracks, artistas, búsqueda, recomendaciones).
- Búsqueda de letras vía Genius.
- Recomendaciones musicales simples por idioma.
- (Base preparada para analítica y futuros modelos ML).

> NOTA IMPORTANTE: El archivo `.env` incluido en el repo contiene claves y tokens que **no** deberían versionarse en un entorno real (Spotify Client Secret, Genius Token, etc.). Genera tus propias credenciales y **no** subas las reales a GitHub.

---

## Índice

1. Visión general
2. Arquitectura y módulos
3. Tecnologías principales
4. Estructura del proyecto
5. Esquema de base de datos
6. Variables de entorno
7. Puesta en marcha (Docker y modo local)
8. Flujo de autenticación (JWT)
9. Integración con Spotify
10. Búsqueda de letras (Genius)
11. Recomendaciones por idioma
12. Endpoints (Resumen con ejemplos)
13. Ejecución de pruebas manuales (cURL / HTTP)
14. Seguridad y buenas prácticas
15. Errores comunes y solución
16. Roadmap / Mejoras sugeridas

---

## 1. Visión general

Este backend expone una API REST construida con FastAPI para ofrecer datos musicales personalizados:
- Permite autenticar usuarios y asociar su cuenta de Spotify (persistiendo tokens).
- Ofrece endpoints para obtener top tracks/artists, buscar canciones y obtener recomendaciones basadas en el idioma preferido.
- Expone endpoints para buscar metadatos de canciones en Genius y devolver la URL donde ver la letra.
- Prepara tablas para registrar eventos de analítica y cachear resultados de modelos/recomendaciones.

---

## 2. Arquitectura y módulos

Componentes principales:

| Componente | Descripción |
|------------|-------------|
| FastAPI    | Framework principal HTTP. |
| PostgreSQL | Persistencia de usuarios, tokens Spotify y eventos. |
| Spotify API | Fuente de datos musicales reales (perfil y recomendaciones). |
| Genius API | Fuente de metadatos / URL de letras. |
| JWT        | Autenticación stateless vía tokens firmados. |
| Passlib (bcrypt) | Hash de contraseñas. |

Flujo típico:
1. Usuario se registra / inicia sesión → recibe JWT.
2. Conecta su cuenta Spotify (faltaría endpoint OAuth en este repo; la base ya está preparada).
3. Cliente llama a `/music/...` pasando `Authorization: Bearer <token>`.
4. Backend renueva access token de Spotify si expiró y consulta API.
5. Respuesta JSON al frontend.

---

## 3. Tecnologías principales

- Python 3.11+ (recomendado)
- FastAPI
- SQLAlchemy (async)
- asyncpg
- httpx (peticiones async a APIs externas)
- Passlib (hash de contraseñas)
- Docker / docker-compose
- PostgreSQL 15

---

## 4. Estructura del proyecto

(Asumiendo que este backend está bajo `./backend/app` o similar)

```
.
├── app/
│   ├── main.py               (Punto de entrada FastAPI)
│   ├── settings.py           (Gestión de variables de entorno)
│   ├── dependencies.py       (Helpers auth: hash, verify, create_jwt, require_user)
│   ├── auth.py               (Modelos de entrada para login/register)
│   ├── users.py              (Rutas de usuarios)
│   ├── music.py              (Rutas integradas a Spotify)
│   ├── lyrics.py             (Búsqueda de letras via Genius)
│   ├── recommendation.py     (Recomendaciones por idioma)
│   ├── analytics.py          (Modelo pydantic para eventos)
│   ├── spotify_client.py     (Función de acceso y refresh tokens Spotify)
│   ├── db/
│   │   ├── postgres.py       (NO incluido en snippet, se asume ORM/base)
│   ├── schemas/
│   │   └── user.py           (Modelos pydantic UserCreate / UserOut)
│   ├── init.sql              (Esquema SQL inicial)
├── requirements.txt
├── docker-compose.yml
├── .env
└── README.md (este documento)
```

---

## 5. Esquema de base de datos

Definido en `init.sql`:

| Tabla | Campos clave | Propósito |
|-------|--------------|-----------|
| app_user | id, email, password_hash, role, preferred_lang | Usuarios de la app. |
| spotify_account | user_id (PK), access/refresh tokens, expiración | Vinculación Spotify. |
| analytics_event | id, user_id, type, payload (JSONB) | Registro de eventos comportamiento. |
| ml_models | user_id, recommendations JSONB | Cache de resultados de modelos. |
| track_language | track_id, lang | Cache heurística de idioma por track. |

Type adicional:
- ENUM `user_role` = ('user','admin').

---

## 6. Variables de entorno

Definidas en `settings.py` y provistas vía `.env`:

| Variable | Descripción | Obligatoria |
|----------|-------------|-------------|
| APP_NAME | Nombre app | No (default) |
| APP_ENV | dev / prod | No |
| APP_PORT | Puerto | No |
| APP_HOST | Host bind | No |
| JWT_SECRET | Clave firma JWT | Sí |
| JWT_ALG | Algoritmo (HS256) | Sí |
| JWT_EXPIRE_MIN | Minutos expiración | Sí |
| PG_URL | Cadena conexión asyncpg | Sí |
| SPOTIFY_CLIENT_ID | OAuth Spotify | Requerido para funciones Spotify |
| SPOTIFY_CLIENT_SECRET | OAuth Spotify | Requerido |
| GENIUS_API_TOKEN | Token Genius | Para /lyrics |
| YOUTUBE_API_KEY | (Reservado / futuro) | No |

Buen práctica: usar `.env.local` y no subirlo.

---

## 7. Puesta en marcha

### Opción A: Docker Compose (recomendada)

Requisitos: Docker + docker-compose.

1. Clona el repo.
2. Crea (o edita) `backend/.env` con tus propios valores (no uses los de ejemplo en producción).
3. Lanza servicios:
   ```
   docker compose up --build
   ```
4. FastAPI quedará en: `http://localhost:8000`
5. Documentación interactiva (Swagger): `http://localhost:8000/docs`
6. Verifica salud:
   ```
   curl http://localhost:8000/health
   ```

La base de datos se crea vacía; ejecuta `init.sql` si tu módulo `postgres.py` no lo hace automáticamente. Si no existe bootstrap, puedes:
```
docker exec -it music_pg psql -U postgres -d music -f /ruta/en/contenedor/init.sql
```
(Alternativamente monta el `init.sql` como volumen de inicialización).

### Opción B: Local sin Docker

Requisitos: Python 3.11+, PostgreSQL en marcha.

1. Crea y activa virtualenv:
   ```
   python -m venv .venv
   source .venv/bin/activate
   ```
2. Instala dependencias:
   ```
   pip install -r requirements.txt
   ```
3. Ajusta `.env` (PG_URL a tu instancia local, e.g. `postgresql+asyncpg://user:pass@localhost:5432/music`)
4. Crea la base y ejecuta `init.sql`.
5. Ejecuta:
   ```
   uvicorn app.main:app --reload --port 8000
   ```

---

## 8. Flujo de autenticación (JWT)

El archivo `dependencies.py` define:
- `hash_password(raw)`
- `verify_password(raw, hashed)`
- `create_jwt(sub, role, scope)`
- `require_user` (lee encabezado `Authorization: Bearer <token>`)

NOTA: El repo no incluye todavía endpoints explícitos `/auth/register` y `/auth/login` en el snippet mostrado (solo modelos en `auth.py`). Para una POC puedes:
1. Insertar un usuario manualmente en `app_user` con un `password_hash` generado:
   ```python
   from app.dependencies import hash_password
   print(hash_password("miclave"))
   ```
2. Generar un token:
   ```python
   from app.dependencies import create_jwt
   token = create_jwt(sub="<id-del-usuario>", role="user")
   print(token)
   ```
3. Usarlo en peticiones:
   ```
   curl -H "Authorization: Bearer <TOKEN>" http://localhost:8000/users/me
   ```

### Recomendación
Implementar (futuro):
- POST `/auth/register` → crea usuario (hash + email único).
- POST `/auth/login` → valida email/password y devuelve JWT.
- (Opcional) refresh tokens propios para rotar JWT.

---

## 9. Integración con Spotify

Se guardan tokens OAuth en `spotify_account`. El flujo OAuth completo no está totalmente implementado aquí, pero el backend:
- Refresca tokens vencidos (`ensure_access_token`).
- Llama endpoints `/me/top/*`, `/search`, `/tracks/{id}`, `/recommendations`.

Necesitas:
- Crear app en https://developer.spotify.com/dashboard
- Configurar redirect URI en la app (ej. `http://localhost:3000/callback`).
- Implementar (frontend + endpoint) el intercambio `authorization_code` → `access_token` y luego llamar a `save_spotify_tokens`.

En producción: cifrar/ocultar `refresh_token` (almacenamiento seguro).

---

## 10. Búsqueda de letras (Genius)

Endpoint `/lyrics`:
- `/lyrics?artist=...&title=...` → devuelve metadatos + URL
- `/lyrics/search?q=...` → búsqueda libre
- No provee letra completa (limitaciones de licencias). Devuelve `genius_url` para que el cliente abra la página.

Requiere `GENIUS_API_TOKEN`.

---

## 11. Recomendaciones por idioma

Endpoint: `POST /reco/by-language`
- Toma idioma (`lang`) y deduce un mercado Spotify (`market`) usando `LANG_TO_MARKETS`.
- Usa top tracks / artists del usuario como semillas (`seed_tracks` / `seed_artists`).
- Devuelve lista simplificada de canciones recomendadas.

Limitaciones:
- Si el usuario no tiene suficiente historial en Spotify → 400.
- Heurística básica (solo primer market por idioma).
- Futuro: filtrar por detección de idioma real de la letra o metadata.

---

## 12. Endpoints (Resumen)

(Asumiendo todos requieren JWT salvo `/health` y los de lyrics health)

| Método | Ruta | Descripción | Auth |
|--------|------|-------------|------|
| GET | /health | Salud general | No |
| GET | /users/health | Salud módulo usuarios | No |
| GET | /users/me | Perfil actual | Sí |
| GET | /users | Lista usuarios (admin) | Sí (role=admin) |
| POST | /users | Crear usuario (pruebas) | No (pero password ya debe venir hasheado) |
| PATCH | /users/me/lang?lang=es | Actualiza idioma preferido | Sí |
| GET | /music/health | Salud módulo música | No |
| GET | /music/me/top/tracks | Top tracks usuario | Sí |
| GET | /music/me/top/artists | Top artistas usuario | Sí |
| GET | /music/search?q=... | Búsqueda Spotify | Sí |
| GET | /music/track/{id} | Track individual | Sí |
| GET | /lyrics/health | Salud módulo letras | No |
| GET | /lyrics?artist=...&title=... | Metadatos canción + URL Genius | Sí |
| GET | /lyrics/search?q=... | Búsqueda libre Genius | Sí |
| POST | /reco/by-language | Recomendaciones por idioma | Sí |

---

## 13. Ejemplos (cURL)

Obtener token manual (ejemplo conceptual):
```bash
python -c "from app.dependencies import create_jwt; print(create_jwt('USER-ID-123'))"
```

Perfil:
```bash
curl -H "Authorization: Bearer <TOKEN>" http://localhost:8000/users/me
```

Top tracks:
```bash
curl -H "Authorization: Bearer <TOKEN>" "http://localhost:8000/music/me/top/tracks?limit=5"
```

Letra (metadatos):
```bash
curl -H "Authorization: Bearer <TOKEN>" "http://localhost:8000/lyrics?artist=Radiohead&title=Creep"
```

Recomendaciones:
```bash
curl -X POST -H "Authorization: Bearer <TOKEN>" -H "Content-Type: application/json" \
  -d '{"lang":"es","limit":15}' \
  http://localhost:8000/reco/by-language
```

