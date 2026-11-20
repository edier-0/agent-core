---
name: frontend-ui-engineering
description: Construye interfaces de usuario profesionales, accesibles y responsivas. Usar al crear o modificar componentes, vistas, modales o flujos de estado en frontend.
---

# Frontend UI Engineering — Guía de Ingeniería de Interfaz de Usuario

Guía canónica de ingeniería de interfaz de usuario para aplicaciones web modernas. Diseñada para construir vistas y componentes que sean altamente usables, accesibles conforme a estándares WCAG, visualmente pulidos y optimizados tanto para dispositivos móviles como para pantallas de escritorio.

---

## 1. Principios de Arquitectura Frontend

- **Separación de Responsabilidades:** Vistas delgadas que delegan el estado a almacenes (*stores*) y la comunicación a un cliente HTTP centralizado.
- **Tipado Estricto:** Props, eventos emitidos y DTOs de API tipados rigurosamente con TypeScript.
- **Accesibilidad Primero (a11y):** Estructura semántica, contraste de color WCAG AA ($\ge 4.5:1$), soporte de navegación por teclado y etiquetas `aria-*` descriptivas.
- **Diseño Responsivo (*Mobile-First*):** Interfaces fluidas que operan sin desbordamientos desde pantallas móviles (375 px) hasta monitores ultrapanorámicos.

---

## 2. La Regla de los Modales (3 Capas Fijas)

Todo modal operativo o formulario superpuesto debe estructurarse obligatoriamente en **3 capas fijas** para erradicar saltos visuales y desbordamientos en pantallas móviles:

```html
<template>
  <div class="fixed inset-0 z-50 flex items-center justify-center bg-black/60 p-4">
    <div class="flex max-h-[90vh] w-full max-w-lg flex-col rounded-xl bg-white shadow-2xl">
      
      <!-- CAPA 1: Cabecera Fija -->
      <header class="flex shrink-0 items-center justify-between border-b px-6 py-4">
        <h2 class="text-lg font-semibold text-gray-900">{{ title }}</h2>
        <button 
          @click="emit('close')" 
          class="rounded-lg p-2 text-gray-400 hover:bg-gray-100 hover:text-gray-600"
          aria-label="Cerrar modal"
        >
          <XIcon class="h-5 w-5" />
        </button>
      </header>

      <!-- CAPA 2: Cuerpo Desplazable (Scrollable Body) -->
      <div class="flex-1 overflow-y-auto min-h-0 px-6 py-4 space-y-4">
        <slot />
      </div>

      <!-- CAPA 3: Pie de Acción Fijo -->
      <footer class="flex shrink-0 items-center justify-end gap-3 border-t bg-gray-50 px-6 py-4 rounded-b-xl">
        <button 
          type="button" 
          @click="emit('close')" 
          class="rounded-lg border px-4 py-2 text-sm font-medium text-gray-700 hover:bg-gray-100"
        >
          Cancelar
        </button>
        <button 
          type="submit" 
          :disabled="loading" 
          class="rounded-lg bg-primary-600 px-4 py-2 text-sm font-medium text-white hover:bg-primary-700 disabled:opacity-50"
        >
          {{ loading ? 'Guardando...' : 'Confirmar' }}
        </button>
      </footer>

    </div>
  </div>
</template>
```

> **Invariante:** La combinación `flex-1 overflow-y-auto min-h-0` en el cuerpo asegura que los inputs y listas largas nunca empujen los botones de acción fuera de la pantalla en dispositivos móviles.

---

## 3. Estados de Interfaz Obligatorios en Toda Vista

Ninguna vista o componente de listado/detalle puede considerarse terminado sin manejar explícitamente los 4 estados de ciclo de vida:

1. **Estado de Carga (*Loading State*):** Mostrar Skeleton loaders o indicadores de progreso mientras la petición de red se resuelve; nunca dejar una pantalla en blanco.
2. **Estado Vacío (*Empty State*):** Si no hay registros (ej. sin órdenes registradas o sin resultados en el filtro), mostrar una ilustración o ícono, un mensaje explicativo y un botón de acción principal ("Crear Nuevo Registro").
3. **Estado de Error (*Error State*):** Mostrar alertas contextuales con mensajes claros y opción de reintentar la operación (`Reintentar`).
4. **Estado de Éxito (*Data State*):** Presentación clara de datos con tipografía legible y formateo numérico y de fechas consistente.

---

## 4. Responsividad Móvil y Usabilidad Táctil

- **Objetivos Táctiles (*Tap Targets*):** Botones y elementos interactivos deben medir al menos **44×44 px** (`h-11`, `p-3`).
- **Contraste de Color:** Ratios de contraste $\ge 4.5:1$ conforme a WCAG AA para asegurar legibilidad bajo condiciones de alta luminosidad.
- **Teclados Virtuales Móviles:** Usar tipos de input semánticos:
  - `type="number"` o `inputmode="numeric"` para cantidades y dinero.
  - `type="tel"` para números de teléfono.
  - `type="email"` para correos.
  - `type="date"` con control de zona horaria local.

---

## 5. Patrón Canónico de Consumo de APIs

```typescript
<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { apiClient } from '@/api/client';

const items = ref<Item[]>([]);
const loading = ref(true);
const error = ref<string | null>(null);

const fetchData = async () => {
  loading.value = true;
  error.value = null;
  try {
    const response = await apiClient.get('/items');
    items.value = response.data.data;
  } catch (err: any) {
    error.value = err.response?.data?.error || 'Error al cargar los datos';
  } finally {
    loading.value = false;
  }
};

onMounted(fetchData);
</script>
```

---

## 6. Verificación de Calidad Frontend
Antes de dar por completado un cambio en la interfaz:
- [ ] Compilación y empaquetado sin errores de tipos (`BUILD_COMMAND`).
- [ ] Verificado en viewport móvil (375px) y escritorio (1280px).
- [ ] Modales cumplen la estructura de 3 capas sin salto de scroll.
- [ ] Textos, mensajes y modales 100% en español.
- [ ] Formularios con validación en cliente antes del envío.
