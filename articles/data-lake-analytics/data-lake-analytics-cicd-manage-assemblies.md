---
title: 'Administración de ensamblados de U-SQL en una canalización de CI/CD: Azure Data Lake'
description: Conozca el procedimiento recomendado para administrar ensamblados de C# de U-SQL en una canalización de CI/CD con Azure DevOps.
author: liudan66
ms.author: liud
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 10/30/2018
ms.openlocfilehash: 4bb6ee60df291c1939d3bb0d72a9b3992be9b3c0
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132150"
---
# <a name="best-practices-for-managing-u-sql-assemblies-in-a-cicd-pipeline"></a>Procedimiento recomendado para administrar ensamblados de U-SQL en una canalización de CI/CD

En este artículo, aprenderá a administrar código fuente de ensamblados de U-SQL con el nuevo proyecto de base de datos U-SQL incorporado. También aprenderá a configurar una canalización de integración e implementación continua (CI/CD) para el registro de ensamblados con Azure DevOps.

## <a name="use-the-u-sql-database-project-to-manage-assembly-source-code"></a>Usar el proyecto de base de datos U-SQL para administrar el código fuente de ensamblados

[El proyecto de base de datos U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md) es un tipo de proyecto de Visual Studio que ayuda a los desarrolladores a desarrollar, administrar e implementar sus bases de datos U-SQL rápida y fácilmente. Puede administrar todos los objetos de base de datos U-SQL (excepto las credenciales) con el proyecto de base de datos U-SQL. 

Para administrar el código fuente de ensamblados de C# y los scripts U-SQL de DDL del registro de ensamblados, utilice:

* El proyecto de base de datos U-SQL para administrar scripts U-SQL del registro de ensamblados.
* La biblioteca de clases (para la aplicación U-SQL) para administrar el código fuente de C# y las dependencias de los operadores, las funciones y los agregadores definidos por el usuario (UDO, UDF y UDAG).
* El proyecto de base de datos U-SQL para hacer referencia al proyecto de biblioteca de clases. 

Un proyecto de base de datos U-SQL puede hacer referencia a un proyecto de biblioteca de clases (para la aplicación U-SQL). Se pueden crear ensamblados registrados en la base de datos U-SQL mediante el código fuente de C# de referencia desde este proyecto de biblioteca de clases (para la aplicación U-SQL).

Siga estos pasos para crear proyectos y agregarles referencias.
1. Crear un proyecto de biblioteca de clases (para la aplicación U-SQL) seleccionando **Archivo** > **Nuevo** > **Proyecto**. El proyecto se encuentra bajo el nodo **Azure Data Lake > U-SQL**.

   ![Herramientas de Data Lake para Visual Studio: Creación de proyecto de biblioteca de clases de C#](./media/data-lake-analytics-cicd-manage-assemblies/create-c-sharp-class-library-project.png)
1. Agregar el código de C# definido por el usuario en el proyecto de biblioteca de clases (para la aplicación U-SQL).

1. Cree un proyecto de U-SQL, para lo que debe seleccionar **Archivo** >  **Nuevo** > **Proyecto**. El proyecto se encuentra bajo el nodo **Azure Data Lake** > **U-SQL**.

   ![Herramientas de Data Lake para Visual Studio: Creación de proyecto de base de datos U-SQL](media/data-lake-analytics-cicd-manage-assemblies/create-u-sql-database-project.png)
1. Agregar una referencia al proyecto de biblioteca de clases de C# para el proyecto de base de datos U-SQL.

    ![Herramientas de Data Lake para Visual Studio: Adición de referencia de proyecto de base de datos de U-SQL](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-add-project-reference.png) 

    ![Herramientas de Data Lake para Visual Studio: Adición de referencia de proyecto de base de datos de U-SQL](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-add-project-reference-wizard.png)

5. Para crear un script de ensamblado en el proyecto de base de datos U-SQL, haga clic con el botón derecho en el proyecto y seleccione **Agregar nuevo elemento**.

   ![Herramientas de Data Lake para Visual Studio: Adición de script de ensamblado](media/data-lake-analytics-cicd-manage-assemblies/add-assembly-script.png)

1. Abra el script de ensamblado en la vista de diseño del ensamblado. Seleccione el ensamblado referenciado en el menú desplegable **Cree el ensamblado desde la referencia**.

    ![Herramientas de Data Lake para Visual Studio: Creación de un ensamblado a partir de una referencia](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-create-assembly-from-reference.png)

7. Agregue **Dependencias administradas** y **Archivos adicionales**, si los hay. Al agregar archivos adicionales, la herramienta usa la ruta de acceso relativa para asegurarse de que puede encontrar los ensamblados en la máquina local y en la máquina de compilación más adelante.

**\@_DeployTempDirectory** en la ventana del editor de la parte inferior es una variable predefinida que dirige la herramienta a la carpeta de salida de compilación. En la carpeta de salida de compilación, cada ensamblado tiene una subcarpeta denominada con el nombre del ensamblado. Todos los archivos adicionales y DLL están en esa subcarpeta.

## <a name="build-a-u-sql-database-project"></a>Compilación de un proyecto de base de datos U-SQL

La salida de compilación del proyecto de base de datos U-SQL es un paquete de implementación de la base de datos U-SQL, denominado con el sufijo `.usqldbpack`. El paquete `.usqldbpack` es un archivo ZIP que incluye todas las instrucciones DDL de un único script U-SQL en una carpeta DDL. Todos los archivos .dll compilados y archivos adicionales para los ensamblados están en la carpeta Temp.

## <a name="deploy-a-u-sql-database"></a>Implementación de la base de datos U-SQL

El paquete `.usqldbpack` puede implementarse tanto en una cuenta local como en una cuenta de Azure Data Lake Analytics. Use Visual Studio o el SDK de implementación. 

### <a name="deploy-a-u-sql-database-in-visual-studio"></a>Implementación de una base de datos U-SQL en Visual Studio

Puede implementar una base de datos U-SQL mediante un proyecto de base de datos U-SQL o un paquete `.usqldbpack` en Visual Studio.

#### <a name="deploy-by-using-a-u-sql-database-project"></a>Implementación mediante un proyecto de base de datos U-SQL

1.  Haga clic con el botón derecho en el proyecto de base de datos U-SQL y, después, seleccione **Implementar**.
2.  En el **Asistente para implementar la base de datos de U-SQL**, seleccione la **cuenta de ADLA** en la que desea implementar la base de datos. Se admite tanto una cuenta local como una cuenta de ADLA.
3.  El **origen de la base de datos** se rellena automáticamente. Señala al paquete .usqldbpack en la carpeta de salida de compilación del proyecto.
4.  Escriba un nombre en **Nombre de base de datos** para crear una base de datos. Si ya hay una base de datos con el mismo nombre en la cuenta de Azure Data Lake Analytics de destino, todos los objetos definidos en el proyecto de base de datos se crean sin volver a crear la base de datos.
5.  Para implementar la base de datos de U-SQL, seleccione **Enviar**. Se cargan todos los recursos, como ensamblados y archivos adicionales. Se envía un trabajo de U-SQL que incluye todas las instrucciones de DDL.

    ![Herramientas de Data Lake para Visual Studio: Implementación un proyecto de base de datos U-SQL](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-deploy-usql-database-project.png)

    ![Herramientas de Data Lake para Visual Studio: Implementación mediante el asistente del proyecto de base de datos de U-SQL](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-deploy-usql-database-project-wizard.png)

### <a name="deploy-a-u-sql-database-in-azure-devops"></a>Implementación de una base de datos U-SQL en Azure DevOps

`PackageDeploymentTool.exe` proporciona interfaces de programación y línea de comandos que ayudan a implementar bases de datos de U-SQL. El SDK se incluye en el [paquete Nuget del SDK de U-SQL](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/), que se encuentra en `build/runtime/PackageDeploymentTool.exe`.

En Azure DevOps, puede usar una tarea de línea de comandos y este SDK para configurar una canalización de automatización para la actualización de la base de datos U-SQL. [Más información sobre el SDK y cómo configurar una canalización de CI/CD para la implementación de bases de datos de U-SQL](data-lake-analytics-cicd-overview.md#deploy-u-sql-database-through-azure-pipelines).

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de una canalización de CI/CD para Azure Data Lake Analytics](data-lake-analytics-cicd-overview.md).
* [Prueba del código de Azure Data Lake Analytics](data-lake-analytics-cicd-test.md)
* [Ejecución local de scripts U-SQL](data-lake-analytics-data-lake-tools-local-run.md)
