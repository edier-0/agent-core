---
name: ui-ux-pro-max
description: Inteligencia y catálogo de diseño UI/UX. 67 estilos, 96 paletas, 57 combinaciones tipográficas, 25 tipos de gráficos y 99 directrices UX para múltiples stacks y Tailwind.
---

# UI/UX Pro Max — Inteligencia y Catálogo de Diseño UI/UX

Guía exhaustiva y motor de búsqueda de diseño para aplicaciones web y móviles. Contiene una base de datos con 67 estilos visuales, 96 paletas de color, 57 combinaciones tipográficas, 99 directrices UX y 25 tipos de gráficos adaptables a múltiples frameworks (Tailwind CSS, Vue, React, Next.js, Svelte, Flutter, etc.).

---

## 1. Requisitos Previos

Verificar que Python esté instalado en la máquina:
```bash
python --version
```

---

## 2. Flujo de Trabajo para Diseño UI/UX

Cuando se requiera diseñar, mejorar o crear una interfaz (dashboard, módulo transaccional, formulario o vista móvil):

### Paso 1: Analizar los Requisitos
Identifica los parámetros de la vista:
- **Tipo de Producto:** Dashboard analítico, módulo POS, SaaS B2B, landing page, app móvil.
- **Palabras Clave de Estilo:** Minimalista, profesional, oscuro, contraste alto, moderno, editorial.
- **Stack:** `html-tailwind`, `vue`, `react`, `nextjs`, `shadcn`, `svelte`, `flutter`.

### Paso 2: Generar el Sistema de Diseño
Ejecuta el buscador para obtener recomendaciones integradas de paleta, tipografía y componentes:
```bash
python .agent/skills/frontend/ui-ux-pro-max/scripts/search.py "operations management dashboard" --design-system -p "App"
```

El script genera:
1. Paleta de colores recomendada con códigos HEX y clases utilitarias.
2. Jerarquía tipográfica adecuada para dashboards y formularios densos.
3. Efectos, sombras y bordes acordes a la identidad visual.
4. Antipatrones visuales a evitar.

### Paso 3: Búsquedas Específicas por Dominio
Para necesidades puntuales dentro de una vista:

| Necesidad | Dominio | Ejemplo de Comando |
| :--- | :--- | :--- |
| **Gráficos de métricas o series temporales** | `chart` | `python .agent/skills/frontend/ui-ux-pro-max/scripts/search.py "sales metrics daily volume" --domain chart` |
| **Directrices de Usabilidad Móvil / a11y** | `ux` | `python .agent/skills/frontend/ui-ux-pro-max/scripts/search.py "mobile touch targets form validation" --domain ux` |
| **Estilos visuales específicos** | `style` | `python .agent/skills/frontend/ui-ux-pro-max/scripts/search.py "clean flat minimal card" --domain style` |

---

## 3. Integración con Frontend UI Engineering

Las recomendaciones extraídas mediante esta skill deben aplicarse directamente dentro de los componentes del frontend respetando los estándares de **`frontend-ui-engineering`** (modales de 3 capas fijas, responsividad móvil, estados de carga y contraste WCAG).
