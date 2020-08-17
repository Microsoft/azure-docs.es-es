---
title: Configuración de Google Firebase Cloud Messaging en Azure Notification Hubs | Microsoft Docs
description: Aprenda a configurar un centro de notificaciones de Azure con la configuración de Google Firebase Cloud Messaging.
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.topic: article
ms.date: 08/04/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 03/25/2019
ms.openlocfilehash: 0da103c11e2412108535ca322917632f5d95559d
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760845"
---
# <a name="configure-google-firebase-settings-for-a-notification-hub-in-the-azure-portal"></a>Configuración de valores de Google Firebase para un centro de notificaciones en Azure Portal

En este artículo se muestra cómo establecer la configuración de Google Firebase Cloud Messaging (FCM) para un centro de notificaciones de Azure mediante Azure Portal.  

## <a name="prerequisites"></a>Prerrequisitos

Si aún no ha creado un centro de notificaciones, cree uno ahora. Para más información, consulte [Creación de un centro de notificaciones de Azure en Azure Portal](create-notification-hub-portal.md).

## <a name="configure-google-firebase-cloud-messaging-fcm"></a>Configuración de Google Firebase Cloud Messaging (FCM)

En el siguiente procedimiento se describen los pasos para establecer la configuración de Google Firebase Cloud Messaging (FCM) para un centro de notificaciones:

1. En Azure Portal, en la página **Centro de notificaciones**, seleccione **Google (GCM/FCM)** en el menú izquierdo.
2. Pegue la **clave de API** para el proyecto FCM que ha guardado anteriormente.
3. Seleccione **Guardar**.

   ![Captura de pantalla que muestra cómo configurar Notification Hubs para Google FCM](./media/notification-hubs-android-push-notification-google-fcm-get-started/fcm-server-key.png)

## <a name="next-steps"></a>Pasos siguientes

Para ver un tutorial con instrucciones paso a paso para enviar notificaciones en dispositivos Android mediante Azure Notification Hubs y Google Firebase Cloud Messaging, consulte [Tutorial: Envío de notificaciones push a dispositivos Android con Azure Notification Hubs y Google Firebase Cloud Messaging](notification-hubs-android-push-notification-google-fcm-get-started.md).
