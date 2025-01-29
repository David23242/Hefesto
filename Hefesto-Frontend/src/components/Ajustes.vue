<template>
  <div class="container-fluid h-100">
    <div class="card-grid h-100">
      <!-- Tarjetas configuración -->
      <button
        v-for="(card, index) in cards"
        :key="index"
        class="glass-card large-card grow full-height"
        @click="openPopup(card.type)"
      >
        <div class="card-body text-center d-flex flex-column justify-content-center align-items-center">
          <img
            :src="getCardIconSrc(card.icon)"
            :alt="card.altText"
            class="img-fluid mb-3 huge-icon"
          >
          <p class="card-text huge-text">{{ card.text }}</p>
        </div>
      </button>
    </div>

    <!-- Popup principal -->
    <GlassmorphicPopup
      :visible="popup.visible"
      :title="popup.title"
      :subtitle="popup.subtitle"
      :closeButtonText="popup.closeText"
      :actionButtonText="popup.actionText"
      :imageUrl="userImagePath"
      @close="closePopup"
      @action="handleAction"
      @image-changed="updateProfileImage"
    >
      <!-- Contenido para editar perfil -->
      <template v-if="popup.type === 'perfil'" #popup-content>
        <div class="centered-file-upload" @dragover.prevent @drop="handleDrop">
          <form class="file-upload-form">
            <label for="file" class="file-upload-label">
              <div class="file-upload-design">
                <svg viewBox="0 0 640 512" height="1em">
                  <path
                    d="M144 480C64.5 480 0 415.5 0 336c0-62.8 40.2-116.2 96.2-135.9c-.1-2.7-.2-5.4-.2-8.1c0-88.4 71.6-160 160-160c59.3 0 111 32.2 138.7 80.2C409.9 102 428.3 96 448 96c53 0 96 43 96 96c0 12.2-2.3 23.8-6.4 34.6C596 238.4 640 290.1 640 352c0 70.7-57.3 128-128 128H144zm79-217c-9.4 9.4-9.4 24.6 0 33.9s24.6 9.4 33.9 0l39-39V392c0 13.3 10.7 24 24 24s24-10.7 24-24V257.9l39 39c9.4 9.4 24.6 9.4 33.9 0s9.4-24.6 0-33.9l-80-80c-9.4-9.4-24.6-9.4-33.9 0l-80 80z"
                  ></path>
                </svg>
                <span v-if="!fileState.uploadSuccess" class="browse-button">Seleccionar archivo</span>
                <div v-if="fileState.uploadSuccess" class="success-message">
                  Imagen cargada correctamente!
                </div>
              </div>
              <input id="file" type="file" @change="handleFileSelect" style="display: none;" />
            </label>
          </form>
        </div>
      </template>

      <!-- Contenido para cambiar contraseña -->
      <template v-else-if="popup.type === 'pwd'" #popup-content>
        <label for="current_password">Contraseña actual:</label>
        <input
          type="password"
          id="current_password"
          class="form-control"
          placeholder="Tu contraseña actual"
          v-model="passwordState.current"
        />
        <label for="new_password">Contraseña nueva:</label>
        <input
          type="password"
          id="new_password"
          class="form-control"
          placeholder="La contraseña nueva"
          v-model="passwordState.new"
        />
      </template>

      <!-- Contenido para fondos de pantalla -->
      <template v-else-if="popup.type === 'fondo'" #popup-content>
        <div class="background-options">
          <div
            v-for="(bg, index) in backgrounds"
            :key="index"
            class="background-option"
            @click="changeBackground(bg.id)"
          >
            <div class="preview" :style="{ background: bg.color }"></div>
            <p>{{ bg.name }}</p>
          </div>
        </div>
      </template>
    </GlassmorphicPopup>
  </div>
</template>

<script setup>
import GlassmorphicPopup from "./GlassmorphicPopup.vue";
import { ref, onMounted, computed, reactive } from "vue";
import axios from "axios";
import { useToast } from "vue-toastification";

// Importa los iconos de las tarjetas
import profileIcon from '../assets/images/icons/profile.svg';
import passwordIcon from '../assets/images/icons/password.svg';
import wallpaIcon from '../assets/images/icons/wallpa.svg';

// Constantes de configuración
const API_URLS = {
  auth: import.meta.env.VITE_API_AUTH_URL,
  get me() { return `${this.auth}/auth/me` },
  get image() { return `${this.auth}/image/upload` },
  get password() { return `${this.auth}/usuario/reset_password` }
};

// Estados reactivos
const popup = reactive({
  visible: false,
  type: null,
  title: null,
  subtitle: null,
  closeText: 'Cerrar',
  actionText: 'Guardar'
});

const userData = reactive({
  picture: null,
  name: null,
  lastName: null
});

const fileState = reactive({
  selected: null,
  uploadSuccess: false
});

const passwordState = reactive({
  current: null,
  new: null
});

const currentStyle = ref(null);
const toast = useToast();

// Configuraciones de componentes
const cards = [
  { type: 'perfil', icon: 'profile', altText: 'Foto de perfil', text: 'Foto de perfil' },
  { type: 'pwd', icon: 'password', altText: 'Contraseña', text: 'Contraseña' },
  { type: 'fondo', icon: 'wallpa', altText: 'Fondo de pantalla', text: 'Fondo de pantalla' }
];

const backgrounds = [
  { id: 'style1', color: '#1e1f1e', name: 'Animado Negro' },
  { id: 'style9', color: '#1e1f1e', name: 'Negro' },
  { id: 'style2', color: '#74309c', name: 'Animado Morado' },
  { id: 'style3', color: '#b64aec', name: 'Morado' },
  { id: 'style4', color: '#ADD8E6', name: 'Azul' },
  { id: 'style5', color: '#90EE90', name: 'Verde' },
  { id: 'style6', color: '#FF69B4', name: 'Rosa' },
  { id: 'style7', color: '#FF4500', name: 'Rojo' },
  { id: 'style8', color: '#FFFF00', name: 'Amarillo'}
];

// Computados
const userImagePath = computed(() =>
  userData.picture
    ? `../src/assets/images/userpicture/${userData.picture}`
    : null
);

// Métodos para obtener las rutas de los iconos de las tarjetas
const getCardIconSrc = (iconName) => {
  switch (iconName) {
    case 'profile':
      return profileIcon;
    case 'password':
      return passwordIcon;
    case 'wallpa':
      return wallpaIcon;
    default:
      return null; // o una imagen por defecto si lo deseas
  }
};


// Métodos principales
const openPopup = (type) => {
  Object.assign(popup, {
    visible: true,
    type,
    title: getPopupTitle(type),
    actionText: type === 'fondo' ? 'Aceptar' : 'Guardar'
  });

  fileState.uploadSuccess = false;
};

const closePopup = () => {
  popup.visible = false;
};

const handleAction = async () => {
  const actions = {
    pwd: resetPwd,
    perfil: updateProfileImage
  };

  if (actions[popup.type]) await actions[popup.type]();
  closePopup();
};

// Métodos de archivos
const handleFileSelect = (event) => {
  fileState.selected = event.target.files[0];
  updateProfileImage();
};

const handleDrop = (event) => {
  event.preventDefault();
  fileState.selected = event.dataTransfer.files[0];
  updateProfileImage();
};

const updateProfileImage = async () => {
  if (!fileState.selected) return;

  const formData = new FormData();
  formData.append("image", fileState.selected);

  try {
    const { data } = await axios.post(API_URLS.image, formData, {
      headers: authHeader()
    });

    userData.picture = data.path;
    fileState.uploadSuccess = true;
    toast.success("Imagen actualizada correctamente");
  } catch (error) {
    handleError(error, "Error al actualizar la imagen");
  }
};

// Métodos de contraseña
const resetPwd = async () => {
  try {
    await axios.put(API_URLS.password, {
      current_password: passwordState.current,
      new_password: passwordState.new
    }, { headers: authHeader() });

    toast.success("Contraseña actualizada correctamente");
    passwordState.current = null;
    passwordState.new = null;
  } catch (error) {
    handleError(error, "Error al cambiar contraseña");
  }
};

// Métodos de fondo
const changeBackground = (styleName) => {
  removeCurrentBackground();
  applyNewBackground(styleName);
  localStorage.setItem("selectedBackground", styleName);
  currentStyle.value = styleName;
  closePopup();
};

// Helpers
const getPopupTitle = (type) => ({
  perfil: 'Editar perfil',
  pwd: 'Cambiar contraseña',
  fondo: 'Cambiar fondo'
}[type]);

const authHeader = () => ({
  Authorization: `Bearer ${localStorage.getItem("token")}`
});

const removeCurrentBackground = () => {
  const oldStyle = document.getElementById("dynamic-background-style");
  oldStyle?.remove();
};

const applyNewBackground = (styleName) => {
  // Directamente construimos la ruta al CSS aquí, ya que no es una imagen
  const link = document.createElement("link");
  link.id = "dynamic-background-style";
  link.rel = "stylesheet";
  link.href = `../backgrounds/${styleName}.css`;
  document.head.appendChild(link);
};

const handleError = (error, message) => {
  console.error(message, error);
  toast.error(error.response?.data?.message || message);
};

// Ciclo de vida
onMounted(async () => {
  try {
    const { data } = await axios.get(API_URLS.me, { headers: authHeader() });
    Object.assign(userData, {
      name: data.name,
      lastName: data.primer_apellido,
      picture: data.foto_perfil
    });
  } catch (error) {
    handleError(error, "Error al cargar datos del usuario");
  }

  const savedBackground = localStorage.getItem("selectedBackground");
  if (savedBackground) changeBackground(savedBackground);
});
</script>

<style lang="scss" scoped>
/* Variables SCSS reutilizables */
$white-opacities: (
  01: 0.1,
  02: 0.2,
  03: 0.3,
  05: 0.5,
  06: 0.6,
  07: 0.7,
  08: 0.8,
  09: 0.9
);

@function white($opacity) {
  @return rgba(255, 255, 255, map-get($white-opacities, $opacity));
}

/* Mixins reutilizables */
@mixin glass-card($opacity: 07) {
  background: white($opacity) !important;
  backdrop-filter: blur(8px);
  border: 1px solid white($opacity) !important;
  box-shadow: 0 4px 24px 0 rgba(0, 0, 0, 0.3);
  border-radius: 16px;
  transition: transform 0.2s ease-in-out;

  &:hover {
    transform: scale(1.05);
  }
}

/* Estilos optimizados */
.container-fluid {
  padding: 15px;
}

.card-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  height: 100%;
  justify-content: center;
  align-content: flex-start;

  @media (min-width: 768px) {
    flex-wrap: nowrap;
  }
}

.glass-card {
  @include glass-card();
  width: 100%;
  height: auto;
  padding: 25px;
  margin-bottom: 15px;
  cursor: pointer;

  &.large-card { padding: 45px; }

  .huge-icon { max-width: 120px; }
  .huge-text { font-size: 1.8rem; }
}

.background-options {
  display: flex;
  flex-direction: column;
  gap: 10px;

  .background-option {
    display: flex;
    align-items: center;
    padding: 8px;
    cursor: pointer;
    background: white(02);
    border-radius: 8px;
    transition: background 0.3s ease;

    &:hover { background: white(03); }

    .preview {
      width: 30px;
      height: 30px;
      border-radius: 4px;
      margin-right: 10px;
    }
  }
}

.file-upload-form {
  width: fit-content;
  height: fit-content;

  .file-upload-label {
    cursor: pointer;
    background-color: #ddd;
    padding: 30px 70px;
    border-radius: 40px;
    border: 2px dashed #525252;

    .file-upload-design {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 5px;

      svg { height: 50px; fill: #525252; margin-bottom: 20px; }

      .browse-button {
        background-color: #525252;
        padding: 5px 15px;
        border-radius: 10px;
        color: white;
        transition: background 0.3s;

        &:hover { background-color: #0e0e0e; }
      }
    }
  }
}

.centered-file-upload {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 300px;
}

.success-message { color: green; }
</style>