---
title: Supervisión de aplicaciones móviles con Azure Monitor Application Insights
description: Proporciona instrucciones para configurar rápidamente la supervisión de una aplicación móvil con Application Insights y App Center
ms.subservice: application-insights
ms.topic: quickstart
author: mrbullwinkle
ms.author: mbullwin
ms.date: 06/26/2019
ms.reviewer: daviste
ms.custom: mvc
ms.openlocfilehash: 0539dcecbbbfe28817122e0f47e813182a9abd9d
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322149"
---
# <a name="start-analyzing-your-mobile-app-with-app-center-and-application-insights"></a>Comience a analizar la aplicación móvil con App Center y Application Insights.

Esta guía de inicio rápido le orienta a través de la conexión de la instancia de App Center de la aplicación a Application Insights. Con Application Insights, puede consultar, segmentar, filtrar y analizar la telemetría con herramientas más eficaces que las disponibles en el servicio [Analytics](/mobile-center/analytics/) de App Center.

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido necesita instalar:

- Suscripción a Azure.
- Una aplicación iOS, Android, Xamarin, Universal Windows o React Native.
 
Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-up-with-app-center"></a>Registro en App Center
Para empezar, cree una cuenta y [regístrese en App Center](https://appcenter.ms/signup?utm_source=ApplicationInsights&utm_medium=Azure&utm_campaign=docs).

## <a name="onboard-to-app-center"></a>Incorporación a App Center

Antes de poder usar Application Insights con su aplicación móvil, debe integrar la aplicación en [App Center](/mobile-center/). Application Insights no recibe datos de telemetría de su aplicación móvil directamente. En su lugar, la aplicación envía telemetría de eventos personalizada a App Center. A continuación, App Center exporta continuamente copias de estos eventos personalizados a Application Insights a medida que se reciben los eventos. (Esto no es válido para el [SDK de Application Insights JS](https://github.com/Microsoft/ApplicationInsights-JS) ni para el [complemento React Native](https://github.com/Microsoft/ApplicationInsights-JS/tree/master/extensions/applicationinsights-react-native), donde la telemetría se envía directamente a Application Insights).

Para incorporar la aplicación, siga la guía de inicio rápido de App Center para cada plataforma que admita la aplicación. Cree instancias de App Center independientes para cada plataforma:

* [iOS](/mobile-center/sdk/getting-started/ios).
* [Android](/mobile-center/sdk/getting-started/android).
* [Xamarin](/mobile-center/sdk/getting-started/xamarin).
* [Universal Windows](/mobile-center/sdk/getting-started/uwp).
* [React Native](/mobile-center/sdk/getting-started/react-native).

## <a name="track-events-in-your-app"></a>Seguimiento de eventos en la aplicación

Una vez que la aplicación esté incorporada a App Center, se debe modificar para que envíe telemetría de eventos personalizados con el SDK de App Center. Los eventos personalizados son el único tipo de telemetría de App Center que se exporta a Application Insights.

Para enviar eventos personalizados desde aplicaciones iOS, utilice los métodos `trackEvent` o `trackEvent:withProperties` en el SDK de App Center. [Más información sobre el seguimiento de eventos de las aplicaciones iOS.](/mobile-center/sdk/analytics/ios)

```Swift
MSAnalytics.trackEvent("Video clicked")
```

Para enviar eventos personalizados desde aplicaciones Android, utilice el método `trackEvent` en el SDK de App Center. [Más información sobre el seguimiento de eventos de las aplicaciones Android.](/mobile-center/sdk/analytics/android)

```Java
Analytics.trackEvent("Video clicked")
```

Para enviar eventos personalizados desde otras plataformas de aplicaciones, use los métodos `trackEvent` en los SDK de App Center.

Para asegurarse de que se reciben los eventos personalizados, vaya a la pestaña **Eventos** de la sección **Analytics** de App Center. Los eventos pueden tardar un par de minutos en aparecer desde que se enviaron desde la aplicación.

## <a name="create-an-application-insights-resource"></a>Creación de recursos en Application Insights

Cuando la aplicación envía eventos personalizados y App Center los recibe, tiene que crear un recurso de Application Insights para App Center en Azure Portal:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Crear un recurso** > **Herramientas de desarrollo** > **Application Insights**.

    > [!NOTE]
    > Si esta es la primera vez que crea un recurso de Application Insights, puede obtener más información visitando la documentación [Creación de recursos en Application Insights](../app/create-new-resource.md).

    Aparecerá un cuadro de configuración. Use la tabla siguiente para rellenar los campos de entrada.

    | Configuración        |  Value           | Descripción  |
   | ------------- |:-------------|:-----|
   | **Nombre**      | Algún valor único global, como "myApp-iOS" | Nombre que identifica la aplicación que se está supervisando |
     | **Grupo de recursos**     | Un grupo de recursos nuevo o uno existente desde el menú | El grupo de recursos en que se va a crear el recurso de Application Insights |
   | **Ubicación** | Una ubicación en el menú | Elija una ubicación cerca de usted o de donde se hospeda la aplicación |

3. Haga clic en **Crear**.

Si la aplicación admite varias plataformas (iOS, Android, etc.), es mejor crear recursos de Application Insights independientes, uno para cada plataforma.

## <a name="export-to-application-insights"></a>Exportación a Application Insights

En el nuevo recurso de Application Insights, en la página **Introducción**. Copie la clave de instrumentación del recurso.

En la instancia de [App Center](https://appcenter.ms/) de la aplicación:

1. En la página **Configuración**, haga clic en **Exportar**.
2. Elija **Nueva exportación**, seleccione **Application Insights** y después haga clic en **Personalizar**.
3. Pegue la clave de instrumentación de Application Insights en el cuadro.
4. Dé su consentimiento para aumentar el uso de la suscripción de Azure que contiene el recurso de Application Insights. El primer GB de datos recibidos al mes de cada recurso de Application Insights es gratis. [Más información sobre los precios de Application Insights](https://azure.microsoft.com/pricing/details/application-insights/)

Recuerde que debe repetir este proceso para cada plataforma que su aplicación admita.

Una vez configurada la [exportación](/mobile-center/analytics/export), cada evento personalizado que recibe App Center se copia en Application Insights. Los eventos pueden tardar varios minutos en llegar a Application Insights, así que, si no aparecen inmediatamente, espere un poco antes de realizar otro diagnóstico.

Para proporcionarle más datos cuando se conecte por primera vez, las 48 horas más recientes de eventos personalizados de App Center se exportan automáticamente a Application Insights.

## <a name="start-monitoring-your-app"></a>Inicio de la supervisión de la aplicación

Application Insights puede consultar, segmentar, filtrar y analizar la telemetría de eventos personalizada desde las aplicaciones, de forma más exhaustiva que las herramientas de análisis que App Center ofrece.

1. **Consulte la telemetría de eventos personalizados.** En la página de **información general** de Application Insights, seleccione **Logs (Analytics)** .

   Se abre el portal Logs (Analytics) de Application Insights asociado con el recurso de Application Insights. El portal de Logs (Analytics) permite consultar directamente los datos mediante el lenguaje de consulta de Log Analytics, por lo que puede hacer preguntas arbitrariamente complejas sobre la aplicación y sus usuarios.
   
   Abra una pestaña nueva en el portal de Logs (Analytics) y después pegue la siguiente consulta. Devuelve un recuento de cuántos usuarios distintos han enviado cada evento personalizado desde la aplicación en las últimas 24 horas, ordenados por estos recuentos distintivos.

   ```AIQL
   customEvents
   | where timestamp >= ago(24h)
   | summarize dcount(user_Id) by name 
   | order by dcount_user_Id desc 
   ```

   ![Portal de Logs (Analytics)](./media/mobile-center-quickstart/analytics-portal-001.png)

   1. Seleccione la consulta haciendo clic en cualquier parte de la consulta en el editor de texto.
   2. Después haga clic en **Ir** para ejecutar la consulta. 

   Obtenga más información sobre [Analytics en Application Insights](../log-query/log-query-overview.md) y el [lenguaje de consulta de Log Analytics](https://aka.ms/LogAnalyticsLanguageReference).


2. **Segmente y filtre la telemetría de eventos personalizados.** En la página de **información general** de Application Insights, seleccione **Usuarios** en la tabla de contenido.

   ![Icono de la herramienta de usuarios](./media/mobile-center-quickstart/users-icon-001.png)

   La herramienta Usuarios muestra cuántos usuarios de la aplicación hicieron clic en determinados botones, visitaron determinadas pantallas o realizaron cualquier otra acción de la que se realiza un seguimiento como evento con el SDK de App Center. Si busca una forma de segmentar y filtrar los eventos de App Center, la herramienta Usuarios es una excelente opción.

   ![Herramienta Usuarios](./media/mobile-center-quickstart/users-001.png) 

   Por ejemplo, segmente el uso por zona geográfica; para ello, elija **País o región** en el menú desplegable **Dividido por**.

3. **Analice los patrones de conversión, retención y navegación de la aplicación.** En la página de **información general** de Application Insights, seleccione **Flujos de usuario** en la tabla de contenido.

   ![Herramienta Flujos de usuario](./media/mobile-center-quickstart/user-flows-001.png)

   En la herramienta Flujos de usuario se visualizan los eventos que los usuarios envían después de algún evento inicial. Es útil para obtener una visión general de cómo los usuarios navegan por la aplicación. También puede indicar los lugares que muchos usuarios recorren desde la aplicación o repiten las mismas acciones una y otra vez.

   Además de Flujos de usuario, Application Insights dispone de otras herramientas de análisis de comportamiento de usuarios para responder a preguntas específicas:

   * **Embudos** para analizar y supervisar las tasas de conversión.
   * **Retención** para analizar el grado en que la aplicación retiene a los usuarios con el paso del tiempo.
   * **Workbooks** para combinar visualizaciones y texto en un informe que se pueda compartir.
   * **Cohortes** para nombrar y guardar grupos de usuarios o eventos específicos, para poder hacer referencia a ellos fácilmente desde otras herramientas de análisis.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no desea continuar utilizando Application Insights con App Center, desactive la exportación en App Center y elimine el recurso de Application Insights. Esto evitará que se le cobren cargos adicionales por Application Insights para este recurso.

Para desactivar la exportación en App Center:

1. En App Center, vaya a **Configuración** y elija **Exportar**.
2. Haga clic en la exportación de Application Insights que desee eliminar y después haga clic en **Eliminar exportación** en la parte inferior y confirme.

Para eliminar el recurso de Application Insights:

1. En el menú izquierdo de Azure Portal, haga clic en **Grupos de recursos** y después elija el grupo de recursos en que se creó el recurso de Application Insights.
2. Abra el recurso de Application Insights que desea eliminar. Después haga clic en **Eliminar** en el menú superior del recurso y confirme. Se eliminará de forma permanente la copia de los datos exportados a Application Insights.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Información sobre la forma en que los clientes usan la aplicación](../app/usage-overview.md)

