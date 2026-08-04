<script setup lang="ts">// Aquí irá la lógica de TypeScript
import { ref, computed, watch, onMounted } from 'vue';
//Importar los componentes que creamos para usarlos en App.vue
import TaskForm from './components/TaskForm.vue';
import TaskItem from './components/TaskItem.vue';

type Prioridad = 'baja' | 'media' | 'alta';

//Definir que datos tiene una tarea
interface Tarea {
  id: number;
  texto: string;
  completada: boolean;
  prioridad?: Prioridad;
  archivada?: boolean; // Saber si la tarea está en el historial
}

//Variables reactivas
const nuevaTarea = ref(''); //guarda lo que el usuario escribe en el input
const tareas = ref<Tarea[]>([]); //Lista que guarda todas las tareas
const filtro = ref<'todas'| 'pendientes' | 'completadas'| 'archivadas'>('todas'); //Para saber qué el filtro está activo

//Estado para la barra de búsqueda
const busqueda = ref('');

//Estado para controlar el Modo Oscuro
const modoOscuro = ref(false);

const cambiarModoOscuro = () => {
  modoOscuro.value = !modoOscuro.value;
  localStorage.setItem('modo-oscuro', JSON.stringify(modoOscuro.value));
  actualizarClaseBody();
};

const actualizarClaseBody = () => {
  if (modoOscuro.value) {
    document.body.classList.add('dark');
  } else {
    document.body.classList.remove('dark');
  }
};

//Persistencia con localStorage
onMounted(() => {
  const tareasGuardadas = localStorage.getItem('tareas');
  if (tareasGuardadas) {
    tareas.value = JSON.parse(tareasGuardadas);
  }

  //Cargar la preferencia de Modo Oscuro al iniciar
  const modoOscuroGuardado = localStorage.getItem('modo-oscuro');
  if (modoOscuroGuardado) {
    modoOscuro.value = JSON.parse(modoOscuroGuardado);
    actualizarClaseBody();
  }
});

watch(tareas, (nuevasTareas) => {
  localStorage.setItem('tareas', JSON.stringify(nuevasTareas));
}, { deep: true });

//Función para añadir una nueva tarea a la lista
const agregarTarea = (textoNuevo: string, prioridadNueva: Prioridad = 'media') => {
  if (textoNuevo.trim() === '') return;
  tareas.value.push({
    id: Date.now(),
    texto: textoNuevo,
    completada: false,
    prioridad: prioridadNueva,
    archivada: false
  });
  nuevaTarea.value = '';
};

//Función para eliminar una tarea de la lista
const eliminarTarea = (id: number) => {
  tareas.value = tareas.value.filter(tarea => tarea.id !== id);
}; 

//Función para manejar el checkbox desde taskItem.vue
const alternarEstadoTarea = (id: number) => {
  const tarea = tareas.value.find(t => t.id === id);
  if (tarea && !tarea.archivada) {
    tarea.completada = !tarea.completada;
  }
};

//Función para editar el texto de una tarea existente
const editarTarea = (id: number, nuevoTexto: string) => {
  const tarea = tareas.value.find(t => t.id === id);
  if (tarea) {
    tarea.texto = nuevoTexto;
  }
};

//Función para marcar/desmarcar todas las tareas activas
const marcarTodasCompletadas = () => {
  const activas = tareas.value.filter(t => !t.archivada);
  const todasHechas = activas.every(t => t.completada);
  activas.forEach(t => t.completada = !todasHechas);
};

//Función para mover al historial/archivar las tareas completadas
const archivarCompletadas = () => {
  tareas.value.forEach(t => {
    if (t.completada) {
      t.archivada = true;
    }
  });
};

// Mapa para asignar peso interno a cada prioridad y poder ordenarlas (Alta -> Media -> Baja)
const pesoPrioridad: Record<Prioridad, number> = {
  alta: 3,
  media: 2,
  baja: 1
};

//Propiedades computadas
//Filtra por estado, búsqueda Y ordena por prioridad
const tareasFiltradas = computed(() => {
  const filtradas = tareas.value.filter(t => {
    let coincideFiltro = false;

    if (filtro.value === 'todas') {
      coincideFiltro = !t.archivada;
    } else if (filtro.value === 'pendientes') {
      coincideFiltro = !t.completada && !t.archivada;
    } else if (filtro.value === 'completadas') {
      coincideFiltro = t.completada && !t.archivada;
    } else if (filtro.value === 'archivadas') {
      coincideFiltro = !!t.archivada;
    }

    const coincideBusqueda = t.texto.toLowerCase().includes(busqueda.value.toLowerCase());

    return coincideFiltro && coincideBusqueda;
  });

  // Ordena las tareas para que Alta quede arriba, luego Media y al final Baja
  return filtradas.sort((a, b) => {
    const pesoA = pesoPrioridad[a.prioridad || 'media'];
    const pesoB = pesoPrioridad[b.prioridad || 'media'];
    return pesoB - pesoA;
  });
});

const tareasPendientes = computed(() => {
  return tareas.value.filter(t => !t.completada && !t.archivada).length;
});

// Saber si hay al menos una tarea completada activa para mostrar el botón de archivar
const hayCompletadasActivas = computed(() => {
  return tareas.value.some(t => t.completada && !t.archivada);
});

//Calcula el porcentaje de avance de las tareas activas
const porcentajeProgreso = computed(() => {
  const activas = tareas.value.filter(t => !t.archivada);
  if (activas.length === 0) return 0;
  const completadas = activas.filter(t => t.completada).length;
  return Math.round((completadas / activas.length) * 100);
});
</script>

<template>
  <!-- Le aplicamos dinámicamente la clase 'dark' cuando el modo oscuro está activo -->
  <div :class="['app-wrapper', { dark: modoOscuro }]">
    <div class="contenedor">
      
      <!-- Encabezado con título y botón para alternar el Modo Oscuro -->
      <div class="encabezado">
        <h1>Lista de Tareas</h1>
        <button class="btn-tema" @click="cambiarModoOscuro" title="Cambiar modo claro/oscuro">
          {{ modoOscuro ? '☀️' : '🌙' }}
        </button>
      </div>

      <!-- NUEVO: Barra de progreso visual % -->
      <div class="seccion-progreso" v-if="tareas.length > 0">
        <div class="info-progreso">
          <span>Progreso</span>
          <strong>{{ porcentajeProgreso }}%</strong>
        </div>
        <div class="barra-fondo">
          <div class="barra-relleno" :style="{ width: porcentajeProgreso + '%' }"></div>
        </div>
      </div>

      <TaskForm @agregar="agregarTarea"/>

      <!-- NUEVO: Input para buscar tareas -->
      <div class="caja-busqueda" v-if="tareas.length > 0">
        <input 
          v-model="busqueda" 
          type="text" 
          placeholder="🔍 Buscar tarea..." 
        />
      </div>

       <div class="filtros">
        <button :class="{ activo: filtro === 'todas' }" @click="filtro = 'todas'">
          Todas </button>
        <button :class="{ activo: filtro === 'pendientes' }" @click="filtro = 'pendientes'">
          Pendientes </button>
        <button :class="{ activo: filtro === 'completadas' }" @click="filtro = 'completadas'">
          Completadas </button>
        <button :class="{ activo: filtro === 'archivadas' }" @click="filtro = 'archivadas'">
          Historial 📦 </button>
      </div>

      <!--Reemplaza <ul> por <TransitionGroup tag="ul"> para animar la lista -->
      <TransitionGroup name="animacion-lista" tag="ul">
        <!-- Ahora le escuchamos el evento @editar a TaskItem -->
        <TaskItem 
          v-for="tarea in tareasFiltradas" 
          :key="tarea.id" 
          :tarea="tarea"
          @eliminar="eliminarTarea"
          @cambiar-estado="alternarEstadoTarea"
          @editar="editarTarea"
        />
      </TransitionGroup>

      <div class="pie-pagina">  
        <p v-if="tareas.length === 0" class="mensaje-vacio">
          No hay tareas pendientes. ¡Relájate o añade una nueva! ☕
        </p>
        <p v-else-if="tareasFiltradas.length === 0" class="mensaje-vacio">
          No se encontraron tareas en esta sección 🔍
        </p>

        <!-- Panel con botones para acciones masivas -->
        <div v-else class="panel-inferior">
          <p class="estadisticas">
            Te quedan <strong>{{ tareasPendientes }} </strong> tareas por terminar.
          </p>

          <div class="acciones-masivas">
            <button class="btn-accion" @click="marcarTodasCompletadas" title="Marcar todas como completadas">
              ✓ Marcar todas
            </button>
            <button 
              v-if="hayCompletadasActivas" 
              class="btn-accion btn-limpiar" 
              @click="archivarCompletadas"
              title="Mover tareas hechas al Historial"
            >
              📦 Archivar hechas
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>

/*Envoltorio para manejar los colores globales claro/oscuro */
.app-wrapper {
  min-height: 100vh;
  padding: 10px;
  background-color: #f1f5f9;
  transition: background-color 0.3s ease;
  --bg-tarjeta: white;
  --bg-item: #f8fafc;
  --text-item: #334155;
  --bg-input: white;
  --color-titulo: #2c3e50;
  --color-subtexto: #64748b;
}

/* Variables cuando activa el Modo Oscuro */
.app-wrapper.dark {
  background-color: #0f172a;
  --bg-tarjeta: #1e293b;
  --bg-item: #334155;
  --text-item: #f1f5f9;
  --bg-input: #0f172a;
  --color-titulo: #f8fafc;
  --color-subtexto: #94a3b8;
}

.contenedor {
  max-width: 500px;
  margin: 40px auto;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: var(--bg-tarjeta);
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.05);
  transition: background 0.3s ease;
}

.encabezado {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

h1 {
  text-align: center;
  color: var(--color-titulo);
  margin: 0;
}

/*Estilos para la barra de progreso */
.seccion-progreso {
  margin-bottom: 20px;
}

.info-progreso {
  display: flex;
  justify-content: space-between;
  font-size: 0.85em;
  color: var(--color-subtexto);
  margin-bottom: 5px;
}

.barra-fondo {
  width: 100%;
  height: 8px;
  background: #e2e8f0;
  border-radius: 10px;
  overflow: hidden;
}

.barra-relleno {
  height: 100%;
  background: #42b883;
  transition: width 0.4s ease;
}

/*Estilos para la caja de búsqueda */
.caja-busqueda {
  margin-bottom: 15px;
}

.caja-busqueda input {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  background: var(--bg-input);
  color: var(--color-titulo);
  box-sizing: border-box;
  transition: background 0.3s, color 0.3s;
}

/*Estilo para el botón de luna/sol */
.btn-tema {
  background: transparent;
  border: none;
  font-size: 1.3em;
  cursor: pointer;
  padding: 5px;
  border-radius: 50%;
  transition: transform 0.2s;
}

.btn-tema:hover {
  transform: scale(1.2);
}

.filtros {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.filtros button {
  padding: 6px 12px;
  background: transparent;
  color: var(--color-subtexto);
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  cursor: pointer;
}

.filtros button:hover {
  background: #f1f5f9;
}

.filtros button.activo {
  background: #42b883;
  color: white;
  border-color: #42b883;
}

ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.pie-pagina {
  margin-top: 25px;
  text-align: center;
  font-size: 0.95em;
  color: var(--color-subtexto);
  border-top: 1px solid #e2e8f0;
  padding-top: 15px;
}

/*Estilos para acciones masivas */
.panel-inferior {
  display: flex;
  flex-direction: column;
  gap: 10px;
  align-items: center;
}

.panel-inferior .estadisticas {
  margin: 0;
}

.acciones-masivas {
  display: flex;
  gap: 10px;
}

.btn-accion {
  background: transparent;
  border: 1px solid #cbd5e1;
  color: var(--color-subtexto);
  padding: 6px 12px;
  border-radius: 6px;
  font-size: 0.85em;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-accion:hover {
  background: rgba(66, 184, 131, 0.1);
  border-color: #42b883;
  color: #42b883;
}

.btn-accion.btn-limpiar:hover {
  background: rgba(59, 130, 246, 0.1);
  border-color: #3b82f6;
  color: #3b82f6;
}

/*Reglas de animación para la lista */
.animacion-lista-enter-active,
.animacion-lista-leave-active {
  transition: all 0.3s ease;
}

.animacion-lista-enter-from {
  opacity: 0;
  transform: translateY(-15px);
}

.animacion-lista-leave-to {
  opacity: 0;
  transform: translateX(25px);
}

.animacion-lista-move {
  transition: transform 0.3s ease;
}
</style>

<style>
body {
  margin: 0;
  background-color: #f1f5f9;
  transition: background-color 0.3s ease;
  --bg-input: white;
  --bg-item: #f8fafc;
  --text-item: #334155;
}

body.dark {
  background-color: #0f172a;
  --bg-tarjeta: #1e293b;
  --bg-item: #334155;
  --bg-input: #0f172a;
  --color-titulo: #f8fafc;
  --text-item: #f1f5f9;
  --color-subtexto: #94a3b8;
}
</style>