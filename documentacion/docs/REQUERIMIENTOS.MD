# Requerimientos de Negocio y Especificaciones Técnicas

Este documento detalla las reglas de negocio, los casos de uso y las restricciones técnicas provistas originalmente para el desarrollo del sistema de gestión, migración de datos y modelo de Inteligencia de Negocios (BI) de la cadena de supermercados.

## 1. Casos de Uso del Sistema Transaccional

El sistema debe soportar y registrar cuatro operaciones principales del supermercado, manejando datos previamente desnormalizados y con inconsistencias de origen:

### 1.1 Gestión de Ventas
Cada venta debe registrarse emitiendo un ticket que consolide la siguiente información:
* **Cabecera:** Sucursal, Nro. de Ticket, Fecha y Hora, número y tipo de Caja, legajo del Empleado y Tipo de comprobante.
* **Detalle de Productos:** Código del producto (asociado a su categoría y subcategoría), Cantidad vendida, Precio Unitario, Promociones aplicadas individualmente y el Total por producto.
* **Totales:** Subtotal del ticket (sin descuentos), Total de promociones aplicadas, Total descontado por medio de pago y el Importe Total final de la venta.

### 1.2 Gestión de Pagos
El sistema debe permitir que una misma venta se abone con múltiples pagos y medios de pago.
* **Registro Base:** Nro. de Pago, Fecha y Hora, Medio de Pago e Importe abonado.
* **Detalle Financiero (Tarjetas):** Para pagos con tarjeta se debe persistir la información del Cliente (Nombre, DNI), Nro. de Tarjeta, Fecha de Vencimiento y Cantidad de Cuotas.
* **Descuentos:** Se deben calcular y registrar los descuentos específicos que apliquen al momento del pago según el medio elegido.

### 1.3 Gestión de Promociones y Descuentos
Se debe modelar un motor de reglas comerciales dividido en dos ramas:
* **Promociones por Producto:** Configuración de promociones vigentes entre fechas, con reglas específicas que determinan: Porcentaje de descuento, Cantidad de productos para que aplique la regla (ej. el "4" en un 4x2), Cantidad a descontar (ej. el "2" en un 4x2), Tope máximo de aplicaciones, y limitantes lógicas (si aplica a misma marca o mismo producto).
* **Descuentos por Medio de Pago:** Beneficios financieros con fecha de vigencia, porcentaje de descuento fijo y un tope máximo monetario.

### 1.4 Gestión de Envíos
Registro del proceso logístico asociado a un ticket de venta específico.
* **Datos Operativos:** Nro. de Envío, Ticket asociado, Fecha programada, Rango horario estimado (inicio y fin), Costo del envío y Estado de entrega.
* **Datos del Cliente:** Domicilio y datos de contacto para efectivizar la entrega.

---

## 2. Requerimientos Técnicos y Restricciones de Migración

* **Motor de Base de Datos:** Implementación íntegra en Microsoft SQL Server 2019.
* **Restricción de Herramientas (ETL):** Queda estrictamente prohibido el uso de herramientas externas para la migración. Todo el volcado de datos desde la tabla maestra desorganizada (más de 1.8 GB) hacia el modelo relacional normalizado debe resolverse mediante código T-SQL nativo y Stored Procedures.
* **Integridad y Limpieza de Datos:** No se permite inventar información para forzar relaciones. Las inconsistencias de origen (como DNI duplicados, fechas invertidas o ventas cruzadas de sucursales) deben resolverse estructuralmente a nivel de arquitectura en el modelo de base de datos.

---

## 3. Especificaciones del Modelo Dimensional (BI)

Sobre el modelo transaccional normalizado, se requiere construir un Data Warehouse mediante un esquema de estrella/copo de nieve dentro de la misma base de datos (utilizando el prefijo `BI_`). 

El modelo debe soportar las siguientes dimensiones mínimas: Tiempo (Año, Cuatrimestre, Mes), Ubicación (Provincia/Localidad), Sucursal, Rango Etario (<25, 25-35, 35-50, >50), Turnos (08-12, 12-16, 16-20, 20-08), Medio de Pago y Categoría/Subcategoría de Productos.

A partir de este modelo dimensional, se deben construir vistas SQL para calcular los siguientes **12 KPIs gerenciales**:
1. Ticket Promedio mensual (en $) según localidad, año y mes.
2. Cantidad de unidades promedio vendidas por turno y cuatrimestre.
3. Porcentaje anual de ventas por rango etario del empleado, tipo de caja y cuatrimestre.
4. Cantidad de ventas registradas por turno, localidad y mes.
5. Porcentaje de descuento aplicado sobre el total de tickets por mes y año.
6. Top 3 categorías de productos con mayor descuento aplicado por promociones.
7. Porcentaje de cumplimiento de envíos (desvío logístico) por sucursal y mes.
8. Cantidad de envíos por rango etario de clientes por cuatrimestre.
9. Top 5 localidades de clientes con mayor costo logístico de envío.
10. Top 3 sucursales con mayor importe de pagos en cuotas, por medio de pago y mes.
11. Promedio del importe de la cuota en función del rango etario del cliente.
12. Porcentaje de descuento aplicado por cada medio de pago sobre el valor total de pagos (sin descuento).
