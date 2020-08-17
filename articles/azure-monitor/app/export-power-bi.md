---
title: Exportación a Power BI desde Azure Application Insights | Microsoft Docs
description: Las consultas de Analytics se pueden mostrar en Power BI.
ms.topic: conceptual
ms.date: 08/10/2018
ms.openlocfilehash: 51a553452643fc979846ab4604762f11c5b7ff68
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324376"
---
# <a name="feed-power-bi-from-application-insights"></a>Alimentación de Power BI desde Application Insights
[Power BI](https://www.powerbi.com/) es un conjunto de herramientas de negocios que pueden ayudar a analizar datos y compartir conocimientos. Cada dispositivo cuenta con paneles que incluyen gran cantidad de datos. Puede combinar datos de varios orígenes, incluidas las consultas de Analytics en [Azure Application Insights](./app-insights-overview.md).

Hay tres métodos para exportar datos de Application Insights a Power BI:

* [**Exportación de consultas de Analytics**](#export-analytics-queries). Este es el método preferido. Escriba cualquier consulta que desee y expórtela a Power BI. Puede colocar esta consulta en un panel junto con otros datos.
* [**Exportación continua y Azure Stream Analytics**](./export-stream-analytics.md). Este método es útil si desea almacenar los datos durante largos períodos. Si no tiene un requisito de retención de datos extendido, utilice el método de consulta de análisis de exportación. Exportación continua y Stream Analytics implica más trabajo de configuración y una sobrecarga de almacenamiento adicional.
* **Adaptador de Power BI**. El conjunto de gráficos está predefinido, pero puede agregar sus propias consultas de cualquier otro origen.

> [!NOTE]
> El adaptador de Power BI está **en desuso**. Los gráficos predefinidos para esta solución se rellenan con consultas estáticas no modificables. No tiene la capacidad de editar estas consultas y, según ciertas propiedades de los datos, es posible que la conexión con Power BI se realice correctamente, pero no se rellene ningún dato. Esto se debe a los criterios de exclusión que se establecen en la consulta codificado de forma rígida. Aunque esta solución incluso puede funcionar para algunos clientes, debido a la falta de flexibilidad del adaptador, la solución recomendada es usar la funcionalidad de [**exportación de consultas de Analytics**](#export-analytics-queries).

## <a name="export-analytics-queries"></a>Exportación de consultas de Analytics
Esta ruta le permite escribir las consultas de Analytics que quiera o exportarlas desde los embudos de uso y luego exportarlas a un panel de Power BI. (Puede agregar al panel creado por el adaptador).

### <a name="one-time-install-power-bi-desktop"></a>Una vez: instalar Power BI Desktop
Para importar la consulta de Application Insights, utilice la versión de escritorio de Power BI. Después, puede publicarla en la web o en el área de trabajo de nube de Power BI. 

Instale [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/).

### <a name="export-an-analytics-query"></a>Exportación de una consulta de Analytics
1. [Abra Analytics y escriba la consulta](../log-query/get-started-portal.md).
2. Pruebe y refine la consulta hasta que esté satisfecho con los resultados. Asegúrese de que la consulta se ejecuta correctamente en Analytics antes de exportarla.
3. En el menú **Exportar**, elija **Power BI (M)** . Guarde el archivo de texto.
   
    ![Captura de pantalla de Analytics, con el menú Exportar resaltado](./media/export-power-bi/analytics-export-power-bi.png)
4. En Power BI Desktop, seleccione **Obtener datos** > **Consulta en blanco**. A continuación, en el editor de consultas, en **Ver**, seleccione **Editor avanzado**.

    Pegue el script del lenguaje M exportado en el Editor avanzado.

    ![Capturas de pantalla de Power BI Desktop, con el Editor avanzado resaltado](./media/export-power-bi/power-bi-import-analytics-query.png)

5. Para que Power BI acceda a Azure, es posible que deba proporcionar credenciales. Utilice la **cuenta de organización** para iniciar sesión con su cuenta Microsoft.
   
    ![Captura de pantalla del cuadro de diálogo Configuración de la consulta de Power BI](./media/export-power-bi/power-bi-import-sign-in.png)

    Si tiene que comprobar las credenciales, use el comando del menú **Configuración de origen de datos** en el Editor de consultas. Asegúrese de especificar las credenciales que use para Azure, ya que pueden ser diferentes de las credenciales de Power BI.
6. Elija una visualización para la consulta y seleccione los campos para el eje x, el eje y y la dimensión de segmentación.
   
    ![Captura de pantalla de opciones de visualización de Power BI Desktop](./media/export-power-bi/power-bi-analytics-visualize.png)
7. Publique el informe en el área de trabajo de nube de Power BI. Desde allí, puede incluir una versión sincronizada en otras páginas web.
   
    ![Captura de pantalla de Power BI Desktop, con el botón Publicar resaltado](./media/export-power-bi/publish-power-bi.png)
8. Actualice el informe manualmente a intervalos o configure una actualización programada en la página Opciones.

### <a name="export-a-funnel"></a>Exportar un embudo
1. [Cree un embudo](./usage-funnels.md).
2. Seleccione **Power BI**.

   ![Captura de pantalla del botón de Power BI](./media/export-power-bi/button.png)

3. En Power BI Desktop, seleccione **Obtener datos** > **Consulta en blanco**. A continuación, en el editor de consultas, en **Ver**, seleccione **Editor avanzado**.

   ![Captura de pantalla de Power BI Desktop, con el botón Consulta en blanco resaltado](./media/export-power-bi/blankquery.png)

   Pegue el script del lenguaje M exportado en el Editor avanzado. 

   ![Capturas de pantalla de Power BI Desktop, con el Editor avanzado resaltado](./media/export-power-bi/advancedquery.png)

4. Seleccione elementos de la consulta y elija una visualización de embudo.

   ![Captura de pantalla de opciones de visualización de Power BI Desktop](./media/export-power-bi/selectsequence.png)

5. Cambie el título para que sea descriptivo y publique el informe en el área de trabajo en la nube de Power BI. 

   ![Captura de pantalla de Power BI Desktop, con el cambio de título resaltado](./media/export-power-bi/changetitle.png)

## <a name="troubleshooting"></a>Solución de problemas

Pueden producirse errores relacionados con las credenciales o el tamaño del conjunto de datos. Aquí se ofrece cierta información sobre qué hacer cuando se producen estos errores.

### <a name="unauthorized-401-or-403"></a>No autorizado (401 o 403)
Esto puede ocurrir si no se ha actualizado el token de actualización. Pruebe estos pasos para asegurarse de que todavía tiene acceso:

1. Inicie sesión en Azure Portal y asegúrese de que puede acceder al recurso.
2. Intente actualizar las credenciales para el panel.
3. Intente borrar la caché del escritorio de PowerBI.


   Si tiene acceso, pero la actualización de las credenciales no funciona, abra una incidencia de soporte técnico.

### <a name="bad-gateway-502"></a>Puerta de enlace incorrecta (502)
Esto se debe normalmente a una consulta de Analytics que devuelve demasiados datos. Intente utilizar un intervalo de tiempo más reducido para la consulta. 

Si la reducción del conjunto de datos que procede de la consulta de Analytics no se ajusta a sus requisitos, plantéese el uso de la [API](https://dev.applicationinsights.io/documentation/overview) para extraer un conjunto de datos más grande. Aquí se muestra cómo convertir la exportación de M-Query para usar la API.

1. Cree una [clave de API](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID).
2. Actualice el script M de Power BI que exportó desde Analytics reemplazando la dirección URL de Azure Resource Manager por la API de Application Insights.
   * Reemplace **https:\//management.azure.com/subscriptions/...**
   * con **https:\//api.applicationinsights.io/beta/apps/...**
3. Por último, actualice las credenciales al modo básico y use su clave de API.

**Script existente**
 ```
 Source = Json.Document(Web.Contents("https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups//providers/microsoft.insights/components//api/query?api-version=2014-12-01-preview",[Query=[#"csl"="requests",#"x-ms-app"="AAPBI"],Timeout=#duration(0,0,4,0)]))
 ```
**Script actualizado**
 ```
 Source = Json.Document(Web.Contents("https://api.applicationinsights.io/beta/apps/<APPLICATION_ID>/query?api-version=2014-12-01-preview",[Query=[#"csl"="requests",#"x-ms-app"="AAPBI"],Timeout=#duration(0,0,4,0)]))
 ```

## <a name="about-sampling"></a>Acerca del muestreo
Según la cantidad de datos que envíe la aplicación, es posible que desee utilizar la característica de muestreo adaptativo, que envía solamente un porcentaje de los datos de telemetría. Esto se aplica igualmente si ha configurado el muestreo manualmente en el SDK o en la recopilación. [Obtenga más información sobre el muestreo](./sampling.md).

## <a name="power-bi-adapter-deprecated"></a>Adaptador de Power BI (en desuso)
Este método crea un panel completo de telemetría. El conjunto de datos inicial está predefinido, pero puede agregar más datos a él.

### <a name="get-the-adapter"></a>Obtención del adaptador
1. Inicie sesión en [Power BI](https://app.powerbi.com/).
2. Abra **Obtener datos** ![Captura de pantalla del icono de obtener datos en la esquina inferior izquierda](./media/export-power-bi/001.png), **Servicios**.

    ![Capturas de pantalla de Obtener de un origen de datos de Application Insights](./media/export-power-bi/002.png)

3. Seleccione **Obtenerla ahora** en Application Insights.

   ![Capturas de pantalla de Obtener de un origen de datos de Application Insights](./media/export-power-bi/003.png)
4. Proporcione los detalles del recurso de Application Insights y luego **inicie sesión**.

    ![Captura de pantalla de Obtener de un origen de datos de Application Insights](./media/export-power-bi/005.png)

     Esta información puede encontrarse en el panel Introducción de Application Insights:

     ![Captura de pantalla de Obtener de un origen de datos de Application Insights](./media/export-power-bi/004.png)

5. Abra la aplicación recién creada de Power BI en Application Insights.

6. Espere un minuto o dos para que se importen los datos.

    ![Captura de pantalla del adaptador de Power BI](./media/export-power-bi/010.png)

Puede modificar el panel, combinando los gráficos de Application Insights con los de otras fuentes y con consultas de Analytics. Puede obtener más gráficos en la galería de visualización, y cada gráfico tiene a su vez parámetros que puede establecer.

Después de la importación inicial, el panel y los informes seguirán actualizándose a diario. Puede controlar la programación de la actualización en el conjunto de datos.

## <a name="next-steps"></a>Pasos siguientes
* [Power BI: más información](https://www.powerbi.com/learning/)
* [Tutorial de Analytics](../log-query/get-started-portal.md)

