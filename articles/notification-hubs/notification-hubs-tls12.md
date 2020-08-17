---
title: Actualizaciones de TLS para Notification Hubs
description: Más información sobre la compatibilidad con TLS en Azure Notification Hubs.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
ms.date: 04/29/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 01/28/2020
ms.openlocfilehash: 87a3627d7820f9f456ac08e2f20b70af961f817e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084252"
---
# <a name="transport-layer-security-tls"></a>Seguridad de la capa de transporte (TLS)

Para garantizar un mayor nivel de seguridad, Notification Hubs deshabilitará la compatibilidad con las versiones 1.0 y 1.1 de TLS el **31 de diciembre de 2020** (se ha ampliado desde el 30 de abril de 2020). Estos protocolos antiguos proporcionan una criptografía no segura y son vulnerables a ataques de BEAST y POODLE. Este cambio no afecta a las aplicaciones implementadas en dispositivos que ejecutan Android versión 5 o posterior, o iOS versión 5 o superior, ya que esos sistemas operativos admiten TLS 1.2 y el cliente y el servidor negocian la versión más reciente del protocolo admitida mutuamente al conectarse.

Se recomienda revisar todas las aplicaciones que usan Azure Notification Hubs para asegurarse de que usan las bibliotecas y pilas de TLS más adecuadas que admiten TLS 1.2.

## <a name="update-apps"></a>Actualización de aplicaciones

Puede asegurarse de que las aplicaciones de iOS usan TLS 1.2 mediante la característica de seguridad de red de Apple denominada App Transport Security (ATS). ATS no se puede usar para los SDK anteriores a iOS 9.0 o macOS 10.11, y puede leer más acerca de esta característica en la [documentación de Apple](https://developer.apple.com/documentation/security/preventing_insecure_network_connections).

En el caso de las aplicaciones de Android que usan instancias de SSLSocket, establezca protocolos habilitados en cada instancia de SSLSocket como se indica en [setEnabledProtocols](https://developer.android.com/reference/javax/net/ssl/SSLSocket#setEnabledProtocols(java.lang.String%5B%5D)).

La tabla de la página de soporte sobre [compatibilidad con el protocolo TLS](https://support.globalsign.com/customer/portal/articles/2934392-tls-protocol-compatibility) ayuda a asignar sistemas operativos con versiones compatibles de TLS.

Para más información, consulte la información general sobre la [compatibilidad con los protocolos TLS en Windows](/archive/blogs/kaushal/support-for-ssltls-protocols-on-windows).

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a Notification Hubs](notification-hubs-push-notification-overview.md)
