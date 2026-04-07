# UTN-GestionDeDatos-Migracion-BI
Proyecto de migración de datos (T-SQL) y modelado dimensional (BI) para un sistema de supermercados.
> 📄 **Documentación Oficial:** Puedes consultar el enunciado original y los requerimientos del proyecto descargando el [archivo local](docs/Especificacion_Proyecto.pdf) o visualizándolo online directamente desde [Google Drive](https://drive.google.com/file/d/1Cl2lF2-SjuyEL5ZyTZXmz940xS8bp81_/view?usp=sharing](https://drive.google.com/file/d/1Cl2lF2-SjuyEL5ZyTZXmz940xS8bp81_/view?usp=sharing).

# Migración y Arquitectura de Datos - Sistema de Supermercados 🛒📊

Este repositorio contiene la resolución del proyecto final de la asignatura **Gestión de Datos** (Universidad Tecnológica Nacional - UTN). El objetivo principal fue realizar la migración completa de un sistema desnormalizado hacia un modelo relacional transaccional, y posteriormente, construir un modelo dimensional de Inteligencia de Negocios (BI) para la toma de decisiones gerenciales.

## 🛠️ Tecnologías Utilizadas
* **Motor de Base de Datos:** SQL Server 2019
* **Lenguaje:** T-SQL (Transact-SQL)
* **Conceptos Aplicados:** Normalización, Stored Procedures, Índices, Modelado Relacional, Modelado Dimensional (Data Warehouse / BI).

## 📂 Estructura del Repositorio
* `script_creacion_inicial.sql`: Script T-SQL que genera el esquema transaccional desde cero, crea las tablas con sus restricciones (PKs, FKs) y ejecuta los *Stored Procedures* para migrar más de 1.8 GB de datos crudos.
* `script_creacion_BI.sql`: Script que construye el modelo dimensional (tablas de Hechos y Dimensiones) y genera las vistas necesarias para resolver 12 KPIs estratégicos (Ticket promedio, desvío de envíos, categorías con mayores descuentos, etc.).
* `DER.jpeg`: Diagrama de Entidad-Relación del modelo transaccional.
* `DER_BI.jpeg`: Diagrama de Entidad-Relación del modelo de Inteligencia de Negocios.
* `Estrategia.pdf`: Documentación técnica respaldando las decisiones de diseño y arquitectura.

## 🚀 Principales Logros del Proyecto
1. **Migración Exitosa:** Limpieza y estructuración de una tabla maestra desorganizada con millones de registros hacia un modelo fuertemente normalizado.
2. **Eficiencia:** Implementación de consultas SQL optimizadas y creación de índices para garantizar un rendimiento adecuado en el volumen de datos manejado.
3. **Business Intelligence:** Diseño de un modelo en estrella/copo de nieve para unificar la información de ventas, pagos, promociones y envíos, facilitando la creación de tableros de control gerenciales.

---
*Proyecto desarrollado por Luca Aiello y equipo.*
