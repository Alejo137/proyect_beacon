# Historia de Usuario y Uso Diario: Flujo Operativo Beacon

Este documento relata el flujo de trabajo diario utilizando la plataforma Beacon, conectando a los usuarios (A), la plataforma digital (B) y los espacios físicos (C) de la clínica.

## El Modelo Conceptual (A - B - C)
*   **[A] Personas / Usuarios:** Clínico y Administrativo.
*   **[B] Plataforma:** Sistema web Beacon (Frontend en React + BBDD en Firebase).
*   **[C] Espacios Físicos:** Bodega (almacenamiento) y Sala (área de atención clínica).

---

## 1. El Origen de la Necesidad (A -> B)
El ciclo comienza en la **SALA [C]**, donde el **CLÍNICO [A]** evalúa su arsenal y detecta qué insumos necesita para la jornada.
*   El Clínico toma su dispositivo móvil e ingresa a la **PLATAFORMA [B]**.
*   A través de una interfaz amigable (Mobile-First), busca los productos tecleando 3 letras, selecciona desde su lista de "Favoritos" e indica las cantidades (en unidades o cajas).
*   Al confirmar, el sistema genera la solicitud.
*   **Status del Pedido:** `ABIERTO`.

## 2. La Notificación y Recepción (B -> A)
La información viaja instantáneamente por la **PLATAFORMA [B]** hacia la bandeja del **ADMINISTRATIVO [A]**.
*   El Administrativo recibe una alerta visual persistente y un breve aviso sonoro ("ping") indicando: *"Tienes nuevos pedidos pendientes"*.
*   En la Plataforma, el Administrativo revisa el Dashboard, visualizando el contador de **Pedidos Abiertos**.
*   Al hacer clic y tomar la responsabilidad del requerimiento, el Clínico recibe una notificación en su panel de que su solicitud está siendo atendida.
*   **Status del Pedido:** `EN PROCESO`.

## 3. La Acción Física y Trazabilidad (A -> C)
El **ADMINISTRATIVO [A]** se desplaza físicamente a la **BODEGA [C]** con su dispositivo o tablet.
*   Mientras camina por los pasillos, utiliza la vista de preparación (Picking) de la Plataforma.
*   **Resolución por Línea:** A medida que saca los productos de los estantes, valida el proceso en el sistema ingresando el **Lote** (texto libre o "SIN LOTE"). 
*   Si hay quiebres de stock o necesita sacar el mismo producto de dos cajas distintas, utiliza la función de *Entregas Parciales* o *División de Línea*.
*   Si un ítem no está disponible, lo marca como *Sin Procesar* y selecciona un motivo de rechazo.

## 4. La Entrega Final (C -> C)
Con la caja lista, el **ADMINISTRATIVO [A]** traslada los insumos desde la **BODEGA [C]** hacia la **SALA [C]**.
*   Los productos físicos son entregados al Clínico.
*   El Administrativo cierra el ciclo en la Plataforma, generando automáticamente un número de **Folio** único asociado al nombre de quien solicitó.
*   **Status del Pedido:** `CERRADO` (Total o Parcial) o `ANULADO` (si todo fue rechazado).

## 5. Auditoría y Resumen (Capa Transversal B)
Todo el flujo anterior alimenta en tiempo real el Dashboard general dentro de la **PLATAFORMA [B]**, el cual cuenta con dos funciones principales:

1.  **Monitor en Vivo (Operación Diaria):**
    *   Muestra los contadores actualizados al minuto: *Pedidos Abiertos, Pedidos en Proceso, Pedidos Cerrados (Hoy)*.
    *   Para optimizar el rendimiento y mantener la pantalla limpia, el histórico visual del frontend **se limpia automáticamente al día siguiente a las 00:00 hrs**.
2.  **Reportabilidad y Descarga (Consolidación):**
    *   El Superusuario o Administrativo puede descargar los historiales en archivos Excel consolidados (Requerimientos vs. Picking real).
    *   Este archivo contiene la trazabilidad completa (Folio, Solicitante, SKU, Cantidades, Lotes y Motivos de rechazo) lista para ser inyectada al ERP central de la institución.