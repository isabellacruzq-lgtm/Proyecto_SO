# Proyecto Final Corte 3 – Sistemas Operativos

**Asignatura:** Sistemas Operativos  
**Evaluación:** Trabajo Final – Corte 3  
**Docente:** Cristian Gutierrez

**Estudiantes:**
- Isabella Cruz Q.
- Juan Pablo Aguirre S.

---

## 1. Descripción del proyecto

Este proyecto consiste en desplegar una aplicación web distribuida utilizando servicios de computación en la nube de **Amazon Web Services (AWS)** y **Microsoft Azure**.

La aplicación presenta información histórica sobre sistemas operativos, obtenida desde una API conectada a una base de datos PostgreSQL.

El objetivo principal es implementar y comprobar la comunicación entre un frontend alojado en AWS y un backend desplegado en Azure, aplicando conceptos de administración de servidores Linux, contenedores Docker, redes, servidores web, dominios y certificados SSL.

## 2. Arquitectura del sistema

La arquitectura está compuesta por los siguientes elementos:

| Componente | Tecnología | Plataforma |
|---|---|---|
| Frontend | HTML, CSS y JavaScript | AWS EC2 |
| Administrador de procesos | PM2 | AWS EC2 |
| Servidor web | Nginx | AWS EC2 |
| Backend | Python y Flask | Microsoft Azure |
| Servidor de aplicaciones | Gunicorn | Docker |
| Base de datos | PostgreSQL 16 | Docker en Azure |
| Orquestación de contenedores | Docker Compose | Microsoft Azure |
| Repositorio de imágenes | Docker Hub | Docker Hub |
| Certificados SSL | Let's Encrypt y Certbot | AWS y Azure |
| Control de versiones | Git y GitHub | GitHub |

### Flujo de comunicación

```text
Usuario
   |
   v
Frontend HTTPS (AWS EC2)
   |
   v
Nginx + PM2
   |
   | Solicitud HTTPS
   v
API HTTPS (Microsoft Azure)
   |
   v
Nginx (proxy inverso)
   |
   v
Backend Flask + Gunicorn (Docker)
   |
   v
PostgreSQL (Docker)
   |
   v
Respuesta JSON
   |
   v
Frontend: tabla de sistemas operativos
```

El frontend consume la API mediante solicitudes HTTP GET sobre HTTPS. El backend consulta PostgreSQL y devuelve los datos en formato JSON.

## 3. Estructura del proyecto

```text
Proyecto_SO/
│
├── frontend/
│   ├── index.html
│   └── config.js
│
├── backend/
│   ├── app.py
│   ├── Dockerfile
│   ├── requirements.txt
│   └── .env.example
│
├── db/
│   └── init.sql
│
├── Evidencias/
│   ├── 01_docker_hub.png
│   ├── 02_imagen_docker_local.png
│   ├── 03_contenedores_azure.png
│   ├── 04_api_salud.png
│   ├── 05_api_sistemas.png
│   ├── 06_certificado_backend.png
│   ├── 07_integracion_aws_azure.png
│   ├── 08_prueba_cors.png
│   ├── 09_instancia_EC2_AWS.png
│   ├── 10_PM2_frontend_AWS.png
│   ├── 11_configuracion_Nginx_AWS.png
│   ├── 12_certificado_HTTPS_AWS.png
│   └── 13_integracion_AWS_Azure_CORS.png
│
├── compose.yaml
├── .gitignore
└── README.md
```

## 4. Despliegue del frontend en AWS

El frontend se desplegó en una instancia EC2 con Ubuntu.

Se configuraron manualmente los servicios necesarios para ejecutar la aplicación y permitir su acceso desde Internet.

### Servicios utilizados

**Amazon EC2:** proporciona el servidor Linux donde se encuentra alojado el frontend.

**PM2:** administra el proceso de la aplicación y permite supervisar su ejecución.

**Nginx:** funciona como proxy inverso, recibe las solicitudes de los usuarios y las dirige al servicio local del frontend.

**Certbot:** permite generar y configurar certificados SSL de Let's Encrypt para habilitar HTTPS.

### Verificación de PM2

```bash
pm2 list
```

El proceso `frontend-juanisa` se verificó con estado `online`.

### Verificación de Nginx

```bash
sudo nginx -t
```

La prueba de configuración se completó correctamente.

### Verificación del certificado SSL

```bash
sudo certbot certificates
```

Se comprobó que el certificado del dominio del frontend estaba vigente.

### Dirección del frontend

https://juanisa.scaleautomatization.tech

## 5. Despliegue del backend en Azure

El backend se desplegó en una máquina virtual Linux de Microsoft Azure.

Se utilizó Docker para ejecutar la aplicación Flask y PostgreSQL en contenedores independientes.

### Servicios implementados

**Contenedor `so-backend`:** ejecuta la API desarrollada en Python con Flask y Gunicorn.

**Contenedor `so-db`:** ejecuta PostgreSQL y almacena los registros de sistemas operativos.

**Docker Compose:** define los servicios, la red interna, los volúmenes y las dependencias entre los contenedores.

**Nginx:** funciona como proxy inverso para recibir las solicitudes HTTPS y dirigirlas al backend.

### Verificación de los contenedores

```bash
sudo docker compose ps
```

Se comprobó el funcionamiento de:

- `so-backend`: contenedor en ejecución.
- `so-db`: contenedor en ejecución y saludable.

La base de datos no se expone directamente a Internet. La comunicación con el backend se realiza mediante la red interna de Docker.

### Dirección de la API

https://api-juanisa.scaleautomatization.tech

## 6. Creación y publicación de la imagen Docker

La imagen del backend se construyó a partir de un Dockerfile que utiliza Python 3.12.

El Dockerfile instala las dependencias y configura Gunicorn para ejecutar la aplicación Flask.

### Construcción de la imagen

```bash
docker build -t isabellacruz/so-backend:1.0 ./backend
```

### Publicación en Docker Hub

```bash
docker push isabellacruz/so-backend:1.0
```

La imagen fue publicada correctamente en Docker Hub.

**Imagen utilizada:**

`isabellacruz/so-backend:1.0`

**Repositorio:**

https://hub.docker.com/r/isabellacruz/so-backend

## 7. Configuración con Docker Compose

El archivo `compose.yaml` define dos servicios principales.

### Servicio de base de datos

- Utiliza la imagen `postgres:16`.
- Ejecuta el script `db/init.sql` durante la inicialización de una base de datos nueva.
- Utiliza un volumen persistente para almacenar los datos.
- Incluye una comprobación de salud mediante `pg_isready`.

### Servicio del backend

- Utiliza la imagen publicada en Docker Hub.
- Carga sus variables de entorno desde `backend/.env`.
- Expone el servicio únicamente en `127.0.0.1:8000` del servidor.
- Espera a que PostgreSQL esté saludable antes de iniciarse.

### Comandos de despliegue

Desde la carpeta raíz del proyecto en Azure:

```bash
sudo docker compose pull backend
sudo docker compose up -d
sudo docker compose ps
```

Para consultar los registros:

```bash
sudo docker compose logs backend
```

## 8. Base de datos PostgreSQL

La base de datos contiene información histórica sobre ocho sistemas operativos.

| Año | Sistema operativo | Tipo |
|---|---|---|
| 1969 | UNIX | Tiempo compartido |
| 1981 | MS-DOS | Monousuario |
| 1984 | Mac OS | Escritorio |
| 1991 | Linux | Kernel monolítico |
| 1993 | Windows NT | Kernel híbrido |
| 2003 | FreeRTOS | Tiempo real |
| 2007 | iOS | Móvil |
| 2008 | Android | Móvil |

Los registros se cargan mediante el archivo `db/init.sql`.

## 9. Endpoints de la API

La API dispone de dos endpoints principales.

### GET /api/salud

Permite comprobar el funcionamiento del backend y su conexión con PostgreSQL.

**URL:**

https://api-juanisa.scaleautomatization.tech/api/salud

**Respuesta esperada:**

```json
{
  "estado": "ok"
}
```

### GET /api/sistemas

Consulta los registros almacenados en PostgreSQL y devuelve la información en formato JSON.

**URL:**

https://api-juanisa.scaleautomatization.tech/api/sistemas

La respuesta contiene los ocho sistemas operativos utilizados por el frontend para construir la tabla.

## 10. Dominios y seguridad HTTPS

Se configuraron dos subdominios independientes:

| Servicio | Dominio |
|---|---|
| Frontend | juanisa.scaleautomatization.tech |
| Backend | api-juanisa.scaleautomatization.tech |

Ambos servicios utilizan HTTPS mediante certificados SSL de Let's Encrypt.

En Azure, Nginx recibe las conexiones públicas por HTTPS y las redirige al backend que se ejecuta localmente en el puerto 8000.

La base de datos PostgreSQL permanece dentro de la red de Docker y no publica su puerto 5432 hacia Internet.

## 11. Configuración de CORS

El backend utiliza la variable de entorno `ORIGEN_PERMITIDO` para autorizar las solicitudes del frontend.

```env
ORIGEN_PERMITIDO=https://juanisa.scaleautomatization.tech
```

Se verificó la comunicación mediante las herramientas de desarrollo del navegador.

La solicitud a `/api/sistemas` obtuvo:

```text
Status Code: 200 OK
Access-Control-Allow-Origin: https://juanisa.scaleautomatization.tech
```

Esto confirma que el navegador puede recibir correctamente las respuestas de la API desde el dominio del frontend.

## 12. Pruebas realizadas

Se realizaron las siguientes verificaciones:

1. Publicación de la imagen Docker en Docker Hub.
2. Comprobación de los contenedores mediante Docker Compose.
3. Verificación del estado saludable de PostgreSQL.
4. Consulta del endpoint `/api/salud`.
5. Consulta del endpoint `/api/sistemas`.
6. Verificación del certificado HTTPS del backend.
7. Comprobación de la comunicación entre frontend y backend.
8. Verificación de los encabezados CORS.
9. Comprobación del estado de la instancia EC2.
10. Verificación del proceso del frontend mediante PM2.
11. Validación de la configuración de Nginx en AWS.
12. Verificación del certificado HTTPS del frontend.
13. Inspección de la petición HTTP desde el navegador, confirmando la respuesta `200 OK`.

Las capturas correspondientes se encuentran en la carpeta `Evidencias/`.

## 13. Variables de entorno

El archivo `backend/.env.example` contiene la estructura de las variables necesarias para ejecutar los servicios.

Para realizar el despliegue se debe crear un archivo `backend/.env` con los valores correspondientes al entorno.

El archivo `.env` no debe publicarse en GitHub porque contiene información confidencial, como las credenciales de la base de datos.

## 14. Integrantes del proyecto

**Isabella Cruz Q.**

Participación en la configuración y despliegue del backend en Microsoft Azure, creación de la imagen Docker, publicación en Docker Hub, configuración de Docker Compose, PostgreSQL, Nginx, HTTPS y realización de pruebas de los servicios.

**Juan Pablo Aguirre S.**

Participación en la configuración y despliegue del frontend en AWS EC2, administración del proceso mediante PM2, configuración de Nginx, certificado HTTPS y verificación de la integración del frontend con la API.

## 15. Conclusiones

El proyecto permitió implementar una arquitectura distribuida utilizando dos proveedores de servicios en la nube: AWS y Microsoft Azure.

Se logró desplegar el frontend en una instancia EC2 y el backend en contenedores Docker sobre Azure, estableciendo comunicación entre ambos mediante HTTPS.

La implementación de Docker Compose permitió administrar el backend y la base de datos PostgreSQL como servicios independientes, mientras que Nginx y Certbot facilitaron la publicación segura de la aplicación.

Finalmente, se comprobó que el frontend puede consultar correctamente la API y mostrar los ocho registros almacenados en PostgreSQL.

---

**Proyecto Final – Sistemas Operativos**  
**Docente:** Cristian Gutierrez  
**Estudiantes:** Isabella Cruz Q. y Juan Pablo Aguirre S.