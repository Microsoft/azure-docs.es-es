---
title: 'Uso de paquetes DACPAC y BACPAC de SQL Database: Azure SQL Edge'
description: Obtenga información sobre el uso de paquetes DACPAC y BACPAC en Azure SQL Edge
keywords: SQL Edge, sqlpackage
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/03/2020
ms.openlocfilehash: 6c8be6e67b1d7b919d6ea221c473c8975e559658
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90887482"
---
# <a name="sql-database-dacpac-and-bacpac-packages-in-sql-edge"></a>Paquetes DACPAC y BACPAC de SQL Database en SQL Edge

Azure SQL Edge es un motor de base de datos relacional optimizado orientado a implementaciones de IoT y Edge. Se basa en las versiones más recientes del motor de base de datos de Microsoft SQL Database, que proporciona funcionalidades de rendimiento, seguridad y procesamiento de consultas líderes en el sector. Junto con las capacidades de administración de bases de datos relacionales líderes del sector de SQL Server, Azure SQL Edge proporciona funcionalidad de streaming integrada para el análisis en tiempo real y el procesamiento de eventos complejos.

Azure SQL Edge también proporciona una implementación nativa de SqlPackage.exe que le permite implementar un paquete [DACPAC y BACPAC de SQL Database](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/data-tier-applications) durante la implementación de SQL Edge. 

Los paquetes DACPAC y BACPAC de SQL Database se pueden implementar en SQL Edge mediante la variable de entorno `MSSQL_PACKAGE`. La variable de entorno se puede configurar con cualquiera de las opciones siguientes.  
- Una ubicación de carpeta local dentro del contenedor de SQL que contiene los archivos dacpac y bacpac. Esta carpeta se puede asignar a un volumen de host mediante puntos de montaje o contenedores de volúmenes de datos. 
- Una ruta de acceso de archivo local dentro de la asignación del contenedor SQL al archivo dacpac o bacpac. Esta ruta de acceso de archivo se puede asignar a un volumen de host mediante puntos de montaje o contenedores de volúmenes de datos. 
- Una ruta de acceso de archivo local dentro de la asignación del contenedor SQL a un archivo ZIP que contiene archivos dacpac o bacpac. Esta ruta de acceso de archivo se puede asignar a un volumen de host mediante puntos de montaje o contenedores de volúmenes de datos. 
- Una dirección URL de SAS de blob de Azure a un archivo ZIP que contiene los archivos dacpac y bacpac.
- Una dirección URL de SAS de blob de Azure a un archivo dacpac o bacpac. 

## <a name="use-a-sql-database-dac-package-with-sql-edge"></a>Uso de un paquete DAC de SQL Database con SQL Edge

Para implementar (o importar) un paquete DAC de SQL Database `(*.dacpac)` o un archivo BACPAC `(*.bacpac)` con Azure Blob Storage y un archivo ZIP, siga los pasos que se indican a continuación. 

1. Cree o extraiga un paquete DAC o exporte un archivo Bacpac mediante el mecanismo siguiente. 
    - Cree o extraiga un paquete DAC de SQL Database. Consulte [Extracción de un paquete DAC de una base de datos](/sql/relational-databases/data-tier-applications/extract-a-dac-from-a-database/) para más información sobre cómo generar un paquete DAC para una base de datos de SQL Server existente.
    - Exportación de un paquete DAC implementado o una base de datos. Vea [Exportación de una aplicación de capa de datos](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/export-a-data-tier-application/) para obtener información sobre cómo generar un archivo bacpac para una base de datos de SQL Server existente.

2. Comprima el archivo `*.dacpac` o `*.bacpac`, y cárguelo en una cuenta de Azure Blob Storage. Para más información sobre la carga de archivos en Azure Blob Storage, consulte [Carga, descarga y enumeración de blobs con Azure Portal](../storage/blobs/storage-quickstart-blobs-portal.md).

3. Genere una firma de acceso compartido para el archivo ZIP mediante Azure Portal. Para más información, consulte [Delegación de acceso con firmas de acceso compartidas (SAS)](../storage/common/storage-sas-overview.md).

4. Actualice la configuración del módulo SQL Edge para incluir el URI de acceso compartido para el paquete DAC. Para actualizar el módulo SQL Edge, realice estos pasos:

    1. En Azure Portal, vaya a la implementación de IoT Hub.

    2. En el panel izquierdo, seleccione **IoT Edge**.

    3. En la página de **IoT Edge**, busque y seleccione la instancia de IoT Edge en la que se implementa el módulo de SQL Edge.

    4. En la página de **Dispositivo de IoT Edge**, seleccione **Establecer módulo**.

    5. En la página **Establecer módulos**, haga clic en el módulo Azure SQL Edge.

    6. En el panel **Actualizar módulo IoT Edge**, seleccione **Variables de entorno**. Agregue la variable de entorno `MSSQL_PACKAGE` y especifique la dirección URL de SAS generada en el paso 3 anterior como el valor de la variable de entorno. 

    7. Seleccione **Actualizar**.

    8. En la página **Establecer módulos**, seleccione **Revisar y crear**.

    9. En la página **Establecer módulos**, seleccione **Crear**.

5. Después de actualizar el módulo, los archivos de paquete se descargan, se descomprimen y se implementan en la instancia de SQL Edge.

En cada reinicio del contenedor de Azure SQL Edge, SQL Edge intenta descargar el archivo comprimido y evalúa si hay cambios. Si se encuentra una nueva versión del archivo dacpac, los cambios se implementan en la base de datos de SQL Edge.

## <a name="next-steps"></a>Pasos siguientes

- [Implementación de SQL Edge mediante Azure Portal](deploy-portal.md).
- [Datos de streaming](stream-data.md)
- [Aprendizaje automático e IA con ONNX en SQL Edge](onnx-overview.md)
