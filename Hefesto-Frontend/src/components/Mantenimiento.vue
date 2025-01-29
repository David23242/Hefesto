<template>
  <div class="row g-4 mb-4">
    <!-- Cabecera con paneles de resumen -->
    <div v-if="isAdmin" class="col-sm-6 col-md-3" @click="openCreateMantenimientoPopup">
      <div class="card glassmorphic-card colored-shadow-pending create-incidencia-card">
        <div
          class="card-bodytotal card-body-same-height d-flex justify-content-between align-items-center"
        >
          <h6 class="mb-0">Crear Mantenimiento</h6>
          <img src="../assets/images/icons/crear.svg" alt="Crear Mantenimiento" />
        </div>
      </div>
    </div>
    <div v-if="isAdmin" class="col-sm-6 col-md-3" @click="openAsignarMantenimientoPopup">
      <div class="card glassmorphic-card colored-shadow-pending create-incidencia-card">
        <div
          class="card-bodytotal card-body-same-height d-flex justify-content-between align-items-center"
        >
          <h6 class="mb-0">Asignar a Máquina</h6>
          <img src="../assets/images/icons/asignar.svg" alt="Asignar a Máquina" />
        </div>
      </div>
    </div>

    <!-- Listado de mantenimientos -->
    <div class="col-12">
      <div class="incidencia-list glassmorphic-card">
        <div class="incidencia-list-header">
          <div>Mantenimientos</div>
          <div class="priority-legend">
            <!-- Botón y leyenda de filtro -->
            <div class="boton-filtro" @click="openFiltroPopup">
              <img src="../assets/images/icons/filtro.svg" alt="Filtro" class="imagen-filtro" />
              Filtros
            </div>
            <!-- Filtros de prioridad -->
            <span
              @click="seleccionarPrioridad('alta')"
              :class="{ selected: prioridadSeleccionada === 'alta' }"
              class="priority-filter"
            >
              <span class="priority-dot alta"></span> Alta
            </span>
            <span
              @click="seleccionarPrioridad('media')"
              :class="{ selected: prioridadSeleccionada === 'media' }"
              class="priority-filter"
            >
              <span class="priority-dot media"></span> Media
            </span>
            <span
              @click="seleccionarPrioridad('baja')"
              :class="{ selected: prioridadSeleccionada === 'baja' }"
              class="priority-filter"
            >
              <span class="priority-dot baja"></span> Baja
            </span>
            <span
              @click="seleccionarPrioridad(null)"
              :class="{ selected: prioridadSeleccionada === null }"
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
          <span>Cargando mantenimientos...</span>
        </div>
        <div v-else-if="error">Error al cargar los mantenimientos.</div>

        <!-- Contenedor de mantenimientos -->
        <div v-else class="incidencias-container">
          <div
            v-for="incidencia in filteredIncidenciasPrioridad"
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

    <!-- Popup para crear mantenimiento -->
    <GlassmorphicPopup
      :visible="showCreateMantenimientoPopup"
      title="Crear Mantenimiento Preventivo"
      closeButtonText="Cancelar"
      actionButtonText="Crear Mantenimiento"
      @close="closeCreateMantenimientoPopup"
      @action="handleCreateMantenimiento"
    >
      <template #popup-content>
        <div class="form-group mb-3">
          <label for="nombre">Nombre:</label>
          <CustomInput placeholder="Ingrese el nombre" v-model="newMantenimiento.nombre" />
        </div>
        <div class="form-group mb-3">
          <label for="descripcion">Descripción:</label>
          <CustomInput
            placeholder="Ingrese la descripción"
            v-model="newMantenimiento.descripcion"
          />
        </div>
        <div class="form-group mb-3">
          <label for="periodicidad">Periodicidad (días):</label>
          <CustomInput
            type="number"
            placeholder="Ingrese la periodicidad"
            v-model="newMantenimiento.periodicidad"
          />
        </div>
      </template>
    </GlassmorphicPopup>

    <!-- Popup para asignar mantenimiento a máquina -->
    <GlassmorphicPopup
      :visible="showAsignarMantenimientoPopup"
      title="Asignar Mantenimiento a Máquina"
      closeButtonText="Cancelar"
      actionButtonText="Asignar"
      @close="closeAsignarMantenimientoPopup"
      @action="handleAsignarMantenimiento"
    >
      <template #popup-content>
        <div class="form-group mb-3">
          <label for="mantenimiento">Mantenimiento:</label>
          <CustomSelect
            :options="mantenimientosOptions"
            v-model="asignarMantenimiento.id_mantenimiento"
          />
        </div>
        <div class="form-group mb-3">
          <label for="maquina">Máquina:</label>
          <CustomSelect :options="maquinasOptions" v-model="asignarMantenimiento.id_maquina" />
        </div>
      </template>
    </GlassmorphicPopup>

    <!-- Popup para detalles de incidencia -->
    <GlassmorphicPopup
      :visible="showIncidenciaDetailsPopup"
      title="Detalles del Mantenimiento"
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
          <div class="action-buttons mt-auto">
            <button
              v-if="(isTecnico || isAdmin) && selectedIncidencia.status !== 'En curso' && !hasReclamada"
              class="popup-btn primary"
              @click="handleReclamarIncidencia"
              :disabled="reclamandoIncidencia"
            >
              {{ reclamandoIncidencia ? 'Reclamando...' : 'Reclamar Mantenimiento' }}
            </button>
            <div
              v-if="(isTecnico || isAdmin) && selectedIncidencia.status === 'En curso'"
              class="d-flex justify-content-center gap-2"
            >
              <button class="popup-btn1 cancel-btn" @click="openMotivoSalidaPopup">
                Salir de Mantenimiento
              </button>
              <button class="popup-btn primary" @click="openMotivoCierrePopup">
                Cerrar Mantenimiento
              </button>
            </div>
             <!-- Botón para ver detalles de técnicos - Añadido aquí -->
            <button v-if="(isTecnico || isAdmin)" class="popup-btn primary mt-3" @click="openTecnicoDetailsPopup">Detalle Técnicos</button>
          </div>
        </div>
        <div v-else>
          <p>No hay detalles de mantenimiento para mostrar</p>
        </div>
      </template>
    </GlassmorphicPopup>

    <!-- Popup de detalles técnicos en el mantenimiento - Nuevo Popup -->
    <GlassmorphicPopup
      :visible="showTecnicoDetailsPopup"
      title="Detalle de Técnicos en Mantenimiento"
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
          <p>No hay detalles de técnicos para mostrar en este mantenimiento.</p>
        </div>
      </template>
    </GlassmorphicPopup>

    <!-- Popup de error genérico -->
    <GlassmorphicPopup :visible="showErrorPopup" title="Error" closeButtonText="Cerrar" @close="closeErrorPopup">
      <template #popup-content>
        <p>{{ errorMessage }}</p>
      </template>
    </GlassmorphicPopup>

    <!-- Popup para motivo de cierre de mantenimiento -->
    <GlassmorphicPopup
      :visible="showMotivoCierrePopup"
      title="Motivo de Cierre de Mantenimiento"
      closeButtonText="Cancelar"
      actionButtonText="Confirmar Cierre"
      @close="closeMotivoCierrePopup"
      @action="handleCerrarIncidencia"
    >
      <template #popup-content>
        <div class="form-group mb-3">
          <label for="motivoCierre">Motivo:</label>
          <textarea
            id="motivoCierre"
            v-model="motivoCierre"
            class="form-control custom-textarea"
            rows="3"
            placeholder="Indique el motivo del cierre"
          ></textarea>
        </div>
      </template>
    </GlassmorphicPopup>

    <!-- Popup para motivo de salida de mantenimiento -->
    <GlassmorphicPopup
      :visible="showMotivoSalidaPopup"
      title="Motivo de Salida de Mantenimiento"
      closeButtonText="Cancelar"
      actionButtonText="Confirmar Salida"
      @close="closeMotivoSalidaPopup"
      @action="handleSalirIncidencia"
    >
      <template #popup-content>
        <div class="form-group mb-3">
          <label for="motivoSalida">Motivo:</label>
          <textarea
            id="motivoSalida"
            v-model="motivoSalida"
            class="form-control custom-textarea"
            rows="3"
            placeholder="Indique el motivo de su salida"
          ></textarea>
        </div>
      </template>
    </GlassmorphicPopup>

    <!-- Popup para filtros de mantenimientos -->
    <GlassmorphicPopup
      :visible="showFiltroPopup"
      title="Filtros mantenimientos"
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
import { useToast } from 'vue-toastification';
import moment from 'moment';

// Estados reactivos
const loading = ref(true);
const error = ref(null);
const incidencias = ref([]);
const tiposIncidencia = ref([]);
const maquinas = ref([]);
const tecnicoIncidencias = ref([]);
const API_AUTH_URL = import.meta.env.VITE_API_AUTH_URL;
const ALL_INCIDENCIAS_URL = `${API_AUTH_URL}/incidencia/all_mantenimientos`;
const ALL_TIPO_INCIDENCIAS_URL = `${API_AUTH_URL}/tipo_incidencia/all`;
const ALL_MAQUINAS_URL = `${API_AUTH_URL}/maquina/all`;
const ALL_TECNICO_INCIDENCIA_URL = `${API_AUTH_URL}/tecnico_incidencia/all`;
const ME_URL = `${API_AUTH_URL}/auth/me`;
const CAMPUS_ALL_URL = `${API_AUTH_URL}/campus/all`;
const SECCION_ALL_URL = `${API_AUTH_URL}/seccion/all`;
const TECNICO_INCIDENCIA_DETALLES_URL = `${API_AUTH_URL}/tecnico_incidencia/detalle_incidencia_tecnicos`; // Nueva URL para detalles tecnicos
const userPicture = localStorage.getItem('picture');
const userId = localStorage.getItem('id');
const userRole = ref(null);
const prioridadSeleccionada = ref(null);
const isTecnico = computed(() => userRole.value === 'tecnico');
const isAdmin = computed(() => userRole.value === 'administrador');
const isOperario = computed(() => userRole.value === 'operario');
const isTecnicoOrAdmin = computed(() => isTecnico.value || isAdmin.value);
const showMotivoSalidaPopup = ref(false);
const motivoSalida = ref('');
const showCreateMantenimientoPopup = ref(false);
const showAsignarMantenimientoPopup = ref(false);
const guardandoComentario = ref(false);
const newMantenimiento = ref({ nombre: '', descripcion: '', periodicidad: null });
const asignarMantenimiento = ref({ id_mantenimiento: null, id_maquina: null });
const reclamandoIncidencia = ref(false);
const showMotivoCierrePopup = ref(false);
const motivoCierre = ref('');
const debouncedUpdateDescription = ref(null);
const hasReclamada = ref(false);
const showErrorPopup = ref(false);
const errorMessage = ref('');
const mantenimientosOptions = ref([]);
const maquinasOptions = ref([]); // Asegúrate de que maquinasOptions esté definido
const showIncidenciaDetailsPopup = ref(false);
const selectedIncidencia = ref(null);
const commentText = ref('');
const showFiltroPopup = ref(false);
const searchQuery = ref('');
const selectedSeccion = ref(null);
const seccionOptions = ref([]);
const selectedCampus = ref(null);
const campusOptions = ref([]);
const searchQueryTemp = ref('');
const selectedSeccionTemp = ref(null);
const selectedCampusTemp = ref(null);

// Nuevo estado para el popup de detalles técnicos
const showTecnicoDetailsPopup = ref(false);
const tecnicoDetailsData = ref([]);

// Toast instance
const toast = useToast();
const defaultToastOptions = { position: 'top-right', timeout: 3000, closeOnClick: true, hideProgressBar: true };
const showToast = (message, type = 'success') => toast[type](message, defaultToastOptions);
const showErrorToast = (message, options = {}) => {
toast.error(message, { ...defaultToastOptions, ...options });
};

const obtenerPrioridad = (prioridad) => ['alta', 'media', 'baja'].includes(prioridad) ? prioridad : 'baja';

const obtenerEstado = (estado) => {
  switch (estado) {
    case 0: return 'Nueva';
    case 1: return 'Pendiente';
    case 2: return 'En curso';
    case 3: return 'Cerrada';
    case 4: return 'Mantenimiento';
    default: return '';
  }
};

const formatDate = (dateString) => {
  if (!dateString) return 'Invalid Date'; // Seguridad por si es null o undefined
  const date = new Date(dateString);
  if (isNaN(date)) {
    return 'Invalid Date';
  }
  const options = { year: 'numeric', month: 'long', day: 'numeric' };
  return date.toLocaleDateString(undefined, options);
};

const formatTime = (dateString) => {
  if (!dateString) return 'Invalid Date'; // Seguridad por si es null o undefined
  const date = new Date(dateString);
  if (isNaN(date)) {
    return 'Invalid Date';
  }
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


const fetchData = async () => {
  const token = localStorage.getItem('token');
  if (!token) throw new Error('No token found');
  try {
    const response = await axios.post(ALL_INCIDENCIAS_URL, {}, { headers: { Authorization: `Bearer ${token}` } });
    return response.data;
  } catch (err) {
    console.error('Error al obtener los datos de las incidencias:', err);
    error.value = 'Error al obtener los datos de las incidencias';
    throw error.value;
  }
};

const fetchTipoIncidencias = async () => {
  const token = localStorage.getItem('token');
  if (!token) throw new Error('No token found');
  try {
    const response = await axios.get(ALL_TIPO_INCIDENCIAS_URL, { headers: { Authorization: `Bearer ${token}` } });
    return response.data;
  } catch (error) {
    console.error('Error al obtener los tipos de incidencia:', error);
    error.value = 'Error al obtener los tipos de incidencia';
    throw error.value;
  }
};

const fetchMaquinas = async () => {
  const token = localStorage.getItem('token');
  if (!token) throw new Error('No token found');
  try {
    const response = await axios.post(ALL_MAQUINAS_URL, {}, { headers: { Authorization: `Bearer ${token}` } });
    if (response.data && Array.isArray(response.data.data)) {
      return response.data.data || [];
    } else {
      return [];
    }
  } catch (error) {
    console.error('Error al obtener las maquinas:', error);
    error.value = 'Error al obtener las maquinas';
    throw error.value;
  }
};

const fetchTecnicoIncidencias = async () => {
  try {
    if (!isOperario.value) {
      const token = localStorage.getItem('token');
      if (!token) throw new Error('No token found');
      const response = await axios.get(ALL_TECNICO_INCIDENCIA_URL, { headers: { Authorization: `Bearer ${token}` } });
      tecnicoIncidencias.value = response.data.data;
    }
  } catch (error) {
    console.error('Error al obtener los técnicos de las incidencias:', error);
    error.value = 'Error al obtener los técnicos de las incidencias';
    throw error.value;
  }
};

const fetchTecnicoIncidenciaDetails = async (incidenciaId) => {
  try {
    const token = localStorage.getItem('token');
    if (!token) throw new Error('No token found');
    const response = await axios.get(
      `${TECNICO_INCIDENCIA_DETALLES_URL}/${incidenciaId}`, { headers: { Authorization: `Bearer ${token}` } });
    tecnicoDetailsData.value = response.data.data;
  } catch (error) {
    console.error('Error al obtener los detalles de técnicos del mantenimiento:', error);
    showErrorToast('Error al obtener los detalles de técnicos.');
  }
};


const loadIncidencias = async () => {
  try {
    loading.value = true;
    const incidenciasData = await fetchData();
    incidencias.value = incidenciasData.map((incidencia) => ({
      ...incidencia,
      id: incidencia.id,
      titulo: incidencia.titulo, // <-- Usando 'titulo' del JSON
      subtitulo: incidencia.subtitulo, // <-- Usando 'subtitulo' del JSON
      priority: obtenerPrioridad(incidencia.prioridad),
      status: obtenerEstado(incidencia.estado),
      date: formatDate(incidencia.fecha_apertura), // <-- Usando 'fecha_apertura'
      time: formatTime(incidencia.fecha_apertura), // <-- Usando 'fecha_apertura'
      descripcion: incidencia.descripcion,
      id_tecnico: incidencia.id_mantenimiento,
    }));
  } catch (err) {
    error.value = err;
  } finally {
    loading.value = false;
  }
};

const fetchUserData = async () => {
  const token = localStorage.getItem('token');
  if (token) {
    try {
      const response = await axios.get(ME_URL, { headers: { Authorization: `Bearer ${token}` } });
      userRole.value = response.data.rol;
    } catch (error) {
      console.error('Error fetching user data:', error);
    }
  }
};

const checkUserHasReclamada = () => {
  if (isTecnico.value || isAdmin.value) {
    hasReclamada.value = tecnicoIncidencias.value.some(
      (incidencia) => incidencia.id_tecnico === Number(userId) && incidencia.estado_tecnico === 'activo'
    );
  }
};

const fetchMantenimientosOptions = async () => {
  try {
    const mantenimientosData = await fetchMantenimientos();
    mantenimientosOptions.value = mantenimientosData.map((mantenimiento) => ({
      id: mantenimiento.id,
      label: mantenimiento.nombre,
    }));
  } catch (error) {
    console.error('Error al obtener los mantenimientos para las opciones:', error);
    error.value = 'Error al obtener los mantenimientos para las opciones';
  }
};

const fetchMantenimientos = async () => {
  const token = localStorage.getItem('token');
  if (!token) throw new Error('No token found');
  try {
    const response = await axios.get(`${API_AUTH_URL}/mantenimiento_preventivo/all`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    return response.data.data || [];
  } catch (error) {
    console.error('Error al obtener los mantenimientos:', error);
    error.value = 'Error al obtener los mantenimientos';
    throw error.value;
  }
};

const fetchCampusAndSections = async () => {
  try {
    const token = localStorage.getItem('token');
    if (!token) throw new Error('No token found');

    const campusResponse = await axios.get(CAMPUS_ALL_URL, { headers: { Authorization: `Bearer ${token}` } });
    campusOptions.value = campusResponse.data.data.map((campus) => ({ id: campus.id, label: campus.nombre_campus }));

    const seccionResponse = await axios.get(SECCION_ALL_URL, { headers: { Authorization: `Bearer ${token}` } });
    seccionOptions.value = seccionResponse.data.data.map((seccion) => ({
      id: seccion.id,
      label: seccion.nombre_seccion,
      id_campus: seccion.id_campus,
    }));
  } catch (error) {
    console.error('Error fetching campus and sections:', error);
    error.value = 'Error fetching campus and sections';
  }
};

const filteredIncidenciasPrioridad = computed(() => {
  let filtered = [...incidencias.value];
  if (prioridadSeleccionada.value) {
    filtered = filtered.filter((incidencia) => incidencia.priority === prioridadSeleccionada.value);
  }
  return filtered;
});

const filteredSeccionOptions = computed(() =>
  selectedCampus.value
    ? seccionOptions.value.filter((seccion) => seccion.id_campus === selectedCampus.value)
    : seccionOptions.value
);

const seleccionarPrioridad = (prioridad) => {
  prioridadSeleccionada.value = prioridad;
};

const cancelFilters = () => {
  selectedCampusTemp.value = selectedCampus.value;
  selectedSeccionTemp.value = selectedSeccion.value;
  closeFiltroPopup();
};

const applyFilters = () => {
  selectedCampus.value = selectedCampusTemp.value;
  selectedSeccion.value = selectedSeccionTemp.value;
  closeFiltroPopup();
};

// Lifecycle Hooks
onMounted(async () => {
  try {
    await Promise.all([
      fetchUserData(),
      fetchTipoIncidencias(),
      fetchMantenimientosOptions(),
      fetchCampusAndSections(),
      fetchTecnicoIncidencias() // Cargar técnicos de incidencia al inicio
    ]);

    const maquinasData = await fetchMaquinas();
    maquinas.value = maquinasData;

    maquinasOptions.value = maquinas.value.map((maquina) => ({
      id: maquina.id,
      label: maquina.nombre_maquina,
    }));


    await loadIncidencias();
    checkUserHasReclamada();
  } catch (err) {
    error.value = err;
    loading.value = false;
    showToast(error.value, 'error');
  }
});

watch(incidencias, checkUserHasReclamada);

// Popup Handlers
const openCreateMantenimientoPopup = () => (showCreateMantenimientoPopup.value = true);
const closeCreateMantenimientoPopup = () => {
  showCreateMantenimientoPopup.value = false;
  newMantenimiento.value = { nombre: '', descripcion: '', periodicidad: null };
};
const openAsignarMantenimientoPopup = () => (showAsignarMantenimientoPopup.value = true);
const closeAsignarMantenimientoPopup = () => {
  showAsignarMantenimientoPopup.value = false;
  asignarMantenimiento.value = { id_mantenimiento: null, id_maquina: null };
};
const openIncidenciaDetailsPopup = () => (showIncidenciaDetailsPopup.value = true);
const closeIncidenciaDetailsPopup = () => {
  showIncidenciaDetailsPopup.value = false;
  selectedIncidencia.value = null;
  commentText.value = '';
  motivoSalida.value = '';
  motivoCierre.value = '';
  closeTecnicoDetailsPopup(); // Asegurar que se cierre el popup de detalles técnicos también
};
const openMotivoSalidaPopup = () => (showMotivoSalidaPopup.value = true);
const closeMotivoSalidaPopup = () => {
  showMotivoSalidaPopup.value = false;
  motivoSalida.value = '';
};
const openMotivoCierrePopup = () => (showMotivoCierrePopup.value = true);
const closeMotivoCierrePopup = () => {
  showMotivoCierrePopup.value = false;
  motivoCierre.value = '';
};
const closeErrorPopup = () => {
  showErrorPopup.value = false;
  errorMessage.value = '';
};
const openFiltroPopup = () => {
  showFiltroPopup.value = true;
  selectedCampusTemp.value = selectedCampus.value;
  selectedSeccionTemp.value = selectedSeccion.value;
};
const closeFiltroPopup = () => {
  showFiltroPopup.value = false;
};

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


// Handlers for Incidencias/Mantenimientos Actions
const handleCreateMantenimiento = async () => {
  const token = localStorage.getItem('token');
  if (!token) throw new Error('No token found');
  try {
    const apiUrl = `${API_AUTH_URL}/mantenimiento_preventivo/store`;
    const response = await axios.post(apiUrl, newMantenimiento.value, {
      headers: { Authorization: `Bearer ${token}` },
    });
    if (response.status === 201) {
      closeCreateMantenimientoPopup();
      await fetchMantenimientosOptions();
      await loadIncidencias();
      showToast('Mantenimiento creado con éxito!');
    }
  } catch (error) {
    console.error('Error al crear el mantenimiento:', error);
    showToast('Error al crear el mantenimiento', 'error');
  }
};

const handleAsignarMantenimiento = async () => {
  if (!asignarMantenimiento.value.id_mantenimiento || !asignarMantenimiento.value.id_maquina) {
    return showToast('Por favor seleccione mantenimiento y máquina', 'warning');
  }
  const token = localStorage.getItem('token');
  if (!token) throw new Error('No token found');
  try {
    const apiUrl = `${API_AUTH_URL}/mantenimiento_maquina/asignar_mantenimiento`;
    const response = await axios.post(apiUrl, asignarMantenimiento.value, {
      headers: { Authorization: `Bearer ${token}` },
    });
    if (response.data.success || response.status === 200) {
      closeAsignarMantenimientoPopup();
      await loadIncidencias();
      showToast('Mantenimiento asignado correctamente');
    } else {
      throw new Error(response.data.message || 'Error al asignar el mantenimiento');
    }
  } catch (error) {
    console.error('Error al asignar el mantenimiento:', error);
    showToast(error.response?.data?.message || 'Error al asignar el mantenimiento', 'error');
  }
};

const handleIncidenciaClick = (incidencia) => {
  selectedIncidencia.value = incidencia;
  commentText.value = incidencia.descripcion || '';
  openIncidenciaDetailsPopup();
};

const handleCommentChange = () => {
  clearTimeout(debouncedUpdateDescription.value);
  debouncedUpdateDescription.value = setTimeout(handleUpdateDescription, 1000);
};

const handleUpdateDescription = async () => {
  if (!selectedIncidencia.value) return;
  guardandoComentario.value = true;
  const token = localStorage.getItem('token');
  if (!token) throw new Error('No token found');
  try {
    const apiUrl = `${API_AUTH_URL}/incidencia/update_descripcion/${selectedIncidencia.value.id}`;
    const response = await axios.put(apiUrl, { descripcion: commentText.value }, {
      headers: { Authorization: `Bearer ${token}` },
    });
    if (response.status === 200) selectedIncidencia.value.descripcion = commentText.value;
  } catch (error) {
    console.error('Error al actualizar el comentario:', error);
    showToast('Error al actualizar el comentario.', 'error');
  } finally {
    guardandoComentario.value = false;
  }
};

const handleReclamarIncidencia = async () => {
  if (!selectedIncidencia.value) return;
  reclamandoIncidencia.value = true;
  const token = localStorage.getItem('token');
  if (!token) throw new Error('No token found');
  try {
    const apiUrl = `${API_AUTH_URL}/tecnico_incidencia/reclamar_incidencia`;
    const response = await axios.post(apiUrl, { id_incidencia: selectedIncidencia.value.id }, {
      headers: { Authorization: `Bearer ${token}` },
    });
    if (response.status === 200 || response.status === 201) {
      await loadIncidencias();
      selectedIncidencia.value.status = 'En curso';
      showToast('Mantenimiento reclamado!');
    } else if (response.status === 400) {
      errorMessage.value = 'No puedes reclamar más de un mantenimiento a la vez.';
      showErrorPopup.value = true;
    } else {
      errorMessage.value = `Error al reclamar el mantenimiento: ${response.data.message || 'Compruebe la consola'}`;
      showErrorPopup.value = true;
    }
  } catch (error) {
    if (error.message.includes('400')) {
      errorMessage.value = 'No puedes reclamar más de un mantenimiento a la vez.';
      showErrorPopup.value = true;
    } else {
      errorMessage.value = `Error al reclamar el mantenimiento: ${error.message || 'Compruebe la consola'}`;
      showErrorPopup.value = true;
    }
  } finally {
    reclamandoIncidencia.value = false;
  }
};

const handleSalirIncidencia = async () => {
  if (!selectedIncidencia.value) return;
  const token = localStorage.getItem('token');
  if (!token) throw new Error('No token found');
  try {
    const apiUrl = `${API_AUTH_URL}/tecnico_incidencia/salir_incidencia`;
    const response = await axios.put(apiUrl, { id_incidencia: selectedIncidencia.value.id, motivo_salida: motivoSalida.value }, {
      headers: { Authorization: `Bearer ${token}` },
    });
    if (response.status === 200) {
      closeMotivoSalidaPopup();
      closeIncidenciaDetailsPopup();
      await loadIncidencias();
      showToast('Saliste del mantenimiento!');
    }
  } catch (error) {
    console.error('Error al salir del mantenimiento:', error);
    showToast('Error al salir del mantenimiento.', 'error');
  }
};

const handleCerrarIncidencia = async () => {
  if (!selectedIncidencia.value) return;
  const token = localStorage.getItem('token');
  if (!token) throw new Error('No token found');
  try {
    const apiUrl = `${API_AUTH_URL}/tecnico_incidencia/cerrar_incidencia`;
    const response = await axios.put(apiUrl, { id_incidencia: selectedIncidencia.value.id, motivo_salida: motivoCierre.value }, {
      headers: { Authorization: `Bearer ${token}` },
    });
    if (response.status === 200) {
      await loadIncidencias();
      selectedIncidencia.value.status = 'Cerrada';
      closeMotivoCierrePopup();
      closeIncidenciaDetailsPopup();
      showToast('Mantenimiento cerrado!');
    }
  } catch (error) {
    console.error('Error al cerrar el mantenimiento', error);
    showToast('Error al cerrar el mantenimiento.', 'error');
  }
};
</script>

<style lang="scss" scoped>
// Estilos CSS (sin cambios)
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
$color-text: #333;

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

@mixin flex-center($justify: center, $align: center, $direction: row) {
  display: flex;
  justify-content: $justify;
  align-items: $align;
  flex-direction: $direction;
}

@mixin glassmorphic-card() {
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

  span {
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
$estados: (
  nueva: $color-new,
  pendiente: $color-pending,
  mantenimiento: $color-maintenance,
  cerrada: $color-closed,
  'en-curso': $color-in-progress
);
@each $estado-name, $estado-color in $estados {
  .#{$estado-name} {
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
.total-h2 {
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
  @include flex-center(center, center, column);
  gap: 0.5rem;
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

.boton-filtro {
  cursor: pointer;
  position: relative;
  padding: 5px;
  @include flex-center();
  border-radius: 12px;
  &:hover {
    opacity: 0.8;
  }
}

.imagen-filtro {
  width: 20px;
}

.glassmorphic-select {
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
  .popup-content {
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
  .popup-btn,
  .popup-btn1 {
    font-size: 0.9rem;
    padding: 0.6rem 1.2rem;
    border-radius: 1.5rem;
    text-align: center;
  }
  .incidencias-container {
    padding: 0 5px;
  }
  .priority-legend {
    gap: 10px;
    font-size: 0.8rem;
  }
  .boton-filtro {
    padding: 3px;
  }
  .imagen-filtro {
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