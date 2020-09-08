---
title: Recuperación de la lista de direcciones IP POP actual para Azure CDN| Microsoft Docs
description: Obtenga información sobre cómo obtener servidores POP mediante la API REST. Los servidores POP realizan solicitudes a los servidores de origen asociados a los puntos de conexión de Azure Content Delivery Network.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/22/2019
ms.author: allensu
ms.custom: ''
ms.openlocfilehash: 13d4587a80cbbe66e18eb1bec8f3154221ac62b5
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192562"
---
# <a name="retrieve-the-current-pop-ip-list-for-azure-cdn"></a>Recuperación de la lista de direcciones IP POP para Azure CDN

## <a name="retrieve-the-current-verizon-pop-ip-list-for-azure-cdn"></a>Recuperación de la lista de direcciones IP POP de Verizon para Azure CDN

Puede utilizar la API REST para recuperar el conjunto de direcciones IP de los servidores de punto de presencia (POP) de Verizon. Estos servidores de POP realizan solicitudes a los servidores de origen que están asociados a los puntos de conexión de Azure Content Delivery Network (CDN) en un perfil de Verizon (**Azure CDN de Verizon estándar** o **Azure CDN de Verizon premium**). Tenga en cuenta que este conjunto de direcciones IP es diferente de las direcciones IP que un cliente vería al realizar solicitudes a los POP. 

Para obtener la sintaxis de la operación de API REST para recuperar la lista de POP, consulte [Edge Nodes - List](https://docs.microsoft.com/rest/api/cdn/edgenodes/list) (Nodos de Edge: lista).

## <a name="retrieve-the-current-microsoft-pop-ip-list-for-azure-cdn"></a>Recuperación de la lista de direcciones IP POP actual de Microsoft para Azure CDN

Para bloquear la aplicación y que solo acepte el tráfico de Azure CDN de Microsoft, deberá configurar las ACL de IP para el back-end. También puede restringir el conjunto de valores aceptados para el encabezado "X-Forwarded-Host" enviado por Azure CDN de Microsoft. Estos pasos se detallan a continuación:

Configure la creación de listas de control de acceso de IP de los back-end para que acepten tráfico de Azure CDN solo del espacio de direcciones IP de back-end de Microsoft y de los servicios de infraestructura de Azure. 

* Azure CDN del espacio IP de back-end IPv4 de Microsoft: 147.243.0.0/16
* Azure CDN del espacio IP de back-end IPv6 de Microsoft: 2a01:111:2050::/44

Para usar etiquetas de servicio con Azure CDN de Microsoft, use la etiqueta de Azure Front Door. Los intervalos IP y las etiquetas de servicio de los servicios de Microsoft se pueden encontrar [aquí](https://www.microsoft.com/download/details.aspx?id=56519).


## <a name="typical-use-case"></a>Caso de uso típico

Por motivos de seguridad, puede usar esta lista de direcciones IP para exigir que las solicitudes a su servidor de origen se realicen solo desde un POP de Verizon válido. Por ejemplo, si alguien detectase el nombre de host o la dirección IP de un servidor de origen del punto de conexión de CDN, se podrían realizar las solicitudes directamente al servidor de origen, por lo que se omitirían las funcionalidades de seguridad y escalado proporcionadas por Azure CDN. El establecimiento de las direcciones IP en la lista devuelta como las únicas permitidas en un servidor de origen puede evitar esta situación. Para garantizar que cuenta con la lista de POP más reciente, recupérela al menos una vez al día. 

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de la API REST, consulte [Azure CDN REST API](https://docs.microsoft.com/rest/api/cdn/) (API REST de Azure CDN).
