<template>
  <div class="container-fluid h-100">
    <div class="card-grid h-100">
      <!-- Card 1 -->
      <button class="glass-card large-card grow full-height" @click="openPopup('perfil')">
        <div class="card-body text-center d-flex flex-column justify-content-center align-items-center">
          <img src="../assets/images/icons/profile.svg" alt="Foto de perfil" class="img-fluid mb-3 huge-icon">
          <p class="card-text huge-text">Foto de perfil</p>
        </div>
      </button>
      <!-- Card 2 -->
      <button class="glass-card large-card grow full-height" @click="openPopup('pwd')">
        <div class="card-body text-center d-flex flex-column justify-content-center align-items-center">
          <img src="../assets/images/icons/password.svg" alt="Contraseña" class="img-fluid mb-3 huge-icon">
          <p class="card-text huge-text">Contraseña</p>
        </div>
      </button>
      <!-- Card 3 -->
      <button class="glass-card large-card grow full-height" @click="openPopup('fondo')">
        <div class="card-body text-center d-flex flex-column justify-content-center align-items-center">
          <img src="../assets/images/icons/wallpa.svg" alt="Fondo de pantalla" class="img-fluid mb-3 huge-icon">
          <p class="card-text huge-text">Fondo de pantalla</p>
        </div>
      </button>
    </div>
    <!-- Popups (sin cambios mayores) -->
    <GlassmorphicPopup
      :visible="popupVisible"
      :title="popupTitle"
      :subtitle="popupSubtitle"
      :closeButtonText="popupCloseButtonText"
      :actionButtonText="popupActionButtonText"
      :imageUrl="userImagePath"
      @close="closePopup"
      @action="handleAction"
      @image-changed="updateProfileImage"
    >
      <template #popup-content v-if="popupType === 'perfil'">
        <!-- Nuevo componente de subida de archivo -->
        <div class="centered-file-upload" @dragover.prevent @drop="handleDrop">
           <form class="file-upload-form">
                <label for="file" class="file-upload-label">
                    <div class="file-upload-design">
                    <svg viewBox="0 0 640 512" height="1em">
                        <path
                        d="M144 480C64.5 480 0 415.5 0 336c0-62.8 40.2-116.2 96.2-135.9c-.1-2.7-.2-5.4-.2-8.1c0-88.4 71.6-160 160-160c59.3 0 111 32.2 138.7 80.2C409.9 102 428.3 96 448 96c53 0 96 43 96 96c0 12.2-2.3 23.8-6.4 34.6C596 238.4 640 290.1 640 352c0 70.7-57.3 128-128 128H144zm79-217c-9.4 9.4-9.4 24.6 0 33.9s24.6 9.4 33.9 0l39-39V392c0 13.3 10.7 24 24 24s24-10.7 24-24V257.9l39 39c9.4 9.4 24.6 9.4 33.9 0s9.4-24.6 0-33.9l-80-80c-9.4-9.4-24.6-9.4-33.9 0l-80 80z"
                        ></path>
                    </svg>
                        <span v-if="!uploadSuccess" class="browse-button">Browse file</span>
                     <div v-if="uploadSuccess" class="success-message">
                       Imagen cargada correctamente!
                    </div>
                   </div>
                    <input id="file" type="file" @change="onFileChange" />
                </label>
            </form>
        </div>
      </template>
      <template #popup-content v-else-if="popupType === 'pwd'">
        <label for="current_password">Contraseña actual:</label>
        <input
          type="password"
          id="current_password"
          class="form-control"
          placeholder="Tu contraseña actual"
          v-model="currentPassword"
        />
        <label for="new_password">Contraseña nueva:</label>
        <input
          type="password"
          id="new_password"
          class="form-control"
          placeholder="La contraseña nueva"
          v-model="newPassword"
        />
      </template>
      <template #popup-content v-else-if="popupType === 'fondo'">
        <div class="background-options">
          <div class="background-option" @click="changeBackground('style1')">
            <div class="preview" :style="{ background: '#3E1E68' }"></div>
            <p>Fondo 1</p>
          </div>
          <div class="background-option" @click="changeBackground('style2')">
            <div class="preview" :style="{ background: '#1E3A57' }"></div>
            <p>Fondo 2</p>
          </div>
          <div class="background-option" @click="changeBackground('style3')">
            <div class="preview" :style="{ background: '#3D1E57' }"></div>
            <p>Fondo 3</p>
          </div>
          <!-- Agrega más opciones según sea necesario -->
        </div>
      </template>
      <template #popup-content v-else-if="popupType === 'aviso'">
      </template>
    </GlassmorphicPopup>
    <GlassmorphicPopup
      :visible="popupRespuestaVisible"
      :title="popupTitle"
      :subtitle="popupSubtitle"
      :closeButtonText="popupCloseButtonText"
      :actionButtonText="popupActionButtonText"
      :imageUrl="userImagePath"
      @close="closePopup"
      @action="handleAction"
      @image-changed="updateProfileImage"
    >
      <template #popup-content v-if="popupType === 'imagenCargada'">
      </template>
    </GlassmorphicPopup>
  </div>
</template>

<script setup>
import GlassmorphicPopup from "./GlassmorphicPopup.vue";
import "bootstrap/dist/css/bootstrap.css";
import "bootstrap/dist/js/bootstrap.js";
import { ref, onMounted, computed } from "vue";
import axios from "axios";
import { useToast } from "vue-toastification";

const API_AUTH_URL = import.meta.env.VITE_API_AUTH_URL;
const ME_URL = `${API_AUTH_URL}/auth/me`;
const UPDATE_PROFILE_IMAGE_URL = `${API_AUTH_URL}/image/upload`;
const RESET_PWD_URL = `${API_AUTH_URL}/usuario/reset_password`;

const popupVisible = ref(false);
const popupRespuestaVisible = ref(false);
const popupType = ref(null);
const popupTitle = ref(null);
const popupSubtitle = ref(null);
const popupCloseButtonText = ref(null);
const popupActionButtonText = ref(null);
const userPicture = ref(null);
const userName = ref(null);
const selectedFile = ref(null);
const userLastName = ref(null);
const currentPassword = ref(null);
const newPassword = ref(null);
const titulo = ref(null);
const toast = useToast();
const uploadSuccess = ref(false);

// Estado para rastrear el estilo actual
const currentStyle = ref(null);

const openPopup = (type) => {
  popupVisible.value = true;
  popupType.value = type;
  uploadSuccess.value = false;

  if (type === "perfil") {
    popupTitle.value = "Editar perfil";
    popupCloseButtonText.value = "Cerrar";
    popupActionButtonText.value = "Guardar";
  } else if (type === "pwd") {
    popupTitle.value = "Cambiar contraseña";
    popupCloseButtonText.value = "Cerrar";
    popupActionButtonText.value = "Guardar";
  } else if (type === "fondo") {
    popupTitle.value = "Cambiar fondo";
    popupCloseButtonText.value = "Cerrar";
    popupActionButtonText.value = "Aceptar";
  } else if (type === "aviso") {
    popupVisible.value = true;
    popupTitle.value = titulo.value;
    popupActionButtonText.value = "Volver";
  }
};

const closePopup = () => {
  popupVisible.value = false;
};

const handleAction = async () => {
  if (popupType.value === "perfil") {
    // Handle profile picture update logic here
    await updateProfileImage();
  } else if (popupType.value === "aviso") {
    location.reload();
  } else if (popupType.value === "pwd") {
    await resetPwd();
  } else if (popupType.value === "avisoPwd") {
    console.log("hola");
  } else {
    //alert(`Action clicked on ${popupType.value}`);
  }
  closePopup();
};

const onFileChange = (event) => {
  selectedFile.value = event.target.files[0];
  updateProfileImage()
};

const updateProfileImage = async () => {
    if (!selectedFile.value) return;
  const token = localStorage.getItem("token");
  try {
    const formData = new FormData();
    formData.append("image", selectedFile.value);
    const response_upload = await axios.post(UPDATE_PROFILE_IMAGE_URL, formData, {
      headers: {
        Authorization: `Bearer ${token}`,
        "Content-Type": "multipart/form-data",
      },
    });

    if (response_upload.status !== 200) {
      console.log(response_upload);
      throw new Error(response_upload.error);
    }

    userPicture.value = response_upload.data.path;
    uploadSuccess.value = true;
    toast.success("La imagen ha sido cargada con éxito");
    //location.reload();
  } catch (error) {
    if (error.response) {
      // El servidor respondió con un código de estado fuera del rango 2xx
      console.error("Error response:", error.response.data);
      alert(
        `Failed to upload profile image: ${
          error.response.data.message || error.response.data
        }`
      );
    } else if (error.request) {
      // La solicitud fue hecha pero no se recibió respuesta
      console.error("Error request:", error.request);
      alert("Failed to upload profile image: No response from server.");
    } else {
      // Algo pasó al configurar la solicitud que desencadenó un error
      console.error("Error message:", error.message);
      alert(`Failed to upload profile image: ${error.message}`);
    }
  }
};

const resetPwd = async () => {
  const token = localStorage.getItem("token");
  try {
    const response = await axios.put(
      RESET_PWD_URL,
      {
        current_password: currentPassword.value,
        new_password: newPassword.value,
      },
      {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      }
    );

    if (response.status !== 200) {
      console.log(response);
      throw new Error(response.error);
    }

    console.log("Contraseña modificada");
    closePopup();
    toast.success("La contraseña ha sido modificada correctamente!");
  } catch (error) {
    if (error.response) {
      // El servidor respondió con un código de estado fuera del rango 2xx
      console.error("Error response:", error.response.data);
      closePopup();
      toast.error(error.response.data.data);
      console.log(`${error.response.data.data || error.response.data}`);
    } else if (error.request) {
      // La solicitud fue hecha pero no se recibió respuesta
      closePopup();
      toast.error("Error request:", error.request);
    } else {
      // Algo pasó al configurar la solicitud que desencadenó un error
      closePopup();
      toast.error("Error message:", error.message);
    }
  }
};

const userImagePath = computed(() => {
  return userPicture.value
    ? `../src/assets/images/userpicture/${userPicture.value}`
    : null;
});

onMounted(async () => {
  const token = localStorage.getItem("token");
  if (token) {
    try {
      const response = await axios.get(ME_URL, {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });
      userName.value = response.data.name;
      userLastName.value = response.data.primer_apellido;
      userPicture.value = response.data.foto_perfil;
    } catch (error) {
      console.error("Error fetching user data:", error);
    }
  }

  const savedBackground = localStorage.getItem("selectedBackground");
  if (savedBackground) {
    changeBackground(savedBackground);
  }
});

// Función para cambiar el fondo
const changeBackground = (styleName) => {
  // Eliminar el estilo anterior si existe
  if (currentStyle.value) {
    const oldStyle = document.getElementById("dynamic-background-style");
    if (oldStyle) {
      oldStyle.remove();
    }
  }

  // Crear un nuevo elemento <link> para cargar el nuevo estilo
  const link = document.createElement("link");
  link.id = "dynamic-background-style";
  link.rel = "stylesheet";
  link.href = `/src/assets/backgrounds/${styleName}.css`; // Ruta al archivo CSS
  document.head.appendChild(link);

  // Guardar la selección en localStorage
  localStorage.setItem("selectedBackground", styleName);

  // Actualizar el estado
  currentStyle.value = styleName;

  // Cerrar el popup
  closePopup();
};

// Función para manejar el arrastre de archivos
const handleDrop = (event) => {
  event.preventDefault();
  selectedFile.value = event.dataTransfer.files[0];
  updateProfileImage()

};
</script>

<style lang="scss" scoped>
$white-01: rgba(255, 255, 255, 0.1);
$white-02: rgba(255, 255, 255, 0.2);
$white-03: rgba(255, 255, 255, 0.3);
$white-05: rgba(255, 255, 255, 0.5);
$white-06: rgba(255, 255, 255, 0.6);
$white-07: rgba(255, 255, 255, 0.7);
$white-08: rgba(255, 255, 255, 0.8);
$white-09: rgba(255, 255, 255, 0.9);

.container-fluid {
  padding: 15px;
}

.card-grid {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between; /* Space items out */
  align-content: flex-start; /* Align items to the start of each line */
  height: 100%;
  gap: 15px;

  @media (min-width: 768px) {
    flex-wrap: nowrap; /* Prevent wrapping on desktop */
    justify-content: center; /* Center cards horizontally on desktop*/
    gap: 15px;
  }
}
@mixin glassmorphic-card(){
  background: rgba($white-09, 0.7) !important;
    backdrop-filter: blur(8px);
    border: 1px solid rgba($white-09, 0.05) !important;
    box-shadow: 0 4px 24px 0 rgba(0, 0, 0, 0.3);
}
.glass-card {
  @include glassmorphic-card();
  border-radius: 16px;
  margin-bottom: 15px;
  width: 100%;
  height: auto;
  padding: 25px;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  cursor: pointer;
  transition: transform 0.2s ease-in-out;

  &:hover {
    transform: scale(1.05);
  }

  .card-text {
    margin-top: 0;
    color: #333;
  }

  img {
    max-width: 50px;
    height: auto;
  }

  &.large-card {
    padding: 45px;
  }

  &.grow {
    flex-grow: 1;
  }

  .huge-icon {
    max-width: 120px;
  }

  .huge-text {
    font-size: 1.8rem;
  }
}

.background-options {
  display: flex;
  flex-direction: column;
  gap: 10px;

  .background-option {
    display: flex;
    align-items: center;
    cursor: pointer;
    background: $white-02;
    border-radius: 8px;
    padding: 8px;
    transition: background 0.3s ease;

    &:hover {
      background: $white-03;
    }

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
  display: flex;
  align-items: center;
  justify-content: center;
}

.file-upload-label input {
  display: none;
}

.file-upload-label svg {
  height: 50px;
  fill: rgb(82, 82, 82);
  margin-bottom: 20px;
}

.file-upload-label {
  cursor: pointer;
  background-color: #ddd;
  padding: 30px 70px;
  border-radius: 40px;
  border: 2px dashed rgb(82, 82, 82);
}

.file-upload-design {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 5px;
}

.browse-button {
  background-color: rgb(82, 82, 82);
  padding: 5px 15px;
  border-radius: 10px;
  color: white;
  transition: all 0.3s;
}

.browse-button:hover {
  background-color: rgb(14, 14, 14);
}

.centered-file-upload {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 300px;
}

.success-message {
  color: green;
  margin-top: 10px;
}
</style>