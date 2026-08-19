# Arquitectura Técnica y Stack Tecnológico: Proyecto Beacon

Este documento define la arquitectura técnica para el desarrollo del Proyecto Beacon. El enfoque está diseñado para construir un Producto Mínimo Viable (MVP) funcional y ágil, pero con bases sólidas que permitan escalar a múltiples sucursales, integrar nuevas funcionalidades y soportar un mayor volumen de datos en el futuro.

---

## 1. Arquitectura Frontend (Interfaz de Usuario)

El frontend es el punto de interacción para los perfiles Clínico, Administrativo y Superusuario. Se prioriza la responsividad (Mobile-First) y la velocidad de carga.

### Stack y Versiones Recomendadas
*   **Framework Principal:** React (v18.x) - Utilizando Functional Components y Hooks.
*   **Enrutamiento:** React Router (v6.x) - Para manejar la navegación entre la vista clínica, bandeja administrativa y dashboard de superusuario (SPA).
*   **Diseño y UI:** Bootstrap (v5.x) mediante `react-bootstrap`. Garantiza un diseño adaptable a móviles (esencial para el picking en bodega y solicitudes clínicas) con componentes prefabricados.
*   **Gestión de Estado Global:** Zustand (v4.x) o React Context API. Zustand es ideal para un MVP escalable, ya que es más ligero y fácil de implementar que Redux para manejar el carrito de pedidos o el estado del usuario logueado.
*   **Gestión de Fechas:** `date-fns` o `Day.js` - Para el manejo ligero de las alertas de 15 minutos y las líneas de tiempo del dashboard.

### Buenas Prácticas
*   **Componentización:** Separar la lógica de la vista. Crear componentes reutilizables (ej. `<ProductCard />`, `<OrderRow />`).
*   **Lazy Loading:** Cargar los módulos bajo demanda (ej. no cargar el código del Dashboard de Superusuario si quien entra es un Clínico).
*   **Debounce en Búsqueda:** En el buscador predictivo (a partir de 3 letras), implementar un retraso ("debounce") de ~300ms antes de consultar la base de datos para no saturar las peticiones.

### Terminología Clave
*   **SPA (Single Page Application):** Aplicación web que carga una sola página HTML y actualiza el contenido dinámicamente sin recargar la pestaña.
*   **Mobile-First:** Estrategia de diseño que inicia adaptando la web para pantallas móviles y luego escala hacia pantallas de escritorio.
*   **Hook:** Funciones de React que permiten "enganchar" el estado y el ciclo de vida de los componentes (ej. `useState`, `useEffect`).

---

## 2. Arquitectura Backend y Flujo de Datos

Dado el uso de Firebase, la arquitectura backend opera bajo un modelo BaaS (Backend as a Service). Sin embargo, se requiere lógica de servidor para la ingesta de datos maestros y la automatización de limpieza.

### Stack y Versiones Recomendadas
*   **Plataforma Cloud:** Firebase (Proyecto en plan Blaze para habilitar funciones en la nube).
*   **Autenticación:** Firebase Authentication (Integrado con roles en Firestore).
*   **Automatización de BBDD:** Firebase Cloud Functions (Node.js v18+). Ideal para programar el "reseteo" diario del histórico visual a las 00:00 hrs.
*   **ETL y Carga de Datos (Scripting):** Python (v3.10+) utilizando las librerías `pandas` y `firebase-admin`. Para la lectura de los Excel maestros (Productos, Sucursales, Perfiles) y su subida masiva.

### Buenas Prácticas
*   **RBAC (Role-Based Access Control):** La seguridad no debe depender del frontend. Las reglas de seguridad de Firestore (Security Rules) deben validar que un "Clínico" no pueda escribir en la colección de "Transacciones" en el estado de picking.
*   **Operaciones Batch:** Para la carga desde Excel usando Python, utilizar operaciones por lotes (`batch writes`) para subir hasta 500 registros de productos de golpe, optimizando tiempos y costos.
*   **Separación de Entornos:** Mantener al menos dos proyectos en Firebase: `beacon-dev` (para pruebas) y `beacon-prod` (para operación real).

### Terminología Clave
*   **BaaS (Backend as a Service):** Modelo en la nube donde el proveedor gestiona los servidores, bases de datos y autenticación.
*   **ETL (Extract, Transform, Load):** Proceso de extraer datos (Excel), transformarlos (pandas) y cargarlos a un sistema destino (Firebase).
*   **Cloud Functions / Serverless:** Código que se ejecuta en los servidores de Google en respuesta a eventos (ej. limpiar la base de datos cada medianoche) sin tener que mantener un servidor encendido 24/7.

---

## 3. Arquitectura de Base de Datos (BBDD)

Se utilizará una base de datos NoSQL orientada a documentos, que ofrece alta flexibilidad y lecturas en tiempo real, vital para las alertas de bandeja de entrada.

### Stack Recomendado
*   **Motor de BBDD:** Firebase Firestore.

### Estructura Conceptual (Colecciones y Documentos)
1.  **`productos` (Colección Maestra)**
    *   `id` (String - SKU autogenerado o código interno)
    *   `nombre` (String)
    *   `familia` (String - ej. "Insumos Quirúrgicos")
    *   `factor_empaque` (Number - ej. 100)
    *   `sucursal_id` (Reference - Dónde está disponible)
2.  **`perfiles` (Colección Maestra)**
    *   `uid` (String - Vinculado a Firebase Auth)
    *   `rol` (String - "CLINICO", "BODEGA", "SUPERADMIN")
    *   `sucursal_id` (Reference)
    *   `favoritos` (Array of Strings - SKUs guardados por el clínico)
3.  **`sucursales` (Colección Maestra)**
    *   `id` (String)
    *   `nombre` (String)
4.  **`transacciones` (Flujo Operativo)**
    *   `folio` (Number/String - Identificador único)
    *   `solicitado_por` (String - Nombre del clínico)
    *   `fecha_solicitud` (Timestamp)
    *   `estado_actual` (Number - 1 al 6)
    *   `detalle_pedido` (Array de Objetos):
        *   `sku_producto` (String)
        *   `cantidad_solicitada` (Number)
        *   `cantidad_entregada` (Number)
        *   `estado_linea` (String - "TOTAL", "PARCIAL", "SIN_PROCESAR")
        *   `motivo_rechazo` (String - Opcional)
        *   `lotes` (Array of Strings - "SIN LOTE", "LOTE-A", etc.)

### Buenas Prácticas
*   **Desnormalización Estratégica:** En NoSQL, es mejor duplicar cierta información para evitar múltiples lecturas. Por ejemplo, guardar el `nombre` del producto directamente dentro del `detalle_pedido` de la transacción, para no tener que cruzar la colección de transacciones con la de productos al generar el archivo Excel de salida.
*   **Índices Compuestos:** Crear índices en Firestore para consultas complejas, como "Mostrar transacciones de la sucursal X que estén en estado 3 (Enviado) ordenadas por fecha".
*   **Archivado vs Borrado:** Para el reseteo diario que optimiza el dashboard, no borrar los documentos. Cambiarles un atributo `archivado: true` o moverlos mediante Python a una base de datos más económica (como un almacenamiento para dashboards históricos).

### Terminología Clave
*   **NoSQL:** Bases de datos que no usan tablas relacionales (SQL), sino que almacenan datos en formatos flexibles como documentos similares a JSON.
*   **Documento:** Unidad básica de almacenamiento en Firestore (equivalente a una fila en SQL). Contiene pares clave-valor.
*   **Colección:** Grupo de documentos (equivalente a una tabla en SQL).
*   **Desnormalización:** Técnica de optimización en bases de datos NoSQL donde se duplican datos intencionalmente para agilizar las consultas de lectura.