<script setup lang="ts">
import { ref } from 'vue';

type Prioridad = 'baja' | 'media' | 'alta';

//Define la interfaz aquí también para que el componente sepa cómo luce una tarea
// (La prioridad es opcional '?' para soportar tareas viejas que no la tenían)
interface Tarea {
  id: number;
  texto: string;
  completada: boolean;
  prioridad?: Prioridad;
  archivada?: boolean; // Saber si la tarea está en el historial
}

//DefineProps recibe los datos desde App.vue
const props = defineProps<{
  tarea: Tarea;
}>();

//Avisa a App.vue si el usuario quiere eliminar o marcar la tarea
const emit = defineEmits<{
  (e: 'eliminar', id: number): void;
  (e: 'cambiar-estado', id: number): void;
  (e: 'editar', id: number, nuevoTexto: string): void;
}>();

// Estado local para controlar si estamos editando esta tarea en específico
const editando = ref(false);
const textoEditado = ref(props.tarea.texto);

// Si la tarea no tiene prioridad asignada aún, usa 'media' por defecto para evitar fallos
const prioridadEfectiva = props.tarea.prioridad || 'media';

const activarEdicion = () => {
  if (props.tarea.archivada) return; // No permitir edición si está archivada   
  editando.value = true;
  textoEditado.value = props.tarea.texto; //Carga el texto actual
};

const guardarEdicion = () => {
  if (textoEditado.value.trim() === '') return; // No permitir texto vacío
  emit('editar', props.tarea.id, textoEditado.value);
  editando.value = false;
};

const cancelarEdicion = () => {
  editando.value = false;
  textoEditado.value = props.tarea.texto; // Restablece el texto original
};
</script>

<template>
<!-- Se le asigna una clase según la prioridad para cambiar el borde de color -->
  <li :class="['item-tarea', prioridadEfectiva, {archivada: tarea.archivada}]" >
    
    <!-- Solo muestra el checkbox si la tarea no está archivada -->
    <input 
      v-if="!tarea.archivada"
      type="checkbox" 
      :checked="tarea.completada" 
      @change="emit('cambiar-estado', tarea.id)" 
    />
    <!--Icono de caja de archivo-->
    <span v-else class="icono-archivo" title="Tarea en historial">📦</span>

    <!-- Si está en modo edición muestra este input -->
    <div v-if="editando" class="modo-edicion">
      <input 
        v-model="textoEditado" 
        type="text" 
        @keyup.enter="guardarEdicion"
        @keyup.esc="cancelarEdicion"
      />
      <button class="btn-guardar" @click="guardarEdicion">💾</button>
      <button class="btn-cancelar" @click="cancelarEdicion">❌</button>
    </div>

    <!-- Modo normal (mostrar texto y botones) -->
    <template v-else>
      <span :class="{ tachado: tarea.completada || tarea.archivada }">
        {{ tarea.texto }}
      </span>

      <span :class="['insignia-prioridad', prioridadEfectiva]">
        {{ prioridadEfectiva.toUpperCase() }}
      </span>

      <!-- NUEVO: Oculta los botones de editar y eliminar cuando la tarea está archivada -->
      <template v-if="!tarea.archivada">
        <button class="btn-editar" @click="activarEdicion">✏️</button>
        <button class="btn-eliminar" @click="emit('eliminar', tarea.id)">🗑️</button>
      </template>
    </template>
  </li>
</template>

<style scoped>
.item-tarea {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 15px;
  background: var(--bg-item, #f8fafc);
  margin-bottom: 10px;
  border-radius: 8px;
  border-left: 5px solid #cbd5e1;
  transition: background 0.3s, transform 0.2s;
}

.item-tarea.baja { border-left-color: #22c55e; }
.item-tarea.media { border-left-color: #eab308; }
.item-tarea.alta { border-left-color: #ef4444; }

.item-tarea.archivada {
  opacity: 0.7;
  border-left-color: #94a3b8;
  background: rgba(0, 0, 0, 0.03);
}

.icono-archivo {
  font-size: 0.9em;
  flex: none;
}

.item-tarea:hover {
  transform: translateY(-1px);
}

/* Checkbox estilizado y adaptado al tema */
input[type="checkbox"] {
  width: 18px;
  height: 18px;
  cursor: pointer;
  accent-color: #42b883;
}

span {
  flex: 1;
  font-size: 1.05em;
  color: var(--text-item, #334155);
}

.tachado {
  text-decoration: line-through;
  color: #94a3b8;
}

.insignia-prioridad {
  font-size: 0.7em;
  font-weight: bold;
  padding: 3px 8px;
  border-radius: 12px;
  text-transform: uppercase;
  flex: none;
}

.insignia-prioridad.baja { background: #dcfce7; color: #15803d; }
.insignia-prioridad.media { background: #fef9c3; color: #a16207; }
.insignia-prioridad.alta { background: #fee2e2; color: #b91c1c; }

.modo-edicion {
  display: flex;
  flex: 1;
  gap: 5px;
}

.modo-edicion input {
  flex: 1;
  padding: 6px;
  border: 1px solid #42b883;
  border-radius: 4px;
  background: var(--bg-input, white);
  color: var(--color-titulo, #2c3e50);
}

button {
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 6px;
  border-radius: 4px;
  font-size: 0.9em;
  transition: background 0.2s;
}

button:hover {
  background: rgba(0,0,0,0.08);
}
</style>