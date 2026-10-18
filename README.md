# Delta - Gestión de Configuración

Plataforma web para la gestión de riesgos y configuración empresarial. El
repositorio contiene una aplicación full-stack compuesta por un frontend web,
una API backend y una base de datos PostgreSQL.

## Contenido del repositorio

| Componente | Tecnología | Versión |
| --- | --- | --- |
| Frontend | React | 19.2.6 |
| Frontend | Vite | 8.0.12 |
| Frontend | Tailwind CSS | 4.3.0 |
| Frontend | React Router | 7.15.1 |
| Backend | Node.js | 22.x |
| Backend | Express | 5.1.0 |
| Backend | Vitest | 5.0.5 |
| Base de datos | PostgreSQL | 16 (imagen `postgres:16-alpine`) |
| Servidor de archivos estáticos | Nginx | `alpine` |
| Pruebas E2E | Playwright | 1.64.1 |

El frontend y el backend se mantienen como submódulos Git:

- `Delta-Frontend`: aplicación React/Vite.
- `Delta-Backend`: API Express.js con autenticación JWT y conexión a
  PostgreSQL.
- `database/init.sql`: script inicial para crear las tablas de la base de
  datos.
- `compose.yml`: configuración para ejecutar todos los servicios con Docker.

## Requisitos

### Para ejecutar con Docker

- Git.
- Docker Desktop con Docker Compose v2.
- Una variable `JWT_SECRET` definida en el archivo `.env` del directorio raíz.

### Para desarrollo local

- Node.js 22.x y npm.
- Docker Desktop, para ejecutar PostgreSQL localmente.
- Chromium instalado por Playwright si se ejecutan las pruebas E2E.

## Descargar el proyecto

Clonar el repositorio junto con sus submódulos:

```bash
git clone --recurse-submodules https://github.com/ERICKJLLO/Delta-Gestion-Configuracion.git
cd Delta-Gestion-Configuracion
```

Si el repositorio ya se clonó sin submódulos:

```bash
git submodule update --init --recursive
```

## Opción 1: ejecución completa con Docker Compose

1. Crear el archivo `.env` en la raíz del proyecto:

   ```env
   JWT_SECRET=un_secreto_largo_y_aleatorio
   ```

   No se debe publicar este archivo ni compartir su valor.

2. Descargar las imágenes y levantar los servicios:

   ```bash
   docker compose pull
   docker compose up -d
   ```

   Docker Compose inicia:

   - PostgreSQL en `localhost:5433`.
   - Backend en `http://localhost:3001`.
   - Frontend en `http://localhost:8080`.

3. Abrir la aplicación en <http://localhost:8080>.

   La imagen del frontend usa Nginx y redirige las solicitudes `/api/` al
   contenedor del backend. La base de datos se inicializa automáticamente con
   [`database/init.sql`](./database/init.sql) en el primer arranque del volumen.

4. Consultar el estado o detener los servicios:

   ```bash
   docker compose ps
   docker compose logs -f
   docker compose down
   ```

   `docker compose down` conserva los datos de PostgreSQL. Para eliminar
   también el volumen local de la base de datos se debe ejecutar explícitamente:

   ```bash
   docker compose down -v
   ```

## Opción 2: desarrollo local

### 1. Iniciar PostgreSQL

Desde la raíz del repositorio, iniciar únicamente la base de datos:

```bash
docker compose up -d db
```

La base de datos queda disponible con estos valores:

```text
Host: localhost
Puerto: 5433
Base de datos: delta_db
Usuario: delta_user
Contraseña: delta_local_dev_password
```

### 2. Configurar y ejecutar el backend

En una terminal:

```bash
cd Delta-Backend
copy .env.example .env
```

En macOS/Linux, el comando equivalente para copiar el archivo es:

```bash
cp .env.example .env
```

Revisar `.env` y establecer un valor privado para `JWT_SECRET`. Para la
configuración local, `DATABASE_URL` debe apuntar a PostgreSQL en
`localhost:5433`. Después instalar dependencias e iniciar la API:

```bash
npm ci
npm run dev
```

La API estará disponible en `http://localhost:3001`.

### 3. Configurar y ejecutar el frontend

En otra terminal:

```bash
cd Delta-Frontend
npm ci
npm run dev
```

Vite iniciará el frontend en `http://localhost:5173`. Por defecto, el cliente
usa `http://localhost:3001/api` como URL de la API. Si se necesita cambiarla,
crear `Delta-Frontend/.env` con:

```env
VITE_API_URL=http://localhost:3001/api
```

## API y documentación

- Comprobación de salud: <http://localhost:3001/api/health>
- Swagger UI: <http://localhost:3001/api/docs>
- Especificación OpenAPI JSON: <http://localhost:3001/api/docs.json>

Las rutas principales incluyen autenticación (`/api/auth`) y funcionalidad de
IA (`/api/ai`).

## Pruebas y validaciones

Backend:

```bash
cd Delta-Backend
npm test
```

Frontend:

```bash
cd Delta-Frontend
npm run lint
npm run build
```

Pruebas E2E del frontend:

```bash
cd Delta-Frontend
npx playwright install chromium
npm run test:e2e
```

Las pruebas E2E levantan automáticamente el servidor de desarrollo de Vite en
`http://127.0.0.1:5173`. Para que las pruebas que dependen de la API funcionen,
el backend y PostgreSQL deben estar ejecutándose.

## Construir las imágenes Docker localmente

Si se desea construir las imágenes en lugar de descargarlas de Docker Hub:

```bash
docker build -t delta-backend:local ./Delta-Backend
docker build --build-arg VITE_API_URL=/api -t delta-frontend:local ./Delta-Frontend
```

La configuración estándar de `compose.yml` utiliza las imágenes publicadas
`erickjllo/delta-backend:latest` y `erickjllo/delta-frontend:latest`. Para usar
las imágenes locales, hay que cambiar temporalmente los campos `image` del
archivo Compose por `delta-backend:local` y `delta-frontend:local`.

## Notas de seguridad

- No subir `.env` ni exponer `JWT_SECRET`.
- Cambiar las credenciales y secretos predeterminados antes de usar el sistema
  fuera de un entorno local.
- Las credenciales de PostgreSQL incluidas en Compose son únicamente para
  desarrollo local.
