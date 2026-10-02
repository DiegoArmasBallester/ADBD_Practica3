# Documentación del Modelo Entidad-Relación: Tajinaste S.A.

Este documento describe el modelo conceptual de datos para el sistema de gestión de Tajinaste S.A., cubriendo viveros, stock de productos, histórico de empleados, pedidos y clientes (programa de fidelización).

---

## 1. Descripción de Entidades

*   **Vivero (Fuerte):** Representa cada una de las instalaciones físicas de venta de la empresa.
*   **Zona (Débil):** Representa las diferentes áreas organizativas dentro de un vivero (ej. Exterior, Almacén). Su existencia depende totalmente del Vivero al que pertenece.
*   **Producto (Fuerte):** Representa los artículos físicos (plantas, jardinería, decoración) comercializados por la empresa.
*   **Empleado (Fuerte):** Almacena la información de los trabajadores encargados de las tareas en los viveros y gestión de pedidos.
*   **Cliente (Fuerte):** Representa a los compradores de la empresa, de los cuales se registra información especialmente para gestionar el programa "Tajinaste Plus".
*   **Pedido (Fuerte):** Representa cada transacción de compra realizada por un cliente.

---

## 2. Atributos y Dominios

### Atributos de Entidades

*   **Vivero**
    *   `id_vivero` (Clave Primaria): Entero único. *Ej: 101*
    *   `nombre`: Cadena de texto. *Ej: "Tajinaste Norte"*
    *   `georef`: Atributo compuesto (Latitud, Longitud) en formato decimal. *Ej: (28.4636, -16.2518)*

*   **Zona**
    *   `id_zona` (Clave Parcial): Alfanumérico, único dentro de cada vivero. *Ej: "Z1-EXT"*
    *   `nombre`: Cadena de texto. *Ej: "Zona Exterior"*
    *   `georef`: Atributo compuesto (Latitud, Longitud) en formato decimal.

*   **Producto**
    *   `id_producto` (Clave Primaria): Entero único. *Ej: 50201*
    *   `nombre`: Cadena de texto. *Ej: "Saco de abono 50L"*
    *   `categoria`: Cadena de texto o enumerado. *Ej: "Jardinería", "Decoración"*

*   **Empleado**
    *   `id_empleado` (Clave Primaria): Entero único. *Ej: 884*
    *   `nombre`: Cadena de texto. *Ej: "María López"*

*   **Cliente**
    *   `id_cliente` (Clave Primaria): Entero único. *Ej: 9910*
    *   `nombre`: Cadena de texto. *Ej: "Carlos Pérez"*
    *   `fecha_ingreso_plus`: Fecha (Date). Admite valor NULO. Si tiene valor, indica que pertenece al programa Plus. *Ej: 2023-05-12*

*   **Pedido**
    *   `id_pedido` (Clave Primaria): Entero único autoincremental. *Ej: 44021*
    *   `fecha_pedido`: Fecha y hora (DateTime). *Ej: 2023-11-20 15:30:00*

### Atributos de Relaciones

*   **Relación "Puesto" (Empleado - Zona)**
    *   `fecha_inicio`: Fecha (Date). Indica el primer día en ese puesto. *Ej: 2023-01-01*
    *   `fecha_final`: Fecha (Date). Admite NULO si es el puesto actual. *Ej: 2023-06-30*
    *   `productividad`: Decimal (Float o Integer). Métrica de desempeño. *Ej: 85 (porcentaje) o 8.5 (puntuación)*

*   **Relación "Se asigna" (Producto - Zona)**
    *   `stock`: Entero positivo. Representa las unidades disponibles de ese producto en esa zona concreta. *Ej: 150*

*   **Relación "Contiene" (Pedido - Producto)**
    *   `cantidad`: Entero positivo. Unidades compradas de un producto en un pedido. *Ej: 3*

---

## 3. Descripción de Relaciones y Cardinalidades

*   **Vivero (1) - Tiene - (N) Zona**
    *   *Descripción:* Relación identificadora (dependencia existencial).
    *   *Cardinalidad:* `1:N`. Un vivero tiene obligatoriamente una o más zonas. Una zona pertenece a un único vivero. Si se elimina un vivero, sus zonas desaparecen.

*   **Zona (N) - Se asigna - (M) Producto**
    *   *Descripción:* Asignación física del inventario a las ubicaciones.
    *   *Cardinalidad:* `N:M`. Una zona contiene muchos productos distintos. Un tipo de producto puede estar almacenado en múltiples zonas (ej. macetas en el almacén y en la exposición exterior).

*   **Empleado (N) - Puesto - (M) Zona**
    *   *Descripción:* Historial laboral de la ubicación del empleado en la empresa a lo largo del tiempo.
    *   *Cardinalidad:* `N:M`. Un empleado puede haber trabajado en varias zonas a lo largo del tiempo. Una zona es atendida por múltiples empleados históricamente.

*   **Empleado (1) - Gestiona - (N) Pedido**
    *   *Descripción:* Control de responsabilidad sobre las ventas para calcular objetivos de productividad.
    *   *Cardinalidad:* `1:N`. Un empleado gestiona múltiples pedidos a lo largo del mes. Cada pedido es gestionado por un único empleado responsable.

*   **Cliente (1) - Hace - (N) Pedido**
    *   *Descripción:* Registro de las compras realizadas por cada cliente para calcular su volumen mensual y bonificaciones.
    *   *Cardinalidad:* `1:N`. Un cliente realiza múltiples pedidos. Un pedido pertenece en exclusiva a un único cliente.

*   **Pedido (N) - Contiene - (M) Producto**
    *   *Descripción:* Detalle o líneas del recibo de compra.
    *   *Cardinalidad:* `N:M`. Un pedido puede contener una variedad de múltiples productos. Un producto específico forma parte de múltiples pedidos distintos.

---

## 4. Restricciones Semánticas Propuestas

Para asegurar la coherencia de los datos en relación al modelo de negocio de Tajinaste S.A., se proponen las siguientes restricciones:

1.  **Exclusividad Temporal de Destino:** Para un mismo `id_empleado`, los rangos de tiempo definidos por `fecha_inicio` y `fecha_final` en la relación *Puesto* no pueden solaparse. Esto garantiza la regla de negocio que estipula que un empleado "nunca va a tener dos destinos" de forma simultánea.
2.  **Validación de Fechas:** En la relación *Puesto*, la `fecha_final` debe ser mayor o igual a la `fecha_inicio` (o NULA si la asignación sigue activa).
3.  **Dominios de Cantidades:** El atributo `stock` (relación *Se asigna*) debe ser $\ge 0$. El atributo `cantidad` (relación *Contiene*) debe ser $> 0$.
4.  **Pedidos de Fidelización (Tajinaste Plus):** Para contabilizar los pedidos en las métricas de un cliente Plus, la `fecha_pedido` debe ser igual o estrictamente mayor a su `fecha_ingreso_plus`.
5.  **Borrado en Cascada de Zonas:** Por su naturaleza de entidad débil, la eliminación de un registro de `Vivero` deberá forzar la eliminación en cascada de todos los registros de `Zona` asociados al mismo.
