
## Platanforma de gestión de incidencias 

![Logo de HefestoxEgibide](https://i.imgur.com/q7tnMn0.png)

Esta versión de Hefesto es una colaboración con Egibide 

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


## Instalación de la aplicación 
 **Iniciar la aplicación con Docker Compose:**
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
