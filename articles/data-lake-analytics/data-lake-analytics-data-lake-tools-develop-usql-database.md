---
title: 'Desarrollo de un proyecto de base de datos de U-SQL: Azure Data Lake'
description: Aprenda a desarrollar una base de datos de U-SQL mediante Herramientas de Azure Data Lake para Visual Studio.
author: liudan66
ms.author: liud
ms.reviewer: jasonh
ms.assetid: 66dd58b1-0b28-46d1-aaae-43ee2739ae0a
ms.service: data-lake-analytics
ms.topic: how-to
ms.workload: big-data
ms.date: 07/03/2018
ms.openlocfilehash: ee35385b88bf4fbd5f899fde032b11b99a20d050
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87130042"
---
# <a name="use-a-u-sql-database-project-to-develop-a-u-sql-database-for-azure-data-lake"></a>Uso de un proyecto de base de datos de U-SQL para desarrollar una base de datos de U-SQL para Azure Data Lake

La base de datos de U-SQL proporciona vistas estructuradas a través de los datos no estructurados y datos estructurados administrados en las tablas. También proporciona un sistema de catálogo de metadatos generales para organizar los datos estructurados y el código personalizado. La base de datos es el concepto que agrupa estos objetos relacionados.

Obtenga más información sobre la [base de datos de U-SQL y el lenguaje de definición de datos (DDL)](/u-sql/data-definition-language-ddl-statements). 

El proyecto de base de datos de U-SQL es un tipo de proyecto de Visual Studio que ayuda a los desarrolladores a desarrollar, administrar e implementar sus bases de datos de U-SQL rápida y fácilmente.

## <a name="create-a-u-sql-database-project"></a>Creación de un proyecto de base de datos de U-SQL

Las Herramientas de Azure Data Lake para Visual Studio agregaron una nueva plantilla de proyecto llamada proyecto de base de datos de U-SQL en la versión 2.3.3000.0. Para crear un proyecto de U-SQL, seleccione **Archivo > Nuevo > Proyecto**. El proyecto de base de datos de U-SQL puede encontrarse en **Azure Data Lake > Nodo U-SQL**.

![Herramientas de Data Lake para Visual Studio: Creación de proyecto de base de datos de U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-create-usql-database-project-creation.png) 

## <a name="develop-u-sql-database-objects-by-using-a-database-project"></a>Desarrollo de objetos de bases de datos de U-SQL mediante un proyecto de base de datos

Haga clic con el botón derecho en el proyecto de base de datos de U-SQL. Seleccione **Agregar > Nuevo elemento**. Puede encontrar todos los nuevos tipos de objetos admitidos en el Asistente para **Agregar nuevo elemento**. 

Para un objeto no ensamblado (por ejemplo, una función con valores de tabla), se crea un nuevo script U-SQL después de agregar el nuevo elemento. Puede empezar a desarrollar la instrucción DDL para ese objeto en el editor.

Para un objeto de ensamblado, la herramienta proporciona un editor de interfaz de usuario fácil de usar que le ayuda a registrar el ensamblado e implementar archivos DLL y otros adicionales. Estos pasos le muestran cómo agregar una definición de objeto de ensamblado al proyecto de base de datos de U-SQL:

1.  Agregue las referencias del proyecto de C# que incluya UDO/UDAG/UDF para el proyecto de base de datos de U-SQL.

    ![Herramientas de Data Lake para Visual Studio: Adición de referencia de proyecto de base de datos de U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-project-reference.png) 

    ![Herramientas de Data Lake para Visual Studio: Adición de referencia de proyecto de base de datos de U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-project-reference-wizard.png)

2.  En la vista de diseño del ensamblado, elija el ensamblado referenciado en el menú desplegable **Create assembly from reference** (Crear el ensamblado desde la referencia).

    ![Herramientas de Data Lake para Visual Studio: Creación de un ensamblado a partir de una referencia](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-create-assembly-from-reference.png)

3.  Agregue **Dependencias administradas** y **Archivos adicionales**, si los hay. Al agregar archivos adicionales, la herramienta usa la ruta de acceso relativa para asegurarse de que puede encontrar los ensamblados tanto en la máquina local como en la máquina de compilación más adelante. 

@_DeployTempDirectory es una variable predefinida que dirige la herramienta a la carpeta de salida de compilación. En la carpeta de salida de compilación, cada ensamblado tiene una subcarpeta denominada con el nombre del ensamblado. Todos los archivos adicionales y DLL están en esa subcarpeta. 
 
## <a name="build-a-u-sql-database-project"></a>Compilación de un proyecto de base de datos U-SQL

La salida de compilación del proyecto de base de datos U-SQL es un paquete de implementación de la base de datos U-SQL, denominado con el sufijo `.usqldbpack`. El paquete `.usqldbpack` es un archivo ZIP que incluye todas las instrucciones DDL en un único script U-SQL en la carpeta **DDL** y todos los archivos DLL y archivos adicionales para los ensamblados en la carpeta **Temp**.

Mas información sobre [cómo compilar un proyecto de base de datos U-SQL con la línea de comandos de MSBuild y la tarea de compilación de Azure DevOps Services](data-lake-analytics-cicd-overview.md).

## <a name="deploy-a-u-sql-database"></a>Implementación de la base de datos U-SQL

El paquete .usqldbpack puede implementarse tanto en una cuenta local como en una cuenta de Azure Data Lake Analytics mediante Visual Studio o el SDK de implementación. 

### <a name="deploy-a-u-sql-database-in-visual-studio"></a>Implementación de una base de datos U-SQL en Visual Studio

Puede implementar una base de datos de U-SQL mediante un proyecto de base de datos de U-SQL o un paquete .usqldbpack en Visual Studio.

#### <a name="deploy-through-a-u-sql-database-project"></a>Implementación mediante un proyecto de base de datos U-SQL

1.  Haga clic con el botón derecho en el proyecto de base de datos U-SQL y, después, seleccione **Implementar**.
2.  En el **Asistente para implementar la base de datos de U-SQL**, seleccione la **cuenta de ADLA** en la que desea implementar la base de datos. Se admite tanto una cuenta local como una cuenta de ADLA.
3.  El **origen de la base de datos** se rellena automáticamente y señala al paquete .usqldbpack en la carpeta de salida de compilación del proyecto.
4.  Escriba un nombre en **Nombre de base de datos** para crear una base de datos. Si ya hay una base de datos con el mismo nombre en la cuenta de Azure Data Lake Analytics de destino, todos los objetos definidos en el proyecto de base de datos se crean sin volver a crear la base de datos.
5.  Para implementar la base de datos de U-SQL, seleccione **Enviar**. Se cargarán todos los recursos (ensamblados y archivos adicionales) y un trabajo de U-SQL que incluya todas las instrucciones DDL enviadas.

    ![Herramientas de Data Lake para Visual Studio: Implementación un proyecto de base de datos U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-project.png)

    ![Herramientas de Data Lake para Visual Studio: Implementación mediante el asistente del proyecto de base de datos de U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-project-wizard.png)

#### <a name="deploy-through-a-u-sql-database-deployment-package"></a>Implementación mediante el paquete de implementación de base de datos de U-SQL

1.  Abra el **Explorador de servidores**. A continuación, expanda la **cuenta de Azure Data Lake Analytics** en la que desee implementar la base de datos.
2.  Haga clic con el botón derecho en **U-SQL Databases** (Bases de datos U-SQL) y seleccione **Implementar base de datos**.
3.  Defina **Origen de la base de datos** como la ruta de acceso del paquete de implementación de la base de datos de U-SQL (archivo .usqldbpack).
4.  Escriba el **Nombre de base de datos** para crear una base de datos. Si ya hay una base de datos con el mismo nombre en la cuenta de Azure Data Lake Analytics de destino, todos los objetos definidos en el proyecto de base de datos se crean sin volver a crear la base de datos.

    ![Herramientas de Data Lake para Visual Studio: Implementación de un paquete de base de datos U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-package.png)

    ![Herramientas de Data Lake para Visual Studio: Implementación mediante el asistente de paquetes de base de datos U-SQL](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-deploy-usql-database-package-wizard.png)
  
### <a name="deploy-u-sql-database-by-using-the-sdk"></a>Implementación de una base de datos U-SQL mediante el SDK

`PackageDeploymentTool.exe` proporciona interfaces de programación y línea de comandos que ayudan a implementar bases de datos de U-SQL. El SDK se incluye en el [paquete Nuget del SDK de U-SQL](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/), que se encuentra en `build/runtime/PackageDeploymentTool.exe`.

[Más información sobre el SDK y cómo configurar la canalización de CI/CD para la implementación de bases de datos de U-SQL](data-lake-analytics-cicd-overview.md).

## <a name="reference-a-u-sql-database-project"></a>Referencia de un proyecto de base de datos de U-SQL

Un proyecto de U-SQL puede hacer referencia a un proyecto de base de datos de U-SQL. La referencia afecta a dos cargas de trabajo:

- *Compilación del proyecto*: configure los entornos de bases de datos referenciadas antes de compilar los scripts U-SQL. 
- *Ejecución local en cuenta (local-project)* : los entornos de bases de datos referenciadas se implementan en la cuenta (local-project) antes de la ejecución del script U-SQL. [Más información sobre la ejecución local y las diferencias entre las cuentas (local-machine) y (local-project)](data-lake-analytics-data-lake-tools-local-run.md).

### <a name="how-to-add-a-u-sql-database-reference"></a>Procedimiento para agregar una referencia de base de datos U-SQL

1. Haga clic con el botón derecho en el **Explorador de soluciones** y, después, elija **Agregar referencia de base de datos U-SQL**.

    ![Herramientas de Data Lake para Visual Studio: Adición de una referencia de proyecto de base de datos](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-database-project-reference.png)

2. Configure una referencia de base de datos desde un proyecto de base de datos U-SQL en la solución actual o un archivo de paquete de base de datos U-SQL.
3. Proporcione el nombre de la base de datos.

    ![Asistente para la adición de referencia de proyecto de base de datos de Herramientas de Data Lake para Visual Studio](./media/data-lake-analytics-data-lake-tools-develop-usql-database/data-lake-tools-add-database-project-reference-wizard.png)

## <a name="next-steps"></a>Pasos siguientes

- [Procedimiento para configurar una canalización de CI/CD para Azure Data Lake Analytics](data-lake-analytics-cicd-overview.md)
- [Prueba de código de Azure Data Lake Analytics](data-lake-analytics-cicd-test.md)
- [Ejecución local de scripts U-SQL](data-lake-analytics-data-lake-tools-local-run.md)
