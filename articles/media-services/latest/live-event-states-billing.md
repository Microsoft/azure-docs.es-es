---
title: Estados y facturación de LiveEvent en Azure Media Services | Microsoft Docs
description: En este tema se proporciona información general sobre los estados y facturación de LiveEvent en Azure Media Services.
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
ms.date: 10/24/2019
ms.author: juliako
ms.openlocfilehash: dd63152776e0268b8f9e3e0ce116fea897cdf4e2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87000880"
---
# <a name="live-event-states-and-billing"></a>Estados y facturación de LiveEvent

En Azure Media Services, un objeto LiveEvent comienza la facturación tan pronto como su estado realiza la transición a **En ejecución**. Se le facturará incluso si no hay ningún flujo de vídeo a través del servicio. Para detener la facturación del objeto LiveEvent, debe pararlo. El objeto LiveTranscription se factura igual que el objeto LiveEvent.

Si **LiveEventEncodingType** en el objeto [LiveEvent](/rest/api/media/liveevents) está establecido en Estándar o Premium1080p, Media Services cerrará automáticamente cualquier objeto LiveEvent que tenga aún el estado **En ejecución** doce horas después de que se pierda la fuente de entrada y si no hay ningún objeto **LiveOutput** en ejecución. Sin embargo, se le facturará por el tiempo que el objeto LiveEvent haya estado en estado **En ejecución**.

> [!NOTE]
> Los eventos en directo de paso a través no se cierran automáticamente y se deben detener explícitamente a través de la API para evitar una facturación excesiva. 

## <a name="states"></a>States

LiveEvent puede estar en uno de los siguientes estados.

|State|Descripción|
|---|---|
|**Stopped**| Este es el estado inicial del objeto LiveEvent después de su creación (a menos que haya establecido el inicio automático en true). No se produce ninguna facturación en este estado. En este estado, se pueden actualizar las propiedades del objeto LiveEvent pero no se permite el streaming.|
|**Starting** (iniciándose)| Se inicia LiveEvent y se asignan los recursos. No se produce ninguna facturación en este estado. Durante este estado no se permite realizar actualizaciones ni streaming. Si se produce un error, el objeto LiveEvent vuelve al estado Detenido.|
|**Ejecución**| Se han asignado los recursos de LiveEvent, se han generado las direcciones URL de ingesta y visualización previa y puede recibir transmisiones en vivo. En este momento, la facturación está activa. Debe llamar explícitamente a Stop en el recurso del objeto LiveEvent para evitar que continúe la facturación.|
|**Stopping** (Deteniéndose)| Se detiene LiveEvent y se desaprovisionan los recursos. No se produce facturación en este estado transitorio. Durante este estado no se permite realizar actualizaciones ni streaming.|
|**Eliminando**| El objeto LiveEvent se está eliminando. No se produce facturación en este estado transitorio. Durante este estado no se permite realizar actualizaciones ni streaming.|

Al crear el objeto LiveEvent puede elegir habilitar LiveTranscriptions. Si lo hace, se le facturará el objeto LiveTranscriptions cuando el objeto LiveEvent se encuentre en estado **En ejecución**. Tenga en cuenta que se le facturará aunque no haya ningún flujo de audio a través del objeto LiveEvent.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción al streaming en vivo](live-streaming-overview.md)
- [Tutorial de Live Streaming](stream-live-tutorial-with-api.md)
