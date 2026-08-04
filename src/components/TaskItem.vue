<script setup lang="ts">
import { ref, computed } from 'vue';

type Prioridad = 'baja' | 'media' | 'alta';

//Define la interfaz aquí también para que el componente sepa cómo luce una tarea
// (La prioridad es opcional '?' para soportar tareas viejas que no la tenían)
interface Tarea {
  id: number;
  texto: string;
  completada: boolean;
  prioridad?: Prioridad;
  archivada?: boolean; // Saber si la tarea está en el historial
  fechaVencimiento?: string;
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

// Evaluación del estado y formato de la fecha de vencimiento
const estadoFecha = computed(() => {
  if (!props.tarea.fechaVencimiento || props.tarea.completada || props.tarea.archivada) {
    return null;
  }

  const hoy = new Date();
  hoy.setHours(0, 0, 0, 0);

  const [año, mes, dia] = props.tarea.fechaVencimiento.split('-').map(Number);
  const fechaTarea = new Date(año, mes - 1, dia);

  if (fechaTarea.getTime() === hoy.getTime()) {
    return { tipo: 'hoy', texto: '📅 Hoy' };
  } else if (fechaTarea < hoy) {
    return { tipo: 'vencida', texto: '⚠️ Vencida' };
  } else {
    return { tipo: 'futura', texto: `📅 ${dia}/${mes}/${año}` };
  }
});

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
  <li :class="['item-tarea', prioridadEfectiva, { archivada: tarea.archivada, vencida: estadoFecha?.tipo === 'vencida' }]">
    
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
      <div class="contenido-tarea">
        <span :class="{ tachado: tarea.completada || tarea.archivada }">
          {{ tarea.texto }}
        </span>

        <!-- Muestra la etiqueta de fecha cuando existe -->
        <span v-if="estadoFecha" :class="['insignia-fecha', estadoFecha.tipo]">
          {{ estadoFecha.texto }}
        </span>
      </div>

      <span :class="['insignia-prioridad', prioridadEfectiva]">
        {{ prioridadEfectiva.toUpperCase() }}
      </span>

      <!--Oculta los botones de editar y eliminar cuando la tarea está archivada -->
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

.item-tarea.vencida {
  border-left-color: #ef4444;
  background: rgba(239, 68, 68, 0.05);
}

.item-tarea.archivada {
  opacity: 0.7;
  border-left-color: #94a3b8;
  background: rgba(0, 0, 0, 0.03);
}

.contenido-tarea {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 2px;
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

.insignia-fecha {
  font-size: 0.75em;
  font-weight: 600;
  width: fit-content;
}

.insignia-fecha.hoy { color: #d97706; }
.insignia-fecha.vencida { color: #dc2626; font-weight: bold; }
.insignia-fecha.futura { color: #64748b; }

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