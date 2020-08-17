---
title: Configuración de Servicios de notificaciones de inserción de Windows en Azure Notification Hubs | Microsoft Docs
description: Aprenda a configurar Servicios de notificaciones de inserción de Windows para un centro de notificaciones de Azure.
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
ms.openlocfilehash: 62d73a23fff3bd15d873a579d451b2f91509b7fe
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87758737"
---
# <a name="configure-windows-push-notification-service-settings-in-the-azure-portal"></a>Configuración de los valores de configuración del Servicio de notificaciones push de Windows en Azure Portal

En este artículo podrá ver cómo establecer la configuración del Servicio de notificaciones de Windows (WNS) para un centro de notificaciones de Azure mediante Azure Portal.  

## <a name="prerequisites"></a>Prerrequisitos

Si aún no ha creado un centro de notificaciones, cree uno ahora. Para más información, consulte [Creación de un centro de notificaciones de Azure en Azure Portal](create-notification-hub-portal.md).

## <a name="configure-windows-push-notification-service-wns"></a>Configuración de Servicios de notificaciones de inserción de Windows (WNS)

En el siguiente procedimiento se describen los pasos para establecer la configuración de Servicios de notificaciones de inserción de Windows (WNS) para un centro de notificaciones:

1. En Azure Portal, en la página **Centro de notificaciones**, seleccione **Windows (WNS)** en el menú izquierdo.
2. Especifique los valores de **SID de paquete** y **Clave de seguridad**.
3. Seleccione **Guardar**.

   ![Captura de pantalla que muestra los cuadros de SID del paquete y Clave de seguridad](./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-configure-wns.png)

## <a name="next-steps"></a>Pasos siguientes

Para ver un tutorial con instrucciones paso a paso sobre cómo enviar notificaciones de inserción en aplicaciones de Plataforma universal de Windows mediante Azure Notification Hubs y Servicios de notificaciones de inserción de Windows (WNS), consulte [Envío de notificaciones a aplicaciones de Plataforma universal de Windows mediante Azure Notification Hubs](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).
