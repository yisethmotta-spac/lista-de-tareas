<script setup lang="ts">
import { ref } from 'vue'; 

type Prioridad = 'baja' | 'media' | 'alta';

//Define emits le dice a Vue que eventos puede enviar este componente hacia arriba
const emit = defineEmits<{
  (e: 'Agregar', texto: string, prioridad: Prioridad, fechaVencimiento: string): void;
}>();

const textoNuevaTarea = ref('');
const prioridadSeleccionada = ref<Prioridad>('media'); // Valor por defecto
const fechaSeleccionada = ref('');

const manejarEnvio = () => {
  if (textoNuevaTarea.value.trim() === '') return;
  //En vez de agregarla aquí, emite el evento con el texto y la prioridad para que App.vue lo reciba y agregue la tarea a la lista
  emit('Agregar', textoNuevaTarea.value, prioridadSeleccionada.value, fechaSeleccionada.value);
  textoNuevaTarea.value = '';
  prioridadSeleccionada.value = 'media'; // Reinicia la prioridad a la predeterminada
  fechaSeleccionada.value = '';
};
</script>

<template>
  <div class="entrada-datos">
    <input
      v-model="textoNuevaTarea"
      @keyup.enter="manejarEnvio"
      type="text"
      placeholder="¿Qué necesitas hacer hoy?"
    />
    
    <!-- Selector de Prioridad -->
    <select v-model="prioridadSeleccionada" class="select-prioridad">
      <option value="baja">🟢 Baja</option>
      <option value="media">🟡 Media</option>
      <option value="alta">🔴 Alta</option>
    </select>

    <!--Input para la Fecha de Vencimiento -->
    <input 
      v-model="fechaSeleccionada" 
      type="date" 
      class="input-fecha"
      title="Fecha de vencimiento"
    />

    <button class="btn-agregar" @click="manejarEnvio">Agregar</button>
  </div>
</template>

<style scoped>
.entrada-datos {
  display: flex;
  gap: 8px;
  margin-bottom: 20px;
  flex-wrap: wrap; /* Mantiene la flexibilidad si la pantalla es estrecha */
}

input[type="text"] {
  flex: 1;
  padding: 10px;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  font-size: 0.95em;
  background: var(--bg-input, white);
  color: var(--color-titulo, #2c3e50);
  transition: background 0.3s, color 0.3s, border-color 0.3s;
}

input[type="text"]::placeholder {
  color: var(--color-subtexto, #94a3b8);
}

.select-prioridad,
.input-fecha {
  padding: 10px;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  background: var(--bg-input, white);
  color: var(--color-titulo, #2c3e50);
  cursor: pointer;
  font-size: 0.9em;
  transition: background 0.3s, color 0.3s, border-color 0.3s;
}

button.btn-agregar {
  padding: 10px 16px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
}

button.btn-agregar:hover {
  background-color: #33a06f;
}
</style>