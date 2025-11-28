# Guía de Instalación y Configuración - EDANOS

## 📋 Requerimientos del Sistema

### Hardware Mínimo

**Servidor (Backend + Database)**

- CPU: 2 cores
- RAM: 4 GB
- Almacenamiento: 20 GB SSD
- Conectividad: Internet estable

**Estación de Trabajo (Web)**

- Navegador moderno (Chrome, Firefox, Edge)
- Conexión a Internet

**Dispositivos Móviles**

- Android 8.0+ o iOS 12+
- 2 GB RAM mínimo
- GPS activo
- Cámara

### Software Necesario

```bash
Node.js >= 18.0.0
npm >= 9.0.0
PostgreSQL >= 14.0
Git >= 2.30
```

---

## 🚀 Instalación Paso a Paso

### 1. Preparación del Entorno

#### Instalar Node.js

**Windows:**

```powershell
# Descargar desde https://nodejs.org/
# O usar Chocolatey
choco install nodejs
```

**Linux/macOS:**

```bash
# Usar nvm (recomendado)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
nvm use 18
```

#### Instalar PostgreSQL

**Windows:**

```powershell
# Descargar desde https://www.postgresql.org/
choco install postgresql
```

**Linux (Ubuntu/Debian):**

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

**macOS:**

```bash
brew install postgresql
brew services start postgresql
```

### 2. Clonar el Repositorio

```bash
git clone https://github.com/tu-usuario/edanos.git
cd edanos
```

### 3. Configurar Base de Datos

```bash
# Conectar a PostgreSQL
psql -U postgres

# Crear base de datos
CREATE DATABASE edanos;
CREATE USER edanos_user WITH ENCRYPTED PASSWORD 'tu_password_seguro';
GRANT ALL PRIVILEGES ON DATABASE edanos TO edanos_user;

# Salir
\q
```

### 4. Configurar Backend

```bash
cd backend

# Instalar dependencias
npm install

# Crear archivo de configuración
cp .env.example .env
```

**Editar `.env`:**

```env
# Base de Datos
DATABASE_URL="postgresql://edanos_user:tu_password_seguro@localhost:5432/edanos"

# JWT
JWT_SECRET="tu_secreto_jwt_muy_seguro_minimo_32_caracteres"

# AWS S3 (Opcional, para imágenes)
AWS_ACCESS_KEY_ID="tu_access_key"
AWS_SECRET_ACCESS_KEY="tu_secret_key"
AWS_REGION="us-east-1"
AWS_BUCKET_NAME="edanos-imagenes"

# Servidor
PORT=4500
NODE_ENV=development

# Email (Para notificaciones)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=tu_email@gmail.com
EMAIL_PASS=tu_app_password
```

**Ejecutar migraciones:**

```bash
npx prisma migrate dev
npx prisma generate

# (Opcional) Seed inicial
npm run seed
```

**Iniciar backend:**

```bash
npm run dev
# El servidor estará en http://localhost:4500
```

### 5. Configurar Frontend Web

```bash
cd ../frontendpc

# Instalar dependencias
npm install

# Crear archivo de configuración
cp .env.example .env
```

**Editar `.env`:**

```env
VITE_API_URL=http://localhost:4500
VITE_SOCKET_URL=http://localhost:4500
```

**Iniciar aplicación:**

```bash
npm run dev
# La app estará en http://localhost:5173
```

### 6. Configurar Aplicación Móvil

```bash
cd ../edan1.2

# Instalar dependencias
npm install

# Instalar Expo CLI globalmente si no lo tienes
npm install -g expo-cli
```

**Editar configuración:**

```bash
# Abrir edan1.2/confg/constants.ts
```

```typescript
// Cambiar la URL del API
//export const API_URL = 'http://localhost:4500';
export const API_URL = "https://tu-servidor.com"; // En producción

export const AUTH_TOKEN_KEY = "edanos_auth_token";
export const USER_DATA_KEY = "edanos_user_data";
export const EVALUATIONS_KEY = "edanos_evaluations";
```

**Iniciar con Expo:**

```bash
npx expo start

# Escanear QR con Expo Go app
# O presionar 'a' para Android emulator
# O presionar 'i' para iOS simulator
```

---

## 🔧 Configuración Avanzada

### Configurar HTTPS (Producción)

#### Con Nginx

```nginx
server {
    listen 80;
    server_name tu-dominio.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name tu-dominio.com;

    ssl_certificate /etc/letsencrypt/live/tu-dominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tu-dominio.com/privkey.pem;

    location /api {
        proxy_pass http://localhost:4500;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location / {
        root /var/www/edanos/frontend;
        try_files $uri /index.html;
    }
}
```

#### Obtener Certificado SSL

```bash
# Instalar Certbot
sudo apt install certbot python3-certbot-nginx

# Obtener certificado
sudo certbot --nginx -d tu-dominio.com
```

### Configurar PM2 (Process Manager)

```bash
# Instalar PM2
npm install -g pm2

# En el directorio backend
pm2 start src/index.js --name edanos-api

# Guardar configuración
pm2 save

# Iniciar en boot
pm2 startup
```

### Docker (Opcional)

**docker-compose.yml:**

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: edanos
      POSTGRES_USER: edanos_user
      POSTGRES_PASSWORD: tu_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  backend:
    build: ./backend
    ports:
      - "4500:4500"
    environment:
      DATABASE_URL: postgresql://edanos_user:tu_password@postgres:5432/edanos
      JWT_SECRET: tu_secreto_jwt
    depends_on:
      - postgres

  frontend:
    build: ./frontendpc
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  postgres_data:
```

**Ejecutar:**

```bash
docker-compose up -d
```

---

## 👥 Configuración Inicial de Usuarios

### Crear SuperAdmin

```bash
# Conectar a PostgreSQL
psql -U edanos_user -d edanos

# Insertar SuperAdmin
INSERT INTO "User" (id, cedula, nombre, apellido, "contrasenaHash", rol, estado, "nivelCuenta", activo, "createdAt", "updatedAt")
VALUES (
  gen_random_uuid(),
  'V12345678',
  'Admin',
  'Sistema',
  '$2a$10$hashedPasswordHere', -- Usar bcrypt para hashear
  'superadmin',
  'Nacional',
  'nacional',
  true,
  NOW(),
  NOW()
);
```

### Login Inicial

```
Usuario: V12345678
Contraseña: [tu password]
```

---

## 📊 Verificación de Instalación

### Checklist

- [ ] PostgreSQL corriendo y accesible
- [ ] Backend iniciado en puerto 4500
- [ ] Frontend accesible en navegador
- [ ] App móvil conecta con backend
- [ ] Socket.IO funcionando (tiempo real)
- [ ] Login exitoso en ambas plataformas
- [ ] Creación de evaluación en móvil
- [ ] Visualización en dashboard web

### Comandos de Diagnóstico

```bash
# Verificar PostgreSQL
psql -U edanos_user -d edanos -c "SELECT version();"

# Verificar backend
curl http://localhost:4500/health

# Ver logs del backend
pm2 logs edanos-api

# Ver estado de servicios
pm2 status
```

---

## 🆘 Resolución de Problemas

### Error: Cannot connect to database

```bash
# Verificar que PostgreSQL está corriendo
sudo systemctl status postgresql

# Reiniciar PostgreSQL
sudo systemctl restart postgresql

# Verificar credenciales en .env
```

### Error: Port 4500 already in use

```bash
# Windows
netstat -ano | findstr :4500
taskkill /PID [PID] /F

# Linux/macOS
lsof -ti:4500 | xargs kill -9
```

### Error: Expo cannot connect to dev server

```
# Asegurarse de estar en la misma red
# Verificar firewall
# Usar túnel: npx expo start --tunnel
```

### Error: Socket.IO not connecting

```javascript
// Verificar CORS en backend
const io = new Server(server, {
	cors: {
		origin: "*", // En desarrollo
		methods: ["GET", "POST"],
	},
});
```

---

## 📞 Soporte

Si encuentras problemas durante la instalación:

1. Revisa los logs del servidor
2. Verifica las variables de entorno
3. Consulta la documentación completa en `/docs`
4. Abre un issue en GitHub

---

**Última actualización:** Noviembre 2024  
**Versión:** 1.2  
**Documentación mantenida por:** Gabriel Osuna
