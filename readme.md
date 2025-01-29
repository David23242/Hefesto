# Panel de Administración Glassmorphic

Este proyecto web es un panel de administración construido con Vue.js y un diseño moderno glassmorphic. Permite gestionar de forma intuitiva diversas entidades de la aplicación, como usuarios, campus, secciones y máquinas, ofreciendo una interfaz de usuario atractiva y funcional.

## Características Principales

*   **Interfaz Glassmorphic:** Diseño moderno y atractivo con tarjetas y popups glassmorphic para una experiencia de usuario agradable.
*   **Gestión CRUD (Crear, Leer, Actualizar, Eliminar) para:**
    *   **Usuarios:** Administración completa de usuarios, incluyendo creación, edición, habilitación/deshabilitación y gestión de roles.
    *   **Campus:** Gestión de campus, permitiendo crear, editar y deshabilitar ubicaciones.
    *   **Secciones:** Administración de secciones dentro de los campus, con funcionalidades CRUD y filtro por campus.
    *   **Máquinas:** Gestión de máquinas o equipos, incluyendo tipo, prioridad, sección asignada y estado de habilitación, con filtros por campus y sección.
*   **Dashboard de Inicio:**
    *   **Tarjetas de estadísticas:**  Muestra un resumen visual del estado de las incidencias (pendientes, en curso, cerradas, totales) para una visión rápida del rendimiento.
    *   **Gráficos:** Incluye gráficos de líneas para incidencias abiertas por hora y gráficos de barras para el promedio de incidencias resueltas y abiertas en los últimos 7 días.
    *   **Lista de incidencias recientes:**  Presenta un listado de las incidencias más recientes con detalles clave como fecha, título, subtítulo y estado.
*   **Popups Dinámicos:**
    *   **Popup de Perfil:** Permite a los usuarios editar su foto de perfil mediante carga de archivos o drag-and-drop.
    *   **Popup de Contraseña:** Facilita el cambio de contraseña de usuario de forma segura.
    *   **Popup de Fondo de Pantalla:**  Ofrece opciones para personalizar el fondo de pantalla del panel, permitiendo a los usuarios adaptar la interfaz a sus preferencias.
    *   **Popup de Máquinas (Listado):** Muestra un listado detallado de máquinas, aunque este popup parece ser un componente en desarrollo o una funcionalidad específica.
*   **Filtros y Búsqueda:**  Las listas de usuarios, campus, secciones y máquinas incluyen funcionalidades de búsqueda por nombre y filtros específicos (como campus para secciones y máquinas) para facilitar la gestión de grandes volúmenes de datos.
*   **Paginación:** Las listas extensas se dividen en páginas para mejorar el rendimiento y la experiencia de usuario al navegar por grandes conjuntos de datos.
*   **Control de Estado (Habilitar/Deshabilitar):**  Permite habilitar o deshabilitar entidades (usuarios, campus, secciones, máquinas) directamente desde las listas, controlando el acceso y la visibilidad dentro de la aplicación.
*   **Componentes Reutilizables:** Utilización de componentes Vue.js para elementos comunes como popups glassmorphic, selects personalizados y inputs, promoviendo la reutilización y el mantenimiento del código.
*   **Gestión de Sesión:**  Implementa un sistema de inicio y cierre de sesión, con protección de rutas basada en tokens JWT (almacenados en `localStorage`).
*   **Personalización de Estilos:**  Uso de SCSS para estilos, facilitando la personalización y extensión del diseño glassmorphic y otros aspectos visuales del panel.
*   **Notificaciones:**  Integración con `vue-toastification` para mostrar notificaciones y mensajes de éxito o error de manera atractiva y no intrusiva.

## Tecnologías Utilizadas

*   **Vue.js:** Framework JavaScript progresivo para construir interfaces de usuario.
*   **Vue Router (implícito):**  Para la navegación entre diferentes vistas del panel (Dashboard, CRUDs, Ajustes, etc.).
*   **Axios:** Cliente HTTP basado en promesas para realizar peticiones al backend API.
*   **Vue Toastification:**  Para la gestión de notificaciones toast.
*   **SCSS:** Preprocesador CSS para estilos, permitiendo el uso de variables, mixins y otras funcionalidades para un CSS más mantenible y organizado.
*   **Iconos SVG:**  Utilización de iconos SVG para una interfaz nítida y escalable en diferentes resoluciones.

## Configuración

Este componente Vue.js está diseñado para ser integrado en un proyecto Vue.js existente. Para su correcto funcionamiento, asegúrate de tener configuradas las siguientes variables de entorno en tu proyecto:

*   `VITE_API_AUTH_URL`:  URL base de la API de backend para la autenticación y gestión de datos (ej. `http://localhost:8000/api/v1`).
*   `VITE_IMAGE_URL`: URL base para acceder a las imágenes de perfil de usuario y otros recursos estáticos (ej. `http://localhost:8000/`).

**Dependencias:**

Asegúrate de tener instaladas las dependencias necesarias para Vue.js, Axios y Vue Toastification. Puedes instalarlas utilizando npm o yarn:

```bash
npm install vue axios vue-toastification chart.js
# o
yarn add vue axios vue-toastification chart.js
```

## Instalación con Docker

Para facilitar la puesta en marcha y despliegue de este panel de administración, se puede utilizar Docker y Docker Compose. A continuación, se describen los pasos para instalar y ejecutar la aplicación utilizando contenedores Docker.

**Prerrequisitos:**

*   **Docker:** Asegúrate de tener Docker instalado en tu sistema. Puedes descargarlo e instalarlo desde [https://www.docker.com/get-started](https://www.docker.com/get-started).
*   **Docker Compose (opcional, pero recomendado):** Docker Compose facilita la gestión de aplicaciones multi-contenedor. Si no lo tienes instalado, puedes seguir las instrucciones en [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/).

**Pasos para la instalación con Docker:**

1.  **Construir la aplicación Frontend:**
    Antes de Dockerizar la aplicación, necesitas construir la versión de producción del frontend.  Desde la raíz del proyecto, ejecuta el comando de construcción de tu proyecto Vue.js (normalmente):

    ```bash
    npm install         # Instala las dependencias (si no lo has hecho ya)
    npm run build       # Construye la aplicación para producción
    ```

    Este comando generará una carpeta `dist` (o similar, dependiendo de tu configuración de Vue.js) con los archivos estáticos de la aplicación.

2.  **Crear un Dockerfile para el Frontend:**
    En la raíz de tu proyecto, crea un archivo llamado `Dockerfile` (sin extensión) con el siguiente contenido:

    ```dockerfile
    # Usa una imagen base de Nginx para servir archivos estáticos
    FROM nginx:alpine

    # Copia los archivos construidos del frontend a la carpeta de Nginx
    COPY dist /usr/share/nginx/html

    # (Opcional) Copia un archivo de configuración de Nginx personalizado si lo necesitas
    # COPY nginx.conf /etc/nginx/conf.d/default.conf

    # Expone el puerto 80 para acceder a la aplicación
    EXPOSE 80

    CMD ["nginx", "-g", "daemon off;"]
    ```

    **Explicación del Dockerfile:**

    *   `FROM nginx:alpine`:  Utiliza la imagen oficial de Nginx basada en Alpine Linux, que es ligera y segura.
    *   `COPY dist /usr/share/nginx/html`: Copia la carpeta `dist` (donde se construyó tu frontend) al directorio raíz de Nginx donde se sirven los archivos web.
    *   `EXPOSE 80`: Expone el puerto 80 del contenedor, que es el puerto por defecto de Nginx.
    *   `CMD ["nginx", "-g", "daemon off;"]`:  Inicia el servidor Nginx cuando se ejecuta el contenedor.

3.  **Construir la imagen Docker del Frontend:**
    Abre una terminal en la raíz de tu proyecto (donde se encuentra el Dockerfile) y ejecuta el siguiente comando para construir la imagen Docker. Reemplaza `nombre-de-tu-imagen-frontend` con un nombre descriptivo para tu imagen (ej. `glassmorphic-admin-frontend`):

    ```bash
    docker build -t nombre-de-tu-imagen-frontend .
    ```

4.  **Ejecutar el contenedor Docker (Frontend solamente):**
    Para ejecutar la aplicación en un contenedor Docker, utiliza el siguiente comando. Reemplaza `nombre-de-tu-imagen-frontend` con el nombre que elegiste en el paso anterior y `-p 8080:80` mapea el puerto 80 del contenedor al puerto 8080 de tu máquina local (puedes cambiar el puerto local si lo deseas):

    ```bash
    docker run -d -p 8080:80 nombre-de-tu-imagen-frontend
    ```

    Ahora deberías poder acceder a tu panel de administración en tu navegador web en `http://localhost:8080`.

5.  **(Opcional) Usar Docker Compose para una configuración multi-contenedor (Frontend, Backend, Base de Datos):**
    Si tu aplicación requiere un backend API y/o una base de datos (que es lo más común en una aplicación web completa), Docker Compose es la forma recomendada de gestionarlos juntos.

    Crea un archivo llamado `docker-compose.yml` en la raíz de tu proyecto con un contenido similar a este (ajústalo según las necesidades de tu backend y base de datos):

    ```yaml
    version: "3.9"
    services:
      frontend:
        image: nombre-de-tu-imagen-frontend # Reemplaza con el nombre de tu imagen frontend
        ports:
          - "8080:80"
        # depends_on: # Descomenta si tu frontend depende del backend para iniciar
        #   - backend

      backend:
        # Configura tu servicio backend aquí (ejemplo con una imagen ficticia)
        image: nombre-de-tu-imagen-backend # Reemplaza con la imagen de tu backend
        # build: ./backend # O construye desde un Dockerfile en ./backend
        ports:
          - "3000:3000" # Puerto del backend (ejemplo)
        environment:
          # Define variables de entorno para tu backend si es necesario
          # DATABASE_URL: ...

      database:
        # Configura tu servicio de base de datos aquí (ejemplo con PostgreSQL)
        image: postgres:13-alpine
        ports:
          - "5432:5432" # Puerto de la base de datos (ejemplo)
        environment:
          POSTGRES_USER: tu_usuario_db
          POSTGRES_PASSWORD: tu_contraseña_db
          POSTGRES_DB: nombre_db

    ```

    **Explicación del `docker-compose.yml`:**

    *   `version: "3.9"`:  Versión de Docker Compose.
    *   `services:`: Define los servicios (contenedores) que formarán tu aplicación.
        *   `frontend:`: Servicio para el frontend.
            *   `image: nombre-de-tu-imagen-frontend`:  Utiliza la imagen frontend que construiste previamente.
            *   `ports: - "8080:80"`:  Mapea puertos (igual que en el comando `docker run`).
            *   `depends_on: - backend` (opcional):  Asegura que el backend se inicie antes que el frontend (si es necesario).
        *   `backend:`:  **Servicio de Backend (Ejemplo):**  Debes configurar este servicio con la imagen de Docker de tu backend API, o definir un `build:` context para construirlo desde un Dockerfile si tienes el código fuente del backend en el mismo proyecto.
        *   `database:`: **Servicio de Base de Datos (Ejemplo):**  Se utiliza una imagen de PostgreSQL como ejemplo.  Debes reemplazar esto con la imagen y configuración de la base de datos que uses (MySQL, MongoDB, etc.).  Las variables de entorno (`environment:`) se utilizan para configurar la base de datos.

6.  **Iniciar la aplicación con Docker Compose:**
    En la terminal, en la misma carpeta donde se encuentra `docker-compose.yml`, ejecuta:

    ```bash
    docker-compose up -d --build
    ```

    *   `docker-compose up -d`: Inicia los contenedores en segundo plano.
    *   `--build`:  Reconstruye las imágenes si hay cambios en los Dockerfiles o en `docker-compose.yml`.

    Después de ejecutar este comando, Docker Compose construirá las imágenes (si es necesario) e iniciará los contenedores definidos en `docker-compose.yml`.  Podrás acceder al panel de administración frontend en `http://localhost:8080` (o el puerto que hayas configurado).

**Notas Importantes:**

*   **Configuración de la URL de la API Backend en Docker:**  Si tu frontend necesita comunicarse con un backend API que también está en Docker, asegúrate de configurar correctamente la variable de entorno `VITE_API_AUTH_URL` en tu frontend para que apunte al servicio backend dentro de la red de Docker Compose.  Por ejemplo, si tu servicio backend se llama `backend` en `docker-compose.yml` y se ejecuta en el puerto 3000 dentro del contenedor, podrías configurar `VITE_API_AUTH_URL` como `http://backend:3000/api/v1` (asumiendo que `/api/v1` es el endpoint base de tu API).  **No uses `localhost` dentro del contenedor frontend para referirte al backend en otro contenedor; utiliza el nombre del servicio Docker Compose.**
*   **Variables de entorno en Docker Compose:**  Utiliza la sección `environment:` en `docker-compose.yml` para definir variables de entorno necesarias para tus servicios (frontend, backend, base de datos).
*   **Volúmenes (Persistencia de datos):** Si necesitas persistir datos de la base de datos o configuraciones, considera usar volúmenes Docker en `docker-compose.yml` para evitar la pérdida de datos al reiniciar los contenedores.
*   **Personalización de Nginx:** Si necesitas una configuración de Nginx más avanzada (ej. para SSL, reescrituras de URLs, etc.), puedes crear un archivo de configuración `nginx.conf` personalizado y copiarlo a la configuración de Nginx en el Dockerfile (como se comenta en el ejemplo del Dockerfile).

Recuerda ajustar los nombres de las imágenes, puertos, variables de entorno y configuraciones en los ejemplos de `Dockerfile` y `docker-compose.yml` para que coincidan con la configuración específica de tu proyecto y backend API.
