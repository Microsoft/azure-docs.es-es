---
title: 'Inicio rápido: Carga masiva de datos con Synapse SQL'
description: Uso de Synapse Studio para cargar datos de forma masiva en Synapse SQL
services: synapse-analytics
author: kevinvngo
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 05/06/2020
ms.author: kevin
ms.reviewer: jrasnick
ms.openlocfilehash: fb618f155e659ecd1fa5760117193a9045e6da35
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87535134"
---
# <a name="bulk-loading-with-synapse-sql"></a>Carga masiva con Synapse SQL

La carga de datos nunca ha sido más fácil que cuando se usa el Asistente para carga masiva en Synapse Studio. Este asistente le guiará a través de la creación de un script de T-SQL con la [instrucción COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest) para cargar datos de forma masiva. 

## <a name="entry-points-to-the-bulk-load-wizard"></a>Puntos de entrada al Asistente para carga masiva

Ahora puede cargar datos de forma masiva fácilmente mediante grupos de SQL. Solo tiene que hacer clic con el botón derecho en las siguientes áreas en Synapse Studio:

- Un archivo o una carpeta de una cuenta de Azure Storage asociada al área de trabajo ![Hacer clic con el botón derecho en un archivo o una carpeta de una cuenta de almacenamiento](./sql/media/bulk-load/bulk-load-entry-point-0.png)

## <a name="prerequisites"></a>Requisitos previos

- Este asistente genera una instrucción COPY que usa el paso a través de AAD para la autenticación. Su [usuario de AAD debe tener acceso](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/quickstart-bulk-load-copy-tsql-examples#d-azure-active-directory-authentication-aad) al área de trabajo al menos con el rol de Azure Colaborador de datos de Storage Blob en la cuenta de ADLS Gen2.

- Debe tener los permisos necesarios [para usar la instrucción COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest#permissions) y tener permisos de creación de tablas si va a crear una tabla en la que cargar datos.

- El servicio vinculado asociado a la cuenta de Azure Data Lake Storage Gen2 **debe tener acceso al archivo**/**carpeta** que se va a cargar. Por ejemplo, si el mecanismo de autenticación del servicio vinculado es Identidad administrada, la identidad administrada del área de trabajo debe tener, al menos, permiso de lectura de Blob Storage en la cuenta de almacenamiento.

- Si hay una red virtual habilitada en el área de trabajo, asegúrese de que el runtime integrado asociado a la cuenta de Azure Data Lake Storage Gen2 los servicios vinculados para los datos de origen y la ubicación del archivo de error tiene habilitada la creación interactiva. La creación interactiva es necesaria para la detección automática de esquemas, la vista previa del contenido del archivo de origen y la exploración de cuentas de almacenamiento de Azure Data Lake Storage Gen2 en el asistente.

### <a name="steps"></a>Pasos

1. Seleccione la cuenta de almacenamiento y el archivo o la carpeta desde donde se va a realizar la carga en el panel Source storage location (Ubicación de almacenamiento de origen): ![Seleccionar una ubicación de origen](./sql/media/bulk-load/bulk-load-source-location.png)

2. Seleccione la configuración del formato de archivo, incluida la cuenta de almacenamiento en la que desea escribir las filas rechazadas (archivo de error). Actualmente solo se admiten archivos .csv y Parquet.

    ![Seleccionar configuración de formato de archivo](./sql/media/bulk-load/bulk-load-file-format-settings.png)

3. Puede hacer clic en "Preview data" (Vista previa de los datos) para ver cómo va a analizar el archivo la instrucción COPY para ayudarle en la configuración del formato de archivo. Haga clic en "Preview data" (Vista previa de los datos) cada vez que cambie la configuración del formato de archivo para ver cómo va a analizar el archivo la instrucción COPY con la configuración actualizada: ![Vista previa de los datos](./sql/media/bulk-load/bulk-load-file-format-settings-preview-data.png) 

4. Seleccione el grupo de SQL que va a usar para cargar, incluido si la carga será para una tabla existente o una nueva: ![Seleccionar ubicación de destino](./sql/media/bulk-load/bulk-load-target-location.png)

5. Haga clic en "Configure column mapping" (Configurar asignación de columnas) para asegurarse de que tiene la asignación de columnas adecuada. En el caso de las nuevas tablas, la configuración de la asignación de columnas es fundamental para actualizar los tipos de datos de la columna de destino: ![Configurar la asignación de columnas](./sql/media/bulk-load/bulk-load-target-location-column-mapping.png)

6. Haga clic en "Open script" (Abrir script) y se generará un script de T-SQL con la instrucción COPY para cargar desde un lago de datos: ![Abrir el script SQL](./sql/media/bulk-load/bulk-load-target-final-script.png)

## <a name="next-steps"></a>Pasos siguientes

- Consulte el artículo [Instrucción COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest#syntax) para más información sobre las funcionalidades de COPY
- Consulte el artículo [Introducción a la carga de datos](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/design-elt-data-loading#what-is-elt)
