---
title: Cambio del plan de tarifa del espacio de nombres de Notification Hubs | Microsoft Docs
description: Aprenda a cambiar el plan de tarifa de un espacio de nombres de Azure Notification Hubs.
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 08/03/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 01/28/2019
ms.openlocfilehash: 1455259bc42aea9d506a9a2a19d725cac3d643f8
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87562776"
---
# <a name="change-pricing-tier-of-an-azure-notification-hubs-namespace"></a>Cambio del plan de tarifa de un espacio de nombres de Azure Notification Hubs

Notification Hubs se ofrece en tres niveles: **gratis**, **básico** y **estándar**. En este artículo se muestra cómo cambiar el plan de tarifa para un espacio de nombres de Azure Notification Hubs.

## <a name="overview"></a>Información general

En Azure Notification Hubs, un *centro de notificaciones* es la entidad o recurso más pequeño. Por lo general, se asigna a una aplicación y puede contener un certificado para cada Sistema de notificación de plataforma (PNS) que se admite en la aplicación. La aplicación puede ser híbrida, nativa o multiplataforma.

Un *espacio de nombres* es una colección de centros de notificaciones. Por lo general, cada espacio de nombres consta de centros relacionados y que se usan con un propósito específico. Por ejemplo, puede tener tres espacios de nombres distintos con fines de desarrollo, prueba y producción, respectivamente.

Puede asociar un espacio de nombres con los planes de tarifa **gratis**, **básico** o **estándar**. Para los espacios de nombres, puede usar el nivel que se adapte a sus requisitos. Las secciones siguientes muestran cómo cambiar el plan de tarifa de un espacio de nombres de Notification Hubs.

## <a name="use-azure-portal"></a>Usar Azure Portal

Al usar Azure Portal, puede cambiar el plan de tarifa de un espacio de nombres en la página del espacio de nombres o en la página de un centro. Cuando lo cambie en la página de un centro, realmente lo cambia en el nivel de espacio de nombres. Cambia el plan de tarifa para el espacio de nombres y todos los centros del espacio de nombres.

### <a name="change-tier-on-the-namespace-page"></a>Cambio del nivel en la página del espacio de nombres

El procedimiento siguiente le muestra cómo cambiar el plan de tarifa de un espacio de nombres en la página del espacio de nombres. Al cambiar el nivel de un espacio de nombres, se aplica a todos los centros de ese espacio de nombres.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** en el menú de la izquierda.
3. Seleccione **Espacios de nombres del centro de notificaciones** en la sección **Internet de las cosas**. Si selecciona el asterisco (`*`) que aparece junto al texto, se agrega en la barra de navegación de la izquierda bajo **FAVORITOS**. Le ayuda a tener acceso a la página del espacio de nombres con mayor rapidez la próxima vez. Una vez que lo agregue a **FAVORITOS**, seleccione **Espacios de nombre del centro de notificaciones**.

    ![Todos los servicios -> Espacios de nombre del centro de notificaciones](./media/change-pricing-tier/all-services-nhub.png)

4. En la página **Espacios de nombres del centro de notificaciones**, seleccione el espacio de nombres para el que quiere cambiar el plan de tarifa.
5. En la página **Espacio de nombres del centro de notificaciones** correspondiente al espacio de nombres, puede ver el plan de tarifa actual para el espacio de nombres de la sección **Essentials**. En la imagen siguiente, puede ver que el plan de tarifa del espacio de nombres es **Gratis**.

    ![Plan de tarifa actual en la página del espacio de nombres](./media/change-pricing-tier/pricing-tier-before.png)

6. En la página **Espacio de nombres del centro de notificaciones** correspondiente a su espacio de nombres, seleccione **Plan de tarifa** en la sección **Administrar**.

    ![Selección del plan de tarifa en la página del espacio de nombres](./media/change-pricing-tier/namespace-select-pricing-menu.png)

7. Cambie el plan de tarifa y, a continuación, haga clic en el botón **Seleccionar**.
8. Puede ver el estado de la acción de cambio del plan en las **alertas**.
9. Cámbiese a la página **Información general**. Confirme que se muestra el nivel nuevo para el campo **Plan de tarifa** en la sección **Essentials**.
10. Este paso es opcional. Seleccione cualquier centro del espacio de nombres. Confirme que ve el mismo plan de tarifa en la sección **Essentials**. Debería ver el mismo plan de tarifa para todos los centros del espacio de nombres.

### <a name="change-tier-on-the-hub-page"></a>Cambio del nivel en la página del centro

Siga estos pasos para cambiar el plan de tarifa de un espacio de nombres en la página del centro. Aunque lleve a cabo estos pasos desde la página del centro, en realidad cambia el plan de tarifa del espacio de nombres y todos los centros en el espacio de nombres:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** en el menú de la izquierda.
3. Seleccione **Notification Hubs** en la sección **Internet de las cosas**.
4. Seleccione su **centro** de notificaciones.
5. Seleccione **Plan de tarifa** en el menú de la izquierda.
6. Cambie el plan de tarifa y haga clic en el botón **Seleccionar**. Esta acción cambia la configuración del plan de tarifa para el espacio de nombres que contiene el centro. De este modo, verá el plan de tarifa nuevo en la página del espacio de nombres y en todas las páginas de los centros.

> [!NOTE]
> Todos los cambios del plan de tarifa se aplican de inmediato.

## <a name="use-rest-api"></a>Use la API de REST

Puede usar las API REST de proveedor de recursos siguientes para obtener el plan de tarifa actual y actualizarlo.

### <a name="get-current-pricing-tier-for-a-namespace"></a>Obtención del plan de tarifa actual para un espacio de nombres

Para obtener el plan del espacio de nombres actual, envíe un comando GET como se muestra en el ejemplo siguiente:

```REST
GET: https://management.core.windows.net/{subscription ID}/services/ServiceBus/Namespaces/{namespace name}/notificationhubplan
```

### <a name="update-pricing-tier-for-a-namespace"></a>Actualización del plan de tarifa de un espacio de nombres

Para actualizar el plan del espacio de nombres, envíe un comando PUT como se muestra en el ejemplo siguiente:

```REST
PUT: https://management.core.windows.net/{subscription ID}/services/ServiceBus/Namespaces/{namespace name}/notificationhubplan
Body: <NotificationHubPlan xmlns:i="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/netservices/2010/10/servicebus/connect"><SKU>Standard</SKU></NotificationHubPlan>
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre estos niveles y precios, consulte [Precios de Notification Hubs](https://azure.microsoft.com/pricing/details/notification-hubs/).
