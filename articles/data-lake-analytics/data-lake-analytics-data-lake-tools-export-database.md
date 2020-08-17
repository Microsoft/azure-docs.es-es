---
title: 'Exportación de bases de datos de U-SQL: Herramientas de Azure Data Lake para Visual Studio'
description: Obtenga información acerca de cómo usar Herramientas de Azure Data Lake para Visual Studio para exportar una base de datos de U-SQL e importarla automáticamente en una cuenta local.
services: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: dc9b21d8-c5f4-4f77-bcbc-eff458f48de2
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 11/27/2017
ms.openlocfilehash: 05e55d1608323b8e746a1c82ccb915a2db58ce97
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132099"
---
# <a name="export-a-u-sql-database"></a>Exportación de una base de datos de U-SQL

En este artículo, obtendrá información sobre cómo usar [Herramientas de Azure Data Lake para Visual Studio](https://aka.ms/adltoolsvs) para exportar una base de datos de U-SQL como un único script de U-SQL y recursos descargados. En el mismo proceso, puede importar la base de datos exportada en la cuenta local.

Normalmente, los clientes mantienen varios entornos de desarrollo, prueba y producción. Estos entornos se hospedan tanto en una cuenta local en el equipo local de los desarrolladores como en una cuenta de Azure Data Lake Analytics en Azure. 

Al desarrollar y optimizar las consultas U-SQL en entornos de desarrollo y pruebas, normalmente, los desarrolladores tienen que volver a crear su trabajo en una base de datos de producción. El Asistente para la exportación de bases de datos ayuda a acelerar este proceso. Mediante este asistente, los programadores pueden clonar el entorno de base de datos existente y los datos de ejemplo a otras cuentas de Data Lake Analytics.

## <a name="export-steps"></a>Pasos de exportación

### <a name="step-1-export-the-database-in-server-explorer"></a>Paso 1: Exportar la base de datos en el Explorador de servidores

Todas las cuentas de Data Lake Analytics para las que tiene permiso se muestran en el Explorador de servidores. Para exportar la base de datos, siga estos pasos:

1. En el Explorador de servidores, expanda la cuenta que contiene la base de datos que desea exportar.
2. Haga clic con el botón derecho en la base de datos y seleccione **Exportar**. 
   
    ![Explorador de servidores: exportación de una base de datos](./media/data-lake-analytics-data-lake-tools-export-database/export-database.png)

     Si la opción de menú **exportar** no está disponible, deberá [actualizar la herramienta a la última versión](https://aka.ms/adltoolsvs).

### <a name="step-2-configure-the-objects-that-you-want-to-export"></a>Paso 2: Configure los objetos que desea exportar

Si solo necesita una pequeña parte de una base de datos grande, puede configurar un subconjunto de objetos que desea exportar en el Asistente para exportación. 

La acción de exportación se realiza mediante la ejecución de un trabajo de U-SQL. Por lo tanto, la exportación desde una cuenta de Azure conlleva algunos costos.

![Asistente para exportación de base de datos: selección de Exportar objetos](./media/data-lake-analytics-data-lake-tools-export-database/export-database-wizard.png)

### <a name="step-3-check-the-objects-list-and-other-configurations"></a>Paso 3: Comprobar la lista de objetos y otras configuraciones

En este paso, puede comprobar los objetos seleccionados en la casilla **Exportar lista de objetos**. Si hay errores, seleccione **Anterior** para volver atrás y configurar correctamente los objetos que desea exportar.

También puede configurar otras opciones para el destino de exportación. Las descripciones de configuración se muestran en la tabla siguiente:

|Configuración|Descripción|
|-------------|-----------|
|Nombre del destino|Este nombre indica el lugar en el que desea guardar los recursos de base de datos exportados. Por ejemplo, ensamblados, archivos adicionales y datos de ejemplo. Se creará una carpeta con este nombre en la carpeta raíz de datos locales.|
|Directorio del proyecto|Esta ruta de acceso define donde desea guardar el script U-SQL exportado. Todas las definiciones de objeto de base de datos se guardan en esta ubicación.|
|Solo esquema|Al seleccionar esta opción, solo se exportan las definiciones y los recursos de la base de datos (como los ensamblados y los archivos adicionales).|
|Esquema y datos|Al seleccionar esta opción, se exportan las definiciones de base de datos, los recursos y los datos. Se exportan las primeras N filas de las tablas.|
|Importar de forma automática en la base de datos local|Al seleccionar esta opción, la base de datos exportada se importa automáticamente en la base de datos local cuando se completa la exportación.|

![Asistente para exportación de bases de datos: exportación de listas de objetos y otras configuraciones](./media/data-lake-analytics-data-lake-tools-export-database/export-database-wizard-configuration.png)

### <a name="step-4-check-the-export-results"></a>Paso 4: Compruebe los resultados de la exportación

Una vez finalizada la exportación, puede ver los resultados exportados en la ventana de registros del asistente. En el ejemplo siguiente se muestra cómo buscar scripts U-SQL y recursos de bases de datos exportados, incluidos ensamblados, archivos adicionales y datos de ejemplo:

![Asistente para exportación de bases de datos: exportación de resultados](./media/data-lake-analytics-data-lake-tools-export-database/export-database-wizard-completed.png)

## <a name="import-the-exported-database-to-a-local-account"></a>Importación de la base de datos exportada a una cuenta local

La forma de cómoda de importar la base de datos exportada consiste en activar la casilla **Importar de forma automática en la base de datos local** durante el proceso de exportación en el paso 3. Si no activa esta casilla, primero busque el script de U-SQL exportado en el registro de exportación. Después, ejecute el script de U-SQL localmente para importar la base de datos en la cuenta local.

## <a name="import-the-exported-database-to-a-data-lake-analytics-account"></a>Importación de la base de datos exportada en la cuenta de Data Lake Analytics

Para importar la base de datos exportada en otra cuenta de Data Lake Analytics, siga estos pasos:

1. Cargue los recursos exportados, incluidos los ensamblados, los archivos adicionales y los datos de ejemplo en la cuenta de Azure Data Lake Store predeterminada de la cuenta de Data Lake Analytics en la que desea importar. Puede encontrar la carpeta de recursos exportados en la carpeta raíz de datos locales. Cargue toda la carpeta en la raíz de la cuenta de Data Lake Store.
2. Cuando acabe el proceso de carga, envíe el script de U-SQL exportado a la cuenta de Data Lake Analytics en la que desea importar la base de datos.

## <a name="known-limitations"></a>Restricciones conocidas

Actualmente, si seleccionó **Esquema y datos** en el paso 3, la herramienta ejecuta un trabajo de U-SQL para exportar los datos almacenados en tablas. Por este motivo, el proceso de exportación de datos podría ser lento y conllevar algunos costos. 

## <a name="next-steps"></a>Pasos siguientes

* [Información sobre las bases de datos de U-SQL](/u-sql/data-definition-language-ddl-statements) 
* [Prueba y depuración de trabajos U-SQL mediante la ejecución local y el SDK de U-SQL para Azure Data Lake](data-lake-analytics-data-lake-tools-local-run.md)


