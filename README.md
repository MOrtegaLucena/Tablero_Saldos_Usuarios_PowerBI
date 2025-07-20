# Tablero de Saldos de Usuarios Activos e Inactivos en Power BI

## 🎯 Objetivo del Tablero

He desarrollado un **tablero dinámico en Power BI** que permite visualizar, en tiempo real, los **saldos de usuarios activos e inactivos** de una aplicación perteneciente a una empresa del sector Fintech.

Este tablero tiene como propósito principal identificar comportamientos de inactividad en los usuarios, definidos como aquellos que no han realizado transacciones en más de 6 meses, ya sea por bloqueos judiciales o por simple abandono de los servicios. Esto permite a la empresa generar estrategias de retención proactiva para recuperar clientes y prevenir la pérdida de usuarios valiosos.

## 📊 Información Visualizada

- Identificación y Saldo de usuarios activos e inactivos, y sus correspondientes filtros de fecha. 
- Fecha de la última transacción realizada por cada usuario. 
- Cantidad de usuarios comerciales y personales.
- Cantidad de Personas Físicas y Personas Jurídicas.
- Tarjetas que muestran el: 
    - Saldo total de Usuarios
    - Saldo total de usuarios activos
    - Saldos total de usuarios inactivos.

Toda la información precedente puede ser filtrada por:
- Rango de fechas
- Jurisdicción
- Tipo de cuenta (comercial o personal)
- Tipo de usuario (PF o PJ)

## ⚙️ Procedimiento de Desarrollo

### 🔗 Conexión a Fuentes de Datos

- Conexión a **base de datos web** que contiene información personal de los usuarios.
    - **Extraer y leer datos de archivo en formato JSON.**

<img width="1698" height="685" alt="image" src="https://github.com/user-attachments/assets/26d6efa7-564f-4106-99bf-0805199e914b" />

- Conexión a una **base de datos PostgreSQL** para acceder a las siguientes tablas:
- Tabla de movimientos/transacciones
- Tabla de cargas impositivas y comisiones

Se utilizó Direct Query para optimizar el rendimiento, disminuir el almacenamiento requerido y asegurar una actualización en tiempo real de los datos.

### 🧼 Transformación y Limpieza de Datos

- Eliminación de columnas irrelevantes
- Corrección de tipos de datos y formatos (fechas, monedas, etc.)
- Tratamiento de valores nulos, duplicados y errores
- Creación de columnas condicionales para clasificar los movimientos como “completados” o “fallidos”
<img width="975" height="160" alt="image" src="https://github.com/user-attachments/assets/ae12387f-5c54-4926-9492-bcfcbd0162b4" />

- Adición de columnas personalizadas para:
    - Mostrar montos con decimales y signo (+/-), dependiendo si se trata de ingresos o egresos.
    - Calcular saldo de usuarios y monto neto por transacción.

<img width="975" height="617" alt="image" src="https://github.com/user-attachments/assets/4c438887-627a-404f-b0f6-6dc0055ae194" />

<img width="975" height="621" alt="image" src="https://github.com/user-attachments/assets/f4657e5f-bbdc-4fd0-b279-b9757f185608" />

<img width="975" height="627" alt="image" src="https://github.com/user-attachments/assets/c2b69975-7350-41e2-ac79-bc07eb7cfe73" />


### 🧩 Modelado de Datos

- Relaciones entre tablas “Datos Personales de Usuarios” y “Movimientos/Transacciones de usuarios” mediante IDs comunes
- Adicionalmente, se establece una relación entre la tabla “Movimientos/Transacciones de Usuarios” y la tabla de “Cargas Impositivas/Comisiones”.
<img width="975" height="727" alt="image" src="https://github.com/user-attachments/assets/ed32c625-aed0-45df-810f-c700587891f4" />

  
- Creación de tabla DAX para obtener el último movimiento y saldo actualizado de cada usuario, y relacionarla con las tablas precedentes.

**Fórmula DAX utilizada:**

    UltimoMovimientoCompleto =
    ADDCOLUMNS (
        SUMMARIZE (
            'public powerbi_movement',
            'public powerbi_movement'[wallet_id]
        ),
        "FechaUltimoMovimiento",
            CALCULATE (
                MAX('public powerbi_movement'[date])
            ),
        "BalanceUltimoMovimiento",
            VAR FechaMax = CALCULATE (
                MAX('public powerbi_movement'[date])
            )
            VAR Movimiento =
                TOPN (
                    1,
                    FILTER (
                        'public powerbi_movement',
                        'public powerbi_movement'[wallet_id] = EARLIER('public powerbi_movement'[wallet_id]) &&
                        'public powerbi_movement'[date] = FechaMax
                    ),
                    'public powerbi_movement'[fingerprint], DESC
                )
            RETURN
                SELECTCOLUMNS(Movimiento, "Balance", 'public powerbi_movement'[Balance ok])
    )

<img width="975" height="492" alt="image" src="https://github.com/user-attachments/assets/415b7da0-1000-401d-b1a1-6f5ee51a2c43" />


### 🎨 Diseño de Visualizaciones

- Tablas separadas para usuarios activos e inactivos
- Tarjetas con saldos totales segmentados: 
    - Saldo total de usuarios
    - Saldo total de usuarios activos 
    - Saldo total de usuarios inactivos
- Slicers para filtros por fechas
- Segmentación de datos para filtrar por jurisdicciones
- Gráficos de anillos para segmentar tipos de cuenta, y tipos de usuario
- Para mejorar la usabilidad del tablero y permitir un análisis más preciso y segmentado, **se configuraron interacciones personalizadas entre los objetos visuales** (tablas, tarjetas, filtros y gráficos), de manera que cada elemento reaccione de forma coherente según el contexto seleccionado.

    **Configuraciones clave:**
    - Se establecieron filtros independientes por tipo de usuario (activo vs. inactivo). Por ejemplo:

     - El filtro de fecha de usuarios activos afecta únicamente a la tabla y tarjeta de usuarios activos.

     - El filtro de fecha de usuarios inactivos se aplica sólo a sus respectivas visualizaciones.

     - Los gráficos de anillos, utilizados para segmentar por tipo de cuenta (Comercial/Personal) y tipo de usuario (Persona Física o Jurídica), están conectados a ambas tablas (activos e inactivos). Esto permite que, al seleccionar una categoría dentro de estos gráficos, se actualicen simultáneamente ambas tablas y tarjetas, facilitando una comparación directa entre segmentos.

> Esto genera una experiencia de análisis dinámica y flexible, que se adapta a distintas necesidades del equipo de negocio.


Ejemplo de Tablero completo filtrando por fecha para Usuarios activos e inactivos:
<img width="975" height="542" alt="image" src="https://github.com/user-attachments/assets/53a31d38-c9bb-4ceb-a4f7-f84e15bc17d0" />


Ejemplo de Tablero filtrando por fecha y teniendo en cuenta sólo las Personas Jurídicas, al realizar un click sobre el gráfico de anillos ”Personas Físicas vs. Personas Jurídicas”:
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/b6dccf33-5315-45e3-879a-e8b2b1e3de90" />


Tablero finalizado en Power BI Desktop: 
<img width="975" height="471" alt="image" src="https://github.com/user-attachments/assets/99ba2936-2d43-4fd4-aa4b-601045099583" />


### 🚀 Publicación y Distribución

- Publicación en Power BI Service
- Configuración de actualizaciones automáticas
- Asignación de permisos de acceso a los stakeholders

### ✅ Ventajas para el Negocio

- Monitoreo en tiempo real del comportamiento financiero de usuarios
- Identificación oportuna de riesgo de abandono
- Segmentación para acciones comerciales dirigidas
- Toma de decisiones estratégicas basada en datos
- Mejora de la retención y recuperación de usuarios
