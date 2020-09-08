---
title: 'Reproducción de las grabaciones de varios días: Azure'
description: En este tutorial, obtendrá información sobre cómo usar las API de Azure Media Services para reproducir una grabación continua de vídeo de varios días.
ms.topic: tutorial
ms.date: 05/27/2020
ms.openlocfilehash: 19a42c61ef250fecce4503d3aed70b36d71ee3cf
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88649098"
---
# <a name="tutorial-playback-of-multi-day-recordings"></a>Tutorial: Reproducción de las grabaciones de varios días  

Este tutorial se basa en el tutorial [Grabación continua de vídeo](continuous-video-recording-concept.md) (CVR). En este tutorial, obtendrá información sobre cómo usar las API de Azure Media Services para reproducir una grabación continua de vídeo de varios días desde la nube. 

Esto resulta útil en escenarios como la seguridad pública, en los que es necesario mantener una grabación de la secuencia de una cámara durante varios días (o semanas) y hay una necesidad ocasional de ver partes específicas de esa secuencia.

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Ejecutar la aplicación de ejemplo que muestra cómo se puede examinar el contenido de una grabación de varios días
> * Ver partes seleccionadas de esa grabación

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura  

Se recomienda leer las siguientes páginas de documentación:

* [Introducción a Live Video Analytics en IoT Edge](overview.md)
* [Terminología de Live Video Analytics en IoT Edge](terminology.md)
* [Concepto de grafo multimedia](media-graph-concept.md)
* [Grabación continua de vídeo](continuous-video-recording-concept.md) 
* [Guía de procedimientos: Reproducción de grabaciones](playback-recordings-how-to.md)
* [Tutorial: Grabación continua de vídeo](continuous-video-recording-tutorial.md)

## <a name="prerequisites"></a>Requisitos previos

* Complete el [Tutorial de CVR](continuous-video-recording-tutorial.md). Aunque este tutorial y las API pertinentes que se describen en [Tutorial: Grabación continua de vídeo](continuous-video-recording-tutorial.md) se aplican a grabaciones que tienen 5 minutos o más, se recomienda grabar un vídeo de al menos 5 horas. Las API que se utilizan para examinar las grabaciones se muestran mejor con grabaciones largas.
* Se recomienda ejecutar este tutorial mientras el [Tutorial: Grabación continua de vídeo](continuous-video-recording-tutorial.md) todavía está en ejecución, es decir, todavía está grabando vídeo en la nube.

## <a name="run-the-sample"></a>Ejecución del ejemplo 

Como parte del [tutorial de CVR](continuous-video-recording-tutorial.md), ya habrá creado una cuenta de Media Services. Para este tutorial, deberá tener acceso completo a la API para esa cuenta. Puede usar los pasos descritos en [Obtención de credenciales para acceder a la API de Media Services](../latest/access-api-howto.md?tabs=portal) para crear una entidad de servicio. Debería poder obtener un bloque JSON desde Azure Portal con un aspecto similar a este:

```
{
    "AadClientId": "<<INSERT_AZURE_AD_APP_ID_HERE>>",
    "AadSecret": "<<INSERT_AZURE_AD_APP_SECRET_HERE>>",
    "AadTenantDomain": "<<YOUR_TENANT_DOMAIN>>",
    "AadTenantId": "<<YOUR_TENANT_ID>>",
    "AccountName": "<<YOUR_ACCOUNT_NAME>>",
    "Location": "<<YOUR_REGION_NAME>>",
    "ResourceGroup": "<<YOUR_RESOURCE_GROUP>>",
    "SubscriptionId": "<<YOUR_SUBSCRIPTION_ID>>",
    "ArmAadAudience": "https://management.core.windows.net",
    "ArmEndpoint": "https://management.azure.com"
}
```

Después, en Visual Studio Code, abra src/ams-asset-player. Esta carpeta contiene los archivos necesarios para este tutorial. Abra el archivo appsettings.json y copie su contenido en un archivo nuevo, appsettings.development.json. Realice las siguientes modificaciones en el último archivo:

```
  "AMS" : {
    "subscriptionId" : "Use value of SubscriptionId above",
    "resourceGroup" : "Use value of ResourceGroup above",
    "accountId" : "Use value of AccountName above",
    "aadTenantId" : "Use value of AadTenantId above",
    "aadClientId" : "Use value of AadClientId above",
    "aadSecret" : "Use value of AadSecret above"
} 
```

En Visual Studio Code, puede hacer clic en el icono de ejecución de la izquierda (o Ctrl + Mayús + D) para que se ejecuten las aplicaciones disponibles:

![Ejecute](./media/playback-multi-day-recordings-tutorial/run.png)
 
Seleccione la aplicación del reproductor de recursos de AMS en el cuadro desplegable, como se muestra a continuación, y presione F5 para iniciar la depuración.

![Depurar](./media/playback-multi-day-recordings-tutorial/debug.png)

La aplicación de ejemplo compilará e iniciará la aplicación predeterminada del explorador y abrirá la página del reproductor de recursos de AMS.

> [!NOTE]
> En función de la configuración de seguridad del explorador, es posible que aparezca un mensaje de advertencia. Dado que la página web se ejecuta localmente, puede omitir la advertencia.

El reproductor de recursos de AMS le pide que escriba el nombre de un recurso de Media Services. Debe usar el nombre del recurso que usó para grabar vídeo en [Tutorial: Grabación continua de vídeo](continuous-video-recording-tutorial.md).

Tras escribir el nombre del recurso y pulsar el botón de envío, el código del reproductor cargará la dirección URL de streaming. Para más información, consulte [Guía de procedimientos: Reproducción de grabaciones](playback-recordings-how-to.md). Si, como se recomienda, todavía sigue la grabación en el recurso, el reproductor lo detectará e intentará situar la reproducción en la parte más reciente del vídeo grabado. Puede ver la marca de tiempo (en UTC) en la parte superior izquierda del reproductor. En la captura de pantalla siguiente, puede observar que está seleccionado el botón "Live" (En directo).

![STREAM](./media/playback-multi-day-recordings-tutorial/assetplayer1.png)
 
En el lado derecho del reproductor, puede ver los controles para examinar el archivo. Los años, meses y fechas de este control se rellenan mediante la API availableMedia documentada en [Guía de procedimientos: Reproducción de grabaciones](playback-recordings-how-to.md).
Al expandir el día, si el tutorial de CVR se ha ejecutado durante varias horas, verá un resultado similar al siguiente:

![Examinar archivo](./media/playback-multi-day-recordings-tutorial/results.png)

El origen de la fuente de vídeo del tutorial es un archivo MKV. Cuando el simulador RSTP (consulte [Live555 Media Server](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555)) alcanza el final del archivo, finaliza la secuencia. El nodo de origen RTSP del grafo multimedia lo detecta, vuelve a establecer la conexión y el vídeo se reanuda. Entre cada uno de estos fines de archivo y la reconexión, hay un vacío en el archivo grabado, que se muestra como una nueva entrada en los resultados de availableMedia.

![Results](./media/playback-multi-day-recordings-tutorial/assetplayer2.png)
 
Al hacer clic en una entrada de la lista, la aplicación crea una dirección URL de streaming con el filtro adecuado, como https://{hostname}/{locatorId}/content.ism/manifest(format=mpd-time-csf,startTime=YYYY-MM-DDTHH:MM:SS). El reproductor cargará esta dirección URL y la reproducción comenzará en el valor de startTime (hora de inicio) deseado.

Como alternativa, puede usar horas de inicio y finalización específicas, mediante los controles de la parte inferior de la página. Puede usar los resultados de la llamada a availableMedia, como se muestra en la parte derecha, como guía para los valores permitidos para las horas de inicio y finalización. El detalle de las restricciones de los filtros de hora de inicio y de hora de finalización se documentan en [Guía de procedimientos: Reproducción de grabaciones](playback-recordings-how-to.md). Después de seleccionar los valores de las horas, al hacer clic en Submit (Enviar), la aplicación cargará el reproductor con una dirección URL de streaming como: https://{hostname}/{locatorId}/content.ism/manifest(format=mpd-time-csf,startTime=YYYY-MM-DDTHH:MM:SS,endTime= YYYY-MM-DDTHH:MM:SS) y la reproducción comenzará en el valor de startTime (hora de inicio) deseado.

## <a name="next-steps"></a>Pasos siguientes

[Grabación de vídeo basada en eventos en la nube y reproducción desde la nube](event-based-video-recording-tutorial.md)
