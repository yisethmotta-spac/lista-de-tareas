<script setup lang="ts">
import { ref, computed } from 'vue'; 

type Prioridad = 'baja' | 'media' | 'alta';
type Categoria= 'general' | 'trabajo' | 'hogar'| 'estudio' | 'compras';

//Define emits le dice a Vue que eventos puede enviar este componente hacia arriba
const emit = defineEmits<{
  (e: 'agregar', texto: string, prioridad: Prioridad, fechaVencimiento: string, categoria: Categoria): void;
}>();

const textoNuevaTarea = ref('');
const prioridadSeleccionada = ref<Prioridad>('media'); // Valor por defecto
const fechaSeleccionada = ref('');
const categoriaSeleccionada = ref<Categoria>('general');
const mensajeError = ref('');

// Obtiene la fecha de hoy en formato YYYY-MM-DD para el atributo min del calendar
const fechaMinima = computed(() => {
  const hoy = new Date();
  const año = hoy.getFullYear();
  const mes = String(hoy.getMonth() + 1).padStart(2, '0');
  const dia = String(hoy.getDate()).padStart(2, '0');
  return `${año}-${mes}-${dia}`;
});

const manejarEnvio = () => {
  mensajeError.value = '';
  if (textoNuevaTarea.value.trim() === '') return;

  // Validación para impedir que se asignen fechas que ya pasaron
  if (fechaSeleccionada.value) {
    const hoy = new Date();
    hoy.setHours(0, 0, 0, 0);

    const [año, mes, dia] = fechaSeleccionada.value.split('-').map(Number);
    const fechaTarea = new Date(año, mes - 1, dia);

    if (fechaTarea < hoy) {
      mensajeError.value = 'No se pueden agregar tareas para un día que ya pasó.';
      return;
    }
  }

  //En vez de agregarla aquí, emite el evento con el texto y la prioridad para que App.vue lo reciba y agregue la tarea a la lista
  emit('agregar', textoNuevaTarea.value, prioridadSeleccionada.value, fechaSeleccionada.value, categoriaSeleccionada.value);
  textoNuevaTarea.value = '';
  prioridadSeleccionada.value = 'media'; // Reinicia la prioridad a la predeterminada
  categoriaSeleccionada.value = 'general'; // Reinicia la categoría a la predeterminada
  fechaSeleccionada.value = '';
  mensajeError.value = '';
};
</script>

<template>
  <div class="contenedor-formulario">
    <div class="entrada-datos">
      <input
        v-model="textoNuevaTarea"
        @keyup.enter="manejarEnvio"
        type="text"
        placeholder="Añade tus tareas aquí"
      />
      
      <!-- Selector de Categoria -->
      <select v-model="categoriaSeleccionada" class="select-opcion">
        <option value="general">📌 General</option>
        <option value="trabajo">💼 Trabajo</option>
        <option value="hogar">🏠 Hogar</option>
        <option value="estudio">📚 Estudio</option>
        <option value="compras">🛒 Compras</option>
      </select>

      <!-- Selector de Prioridad -->
      <select v-model="prioridadSeleccionada" class="select-prioridad">
        <option value="baja">🟢 Baja</option>
        <option value="media">🟡 Media</option>
        <option value="alta">🔴 Alta</option>
      </select>

      <!--Input para la Fecha de Vencimiento -->
      <input 
        v-model="fechaSeleccionada" 
        :min="fechaMinima"
        type="date" 
        class="input-fecha"
        title="Fecha de vencimiento"
        @change="mensajeError = ''"
      />

      <button class="btn-agregar" @click="manejarEnvio">agregar</button>
    </div>

    <!-- Mensaje de advertencia si la fecha es menor al día actual -->
    <p v-if="mensajeError" class="mensaje-error">
      ⚠️ {{ mensajeError }}
    </p>
  </div>
</template>

<style scoped>
.contenedor-formulario {
  margin-bottom: 20px;
}

.entrada-datos {
  display: flex;
  gap: 8px;
  flex-wrap: wrap; /* Mantiene la flexibilidad si la pantalla es estrecha */
}

input[type="text"] {
  flex: 1;
  min-width: 200px;
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
.select-opcion,
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

.mensaje-error {
  color: #ef4444;
  font-size: 0.85em;
  font-weight: 600;
  margin: 6px 0 0 0;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-3px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>