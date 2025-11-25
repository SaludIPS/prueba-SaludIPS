# SaludVital Cloud Platform

Plataforma web basada en microservicios para gestionar pacientes, médicos, citas e inventario/recetas de farmacia de la IPS simulada **SaludVital IPS**, desplegada en **Microsoft Azure**. 

---

## 1. Descripción del proyecto

**SaludVital Cloud Platform** busca apoyar la transformación digital de SaludVital IPS, permitiendo:

* Registrar y gestionar pacientes y médicos.
* Agendar, consultar, reprogramar y cancelar citas médicas.
* Gestionar inventario de medicamentos y recetas.
* Centralizar el acceso mediante un portal web con login y control de roles (paciente, médico, admin). 

---

## 2. Arquitectura y módulos

La solución está construida con una arquitectura de **microservicios**:

* **Frontend**

  * `login-client`: App React para autenticación de usuarios.
  * `portal`: App React que muestra los módulos disponibles según el rol.
* **API Gateway**

  * `api-gateway`: Servicio Node.js/Express que centraliza las peticiones, valida el JWT y enruta a los microservicios.
* **Microservicios backend**

  * `patients-api`: CRUD de pacientes.
  * `doctors-api`: CRUD de médicos.
  * `appointments-api`: Gestión de citas médicas.
  * `pharmacy-api`: Gestión de medicamentos y recetas (usa Mongo/Cosmos DB). 

Cada microservicio se despliega en contenedores Docker y se expone a través del API Gateway.

---

## 3. Stack tecnológico

### Frontend

* **React + Vite** (portal y login).
* Consumo de APIs REST usando `fetch` hacia el **API Gateway**.
* Despliegue: **Azure App Service (Web App for Containers)**.

### Backend

* **Node.js + Express** para:

  * `api-gateway`
  * `patients-api`, `doctors-api`, `appointments-api`, `pharmacy-api`
* APIs REST (GET, POST, PUT, DELETE) con JSON.

### Autenticación y seguridad

* **JWT (JSON Web Token)**:

  * Generación de *access token* y *refresh token* en el login.
  * El token incluye rol e identificador de usuario para controlar el acceso a módulos. 

### Bases de datos

* **PostgreSQL** (Azure Database for PostgreSQL):

  * `patients-api`
  * `doctors-api`
  * `appointments-api`
* **Azure Cosmos DB – API MongoDB**:

  * `pharmacy-api` (inventario y recetas; colección `counters` para autoincrementar IDs). 

### Infraestructura en la nube

* **Nube**: Microsoft **Azure**.
* **Docker** para empaquetar cada microservicio y frontend.
* **Azure Container Registry (ACR)** para almacenar las imágenes.
* **Azure App Service (Containers)** para desplegar las imágenes.
* **GitHub Actions** para CI/CD:

  * Build de imágenes.
  * Push a ACR.
  * Deploy a App Service. 

---

## 4. Requisitos previos

Para ejecutar el proyecto localmente y/o desplegarlo:

* **Node.js** >= 18.x
* **npm** (o yarn)
* **Docker** instalado y en ejecución
* Cuenta en **Microsoft Azure** con:

  * Azure Container Registry (ACR)
  * Azure App Service para contenedores
  * Azure Database for PostgreSQL
  * Azure Cosmos DB (API MongoDB)

---

## 5. Estructura del repositorio (sugerida)

> Ajusta los nombres según tu repo real si cambian.

```text
.
├── api-gateway/
├── patients-api/
├── doctors-api/
├── appointments-api/
├── pharmacy-api/
├── login-client/
├── portal/
└── README.md
```

Cada carpeta contiene un proyecto Node.js (para APIs/gateway) o React (para frontends).

---

## 6. Configuración de variables de entorno

Cada servicio usa un archivo `.env` (no se debe subir al repositorio). A continuación, variables típicas (pueden variar según tu implementación):

### 6.1. API Gateway (`api-gateway/.env`)

```env
PORT=4000

JWT_SECRET=super_secreto
JWT_EXPIRES_IN=1h
REFRESH_TOKEN_SECRET=super_secreto_refresh
REFRESH_TOKEN_EXPIRES_IN=7d

PATIENTS_API_URL=http://localhost:4100
DOCTORS_API_URL=http://localhost:4200
APPOINTMENTS_API_URL=http://localhost:4300
PHARMACY_API_URL=http://localhost:4400
```

### 6.2. Microservicios PostgreSQL

`patients-api/.env` (similar para `doctors-api` y `appointments-api`):

```env
PORT=4100

DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=tu_password
DB_NAME=saludvital_patients
```

### 6.3. pharmacy-api (Cosmos DB / MongoDB)

`pharmacy-api/.env`:

```env
PORT=4400

MONGO_URI=mongodb://usuario:password@host:puerto
MONGO_DB_NAME=saludvital_pharmacy
```

### 6.4. Frontends (`login-client/.env`, `portal/.env`)

Para Vite normalmente:

```env
VITE_API_BASE=http://localhost:4000
```

En Azure, `VITE_API_BASE` debe apuntar a la URL pública del **API Gateway**.

---

## 7. Ejecución local (sin Docker)

### 7.1. Clonar el repositorio

```bash
git clone https://github.com/<tu-usuario>/<tu-repo>.git
cd <tu-repo>
```

### 7.2. Instalar dependencias backend

En cada carpeta de backend:

```bash
cd api-gateway
npm install
cd ../patients-api
npm install
cd ../doctors-api
npm install
cd ../appointments-api
npm install
cd ../pharmacy-api
npm install
```

### 7.3. Instalar dependencias frontend

```bash
cd ../login-client
npm install
cd ../portal
npm install
```

### 7.4. Iniciar bases de datos (local)

* Crear una instancia de **PostgreSQL** local (o usar una remota) y crear las bases necesarias.
* Crear una instancia de **MongoDB** local (o usar Cosmos DB) y configurar la `MONGO_URI`.

Actualiza las variables de entorno de cada API con tus credenciales/conexiones.

### 7.5. Levantar los servicios backend

En terminales diferentes (o usando herramientas como `concurrently`):

```bash
# API Gateway
cd api-gateway
npm run dev  # o npm start

# Patients API
cd ../patients-api
npm run dev

# Doctors API
cd ../doctors-api
npm run dev

# Appointments API
cd ../appointments-api
npm run dev

# Pharmacy API
cd ../pharmacy-api
npm run dev
```

### 7.6. Levantar los frontends

```bash
# Login
cd ../login-client
npm run dev

# Portal
cd ../portal
npm run dev
```

### 7.7. Acceso a la aplicación

1. Abrir el **login** en el navegador (por ejemplo, `http://localhost:5173` o el puerto que configure Vite).
2. Iniciar sesión con un usuario válido.
3. El sistema redirige al **portal**, que consume el **API Gateway** y muestra los módulos según el rol.

---

## 8. Ejecución con Docker (local / pruebas)

> Si ya tienes Dockerfiles en cada servicio, puedes construir y correr contenedores.

### 8.1. Construir imágenes

Desde la raíz del repo (o dentro de cada carpeta):

```bash
# Ejemplo para api-gateway
cd api-gateway
docker build -t saludvital/api-gateway .

# Repetir para cada servicio y frontend
```

### 8.2. Ejecutar contenedores

```bash
docker run -d --name api-gateway -p 4000:4000 --env-file .env saludvital/api-gateway
# Repetir para cada microservicio y frontend, mapeando puertos y .env correspondientes
```

> Opcionalmente, puedes crear un `docker-compose.yml` para orquestar todos los servicios.

---

## 9. Despliegue en Microsoft Azure (alto nivel)

1. **Crear Azure Container Registry (ACR)**

   * Crear un registro (ej. `acrsaludvital.azurecr.io`).
   * Hacer login desde GitHub Actions / local.

2. **Build & push de imágenes a ACR**

   * En GitHub Actions (workflows) o local:

     ```bash
     docker build -t acrsaludvital.azurecr.io/api-gateway:latest ./api-gateway
     docker push acrsaludvital.azurecr.io/api-gateway:latest
     ```
   * Repetir para cada API y frontend.

3. **Crear Azure App Service para contenedores**

   * Uno por cada servicio (gateway, microservicios, login, portal) o los que definas.
   * Configurar la imagen de contenedor correspondiente (desde ACR).

4. **Configurar variables de entorno en App Service**

   * Definir todas las variables `.env` de cada servicio en la sección **Configuration**.
   * En el API Gateway, usar las URL **internas o públicas** de cada microservicio en Azure.

5. **Configurar bases de datos en Azure**

   * Crear **Azure Database for PostgreSQL** y configurar:

     * Host, puerto, usuario, contraseña y nombres de BD en las variables de entorno de `patients-api`, `doctors-api`, `appointments-api`.
   * Crear **Azure Cosmos DB – API MongoDB** y configurar `MONGO_URI` y `MONGO_DB_NAME` en `pharmacy-api`.

6. **Automatizar con GitHub Actions**

   * El repositorio puede incluir workflows de CI/CD que:

     * Compilen y construyan imágenes Docker.
     * Hagan push a ACR.
     * Desplieguen automáticamente al App Service correspondiente.

---

## 10. Uso básico de la aplicación en producción

1. Ingresar a la URL pública del **login** (App Service del login).
2. Autenticarse con un usuario válido.
3. Una vez dentro del **portal**, navegar entre:

   * **Pacientes**: CRUD de pacientes.
   * **Médicos**: CRUD de médicos.
   * **Citas**: crear, reprogramar, cancelar, listar.
   * **Farmacia**: inventario de medicamentos y recetas asociadas a pacientes. 

---

## 11. Limitaciones actuales (deuda técnica resumida)

* No se han implementado conectores **RIPS** y **MIPRES**.
* No existe aún una **historia clínica electrónica completa** (EHR) ni módulo PQRS.
* Faltan mecanismos avanzados de seguridad (SSO Entra ID, private endpoints) y observabilidad extendida (trazas distribuidas, SLO/SLA formales).
* Pruebas automatizadas aún pueden ampliarse (unitarias, integración, contrato entre microservicios). 

---

## 12. Autores

* Jean Paul Solano
* Obrian Steven Sanchez
* Deyvid Santiago Galarza
* Andrés Felipe Cardoso

---
