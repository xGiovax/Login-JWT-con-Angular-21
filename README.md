# Login JWT — Angular 21 + Node.js + MySQL + Angular Material

Sistema de autenticación completo con JSON Web Tokens, construido con Angular 21 en el frontend, Node.js + Express en el backend y MySQL como base de datos. La interfaz usa componentes de Angular Material.

---

## Tecnologías utilizadas

| Capa | Tecnología | Versión |
|---|---|---|
| Frontend | Angular | 21.2.7 |
| UI Library | Angular Material | 21.2.7 |
| Backend | Node.js + Express | v25 / 4.x |
| Autenticación | JSON Web Tokens | jsonwebtoken |
| Encriptación | bcryptjs | 2.x |
| Base de datos | MySQL | 8.0 |
| ORM/Driver | mysql2 | 3.x |
| Dev server | nodemon | 3.x |

---

## Estructura del proyecto

```
Login JWT/
├── frontend/                        # Proyecto Angular 21
│   └── src/
│       └── app/
│           ├── auth/
│           │   ├── guards/
│           │   │   └── auth-guard.ts        # Protege rutas privadas
│           │   ├── interceptors/
│           │   │   └── jwt-interceptor.ts   # Agrega token a cada request
│           │   ├── login/
│           │   │   ├── login.ts             # Componente login
│           │   │   └── login.html           # Template con Angular Material
│           │   └── services/
│           │       └── auth.ts              # Lógica de autenticación
│           ├── dashboard/
│           │   ├── dashboard.ts             # Componente dashboard protegido
│           │   └── dashboard.html           # Template del dashboard
│           ├── app.config.ts                # Configuración global + interceptor
│           └── app.routes.ts                # Definición de rutas
│
└── backend/                         # Servidor Node.js
    ├── db/
    │   └── connection.js            # Conexión pool a MySQL
    ├── middleware/
    │   └── auth.middleware.js       # Verifica JWT en rutas protegidas
    ├── routes/
    │   └── auth.routes.js           # POST /auth/login y /auth/register
    ├── .env                         # Variables de entorno (no subir a git)
    └── server.js                    # Punto de entrada del servidor
```

---

## Requisitos previos

Antes de correr el proyecto asegurate de tener instalado:

- [Node.js](https://nodejs.org/) v18 o superior
- [Angular CLI](https://angular.dev/tools/cli) v21
- [MySQL Server](https://dev.mysql.com/downloads/) 8.0
- [MySQL Workbench](https://dev.mysql.com/downloads/workbench/) (recomendado)

---

## Instalación y configuración

### 1. Clonar o descargar el proyecto

```bash
cd "Login JWT"
```

### 2. Configurar la base de datos

Abrí MySQL Workbench y ejecutá:

```sql
CREATE DATABASE jwt_app;
USE jwt_app;

CREATE TABLE users (
  id         INT PRIMARY KEY AUTO_INCREMENT,
  name       VARCHAR(100) NOT NULL,
  email      VARCHAR(150) UNIQUE NOT NULL,
  password   VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Insertar un usuario de prueba (contraseña: `password`):

```sql
INSERT INTO users (name, email, password)
VALUES (
  'Usuario Test',
  'test@mail.com',
  '$2b$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi'
);
```

### 3. Configurar el backend

```bash
cd backend
npm install
```

Editá el archivo `.env` con tus credenciales:

```env
PORT=3000
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=tu_password_mysql
DB_NAME=jwt_app
JWT_SECRET=mi_clave_super_secreta_jwt_2024
JWT_EXPIRES_IN=1h
```

### 4. Configurar el frontend

```bash
cd frontend
npm install
```

---

## Correr el proyecto

Necesitás **dos terminales abiertas al mismo tiempo**:

**Terminal 1 — Backend:**
```bash
cd backend
npx nodemon server.js
# Debe mostrar: Servidor corriendo en puerto 3000
```

**Terminal 2 — Frontend:**
```bash
cd frontend
ng serve
# Abrí: http://localhost:4200
```

---

## Credenciales de prueba

```
Email:      test@mail.com
Contraseña: password
```

---

## Flujo de autenticación

```
1. Usuario ingresa email + password en el formulario
2. Angular llama a POST http://localhost:3000/auth/login
3. Backend busca el usuario en MySQL por email
4. bcrypt.compare() verifica la contraseña contra el hash guardado
5. Si es válida: jwt.sign() genera un token con expiración de 1h
6. Angular guarda el token en localStorage
7. Angular redirige a /dashboard
8. AuthGuard verifica el token antes de permitir acceso
9. JWT Interceptor agrega "Authorization: Bearer <token>" a cada request
10. Al expirar el token: interceptor detecta 401 y redirige al login
```

---

## Componentes de Angular Material utilizados

| Componente | Uso |
|---|---|
| `MatCard` | Contenedor del formulario de login |
| `MatFormField` | Campos de email y contraseña |
| `MatInput` | Inputs del formulario |
| `MatButton` | Botón de login y cerrar sesión |
| `MatToolbar` | Barra de navegación del dashboard |
| `MatSnackBar` | Mensajes de error en el login |
| `MatProgressSpinner` | Indicador de carga |

---

## Variables de entorno

| Variable | Descripción | Ejemplo |
|---|---|---|
| `PORT` | Puerto del servidor backend | `3000` |
| `DB_HOST` | Host de MySQL | `localhost` |
| `DB_USER` | Usuario de MySQL | `root` |
| `DB_PASSWORD` | Contraseña de MySQL | `admin123` |
| `DB_NAME` | Nombre de la base de datos | `jwt_app` |
| `JWT_SECRET` | Clave secreta para firmar tokens | `clave_larga_aqui` |
| `JWT_EXPIRES_IN` | Tiempo de expiración del token | `1h`, `8h`, `7d` |

---

## Endpoints del backend

| Método | Ruta | Descripción | Auth requerida |
|---|---|---|---|
| POST | `/auth/login` | Inicia sesión y retorna JWT | No |
| POST | `/auth/register` | Registra un nuevo usuario | No |

### POST /auth/login

Request:
```json
{
  "email": "test@mail.com",
  "password": "password"
}
```

Response exitoso (200):
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "name": "Usuario Test"
}
```

Response con error (401):
```json
{
  "message": "Credenciales inválidas"
}
```

---

## Seguridad implementada

- Las contraseñas se encriptan con **bcrypt** (salt rounds: 10) antes de guardarse
- Los tokens JWT expiran automáticamente según `JWT_EXPIRES_IN`
- El archivo `.env` está en `.gitignore` y nunca se sube al repositorio
- El middleware verifica el token en cada request a rutas protegidas
- El interceptor maneja automáticamente los errores 401 redirigiendo al login

---

## Autor
Giovanni Alberto Ruano Martínez
