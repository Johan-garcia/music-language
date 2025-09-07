# Music Language / Music Recommendation Backend
Realizado por: Marco Duarte, Elkin Benitez, Alejandro Caro, Johan Garcia.

---

## Table of Contents / Índice

1. Project Summary (EN) / Resumen del Proyecto (ES)  
2. Features / Características  
3. Arquitectura y Módulos Actuales  
4. Diferencias vs. Diseño Original (Gap Analysis)  
5. Estructura Real del Código (Repo actual)  
6. Base de Datos / Esquema  
7. Configuración y Variables de Entorno  
8. Puesta en Marcha (Docker / Local)  
9. Autenticación (JWT)  
10. Integración Spotify  
11. Búsqueda de Letras (Genius)  
12. Recomendaciones por Idioma  
13. Endpoints (Resumen)  
14. Ejemplos de Uso (cURL)   

---

## 1. Project Summary (English)

The Music Recommendation API is a backend application built with FastAPI that integrates Spotify and Genius (YouTube planned) to offer:
- User management with JWT authentication.
- Music metadata, top tracks/artists retrieval.
- Lyrics metadata lookup (via Genius search URL).
- Simple recommendation endpoint based on user top items and preferred language.
- Expandable schemas for analytics and ML-driven personalization.

Planned (partially/not yet implemented in current codebase):
- Admin panel endpoints.
- YouTube search / streaming integration.
- Advanced recommendation engine module layer.
- API usage tracking with dashboards.

### Resumen (Español)

Backend orientado a descubrir música y ofrecer recomendaciones ligeras:
- Gestión de usuarios (roles básicos) y autenticación JWT.
- Integración con Spotify (top tracks, artistas, búsqueda, recomendaciones).
- Endpoint de letras (metadatos + URL de Genius).
- Infraestructura lista para eventos de analítica y cache de modelos.
- Diseño preparado para extender a YouTube y panel administrativo.

---

## 2. Features / Características

| Estado | Feature | Descripción |
|--------|---------|-------------|
| ✅ | User accounts | Tabla `app_user`, JWT, preferencia de idioma. |
| ✅ | Spotify basic integration | Top tracks/artists, track details, search, recommendations. |
| ✅ | Genius metadata search | Devuelve URL y metadatos, no la letra cruda. |
| ✅ | Simple language-based reco | Selección de mercado por idioma + seeds de usuario. |
| ⚠️ Parcial | Analytics | Estructura de tabla creada, endpoints no implementados. |
| ⚠️ Parcial | ML models cache | Tabla creada (`ml_models`), sin lógica avanzada. |
| 🕒 | YouTube integration | Mencionado en README original, aún no en código aquí. |
| 🕒 | Admin panel | Falta router dedicado y endpoints. |
| 🕒 | API usage tracking | Pendiente. |
| 🕒 | Frontend / UI | No incluido en este repo. |

---

## 3. Arquitectura y Módulos Actuales

Componentes clave en el repo real:
- `main.py`: Registro de routers.
- `dependencies.py`: Hash de contraseñas, creación/verificación de JWT, dependencias `require_user` y `require_role`.
- `users.py`: Endpoints para perfil, listado (admin), creación y cambio de idioma.
- `music.py`: Top tracks/artists, búsqueda y detalle de tracks (Spotify).
- `lyrics.py`: Búsqueda de canción y resultados libres en Genius.
- `recommendation.py`: Recomendaciones por idioma.
- `spotify_client.py`: Manejo tokens de Spotify (refresh automático) y helpers de API.
- `settings.py`: Carga de variables mediante `pydantic-settings`.
- `init.sql`: Esquema de base de datos inicial.

Módulos previstos pero no presentes tal como el primer README indicaba (carpetas `services/`, `api/v1/`, etc.) pueden añadirse en una futura reorganización (ver Roadmap).

---

## 4. Diferencias vs. Diseño Original (Gap Analysis)

| Diseño Original (README inicial) | Estado en Repo Actual | Comentario |
|----------------------------------|-----------------------|------------|
| `app/api/v1/auth.py` con OAuth completo | Solo modelos en `auth.py` | Falta flujo completo register/login + OAuth exchange. |
| `admin.py` | No existe | Requiere router y endpoints (stats, user mgmt). |
| Servicios separados (`youtube_service.py`, etc.) | Lógica integrada en archivos planos | Recomendable modularizar en `services/` si crece. |
| Recomendation engine dedicado | Implementación simple en `recommendation.py` | Puede moverse a servicio con algoritmos avanzados. |
| Playlists / songs DB models | No implementado | Solo tablas básicas. |
| Redis caching | No | Se puede añadir para tokens/requests. |
| API usage tracking | No | Reutilizar `analytics_event` o tabla nueva. |
| Makefile / scripts automatizados | No presentes | Se pueden agregar (`make up`, `make test`, etc.). |

---

## 5. Estructura Real del Repositorio

(Ejemplo simplificado basado en los archivos proporcionados)

```
backend/
├── app/
│   ├── main.py
│   ├── settings.py
│   ├── dependencies.py
│   ├── auth.py                 # Modelos de entrada (Register/Login) – faltan endpoints
│   ├── users.py
│   ├── music.py
│   ├── lyrics.py
│   ├── recommendation.py
│   ├── analytics.py            # Solo modelo Event (Pydantic)
│   ├── spotify_client.py
│   ├── db/
│   │   └── postgres.py         # (Asumido, maneja AsyncSession y modelos ORM)
│   ├── schemas/
│   │   └── user.py
│   ├── init.sql
├── requirements.txt
├── docker-compose.yml
├── .env
└── README.md (este)
```

---

## 6. Base de Datos / Esquema

Definida en `init.sql`:

| Tabla | Campos Principales | Uso |
|-------|--------------------|-----|
| `app_user` | id, email, password_hash, role, preferred_lang | Usuarios |
| `spotify_account` | user_id, access_token, refresh_token, expires_at | Tokens Spotify |
| `analytics_event` | user_id, type, payload (JSONB) | Eventos (no expuestos aún) |
| `ml_models` | user_id, recommendations (JSONB) | Cache recomendaciones |
| `track_language` | track_id, lang | Cache heurística idioma |

---

## 7. Configuración y Variables de Entorno

`settings.py` usa `pydantic_settings`.

Principales variables:

| Variable | Ejemplo | Descripción |
|----------|---------|-------------|
| JWT_SECRET | "localdevsecret" | Clave firma JWT |
| JWT_EXPIRE_MIN | 120 | Expiración en minutos |
| PG_URL | postgresql+asyncpg://... | Conexión async a Postgres |
| SPOTIFY_CLIENT_ID / SECRET | ... | Credenciales OAuth |
| GENIUS_API_TOKEN | ... | Token API Genius |
| YOUTUBE_API_KEY | ... | (Planeado) |

No versionar estos valores reales. Usa `.env` local.

---

## 8. Puesta en Marcha

### A) Docker (recomendado)

```
docker compose up --build
```

Servicios:
- Backend: http://localhost:8000
- Swagger UI: http://localhost:8000/docs
- DB Postgres: puerto 5432

Si necesitas cargar el esquema manualmente:

```
docker exec -it music_pg psql -U postgres -d music -f /ruta/a/init.sql
```

(Considera montar `init.sql` como volumen o usar migraciones Alembic a futuro).

### B) Local sin Docker

1. Python 3.11+
2. Instalar dependencias:
   ```
   pip install -r requirements.txt
   ```
3. Exportar/editar `.env`.
4. Crear DB y ejecutar `init.sql`.
5. Ejecutar:
   ```
   uvicorn app.main:app --reload --port 8000
   ```

---

## 9. Autenticación (JWT)

Funciones clave (`dependencies.py`):
- `hash_password`
- `verify_password`
- `create_jwt`
- `require_user` (FastAPI dependency)
- `require_role("admin")`

Faltan endpoints explícitos para:
- `POST /auth/register`
- `POST /auth/login`

Workaround temporal:
1. Insertar usuario manualmente (hasheando password en una shell Python).
2. Generar token con `create_jwt(sub=<user_id>)`.
3. Usar `Authorization: Bearer <TOKEN>`.

---

## 10. Integración Spotify

Archivo: `spotify_client.py`

Funciones:
- `ensure_access_token(session, user_id)` refresca token automáticamente.
- `me_top`, `api_get`, `recommendations`.

Flujo OAuth completo no mostrado; se asume que el cliente obtiene `refresh_token` y se guarda mediante `save_spotify_tokens`.  
Si no existe fila en `spotify_account` → error.

---

## 11. Búsqueda de Letras (Genius)

Endpoints:
- `GET /lyrics?artist=...&title=...`
- `GET /lyrics/search?q=...`

Devuelve:
- `genius_url` (el frontend debe abrir esa URL)
- `thumbnail`, `full_title`, artista, etc.

Necesita `GENIUS_API_TOKEN`.

---

## 12. Recomendaciones por Idioma

`POST /reco/by-language`:
- Entrada: `{ "lang": "es", "limit": 30 }`
- Heurística: idioma → primer mercado listado en `LANG_TO_MARKETS`.
- Usa top tracks/artists (max 3 seeds cada uno) y solicita `/recommendations` a Spotify.

Limitaciones:
- Si no hay historial de usuario → 400.
- No filtra por idioma real de las letras (sólo proxies del market).

---

## 13. Endpoints (Resumen)

| Método | Ruta | Descripción | Auth |
|--------|------|-------------|------|
| GET | /health | Salud general | No |
| GET | /users/health | Salud módulo usuarios | No |
| GET | /users/me | Perfil usuario actual | Sí |
| GET | /users | Lista usuarios (admin) | Sí (admin) |
| POST | /users | Crear usuario (pruebas) | No (password ya hasheado) |
| PATCH | /users/me/lang?lang=xx | Cambiar idioma preferido | Sí |
| GET | /music/health | Salud módulo música | No |
| GET | /music/me/top/tracks | Top tracks | Sí |
| GET | /music/me/top/artists | Top artistas | Sí |
| GET | /music/search?q=... | Buscar tracks | Sí |
| GET | /music/track/{id} | Detalle de track | Sí |
| GET | /lyrics/health | Salud módulo lyrics | No |
| GET | /lyrics?artist=&title= | Metadatos de canción | Sí |
| GET | /lyrics/search?q= | Búsqueda libre | Sí |
| POST | /reco/by-language | Recomendaciones por idioma | Sí |

---

## 14. Ejemplos (cURL)

Generar token (shell Python):
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

Lyrics:
```bash
curl -H "Authorization: Bearer <TOKEN>" "http://localhost:8000/lyrics?artist=Radiohead&title=Creep"
```

Recomendaciones:
```bash
curl -X POST -H "Authorization: Bearer <TOKEN>" -H "Content-Type: application/json" \
  -d '{"lang":"es","limit":15}' \
  http://localhost:8000/reco/by-language
```



Autor / Mantenimiento: `@Johan-garcia`  
Framework: [FastAPI](https://fastapi.tiangolo.com/)  
APIs: Spotify Web API, Genius API (YouTube planificado)  

¿Comentarios o mejoras? Abre un issue o PR. ¡Gracias! 🎧
