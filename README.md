### 1. OBJETIVO DEL SISTEMA

El sistema **Telefonica.Pdt** es una plataforma de gestión transaccional diseñada para la **administración del ciclo de vida de pedidos y cotizaciones** de servicios de telecomunicaciones.

*   **Problema que resuelve:** Centraliza la captura de cotizaciones, la conversión de estas en pedidos y el seguimiento técnico (agendamiento) necesario para la provisión del servicio, integrando datos de múltiples sistemas "legacy".
*   **Procesos automatizados:**
    *   Generación y consulta de cotizaciones por filtros (RUC, cliente, estado).
    *   Gestión de pedidos manuales y complejos.
    *   **Agendamiento** de instalaciones con proveedores y técnicos.
    *   Seguimiento histórico (bitácora) de estados del pedido.
*   **Áreas involucradas:** Ventas, Operaciones/Despacho (Agendamiento), Seguridad y Atención al Cliente.

---

### 2. IDENTIFICACIÓN DE TECNOLOGÍAS

El proyecto se identifica como una aplicación web multicapa basada en el ecosistema Microsoft de mediados de la década del 2000, con actualizaciones menores posteriores.

| Categoría | Tecnologías Detectadas |
| :--- | :--- |
| **Lenguajes Principales** | **C#** (Capas de Negocio, Datos y Entidades) y **VB.NET** (Capa Web/UI). |
| **Framework** | **.NET Framework 2.0** (mencionado en archivos de proyecto). |
| **Base de Datos** | **SQL Server** (uso de `System.Data.SqlClient` y Stored Procedures). |
| **Acceso a Datos** | Patrón **Abstract Factory** personalizado (`DACAbstracFactory`) y `SqlClient`. |
| **UI Framework** | **ASP.NET WebForms** con AJAX (AjaxControlToolkit). |
| **Librerías Externas** | **eWorld.UI** (controles de calendario y UI), `customCalendar.dll`. |
| **Integración Legacy** | Módulos para **ISIS, GESTEL y SIG** (sistemas tradicionales de Telefonica). |
| **Exportación** | Generación de reportes en **Excel**. |
| **Seguridad** | Criptografía simétrica (DES, RC2, Rijndael, TripleDES) para manejo de sesiones. |

---

### 3. ANÁLISIS DE ARQUITECTURA

El sistema presenta una arquitectura **N-Layer (Multicapa)** bien definida, lo que indica un esfuerzo de ingeniería para desacoplar la interfaz de la persistencia:

1.  **UI (AppWeb):** Desarrollada en VB.NET. Utiliza `MasterPages`, `UserControls` (.ascx) y herencia de una clase base `WebPaginaBase` para estandarizar comportamientos.
2.  **Negocio (Business/BL):** Clases en C# que actúan como orquestadores entre la UI y la capa de datos (ej. `pdt_pedidoBL`).
3.  **Datos (Data/DL):** Implementa la lógica de acceso a procedimientos almacenados (ej. `pdt_pedidoDL`).
4.  **Acceso a Datos (DAC):** Un componente genérico para la gestión de conexiones y comandos SQL.
5.  **Entidades (Entities):** Clases que representan el modelo de datos (POCOs) para el transporte de información entre capas.

**Tipo de Sistema:** Es un sistema **Híbrido Cliente-Servidor Web**, diseñado para un entorno corporativo **multiusuario** con control de perfiles centralizado.

---

### 4. MÓDULOS FUNCIONALES

| Módulo | Función Principal |
| :--- | :--- |
| **Cotización** | Registro de prospectos, selección de productos y servicios, y conversión a pedido formal. |
| **Pedidos** | Gestión de órdenes de servicio, incluyendo "pedidos complejos" y manuales. |
| **Agendamiento** | Administración de citas para la instalación de servicios técnicos, coordinando con proveedores. |
| **Seguimiento** | Monitoreo de estados y auditoría de cambios mediante una bitácora histórica. |
| **Seguridad** | Gestión de usuarios, perfiles, opciones de menú y permisos por acción. |
| **Maestros** | Mantenimiento de tablas base como Ubigeo (Departamentos/Provincias), Turnos y Paquetes. |

---

### 5. MODELO DE DATOS E INTEGRACIONES

*   **Tablas Maestras Identificadas:** `SG_USUARIO`, `SG_PERFIL`, `PAQUETE`, `PRD_SERVICIO`, `DEPARTAMENTO`, `UBIGEO`.
*   **Tablas Transaccionales:** `COTIZACION`, `PEDIDO`, `BITACORA`, `CUPO` (para agendamiento).
*   **Integraciones:**
    *   **Sistemas Externos:** Conexión con legacies vía tablas de intercambio o servicios para **GESTEL, ISIS y SPEEDYSIG**.
    *   **Documentación:** Capacidad de cargar y abrir documentos asociados a pedidos (PDF, XLS, DOC).

---

### 6. SEGURIDAD

*   **Autenticación:** Basada en login formal (`frmlogin.aspx`) con persistencia en variables de sesión (`HttpSessionUserCodigoUsuario`).
*   **Autorización:** Implementa un modelo granular de permisos: **Perfil -> Opción (Menú) -> Acción (Botón/Función)**.
*   **Validación:** Uso de `ValidationSummary` de ASP.NET y lógica personalizada en la clase `HelperWeb`.

---

### 7. HALLAZGOS Y RIESGOS (DEUDA TÉCNICA)

*   **Tecnología Obsoleta:** El uso de .NET 2.0 y Visual Studio 2005 representa un riesgo crítico de seguridad y soporte.
*   **Mezcla de Lenguajes:** La coexistencia de C# y VB.NET dificulta el mantenimiento y la unificación de estándares de codificación.
*   **Acoplamiento a DLLs Antiguas:** Dependencia de `eWorld.UI` y `AjaxControlToolkit` en versiones que ya no reciben actualizaciones.
*   **Hardcode Potencial:** Se observa lógica de conexión gestionada por una clase `Configuracion` que lee del `Web.config`, lo cual es correcto, pero el manejo de excepciones muestra mensajes técnicos que podrían exponer información del servidor.

---

### 8. RECOMENDACIONES DE MODERNIZACIÓN

Dada la solidez de la arquitectura multicapa actual, la migración no requiere un rediseño total de la lógica, sino una **actualización de infraestructura**:

1.  **Actualización de Framework:** Migrar de .NET 2.0 a **.NET 8 (Core)** para ganar rendimiento y soporte multiplataforma.
2.  **Unificación de Lenguaje:** Convertir el código de VB.NET a **C#** para estandarizar toda la solución.
3.  **Refactorización de UI:** Reemplazar ASP.NET WebForms por **Angular o React** consumiendo una nueva capa de **APIs REST** (WebAPI).
4.  **Estrategia de Datos:** Mantener SQL Server pero implementar un ORM moderno como **Entity Framework Core** para reemplazar el DAC manual.
5.  **Prioridad:** Alta para el módulo de Seguridad y Agendamiento debido a su criticidad operativa.

**Nivel de confianza del análisis:** **Alto**. La estructura de carpetas y la nomenclatura de los archivos de negocio y datos son consistentes con un sistema ERP/Gestión de mediano tamaño.
< 2023-05-11T22:14:00 --> 
< 2019-02-19T03:45:00 --> 
< 2019-04-09T21:34:00 --> 
< 2016-03-18T15:04:00 --> 
< 2020-03-21T04:20:00 --> 
< 2015-12-09T13:56:00 --> 
< 2023-02-20T00:41:00 --> 
< 2020-03-15T18:21:00 --> 
< 2019-01-20T17:30:00 --> 
< 2016-09-04T18:45:00 --> 
< 2020-02-17T03:11:00 --> 
< 2022-11-24T15:07:00 --> 
