---
title: Notas de la versión de Data Management Gateway
description: Notas de la versión de Data Management Gateway
services: data-factory
author: nabhishek
manager: anandsub
ms.assetid: 14762e82-76d9-41c4-ba9f-14a54da29c36
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: abnarain
robots: noindex
ms.openlocfilehash: d8324b0b861829b02b589f8145121559114b7435
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89289586"
---
# <a name="release-notes-for-data-management-gateway"></a>Notas de la versión de Data Management Gateway
> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si usa la versión actual del servicio Data Factory, consulte el artículo sobre [Integration Runtime autohospedado en V2](../create-self-hosted-integration-runtime.md).

Uno de los desafíos de la integración de datos moderna es la migración de datos entre la ubicación local y la nube. Data Factory realiza la integración con Data Management Gateway, un agente que se puede instalar de forma local para permitir la migración de datos híbridos.

En los siguientes artículos se proporciona información detallada sobre Data Management Gateway y cómo utilizarlo:

*  [Data Management Gateway](data-factory-data-management-gateway.md)
*  [Movimiento de datos entre una ubicación local y la nube mediante Azure Data Factory](data-factory-move-data-between-onprem-and-cloud.md)


## <a name="current-version"></a>VERSIÓN ACTUAL 
Ya no se conservan las notas de la versión aquí. Puede obtener las notas de la versión más recientes [aquí](https://go.microsoft.com/fwlink/?linkid=853077).




## <a name="earlier-versions"></a>Versiones anteriores
## <a name="21063477"></a>2.10.6347.7
### <a name="enhancements-"></a>Mejoras-
- Puede agregar entradas DNS para permitir Service Bus en vez de permitir todas las direcciones IP de Azure del firewall (en caso necesario). Puede encontrar la entrada DNS correspondiente en Azure Portal (Data Factory -> "Crear e implementar" -> "Puertas de enlace" -> "serviceUrls" (en JSON)
- El conector HDFS ahora admite certificados públicos autofirmados al permitirle omitir la validación de TLS.
- Solucionado: Problema con la puerta de enlace sin conexión durante la actualización (debido a la distorsión del reloj)


## <a name="2963132"></a>2.9.6313.2
### <a name="enhancements-"></a>Mejoras-
-   Puede agregar entradas DNS para permitir Service Bus en vez de permitir todas las direcciones IP de Azure del firewall (en caso necesario). Más detalles aquí.
-   Ahora puede copiar hasta 4,75 TB de datos (tamaño máximo admitido para blob en bloques) en un único blob en bloques, y viceversa (el límite anterior era 195 GB).
-   Solucionado: memoria agotada al descomprimir varios archivos pequeños durante la actividad de copia.
-   Solucionado: problema de índice fuera de rango al copiar de Document DB a SQL Server con la característica de idempotencia.
-   Solucionado: el script de limpieza SQL no funciona con SQL Server desde el Asistente para copia.
-   Solucionado: el nombre de columna con espacio al final no funciona en la actividad de copia.

## <a name="28662833"></a>2.8.66283.3
### <a name="enhancements-"></a>Mejoras-
- Solucionado: Problema con la falta de credenciales durante el reinicio de la máquina de puerta de enlace.
- Solucionado: problema de registro durante la restauración de la puerta de enlace mediante un archivo de copia de seguridad.


## <a name="2762401"></a>2.7.6240.1
### <a name="enhancements-"></a>Mejoras-
- Solucionado: lectura incorrecta de valor NULL decimal desde Oracle como origen.

## <a name="2661922"></a>2.6.6192.2
### <a name="whats-new"></a>Novedades
- Los clientes pueden proporcionar comentarios en la experiencia de registro de la puerta de enlace.
- Nuevo formato de compresión compatible: ZIP (Deflate)

### <a name="enhancements-"></a>Mejoras-
- Mejora del rendimiento del origen de HDFS y el receptor de Oracle.
- Corrección de errores de la capacidad de procesamiento en paralelo y la actualización automática de la puerta de enlace.


## <a name="2561641"></a>2.5.6164.1
### <a name="enhancements"></a>Mejoras
- Experiencia de registro de la puerta de enlace mejorada y más sólida: ahora puede realizar el seguimiento del estado del progreso durante el proceso de registro de la puerta de enlace, con lo que la experiencia de registro tiene una mayor capacidad de respuesta.
- Mejora del proceso de restauración de la puerta de enlace: puede recuperar la puerta de enlace incluso si no tiene el archivo de copia de seguridad de la puerta de enlace con esta actualización. Esto requeriría restablecer credenciales de servicio vinculado en el Portal.
- Corrección de errores.

## <a name="2461511"></a>2.4.6151.1

### <a name="whats-new"></a>Novedades

- Ahora puede almacenar localmente las credenciales de origen de datos. Las credenciales se cifran. Las credenciales del origen de datos se pueden recuperar y restaurar usando el archivo de copia de seguridad que se puede exportar desde la puerta de enlace existente, todo de manera local.

### <a name="enhancements-"></a>Mejoras-

- Experiencia de registro de puerta de enlace mejorada y más eficaz.
- Compatibilidad con detección automática de la configuración de QuoteChar para formato de texto en el Asistente para copia y mejorar la precisión de la detección de formato general.

## <a name="2361002"></a>2.3.6100.2

- Compatibilidad con la detección automática de firstRowAsHeader y SkipLineCount en el Asistente para copia para archivos de texto en el sistema de archivos local y HDFS.
- Mejora de la estabilidad de la conexión de red entre la puerta de enlace y Service Bus
- Unas pocas correcciones de errores


## <a name="2260721"></a>2.2.6072.1

*  Admite la configuración de proxy HTTP para la puerta de enlace con el Administrador de configuración de Data Management Gateway. Si está configurado, se accede a Blob de Azure, Tabla de Azure, Azure Data Lake y DocumentDB a través del proxy HTTP.
*  Admite el control de encabezados para TextFormat al copiar datos en Blob de Azure, Azure Data Lake Store, el sistema de archivos local y el HDFS local como origen y destino.
*  Admite la copia de datos desde blobs en anexos y blobs en páginas junto con los blobs en bloques que ya eran compatibles.
*  Introduce un nuevo estado de puerta de enlace **Online (Limited)** (En línea [limitado]), lo que indica que se puede utilizar sin problemas la funcionalidad principal de la puerta de enlace, pero la operación interactiva del Asistente para copia.
*  Mejora la estabilidad del registro de puerta de enlace mediante la clave de registro.

## <a name="216040"></a>2.1.6040.

*  El controlador DB2 se incluye ahora en el paquete de instalación de la puerta de enlace. No es necesario instalarlo por separado.
*  El controlador DB2 ahora admite z/OS y DB2 para i (AS / 400) junto con las plataformas ya admitidas (Linux, Unix y Windows).
*  Admite el uso de Azure Cosmos DB como origen o destino para almacenes de datos locales
*  Admite la copia de datos desde/a blobs de almacenamiento en frío y en caliente junto con la cuenta de almacenamiento general ya admitida.
*  Le permite conectarse a SQL Server a través de la puerta de enlace con privilegios de inicio de sesión remoto.  

## <a name="2060131"></a>2.0.6013.1

*  Puede seleccionar el idioma o la referencia cultural que utilizará una puerta de enlace durante la instalación manual.

*  Cuando la puerta de enlace no funciona del modo previsto, puede optar por enviar los registros de la puerta de enlace de los últimos 7 días a Microsoft para que pueda solucionar el problema. Si la puerta de enlace no está conectada al servicio en la nube, puede guardar y archivar los registros de la puerta de enlace.  

*  Mejoras de la interfaz de usuario del Administrador de configuración de puertas de enlace:

    *  Mayor visibilidad del estado de la puerta de enlace en la pestaña Inicio.

    *  Controles reorganizados y simplificados.

    *  Puede copiar datos desde un almacenamiento mediante la [herramienta de copia sin código](data-factory-copy-data-wizard-tutorial.md). Para obtener más información general sobre esta característica consulte [Copias almacenadas provisionalmente](data-factory-copy-activity-performance.md#staged-copy) .
*  Puede usar Data Management Gateway para transferir los datos directamente desde una base de datos de SQL Server hasta Azure Machine Learning.

*  Mejoras en el rendimiento

    * Se ha mejorado el rendimiento al visualizar los esquemas y la vista previa en SQL Server, mediante la herramienta de copia sin código.

## <a name="11259531"></a>1.12.5953.1

*  Corrección de errores

## <a name="11159181"></a>1.11.5918.1

*  El tamaño máximo del registro de eventos de puerta de enlace ha aumentado de 1 MB a 40 MB.

*  En caso de que se requiera un reinicio durante la actualización automática de la puerta de enlace, se muestra un cuadro de diálogo de advertencia. Puede elegir reiniciar en ese mismo momento o más adelante.

*  En caso de error en la actualización automática, el instalador de puerta de enlace volverá a intentar esta operación 3 veces al máximo.

*  Mejoras en el rendimiento

    * Rendimiento mejorado para la carga de tablas de gran tamaño desde el servidor local en un escenario de copia sin código.

*  Corrección de errores

## <a name="11058921"></a>1.10.5892.1

*  Mejoras en el rendimiento

*  Corrección de errores

## <a name="1958652"></a>1.9.5865.2

*  Funcionalidad de actualización automática sin intervención del usuario
*  Icono de bandeja de nuevo con indicadores de estado de la puerta de enlace
*  Capacidad para "Actualizar ahora" desde el cliente
*  Capacidad para establecer la hora de programación de las actualizaciones
*  Script de PowerShell para activar y desactivar la actualización automática
*  Compatibilidad con formato JSON  
*  Mejoras en el rendimiento
*  Corrección de errores

## <a name="1858221"></a>1.8.5822.1

*  Mejora de la solución de problemas
*  Mejoras en el rendimiento
*  Corrección de errores

### <a name="1757951"></a>1.7.5795.1

*  Mejoras en el rendimiento
*  Corrección de errores

### <a name="1757641"></a>1.7.5764.1

*  Mejoras en el rendimiento
*  Corrección de errores

### <a name="1657351"></a>1.6.5735.1

*  Compatibilidad origen y receptor HDFS locales
*  Mejoras en el rendimiento
*  Corrección de errores

### <a name="1656961"></a>1.6.5696.1

*  Mejoras en el rendimiento
*  Corrección de errores

### <a name="1656761"></a>1.6.5676.1

*  Compatibilidad con herramientas de diagnóstico de Administrador de configuración
*  Columnas de la tabla de soporte técnico para orígenes de datos tabulares de Azure Data Factory
*  Compatibilidad con SQL DW para Azure Data Factory
*  Compatibilidad con Reclusive en BlobSource y FileSource para Azure Data Factory
*  Compatibilidad con CopyBehavior - MergeFiles, PreserveHierarchy y FlattenHierarchy en BlobSink y FileSink con copia binaria para Azure Data Factory
*  Compatibilidad con los informes de progreso de la actividad de copia para Azure Data Factory
*  Compatibilidad con la validación de conectividad de origen de datos para Azure Data Factory
*  Corrección de errores

### <a name="1656721"></a>1.6.5672.1

*  Compatibilidad con nombre de tabla de origen de datos ODBC para Azure Data Factory
*  Mejoras en el rendimiento
*  Corrección de errores

### <a name="1656581"></a>1.6.5658.1

*  Compatibilidad con receptor de archivos para Azure Data Factory
*  Compatibilidad con la conservación jerarquía en copia binaria para Azure Data Factory
*  Compatibilidad con idempotencia de la actividad de copia para Azure Data Factory
*  Corrección de errores

### <a name="1656401"></a>1.6.5640.1

*  Compatibilidad con tres orígenes de datos más para Azure Data Factory (ODBC, OData y HDFS)
*  Compatibilidad con carácter de comillas en el analizador de archivos .csv para Azure Data Factory
*  Compatibilidad con compresión (BZip2)
*  Corrección de errores

### <a name="1556121"></a>1.5.5612.1

*  Compatibilidad con cinco bases de datos relacionales para Azure Data Factory (MySQL, PostgreSQL, DB2, Teradata y Sybase)
*  Compatibilidad de compresión (Gzip y Deflate)
*  Mejoras en el rendimiento
*  Corrección de errores

### <a name="1455491"></a>1.4.5549.1

*  Incorporación de compatibilidad de origen de datos Oracle para Azure Data Factory
*  Mejoras en el rendimiento
*  Corrección de errores

### <a name="1454921"></a>1.4.5492.1

*  Binario unificado que admite los servicios Microsoft Azure Data Factory y Office 365 Power BI
*  Restricción de la interfaz de usuario de configuración y del proceso de registro
*  Azure Data Factory: compatibilidad de entrada y salida de Azure con origen de datos SQL Server

### <a name="1253031"></a>1.2.5303.1

*  Corrección del problema del tiempo de espera para admitir más conexiones de orígenes de datos que requieren mucho tiempo.

### <a name="1155268"></a>1.1.5526.8

*  Requiere .NET Framework 4.5.1 como requisito previo durante la instalación.

### <a name="1051442"></a>1.0.5144.2

*  No hay cambios que afecten a los escenarios de Azure Data Factory.
