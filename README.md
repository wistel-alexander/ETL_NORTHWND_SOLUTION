# ETL_NORTHWND

Descripción
-----------
Proyecto ETL basado en SSIS para cargar datos de la base de ejemplo Northwind desde la fuente hacia una capa de staging y un data warehouse (dimensiones + tabla de hechos). Incluye paquetes para extracción, transformación y carga así como un paquete maestro que orquesta el flujo.

Contenido del repositorio
-------------------------
- `Extract_To_Staging.dtsx` — Extrae datos desde la(s) fuente(s) y carga la zona de staging.
- `Transform_and_Load_Dimensions.dtsx` — Transforma y carga las tablas de dimensiones.
- `Transform_and_Load_FactTable_OrderDetails.dtsx` — Transforma y carga la tabla de hechos `OrderDetails`.
- `Master_Pipeline.dtsx` — Orquestador que ejecuta los paquetes en el orden correcto.
- Solución/archivos de proyecto de Visual Studio: abrir con SQL Server Data Tools / SSDT.

Requisitos
----------
- Visual Studio con soporte SSIS (SSDT).
- SQL Server o motor compatible para base de datos de staging y data warehouse.
- Credenciales y permisos para lectura/escritura en las bases de datos objetivo.
- Opcional: acceso a `SSISDB` para despliegue en catálogo de Integration Services.

Configuración
-------------
1. Abrir la solución en Visual Studio / SSDT.
2. Configurar los Connection Managers usados por los paquetes (ej.: `SourceDB`, `StagingDB`, `DW`) con cadenas válidas.
3. Revisar y ajustar variables de entorno/paquete si existen (paths, batch sizes, etc.).
4. Verificar propiedades relevantes como `MaximumErrorCount` en cada paquete/contendor.

Ejecución
---------
- Para pruebas locales:
  - Abrir `Master_Pipeline.dtsx` y ejecutar el paquete (por ejemplo usando __Debug > Start Debugging__ o ejecutar el paquete individual con __Execute Package__).
  - También puede ejecutar paquetes individuales (`Extract_To_Staging.dtsx`, `Transform_and_Load_Dimensions.dtsx`, `Transform_and_Load_FactTable_OrderDetails.dtsx`) para aislar problemas.
- En producción:
  - Desplegar al catálogo `SSISDB` y programar/ejecutar desde SQL Server Agent o desde SSIS catalog.

Logging y manejo de errores
--------------------------
- Habilitar logging de eventos `OnError`, `OnWarning`, `OnTaskFailed` para capturar detalles.
- Configurar salidas de error en componentes de flujo de datos para `Redirect row` y almacenar filas erróneas en una tabla/archivo para análisis (incluir `ErrorCode`, `ErrorColumn`).
- Nota importante: la propiedad `MaximumErrorCount` por defecto es `1`. Si el número de errores supera ese umbral, el paquete marcará Failure (véase logs con `DTS_W_MAXIMUMERRORCOUNTREACHED`). Aumentar este valor solo temporalmente mientras se corrigen las causas raíz; la solución adecuada es capturar y tratar filas erróneas.
- Usar Data Viewers y puntos de interrupción en Visual Studio para depuración interactiva.

Problemas comunes y soluciones rápidas
-------------------------------------
- Conversiones/longitudes que truncen columnas: revisar tipos y longitudes en metadatos.
- Lookups que fallan por claves ausentes: usar `Redirect row` o mantener dimensiones referenciales actualizadas.
- Violaciones de constraints al insertar: validar datos en staging antes de insertar en DW.
- Si ves advertencias `DTS_W_MAXIMUMERRORCOUNTREACHED`, inspecciona los eventos `OnError` previos para identificar los componentes y columnas afectadas.

Buenas prácticas
----------------
- Mantener transformaciones idempotentes para permitir re-ejecuciones seguras.
- Registrar filas de error con contexto suficiente para reproducir y corregir.
- Versionar paquetes y documentar cambios en el repositorio.
- Automatizar despliegues al catálogo SSIS para entornos controlados.

Contribuir
----------
1. Crear un fork y una rama feature/bugfix.
2. Añadir cambios y pruebas relevantes.
3. Abrir un Pull Request con descripción del cambio.

Licencia
--------
Incluye o añade la licencia que corresponda (por ejemplo `LICENSE.md` en la raíz). Si no se especifica, añadir una licencia explícita antes de usar en producción.

Contacto
--------
Para problemas específicos de ejecución o errores, añade los logs `OnError` completos y los mensajes previos a `DTS_W_MAXIMUMERRORCOUNTREACHED` y se puede ayudar a diagnosticar la causa exacta.
