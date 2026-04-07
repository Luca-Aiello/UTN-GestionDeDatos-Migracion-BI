# Documentación Técnica: Arquitectura y Modelado de Datos

En este documento se detallan exhaustivamente las decisiones técnicas adoptadas durante el diseño del modelo relacional transaccional y el modelo dimensional (Business Intelligence). Se incluye el diccionario de datos completo y la justificación de la normalización aplicada para la migración de la tabla maestra.

---

## 1. Decisiones Generales de Modelado e Integridad

Durante la migración de datos desestructurados, se establecieron las siguientes estrategias de arquitectura:

* **Manejo de Claves Compuestas (PKs):** Se utilizaron para garantizar la unicidad frente a datos origen inconsistentes. Por ejemplo, en `Ticket` se agregó `tick_fecha_hora` a la PK (junto a número, sucursal y tipo) al detectar comprobantes duplicados. En `Item_ticket`, se sumó `item_precio_total` a la clave para diferenciar productos repetidos en un mismo ticket.
* **Claves Subrogadas autoincrementales (`IDENTITY`):** Implementadas masivamente para independizar entidades, optimizar índices y simplificar relaciones en entidades core como `Pago` (`pago_codigo`), `Detalle_Pago_Tarjeta` y `Empleado`, así como en todas las tablas paramétricas de dominios geográficos y tipologías.
* **Resolución de Referencias Cruzadas:** Se corrigieron dependencias circulares del diseño inicial. Por ejemplo, `Detalle_Pago_Tarjeta` quedó estructurada como referenciada desde `Pago`, eliminando la Foreign Key (FK) redundante inversa.

---

## 2. Diccionario de Datos: Modelo Relacional Transaccional

El sistema OLTP normalizado se compone de 25 entidades estructuradas para garantizar el soporte de las operaciones de venta, envíos y promociones:

### Ventas y Transacciones
* **Ticket:** Entidad cabecera. La PK compuesta asegura unicidad mediante `tick_numero`, `tick_sucursal`, `tick_tipo` y `tick_fecha_hora`. Contiene referencias (FKs) a Cliente, Empleado y Caja.
* **Item_ticket:** Detalle de productos por ticket. Su PK integra múltiples campos, incluyendo el precio total, para evitar el colapso de registros duplicados idénticos en el origen.
* **Pago:** Entidad centralizada con una PK única autoincremental (`pago_codigo`) basada en la fecha, importe, medio de pago, detalle (tarjeta) y cliente.
* **Detalle_Pago_Tarjeta:** Almacena los datos de pagos con tarjeta generándose una PK autoincremental (`deta_codigo`) para consolidar número, vencimiento y cuotas de la tarjeta.
* **Envio:** Registra la logística. Utiliza una PK `envi_codigo` (IDENTITY) y una FK múltiple hacia `Ticket`. Agrega el campo `envi_estado` normalizando los estados del servicio.

### Promociones y Descuentos
* **Promocion:** Detalla la vigencia (inicio y fin) y se asocia fuertemente a una `Regla` comercial.
* **Regla:** Parametriza las lógicas del negocio (PK IDENTITY): descripción, descuentos aplicables, cantidades máximas, y si aplica a misma marca o producto.
* **Descuento:** Almacena tipos de descuentos a aplicar por compras, con sus fechas de vigencia, porcentajes, topes y FK a `Medio_De_Pago`.
* **Promocion_X_Producto:** Tabla intermedia que resuelve la relación muchos a muchos (N:M) vinculando qué promociones aplican a qué productos.
* **Promocion_X_Ticket:** Entidad intermedia que registra el rastro de la promoción específica y el descuento monetario aplicado a los productos de un `Item_ticket`.
* **Descuento_X_Medio_Pago:** Vincula y registra la cantidad de descuento aplicada a un `Pago` específico en base a un `Descuento`. PK compuesta por las FKs de ambas tablas.

### Catálogo de Productos
* **Producto:** Almacena el código (IDENTITY), nombre, descripción y precio. Se asocia mediante FKs a su Categoría, Subcategoría y Marca.
* **Categoria:** Administra tanto categorías principales como subcategorías bajo una PK `cate_codigo` autoincremental.
* **Marca:** Entidad paramétrica (PK IDENTITY) para catalogar las marcas de los productos.

### Entidades Geográficas y Corporativas
* **Cliente:** Datos demográficos y de contacto. Integra una FK a `Localidad`.
* **Supermercado:** Detalles fiscales (CUIT, Razón Social) con PK incremental (`supe_codigo`).
* **Sucursal:** Mapeo de sucursales con FK directa a `Localidad` y a `Supermercado`. Se eliminó el campo provincia redundante para alcanzarlo mediante transitividad geográfica.
* **Localidad:** Centraliza localizaciones (PK IDENTITY) utilizadas por clientes, sucursales y supermercados, y delega a `Provincia` por medio de FK.
* **Provincia:** Centraliza las provincias del país con PK IDENTITY.

### Recursos Humanos y Operaciones
* **Empleado:** Tabla generada con PK autoincremental `emp_codigo` para aislar los datos demográficos (DNI, registro, email, nacimiento).
* **Caja:** Identificada por una PK compuesta (`caja_codigo` y `caja_sucursal`) para resolver el escenario de múltiples cajas con el mismo número operando en distintas sucursales.
* **Tipo_Caja:** Entidad paramétrica (PK IDENTITY) con la descripción de los tipos operativos de caja.
* **Medio_De_Pago:** Catálogo de medios existentes dados de alta (PK IDENTITY), referenciando a su `Tipo_Medio_Pago`.
* **Tipo_Medio_Pago:** Normalización de la naturaleza del medio de pago.
* **Estado_Envio:** Centraliza los posibles estados logísticos (PK IDENTITY) facilitando su modificación y lectura.

---

## 3. Diccionario de Datos: Modelo Dimensional (Business Intelligence)

Para dar soporte a la creación de tableros y resolución de KPIs (Ticket Promedio, Desvíos logísticos, Efectividad de promociones, etc.), se diseñó un modelo en esquema Estrella/Copo de Nieve compuesto por 13 entidades.

### Dimensiones (Dimensions)
* **BI_Tiempo:** Define la granularidad analítica principal agrupando por Año, Cuatrimestre y Mes.
* **BI_Turnos:** Clasificación de rangos horarios. Para absorber el turno de la madrugada no previsto inicialmente, se modeló con lógica de rango invertido (Hora Inicial = 20, Hora Final = 8).
* **BI_Rango_Etario:** Segmentación por edad. Se definió un tope superior absoluto de 200 años y se implementó un rango dinámico `NULL` (`edad_inicial IS NULL` y `edad_final IS NULL`) para contabilizar pagos ejecutados sin un cliente registrado.
* **BI_Ubicacion:** Dimensión geográfica cruzando el nombre de la localidad con una FK a `BI_Provincia`.
* **BI_Provincia:** Dimensión complementaria para agrupaciones provinciales.
* **BI_Medio_De_Pago:** Dimensión que almacena nombre y código del medio utilizado.
* **BI_Categoria:** Dimensión estructurada para cruzar analíticas tanto por categoría como subcategoría.
* **BI_Tipo_Caja:** Dimensión para evaluar performance o saturación según la tipología de caja.
* **BI_Sucursal:** Dimensión con código y nombre comercial de cada instalación.

### Tablas de Hechos (Fact Tables)
* **BI_Fact_Table_Ventas:** * *Dimensiones conectadas:* Tiempo, Sucursal, Turnos, Categorías (indirecta), Rango Etario, Ubicación, Tipo de Caja.
  * *Measures (Métricas):* Importe total, Cantidad de artículos, Descuento total aplicado (unificando promociones y descuentos por medio de pago).
* **BI_Fact_Table_Pagos:** * *Dimensiones conectadas:* Tiempo, Medio de Pago, Sucursal, Rango Etario.
  * *Measures (Métricas):* Importe pago total (sin descuento), Valor descontado acumulado, y Cantidad de cuotas (para análisis de pagos diferidos).
* **BI_Fact_Table_Envios:** * *Dimensiones conectadas:* Tiempo, Sucursal, Rango Etario, Ubicación.
  * *Measures (Métricas):* Cantidad de envíos programados, Envíos cumplidos (a tiempo) y Costo de envío logístico.
* **BI_Fact_Table_Promociones:** * *Dimensiones conectadas:* Tiempo, Categoría.
  * *Measures (Métricas):* Descuento en producto por promoción aplicada (permite analizar financieramente qué categorías absorbieron mayor volumen de descuentos).
