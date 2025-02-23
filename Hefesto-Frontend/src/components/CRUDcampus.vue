<template>
    <div class="glassmorphic-container">
      <div v-if="currentView === 'list'" class="campus-list">
            <h2 class="list-title">Lista de Campus</h2>
            <button class="glassmorphic-btn add-campus-btn" @click="registerModalVisible = true">
                <img src="../assets/images/icons/campus.svg" alt="Añadir Campus" class="btn-icon" />
                Añadir Campus
            </button>
             <!-- Barra de búsqueda -->
            <div class="search-bar">
                <input type="text" v-model="searchQuery" placeholder="Buscar por nombre" class="glassmorphic-input"/>
            </div>
            <div class="table-container">
                <table class="glassmorphic-table">
                    <thead>
                    <tr>
                         <th>
                            <input type="checkbox" @change="selectAll" v-model="allSelected" class="glassmorphic-checkbox" />
                        </th>
                        <th>Nombre</th>
                        <th>Acciones</th>
                    </tr>
                    </thead>
                    <tbody>
                    <tr v-for="campus in filteredCampus" :key="campus.id">
                        <td>
                            <input type="checkbox" :value="campus.id" @change="selectCampus" v-model="selectedCampus" class="glassmorphic-checkbox" />
                        </td>
                        <td>{{ campus.nombre_campus }}</td>
                        <td class="action-cell">
                            <label class="switch">
                                <input type="checkbox" :checked="campus.habilitado === 1" @change="toggleCampusStatus(campus)" />
                                <span class="slider"></span>
                            </label>
                            <button class="glassmorphic-btn edit-btn" @click="handleEditCampus(campus)">Editar</button>
                        </td>
                    </tr>
                    </tbody>
                </table>
            </div>
          <!-- Paginacion-->
           <div class="pagination">
                  <button
                          @click="changePage(pagination.current_page - 1)"
                          :disabled="pagination.current_page === 1"
                          class="glassmorphic-btn pagination-btn"
                  >
                    Anterior
                  </button>
  
                 <span class="pagination-info">Página {{ pagination.current_page }} de {{ pagination.last_page }}</span>
  
                  <button
                          @click="changePage(pagination.current_page + 1)"
                          :disabled="pagination.current_page === pagination.last_page"
                          class="glassmorphic-btn pagination-btn"
                  >
                  Siguiente
                  </button>
          </div>
            <button v-if="currentView === 'list'" class="glassmorphic-btn" @click="goBackToList">Volver</button>
             <button class="glassmorphic-btn danger-btn" @click="deleteSelectedCampus" :disabled="selectedCampus.length === 0">Deshabilitar Seleccionados</button>
        </div>
       <teleport to="body">
           <GlassmorphicPopup
            :visible="registerModalVisible"
            title="Registrar Campus"
            closeButtonText="Cancelar"
             actionButtonText="Registrar"
            @close="goBack"
            @action="handleAction"
            closeButtonStyle="cancel"
            actionButtonStyle="primary"
         >
          <template #popup-content>
  
            <div class="form-group">
                <label for="nombre">Nombre:</label>
                <input type="text" id="nombre" v-model="campusData.nombre_campus" class="glassmorphic-input" />
            </div>
  
           </template>
         </GlassmorphicPopup>
       </teleport>
       <teleport to="body">
            <GlassmorphicPopup
              :visible="editModalVisible"
              title="Editar Campus"
              closeButtonText="Cancelar"
              actionButtonText="Guardar"
              @close="closeEditModal"
              @action="updateCampus"
              closeButtonStyle="cancel"
              actionButtonStyle="primary"
            >
              <template #popup-content>
  
                <div class="form-group">
                    <label for="edit_nombre">Nombre:</label>
                    <input type="text" id="edit_nombre" v-model="editCampusData.nombre_campus" class="glassmorphic-input" />
                </div>
  
              </template>
           </GlassmorphicPopup>
      </teleport>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, computed } from 'vue';
  import axios from 'axios';
  import { useToast } from 'vue-toastification';
  import GlassmorphicPopup from './GlassmorphicPopup.vue';
  
  const toast = useToast();
  const API_AUTH_URL = import.meta.env.VITE_API_AUTH_URL;
  const CAMPUS_STORE_URL = `${API_AUTH_URL}/campus/store`;
    const CAMPUS_ALL_URL = `${API_AUTH_URL}/campus/all`;
  const CAMPUS_UPDATE_URL = `${API_AUTH_URL}/campus/update`;
    const CAMPUS_ENABLE_URL = `${API_AUTH_URL}/campus/enable`;
      const CAMPUS_DISABLE_URL = `${API_AUTH_URL}/campus/disable`;
  const CAMPUS_DELETE_URL = `${API_AUTH_URL}/campus/delete`;
  const campusData = ref({
    nombre_campus: '',
  });
  const currentView = ref('list');
  const campus = ref([]);
  const selectedCampus = ref([]);
  const allSelected = ref(false);
  const emit = defineEmits(['close', 'back-to-list'])
  
  const props = defineProps({
      enableCampus: {
          type: Function,
      },
      disableCampus:{
          type: Function,
      }
  });
  
  const searchQuery = ref(''); // Para la barra de búsqueda
  const filteredCampus = computed(() => {
    let filtered = [...campus.value];
      if (searchQuery.value) {
        const searchTerm = searchQuery.value.toLowerCase();
          filtered = filtered.filter(campus => {
              const campusName = `${campus.nombre_campus}`.toLowerCase();
              return campusName.includes(searchTerm);
          });
      }
    return filtered;
  });
  
  
  const openRegisterView = () => {
  currentView.value = 'register';
  };
  const goBack = () => {
    currentView.value = 'list';
      registerModalVisible.value = false;
  
  };
  const goBackToList = () => {
  emit('back-to-list')
  }
  
  
  const registerModalVisible = ref(false);
  
  
  const handleAction = async () => {
    try {
        const token = localStorage.getItem('token');
        const headers = {
            Authorization: `Bearer ${token}`,
        };
          if (!campusData.value.nombre_campus) {
              alert('El campo Nombre es obligatorio.');
              return; // Detiene la ejecución si el nombre está vacío
          }
  
        const response = await axios.post(CAMPUS_STORE_URL, campusData.value, { headers });
        console.log('Registro exitoso:', response.data);
         toast.success("Campus registrado exitosamente!");
  
        // Restablecer los valores de campusData a sus valores iniciales
        campusData.value = {
            nombre_campus: '',
        };
         registerModalVisible.value = false;
         fetchCampus();
    } catch (error) {
        console.error('Error al registrar campus:', error.response ? error.response.data : error);
      toast.error('Error al registrar campus: ' + (error.response ? JSON.stringify(error.response.data) : error));
    }
  };
  
  // Función para obtener los campus de la API
  const fetchCampus = async (page = 1) => {
  try {
    const token = localStorage.getItem('token');
    const headers = {
      Authorization: `Bearer ${token}`,
    };
       const response = await axios.get(`${CAMPUS_ALL_URL}?per_page=20&page=${page}`, { headers });
       console.log('campus response: ',response.data)
      campus.value = response.data.data;
       pagination.value = {
           current_page: response.data.current_page,
           last_page: response.data.last_page,
           per_page: response.data.per_page,
           total: response.data.total,
       }
  } catch (error) {
    console.error('Error al obtener campus:', error);
     toast.error('Error al obtener campus: ' + error.message);
  }
  };
  
  // Función para seleccionar un campus
  const selectCampus = (event) => {
    const campusId = parseInt(event.target.value);
    const index = selectedCampus.value.indexOf(campusId);
  
  if (event.target.checked) {
    // Add the campus if not already selected
    if(index === -1){
        selectedCampus.value.push(campusId);
    }
  } else {
    // Remove the campus if selected
     if (index !== -1) {
        selectedCampus.value.splice(index, 1);
    }
  }
  console.log('Campus seleccionados:', selectedCampus.value);
  };
  
  // Función para seleccionar todos los campus
  const selectAll = () => {
  if (allSelected.value) {
    selectedCampus.value = campus.value.map((campus) => parseInt(campus.id));
  } else {
    selectedCampus.value = [];
  }
  console.log('Campus seleccionados:', selectedCampus.value);
  };
  
  // Computed para sincronizar el checkbox selectAll
  allSelected.value = computed({
  get: () => selectedCampus.value.length === campus.value.length,
  set: (val) => {
    if (val) {
      selectedCampus.value = campus.value.map((campus) => parseInt(campus.id));
    } else {
      selectedCampus.value = [];
    }
  },
  });
  
  
  const toggleCampusStatus = async (campusItem) => {
    try {
      if (campusItem.habilitado === 1) {
          const token = localStorage.getItem('token');
           const headers = {
              Authorization: `Bearer ${token}`,
          };
        await axios.put(`${CAMPUS_DISABLE_URL}/${campusItem.id}`,{},{headers});
         campusItem.habilitado = 0;
           console.log(`Campus ${campusItem.id} deshabilitado`);
      } else {
            const token = localStorage.getItem('token');
           const headers = {
              Authorization: `Bearer ${token}`,
          };
          await axios.put(`${CAMPUS_ENABLE_URL}/${campusItem.id}`,{},{headers});
           campusItem.habilitado = 1;
           console.log(`Campus ${campusItem.id} habilitado`);
      }
    } catch(error){
         console.error('Error al cambiar el estado del campus en la vista:', error);
    }
  };
  
  const editModalVisible = ref(false);
  const editCampusData = ref({
  id:'',
  nombre_campus: '',
  });
  
  const handleEditCampus = async (campusItem) => {
    try{
        const token = localStorage.getItem('token');
      const headers = {
          Authorization: `Bearer ${token}`,
      };
       // Corrected URL to: api/v1/campus/show/{campus}
      const response = await axios.get(`${API_AUTH_URL}/campus/show/${campusItem.id}`,{headers})
        if (response.data.data) {
            const campusData = response.data.data
            editCampusData.value={
                id: campusData.id,
                nombre_campus: campusData.nombre_campus
                }
           editModalVisible.value = true;
        } else {
            console.error('Error: La respuesta de la API no tiene datos:', response);
             toast.error("Error: la respuesta del API es incorrecta");
        }
     } catch (error) {
        console.error('Error al obtener datos del campus:', error);
         toast.error('Error al obtener datos del campus: ' + error.message);
    }
  
  console.log('Editing campus:', campusItem);
  };
  const updateCampus = async () =>{
    try {
        const token = localStorage.getItem('token');
        const headers = {
            Authorization: `Bearer ${token}`,
        };
        const payload = {...editCampusData.value}
        delete payload.id;
      const response = await axios.put(`${CAMPUS_UPDATE_URL}/${editCampusData.value.id}`,payload,{headers})
         console.log("Respuesta de la api: ", response);
         toast.success("Campus modificado exitosamente!");
        closeEditModal();
        fetchCampus(pagination.value.current_page)
    }
    catch (error) {
        console.error('Error al actualizar campus:', error);
      toast.error('Error al actualizar campus: ' +  (error.response ? JSON.stringify(error.response.data) : error));
    }
  
  }
  const closeEditModal = () => {
  editModalVisible.value = false;
  };
  const deleteSelectedCampus = async () => {
    if (confirm('¿Estás seguro de que quieres deshabilitar los campus seleccionados?')) {
        try {
            const token = localStorage.getItem('token');
            const headers = {
                Authorization: `Bearer ${token}`,
            };
  
            // Eliminar campus individualmente
            for (const campusId of selectedCampus.value) {
                await axios.put(`${API_AUTH_URL}/campus/update/${campusId}`, { habilitado: 0 }, { headers });
                console.log(`Campus ${campusId} deshabilitado`);
            }
           toast.success('Campus deshabilitados exitosamente.');
           selectedCampus.value = [];
          fetchCampus(pagination.value.current_page) // Recargar la lista de campus
        } catch (error) {
            console.error('Error al deshabilitar los campus:', error);
            toast.error('Error al deshabilitar los campus: ' + error.message);
        }
    }
  };
  const pagination = ref({
      current_page: 1,
      last_page: null,
      per_page: 20,
      total: null
  })
  const changePage = (page) =>{
      if(page > 0 && page <= pagination.value.last_page){
          fetchCampus(page);
      }
  }
  onMounted(async () => {
     await fetchCampus();
  });
  
  </script>
<style lang="scss" scoped>
@import '@/assets/scss/cruds/_variables.scss';
  @import '@/assets/scss/cruds/_glassmorphic.scss';
  @import '@/assets/scss/cruds/_buttons.scss';
  @import '@/assets/scss/cruds/_forms.scss';
  @import '@/assets/scss/cruds/_tables.scss';
  @import '@/assets/scss/cruds/_pagination.scss';
  @import '@/assets/scss/cruds/_switches.scss';
  @import '@/assets/scss/cruds/_checkboxes.scss';
  @import '@/assets/scss/cruds/_typography.scss';
  @import '@/assets/scss/cruds/_icons.scss';
</style>