---
title: Streaming en vivo con codificadores locales mediante Azure Portal | Microsoft Docs
description: Este tutorial le guiará por los pasos para crear un canal que esté configurado para una entrega de paso a través.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 6f4acd95-cc64-4dd9-9e2d-8734707de326
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: 51d3d859bbd176a6e1fac92b75c6120301a5066f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084524"
---
# <a name="perform-live-streaming-with-on-premises-encoders-using-azure-portal"></a>Realización de streaming en vivo con codificadores locales mediante Azure Portal
> [!div class="op_single_selector"]
> * [Portal](media-services-portal-live-passthrough-get-started.md)
> * [.NET](media-services-dotnet-live-encode-with-onpremises-encoders.md)
> * [REST](/rest/api/media/operations/channel)
> 
> 

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Este tutorial le guiará por los pasos para usar el Portal de Azure para crear un **Canal** que esté configurado para una entrega de paso a través. 

## <a name="prerequisites"></a>Prerrequisitos
Estos son los requisitos previos para completar el tutorial.

* Una cuenta de Azure. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/). 
* Una cuenta de Media Services. Para crear una cuenta de Media Services, consulte el tema [Creación de una cuenta de Media Services](media-services-portal-create-account.md).
* Una cámara web. Por ejemplo, el [codificador Telestream Wirecast](media-services-configure-wirecast-live-encoder.md). 

Es importante que revise los artículos siguientes:

* [Compatibilidad con RTMP de Azure Media Services y codificadores en directo.](https://azure.microsoft.com/blog/2014/09/18/azure-media-services-rtmp-support-and-live-encoders/)
* [Información general de streaming en vivo con Azure Media Services](media-services-manage-channels-overview.md)
* [Streaming en vivo con codificadores locales que crean transmisiones de velocidad de bits múltiple](media-services-live-streaming-with-onprem-encoders.md)

## <a name="common-live-streaming-scenario"></a><a id="scenario"></a>Escenario común de streaming en vivo

Los pasos siguientes describen las tareas que conlleva la creación de aplicaciones comunes de streaming en vivo que usan canales configurados para entregas de paso a través. Este tutorial muestra cómo crear y administrar un canal de paso a través y eventos en directo.

> [!NOTE]
> Asegúrese de que el punto de conexión de streaming desde el que va a transmitir el contenido esté en estado **Running** (En ejecución). 
    
1. Conecte una cámara de vídeo a un equipo. <br/>Para obtener ideas para la configuración, consulte [Simple and portable event video gear setup]( https://link.medium.com/KNTtiN6IeT) (Equipo de vídeo para eventos sencillo y portátil).
1. Inicie y configure un codificador en directo local que genere una transmisión de RTMP o MP4 fragmentado con velocidad de bits múltiple. Para obtener más información, consulte [Compatibilidad con RTMP de Azure Media Services y codificadores en directo](https://go.microsoft.com/fwlink/?LinkId=532824).<br/>Vea también este blog: [Live streaming production with OBS](https://link.medium.com/ttuwHpaJeT) (Producción de streaming en vivo con OBS).
   
    Este paso también puede realizarse después de crear el canal.
1. Cree e inicie un canal de paso a través.
1. Recupere la URL de ingesta de canales. 
   
    El codificador en directo usa la URL de ingesta para enviar la secuencia al canal.
1. Recupere la URL de vista previa de canal. 
   
    Use esta dirección URL para comprobar que el canal recibe correctamente la secuencia en vivo.
1. Cree un evento o programa en directo. 
   
    Con el Portal de Azure, al crear un evento en directo también se crea un recurso. 

1. Inicie el evento o programa cuando esté listo para iniciar el streaming y el archivo.
1. Si lo desea, puede señalar el codificador en directo para iniciar un anuncio. El anuncio se inserta en el flujo de salida.
1. Detenga el evento o programa cuando quiera detener el streaming y el archivo del evento.
1. Elimine el evento o programa (y, opcionalmente, elimine el recurso).     

> [!IMPORTANT]
> Consulte [Streaming en vivo con codificadores locales que crean transmisiones de velocidad de bits múltiple](media-services-live-streaming-with-onprem-encoders.md) para obtener información acerca de los conceptos y las consideraciones relacionadas con el streaming en vivo con codificadores locales y canales de paso a través.
> 
> 

## <a name="to-view-notifications-and-errors"></a>Visualización de notificaciones y errores
Si desea ver las notificaciones y los errores generados por el Portal de Azure, haga clic en el icono de notificación.

![Notificaciones](./media/media-services-portal-passthrough-get-started/media-services-notifications.png)

## <a name="create-and-start-pass-through-channels-and-events"></a>Creación e inicio de canales de paso a través y eventos
Un canal está asociado a eventos y programas que le permiten controlar la publicación y el almacenamiento de segmentos en una transmisión en vivo. Los canales administran los eventos. 

Puede especificar la cantidad de horas que desea conservar el contenido grabado del programa en la configuración de la duración de **Ventana de archivo** . Este valor se puede establecer desde un mínimo de cinco minutos a un máximo de 25 horas. La duración de la ventana de archivo también indica el tiempo máximo que los clientes pueden buscar hacia atrás a partir de la posición en vivo actual. Los eventos pueden transmitirse durante la cantidad de tiempo especificada, pero el contenido que escape de esa longitud de ventana se descartará continuamente. El valor de esta propiedad también determina durante cuánto tiempo los manifiestos de cliente pueden crecer.

Cada evento está asociado a un recurso. Para publicar el evento, debe crear un localizador a petición para el recurso asociado. Contar con este localizador le permite crear una dirección URL de streaming que puede proporcionar a sus clientes.

Un canal es compatible con hasta tres eventos en ejecución simultánea, por lo que puede crear varios archivos de la misma transmisión entrante. Esto le permite publicar y archivar distintas partes de un evento, según sea necesario. Por ejemplo, el requisito de su empresa es solo archivar seis horas de un programa, pero difundir solo los últimos diez minutos. Para lograrlo, necesita crear dos programas en ejecución simultánea. Un programa está definido para archivar seis horas del evento, pero no está publicado. El otro programa está definido para archivar durante diez minutos y este programa sí se publica.

No debería reutilizar eventos en directo ya existentes. En su lugar, cree e inicie un evento nuevo para cada evento.

Inicie el evento cuando esté listo para iniciar el streaming y el archivo. Detenga el programa cuando quiera detener el streaming y el archivo del evento. 

Para eliminar contenido archivado, detenga y elimine el evento y, a continuación, elimine el recurso asociado. No se puede eliminar un recurso si lo está usando un evento; primero se debe eliminar el evento. 

Incluso después de detener y eliminar el evento, los usuarios podrán transmitir el contenido archivado como un vídeo a petición siempre que no elimine el recurso.

Si desea conservar el contenido archivado, pero no hacerlo disponible para la transmisión, elimine el localizador de streaming.

### <a name="to-use-the-portal-to-create-a-channel"></a>Uso del portal para crear un canal
Esta sección muestra cómo utilizar la opción **Creación rápida** para crear un canal de paso a través.

Para más información sobre este tipo de canales, consulte [Streaming en vivo con codificadores locales que crean transmisiones de velocidad de bits múltiple](media-services-live-streaming-with-onprem-encoders.md).

1. En [Azure Portal](https://portal.azure.com/), seleccione la cuenta de Azure Media Services.
2. En la ventana **Configuración**, haga clic en **Streaming en vivo**. 
   
    ![Introducción](./media/media-services-portal-passthrough-get-started/media-services-getting-started.png)
   
    Aparece la ventana **Streaming en vivo** .
3. Haga clic en **Creación rápida** para crear un canal de paso a través con el protocolo de introducción RTMP.
   
    Aparece la ventana **CREAR UN NUEVO CANAL** .
4. Asigne un nombre al nuevo canal y haga clic en **Crear**. 
   
    Con ello, se crea un canal de paso a través con el protocolo de introducción RTMP.

## <a name="create-events"></a>Creación de eventos
1. Seleccione el canal al que desea agregar un evento.
2. Pulse el botón **Evento en directo** .

![Evento](./media/media-services-portal-passthrough-get-started/media-services-create-events.png)

## <a name="get-ingest-urls"></a>Obtención de direcciones URL de introducción
Una vez creado el canal, obtendrá direcciones URL de introducción que se proporcionarán al codificador en directo. El codificador usa estas direcciones URL para introducir una secuencia en vivo.

![Creado](./media/media-services-portal-passthrough-get-started/media-services-channel-created.png)

## <a name="watch-the-event"></a>Visualización del evento
Para ver el evento, haga clic en **Inspección** en el Portal de Azure o copie la dirección URL de streaming y use el reproductor que prefiera. 

![Creado](./media/media-services-portal-passthrough-get-started/media-services-default-event.png)

El evento en directo se convierte automáticamente en contenido a petición cuando se detiene.

## <a name="clean-up"></a>Limpieza
Para más información sobre este tipo de canales, consulte [Streaming en vivo con codificadores locales que crean transmisiones de velocidad de bits múltiple](media-services-live-streaming-with-onprem-encoders.md).

* Un canal se puede detener solo cuando se hayan detenido todos los eventos o programas del canal.  Cuando se detiene el canal, no se incurre en ningún cargo. Cuando necesite iniciarlo de nuevo, tendrá la misma URL de introducción, por lo que no necesitará volver a configurar su codificador.
* Un canal se puede eliminar solo cuando se hayan eliminado todos los eventos en directo del canal.

## <a name="view-archived-content"></a>Visualización del contenido archivado
Incluso después de detener y eliminar el evento, los usuarios podrán transmitir el contenido archivado como un vídeo a petición siempre que no elimine el recurso. No se puede eliminar un recurso si lo está usando un evento; primero se debe eliminar el evento. 

Para administrar los recursos seleccione **Configuración** y haga clic en **Recursos**.

![Recursos](./media/media-services-portal-passthrough-get-started/media-services-assets.png)

## <a name="next-step"></a>Paso siguiente
Consulte las rutas de aprendizaje de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
