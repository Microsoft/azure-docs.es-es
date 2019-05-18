---
title: Análisis de datos en Azure Data Lake Storage Gen1 con Power BI | Microsoft Docs
description: Uso de Power BI para analizar datos almacenados en Azure Data Lake Storage Gen1
services: data-lake-store
documentationcenter: ''
author: twooley
manager: mtillman
editor: cgronlun
ms.assetid: 57d19d27-e135-49d9-a7ea-46c48ef4e3bd
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: d8717b8f365e692b5f27bf8a04d65c5147b8f31b
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65603199"
---
# <a name="analyze-data-in-azure-data-lake-storage-gen1-by-using-power-bi"></a>Análisis de datos en Azure Data Lake Storage Gen1 con Power BI
En este artículo aprenderá a usar Power BI Desktop para analizar y visualizar los datos almacenados en Azure Data Lake Storage Gen1.

## <a name="prerequisites"></a>Requisitos previos
Antes de empezar este tutorial, debe contar con lo siguiente:

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* **Cuenta de Data Lake Storage Gen1**. Siga las instrucciones de [Introducción a Azure Data Lake Storage Gen1 con Azure Portal](data-lake-store-get-started-portal.md). En este artículo se supone que ya creó una cuenta de Data Lake Storage Gen1 denominada **myadlsg1** y le cargó un archivo de datos de ejemplo (**Drivers.txt**). Este archivo de ejemplo está disponible para su descarga en [Azure Data Lake Git Repository](https://github.com/Azure/usql/tree/master/Examples/Samples/Data/AmbulanceData/Drivers.txt)(Repositorio Git de Azure Data Lake).
* **Power BI Desktop**. Puede descargarla del [Centro de descarga de Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=45331). 

## <a name="create-a-report-in-power-bi-desktop"></a>Creación de un informe en Power BI Desktop
1. Inicie Power BI Desktop en el equipo.
2. En la cinta **Inicio**, haga clic en **Obtener datos** y luego en Más. En el cuadro de diálogo **Obtener datos**, haga clic en **Azure**, en **Azure Data Lake Store** y luego en **Conectar**.
   
    ![Conectarse a Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account.png "Connect to Data Lake Storage Gen1")
3. Si ve un cuadro de diálogo que indica que el conector está en fase de desarrollo, opte por continuar.
4. En el cuadro de diálogo **Azure Data Lake Store**, proporcione la dirección URL a su cuenta de Data Lake Storage Gen1 y haga clic en **Aceptar**.
   
    ![URL para Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account-url.png "URL for Data Lake Storage Gen1")
5. En el siguiente cuadro de diálogo, haga clic en **Iniciar sesión** para iniciar sesión en la cuenta de Data Lake Storage Gen1. Se le redirigirá a la página de inicio de sesión de su organización. Siga las indicaciones para iniciar sesión en la cuenta.
   
    ![Iniciar sesión en Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account-signin.png "Sign into Data Lake Storage Gen1")
6. Cuando haya iniciado sesión correctamente, haga clic en **Conectar**.
   
    ![Conectarse a Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account-connect.png "Connect to Data Lake Storage Gen1")
7. En el siguiente cuadro de diálogo se muestra el archivo que cargó a su cuenta de Data Lake Storage Gen1. Compruebe la información y haga clic en **Cargar**.
   
    ![Cargar datos de Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account-load.png "Load data from Data Lake Storage Gen1")
8. Después de cargar correctamente los datos en Power BI, verá los siguientes campos en la pestaña **Campos** .
   
    ![Campos importados](./media/data-lake-store-power-bi/imported-fields.png "Campos importados")
   
    Pero, para visualizar y analizar los datos, es preferible que estos estén disponibles por los campos siguientes.
   
    ![Campos deseados](./media/data-lake-store-power-bi/desired-fields.png "Campos deseados")
   
    En los pasos siguientes, se actualizará la consulta para convertir los datos importados al formato deseado.
9. En la cinta **Inicio**, haga clic en **Editar consultas**.
   
    ![Editar consultas](./media/data-lake-store-power-bi/edit-queries.png "Editar consultas")
10. En el Editor de consultas, en la columna **Contenido**, haga clic en **Binario**.
    
    ![Editar consultas](./media/data-lake-store-power-bi/convert-query1.png "Editar consultas")
11. Verá un icono de archivo, que representa el archivo **Drivers.txt** que cargó. Haga clic en el archivo y luego en **CSV**.    
    
    ![Editar consultas](./media/data-lake-store-power-bi/convert-query2.png "Editar consultas")
12. Debería ver una salida como la siguiente. Los datos están ahora disponibles en un formato que puede usar para crear visualizaciones.
    
    ![Editar consultas](./media/data-lake-store-power-bi/convert-query3.png "Editar consultas")
13. En la cinta **Inicio**, haga clic en **Cerrar y aplicar** y luego en **Cerrar y aplicar**.
    
    ![Editar consultas](./media/data-lake-store-power-bi/load-edited-query.png "Editar consultas")
14. Después de actualizar la consulta, la pestaña **Campos** mostrará los nuevos campos disponibles para su visualización.
    
    ![Campos actualizados](./media/data-lake-store-power-bi/updated-query-fields.png "Campos actualizados")
15. Vamos a crear un gráfico circular para representar los conductores de cada ciudad de un determinado país o región. Para ello, realice las selecciones siguientes.
    
    1. En la pestaña Visualizaciones, haga clic en el símbolo de un gráfico circular.
       
        ![Crear gráfico circular](./media/data-lake-store-power-bi/create-pie-chart.png "Crear gráfico circular")
    2. Las columnas que vamos a utilizar son **columna 4** (nombre de la ciudad) y **columna 7** (nombre del país o región). Arrastre estas columnas de la pestaña **Campos** a la pestaña **Visualizaciones**, tal y como se muestra a continuación.
       
        ![Creación de visualizaciones](./media/data-lake-store-power-bi/create-visualizations.png "Creación de visualizaciones")
    3. El gráfico circular ahora debe ser similar al que se muestra a continuación.
       
        ![Gráfico circular](./media/data-lake-store-power-bi/pie-chart.png "Crear visualizaciones")
16. Si selecciona un país o región específico de los filtros de nivel de página, ahora puede ver el número de controladores en cada ciudad del país o región seleccionado. Por ejemplo, en la pestaña **Visualizaciones**, en **Filtros de nivel de página**, seleccione **Brasil**.
    
    ![Seleccione un país](./media/data-lake-store-power-bi/select-country.png "seleccione un país o región")
17. El gráfico circular se actualiza automáticamente para mostrar los conductores de las ciudades de Brasil.
    
    ![Los controladores en un país](./media/data-lake-store-power-bi/driver-per-country.png "controladores por país o región")
18. En el menú **Archivo**, haga clic en **Guardar** para guardar la visualización como archivo de Power BI Desktop.

## <a name="publish-report-to-power-bi-service"></a>Publicación del informe en el servicio Power BI
Después de crear las visualizaciones en Power BI Desktop, puede compartirlas con otros usuarios publicándolas en el servicio Power BI. Para obtener instrucciones sobre cómo hacerlo, consulte [Publicar desde Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-upload-desktop-files/).

## <a name="see-also"></a>Vea también
* [Análisis de datos en Data Lake Storage Gen1 con Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)

