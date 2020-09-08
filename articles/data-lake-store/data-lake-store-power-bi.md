---
title: 'Análisis de datos en Azure Data Lake Storage Gen1: Power BI'
description: Obtenga información sobre cómo usar Power BI Desktop para analizar y visualizar los datos almacenados en Azure Data Lake Storage Gen1.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 282ac6f87d5ecff75e77b075ee1092f79b4923ad
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192860"
---
# <a name="analyze-data-in-azure-data-lake-storage-gen1-by-using-power-bi"></a>Análisis de datos en Azure Data Lake Storage Gen1 con Power BI
En este artículo se explica cómo usar Power BI Desktop para analizar y visualizar los datos almacenados en Azure Data Lake Storage Gen1.

## <a name="prerequisites"></a>Prerrequisitos
Antes de empezar este tutorial, debe contar con lo siguiente:

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* **Cuenta de Data Lake Storage Gen1**. Siga las instrucciones de [Introducción a Azure Data Lake Storage Gen1 con Azure Portal](data-lake-store-get-started-portal.md). En este artículo se supone que ya creó una cuenta de Data Lake Storage Gen1 denominada **myadlsg1** y le cargó un archivo de datos de ejemplo (**Drivers.txt**). Este archivo de ejemplo está disponible para su descarga en [Azure Data Lake Git Repository](https://github.com/Azure/usql/tree/master/Examples/Samples/Data/AmbulanceData/Drivers.txt)(Repositorio Git de Azure Data Lake).
* **Power BI Desktop**. Puede descargarla del [Centro de descarga de Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=45331). 

## <a name="create-a-report-in-power-bi-desktop"></a>Creación de un informe en Power BI Desktop
1. Inicie Power BI Desktop en el equipo.
2. En la cinta **Inicio**, haga clic en **Obtener datos** y luego en Más. En el cuadro de diálogo **Obtener datos**, haga clic en **Azure**, en **Azure Data Lake Store** y luego en **Conectar**.
   
    ![Conexión a Data Lake Storage Gen 1](./media/data-lake-store-power-bi/get-data-lake-store-account.png "Conexión a Data Lake Storage Gen 1")
3. Si ve un cuadro de diálogo que indica que el conector está en fase de desarrollo, opte por continuar.
4. En el cuadro de diálogo **Azure Data Lake Store**, proporcione la dirección URL a su cuenta de Data Lake Storage Gen1 y haga clic en **Aceptar**.
   
    ![Dirección URL para Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account-url.png "Dirección URL para Data Lake Storage Gen1")
5. En el siguiente cuadro de diálogo, haga clic en **Iniciar sesión** para iniciar sesión en la cuenta de Data Lake Storage Gen1. Se le redirigirá a la página de inicio de sesión de su organización. Siga las indicaciones para iniciar sesión en la cuenta.
   
    ![Inicio de sesión en Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account-signin.png "Inicio de sesión en Data Lake Storage Gen1")
6. Cuando haya iniciado sesión correctamente, haga clic en **Conectar**.
   
    ![Conexión a Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account-connect.png "Conexión a Data Lake Storage Gen 1")
7. En el siguiente cuadro de diálogo se muestra el archivo que cargó a su cuenta de Data Lake Storage Gen1. Compruebe la información y haga clic en **Cargar**.
   
    ![Carga de datos de Data Lake Storage Gen1](./media/data-lake-store-power-bi/get-data-lake-store-account-load.png "Carga de datos de Data Lake Storage Gen1")
8. Después de cargar correctamente los datos en Power BI, verá los siguientes campos en la pestaña **Campos** .
   
    ![Campos importados](./media/data-lake-store-power-bi/imported-fields.png "Campos importados")
   
    Pero, para visualizar y analizar los datos, es preferible que estos estén disponibles por los campos siguientes.
   
    ![Campos deseados](./media/data-lake-store-power-bi/desired-fields.png "Campos deseados")
   
    En los pasos siguientes, se actualizará la consulta para convertir los datos importados al formato deseado.
9. En la cinta **Inicio**, haga clic en **Editar consultas**.
   
    ![Edición de consultas](./media/data-lake-store-power-bi/edit-queries.png "Editar consultas")
10. En el Editor de consultas, en la columna **Contenido**, haga clic en **Binario**.
    
    ![Edición de consultas](./media/data-lake-store-power-bi/convert-query1.png "Editar consultas")
11. Verá un icono de archivo, que representa el archivo **Drivers.txt** que cargó. Haga clic en el archivo y luego en **CSV**.    
    
    ![Edición de consultas](./media/data-lake-store-power-bi/convert-query2.png "Editar consultas")
12. Debería ver una salida como la siguiente. Los datos están ahora disponibles en un formato que puede usar para crear visualizaciones.
    
    ![Edición de consultas](./media/data-lake-store-power-bi/convert-query3.png "Editar consultas")
13. En la cinta **Inicio**, haga clic en **Cerrar y aplicar** y luego en **Cerrar y aplicar**.
    
    ![Edición de consultas](./media/data-lake-store-power-bi/load-edited-query.png "Editar consultas")
14. Después de actualizar la consulta, la pestaña **Campos** mostrará los nuevos campos disponibles para su visualización.
    
    ![Campos actualizados](./media/data-lake-store-power-bi/updated-query-fields.png "Campos actualizados")
15. Vamos a crear un gráfico circular para representar los conductores de cada ciudad de un país o región determinados. Para ello, realice las selecciones siguientes.
    
    1. En la pestaña Visualizaciones, haga clic en el símbolo de un gráfico circular.
       
        ![Creación de un gráfico circular](./media/data-lake-store-power-bi/create-pie-chart.png "Creación de un gráfico circular")
    2. Las columnas que se van a usar son **Columna 4** (nombre de la ciudad) y **Columna 7** (nombre del país o región). Arrastre estas columnas de la pestaña **Campos** a la pestaña **Visualizaciones**, tal y como se muestra a continuación.
       
        ![Creación de visualizaciones](./media/data-lake-store-power-bi/create-visualizations.png "Crear visualizaciones")
    3. El gráfico circular ahora debe ser similar al que se muestra a continuación.
       
        ![Gráfico circular](./media/data-lake-store-power-bi/pie-chart.png "Crear visualizaciones")
16. Si selecciona un país o región específicos en los filtros de nivel de página, ahora puede ver el número de conductores de cada ciudad del país o región seleccionados. Por ejemplo, en la pestaña **Visualizaciones**, en **Filtros de nivel de página**, seleccione **Brasil**.
    
    ![Selección de un país o región](./media/data-lake-store-power-bi/select-country.png "Selección de un país o región")
17. El gráfico circular se actualiza automáticamente para mostrar los conductores de las ciudades de Brasil.
    
    ![Conductores de un país o región](./media/data-lake-store-power-bi/driver-per-country.png "Controladores por país o región")
18. En el menú **Archivo**, haga clic en **Guardar** para guardar la visualización como archivo de Power BI Desktop.

## <a name="publish-report-to-power-bi-service"></a>Publicación del informe en el servicio Power BI
Después de crear las visualizaciones en Power BI Desktop, puede compartirlas con otros usuarios publicándolas en el servicio Power BI. Para obtener instrucciones sobre cómo hacerlo, consulte [Publicar desde Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-upload-desktop-files/).

## <a name="see-also"></a>Consulte también
* [Análisis de datos en Data Lake Storage Gen1 con Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)

