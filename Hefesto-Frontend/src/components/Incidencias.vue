<template>
  <div class="row g-4 mb-4">
    <!-- Cabecera con paneles de resumen -->
    <div class="col-sm-6 col-md-3" @click="openCreateIncidenciaPopup">
      <div class="card glassmorphic-card colored-shadow-pending create-incidencia-card">
        <div class="card-bodytotal card-body-same-height d-flex justify-content-between align-items-center">
          <h6 class="mb-0">Crear incidencia</h6>
          <img src="../assets/images/icons/crear.svg" />
        </div>
      </div>
    </div>
    <div class="col-sm-6 col-md-3">
      <div class="card glassmorphic-card colored-shadow-pending">
        <div class="card-body">
          <h6>Incidencias pendientes</h6>
          <div class="d-flex justify-content-between align-items-center">
            <h2 class="pendiente-h2">{{ incidenciasPendientesCount }}</h2>
            <img src="../assets/images/icons/pendientes.svg" />
          </div>
        </div>
      </div>
    </div>
    <div class="col-sm-6 col-md-3">
      <div class="card glassmorphic-card colored-shadow-in-progress">
        <div class="card-body">
          <h6>Incidencias en curso</h6>
          <div class="d-flex justify-content-between align-items-center">
            <h2 class="curso-h2">{{ incidenciasEnCursoCount }}</h2>
            <img src="../assets/images/icons/curso.svg" />
          </div>
        </div>
      </div>
    </div>
    <div class="col-sm-6 col-md-3">
      <div class="card glassmorphic-card colored-shadow-closed">
        <div class="card-body">
          <h6>Incidencias cerradas</h6>
          <div class="d-flex justify-content-between align-items-center">
            <h2 class="cerrado-h2">{{ incidenciasCerradasCount }}</h2>
            <img src="../assets/images/icons/cerrados.svg" />
          </div>
        </div>
      </div>
    </div>

<!-- Listado de incidencias -->

<div class="col-12">
  <div class="incidencia-list glassmorphic-card">
    <div class="incidencia-list-header">
      <div>Incidencias</div>
      <div class="priority-legend">
        <!-- Botón y leyenda de filtro -->
        <div class="boton-filtro" @click="openFiltroPopup">
          <img src="../assets/images/icons/filtro.svg" alt="Filtro" class="imagen-filtro">
          Filtros
        </div>
        <!-- Filtros de prioridad -->
        <span
          @click="seleccionarPrioridad('alta')"
          :class="{ 'selected': prioridadSeleccionada === 'alta' }"
          class="priority-filter"
        >
          <span class="priority-dot alta"></span> Alta
        </span>
        <span
          @click="seleccionarPrioridad('media')"
          :class="{ 'selected': prioridadSeleccionada === 'media' }"
          class="priority-filter"
        >
          <span class="priority-dot media"></span> Media
        </span>
        <span
          @click="seleccionarPrioridad('baja')"
          :class="{ 'selected': prioridadSeleccionada === 'baja' }"
          class="priority-filter"
        >
          <span class="priority-dot baja"></span> Baja
        </span>
        <span
          @click="seleccionarPrioridad(null)"
          :class="{ 'selected': prioridadSeleccionada === null }"
          class="priority-filter"
        >
          Todos
        </span>
      </div>
    </div>

<!-- Mensajes de carga o error -->
<div v-if="loading" class="loading-container">
  <div class="spinner-border text-dark" role="status">
    <span class="visually-hidden">Cargando...</span>
  </div>
  <span>Cargando incidencias...</span>
</div>
<div v-else-if="error">Error al cargar las incidencias.</div>

<!-- Contenedor de incidencias -->
<div v-else class="incidencias-container">
  <div
    v-for="incidencia in filteredIncidenciaPrioridad"
    :key="incidencia.id"
    class="incidencia-item"
    @click="handleIncidenciaClick(incidencia)"
  >
    <div class="priority-marker" :class="incidencia.priority"></div>
    <div class="incidencia-content">
      <div class="incidencia-date">
        <span>{{ incidencia.date }}</span>
        <span>{{ incidencia.time }}</span>
      </div>
      <div class="incidencia-text">
        <div>{{ incidencia.titulo }}</div>
        <small class="text-muted">{{ incidencia.subtitulo }}</small>
      </div>
      <div class="incidencia-status-box">
        <span
          class="incidencia-status"
          :class="incidencia.status.toLowerCase().replace(' ', '-')"
        >
          {{ incidencia.status }}
        </span>
      </div>
    </div>
  </div>
</div>
  </div>
</div>

<!-- Popups -->

<!-- Popup para crear incidencia -->


<GlassmorphicPopup
:visible="showCreateIncidenciaPopup"
title="Crear Nueva Incidencia"
closeButtonText="Cancelar"
actionButtonText="Crear Incidencia"
@close="closeCreateIncidenciaPopup"
@action="handleCreateIncidencia"
>
<template #popup-content>
<div class="form-group mb-3">
<label for="titulo">Título:</label>
<CustomInput placeholder="Ingrese el título" v-model="newIncidencia.titulo" />
</div>
<div class="form-group mb-3">
<label for="subtitulo">Subtítulo:</label>
<CustomInput
placeholder="Ingrese el subtítulo"
v-model="newIncidencia.subtitulo"
/>
</div>
<div class="form-group mb-3">
<label for="tipoIncidencia">Tipo de Incidencia:</label>
<CustomSelect
:options="tiposIncidencia"
v-model="newIncidencia.id_tipo_incidencia"
/>
</div>
<div class="form-group mb-3">
<label for="maquina">Máquina:</label>
<CustomSelect :options="maquinas" v-model="newIncidencia.id_maquina" />
</div>
</template>
</GlassmorphicPopup>

<!-- Popup para detalles de incidencia -->


<GlassmorphicPopup
:visible="showIncidenciaDetailsPopup"
title="Detalles de la Incidencia"
closeButtonText="Cerrar"
@close="closeIncidenciaDetailsPopup"
>
<template #popup-content>
<div v-if="selectedIncidencia">
<p><strong>Título:</strong> {{ selectedIncidencia.titulo }}</p>
<p><strong>Subtítulo:</strong> {{ selectedIncidencia.subtitulo }}</p>
<p><strong>Fecha:</strong> {{ selectedIncidencia.date }}</p>
<p><strong>Hora:</strong> {{ selectedIncidencia.time }}</p>
<p><strong>Estado:</strong> {{ selectedIncidencia.status }}</p>

<div v-if="!isOperario">
    <label for="comment">Comentario:</label>
    <textarea
      id="comment"
      v-model="commentText"
      class="form-control custom-textarea"
      rows="3"
      placeholder="Agrega un comentario"
      @input="handleCommentChange"
    ></textarea>
  </div>
  <div v-else-if="selectedIncidencia.descripcion">
    <p><strong>Comentario:</strong> {{ selectedIncidencia.descripcion }}</p>
  </div>
  <div v-else>
    <p>No hay comentarios para mostrar</p>
  </div>
  <div class="action-buttons mt-auto"  v-if="!isOperario">
    <div class="action-buttons-row">
        <button
          v-if="(isTecnico || isAdmin) && !isUserActiveInIncidencia && selectedIncidencia.status !== 'Cerrada'"
          class="popup-btn primary"
          @click="handleReclamarIncidencia"
          :disabled="reclamandoIncidencia"
        >
          Reclamar Incidencia
        </button>
        <div v-if="isUserActiveInIncidencia && selectedIncidencia.status === 'En curso'" class="d-flex justify-content-center gap-2">
          <button class="popup-btn1 cancel-btn" @click="openMotivoSalidaPopup">Salir de Incidencia</button>
          <button  v-if="isSoloTecnico" class="popup-btn primary" @click="openMotivoCierrePopup">Cerrar Incidencia</button>
        </div>
    </div>
    <div class="action-buttons-row">
        <!-- Botón para ver detalles de técnicos CENTRADO -->
        <button v-if="(isTecnico || isAdmin)" class="popup-btn primary" @click="openTecnicoDetailsPopup">Detalle Técnicos</button>
    </div>
  </div>

</div>
<div v-else>
  <p>No hay detalles de incidencia para mostrar</p>
</div>
  </template>
</GlassmorphicPopup>

<!-- Popup para detalles de técnicos en la incidencia -->
<GlassmorphicPopup
  :visible="showTecnicoDetailsPopup"
  title="Detalle de Técnicos en Incidencia"
  closeButtonText="Cerrar"
  @close="closeTecnicoDetailsPopup"
>
  <template #popup-content>
    <div v-if="tecnicoDetailsData && tecnicoDetailsData.length > 0">
      <div class="table-responsive">
        <table class="table table-borderless">
          <thead>
            <tr>
              <th>Usuario</th>
              <th>Entrada</th>
              <th>Salida</th>
              <th>Tiempo Trabajado</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="detail in tecnicoDetailsData" :key="detail.id">
              <td>{{ detail.nombre_usuario }}</td>
              <td>{{ formatDateTime(detail.fecha_entrada) }}</td>
              <td>{{ detail.fecha_salida ? formatDateTime(detail.fecha_salida) : 'En curso' }}</td>
              <td>{{ detail.tiempo_trabajado ? formatTimeWorked(detail.tiempo_trabajado) : 'En curso' }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div v-else>
      <p>No hay detalles de técnicos para mostrar en esta incidencia.</p>
    </div>
  </template>
</GlassmorphicPopup>

<!-- Popup para motivo de cierre de incidencia -->


<GlassmorphicPopup  v-if="!isOperario"
:visible="showMotivoCierrePopup"
title="Motivo de Cierre de Incidencia"
closeButtonText="Cancelar"
actionButtonText="Confirmar Cierre"
@close="closeMotivoCierrePopup"
@action="handleCerrarIncidencia"
>
<template #popup-content>
<div class="form-group mb-3">
<label for="motivoCierre">Motivo:</label>
<textarea id="motivoCierre" v-model="motivoCierre" class="form-control custom-textarea" rows="3"
placeholder="Indique el motivo del cierre"></textarea>
</div>
</template>
</GlassmorphicPopup>

<!-- Popup de error genérico -->


<GlassmorphicPopup
:visible="showErrorPopup"
title="Error"
closeButtonText="Cerrar"
@close="closeErrorPopup"
>
<template #popup-content>
<p>{{ errorMessage }}</p>
</template>
</GlassmorphicPopup>

<!-- Popup para motivo de salida de incidencia -->


<GlassmorphicPopup  v-if="!isOperario"
:visible="showMotivoSalidaPopup"
title="Motivo de Salida de Incidencia"
closeButtonText="Cancelar"
actionButtonText="Confirmar Salida"
@close="closeMotivoSalidaPopup"
@action="handleSalirIncidencia"
>
<template #popup-content>
<div class="form-group mb-3">
<label for="motivoSalida">Motivo:</label>
<textarea id="motivoSalida" v-model="motivoSalida" class="form-control custom-textarea" rows="3"
placeholder="Indique el motivo de su salida"></textarea>
</div>
</template>
</GlassmorphicPopup>

<!-- Popup para filtros de incidencias -->


<GlassmorphicPopup
:visible="showFiltroPopup"
title="Filtros incidencias"
closeButtonText="Cancelar"
actionButtonText="Aplicar"
@close="cancelFilters"
@action="applyFilters"
>
<template #popup-content>
<div class="row">
<div class="col">
<div class="search-bar">
<CustomSelect
:options="campusOptions"
v-model="selectedCampusTemp"
placeholder="Filtrar por campus"
class="glassmorphic-select"
/>
<CustomSelect
:options="filteredSeccionOptions"
v-model="selectedSeccionTemp"
placeholder="Filtrar por seccion"
class="glassmorphic-select"
/>
</div>
</div>
</div>
</template>
</GlassmorphicPopup>

  </div>
</template>
<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import axios from 'axios';
import GlassmorphicPopup from './GlassmorphicPopup.vue';
import CustomSelect from './CustomSelect.vue';
import CustomInput from './CustomInput.vue';
import { useToast } from "vue-toastification";
import moment from 'moment';

// Estados reactivos
const loading = ref(true); // Indica si se están cargando datos
const error = ref(null); // Almacena errores de carga
const incidencias = ref([]); // Listado completo de incidencias para dashboard
const incidenciasPanel = ref([]); // Listado de incidencias para el panel principal
const tiposIncidencia = ref([]); // Tipos de incidencia para el selector
const maquinas = ref([]); // Máquinas para el selector
const tecnicoIncidencias = ref([]); // Listado de técnicos y sus incidencias activas
const API_AUTH_URL = import.meta.env.VITE_API_AUTH_URL; // URL base de la API
const ALL_INCIDENCIAS_URL = `${API_AUTH_URL}/incidencia/all`; // URL para todas las incidencias
const ALL_INCIDENCIAS_URL_CERRADAS = `${API_AUTH_URL}/incidencia/all_cerradas`; // URL para incidencias cerradas
const ALL_TIPO_INCIDENCIAS_URL = `${API_AUTH_URL}/tipo_incidencia/all`; // URL para tipos de incidencia
const ALL_MAQUINAS_URL = `${API_AUTH_URL}/maquina/all`; // URL para máquinas
const ALL_TECNICO_INCIDENCIA_URL = `${API_AUTH_URL}/tecnico_incidencia/all`; // URL para técnicos de incidencia
const ME_URL = `${API_AUTH_URL}/auth/me`; // URL para obtener datos del usuario actual
const CAMPUS_ALL_URL = `${API_AUTH_URL}/campus/all`; // URL para obtener todos los campus
const SECCION_ALL_URL = `${API_AUTH_URL}/seccion/all`; // URL para obtener todas las secciones
const userPicture = ref(null); // Foto del usuario
const userId = ref(null); // ID del usuario
const userRole = ref(null); // Rol del usuario
const prioridadSeleccionada = ref(null); // Prioridad seleccionada para filtrar
const seleccionarPrioridad = (prioridad) => { // Función para seleccionar la prioridad de filtrado
prioridadSeleccionada.value = prioridad;
};
const isTecnico = computed(() => userRole.value === 'tecnico'); // Computed para verificar si el usuario es técnico
const isAdmin = computed(() => userRole.value === 'administrador'); // Computed para verificar si el usuario es administrador
const isOperario = computed(() => userRole.value === 'operario'); // Computed para verificar si el usuario es operario
const userImagePath = computed(() => { // Computed para la ruta de la imagen del usuario (no se usa en el template proporcionado)
return userPicture ? `${IMAGE_URL}${userPicture}` : null;
});
const showMotivoSalidaPopup = ref(false); // Controla la visibilidad del popup de motivo de salida
const motivoSalida = ref(''); // Motivo de salida de incidencia
const showCreateIncidenciaPopup = ref(false); // Controla la visibilidad del popup de crear incidencia
const showFiltroPopup = ref(false); // Controla la visibilidad del popup de filtros
const guardandoComentario = ref(false); // Indica si se está guardando un comentario
const newIncidencia = ref({ // Objeto para almacenar los datos de la nueva incidencia
titulo: '',
subtitulo: '',
id_tipo_incidencia: null,
id_maquina: null,
});
const reclamandoIncidencia = ref(false); // Indica si se está reclamando una incidencia
const showMotivoCierrePopup = ref(false); // Controla la visibilidad del popup de motivo de cierre
const motivoCierre = ref(''); // Motivo de cierre de incidencia
const debouncedUpdateDescription = ref(null); // Referencia para limpiar el debounce del comentario
const hasReclamada = ref(false); // Indica si el técnico ya tiene una incidencia reclamada activa
const showErrorPopup = ref(false); // Controla la visibilidad del popup de error
const errorMessage = ref(''); // Mensaje de error para el popup
const isSoloTecnico = computed(()=>{ // Computed para verificar si es el único técnico activo en la incidencia
if(selectedIncidencia.value && (isTecnico.value || isAdmin.value)){
const count = tecnicoIncidencias.value.filter(incidencia => incidencia.id_incidencia === selectedIncidencia.value.id && incidencia.estado_tecnico === 'activo').length;
return count === 1;
}
return false
})

// Nuevo estado para el popup de detalles técnicos
const showTecnicoDetailsPopup = ref(false);
const tecnicoDetailsData = ref([]);

// Toast instance
const toast = useToast();

// Default toast options (can be reused and customized)
const defaultToastOptions = {
position: "top-right",
timeout: 5000,
closeOnClick: true,
pauseOnFocusLoss: true,
pauseOnHover: true,
draggable: true,
draggablePercent: 0.6,
showCloseButtonOnHover: false,
hideProgressBar: true,
closeButton: "button",
icon: true,
rtl: false,
toastClassName: 'glassmorphic-toast',
bodyClassName: 'glassmorphic-toast-body',
style: {
background: "transparent",
},
};

// Helper functions to create toasts with default options
const showSuccessToast = (message, options = {}) => {
toast.success(message, { ...defaultToastOptions, ...options });
};

const showErrorToast = (message, options = {}) => {
toast.error(message, { ...defaultToastOptions, ...options });
};

const showWarningToast = (message, options = {}) => {
toast.warning(message, { ...defaultToastOptions, ...options });
};

const showInfoToast = (message, options = {}) => {
toast.info(message, { ...defaultToastOptions, ...options });
};

// Funciones de formateo y utilidad
const obtenerPrioridad = (prioridad) => { // Función para obtener la prioridad en formato legible
if (["alta", "media", "baja"].includes(prioridad)) {
return prioridad;
} else {
return "baja";
}
};

const obtenerEstado = (estado) => { // Función para obtener el estado en formato legible
switch (estado) {
case 0: return 'Nueva';
case 1: return 'Pendiente';
case 2: return 'En curso';
case 3: return 'Cerrada';
case 4: return 'Mantenimiento';
default: return '';
}
};

const formatDate = (dateString) => { // Función para formatear la fecha
const date = new Date(dateString);
const options = { year: 'numeric', month: 'long', day: 'numeric' };
return date.toLocaleDateString(undefined, options);
};

const formatTime = (dateString) => { // Función para formatear la hora
const date = new Date(dateString);
const options = { hour: '2-digit', minute: '2-digit' };
return date.toLocaleTimeString(undefined, options);
};

const formatDateTime = (dateString) => {
  if (!dateString) return '-';
  return moment(dateString).format('YYYY-MM-DD HH:mm');
};

const formatTimeWorked = (minutes) => {
  if (!minutes) return '-';
  const hours = Math.floor(minutes / 60);
  const mins = minutes % 60;
  return `${hours}h ${mins}m`;
};


const fetchData = async (url) => { // Función genérica para obtener datos de la API
const token = localStorage.getItem('token');
if (!token) throw new Error('No token found');
try {
const response = await axios.post(url, {}, { headers: { Authorization: `Bearer ${token}` } });
return response.data;
} catch (err) {
console.error('Error al obtener los datos de las incidencias:', err);
throw new Error('Error al obtener los datos de las incidencias');
}
};

const fetchTecnicoIncidencias = async () => { // Función para obtener los técnicos de las incidencias
try {
if(!isOperario.value){
const token = localStorage.getItem('token');
if (!token) throw new Error('No token found');
const response = await axios.get(ALL_TECNICO_INCIDENCIA_URL, { headers: { Authorization: `Bearer ${token}` } });
tecnicoIncidencias.value = response.data.data;
}
} catch (error) {
console.error('Error al obtener los técnicos de las incidencias:', error);
throw new Error('Error al obtener los técnicos de las incidencias');
}
};

const fetchTipoIncidencias = async () => { // Función para obtener los tipos de incidencia
try {
const token = localStorage.getItem('token');
if (!token) throw new Error('No token found');
const response = await axios.get(ALL_TIPO_INCIDENCIAS_URL, { headers: { Authorization: `Bearer ${token}` } });
return response.data;
} catch (error) {
console.error('Error al obtener los tipos de incidencia:', error);
throw new Error('Error al obtener los tipos de incidencia');
}
};

const fetchMaquinas = async () => { // Función para obtener las máquinas
try {
const token = localStorage.getItem('token');
if (!token) throw new Error('No token found');
const response = await axios.post(ALL_MAQUINAS_URL, {}, { headers: { Authorization: `Bearer ${token}` } });
return response.data.data || [];
} catch (error) {
console.error('Error al obtener las maquinas:', error);
throw new Error('Error al obtener las maquinas');
}
};

const fetchTecnicoIncidenciaDetails = async (incidenciaId) => {
  try {
    const token = localStorage.getItem('token');
    if (!token) throw new Error('No token found');
    const response = await axios.get(`${API_AUTH_URL}/tecnico_incidencia/detalle_incidencia_tecnicos/${incidenciaId}`, { headers: { Authorization: `Bearer ${token}` } });
    tecnicoDetailsData.value = response.data.data;
  } catch (error) {
    console.error('Error al obtener los detalles de técnicos de la incidencia:', error);
    showErrorToast('Error al obtener los detalles de técnicos.');
  }
};


// Computed properties para contar incidencias por estado
const incidenciasPendientesCount = computed(() => incidencias.value.filter((incidencia) => ['Pendiente', 'Nueva'].includes(obtenerEstado(incidencia.estado))).length);
const incidenciasEnCursoCount = computed(() => incidencias.value.filter((incidencia) => obtenerEstado(incidencia.estado) === 'En curso').length);
const incidenciasCerradasCount = computed(() => incidencias.value.filter((incidencia) => obtenerEstado(incidencia.estado) === 'Cerrada').length);

const loadIncidencias = async (url) => { // Función para cargar incidencias desde una URL dada
try {
loading.value = true;
const incidenciasData = await fetchData(url);
return incidenciasData.map(incidencia => ({
...incidencia,
id: incidencia.id,
priority: obtenerPrioridad(incidencia.prioridad),
status: obtenerEstado(incidencia.estado),
date: formatDate(incidencia.fecha_apertura),
time: formatTime(incidencia.fecha_apertura),
descripcion: incidencia.descripcion,
id_tecnico: incidencia.id_mantenimiento
}));
} catch (err) {
error.value = err;
throw new Error(err);
} finally {
loading.value = false;
}
}
const isUserActiveInIncidencia = computed(() => { // Computed para verificar si el usuario es técnico activo en la incidencia seleccionada
if (!selectedIncidencia.value || !userId.value) return false;
return tecnicoIncidencias.value.some(item =>
item.id_incidencia === selectedIncidencia.value.id &&
item.id_tecnico === Number(userId.value) &&
item.estado_tecnico === 'activo'
);
});

const loadIncidenciasDashboard = async () => { // Función para cargar incidencias para el dashboard (cerradas)
try {
incidencias.value = await loadIncidencias(ALL_INCIDENCIAS_URL_CERRADAS)
} catch (err){
error.value = err;
}
};

const loadIncidenciasPanel = async () => { // Función para cargar incidencias para el panel principal (todas)
try {
incidenciasPanel.value = await loadIncidencias(ALL_INCIDENCIAS_URL)
}catch(err){
error.value = err;
}
};

const fetchUserData = async () => { // Función para obtener datos del usuario
const token = localStorage.getItem('token');
if (token) {
try {
const response = await axios.get(ME_URL, { headers: { Authorization: `Bearer ${token}` } });
userRole.value = response.data.rol;
userPicture.value = response.data.picture;
userId.value = response.data.id
localStorage.setItem('id', response.data.id);
} catch (error) {
console.error('Error fetching user data:', error);
}
}
};

const checkUserHasReclamada = () => { // Función para verificar si el técnico tiene una incidencia reclamada activa
if(isTecnico.value || isAdmin.value){
hasReclamada.value = tecnicoIncidencias.value.some(incidencia => incidencia.id_tecnico === Number(userId.value) && incidencia.estado_tecnico === 'activo')
}
}

// Filtros y búsquedas
const searchQuery = ref(''); // Texto de búsqueda
const selectedSeccion = ref(null); // Sección seleccionada para filtrar
const seccionOptions = ref([]); // Opciones de sección para el filtro
const selectedCampus = ref(null); // Campus seleccionado para filtrar
const campusOptions = ref([]); // Opciones de campus para el filtro

const filteredIncidencia = computed(() => { // Computed para filtrar incidencias por búsqueda, sección y campus
if (!incidenciasPanel.value) return [];
let filtered = [...incidenciasPanel.value];
if (searchQuery.value) {
const searchTerm = searchQuery.value.toLowerCase();
filtered = filtered.filter(incidencia => `${incidencia.titulo}`.toLowerCase().includes(searchTerm));
}
if (selectedSeccion.value) {
filtered = filtered.filter(incidencia => {
const maquina = maquinas.value.find(maquina => maquina.id === incidencia.id_maquina);
return maquina?.id_seccion === selectedSeccion.value;
});
}
if (selectedCampus.value) {
filtered = filtered.filter(incidencia => {
const maquina = maquinas.value.find(maquina => maquina.id === incidencia.id_maquina);
const seccion = seccionOptions.value.find(seccion => seccion.id === maquina?.id_seccion);
return seccion?.id_campus === selectedCampus.value;
});
}
return filtered;
});

const filteredIncidenciaPrioridad = computed(() => { // Computed para filtrar incidencias por prioridad
let filtered = filteredIncidencia.value;
if (prioridadSeleccionada.value) {
filtered = filtered.filter(incidencia => incidencia.priority === prioridadSeleccionada.value);
}
return filtered;
});

const filteredSeccionOptions = computed(() => { // Computed para filtrar opciones de sección por campus seleccionado
return selectedCampus.value ? seccionOptions.value.filter(seccion => seccion.id_campus === selectedCampus.value) : seccionOptions.value;
});

// Variables temporales para filtros antes de aplicar
const searchQueryTemp = ref('');
const selectedSeccionTemp = ref(null);
const selectedCampusTemp = ref(null);

const cancelFilters = () => { // Función para cancelar filtros y cerrar el popup
searchQueryTemp.value = "";
selectedSeccionTemp.value = null;
selectedCampusTemp.value = null;
closeFiltroPopup();
};

const applyFilters = () => { // Función para aplicar filtros y cerrar el popup
searchQuery.value = searchQueryTemp.value;
selectedSeccion.value = selectedSeccionTemp.value;
selectedCampus.value = selectedCampusTemp.value;
closeFiltroPopup();
};

// Ciclo de vida y watchers
onMounted(async () => { // Hook onMounted para cargar datos al montar el componente
try {
const token = localStorage.getItem('token');
const headers = { Authorization: `Bearer ${token}` };
await fetchUserData()
const [tiposData, maquinasData] = await Promise.all([
fetchTipoIncidencias(),
fetchMaquinas(),
]);

tiposIncidencia.value = tiposData.map(tipo => ({ id: tipo.id, label: tipo.tipo }));
    maquinas.value = maquinasData.map(maquina => ({ id: maquina.id, label: maquina.nombre_maquina, id_seccion: maquina.id_seccion }));

    const response = await axios.get(SECCION_ALL_URL, { headers });
      seccionOptions.value = response.data.data.map(seccion => ({ id: seccion.id, label: seccion.nombre_seccion, id_campus: seccion.id_campus }));

    const campusResponse = await axios.get(CAMPUS_ALL_URL, { headers });
    campusOptions.value = campusResponse.data.data.map(campus => ({ id: campus.id, label: campus.nombre_campus }));

    await Promise.all([
        loadIncidenciasDashboard(),
        loadIncidenciasPanel(),
        fetchTecnicoIncidencias()
    ]);
    checkUserHasReclamada();
} catch (err) {
    error.value = err;
    loading.value = false;
}


});

watch(incidencias, checkUserHasReclamada); // Watcher para verificar incidencias reclamadas al cambiar la lista de incidencias

// Funciones para controlar popups
const openCreateIncidenciaPopup = () => { showCreateIncidenciaPopup.value = true; };
const openFiltroPopup = () => {
showFiltroPopup.value = true;
selectedCampusTemp.value = selectedCampus.value;
selectedSeccionTemp.value = selectedSeccion.value;
}
const closeFiltroPopup = () => { showFiltroPopup.value = false; };
const closeCreateIncidenciaPopup = () => {
showCreateIncidenciaPopup.value = false;
newIncidencia.value = { titulo: '', subtitulo: '', id_tipo_incidencia: null, id_maquina: null };
};

const validateIncidenciaData = () => { // Función para validar datos antes de crear incidencia
if (!newIncidencia.value.titulo || !newIncidencia.value.subtitulo || !newIncidencia.value.id_tipo_incidencia || !newIncidencia.value.id_maquina) {
alert('Por favor, complete el título, el subtítulo, el tipo de incidencia y la máquina.');
return false;
}
return true;
};

const handleCreateIncidencia = async () => { // Función para crear una nueva incidencia
if (!validateIncidenciaData()) return;
const token = localStorage.getItem('token');
if (!token) throw new Error('No token found');
try {
const apiUrl = `${API_AUTH_URL}/incidencia/store`;
const requestData = {
titulo: newIncidencia.value.titulo,
subtitulo: newIncidencia.value.subtitulo,
estado: 0,
id_maquina: Number(newIncidencia.value.id_maquina),
tipo_incidencia: Number(newIncidencia.value.id_tipo_incidencia),
};
const response = await axios.post(apiUrl, requestData, { headers: { Authorization: `Bearer ${token}` } });

if (response.status === 201) {
        closeCreateIncidenciaPopup();
        await Promise.all([ loadIncidenciasDashboard(), loadIncidenciasPanel() ]);
        showSuccessToast("Incidencia creada con éxito!"); // Show toast on successful creation
    }
} catch (error) {
    console.error('Error al crear la incidencia:', error);
    showErrorToast("Error al crear la incidencia.");
}


};

const showIncidenciaDetailsPopup = ref(false); // Controla la visibilidad del popup de detalles de incidencia
const selectedIncidencia = ref(null); // Incidencia seleccionada para mostrar detalles
const commentText = ref(''); // Texto del comentario

const handleIncidenciaClick = (incidencia) => { // Función para manejar el click en una incidencia y mostrar el popup de detalles
selectedIncidencia.value = incidencia;
commentText.value = incidencia.descripcion || '';
showIncidenciaDetailsPopup.value = true;
};

const closeIncidenciaDetailsPopup = () => { // Función para cerrar el popup de detalles de incidencia
showIncidenciaDetailsPopup.value = false;
selectedIncidencia.value = null;
commentText.value = '';
motivoSalida.value = '';
motivoCierre.value = '';
closeTecnicoDetailsPopup(); // Asegurar que el popup de detalles técnicos también se cierre
};

const handleCommentChange = () => { // Función para manejar cambios en el comentario con debounce
if (debouncedUpdateDescription.value) clearTimeout(debouncedUpdateDescription.value);
debouncedUpdateDescription.value = setTimeout(async () => { await handleUpdateDescription(); }, 1000);
};

const handleUpdateDescription = async () => { // Función para actualizar la descripción de la incidencia (comentario)
if (!selectedIncidencia.value) return;
guardandoComentario.value = true;
const token = localStorage.getItem('token');
if (!token) throw new Error('No token found');
try {
const apiUrl = `${API_AUTH_URL}/incidencia/update_description/${selectedIncidencia.value.id}`;
const requestData = { descripcion: commentText.value };
const response = await axios.put(apiUrl,requestData, { headers: { Authorization: `Bearer ${token}` } });
if (response.status === 200) selectedIncidencia.value.descripcion = commentText.value;
} catch (error) {
console.error("Error al actualizar el comentario:", error);
showErrorToast("Error al actualizar el comentario.");
} finally {
guardandoComentario.value = false;
}
};

const handleReclamarIncidencia = async () => { // Función para reclamar una incidencia por un técnico
if(!selectedIncidencia.value) return;
reclamandoIncidencia.value = true;
const token = localStorage.getItem('token');
if(!token) throw new Error('No token found');
try{
const apiUrl = `${API_AUTH_URL}/tecnico_incidencia/reclamar_incidencia_multiple`;
const requestData = { id_incidencia: selectedIncidencia.value.id };
const response = await axios.post(apiUrl,requestData,{ headers:{ Authorization: `Bearer ${token}` } });
if(response.status === 200 || response.status === 201){
await Promise.all([ loadIncidenciasDashboard(), loadIncidenciasPanel(), fetchTecnicoIncidencias() ]);
if(selectedIncidencia.value.status !== 'En curso') selectedIncidencia.value.status = 'En curso';
showSuccessToast("Incidencia reclamada!"); // Show toast on successful claim
} else {
if (response.status === 400) {
errorMessage.value = 'No puedes reclamar más de una incidencia a la vez.';
showErrorPopup.value = true;
} else {
console.error('Error al reclamar la incidencia, respuesta no exitosa', response);
showErrorToast("Error al reclamar la incidencia: " + (response.data.message || "Compruebe la consola para más información"));
}
}
}catch(error){
if (error.message.includes('400')) {
errorMessage.value = 'No puedes reclamar más de una incidencia a la vez.';
showErrorPopup.value = true;
} else{
console.error('Error al reclamar la incidencia:', error);
showErrorToast("Error al reclamar la incidencia: " + (error.message || "Compruebe la consola para más información"));
}
}finally{
reclamandoIncidencia.value = false;
}
}

const closeErrorPopup = () =>{ // Función para cerrar el popup de error
showErrorPopup.value = false;
errorMessage.value = '';
}

const openMotivoSalidaPopup = () => { showMotivoSalidaPopup.value = true; }; // Función para abrir el popup de motivo de salida
const closeMotivoSalidaPopup = () => { // Función para cerrar el popup de motivo de salida
showMotivoSalidaPopup.value = false;
motivoSalida.value = '';
};

const handleSalirIncidencia = async () => { // Función para que un técnico salga de una incidencia
if(!selectedIncidencia.value) return;
const token = localStorage.getItem('token');
if(!token) throw new Error('No token found');
try{
const apiUrl = `${API_AUTH_URL}/tecnico_incidencia/salir_incidencia`;
const requestData = { id_incidencia: selectedIncidencia.value.id, motivo_salida: motivoSalida.value };
const response = await axios.put(apiUrl,requestData,{ headers:{ Authorization: `Bearer ${token}` } });
if(response.status === 200){
closeMotivoSalidaPopup();
closeIncidenciaDetailsPopup();
await Promise.all([ loadIncidenciasDashboard(), loadIncidenciasPanel(), fetchTecnicoIncidencias() ]); // Añadimos fetchTecnicoIncidencias() aquí
showSuccessToast("Saliste de la incidencia!"); // Show toast on successful leave
}
}catch (error){
console.error('Error al salir de la incidencia:',error);
showErrorToast('Error al salir de la incidencia.');
}
}

const openMotivoCierrePopup = () => { showMotivoCierrePopup.value = true; }; // Función para abrir el popup de motivo de cierre
const closeMotivoCierrePopup = () => { // Función para cerrar el popup de motivo de cierre
showMotivoCierrePopup.value = false;
motivoCierre.value = '';
};

const handleCerrarIncidencia = async () => { // Función para cerrar una incidencia por un técnico
if(!selectedIncidencia.value) return;
const token = localStorage.getItem('token');
if(!token) throw new Error('No token found');
try{
const apiUrl = `${API_AUTH_URL}/tecnico_incidencia/cerrar_incidencia`;
const requestData = { id_incidencia: selectedIncidencia.value.id, motivo_salida: motivoCierre.value };
const response = await axios.put(apiUrl,requestData,{ headers:{ Authorization: `Bearer ${token}` } });
if(response.status === 200){
await  Promise.all([ loadIncidenciasDashboard(), loadIncidenciasPanel() ]);
selectedIncidencia.value.status = 'Cerrada';
closeMotivoCierrePopup();
closeIncidenciaDetailsPopup();
showSuccessToast("Incidencia cerrada!"); // Show toast on successful close
}
}catch (error){
console.error('Error al cerrar la incidencia', error);
showErrorToast('Error al cerrar la incidencia.');
}
}

// Funciones para el nuevo popup de detalles técnicos
const openTecnicoDetailsPopup = async () => {
  if (selectedIncidencia.value) {
    await fetchTecnicoIncidenciaDetails(selectedIncidencia.value.id);
    showTecnicoDetailsPopup.value = true;
  }
};
const closeTecnicoDetailsPopup = () => {
  showTecnicoDetailsPopup.value = false;
  tecnicoDetailsData.value = [];
};
</script>

<style lang="scss" scoped>
// Estilos CSS (sin cambios, se asume que ya están optimizados y funcionales)
// ... (mismo código CSS proporcionado anteriormente) ...
$color-white: #fff;
$color-light-gray: #f8f8f8;
$color-gray: #ddd;
$color-dark-gray: #666;
$color-shadow: rgba(0, 0, 0, 0.1);
$color-shadow-hover: rgba(0, 0, 0, 0.2);
$color-high-priority: #FF5252;
$color-medium-priority: #FFCA28;
$color-low-priority: #4CAF50;
$color-new: #B89B00;
$color-pending: #B89B00;
$color-maintenance: #000000;
$color-closed: #000000;
$color-in-progress: #600484;
$color-primary: rgba(0, 123, 255, 0.9);
$color-danger: rgba(255, 0, 0, 0.8);
$color-text:#333;


// Estilos base para incidencias
$incidencia-item-padding: 15px;
$incidencia-item-margin-bottom: 10px;
$incidencia-item-border-radius: 8px;
$incidencia-item-gap: 15px;
$incidencia-date-min-width: 100px;
$incidencia-marker-width: 5px;
$incidencia-marker-height: 40px;
$incidencia-marker-margin-right: 5px;
$incidencia-text-margin-left: 10px;
$incidencia-status-padding: 6px 10px;
$incidencia-status-border-radius: 20px;

@mixin flex-center($justify: center, $align: center, $direction: row){
    display: flex;
    justify-content: $justify;
    align-items: $align;
    flex-direction: $direction;
}

@mixin glassmorphic-card(){
    background: rgba($color-white, 0.7) !important;
    backdrop-filter: blur(8px);
    border: 1px solid rgba($color-white, 0.05) !important;
    box-shadow: 0 4px 24px 0 rgba(0, 0, 0, 0.3);
}


.incidencia-panel {
  width: 100%;
  padding: 20px;
  background-color: $color-light-gray;
  border-radius: 10px;
  box-shadow: 0 2px 4px $color-shadow;
}

.incidencia-header {
    @include flex-center(space-between, center);
    margin-bottom: 20px;
}

.create-incidencia-btn {
    @include flex-center(center, center);
  background-color: $color-white;
  border: 1px solid $color-gray;
  border-radius: 5px;
  padding: 12px 20px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 500;
  gap: 8px;

  &:hover {
    background-color: #f0f0f0;
  }
}

.plus-icon {
  font-size: 1.5rem;
  margin-top: -4px;
}

.incidencia-stats {
  @include flex-center($justify: flex-start, $align: center);
  gap: 10px;
}

.incidencia-stat-box {
  background-color: $color-white;
    @include flex-center(center, center, column);
  padding: 15px;
  border-radius: 5px;
  min-width: 140px;
  gap: 5px;
  box-shadow: 0 2px 4px $color-shadow;

  &.active {
    background-color: #e3fae8;
    box-shadow: 0 0 10px $color-shadow-hover;
  }
}

.title {
  font-size: 0.8rem;
}

.count {
  font-size: 1.4rem;
  font-weight: bold;
}

.icon {
  font-size: 1.2rem;
}

.incidencia-list {
    @include glassmorphic-card();
  border-radius: 5px;
  padding: 20px;
  margin-top: 20px;
}

.incidencia-list-header {
    @include flex-center(space-between, center);
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  padding: 0 10px 15px 10px;
  margin-bottom: 15px;
  font-size: 0.8rem;
  color: $color-dark-gray;
}

.priority-legend {
    @include flex-center($justify: flex-start, $align: center);
  gap: 20px;

  span{
      margin-right: 10px;
  }
}

.priority-dot {
  display: inline-block;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  margin-right: 2px;
}

// Bucle para generar las clases de prioridad
$priorities: (alta: $color-high-priority, media: $color-medium-priority, baja: $color-low-priority);

@each $priority-name, $priority-color in $priorities {
  .#{$priority-name} {
    background-color: $priority-color;
  }
}


.incidencia-item {
    @include flex-center(flex-start, flex-start);
    padding: $incidencia-item-padding;
    margin-bottom: $incidencia-item-margin-bottom;
    border-radius: $incidencia-item-border-radius;
    transition: all 0.2s ease;
    gap: $incidencia-item-gap;
    cursor: pointer;

    &:hover {
        background-color: rgba($color-white, 0.1);
    }
}


.incidencia-date {
    @include flex-center(flex-start, flex-start, column);
    font-size: 0.8rem;
    color: $color-dark-gray;
    min-width: $incidencia-date-min-width;
    flex-shrink: 0;

    span:first-child {
        font-weight: 500;
    }
}

.incidencia-content {
    @include flex-center(flex-start, flex-start);
    gap: 20px;
    flex: 1;
}


.priority-marker {
    width: $incidencia-marker-width;
    height: $incidencia-marker-height;
    border-radius: 2px;
    flex-shrink: 0;
    margin-right: $incidencia-marker-margin-right;
}


.incidencia-text {
    @include flex-center(flex-start, flex-start, column);
    font-size: 0.9rem;
    line-height: 1.3;
    flex: 1;
    margin-left: $incidencia-text-margin-left;
}


.incidencia-status-box {
  @include flex-center();
}

.incidencia-status {
    padding: $incidencia-status-padding;
    border-radius: $incidencia-status-border-radius;
    font-size: 0.8rem;
    font-weight: 500;
}
// Bucle para generar las clases de estado
$estados: (nueva: $color-new, pendiente: $color-pending, mantenimiento: $color-maintenance, cerrada: $color-closed, "en-curso": $color-in-progress);
@each $estado-name, $estado-color in $estados{
    .#{$estado-name}{
        background-color: rgba($estado-color, 0.17);
        color: $estado-color;
    }
}
.create-incidencia-card {
    cursor: pointer;
    transition: all 0.3s;

    &:hover {
      transform: scale(1.02);
      box-shadow: 0 4px 16px 0 rgba(0, 0, 0, 0.7);
    }
  }

  .card-body-same-height {
    min-height: 100px;
      @include flex-center(center, center, column);
  }

  .glassmorphic-card {
     @include glassmorphic-card();
  }

  .pendiente-h2 {
    color: $color-pending;
    font-size: 70px;
  }

  .curso-h2 {
    color: $color-in-progress;
    font-size: 70px;
  }

  .cerrado-h2 {
    color: $color-maintenance;
    font-size: 70px;
  }
  .total-h2{
    color: $color-white;
    font-size: 70px;
}

  .text-muted {
    color: rgba(54, 54, 54, 0.6) !important;
  }

  .h3 {
    color: rgba($color-white, 0.9);
  }

  table {
    background-color: transparent !important;
  }

  .table-responsive {
    max-height: calc(100vh - 300px);
    overflow-y: auto;
    background-color: transparent !important;
  }

  .table > :not(caption) > * > * {
    background-color: transparent !important;
  }

  canvas {
    max-height: 300px;
  }

  .status-dot {
    display: inline-block;
    width: 10px;
    height: 10px;
    border-radius: 50%;
  }

  .card.h-100 {
    height: calc(100vh - 200px) !important;
  }

  .card-body {
    padding: 1.5rem;
  }
    .card-bodytotal {
      padding: 1.7rem;
    }

  .incidencias-container {
    max-height: 600px;
    overflow-y: auto;
    padding: 0 10px;
  }

  /* Add smooth scrollbar for the tickets container */
  .incidencias-container::-webkit-scrollbar {
    width: 8px;
  }

  .incidencias-container::-webkit-scrollbar-track {
    background: rgba($color-white, 0.1);
    border-radius: 4px;
  }

  .incidencias-container::-webkit-scrollbar-thumb {
    background: rgba(0, 0, 0, 0.2);
    border-radius: 4px;
  }

  .incidencias-container::-webkit-scrollbar-thumb:hover {
    background: rgba(0, 0, 0, 0.3);
  }

  .form-group {
    margin-bottom: 1rem;
  }

  .form-group label {
    display: block;
    margin-bottom: 0.5rem;
    color: $color-dark-gray;
  }

  .popup-btn1 {
    background: $color-primary;
    color: $color-white;
    border: none;
    text-align: center;
    font-size: 1rem;
    padding: 0.8rem 1.8rem;
    border-radius: 2rem;
    cursor: pointer;
  }

  .popup-btn.primary {
    background-color: rgba($color-white, 0.8);
    color: #000000;
  }
    .popup-btn.danger {
        background-color: $color-danger;
        color: $color-white;
    }

  .popup-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.4);
     @include flex-center();
    z-index: 1000;
  }

  .popup-card {
    width: 700px;
    min-height: 300px;
      @include glassmorphic-card();
    box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
    border-radius: 1.5rem;
    padding: 2rem;
    z-index: 10;
    color: $color-text;
    @include flex-center(flex-start, flex-start, column);
    overflow: hidden;
    /* Asegura que el contenido no se salga del borde */
  }

  .popup-title {
    font-size: 2.5rem;
    margin-bottom: 1rem;
    text-align: center;
    color: #222;
  }

  .popup-subtitle {
    font-size: 1.2rem;
    margin-bottom: 2rem;
    text-align: center;
    color: #444;
  }

  .popup-content {
    margin-bottom: 2rem;
      @include flex-center(flex-start, flex-start, column);
    overflow-y: auto;
    padding-right: 10px;
    gap: 1rem;
  }

  .popup-actions {
    @include flex-center();
    gap: 1rem;
    margin-top: 20px;
    flex-direction: column; // Stack rows vertically
    align-items: stretch; // Make rows take full width
  }

  .action-buttons-row {
    display: flex;
    justify-content: center; // Center buttons horizontally in each row
    gap: 0.5rem;
    margin-bottom: 0.5rem; // Add some space between rows
  }

  .popup-btn {
    background: none;
    border: none;
    text-align: center;
    font-size: 1rem;
    padding: 0.8rem 1.8rem;
    border-radius: 2rem;
    cursor: pointer;
  }

  .glassmorphic-btn {
    background: rgba($color-white, 0.2);
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    border: 1px solid rgba($color-white, 0.2);
    color: $color-white;
    transition: background-color 0.3s ease;
      &:hover {
          background: rgba($color-white, 0.3);
      }
  }


  .popup-btn.primary {
    background-color: rgba($color-white, 0.8);
    color: #000000;
  }

  .glassmorphic-btn.cancel-btn {
    color: #000;
    background: rgba($color-white, 0.8);
      &:hover {
          background: rgba($color-white, 0.9);
      }
    }


  .custom-textarea {
    width: 100%;
    padding: 10px;
    margin-top: 5px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box;
    font-size: 1rem;
    line-height: 1.5;
    transition: border-color 0.3s ease;
    background-color: rgba($color-white, 0.5);
    color: $color-text;
      &:focus {
          border-color: #007bff;
          outline: none;
          box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
      }
      &::placeholder {
          color: #999;
      }
  }
  .action-buttons {
      margin-top: 1rem;
      display: flex; // Asegura que es flex container
      justify-content: center; // Centra horizontalmente los elementos
      align-items: center;
      gap: 0.5rem;
      flex-direction: column; /* Stack rows vertically */
      align-items: stretch; /* Make rows take full width */
  }
  .action-buttons-row {
      display: flex;
      justify-content: center; /* Center buttons horizontally in each row */
      gap: 0.5rem;
      margin-bottom: 0.5rem; /* Add some space between rows */
  }
  :root {
    --color-primario: #007bff;
  }

  .priority-filter {
    cursor: pointer;
    position: relative;
    padding-bottom: 3px;
      &:hover {
          opacity: 0.8;
      }
      &.selected {
          text-decoration-color: var(--color-primario);
          text-decoration-thickness: 4px;
      }
  }


  .boton-filtro{
    cursor: pointer;
    position: relative;
    padding: 5px;
      @include flex-center();
    border-radius: 12px;
      &:hover {
          opacity: 0.8;
      }
  }


  .imagen-filtro{
    width: 20px;
  }


  .glassmorphic-select{
    background: rgba($color-white, 0.2);
    border: 1px solid rgba($color-white, 0.3);
    border-radius: 10px;
    padding: 10px;
    color: $color-text;
    width: 300px;
  }

  .search-bar {
    margin-bottom: 20px;
      @include flex-center();
    gap: 10px;
  }
  @media (max-width: 768px) {
    .row {
      --bs-gutter-x: 0.5rem !important;
    }
    .col-sm-6 {
      flex: 0 0 auto;
      width: 50% !important;
      max-width: 50% !important;
    }

    .incidencia-list {
      padding: 10px;
      margin-top: 10px;
    }

    .incidencia-list-header {
      font-size: 0.9rem;
      padding: 0 5px 10px 5px;
      margin-bottom: 10px;
    }

    .incidencia-item {
      padding: 10px;
      margin-bottom: 5px;
      gap: 10px;
    }

    .incidencia-date {
      min-width: 70px;
      font-size: 0.7rem;
    }

    .incidencia-content {
      gap: 10px;
    }

    .priority-marker {
      height: 30px;
      margin-right: 3px;
    }

    .incidencia-text {
      font-size: 0.8rem;
      margin-left: 5px;
    }

    .incidencia-status {
      padding: 4px 8px;
      font-size: 0.7rem;
    }

    .card-body {
      padding: 1rem;
    }
      .card-bodytotal {
          padding: 1rem;
      }
    .pendiente-h2,
    .curso-h2,
    .cerrado-h2,
        .total-h2 {
      font-size: 45px;
    }
    .action-buttons {
      flex-direction: row;
      flex-wrap: wrap;
    }
      .popup-card {
          width: 90%;
          min-height: auto;
          max-height: 80vh;
          padding: 1rem;
      }
      .popup-content{
          margin-bottom: 1rem;
      }

      .popup-title {
          font-size: 1.5rem;
          margin-bottom: 0.8rem;
      }

      .popup-subtitle {
          font-size: 1rem;
          margin-bottom: 1.5rem;
      }

      .popup-actions {
          margin-top: 10px;
          gap: 0.5rem;
        flex-wrap: wrap;
        justify-content: center;
      }
      .popup-btn, .popup-btn1 {
          font-size: 0.9rem;
          padding: 0.6rem 1.2rem;
          border-radius: 1.5rem;
          text-align: center;
      }
      .incidencias-container{
          padding: 0 5px;
      }
    .priority-legend {
      gap: 10px;
      font-size: 0.8rem;
    }
    .boton-filtro {
          padding: 3px;
      }
      .imagen-filtro{
          width: 15px;
      }

      .col-12 {
          width: 100% !important;
          max-width: 100% !important;
      }

  }

  .loading-container {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 10px;
    padding: 20px;
  }

  /* Glassmorphic toast styles */
  .glassmorphic-toast {
    @include glassmorphic-card();
    color: $color-text;
  }

  .glassmorphic-toast-body {
    padding: 15px;
  }
</style>