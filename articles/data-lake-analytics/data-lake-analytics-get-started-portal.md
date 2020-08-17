---
title: 'Creación y consulta de Azure Data Lake Analytics: Azure Portal'
description: Use Azure Portal para crear una cuenta de Azure Data Lake Analytics y enviar un trabajo de U-SQL.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: b1584d16-e0d2-4019-ad1f-f04be8c5b430
ms.topic: conceptual
ms.date: 03/21/2017
ms.openlocfilehash: d8f0a5697a128deb75537f3d48a3830dc7ba9df9
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132541"
---
# <a name="get-started-with-azure-data-lake-analytics-using-the-azure-portal"></a>Introducción a Azure Data Lake Analytics mediante Azure Portal
[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

En este artículo se describe cómo usar Azure Portal para crear cuentas de Azure Data Lake Analytics, definir trabajos en [U-SQL](data-lake-analytics-u-sql-get-started.md) y enviar trabajos al servicio Data Lake Analytics.

## <a name="prerequisites"></a>Prerrequisitos

Para comenzar este tutorial, es preciso tener una **suscripción a Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

## <a name="create-a-data-lake-analytics-account"></a>Creación de una cuenta de Análisis de Data Lake

Ahora, creará una instancia de Data Lake Analytics y una cuenta de Azure Data Lake Storage Gen1 al mismo tiempo.  Este paso es muy simple y en 60 segundos se ha realizado.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Haga clic en **Crear un recurso** >  **Datos y análisis** > **Data Lake Analytics**.
3. Seleccione los valores de los siguientes elementos:
   * **Name**: nombre de la cuenta de Data Lake Analytics (solo se permiten letras minúsculas y números).
   * **Suscripción**: seleccione la suscripción de Azure usada para la cuenta de Analytics.
   * **Grupo de recursos**. seleccione un grupo de recursos de Azure existente o cree uno nuevo.
   * **Ubicación**. seleccione un centro de datos de Azure para la cuenta de Análisis de Data Lake.
   * **Data Lake Storage Gen1**: siga las instrucciones para crear una nueva cuenta de Data Lake Storage Gen1 o seleccione una existente. 
4. Si lo desea, seleccione un plan de tarifa para la cuenta de Data Lake Analytics.
5. Haga clic en **Crear**. 


## <a name="your-first-u-sql-script"></a>El primer script U-SQL

El siguiente texto es un script U-SQL muy simple. Simplemente se encarga de definir un pequeño conjunto de datos en el script y, a continuación, escribir ese conjunto de datos en la cuenta de Data Lake Storage Gen1 predeterminada como un archivo denominado `/data.csv`.

```usql
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
```

## <a name="submit-a-u-sql-job"></a>Envío de un trabajo de U-SQL

1. Desde la cuenta de Data Lake Analytics, seleccione **Nuevo trabajo**.
2. Pegue el texto del script U-SQL anterior. Asigne un nombre al trabajo. 
3. Seleccione el botón **Enviar** para iniciar el trabajo.   
4. Supervise el **estado** del trabajo y espere a que este cambie a **Correcto**.
5. Seleccione la pestaña **Datos** y, después, seleccione la pestaña **Salidas**. Seleccione el archivo de salida denominado `data.csv` y vea los datos de salida.

## <a name="see-also"></a>Consulte también

* Para empezar a desarrollar aplicaciones con U-SQL, consulte [Desarrollo de scripts U-SQL mediante Data Lake Tools for Visual Studio](data-lake-analytics-data-lake-tools-get-started.md).
* Para obtener más información sobre U-SQL, consulte [Introducción al lenguaje U-SQL de Análisis de Azure Data Lake](data-lake-analytics-u-sql-get-started.md).
* Para conocer las tareas de administración, consulte [Administración de Azure Data Lake Analytics mediante el Azure Portal](data-lake-analytics-manage-use-portal.md).
