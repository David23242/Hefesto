<template>
  <div class="glassmorphic-container">
    <div v-if="currentView === 'list'" class="user-list">
          <h2 class="list-title">Lista de Usuarios</h2>
          <button class="glassmorphic-btn add-user-btn" @click="registerModalVisible = true">
              <img src="../assets/images/icons/usuarios.svg" alt="Añadir Usuario" class="btn-icon" />
              Añadir Usuario
          </button>
           <!-- Barra de búsqueda -->
          <div class="search-bar">
              <input type="text" v-model="searchQuery" placeholder="Buscar por nombre o apellido" class="glassmorphic-input"/>
          
          <CustomSelect
                  :options="campusOptions"
                  :modelValue="selectedCampus"
                   @update:modelValue="handleFilterCampusSelect"
                  placeholder="Filtrar por campus"
                  class="glassmorphic-select"
              />
           </div>
          <div class="table-container">
              <table class="glassmorphic-table">
                  <thead>
                  <tr>
                      <th>
                          <input type="checkbox" @change="selectAll" v-model="allSelected" class="glassmorphic-checkbox" />
                      </th>
                      <th>Foto</th>
                      <th>Nombre</th>
                      <th>Email</th>
                      <th>Rol</th>
                      <th>Campus</th>
                      <th>Acciones</th>
                  </tr>
                  </thead>
                  <tbody>
                  <tr v-for="user in filteredUsers" :key="user.id">
                      <td>
                          <input type="checkbox" :value="user.id" @change="selectUser" v-model="selectedUsers" class="glassmorphic-checkbox" />
                      </td>
                      <td>
                          <img :src="props.IMAGE_URL + user.foto_perfil" alt="Foto de perfil" class="user-avatar" />
                      </td>
                      <td>{{ user.name }} {{ user.primer_apellido }} {{ user.segundo_apellido }}</td>
                      <td>{{ user.email }}</td>
                      <td>{{ user.rol }}</td>
                      <td>{{ getCampusName(user.id_campus) }}</td>
                      <td class="action-cell">
                          <label class="switch">
                              <input type="checkbox" :checked="user.habilitado === 1" @change="toggleUserStatus(user)" />
                              <span class="slider"></span>
                          </label>
                          <button class="glassmorphic-btn edit-btn" @click="handleEditUser(user)">Editar</button>
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
          <button class="glassmorphic-btn danger-btn" @click="deleteSelectedUsers" :disabled="selectedUsers.length === 0">Deshabilitar Seleccionados</button>
      </div>
     <teleport to="body">
         <GlassmorphicPopup
          :visible="registerModalVisible"
          title="Registrar Usuario"
          closeButtonText="Cancelar"
           actionButtonText="Registrar"
          @close="goBack"
          @action="handleAction"
          closeButtonStyle="cancel"
          actionButtonStyle="primary"
       >
        <template #popup-content>
          <div class="radio-group">
              <div class="checkbox-wrapper-47">
                <input type="radio" name="rol" id="tecnico" v-model="isTecnico" :value="true" />
                <label for="tecnico">Técnico</label>
              </div>
              <div class="checkbox-wrapper-47">
                <input type="radio" name="rol" id="usuario" v-model="isTecnico" :value="false" />
                <label for="usuario">Usuario</label>
              </div>
          </div>

          <div class="form-group">
              <label for="nombre">Nombre:</label>
              <input type="text" id="nombre" v-model="userData.name" class="glassmorphic-input" />
          </div>

          <div class="form-group">
              <label for="email">Email:</label>
              <input type="email" id="email" v-model="userData.email" class="glassmorphic-input" />
          </div>

          <div class="form-group">
              <label for="password">Contraseña:</label>
              <input type="password" id="password" v-model="userData.password" class="glassmorphic-input" />
          </div>

          <div class="form-group">
              <label for="primer_apellido">Primer Apellido:</label>
              <input type="text" id="primer_apellido" v-model="userData.primer_apellido" class="glassmorphic-input" />
          </div>

          <div class="form-group">
              <label for="segundo_apellido">Segundo Apellido:</label>
              <input type="text" id="segundo_apellido" v-model="userData.segundo_apellido" class="glassmorphic-input" />
          </div>

          <div class="form-group">
              <label for="campus">Campus:</label>
              <CustomSelect
                  :options="campusOptions"
                  :modelValue="userData.id_campus"
                  @update:modelValue="handleCampusSelect"
                  class="glassmorphic-select"
              />
          </div>
         </template>
       </GlassmorphicPopup>
     </teleport>
     <teleport to="body">
          <GlassmorphicPopup
            :visible="editModalVisible"
            title="Editar Usuario"
            closeButtonText="Cancelar"
            actionButtonText="Guardar"
            @close="closeEditModal"
            @action="updateUser"
            closeButtonStyle="cancel"
            actionButtonStyle="primary"
          >
            <template #popup-content>
             <div class="form-group">
              <label for="edit_nombre">Nombre:</label>
                  <input type="text" id="edit_nombre" v-model="editUserData.name" class="glassmorphic-input" />
              </div>

               <div class="form-group">
                   <label for="edit_email">Email:</label>
                   <input type="email" id="edit_email" v-model="editUserData.email" class="glassmorphic-input" />
               </div>

          <div class="form-group">
              <label for="edit_password">Contraseña:</label>
              <input type="password" id="edit_password" v-model="editUserData.password" class="glassmorphic-input" />
          </div>

           <div class="form-group">
               <label for="edit_primer_apellido">Primer Apellido:</label>
               <input type="text" id="edit_primer_apellido" v-model="editUserData.primer_apellido" class="glassmorphic-input" />
           </div>

           <div class="form-group">
               <label for="edit_segundo_apellido">Segundo Apellido:</label>
               <input type="text" id="edit_segundo_apellido" v-model="editUserData.segundo_apellido" class="glassmorphic-input" />
           </div>
            <div class="form-group">
               <label for="edit_rol">Rol:</label>
                <select id="edit_rol" v-model="editUserData.rol" class="glassmorphic-select">
                 <option value="operario">Operario</option>
                   <option value="tecnico">Técnico</option>
                  </select>
            </div>
           <div class="form-group">
              <label for="edit_campus">Campus:</label>
               <CustomSelect
                  :options="campusOptions"
                   :modelValue="editUserData.id_campus"
                  @update:modelValue="handleEditCampusSelect"
                class="glassmorphic-select"
                  />
           </div>
            </template>
         </GlassmorphicPopup>
    </teleport>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import CustomSelect from './CustomSelect.vue';
import axios from 'axios';
import { useToast } from 'vue-toastification';
import GlassmorphicPopup from './GlassmorphicPopup.vue';

const toast = useToast();
const API_AUTH_URL = import.meta.env.VITE_API_AUTH_URL;
const REGISTER_URL = `${API_AUTH_URL}/auth/register`;
const REGISTER_TECNICO_URL = `${API_AUTH_URL}/auth/register_tecnico`;
const CAMPUS_ALL_URL = `${API_AUTH_URL}/campus/all`;
const USUARIO_ALL_URL = `${API_AUTH_URL}/usuario/all`;
const USUARIO_DELETE_URL = `${API_AUTH_URL}/usuario/update`;
const USUARIO_UPDATE_URL = `${API_AUTH_URL}/usuario/update`;

const isTecnico = ref(false);
const userData = ref({
name: '',
email: '',
password: '',
primer_apellido: '',
segundo_apellido: '',
id_campus: null,
});

const campusOptions = ref([]);
const currentView = ref('list');
const users = ref([]);
const selectedUsers = ref([]);
const allSelected = ref(false);
const emit = defineEmits(['close', 'back-to-list'])
const props = defineProps({
  closeUsuariosCrud: {
      type: Function,
  },
   enableUser: {
      type: Function,
  },
  disableUser: {
    type: Function
 },
  API_AUTH_URL: {
          type: String,
          required: true
      },
       IMAGE_URL: {
          type: String,
          required: true
      }
});
const searchQuery = ref(''); // Para la barra de búsqueda
const selectedCampus = ref(null);
const filteredUsers = computed(() => {
  let filtered = [...users.value];
  
    if (searchQuery.value) {
      const searchTerm = searchQuery.value.toLowerCase();
        filtered = filtered.filter(user => {
            const fullName = `${user.name} ${user.primer_apellido} ${user.segundo_apellido}`.toLowerCase();
            return fullName.includes(searchTerm);
        });
    }
    
    if (selectedCampus.value) {
        filtered = filtered.filter(user => user.id_campus === selectedCampus.value);
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

const handleCampusSelect = (campusId) => {
userData.value.id_campus = campusId;
};
const handleEditCampusSelect = (campusId) => {
 editUserData.value.id_campus = campusId;
};
const registerModalVisible = ref(false);


const handleAction = async () => {
  try {
      const url = isTecnico.value ? REGISTER_TECNICO_URL : REGISTER_URL;
      const payload = {};

      // Campos requeridos
      if (!userData.value.name) {
          alert('El campo Nombre es obligatorio.');
          return; // Detiene la ejecución si el nombre está vacío
      }
      payload.name = userData.value.name; // Cambiado a 'name'

      if (!userData.value.email) {
          alert('El campo Email es obligatorio.');
          return; // Detiene la ejecución si el email está vacío
      }
      payload.email = userData.value.email;

      if (!userData.value.password) {
          alert('El campo Contraseña es obligatorio.');
          return; // Detiene la ejecución si la contraseña está vacía
      }
      payload.password = userData.value.password;
   if (userData.value.id_campus) {
       payload.id_campus = Number(userData.value.id_campus);
        }
      else{
          alert('El campo Campus es obligatorio.');
          return;
      }
      // Campos opcionales
      if (userData.value.primer_apellido) {
          payload.primer_apellido = userData.value.primer_apellido;
      }
      if (userData.value.segundo_apellido) {
          payload.segundo_apellido = userData.value.segundo_apellido;
      }


      const token = localStorage.getItem('token');
      const headers = {
          Authorization: `Bearer ${token}`,
      };

      const response = await axios.post(url, payload, { headers });
       toast.success("Usuario registrado exitosamente!");

      // Restablecer los valores de userData a sus valores iniciales
      userData.value = {
          name: '',
          email: '',
          password: '',
          primer_apellido: '',
          segundo_apellido: '',
          id_campus: null,
      };
       registerModalVisible.value = false;
  } catch (error) {
      console.error('Error al registrar usuario:', error.response ? error.response.data : error);
    toast.error('Error al registrar usuario: ' + (error.response ? JSON.stringify(error.response.data) : error));
  }
};

// Función para obtener el nombre del campus
const getCampusName = (campusId) => {
const campus = campusOptions.value.find((c) => c.id === campusId);
return campus ? campus.label : 'Desconocido';
};

// Función para obtener los usuarios de la API
const fetchUsers = async (page = 1) => {
try {
  const token = localStorage.getItem('token');
  const headers = {
    Authorization: `Bearer ${token}`,
  };
     const response = await axios.get(`${USUARIO_ALL_URL}?per_page=20&page=${page}`, { headers });
    users.value = response.data.data.data;
    pagination.value = {
        current_page: response.data.data.current_page,
        last_page: response.data.data.last_page,
        per_page: response.data.data.per_page,
        total: response.data.data.total,
    }
} catch (error) {
  console.error('Error al obtener usuarios:', error);
   toast.error('Error al obtener usuarios: ' + error.message);
}
};

// Función para seleccionar un usuario
const selectUser = (event) => {
  const userId = parseInt(event.target.value);
  const index = selectedUsers.value.indexOf(userId);

if (event.target.checked) {
  // Add the user if not already selected
  if(index === -1){
      selectedUsers.value.push(userId);
  }
} else {
  // Remove the user if selected
   if (index !== -1) {
      selectedUsers.value.splice(index, 1);
  }
}
};

// Función para seleccionar todos los usuarios
const selectAll = () => {
if (allSelected.value) {
  selectedUsers.value = users.value.map((user) => parseInt(user.id));
} else {
  selectedUsers.value = [];
}
};
// Computed para sincronizar el checkbox selectAll
allSelected.value = computed({
get: () => selectedUsers.value.length === users.value.length,
set: (val) => {
  if (val) {
    selectedUsers.value = users.value.map((user) => parseInt(user.id));
  } else {
    selectedUsers.value = [];
  }
},
});


const toggleUserStatus = async (user) => {
  try {
    if (user.habilitado === 1) {
      await props.disableUser(user.id);
      user.habilitado = 0;
    } else {
       await props.enableUser(user.id);
       user.habilitado = 1;
    }
  } catch(error){
      console.error('Error al cambiar el estado del usuario en la vista:', error);
  }
};
const editModalVisible = ref(false);
const editUserData = ref({
id:'',
name: '',
email: '',
password: '',
primer_apellido: '',
segundo_apellido: '',
rol:'',
  id_campus: null,
});
const handleEditUser = async (user) => {
  try{
      const token = localStorage.getItem('token');
    const headers = {
        Authorization: `Bearer ${token}`,
    };
     // Corrected URL to: api/v1/usuario/show/{usuario}
    const response = await axios.get(`${API_AUTH_URL}/usuario/show/${user.id}`,{headers})
      if (response.data) {
          const userData = response.data
          editUserData.value={
              id: userData.id,
              name: userData.name,
              email: userData.email,
              password: '',
              primer_apellido: userData.primer_apellido,
              segundo_apellido: userData.segundo_apellido,
              rol:userData.rol,
              id_campus: userData.id_campus,
              }
         editModalVisible.value = true;

      } else {
          console.error('Error: La respuesta de la API no tiene datos:', response);
           toast.error("Error: la respuesta del API es incorrecta");
      }


   } catch (error) {
      console.error('Error al obtener datos de usuario:', error);
      toast.error('Error al obtener datos de usuario: ' + error.message);
  }


};
const updateUser = async () =>{
  try {
      const token = localStorage.getItem('token');
      const headers = {
          Authorization: `Bearer ${token}`,
      };
      const payload = {...editUserData.value}
      delete payload.id;
    const response = await axios.put(`${USUARIO_UPDATE_URL}/${editUserData.value.id}`,payload,{headers})
       toast.success("Usuario modificado exitosamente!");
      closeEditModal();
        fetchUsers(pagination.value.current_page)
  }
  catch (error) {
       console.error('Error al actualizar usuario:', error);
      toast.error('Error al actualizar usuario: ' +  (error.response ? JSON.stringify(error.response.data) : error));
  }

}
const closeEditModal = () => {
editModalVisible.value = false;
};
const deleteSelectedUsers = async () => {
if (confirm('¿Estás seguro de que quieres eliminar los usuarios seleccionados?')) {
  try {
    const token = localStorage.getItem('token');
    const headers = {
      Authorization: `Bearer ${token}`,
    };

    // Eliminar usuarios individualmente
    for (const userId of selectedUsers.value) {
      await axios.put(`${USUARIO_DELETE_URL}/${userId}`, { habilitado: 0 }, { headers });
    }

    toast.success('Usuarios deshabilitados exitosamente.');
    selectedUsers.value = []; // Limpiar la selección
      fetchUsers(pagination.value.current_page) // Recargar la lista de usuarios
  } catch (error) {
    console.error('Error al deshabilitar los usuarios:', error);
    toast.error('Error al deshabilitar los usuarios: ' + error.message);
  }
}
};
const handleFilterCampusSelect = (campusId) =>{
  selectedCampus.value = campusId;
}
const pagination = ref({
    current_page: 1,
    last_page: null,
    per_page: 20,
    total: null
})
const changePage = (page) =>{
  if(page > 0 && page <= pagination.value.last_page){
      fetchUsers(page);
  }
}
onMounted(async () => {
try {
  const token = localStorage.getItem('token');
  const headers = {
    Authorization: `Bearer ${token}`,
  };
  const response = await axios.get(CAMPUS_ALL_URL, { headers });
  campusOptions.value = response.data.data.map((campus) => ({
    id: campus.id,
    label: campus.nombre_campus,
  }));
} catch (error) {
  console.error('Error al obtener campus:', error);
  toast.error('Error al obtener campus: ' + error.message);
}
   await fetchUsers();
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
  @import '@/assets/scss/cruds/_avatars.scss';
</style>