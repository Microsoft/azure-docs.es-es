---
title: Uso de cambio de tiempo y salidas activas para crear la reproducción de vídeo a petición
titleSuffix: Azure Media Services
description: En este artículo se describe cómo usar el cambio de tiempo y las salidas activas para grabar streaming en vivo y crear una reproducción a petición.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 05/07/2020
ms.author: juliako
ms.openlocfilehash: 8c5afe45ce864ba76d5d637df3534d426d39167a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87000999"
---
# <a name="use-time-shifting-and-live-outputs-to-create-on-demand-video-playback"></a>Uso de cambio de tiempo y salidas activas para crear la reproducción de vídeo a petición

En Azure Media Services, un objeto [Live Output](/rest/api/media/liveoutputs) es como una grabadora de vídeo digital que capta y graba el streaming en vivo en un recurso de su cuenta de Media Services. El contenido grabado se conserva en el contenedor definido por el recurso [Asset](/rest/api/media/assets) (el contenedor está en la cuenta de Azure Storage asociada a su cuenta). La salida activa también le permite controlar algunas propiedades del streaming en vivo saliente, como la cantidad de la transmisión que se conserva en la grabación del archivo (por ejemplo, la capacidad de la DVR en la nube) o cuándo los espectadores pueden empezar a ver el streaming en vivo. El archivo en disco es una "ventana" circular de archivo que solo incluye la cantidad de contenido que se especifica en la propiedad **archiveWindowLength** de la salida activa. El contenido que está fuera de esta ventana se descarta automáticamente del contenedor de almacenamiento y no se puede recuperar. El valor archiveWindowLength representa una duración de intervalo de tiempo de ISO-8601 (por ejemplo, PTHH:MM:SS), que especifica la capacidad de la DVR. Este valor se puede establecer en 1 minuto como mínimo y 25 horas como máximo.

La relación entre un evento en directo y sus salidas activas es similar a la retransmisión de televisión tradicional, en la que un canal (evento en directo) representa una transmisión constante de vídeo y una grabación (salida activa) está limitada a un segmento de tiempo específico (por ejemplo, las noticias de la tarde, que se emiten de 18:30 a 19:00). Una vez que la transmisión fluye en el evento en directo, puede comenzar el evento de streaming mediante la creación de un recurso, salida activa, y un localizador de streaming. El objeto LiveOutput archivará la secuencia y la pondrá a disposición de los usuarios a través del [punto de conexión de streaming](/rest/api/media/streamingendpoints). Puede crear varios objetos LiveOutput (hasta un máximo de tres) en un objeto LiveEvent con diferentes longitudes y configuraciones de archivo. Para obtener información sobre el flujo de trabajo del streaming en vivo, consulte la sección de [pasos generales](live-streaming-overview.md#general-steps).

## <a name="using-a-dvr-during-an-event"></a>Uso de una DVR durante un evento

En esta sección se explica cómo usar una DVR durante un evento para controlar qué partes de la transmisión están disponibles para el "rebobinado".

El valor `archiveWindowLength` determina cuánto puede retroceder en el tiempo un espectador desde la posición actual en vivo. El valor `archiveWindowLength` también determina durante cuánto tiempo pueden aumentar los manifiestos de cliente.

Supongamos que está transmitiendo un partido de fútbol y que el valor de `ArchiveWindowLength` son solo 30 minutos. Un usuario que empieza a ver el evento a los 45 minutos de iniciado el juego puede buscar hacia atrás hasta la marca de 15 minutos como máximo. Las salidas activas del partido continuarán hasta que se detenga el evento en directo. El contenido que se encuentra fuera de archiveWindowLength se descarta continuamente del almacenamiento y no es recuperable. En este ejemplo, el vídeo entre el inicio del evento y la marca de 15 minutos se habría purgado de la DVR y del contenedor de Blob Storage del recurso. El archivo no se puede recuperar y se quita del contenedor de Azure Blob Storage.

Un evento en directo admite la ejecución simultánea de hasta tres salidas activas (puede crear tres grabaciones o archivos como máximo a partir de un streaming en vivo al mismo tiempo). Esto le permite publicar y archivar distintas partes de un evento, según sea necesario. Suponga que tiene que difundir una fuente lineal en vivo e ininterrumpida y crea "grabaciones" de los distintos programas durante todo el día para ofrecerlas a los clientes como contenido a petición que pueden ver para ponerse al día. En este escenario, primero crea una salida activa principal con una ventana de archivado de una hora o menos (este es el streaming en vivo principal que los espectadores sintonizarán). Puede crear un localizador de streaming para esta salida activa y publicarlo en la aplicación o en el sitio web como la fuente "en directo". Mientras se ejecuta el evento en directo, puede crear mediante programación una segunda salida en directo al comienzo de un programa (o cinco minutos antes para ofrecer algunos manipuladores que permitan recortar la transmisión más adelante). Esta segunda salida en directo puede eliminarse cinco minutos después de que finalice el programa. Con este segundo recurso, puede crear un localizador de streaming para publicar este programa como un recurso a petición en el catálogo de la aplicación. Este proceso se puede repetir varias veces en otros límites o aspectos destacados del programa que desee compartir como vídeos a petición, y todo ello mientras la fuente "en directo" de la primera salida en directo continúa con la difusión de la fuente lineal.

## <a name="creating-an-archive-for-on-demand-playback"></a>Creación de un archivo para la reproducción a petición

El recurso en el que se archiva la salida activa se convierte automáticamente en un recurso a petición cuando se elimina dicha salida. Debe eliminar todas las salidas activas para que un evento en directo pueda detenerse. Puede usar una marca opcional [removeOutputsOnStop](/rest/api/media/liveevents/stop#request-body) para quitar automáticamente las salidas activas cuando se detengan.

Incluso después de detener y eliminar el evento, los usuarios pueden transmitir el contenido archivado como un vídeo a petición, siempre que no se elimine el recurso. No se puede eliminar un recurso si lo está usando un evento. Primero se debe eliminar el evento.

Si ha publicado el recurso de la salida activa mediante un localizador de streaming, el evento en directo (hasta la longitud de la ventana de la DVR) seguirá estando visible hasta que el localizador de streaming expire o se elimine, lo que ocurra primero.

Para más información, consulte:

- [Introducción al streaming en vivo](live-streaming-overview.md)
- [Tutorial de Live Streaming](stream-live-tutorial-with-api.md)

> [!NOTE]
> Cuando se elimina la salida activa, no se eliminan el recurso subyacente ni el contenido del recurso.

## <a name="next-steps"></a>Pasos siguientes

* [Creación de subclips de los vídeos](subclip-video-rest-howto.md).
* [Definición de filtros para los recursos](filters-dynamic-manifest-rest-howto.md).
