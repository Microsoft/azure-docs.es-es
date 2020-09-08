---
title: Almacenamiento de eventos de Azure Media Services en Azure Log Analytics
titleSuffix: Azure Media Services
description: Obtenga información sobre cómo almacenar eventos de Azure Media Services en Azure Log Analytics.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
ms.date: 08/24/2020
ms.author: inhenkel
ms.openlocfilehash: dfd7b3dcc7e27dd1251c35c6272b5aa7f8036006
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88941916"
---
# <a name="tutorial-store-azure-media-services-events-in-azure-log-analytics"></a>Tutorial: Almacenamiento de eventos de Azure Media Services en Azure Log Analytics

## <a name="azure-media-services-events"></a>Eventos de Azure Media Services

Azure Media Services v3 emite eventos en [Azure Event Grid](media-services-event-schemas.md). Puede suscribirse a los eventos de muchas maneras y almacenarlos en almacenes de datos. En este tutorial, se suscribirá a los eventos de Media Services mediante un [flujo de Logic Apps](https://azure.microsoft.com/services/logic-apps/). La instancia de Logic Apps se desencadenará para cada evento y almacenará el cuerpo del evento en Azure Log Analytics. Una vez que los eventos se encuentran en Azure Log Analytics, puede usar otros servicios de Azure para crear un panel, un monitor y una alerta de estos eventos, aunque no vamos a cubrir eso en este tutorial.

> [!NOTE]
> Resultaría útil si ya está familiarizado con el uso de FFmpeg como codificador en el entorno local.  Si no lo está, está bien. A continuación se incluye la línea de comandos y las instrucciones para la transmisión de un vídeo.

Aprenderá a:

> [!div class="checklist"]
> * Crear un flujo de Logic Apps sin código.
> * Suscribirse a los temas de eventos de Media Services.
> * Analizar eventos y almacenarlos en Azure Log Analytics.
> * Consultar eventos de Azure Log Analytics.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

> * [Una suscripción de Azure](how-to-set-azure-subscription.md)
> * Una cuenta y grupo de recursos de [Media Services](create-account-howto.md).
> * Una instalación de [FFmpeg](https://ffmpeg.org/download.html) para su sistema operativo.
> * Un área de trabajo de [Log Analytics](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace)

## <a name="subscribe-to-a-media-services-event-with-logic-app"></a>Suscripción a un evento de Media Services con una aplicación lógica

1. En Azure Portal, si aún no lo ha hecho, cree un área de trabajo de [Log Analytics](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace). Necesitará el id. del área de trabajo y una de las claves, por lo que debe mantener abierta la ventana del explorador. A continuación, abra el portal en otra pestaña o ventana.

1. Vaya a la cuenta de Azure Media Services y seleccione **Eventos**. Esto mostrará todos los métodos para suscribirse a los eventos de Azure Media Services.
    > [!div class="mx-imgBorder"]
    > ![Portal de Azure Media Services](media/tutorial-events-log-analytics/select-events-01a.png)

1. Seleccione el **icono de Logic Apps** para crear una aplicación lógica. Se abrirá el diseñador de aplicación lógica, en el que podrá crear el flujo para capturar los eventos e incluirlos en Log Analytics. 
    > [!div class="mx-imgBorder"]
    > ![Creación de la aplicación lógica](media/tutorial-events-log-analytics/select-logic-app-02.png)

1. Seleccione el **icono +** , seleccione el inquilino que quiere usar y, después, seleccione Iniciar sesión. Verá una solicitud de inicio de sesión de Microsoft.
    > [!div class="mx-imgBorder"]
    > ![Conéctese a Azure Event Grid](media/tutorial-events-log-analytics/select-event-add-grid-03.png)
![Seleccione el inquilino](media/tutorial-events-log-analytics/select-tenant-03a.png)

1. Seleccione **Continuar** para suscribirse a los eventos de Media Services.
    > [!div class="mx-imgBorder"]
    > ![Se ha conectado a Azure Event Grid](media/tutorial-events-log-analytics/select-continue-04.png)

1. En la lista **Tipo de recurso**, busque "Microsoft.Media.MediaServices".
    > [!div class="mx-imgBorder"]
    >![Eventos del recurso de Azure Media Services](media/tutorial-events-log-analytics/locate-azure-media-services-events-05.png)

1. Seleccione el **elemento Tipo de evento**. Se mostrará una lista de todos los eventos que emite Azure Media Services. Puede seleccionar los eventos de los que quiere realizar un seguimiento. Puede agregar varios tipos de eventos. (Más adelante, realizará un pequeño cambio en el flujo de la aplicación lógica para almacenar cada tipo de evento en un registro de Log Analytics independiente y propagar el nombre del tipo de evento al nombre del registro de Log Analytics de forma dinámica).
    > [!div class="mx-imgBorder"]
    > ![Tipo de evento de Azure Media Services](media/tutorial-events-log-analytics/select-azure-media-services-event-type-06.png)

1. Seleccione **Guardar como**.

1. Especifique un nombre para la aplicación lógica.  De forma predeterminada, se selecciona el grupo de recursos. Deje las otras opciones de configuración como están y, a continuación, seleccione **Crear**.  Volverá a la pantalla de inicio de Azure.
    > [!div class="mx-imgBorder"]
    > ![Interfaz de asignación de nombre para aplicaciones lógicas](media/tutorial-events-log-analytics/give-logic-app-name-06a.png)

## <a name="create-an-action"></a>Creación de una acción

Ahora que está suscrito a los eventos, cree una acción.

1. Si el portal le ha devuelto a la pantalla principal, vuelva a la aplicación lógica que acaba de crear; para ello, busque en Todos los recursos el nombre de la aplicación.

1. Seleccione la aplicación y, a continuación, seleccione **Diseñador de aplicación lógica**. Se abrirá el panel del diseñador.

1. Seleccione **+ Nuevo paso**.

1. Ya que quiere enviar los eventos al servicio Azure Log Analytics, busque "Azure Log Analytics" y seleccione "Azure Log Analytics Data Collector".
    > [!div class="mx-imgBorder"]
    > ![Azure Log Analytics Data Collector](media/tutorial-events-log-analytics/select-azure-log-analytics-data-collector-07.png)

1. Para conectarse al área de trabajo de Log Analytics, necesita el id. del área de trabajo y una clave del agente. Abra Azure Portal en una nueva pestaña o ventana y navegue hasta el área de trabajo de Log Analytics que creó antes de iniciar este tutorial.
    > [!div class="mx-imgBorder"]
    > ![Id. del área de trabajo de Azure Log Analytics](media/tutorial-events-log-analytics/log-analytics-workspace-id-08.png)

1. En el menú de la izquierda, busque **Administración de agentes** y seleccione esa opción. Esto le mostrará las claves del agente que se han generado.
    > [!div class="mx-imgBorder"]
    > ![Administración de agentes de Azure Log Analytics](media/tutorial-events-log-analytics/select-agents-management-09.png)

1. Copie el *identificador del área de trabajo*.
    > [!div class="mx-imgBorder"]
    > ![Copie el identificador del área de trabajo](media/tutorial-events-log-analytics/copy-workspace-id.png)

1. En la otra pestaña o ventana del explorador, en Azure Log Analytics Data Collector, seleccione **Enviar datos**, asigne un nombre a la conexión y, a continuación, pegue el *Id. del área de trabajo* en el campo **Id. del área de trabajo**.

1. Vuelva a la pestaña o ventana del explorador correspondiente al área de trabajo y copie la *Clave del área de trabajo*.
    > [!div class="mx-imgBorder"]
    > ![Clave principal de la administración de agentes](media/tutorial-events-log-analytics/agents-management-primary-key-10.png)

1. En la otra pestaña o ventana del explorador, pegue la *Clave del área de trabajo* en el campo **Clave del área de trabajo**.

1. Seleccione **Crear**. Ahora creará el cuerpo de la solicitud JSON y el nombre del registro personalizado.

1. Seleccione el campo **Cuerpo de la solicitud JSON**.  Aparecerá un vínculo para **Agregar contenido dinámico**.

1. Seleccione **Agregar contenido dinámico** y, a continuación, seleccione **Tema**.

1. Haga lo mismo para el **Nombre de registro personalizado**.
    > [!div class="mx-imgBorder"]
    > ![Tema seleccionado](media/tutorial-events-log-analytics/topic-selected.png)

1. Seleccione la **vista Código** de la aplicación lógica. Busque las entradas y las líneas de tipo de registro.
    > [!div class="mx-imgBorder"]
    > ![Vista Código de dos líneas](media/tutorial-events-log-analytics/code-view-two-lines.png)

1. Cambie el valor de `body` de `"@triggerBody()?['topic']"` a `"@{triggerBody()}"`. Esto permite analizar todo el mensaje para Log Analytics.

1. Cambie el valor de `Log-Type` de `"@triggerBody()?['topic']"` a `"@replace(triggerBody()?['eventType'],'.','')"`. (Esto reemplazará los elementos ".", ya que no se permiten en los nombres de los registros de Log Analytics).
    > [!div class="mx-imgBorder"]
    > ![JSON de la aplicación lógica después del cambio](media/tutorial-events-log-analytics/changed-lines.png)

1. Seleccione **Guardar**.

1. Para comprobarlo, seleccione **Diseñador de aplicación lógica**.
    > [!div class="mx-imgBorder"]
    > ![Compruebe los pasos del cuerpo y la función](media/tutorial-events-log-analytics/verify-changes-to-json.png)

1. Cuando examine todos los recursos del grupo de recursos, se enumerará una aplicación lógica y dos conectores de API de esta: uno para los eventos y otro para Log Analytics. Para obtener más información acerca de los temas del sistema de Event Grid, consulte los[Temas del sistema en Event Grid](https://docs.microsoft.com/azure/event-grid/system-topics).
    > [!div class="mx-imgBorder"]
    > ![Vea todos los recursos nuevos del grupo de recursos](media/tutorial-events-log-analytics/contoso-rg-listing.png)

## <a name="test"></a>Prueba

Para probar cómo funciona realmente, cree un evento en directo en Azure Media Services. Cree un evento en directo de RTMP y use FFmpeg para enviar una transmisión "directa" basada en un archivo .mp4 de ejemplo. Una vez creado el evento, obtenga la URL de introducción de RTMP.

1. En la cuenta de Media Services, seleccione **Streaming en vivo**.
    > [!div class="mx-imgBorder"]
    > ![Creación de un evento en directo de Azure Media Services](media/tutorial-events-log-analytics/live-event.png)

1. Seleccione **Agregar evento en directo**.

1. Escriba un nombre en el campo **Nombre del evento en directo**. (El campo **Descripción** es opcional).

1. Seleccione la codificación en la nube **Estándar**.

1. Seleccione **720p predeterminado** para el valor preestablecido de codificación.

1. Seleccione el protocolo de entrada **RTMP**.

1. Seleccione **Sí** para la dirección URL de entrada persistente.

1. Seleccione **Sí** para iniciar el evento cuando se cree.

    > [!WARNING]
    > La facturación se iniciará si selecciona la opción Sí.  Si quiere esperar para iniciar la transmisión *justo antes de* que empiece a transmitir con FFmpeg, seleccione **No** y recuerde iniciar el evento en directo.

    > [!div class="mx-imgBorder"]
    > ![Configuración de eventos en directo](media/tutorial-events-log-analytics/live-event-settings.png)

1. Seleccione la casilla **Tengo todos los derechos para usar el contenido/archivo...** .

1. Seleccione **Revisar + crear**.

1. Revise la configuración y seleccione **Crear**.  Aparecerá la lista de eventos en directo y se mostrará la URL de introducción de eventos en directo.

1. Copie la **URL de introducción** en el Portapapeles.

1. Seleccione el **evento en directo** en la lista para ver la vista de productor.

### <a name="stream-with-ffmpeg-cli"></a>Transmisión con la CLI de FFmpeg

1. Use la siguiente línea de comandos.

    ```AzureCLI
    ffmpeg -i <localpathtovideo> -map 0 -c:v libx264 -c:a copy -f flv <ingestURL>/mystream
    ```

1. Cambie `<ingestURL>` por la URL de introducción que copió en el portapapeles.
1. Cambie `<localpathtovideo>` por la ruta de acceso local del archivo que quiere transmitir desde FFmpeg.
1. Agregue un nombre único al final; por ejemplo, `mystream`.
1. Modifique la línea de comandos para que refleje el archivo de origen de prueba y cualquier otra variable del sistema.
1. Ejecute el comando. Después de unos segundos, el reproductor de la "vista de productor" debe iniciar la transmisión. (Actualice el reproductor si el vídeo no se muestra automáticamente).

    > [!div class="mx-imgBorder"]
    > ![Compruebe la ingesta correcta del vídeo en el reproductor de vista previa del productor](media/tutorial-events-log-analytics/live-event-producer-view.png)

## <a name="verify-the-events"></a>Comprobación de los eventos

Con el streaming en vivo, Azure Media Services emite varios eventos que desencadenan el flujo de la aplicación lógica. Para comprobarlo, vaya a la aplicación lógica y determine si los eventos de Media Services han activado algún desencadenador.

1. Vaya a la página de información general de la aplicación lógica; se debería ver el "Historial de ejecución" con trabajos que se completaron correctamente.
    > [!div class="mx-imgBorder"]
    > ![Compruebe la ejecución correcta del trabajo en la aplicación lógica](media/tutorial-events-log-analytics/run-history.png)

1. Seleccione un trabajo realizado correctamente. Se muestran los detalles del trabajo durante el tiempo de ejecución.
1. Seleccione **Enviar datos** para expandirlo. En este caso, el evento `MicrosoftMediaLiveEventEncoderConnected` muestra que se capturó, así como el cuerpo analizado. Esto es lo que se envía al área de trabajo de Azure Log Analytics.
    > [!div class="mx-imgBorder"]
    > ![Vea los datos enviados](media/tutorial-events-log-analytics/verify-send-data.png)

## <a name="verify-the-logs"></a>Comprobación de los registros

1. Vaya al área de trabajo de Log Analytics que creó anteriormente.

1. Seleccione **Registros**.
1. Cierre el elemento emergente Consultas de ejemplo.
1. Se mostrará una lista de registros personalizados. Seleccione la flecha abajo para expandirla. Allí verá el nombre de evento `MicrosoftMediaLiveEventEncoderConnected`.
1. Seleccione el nombre del evento para expandirlo.
1. Al seleccionar el icono de ojo, se mostrará una vista previa del resultado de la consulta.
1. Seleccione **Ver en el editor de consultas** y, después, seleccione el elemento debajo de la lista **TimeGenerated UTC** para expandirlo y ver los datos sin procesar.

![Consulte la salida detallada de los eventos en Log Analytics](media/tutorial-events-log-analytics/raw-data.png)

## <a name="delete-resources"></a>Eliminar recursos

Si no quiere seguir usando los recursos que creó en este tutorial, asegúrese de eliminarlos todos del grupo de recursos o se le seguirá cobrando.

## <a name="next-steps"></a>Pasos siguientes

Puede crear consultas diferentes y guardarlas. Estas se podrán agregar al [panel de Azure](https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-logs-dashboards).
