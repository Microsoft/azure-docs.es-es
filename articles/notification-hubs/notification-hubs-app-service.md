---
title: Integración con Notification Hubs con App Service Mobile Apps
description: Obtenga información sobre cómo funciona Azure Notification Hubs con Azure App Service Mobile Apps.
author: sethmanheim
manager: femila
services: notification-hubs
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: multiple
ms.devlang: multiple
ms.topic: overview
ms.custom: mvc
ms.date: 08/06/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: e7042be7e407e8e0827e142ba6878dfff812e1f6
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88004062"
---
# <a name="integration-with-app-service-mobile-apps"></a>Integración con App Service Mobile Apps

Para facilitar una experiencia perfecta y unificadora entre servicios de Azure, [App Service Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-value-prop) tiene compatibilidad integrada con notificaciones mediante Azure Notification Hubs. [App Service Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-value-prop) ofrece una plataforma de desarrollo de aplicaciones móviles altamente escalable y disponible globalmente para desarrolladores empresariales e integradores de sistemas que proporciona un amplio conjunto de funcionalidades a desarrolladores móviles.

Los desarrolladores de Mobile Apps pueden usar Notification Hubs con el siguiente flujo de trabajo:

1. Recuperar controlador PNS de dispositivo.
2. Registrar el dispositivo con Notification Hubs a través de las API del registro del SDK de cliente de Mobile Apps.

    > [!NOTE]
    > Tenga en cuenta que Mobile Apps elimina todas las etiquetas en los registros por motivos de seguridad. Trabaje con Notification Hubs desde su back-end directamente para asociar etiquetas a dispositivos.

3. Enviar notificaciones desde su back-end de aplicación con Notification Hubs.

Algunas de las ventajas que proporciona esta integración son:

- **SDK de cliente de Mobile Apps**: Estos SDK de multiplataforma ofrecen las API para el registro y se comunican con el centro de notificaciones vinculado con la aplicación móvil. No necesita las credenciales de Notification Hubs. ni trabajar con un servicio adicional.
  - *Inserción en usuario*: Los SDK etiquetan automáticamente el dispositivo específico con un identificador de usuario autenticado de Mobile Apps para habilitar la "inserción en un escenario de usuario".
  - *Inserción en dispositivo*: Los SDK utilizan automáticamente el identificador de instalación de Mobile Apps como GUID para registrarse con Notification Hubs, y de esta forma que no haya necesidad de mantener varios GUID de servicio.
- **Modelo de instalación**: Mobile Apps funciona con el modelo de inserción más reciente de Notification Hubs para representar todas las propiedades de inserción asociadas a un dispositivo en una instalación de JSON que se alinea con los servicios de notificaciones push y resulta fácil de usar.
- **Flexibilidad**: los desarrolladores siempre pueden elegir trabajar con Notification Hubs directamente, incluso con la integración implementada.
- **Experiencia integrada en [Azure Portal](https://portal.azure.com)** : La inserción como funcionalidad se representa visualmente en Mobile Apps y los desarrolladores pueden trabajar fácilmente con el centro de notificaciones asociado mediante Mobile Apps.
