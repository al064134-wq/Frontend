# AgendaUAC Frontend

Frontend web para el sistema de reservacion de espacios de la Facultad de Ingenieria. Esta aplicacion esta construida con Vue 3, Vite, Pinia, PrimeVue, Tailwind CSS y FullCalendar.

## Requisitos

- Node.js 20 o superior.
- npm.
- Backend AgendaUAC ejecutandose en `http://localhost:8080`.
- Cliente OAuth de Google para el boton de inicio de sesion con Google.

## Estructura principal

```text
Frontend/
├── index.html
├── package.json
├── vite.config.js
├── tailwind.config.js
├── src/
│   ├── main.js
│   ├── App.vue
│   ├── router/
│   ├── stores/
│   ├── utils/
│   ├── views/
│   ├── components/
│   └── assets/
└── .env
```

## Variables de entorno

Debes crear un archivo `.env` en la raiz de `Frontend`.

Ejemplo:

```env
VITE_API_BASE=http://localhost:8080
VITE_CLIENT_ID=tu_google_oauth_client_id.apps.googleusercontent.com
```

Variables:

- `VITE_API_BASE`: URL base del backend .NET.
- `VITE_CLIENT_ID`: Client ID del OAuth Client de Google.

Importante: despues de cambiar el `.env`, detiene y vuelve a iniciar Vite. Las variables `VITE_*` se leen al arrancar el servidor.

## Instalacion

Desde la carpeta `Frontend`:

```bash
npm install
```

Si quieres instalar exactamente las versiones del `package-lock.json`:

```bash
npm ci
```

## Ejecucion en desarrollo

1. Inicia el backend:

```bash
cd ../Backend
dotnet run --project AgendaUAC.csproj
```

2. En otra terminal, inicia el frontend:

```bash
cd ../Frontend
npm run dev
```

3. Abre la URL que imprime Vite. Normalmente:

```text
http://localhost:3000
```

## Compilar para produccion

```bash
npm run build
```

La salida se genera en:

```text
dist/
```

Para previsualizar el build:

```bash
npm run preview
```

## Scripts disponibles

```json
{
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview",
  "server": "json-server --watch src/db.json --port 5000"
}
```

Notas:

- `npm run dev`: inicia el servidor de desarrollo.
- `npm run build`: genera archivos de produccion.
- `npm run preview`: sirve localmente el build de `dist`.
- `npm run server`: script heredado para `json-server`; el flujo actual usa el backend .NET.

## Configuracion de Google OAuth

El login con Google usa `vue3-google-login`, que a su vez llama a Google Identity Services para obtener un access token. El frontend envia ese token al backend en:

```text
POST /auth/login/google
```

El backend valida el token con Google y permite el acceso solo si el correo ya existe en la base de datos.

### Crear el proyecto en Google Cloud

1. Entra a Google Cloud Console:

```text
https://console.cloud.google.com/
```

2. Crea un proyecto nuevo o usa uno existente.

3. Abre la seccion de Google Auth Platform / OAuth.

4. Configura la pantalla de consentimiento:

- App name: `AgendaUAC` o el nombre de tu proyecto.
- User support email: tu correo.
- Developer contact information: tu correo.
- Tipo de usuarios:
  - `External` si cualquier cuenta Google autorizada puede intentar autenticarse.
  - `Internal` solo si tienes Google Workspace y quieres restringirlo a tu organizacion.

5. Si la app esta en modo testing, agrega correos en la lista de test users.

### Crear el OAuth Client ID

1. Ve a:

```text
APIs & Services > Credentials
```

2. Selecciona:

```text
Create Credentials > OAuth client ID
```

3. En Application type selecciona:

```text
Web application
```

4. Agrega estos origenes en Authorized JavaScript origins:

```text
http://localhost:3000
http://127.0.0.1:3000
```

Si Vite inicia en otro puerto, por ejemplo `5173`, agrega tambien:

```text
http://localhost:5173
http://127.0.0.1:5173
```

5. No es necesario agregar Authorized redirect URIs para este flujo, porque la libreria usa un popup/token flow desde el navegador.

6. Crea el cliente y copia el Client ID.

7. Pega el Client ID en el `.env`:

```env
VITE_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.apps.googleusercontent.com
```

Referencia oficial de Google:

- https://developers.google.com/identity/gsi/web/guides/get-google-api-clientid
- https://developers.google.com/identity/oauth2/web/guides/load-3p-authorization-library

## Relacion con el backend

El frontend consume la API usando `src/utils/api.js`.

La variable:

```env
VITE_API_BASE=http://localhost:8080
```

hace que las peticiones se construyan asi:

```text
http://localhost:8080/auth/login
http://localhost:8080/spaces
http://localhost:8080/bookings
```

El token JWT se guarda en `localStorage` y se envia en:

```http
Authorization: Bearer <token>
```

## Rutas principales

Publicas:

- `/login`: inicio de sesion.
- `/account-request`: solicitud de cuenta.
- `/register/invitation/:token`: registro por invitacion.

Administrador:

- `/admin/bookings`: reservaciones por dia.
- `/admin/spaces`: catalogo de espacios.
- `/admin/history?page=1`: historial global.
- `/admin/register`: registro directo de usuario.
- `/admin/invitations`: generacion de link de invitacion.
- `/admin/account-requests`: revision de solicitudes de cuenta.

Usuario:

- `/user/bookings`: reservaciones pendientes.
- `/user/book`: crear reservacion.
- `/user/archive?page=1`: historial del usuario.

## Flujo de autenticacion

### Login con correo y contrasena

1. El usuario escribe correo y contrasena.
2. El frontend llama a `POST /auth/login`.
3. Si el backend responde con JWT, se guarda en `localStorage`.
4. El router redirige segun el rol:
   - `admin` a `/admin/bookings`.
   - `user` a `/user/bookings`.

### Login con Google

1. El usuario presiona el boton de Google.
2. Google devuelve un access token.
3. El frontend llama a `POST /auth/login/google`.
4. El backend consulta Google y extrae el correo.
5. Si ese correo ya existe en `users`, devuelve JWT.

## Flujos nuevos

### Registro por invitacion

1. El admin entra a `/admin/invitations`.
2. Presiona `Generar invitacion`.
3. El sistema devuelve una URL unica.
4. El admin copia el link y lo envia por WhatsApp, correo u otro medio.
5. El invitado abre `/register/invitation/:token`.
6. El invitado captura nombre, correo, contrasena y confirmacion.
7. El backend crea la cuenta con rol `user`.

Regla: el admin solo puede tener una invitacion pendiente activa. No puede crear otra hasta que la invitacion se use o se cancele.

### Solicitud de cuenta

1. En `/login`, el usuario presiona `Mandar peticion de cuenta`.
2. Captura nombre, correo, contrasena, confirmacion y descripcion.
3. El admin entra a `/admin/account-requests`.
4. El admin acepta o rechaza la solicitud.
5. Si acepta, el backend crea el usuario con rol `user`.

## Problemas comunes

### `origin_mismatch` o error de Google

Verifica que el origen exacto de Vite este en Authorized JavaScript origins.

Ejemplos:

```text
http://localhost:3000
http://127.0.0.1:3000
```

El puerto debe coincidir exactamente con el que imprime Vite.

### El frontend llama a una API incorrecta

Revisa `VITE_API_BASE` en `.env`.

Despues reinicia Vite:

```bash
npm run dev
```

### Cambie el `.env` y no se reflejan los cambios

Vite solo carga variables al arrancar. Deten el servidor y vuelve a correr:

```bash
npm run dev
```

### Google login responde que el correo no esta registrado

El backend solo permite Google login si el correo ya existe en la tabla `users`.

Puedes crear el usuario mediante:

- Registro directo por admin.
- Registro por invitacion.
- Solicitud de cuenta aceptada.

## Verificacion rapida

Backend:

```bash
cd ../Backend
dotnet run --project AgendaUAC.csproj
```

Frontend:

```bash
cd ../Frontend
npm run dev
```

Build:

```bash
npm run build
```

