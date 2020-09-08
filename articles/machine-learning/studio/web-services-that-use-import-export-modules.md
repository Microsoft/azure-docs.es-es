---
title: 'ML Studio (clásico): Importación o exportación de datos en servicios web: Azure'
description: Aprenda a usar los módulos de importación y exportación de datos para enviar y recibir datos de un servicio web.
services: machine-learning
author: likebupt
ms.author: keli19
editor: cgronlun
ms.assetid: 3a7ac351-ebd3-43a1-8c5d-18223903d08e
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
ms.date: 03/28/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 7b1d8f939e403c795effeca14943791d1554742f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89010739"
---
# <a name="deploy-azure-machine-learning-studio-classic-web-services-that-use-data-import-and-data-export-modules"></a>Implementación de servicios web de Azure Machine Learning Studio (clásico) que usan módulos de importación y exportación de datos

**SE APLICA A:**  ![yes](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


Cuando se crea un experimento predictivo, normalmente se agregan una entrada y una salida de servicio web. Al implementar el experimento, los consumidores pueden enviar y recibir datos desde el servicio web a través de las entradas y salidas. En algunas aplicaciones, los datos del consumidor pueden estar disponibles desde una fuente de datos o ya residir en un origen de datos externo, como el Almacenamiento de blobs de Azure. En estos casos, no se requiere que lean y escriban datos mediante entradas y salidas del servicio web. En su lugar, pueden utilizar el servicio de ejecución por lotes (BES) para leer los datos del origen de datos mediante un módulo de importación de datos y escribir los resultados de puntuación en una ubicación de datos diferente mediante un módulo de exportación de datos.

Los módulos de importación y exportación de datos pueden realizar operaciones de lectura y escritura en diversas ubicaciones de datos, como una dirección URL web a través de HTTP, una consulta de Hive, una base de datos en Azure SQL Database, almacenamiento de tablas de Azure, Azure Blob Storage, un proveedor de fuentes de distribución de datos o una base de datos de SQL Server.

En este tema se utiliza el "Ejemplo 5: entrenamiento, prueba y evaluación para la clasificación binaria: conjunto de datos sobre adultos" y se da por supuesto que el conjunto de datos se ha cargado en una tabla de Azure SQL denominada censusdata.

## <a name="create-the-training-experiment"></a>Creación del experimento de entrenamiento
Al abrir el "Ejemplo 5: entrenamiento, prueba y evaluación para la clasificación binaria: conjunto de datos sobre adultos" se usa el conjunto de datos de clasificación binaria de ingresos en el censo de adultos. Asimismo, el experimento en el lienzo tendrá una apariencia similar a la siguiente imagen:

![Configuración inicial del experimento.](./media/web-services-that-use-import-export-modules/initial-look-of-experiment.png)

Para leer los datos de la tabla de SQL de Azure:

1. Elimine el módulo del conjunto de datos.
2. En el cuadro de búsqueda de componentes, escriba import.
3. En la lista de resultados, agregue un módulo *Importar datos* al lienzo del experimento.
4. Conecte la salida del módulo *Import Data* (Importar datos) a la entrada del módulo *Clean Missing Data* (Limpiar datos que faltan).
5. En el panel de propiedades, seleccione **Azure SQL Database** en el menú desplegable **Origen de datos**.
6. En los campos **Nombre del servidor de base de datos**, **Nombre de base de datos**, **Nombre de usuario** y **Contraseña**, escriba la información apropiada para la base de datos.
7. En el campo Consulta de base de datos, escriba la siguiente consulta.

    ```tsql
     select [age],
        [workclass],
        [fnlwgt],
        [education],
        [education-num],
        [marital-status],
        [occupation],
        [relationship],
        [race],
        [sex],
        [capital-gain],
        [capital-loss],
        [hours-per-week],
        [native-country],
        [income]
     from dbo.censusdata;
    ```
8. En la parte inferior del lienzo del experimento, haga clic en **Ejecutar**.

## <a name="create-the-predictive-experiment"></a>Creación del experimento predictivo
A continuación, configure el experimento predictivo desde el que se implementa el servicio web.

1. En la parte inferior del lienzo del experimento, haga clic en **Set Up Web Service** (Configurar servicio web) y seleccione **Predictive Web Service [Recommended]** (Servicio web predictivo [recomendado]).
2. Quite los módulos *Web Service Input* (Entrada del servicio web) y *Web Service Output* (Salida del servicio web) del experimento predictivo.
3. En el cuadro de búsqueda de componentes, escriba export.
4. En la lista de resultados, agregue un módulo *Exportar datos* al lienzo del experimento.
5. Conecte la salida del módulo *Score Model* (Puntuar modelo) a la entrada del módulo *Export Data* (Exportar datos).
6. En el panel de propiedades, seleccione **Azure SQL Database** en el menú desplegable Origen de datos.
7. En los campos **Nombre del servidor de base de datos**, **Nombre de base de datos**, **Server user account name** (Nombre de la cuenta de usuario del servidor) y **Server user account password** (Contraseña de la cuenta de usuario del servidor) escriba la información apropiada para la base de datos.
8. En el campo **Comma separated list of columns to be saved** (Lista de elementos separados por comas de las columnas que guardar) escriba Scored Labels.
9. En el campo **Data table name**(Nombre de tabla de datos), escriba dbo.ScoredLabels. Si la tabla no existe, se crea cuando se ejecuta el experimento o se llama al servicio web.
10. En el campo **Comma separated list of datatable columns** (Lista de elementos separados por comas de las columnas de tablas de datos) escriba ScoredLabels.

Cuando se escribe una aplicación que llama al servicio web final, puede especificar una consulta de entrada diferente u otra tabla de destino en tiempo de ejecución. Para configurar estas entradas y salidas, utilice la característica de parámetros del servicio web para establecer la propiedad *Origen de datos* del módulo *Importar datos* y la propiedad de destino de datos del módulo *Exportar datos*.  Para obtener más información sobre los parámetros del servicio web, consulte la entrada [Azure Machine Learning Studio Web Service Parameters](https://blogs.technet.microsoft.com/machinelearning/2014/11/25/azureml-web-service-parameters/) (Parámetros del servicio web de Azure Machine Learning Studio) en el blog de Cortana Intelligence y Machine Learning.

Para configurar los parámetros del servicio web para la consulta de importación y la tabla de destino:

1. En el panel de propiedades del módulo *Import Data* (Importar datos), haga clic en el icono situado en la parte superior derecha del campo **Consulta de base de datos** y seleccione **Set as web service parameter** (Establecer como parámetro de servicio web).
2. En el panel de propiedades del módulo *Export Data* (Exportar datos), haga clic en el icono situado en la parte superior derecha del campo **Nombre de la tabla de datos** y seleccione **Set as web service parameter** (Establecer como parámetro de servicio web).
3. En la parte inferior del panel de propiedades del módulo *Exportar datos* , en la sección **Parámetros del servicio web** , haga clic en Consulta de base de datos y cambie el nombre por Query.
4. Haga clic en **Nombre de la tabla de datos** y cambie el nombre por **Table**.

Cuando haya terminado, el experimento debería tener un aspecto similar a la siguiente imagen:

![Aspecto final del experimento.](./media/web-services-that-use-import-export-modules/experiment-with-import-data-added.png)

Ahora puede implementar el experimento predictivo como servicio web.

## <a name="deploy-the-web-service"></a>Implementación del servicio web
Puede realizar la implementación en un servicio web clásico o nuevo.

### <a name="deploy-a-classic-web-service"></a>Implementación de un servicio web clásico
Para realizar la implementación como un servicio web clásico y crear una aplicación para usarla:

1. En la parte inferior del lienzo del experimento, haga clic en Ejecutar.
2. Cuando la ejecución haya terminado, haga clic en **Deploy Web Service** (Implementar servicio web) y seleccione **Deploy Web Service [Classic]** (Implementar servicio web [clásico]).
3. En el panel del servicio web, busque la clave de API. Copie y guárdela para usarla más adelante.
4. En la tabla **Default Endpoint** (Punto de conexión predeterminado), haga clic en el vínculo **Ejecución de lotes** para abrir la página de Ayuda de API.
5. En Visual Studio, cree una aplicación de consola en C#: **Nuevo** > **Proyecto** > **Visual C#**  > **Escritorio clásico de Windows** > **Aplicación de consola (.NET Framework)** .
6. En la página de Ayuda de API, busque la sección **Sample Code** (Ejemplo de código) en la parte inferior de la página.
7. Copie y pegue el ejemplo de código de C# en el archivo Program.cs y quite todas las referencias al Almacenamiento de blobs.
8. Actualice el valor de la variable *apiKey* con la clave de API guardada anteriormente.
9. Busque la declaración de solicitud y actualice los valores de los parámetros del servicio web que se pasan a los módulos *Import Data* (Importar datos) y *Export Data* (Exportar datos). En este caso, utilice la consulta original, pero defina un nuevo nombre de tabla.

    ```csharp
    var request = new BatchExecutionRequest()
    {
        GlobalParameters = new Dictionary<string, string>() {
            { "Query", @"select [age], [workclass], [fnlwgt], [education], [education-num], [marital-status], [occupation], [relationship], [race], [sex], [capital-gain], [capital-loss], [hours-per-week], [native-country], [income] from dbo.censusdata" },
            { "Table", "dbo.ScoredTable2" },
        }
    };
    ```
10. Ejecute la aplicación.

Al término de la ejecución, se agrega una nueva tabla a la base de datos que contiene los resultados de puntuación.

### <a name="deploy-a-new-web-service"></a>Implementación de servicios web nuevos

> [!NOTE]
> Para implementar un nuevo servicio web, debe tener permisos suficientes en la suscripción en la que lo implementa. Para obtener más información, consulte [Administración de un servicio web mediante el portal Servicios web Azure Machine Learning](manage-new-webservice.md).

Para realizar la implementación como un servicio web nuevo y crear una aplicación para usarla:

1. En la parte inferior del lienzo del experimento, haga clic en **Ejecutar**.
2. Cuando la ejecución haya terminado, haga clic en **Deploy Web Service** (Implementar servicio web) y seleccione **Deploy Web Service [New]** (Implementar servicio web [nuevo]).
3. En la página Deploy Experiment (Implementar experimento), escriba un nombre para el servicio web, seleccione un plan de precios y haga clic en **Implementar**.
4. En la página **Inicio rápido**, haga clic en **Consume** (Consumir).
5. En la sección **Sample Code** (Ejemplo de código), haga clic en **Batch**.
6. En Visual Studio, cree una aplicación de consola en C#: **Nuevo** > **Proyecto** > **Visual C#**  > **Escritorio clásico de Windows** > **Aplicación de consola (.NET Framework)** .
7. Copie y pegue el ejemplo de código de C# en el archivo Program.cs.
8. Actualice el valor de la variable *apiKey* con la **clave principal** ubicada en la sección **Basic consumption info** (Información básica de consumo).
9. Busque la declaración *scoreRequest* y actualice los valores de los parámetros del servicio web que se pasan a los módulos *Import Data* (Importar datos) y *Export Data* (Exportar datos). En este caso, utilice la consulta original, pero defina un nuevo nombre de tabla.

    ```csharp
    var scoreRequest = new
    {
        Inputs = new Dictionary<string, StringTable>()
        {
        },
        GlobalParameters = new Dictionary<string, string>() {
            { "Query", @"select [age], [workclass], [fnlwgt], [education], [education-num], [marital-status], [occupation], [relationship], [race], [sex], [capital-gain], [capital-loss], [hours-per-week], [native-country], [income] from dbo.censusdata" },
            { "Table", "dbo.ScoredTable3" },
        }
    };
    ```
10. Ejecute la aplicación.

