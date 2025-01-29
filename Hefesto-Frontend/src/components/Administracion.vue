<template>
  <!-- Contenedor principal del panel de administración -->
  <div class="container-fluid h-100">
    <!-- Sección principal de tarjetas -->
    <div v-if="!activeCrud">
      <div class="row">
        <!-- Generación dinámica de tarjetas CRUD -->
        <div 
          v-for="(column, index) in crudItems" 
          :key="index" 
          class="col-md-6 h-100"
        >
          <div class="d-flex flex-column h-100">
            <button 
              v-for="item in column" 
              :key="item.name"
              class="glass-card mb-4" 
              @click="openCrud(item.crudKey)"
            >
              <div class="d-flex justify-content-between align-items-center h-100">
                <div class="card-body text-center">
                  <h2 class="card-text">{{ item.title }}</h2>
                </div>
                <div class="card-body text-center image-container">
                  <img 
                    :src="item.icon" 
                    :class="item.imageClass" 
                  />
                </div>
              </div>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Transiciones para componentes CRUD -->
    <transition name="fade">
      <component 
        :is="activeComponent" 
        v-if="activeCrud"
        @close="closeCrud"
        :enableEntity="handleEnable"
        :disableEntity="handleDisable"
        :API_AUTH_URL="API_AUTH_URL"
        :IMAGE_URL="IMAGE_URL"
        @back-to-list="closeCrud"
      />
    </transition>

    <!-- Popup dinámico -->
    <MaquinasPopup
      ref="maquinasPopupRef"
      :visible="popup.visible"
      :title="popup.title"
      :subtitle="popup.subtitle"
      :closeButtonText="popup.closeText"
      :actionButtonText="popup.actionText"
      :token="token"
      @close-popup="closePopup"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import axios from "axios";
import CRUDusuario from "./CRUDusuario.vue";
import CRUDcampus from "./CRUDcampus.vue";
import CRUDsecciones from "./CRUDsecciones.vue";
import CRUDmaquinas from "./CRUDmaquinas.vue";
import MaquinasPopup from "./MaquinasPopup.vue";

// Configuración de constantes
const API_AUTH_URL = import.meta.env.VITE_API_AUTH_URL;
const IMAGE_URL = import.meta.env.VITE_IMAGE_URL;
const ME_URL = `${API_AUTH_URL}/auth/me`;

// Referencias reactivas
const activeCrud = ref(null);
const maquinasPopupRef = ref(null);
const token = ref(null);
const userPicture = ref(null);
const userName = ref(null);

// Configuración del popup
const popup = ref({
  visible: false,
  title: null,
  subtitle: null,
  closeText: "Cerrar",
  actionText: "Aceptar"
});

// Configuración de items CRUD
const crudItems = [
  [
    { 
      title: "Usuarios", 
      icon: "../src/assets/images/icons/usuarios.svg",
      crudKey: "usuarios",
      component: CRUDusuario,
      endpoint: "usuario"
    },
    { 
      title: "Campuses", 
      icon: "../src/assets/images/icons/campuses.svg",
      crudKey: "campus",
      component: CRUDcampus,
      endpoint: "campus"
    }
  ],
  [
    { 
      title: "Secciones", 
      icon: "../src/assets/images/icons/secciones.svg",
      crudKey: "secciones",
      component: CRUDsecciones,
      endpoint: "seccion"
    },
    { 
      title: "Maquinas", 
      icon: "../src/assets/images/icons/maquinas.svg",
      crudKey: "maquinas",
      component: CRUDmaquinas,
      endpoint: "maquina",
      imageClass: "maquinaImage"
    }
  ]
];

// Componente activo computado
const activeComponent = computed(() => {
  return crudItems.flat().find(item => item.crudKey === activeCrud.value)?.component;
});

// Función genérica para habilitar/deshabilitar entidades
const handleEntity = async (action, endpoint, id) => {
  try {
    const headers = { Authorization: `Bearer ${localStorage.getItem('token')}` };
    await axios.put(`${API_AUTH_URL}/${endpoint}/${action}/${id}`, {}, { headers });
    console.log(`${endpoint} ${id} ${action === 'enable' ? 'habilitado' : 'deshabilitado'}`);
  } catch (error) {
    console.error(`Error al ${action} ${endpoint}:`, error);
    throw error;
  }
};

// Manejadores genéricos
const handleEnable = (id, endpoint) => handleEntity('enable', endpoint, id);
const handleDisable = (id, endpoint) => handleEntity('disable', endpoint, id);

// Apertura/cierre de CRUD
const openCrud = (crudKey) => activeCrud.value = crudKey;
const closeCrud = () => activeCrud.value = null;

// Manejo del popup
const openPopup = (type) => {
  const config = {
    perfil: {
      title: "Editar perfil",
      subtitle: "Edita tu perfil"
    },
    contraseña: {
      title: "Cambiar contraseña",
      subtitle: "Introduce tu nueva contraseña"
    },
    fondo: {
      title: "Cambiar fondo",
      subtitle: "Añade la descripción de tu fondo"
    },
    maquinas: {
      title: "Maquinas",
      subtitle: "Listado maquinas",
      onOpen: () => maquinasPopupRef.value.fetchData()
    }
  };

  Object.assign(popup.value, config[type]);
  popup.value.visible = true;
  config[type]?.onOpen?.();
};

const closePopup = () => {
  popup.value.visible = false;
};

// Carga inicial de datos del usuario
onMounted(async () => {
  token.value = localStorage.getItem("token");
  if (!token.value) return;

  try {
    const { data } = await axios.get(ME_URL, {
      headers: { Authorization: `Bearer ${token.value}` }
    });
    userName.value = data.name;
    userPicture.value = data.foto_perfil;
  } catch (error) {
    console.error("Error obteniendo datos del usuario:", error);
  }
});
</script>

<style lang="scss" scoped>
// Variables de color
$white-01: rgba(255, 255, 255, 0.1);
$white-09: rgba(255, 255, 255, 0.9);

// Mixin para efecto glassmorphic
@mixin glassmorphic-card {
  background: rgba($white-09, 0.7);
  backdrop-filter: blur(8px);
  border: 1px solid rgba($white-09, 0.05);
  box-shadow: 0 4px 24px 0 rgba(0, 0, 0, 0.3);
  border-radius: 16px;
}

// Estilos de tarjetas
.glass-card {
  @include glassmorphic-card;
  width: 100%;
  height: 300px;
  cursor: pointer;
  transition: transform 0.2s ease-in-out;
  display: flex;
  align-items: center;
  justify-content: center;

  &:hover {
    transform: scale(1.05);
  }

  .card-text {
    margin-top: 0;
    color: #333;
    font-size: 2rem;
  }

  .card-body {
    padding: 24px;
    text-align: center;
  }
}

// Contenedor de imágenes
.image-container {
  width: 200px;
  height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;

  img {
    max-width: 100%;
    max-height: 100%;
    object-fit: contain;
    
    &.maquinaImage {
      width: 200px;
      height: 200px;
    }
  }
}

// Transiciones
.fade {
  &-enter-from,
  &-leave-to {
    opacity: 0;
  }

  &-enter-active,
  &-leave-active {
    transition: opacity 0.3s;
  }
}
</style>