<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Gestión de Tareas</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Tareas 1</ion-title>
        </ion-toolbar>
      </ion-header>

      <ion-list>
        <ion-item v-for="tarea in Tarea" :key="tarea.id" class="tarea-item">
          <div class="avatar">
            <div class="avatar-text">{{ tarea.text.substring(0, 2).toUpperCase() }}</div>
          </div>
          <ion-label :style="{ textDecoration: tarea.completed ? 'line-through' : 'none' }">
            {{ tarea.text }}
          </ion-label>
          <ion-label color="medium">
            {{ tarea.completed ? 'Completada' : 'No Completada' }}
          </ion-label>
          <ion-checkbox v-model="tarea.completed" @ionChange="cambiarEstadoTarea(tarea.id)" />
          <div class="buttons">
            <ion-button color="warning" @click="editarTarea(tarea)">Editar</ion-button>
            <ion-button color="danger" @click="eliminarTarea(tarea.id)">Eliminar</ion-button>
          </div>
        </ion-item>
      </ion-list>

      <ion-footer>
        <ion-item>
          <ion-input
            placeholder="Nueva tarea"
            v-model="taskInput"
          />
          <ion-button @click="agregarTarea">
            Agregar
          </ion-button>
        </ion-item>
      </ion-footer>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { defineComponent, ref, onMounted } from 'vue';
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonList,
  IonItem,
  IonLabel,
  IonInput,
  IonButton,
  IonCheckbox,
  IonFooter,
} from '@ionic/vue';
import { Preferences } from '@capacitor/preferences';

interface Tarea {
  id: string;
  text: string;
  completed: boolean;
}

const Tarea = ref<Tarea[]>([]);
const taskInput = ref('');

onMounted(() => {
  cargarTareas();
});

const cargarTareas = async () => {
  const { value } = await Preferences.get({ key: 'Tarea' });
  if (value) {
    Tarea.value = JSON.parse(value);
  }
};

const guardarTareas = async (Tarea: Tarea[]) => {
  await Preferences.set({ key: 'Tarea', value: JSON.stringify(Tarea) });
};

const agregarTarea = () => {
  if (taskInput.value.trim()) {
    const nuevaTarea: Tarea = {
      id: new Date().toISOString(),
      text: taskInput.value,
      completed: false,
    };
    Tarea.value = [...Tarea.value, nuevaTarea];
    guardarTareas(Tarea.value);
    taskInput.value = '';
  }
};

const cambiarEstadoTarea = (taskId: string) => {
  Tarea.value = Tarea.value.map((tarea) =>
    tarea.id === taskId ? { ...tarea, completed: !tarea.completed } : tarea
  );
  guardarTareas(Tarea.value);
};

const eliminarTarea = (taskId: string) => {
  Tarea.value = Tarea.value.filter((tarea) => tarea.id !== taskId);
  guardarTareas(Tarea.value);
};

const editarTarea = (tarea: Tarea) => {
  taskInput.value = tarea.text;
  eliminarTarea(tarea.id);
};
</script>

<style scoped>
.tarea-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.avatar {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #d9d9d9;
  margin-right: 10px;
}

.avatar-text {
  font-weight: bold;
  color: #555;
}

.buttons {
  display: flex;
  gap: 5px;
}

ion-button[color="primary"] {
  --background: #ff5252;
  --background-activated: #ff1744;
}

ion-button[color="warning"] {
  --background: #ffca28;
  --background-activated: #ffc107;
}

ion-button[color="danger"] {
  --background: #f44336;
  --background-activated: #d32f2f;
}
</style>