---
title: Ejecución de scripts U-SQL de Azure Data Lake en la máquina local
description: Aprenda a usar las Herramientas de Azure Data Lake para Visual Studio para ejecutar trabajos de U-SQL en la máquina local.
services: data-lake-analytics
author: liudan66
ms.author: liud
ms.reviewer: jasonh
ms.assetid: 66dd58b1-0b28-46d1-aaae-43ee2739ae0a
ms.service: data-lake-analytics
ms.topic: how-to
ms.workload: big-data
ms.date: 07/03/2018
ms.openlocfilehash: 24f1156fa4a97adb500033034bc7396fd1badbeb
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87125741"
---
# <a name="run-u-sql-scripts-on-your-local-machine"></a>Ejecución de scripts U-SQL en la máquina local

Al desarrollar scripts U-SQL, puede ahorrar tiempo y gastos si ejecuta localmente los scripts. Herramientas de Azure Data Lake para Visual Studio permite ejecutar scripts U-SQL en la máquina local. 

## <a name="basic-concepts-for-local-runs"></a>Conceptos básicos de las ejecuciones locales

El gráfico siguiente muestra los componentes para la ejecución local y cómo se corresponden con la ejecución en la nube.

|Componente|Ejecución local|Ejecución en la nube|
|---------|---------|---------|
|Storage|Carpeta raíz de datos local|Cuenta predeterminada de Azure Data Lake Store|
|Proceso|Motor de ejecución local de U-SQL|Servicio Azure Data Lake Analytics|
|Entorno de ejecución|Directorio de trabajo en la máquina local|Clúster de Azure Data Lake Analytics|

Las secciones siguientes proporcionan más información acerca de los componentes de ejecución locales.

### <a name="local-data-root-folders"></a>Carpetas raíz de datos locales

Una carpeta raíz de datos local es un **almacén local** de la cuenta de proceso local. Todas las carpetas del sistema de archivos local de la máquina local pueden ser una carpeta raíz de datos local. Es equivalente a la cuenta de Azure Data Lake Store predeterminada de una cuenta de Data Lake Analytics. Cambiar a una carpeta raíz de datos diferente es como cambiar a otra cuenta de almacén predeterminada. 

La carpeta raíz de datos se utiliza para:
- Almacenamiento de metadatos. Algunos ejemplos son las bases de datos, tablas, funciones con valores de tabla y ensamblados.
- Buscar las rutas de acceso de entrada y salida que se definen como rutas de acceso relativas en el script U-SQL. El uso de rutas de acceso relativas facilita la implementación de sus scripts U-SQL en Azure.

### <a name="u-sql-local-run-engines"></a>Motores de ejecución local de U-SQL

Un motor de ejecución local de U-SQL es una **cuenta de proceso local** para trabajos de U-SQL. Los usuarios pueden ejecutar trabajos de U-SQL localmente mediante las Herramientas de Azure Data Lake para Visual Studio. También se admite las ejecuciones locales mediante línea de comandos e interfaces de programación del SDK de U-SQL de Azure Data Lake. Más información acerca del [SDK de U-SQL para Azure Data Lake](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/).

### <a name="working-directories"></a>Directorios de trabajo

Al ejecutar un script U-SQL, se necesita una carpeta del directorio de trabajo para almacenar en caché los resultados de compilación, los registros de ejecución y realizar otras funciones. En Herramientas de Azure Data Lake para Visual Studio, el directorio de trabajo es el del proyecto de U-SQL. Se encuentra en `<U-SQL project root path>/bin/debug>`. El directorio de trabajo se limpia cada vez que se desencadena una nueva ejecución.

## <a name="local-runs-in-microsoft-visual-studio"></a>Ejecuciones locales en Microsoft Visual Studio

Herramientas de Azure Data Lake para Visual Studio tiene un motor de ejecución local integrado. Las herramientas usan el motor como una cuenta de proceso local. Para ejecutar un script U-SQL localmente, seleccione la cuenta **Local-machine** (máquina local) o **Local-project** (proyecto local) en el menú desplegable del margen del editor del script. Después, seleccione **Enviar**.

![Envío de un script U-SQL a una cuenta local](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-submit-script-to-local-account.png) 
 
## <a name="local-runs-with-a-local-machine-account"></a>Ejecuciones locales con una cuenta Local-machine

Una cuenta **Local-machine** es una cuenta de proceso local compartida con una sola carpeta raíz de datos local como cuenta de almacén local. De forma predeterminada, la carpeta raíz de datos se encuentra en **C:\Users\<username>\AppData\Local\USQLDataRoot**. También es configurable mediante **Tools** > **Data Lake** > **Options and Settings** (Herramientas > Data Lake > Opciones y configuración).

![Configuración de una carpeta raíz de datos local](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-configure-local-data-root.png)
  
Para una ejecución local se necesita un proyecto de U-SQL. El directorio de trabajo del proyecto de U-SQL se usa para el directorio de trabajo de ejecución local de U-SQL. Los resultados de la compilación, los registros de ejecución y otros archivos relacionados con la ejecución del trabajo se generan y almacenan en la carpeta del directorio de trabajo durante la ejecución local. Cada vez que vuelve a ejecutar el script, todos los archivos del directorio de trabajo se limpian y vuelven a generar.

## <a name="local-runs-with-a-local-project-account"></a>Ejecuciones locales con una cuenta Local-project

Una cuenta **Local-project** es una cuenta de proceso local aislada del proyecto para cada proyecto con carpeta raíz de datos local aislada. Cada proyecto de U-SQL activo que se abre en el Explorador de soluciones en Visual Studio tiene una cuenta `(Local-project: <project name>)` correspondiente. Las cuentas se enumeran tanto en el Explorador de servidores en Visual Studio como en el margen del editor de scripts de U-SQL.  

La cuenta **Local-project** proporciona un entorno de desarrollo limpio y aislado. Una cuenta **Local-machine** tiene una carpeta raíz de datos local compartida que almacena los metadatos y los datos de entrada y de salida de todos los trabajos locales. Pero una cuenta **Local-project** crea una carpeta raíz de datos local temporal en un directorio de trabajo del proyecto de U-SQL cada vez que se ejecuta un script U-SQL. Esta carpeta raíz de datos temporal se limpia con cada nueva compilación o ejecución. 

Un proyecto de U-SQL administra el entorno de ejecución local aislado mediante una propiedad y una referencia de proyecto. Puede configurar tanto los orígenes de datos de entrada para los scripts U-SQL del proyecto como los entornos de bases de datos a las que se hace referencia.

### <a name="manage-the-input-data-source-for-a-local-project-account"></a>Administración del origen de datos de entrada para una cuenta Local-project 

Un proyecto de U-SQL crea una carpeta raíz de datos local y configura los datos para una cuenta **Local-project**. Una carpeta raíz de datos temporal se limpia y vuelve a crear en el directorio de trabajo del proyecto de U-SQL con cada nueva compilación y ejecución local. Todos los orígenes de datos configurados por el proyecto de U-SQL se copian en esta carpeta raíz de datos local temporal antes de la ejecución de trabajos locales. 

Puede configurar la carpeta raíz de los orígenes de datos. Haga clic con el botón derecho en **Project U-SQL** (Proyecto de U-SQL) > **Propiedad** > **Probar origen de datos**. Cuando se ejecuta el script U-SQL en una cuenta **Local-project**, todos los archivos y subcarpetas de la carpeta **Probar origen de datos** se copian en la carpeta raíz de datos local temporal. Los archivos de las subcarpetas se incluyen. Tras la ejecución de un trabajo local, los resultados de salida también pueden encontrarse en la carpeta raíz de datos local temporal del directorio de trabajo del proyecto. Toda esta salida se eliminará y limpiará cuando el proyecto se recompile y limpie. 

![Configuración del origen de datos de prueba de un proyecto](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-configure-project-test-data-source.png)

### <a name="manage-a-referenced-database-environment-for-a-local-project-account"></a>Administración de un entorno de bases de datos referenciadas para una cuenta **Local-project** 

Si una consulta U-SQL utiliza o consulta objetos de base de datos U-SQL, los entornos de bases de datos deben estar preparados localmente antes de ejecutar este script U-SQL localmente. Para una cuenta **Local-project**, las dependencias de la base de datos U-SQL pueden administrarse mediante las referencias de proyecto de U-SQL. Puede agregar referencias de proyecto de base de datos de U-SQL al proyecto de U-SQL. Antes de ejecutar scripts U-SQL en una cuenta **Local-project**, todas las bases de datos referenciadas se implementan en la carpeta raíz de datos local temporal. Y, con cada ejecución, se limpia la carpeta raíz de datos temporal como un entorno aislado nuevo.

Consulte este artículo relacionado:
* Aprenda a administrar las definiciones de base de datos U-SQL y las referencias en [proyectos de base de datos U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md).

## <a name="the-difference-between-local-machine-and-local-project-accounts"></a>La diferencia entre las cuentas **Local-machine** y **Local-project**

Una cuenta **Local-machine** simula una cuenta de Azure Data Lake Analytics en la máquina local de los usuarios. Comparte la misma experiencia que una cuenta de Azure Data Lake Analytics. Una cuenta **Local-project** proporciona un entorno de desarrollo local limpio y aislado, fácil de usar. Este entorno ayuda a los usuarios a implementar referencias de base de datos y datos de entrada antes de ejecutar scripts localmente. Una cuenta **Local-machine** proporciona un entorno compartido permanente, accesible en todos los proyectos. Una cuenta **Local-project** proporciona un entorno de desarrollo aislado para cada proyecto. Se actualiza para cada ejecución. Una cuenta **Local-project** ofrece una experiencia de desarrollo más rápida al aplicar los nuevos cambios rápidamente.

En la tabla siguiente se muestran más diferencias entre las cuentas **Local-machine** y **Local-project**:

|Área de diferencia|Local-machine|Local-project|
|----------------|---------------|---------------|
|Acceso local|Accesible para todos los proyectos.|Solo el proyecto correspondiente puede acceder a esta cuenta.|
|Carpeta raíz de datos local|Carpeta local permanente. Configurada mediante **Herramientas** > **Data Lake** >  **Options and Settings** (Opciones y configuración).|Carpeta temporal creada para cada ejecución local en el directorio de trabajo del proyecto de U-SQL. La carpeta se limpia con cada nueva compilación o ejecución.|
|Datos de entrada para un script U-SQL|Ruta de acceso relativa en la carpeta raíz de datos local permanente.|Definida mediante la **Propiedad del proyecto de U-SQL** > **Probar origen de datos**. Se copian todos los archivos y subcarpetas en la carpeta raíz de datos temporal antes de la ejecución local.|
|Datos de salida para un script U-SQL|Ruta de acceso relativa en la carpeta raíz de datos local permanente.|Salida a la carpeta raíz de datos temporal. Los resultados se limpian con cada nueva compilación o ejecución.|
|Implementación de bases de datos referenciadas|Las bases de datos referenciadas no se implementan automáticamente al ejecutarse en una cuenta **Local-machine**. Lo mismo ocurre al enviarse a una cuenta de Azure Data Lake Analytics.|Las bases de datos referenciadas se implementan en la cuenta **Local-project** automáticamente antes de la ejecución local. Todos los entornos de base de datos se limpian y se vuelven a implementar con cada nueva compilación o ejecución.|

## <a name="a-local-run-with-the-u-sql-sdk"></a>Ejecución local con el SDK de U-SQL

Puede ejecutar scripts U-SQL localmente en Visual Studio y también puede usar el SDK de U-SQL para Azure Data Lake para ejecutar scripts U-SQL localmente con la línea de comandos e interfaces de programación. Mediante estas interfaces, puede automatizar las ejecuciones y pruebas locales de U-SQL.

Más información acerca del [SDK de U-SQL para Azure Data Lake](data-lake-analytics-u-sql-sdk.md).

## <a name="next-steps"></a>Pasos siguientes

- [Procedimiento para configurar una canalización de CI/CD para Azure Data Lake Analytics](data-lake-analytics-cicd-overview.md).
- [Procedimiento para probar el código de Azure Data Lake Analytics](data-lake-analytics-cicd-test.md).
