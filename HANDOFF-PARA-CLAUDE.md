# 🤝 Handoff: Sistema de Cotizaciones — Deseos Viajeros

> **Este documento es para la próxima instancia de Claude** que trabaje con el equipo de Deseos Viajeros.
> Te doy todo el contexto del proyecto para que puedas continuar sin tener que reconstruir la historia.

---

## 1. 🎯 Qué es este proyecto

Sistema web de **cotizaciones para agencia de viajes**. Es una SPA (Single Page App) construida como **un solo archivo HTML** (`cotizacion.html`, ~2800 líneas, ~260 KB) con CSS y JS embebidos. No usa frameworks ni build step.

### Cliente
- **Deseos Viajeros MYX SRL** (Costa Rica)
- Cédula: 3-102-862093
- Teléfono: 8627-5400
- Eslogan: "Aquí inicia tu viaje..."
- Colores corporativos: `#429D9D` (primary), `#027584` (primary-dark), `#00D6D4` (vivid), `#e8941a` (accent)
- Tipografías: Playfair Display (títulos), Open Sans (texto), Dancing Script (eslogan)

### Equipo (5 encargadas de cotización)
- Brenda Gomez
- Xavier Castro
- Maria Perez
- Milady Zuñiga
- Susana Castro

### Estado de propiedad
El dueño original del repo (developer externo) está transfiriendo:
- **Repo de GitHub** a la cuenta de Deseos Viajeros
- **Firebase** se va a crear directamente en la cuenta de ellos
- **Ellos serán dueños de todo**; el developer original y tú trabajarán como colaboradores

---

## 2. 📦 Features ya implementadas

### Editor de Cotización
- 3 formatos: **América, Europa/Asia, Cruceros**
- Selector de destino con **imágenes de encabezado personalizadas por país/destino**
- **Datos del cliente** (persona o empresa) con toggle para mostrar/ocultar en la cotización impresa
- **Hospedaje incluido** con rango de fechas (selector de calendario → texto tipo "del 24 al 27 de junio")
- **Tours incluidos** (texto libre)
- **Itinerario propuesto** tipo Excel (paste desde hoja de cálculo + auto-rellenado de fechas consecutivas)
- **Opciones de hoteles** con tabla de precios por tipo de pasajero (Adulto / Niño 2-11 / Niño 0-23m)
- **Calculadora estilo Excel** por hotel:
  - Filas dinámicas (Boleto, Hotel, Tours, Transporte + extras)
  - Reordenables con drag & drop ☰
  - Fórmulas `+ - * /` y paréntesis
  - **Total redondeado al siguiente múltiplo de 9** (ej: 723 → 729)
  - **Enter para aplicar fórmula** (como Excel)
  - `dataset.formula` guarda la fórmula original para poder re-editarla
  - Auto-aplica al precio de la opción (con override manual)
- **Servicios incluidos** y **No incluye** con checkboxes y drag & drop
- **Políticas de viaje** editables
- **Cuentas bancarias** para pago
- **Logo** y **marca de agua** personalizables
- **Adjuntos internos** (solo para uso interno, no aparecen en PDF del cliente):
  - Upload múltiple
  - **Paste con Ctrl+V** desde portapapeles
  - Categorías (Vuelo, Hospedaje, Tour, Traslado, Seguro, Otro)
- **Preview en vivo** de la cotización como PDF
- **Exportación a PDF** (print CSS optimizado con `@page` y márgenes)

### Módulos CRM
- **Dashboard** con KPIs, gráficos en CSS/SVG puro (barras y donut) y listas accionables
- **Historial de cotizaciones** con filtros por estado/encargada + botón "Ver" (solo lectura), Editar, Duplicar, Aceptada, Eliminar
- **Ventas** (generadas automáticamente al aceptar una cotización):
  - Checklist de tareas (pasaporte, aerolínea, hospedaje, etc.) editable por venta
  - Estados: pendiente / en_proceso / completada / cancelada
  - **Estado de cuenta con registro de pagos**:
    - Monto total + moneda
    - Múltiples pagos con fecha, monto, método, referencia, nota
    - **Editar** y **eliminar** pagos individuales
    - Barra de progreso de pago
    - Badges: Pendiente / Parcial / Pagado
  - Generación de **Recibo de Pago** imprimible por cada pago
  - Generación de **Estado de Cuenta** consolidado imprimible
  - Ambos documentos llevan la imagen del destino como header (igual que la cotización)
- **Clientes** con base de datos completa (nombre, documento, nacionalidad, pasaporte, vencimiento, programa viajero frecuente, empresa, notas) — auto-creados al guardar cotizaciones
- **Destinos** (antes llamado "Encabezados") — gestor de imágenes de encabezado por país/destino con indicador de uso de almacenamiento
- **Configuración** con 8 pestañas editables:
  - Agencia (nombre, teléfono, cédula, eslogan, razón social)
  - Encargados
  - Servicios por formato
  - Políticas por formato
  - No incluye por formato
  - Cuentas bancarias
  - Checklist de ventas (por defecto)
  - Fees de calculadora

### UX
- Sidebar de navegación tipo CRM
- **Responsive mobile** con sidebar off-canvas y hamburger
- **Splitter** entre form panel y preview panel (redimensionable)
- **Lightbox** de imágenes en adjuntos
- **Toast/alerts** con mensajes claros
- **Compresión automática** de imágenes antes de guardar (1200×1200 max)

---

## 3. 🏗️ Arquitectura técnica

### Estructura de archivos
```
cotizacion.html       ← TODO el código está acá (HTML + CSS + JS)
index.html            ← Redirect a cotizacion.html (para GitHub Pages)
README.md
.gitignore
Guia Setup - Deseos Viajeros.docx  ← guía para el equipo
HANDOFF-PARA-CLAUDE.md             ← este documento
```

### localStorage keys (prefijo `dv_`)
| Key | Contenido |
|-----|-----------|
| `dv_cotizaciones` | Array de cotizaciones del historial |
| `dv_encabezados` | Objeto `{pais: {ciudad: dataUrl}}` con imágenes |
| `dv_ventas` | Array de ventas con checklist y cobranza |
| `dv_clientes` | Array de clientes |
| `dv_config` | Objeto de configuración (encargados, servicios, fees, etc.) |
| `dv_form_panel_width` | Ancho del splitter (solo UX) |

### Helpers clave
```js
getLS(k) / setLS(k,v)       // arrays
getLSObj(k) / setLSObj(k,v) // objetos (con try/catch QuotaExceeded)
cfgGet()                    // devuelve config con fallback a defaults
num(v)                      // parsea números tolerando "$1,234.50"
evalExpr(v)                 // evalúa fórmulas seguras (+-*/())
roundToEnding9(n)           // redondea al siguiente múltiplo de 9
esc(t)                      // escape HTML
compressImage(file, ...)    // comprime imágenes antes de guardar
getStorageInfo()            // info del uso de localStorage
```

### Event listeners globales
- `paste` → detecta imágenes en clipboard (Ctrl+V en adjuntos)
- `focusin` → restaura fórmula en celda al enfocarla
- `keydown` → Enter en celda del calc aplica fórmula
- `beforeprint` / `afterprint` → agrega clase `print-cot` o `print-doc` al body para imprimir solo la vista correcta

### Vistas (navegación con `nav('viewName')`)
- `dashboard`
- `editor`
- `historial`
- `encabezados` (id interno, UI dice "Destinos")
- `ventas`
- `clientes`
- `configuracion`
- `documento` (no está en el sidebar, se usa al generar recibo/estado de cuenta)

---

## 4. ⚠️ Reglas importantes para trabajar

### REGLA #1: Validar el JS después de cada cambio grande
Este archivo es gigante y un typo en un template literal rompe TODO el script. Tuve 2 bugs de este tipo que dejaron la app inutilizable. **Siempre** validar antes de pushear:

```bash
python -c "
import re
with open('cotizacion.html','r',encoding='utf-8') as f: c=f.read()
m=re.search(r'<script>(.*?)</script>',c,re.DOTALL)
open('/tmp/test.js','w',encoding='utf-8').write(m.group(1))
"
node --check /tmp/test.js && echo "OK"
```

### REGLA #2: No romper compatibilidad hacia atrás
Las cotizaciones viejas del historial deben seguir funcionando. Cada vez que cambies la estructura de datos, agregar fallbacks. Ejemplos existentes:
- `calc.rows || calc.extraRows` (refactor de calculadora)
- `h.aceptada || false` (campo agregado después)
- `cfg.servicios?.america || defaultServiciosAmerica` (config con fallback)

### REGLA #3: El usuario es no-técnico
- Responder en **español rioplatense natural**
- Evitar jerga técnica innecesaria
- Explicar cambios con ejemplos concretos
- No usar emojis excesivamente en respuestas

### REGLA #4: Siempre pushear a GitHub después de cambios
El flujo es: modificar → validar JS → `git commit` → `git push`. GitHub Pages redeploya automáticamente en 1-2 min.

### REGLA #5: Compresión de imágenes es obligatoria
Nunca guardes imágenes directas en localStorage (o en Firebase Storage tampoco sin límite). Usar siempre `compressImage(file, maxW, maxH, quality)` antes de guardar.

---

## 5. 🚀 Próximo gran paso: MIGRACIÓN A FIREBASE

### Por qué
Con localStorage tenemos:
- ❌ Límite de ~5 MB por navegador
- ❌ Datos aislados por usuaria (Brenda no ve las cotizaciones de Xavier)
- ❌ Sin backup (si se formatea la compu, se pierde todo)

Con Firebase resolvemos los 3 problemas. El equipo ya tiene plan de migración.

### Pre-requisitos (los debe hacer Deseos Viajeros)
Ya les enviamos la guía (`Guia Setup - Deseos Viajeros.docx`). Ellos deben:
1. Crear email corporativo
2. Crear proyecto Firebase (Firestore + Storage + Authentication)
3. Crear cuenta de GitHub
4. Transferir el repo al usuario de ellos
5. Agregar al developer como colaborador en Firebase y GitHub

Cuando confirmen que está todo listo, pueden proceder con la migración.

### Plan de migración (2-3 horas)

**Paso 1: Modularización** (45 min)
Dividir `cotizacion.html` en archivos separados. Esto no solo facilita la migración sino que reduce **drásticamente** el consumo de tokens en sesiones futuras (~80% de reducción).

Estructura propuesta:
```
index.html              (solo estructura HTML)
css/
  styles.css            (todos los estilos)
js/
  firebase-init.js      (configuración Firebase)
  storage.js            (helpers: ahora Firestore/Storage)
  auth.js               (login con Firebase Auth)
  app.js                (nav + init)
  editor.js             (editor de cotización)
  calculadora.js        (calc + fórmulas)
  cobranza.js           (estado de cuenta + pagos)
  ventas.js
  clientes.js
  dashboard.js
  destinos.js
  config.js
```

**Paso 2: Firebase SDK y configuración** (15 min)
- Agregar SDK de Firebase vía CDN
- Inicializar app, Firestore, Storage, Auth
- Configurar reglas de seguridad

**Paso 3: Migrar helpers `getLS/setLS`** (30 min)
Mantener las mismas **firmas** de funciones, solo cambiar la implementación:
```js
// Antes:
const getHistorial = () => getLS('dv_cotizaciones');
// Después:
const getHistorial = async () => await firestore.collection('cotizaciones').get();
```

**IMPORTANTE**: Esto convierte las funciones en asíncronas. Hay que actualizar los call sites con `await`. Alternativamente, mantener una capa de caché local sincronizada con Firestore (más complejo pero mejor UX).

**Paso 4: Subir imágenes a Storage** (30 min)
- Las imágenes ahora se suben a Firebase Storage
- En Firestore solo guardamos **URLs**
- Los datos JSON son mucho más chicos, ya no hay límite de 1MB por documento

**Paso 5: Sistema de login** (30 min)
- Crear 5 cuentas en Firebase Auth (una por encargada)
- Pantalla de login simple al entrar
- Persistencia de sesión
- Botón "Cerrar sesión"
- Filtrar el dashboard por encargado actual

**Paso 6: Reglas de seguridad** (15 min)
```
Firestore:
- Solo usuarios autenticados pueden leer/escribir
- Todos ven todo (compartido entre equipo)

Storage:
- Solo autenticados pueden subir/descargar
- Límite de 10 MB por archivo
```

**Paso 7: Deploy y testing** (15 min)
- Commit + push → GitHub Pages redeploya
- Probar todos los flujos con las 5 cuentas
- Verificar que datos llegan a Firestore

### Estructura propuesta de Firestore
```
cotizaciones/{id}
  ├── fecha: timestamp
  ├── aceptada: boolean
  ├── clienteId: string
  ├── data: { ... objeto con todos los campos ... }
  └── createdBy: userId (nuevo)

ventas/{id}
  ├── cotizacionId: string
  ├── clienteId: string
  ├── destino, clienteNombre, fechaInicio, fechaFin
  ├── estado: string
  ├── checklist: [{id, texto, completado}]
  ├── cobranza: { montoTotal, moneda, pagos: [...] }
  └── fechaCreacion: timestamp

clientes/{id}
  └── { nombreCompleto, documento, pasaporte, ... }

destinos/{pais}
  └── ciudades: { ciudad1: storageUrl, ciudad2: storageUrl }

config/default
  └── { encargados, servicios, politicas, fees, agencyInfo, ... }
```

### Estructura propuesta de Storage
```
destinos/{pais}/{ciudad}.jpg
adjuntos/{cotizacionId}/{timestamp}_{random}.jpg
logos/agencia.png
watermark/agencia.png
```

---

## 6. 🔗 Links importantes

- **Repo GitHub**: https://github.com/tony20154689-ui/cotizaciones-deseos-viajeros (en proceso de transferencia)
- **Deploy actual**: https://tony20154689-ui.github.io/cotizaciones-deseos-viajeros/
- **Documentación de Firebase**: https://firebase.google.com/docs/web/setup
- **Guía de setup para Deseos Viajeros**: archivo `Guia Setup - Deseos Viajeros.docx` en el repo

---

## 7. 💡 Lecciones aprendidas (para evitar que las repitas)

1. **SyntaxErrors en template literals invalidan todo el script** — Pasó 2 veces:
   - `\\'` mal escapado en concatenación de string dentro de template literal
   - Código obsoleto dejó `const x` duplicado después de un refactor
   - **Solución**: siempre `node --check` antes de pushear

2. **localStorage Quota** — Imágenes pesadas llenan los 5MB y los `setItem()` fallan silenciosamente. La compresión automática resolvió el problema, pero sigue siendo un bottleneck. Firebase lo resuelve definitivamente.

3. **`page-break-after: always` causa espacios vacíos en PDF** — Si una sección tiene poco contenido, queda mucho blanco al final. Mejor usar `page-break-after: auto` + `page-break-inside: avoid` en secciones críticas.

4. **El `@media print` con viewDocumento/viewEditor en conflicto** — Al principio ambas vistas se imprimían juntas. La solución fue usar clases en body (`print-cot` / `print-doc`) seteadas por `beforeprint` según qué vista está activa.

5. **Formulas que "no se borran"** — Al guardar `dataset.formula` para poder re-editar, si el usuario borra la celda pero no se limpia el dataset, el cálculo sigue usando la fórmula vieja. Siempre limpiar `dataset.formula` en `oninput` cuando el usuario edita manualmente.

6. **Cotizaciones del historial rompen con cambios de schema** — Siempre agregar fallbacks (`d.nuevaProp || defaultValue`) para datos viejos.

---

## 8. 📞 Contacto del developer original

Cuando necesites clarificaciones sobre decisiones técnicas anteriores, el developer original puede explicarte el "por qué" de cada cosa. El usuario del proyecto (el cliente de Claude para el que trabajo) es quien maneja la relación con Deseos Viajeros y con vos.

---

## 9. 🎯 Tu primera acción sugerida

1. **Lee el archivo `cotizacion.html` completo** (aunque sea en partes) para entender la estructura.
2. **Confirmá con el usuario** que tiene todas las cuentas de Firebase/GitHub listas según la guía.
3. **Pedile el `firebaseConfig`** (objeto con apiKey, authDomain, projectId, etc.) que obtienen desde Firebase Console → Project Settings → Web app.
4. **Proponé el plan de migración** como 1 sesión intensiva de 2-3 horas, con los 7 pasos del punto 5 de este documento.
5. **Hacé backup** de las cotizaciones que pueda tener el usuario en su navegador antes de migrar (exportar localStorage como JSON por las dudas).

---

**Buena suerte, próximo Claude. El proyecto está en buen estado y los dueños están motivados.** 🚀
