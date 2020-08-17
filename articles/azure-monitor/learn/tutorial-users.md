---
title: Información sobre sus clientes en Azure Application Insights | Microsoft Docs
description: Tutorial sobre el uso de Azure Application Insights para entender la forma en que los clientes utilizan su aplicación.
ms.subservice: application-insights
ms.topic: tutorial
author: mrbullwinkle
ms.author: mbullwin
ms.date: 09/20/2017
ms.custom: mvc
ms.openlocfilehash: ce9bf6fe6d130cf8511db2b2351c0de01b753d81
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318409"
---
# <a name="use-azure-application-insights-to-understand-how-customers-are-using-your-application"></a>Use Azure Application Insights para entender la forma en que los clientes utilizan su aplicación

Azure Application Insights recopila información de uso que le ayuda a entender la forma en que los usuarios interactúan con su aplicación.  Este tutorial le guía a través de los diferentes recursos disponibles para analizar esta información.  Aprenderá a:

> [!div class="checklist"]
> * Analizar los detalles de los usuarios que accedan a su aplicación
> * Usar información de la sesión para analizar la forma en que los clientes usan su aplicación
> * Definir embudos que le permitan comparar la actividad de usuario deseada con la actividad de usuario real 
> * Crear un libro para consolidar visualizaciones y consultas en un solo documento
> * Agrupar usuarios similares para analizarlos juntos conjuntamente
> * Saber qué usuarios vuelven a su aplicación
> * Inspeccionar la forma en que los usuarios navegan por la aplicación


## <a name="prerequisites"></a>Prerequisites

Para completar este tutorial:

- Instale [Visual Studio 2019](https://www.visualstudio.com/downloads/) con las cargas de trabajo siguientes:
    - ASP.NET y desarrollo web
    - Desarrollo de Azure
- Descargue e instale [Visual Studio Snapshot Debugger](https://aka.ms/snapshotdebugger).
- Implemente una aplicación de .NET en Azure y [habilite el SDK de Application Insights](../app/asp-net.md). 
- [Envíe telemetría desde su aplicación](../app/usage-overview.md#send-telemetry-from-your-app) para agregar eventos personalizados o vistas de página
- Enviar el [contexto del usuario](../app/usage-send-user-context.md) para realizar un seguimiento de lo que un usuario hace con el tiempo y utilizar plenamente las características de uso.

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="get-information-about-your-users"></a>Obtención de información acerca de los usuarios
El panel **Usuarios** le permite conocer detalles importantes acerca de los usuarios de varias formas. Este panel se puede utilizar para saber desde dónde se conectan los usuarios, conocer detalles de sus clientes y las áreas de la aplicación a las que acceden. 

1. Seleccione **Application Insights** y, a continuación, seleccione la suscripción.
2. Seleccione **Usuarios** en el menú.
3. La vista predeterminada muestra el número de usuarios únicos que se han conectado a la aplicación en las últimas 24 horas.  Puede cambiar la ventana de tiempo y establezca otros criterios para filtrar esta información.

    ![Generador de consultas](media/tutorial-users/QueryBuilder.png)

6. Haga clic en la lista desplegable **Durante** y cambie la ventana de tiempo a 7 días.  Así aumentan los datos que se incluyen en los distintos gráficos del panel.

    ![Cambiar el intervalo de tiempo](media/tutorial-users/TimeRange.png)

4. Haga clic en la lista desplegable **Dividido por** para agregar al grafo un desglose por una propiedad del usuario.  Seleccione **País o región**.  El grafo incluye los mismos datos, pero permite ver un desglose del número de usuarios de cada país o región.

    ![Grado de país o región](media/tutorial-users/CountryorRegion.png)

5. Coloque el cursor sobre las distintas barras del gráfico y tenga en cuenta que el recuento de cada país o región refleja solo la ventana de tiempo representada por dicha barra.
6. Eche un vistazo a la columna **Insights** a la derecha, en la que se realiza el análisis de los datos de usuario.  Proporciona información como el número de sesiones únicas a lo largo del período de tiempo, y registros con propiedades comunes significativas de los datos del usuario 

    ![Columna Insights](media/tutorial-users/insights.png)


## <a name="analyze-user-sessions"></a>Análisis de sesiones del usuario
El panel **Sesiones** es similar al panel **Usuarios**.  Mientras que **Usuarios** le ayuda a conocer los detalles acerca de los usuarios que acceden a su aplicación, **Sesiones** le ayuda a saber la forma en que dichos usuarios han utilizado su aplicación.  

1. Seleccione **Sesiones** en el menú.
2. Eche un vistazo al grafo y tenga en cuenta que cuenta con las mismas opciones para filtrar y desglosar los datos que en el panel **Usuarios**.

    ![Generador de consultas de sesiones](media/tutorial-users/SessionsBuilder.png)

3. El panel **Ejemplo de estas sesiones** de la derecha enumera las sesiones que incluyen un gran número de eventos.  Son sesiones cuyo análisis es interesante.

    ![Ejemplo de estas sesiones](media/tutorial-users/SessionsSample.png)

4. Haga clic en una de las sesiones para ver su **escala de tiempo**, que muestra todas las acciones de las sesiones.  Esto puede ayudarle a identificar información como las sesiones con un gran número de excepciones.

    ![Escala de tiempo de las sesión](media/tutorial-users/SessionsTimeline.png)

## <a name="group-together-similar-users"></a>Agrupar usuarios similares
Una **cohorte** es un conjunto de usuarios agrupados por tener características similares.  Las cohortes se pueden usar para filtrar datos de otros paneles, lo que le permite analizar grupos de usuarios concretos.  Por ejemplo, puede desear analizar solo aquellos usuarios que han realizado una compra.

1.  Seleccione **Cohortes** en el menú.
2.  Haga clic en **Nuevo** para crear una cohorte nueva.
3.  Seleccione la lista desplegable **Que usaron** y seleccione una acción.  Solo se incluirán los usuarios que realizaron esta acción dentro de la ventana de tiempo del informe.

    ![Cohorte que llevó a cabo las acciones especificadas](media/tutorial-users/CohortsDropdown.png)

4.  Seleccione **Usuarios** en el menú.
5.  En la lista desplegable **Mostrar**, seleccione la cohorte que acaba de crear.  Los datos del gráfico se limitan a dichos usuarios.

    ![Cohorte en la herramienta de usuarios](media/tutorial-users/UsersCohort.png)


## <a name="compare-desired-activity-to-reality"></a>Comparación de la actividad deseada con la realidad
Mientras que los paneles anteriores se centran en lo que han hecho los usuarios de la aplicación, **Embudos** se centra en lo que desea que hagan los usuarios.  Un embudo representa un conjunto de pasos de una aplicación y el porcentaje de usuarios que se mueven entre dichos pasos.  Por ejemplo, puede crear un embudo que mida el porcentaje de usuarios que se conectan a una aplicación que buscan un producto.  A continuación, puede ver el porcentaje de usuarios que agregan dicho producto a un carro de la compra y, después, el porcentaje de ellos que completan una compra.

1. Seleccione **Embudos** en el menú y haga clic en **Nuevo**. 

    ![Captura de pantalla que muestra cómo crear un nuevo embudo.](media/tutorial-users/funnelsnew.png)

2. Escriba un **nombre de embudo**.
3. Cree un embudo con un mínimo de dos pasos y seleccione una acción para cada paso.  La lista de acciones se crea a partir de los datos de uso recopilados por Application Insights.

    ![Captura de pantalla que muestra cómo crear pasos en un embudo nuevo.](media/tutorial-users/funnelsedit.png)

4. Haga clic en **Guardar** para guardar el embudo y, después, vea sus resultados.  La ventana a la derecha del embudo muestra los eventos más comunes antes de la primera actividad y después de la última actividad para ayudarle a conocer las tendencias del usuario alrededor de la secuencia determinada.

    ![Captura de pantalla que muestra los resultados del evento de un embudo recién creado.](media/tutorial-users/funnelsright.png)


## <a name="learn-which-customers-return"></a>Información acerca de los clientes que vuelven
**Retención** le ayuda a saber qué usuarios vuelven a una aplicación.  

1. Seleccione **Retención** en el menú.
2. De forma predeterminada, la información analizada incluye aquellos usuarios que realizaron alguna acción y volvieron para llevar a cabo alguna acción.  Este filtro se puede cambiar para que incluya a cualquiera, por ejemplo, solo aquellos usuarios que hayan vuelto después de completar una compra.

    ![Captura de pantalla que muestra cómo establecer un filtro de retención.](media/tutorial-users/retentionquery.png)

3. Los usuarios que vuelven que coinciden con los criterios se muestran tanto en forma de gráfico como de tabla en distintas duraciones.  El patrón más frecuente es una caída gradual en la vuelta de los usuarios con el paso del tiempo.  Una caída repentina de un período de tiempo al siguiente puede provocar un problema. 

    ![Captura de pantalla que muestra un grafo para los usuarios que coincidan con los criterios establecidos para un filtro de retención.](media/tutorial-users/retentiongraph.png)

## <a name="analyze-user-navigation"></a>Análisis de la navegación del usuario
Un **flujo de usuarios** visualiza la forma en que los usuarios navegan entre las páginas y características de la aplicación.  Esto le ayuda a responder a preguntas como a qué lugar se suelen mover los usuarios desde una página concreta, cómo salen habitualmente de su aplicación y si hay acciones que se repitan de manera regular.

1.  Seleccione **Flujos de usuarios** en el menú.
2.  Haga clic en **Nuevo** para crear un nuevo flujo de usuario y, a continuación, haga clic en **Editar** para editar sus detalles.
3.  Aumente el valor de **Intervalo de tiempo** a 7 días y, luego, seleccione un evento inicial.  El flujo hará un seguimiento de las sesiones de usuario que empiezan por ese evento.

    ![Captura de pantalla que muestra cómo crear un nuevo flujo de usuario.](media/tutorial-users/flowsedit.png)

4.  Se muestra el flujo de usuario y puede ver las rutas de acceso de los diferentes usuarios y el número de sesiones.  Las líneas azules indican una acción que el usuario ha realizado después de la acción actual.  Una línea roja indica el final de la sesión del usuario.

    ![Captura de pantalla que muestra la visualización de rutas de acceso de usuario y recuentos de sesiones para un flujo de usuario.](media/tutorial-users/flows.png)

5.  Para quitar un evento del flujo, haga clic en la **x** de la esquina de la acción y, después, en **Crear gráfico**.  El grafo se vuelve a dibujar sin ninguna de las instancias de dicho evento.  Haga clic en **Editar** para ver que el evento se ha agregado a **Eventos excluidos**.

    ![Captura de pantalla que muestra la lista de eventos excluidos de un flujo de usuario.](media/tutorial-users/flowsexclude.png)

## <a name="consolidate-usage-data"></a>Consolidación de datos de uso
Los **libros** combinan las visualizaciones de datos, las consultas de Analytics y texto en documentos interactivos.  Se pueden usar para agrupar la información de uso común, consolidar la información de un incidente determinado o informar al equipo sobre el uso de la aplicación.

1.  Seleccione **Libros** en el menú.
2.  Haga clic en **Nuevo** para crear un libro nuevo.
3.  Ya se proporciona una consulta que incluye todos los datos de uso en el último día que se muestra como un gráfico de barras.  Puede usar esta consulta, editarla manualmente o hacer clic en **Consultas de ejemplo** para seleccionar otras consultas útiles.

    ![Captura de pantalla que muestra una lista de consultas de ejemplo que puede usar.](media/tutorial-users/samplequeries.png)

4.  Haga clic en **Edición finalizada**.
5.  Haga clic en **Editar** en el panel superior para editar el texto de la parte superior del libro.  Para darle formato se usa Markdown.

    ![Captura de pantalla que muestra cómo editar el texto en la parte superior del libro.](media/tutorial-users/markdown.png)

6.  Haga clic en **Agregar usuarios** para agregar un grafo con información del usuario.  Si lo desea, edite los detalles del grafo y, después, haga clic en **Edición finalizada** para guardarlo.


## <a name="next-steps"></a>Pasos siguientes
Ahora que ha aprendido a analizar a los usuarios, pase al siguiente tutorial para aprender a crear paneles personalizados que combinan esta información con otros datos útiles de la aplicación.

> [!div class="nextstepaction"]
> [Creación de paneles personalizados](./tutorial-app-dashboards.md)

