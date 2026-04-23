# Sistema de Cotizaciones — Deseos Viajeros

Sistema web de cotizaciones y CRM para **Deseos Viajeros MYX SRL** (Costa Rica).

> *"Aquí inicia tu viaje..."*

## 🚀 Acceso

- **Sistema en vivo:** https://sistemadeseos.github.io/deseos-viajeros-sistema/
- **Repositorio:** https://github.com/sistemadeseos/deseos-viajeros-sistema

## ✨ Qué hace

SPA (Single Page App) de un solo archivo HTML con todo embebido (HTML + CSS + JS). No requiere build step ni instalación.

### Módulos

- **Editor de Cotización** con 3 formatos (América, Europa/Asia, Cruceros)
- **Calculadora tipo Excel** por hotel con fórmulas `+ - * /` y redondeo automático a múltiplos de 9
- **Historial** con filtros y acciones (Ver, Editar, Duplicar, Aceptar, Eliminar)
- **Ventas** con checklist de tareas y estado de cuenta
- **Cobranza** con registro de pagos, recibos y estados de cuenta imprimibles
- **Clientes** con base de datos completa
- **Destinos** con imágenes de encabezado por país/ciudad
- **Dashboard** con KPIs y gráficos
- **Configuración** editable en 8 pestañas

## 🧑‍💻 Equipo

- Brenda Gomez
- Xavier Castro
- Maria Perez
- Milady Zuñiga
- Susana Castro

## 🏗️ Stack

- HTML + CSS + JavaScript vanilla (sin frameworks)
- Persistencia actual: `localStorage`
- Próximo paso: migración a **Firebase** (Firestore + Storage + Auth)

## 📦 Estructura

```
cotizacion.html           ← todo el sistema
index.html                ← redirect a cotizacion.html
HANDOFF-PARA-CLAUDE.md    ← contexto para asistentes IA
README.md
.gitignore
```

## 🖨️ Uso

1. Abrir `cotizacion.html` (o la URL del sistema)
2. Llenar el formulario del lado izquierdo
3. Preview en vivo del lado derecho
4. **Exportar PDF** → imprimir → "Guardar como PDF"

## ⚙️ Contacto

**Deseos Viajeros MYX SRL**
Cédula: 3-102-862093
Tel: 8627-5400
