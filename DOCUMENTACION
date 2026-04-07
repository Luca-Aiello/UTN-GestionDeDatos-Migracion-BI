# Estrategia y Diccionario de Datos

En este documento se detallan las decisiones técnicas tomadas durante el diseño del modelo relacional transaccional y el modelo dimensional (Business Intelligence), incluyendo el diccionario de entidades.

## 1. Decisiones de Arquitectura y Resolución de Inconsistencias

Durante el proceso de migración y normalización de la tabla maestra, implementamos las siguientes estrategias para garantizar la integridad referencial:

* [cite_start]**Manejo de Claves Compuestas (PKs):** En la tabla `Ticket`, la PK se conformó por `tick_numero`, `tick_sucursal`, `tick_tipo` y `tick_fecha_hora` al detectar en los datos de origen tickets duplicados para una misma sucursal [cite: 741-744]. En `Item_ticket`, agregamos `item_precio_total` a la PK para evitar conflictos con productos repetidos que compartían las demás claves[cite: 736].
* [cite_start]**Claves Subrogadas (IDENTITY):** Se generaron PKs autoincrementales para independizar entidades y simplificar relaciones, como en `Pago` (`pago_codigo`) [cite: 758-759], `Detalle_Pago_Tarjeta` [cite: 784-785], `Empleado` [cite: 796-797], y tablas paramétricas como `Localidad`, `Provincia`, `Estado_Envio` y `Tipo_Caja` [cite: 776-779, 790, 793-794, 798-802].
* **Corrección de Referencias:** Se eliminaron FKs cruzadas redundantes. [cite_start]Por ejemplo, `Detalle_Pago_Tarjeta` quedó referenciada desde `Pago`, eliminando la dependencia inversa [cite: 784-785].
* [cite_start]**Tablas Intermedias:** Se crearon `Promocion_X_Producto` y `Promocion_X_Ticket` para normalizar relaciones de muchos a muchos [cite: 732-734, 748-749].

---

## 2. Diccionario de Datos: Modelo Transaccional

* **Promocion_X_Ticket:** Tabla intermedia que registra las promociones aplicadas a los productos dentro de un ítem de ticket. [cite_start]Su PK está compuesta por el ticket y la sucursal [cite: 732-734].
* [cite_start]**Item_ticket:** Detalle de productos por ticket[cite: 736].
* **Envio:** Registra los envíos. Utiliza un `envi_codigo` autoincremental como PK y una FK múltiple hacia Ticket. Migra todos los datos de envío agregando el estado referenciado (`Estado_Envio`) [cite: 738-739].
* **Ticket:** Encabezado de ventas. [cite_start]Incluye FKs hacia Cliente, Empleado y Caja [cite: 741-744].
* [cite_start]**Descuento_X_Medio_Pago:** Almacena el descuento específico aplicado a un pago según su medio[cite: 746].
* [cite_start]**Promocion_X_Producto:** Tabla que resuelve la relación N:M entre promociones y productos [cite: 748-749].
* **Caja:** Identifica las cajas por sucursal. [cite_start]Su PK es compuesta (`caja_codigo` y `caja_sucursal`) [cite: 751-753].
* [cite_start]**Sucursal:** Detalles de sucursales, vinculadas a Localidad y Supermercado [cite: 755-757].
* [cite_start]**Pago:** Registro de pagos con una PK autoincremental para asociar importe, fecha, medio y cliente de forma única [cite: 758-759].
* [cite_start]**Descuento:** Catálogo de descuentos vigentes con topes y porcentajes [cite: 761-763].
* [cite_start]**Cliente:** Información personal, de contacto y FK a Localidad [cite: 764-765].
* [cite_start]**Supermercado:** Detalles corporativos (CUIT, Razón social, etc.) con PK autoincremental [cite: 766-768].
* [cite_start]**Promocion:** Fechas de vigencia de promociones y asociación a reglas [cite: 769-770].
* [cite_start]**Medio_De_Pago:** Catálogo de medios habilitados (PK identity) y su tipo [cite: 772-774].
* [cite_start]**Localidad & Provincia:** Tablas normalizadas para unificar los datos de domicilios de clientes, sucursales y supermercados [cite: 776-779, 798-802].
* [cite_start]**Producto:** Almacena atributos del producto y se relaciona con Categoria y Marca [cite: 780-782].
* [cite_start]**Detalle_Pago_Tarjeta:** Datos de la transacción física (tarjeta, vencimiento, cuotas) [cite: 783-785].
* [cite_start]**Tipo_Medio_Pago / Tipo_Caja / Estado_Envio:** Tablas paramétricas para centralizar valores [cite: 786-794].
* **Empleado:** Datos de RRHH. [cite_start]Se agregó una PK autoincremental para su creación [cite: 795-797].
* [cite_start]**Categoria / Marca:** Dimensiones del producto [cite: 803-806].
* [cite_start]**Regla:** Condiciones matemáticas y lógicas de las promociones (descuento aplicable, cantidades máximas, exclusividad de marca) [cite: 807-808].

---

## 3. Diccionario de Datos y Reglas: Modelo BI (Business Intelligence)

Para soportar los KPIs, se crearon vistas basadas en el siguiente esquema de estrella:

**Dimensiones (Dimensions):**
* **BI_Turnos:** Se definieron rangos horarios. [cite_start]Para el turno nocturno, se manejó la lógica de inicio a las 20:00 y fin a las 08:00 [cite: 933-934].
* **BI_Rango_Etario:** Se agruparon las edades (ej. <25, 25-35, etc.). [cite_start]Se fijó un tope de 200 años para el último rango y se añadió un rango `NULL` para gestionar pagos sin cliente asociado [cite: 939-941].
* **BI_Tiempo:** Granularidad a nivel Año, Cuatrimestre y Mes [cite: 937-938].
* **BI_Ubicacion & BI_Provincia:** Jerarquía geográfica para análisis regional [cite: 942-945].
* [cite_start]**Dimensiones estándar:** `BI_Medio_De_Pago` [cite: 927-928][cite_start], `BI_Categoria` [cite: 929-930][cite_start], `BI_Tipo_Caja` [cite: 931-932] [cite_start]y `BI_Sucursal` [cite: 935-936].

**Tablas de Hechos (Fact Tables):**
* **BI_Fact_Table_Ventas:** Relaciona tiempo, sucursal, turnos, rango etario, ubicación y tipo de caja. [cite_start]Registra el importe total, cantidad de artículos y la suma unificada de descuentos (promociones + medios de pago) [cite: 946-948].
* **BI_Fact_Table_Pagos:** Relaciona tiempo, medio de pago, sucursal y rango etario. [cite_start]Cuantifica el importe base (sin descuento), el valor descontado y la cantidad de cuotas [cite: 949-952].
* **BI_Fact_Table_Envios:** Relaciona tiempo, sucursal, rango etario y ubicación. [cite_start]Analiza la cantidad total de envíos, métricas de cumplimiento y costos logísticos [cite: 953-954].
* **BI_Fact_Table_Promociones:** Relaciona tiempo y categoría. [cite_start]Mide el impacto monetario de las promociones por segmento de producto [cite: 955-957].
