---
title: 'Mantenimiento programado en Azure Database for PostgreSQL: Servidor flexible'
description: 'En este artículo se describe la característica de mantenimiento programado de Azure Database for PostgreSQL: Servidor flexible.'
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 35e5e688f5589c23c7c583c7bcbca3fd0956eec7
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931892"
---
# <a name="scheduled-maintenance-in-azure-database-for-postgresql--flexible-server"></a>Mantenimiento programado en Azure Database for PostgreSQL: Servidor flexible
 
Azure Database for PostgreSQL: Servidor flexible realiza un mantenimiento periódico para conservar la base de datos administrada protegida, estable y actualizada. Durante el mantenimiento, el servidor obtiene características, actualizaciones y revisiones nuevas.
 
> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible está en versión preliminar
 
## <a name="selecting-a-maintenance-window"></a>Selección de una ventana de mantenimiento
 
Puede programar el mantenimiento durante un día concreto de la semana y una ventana de tiempo en ese día. También puede dejar que el sistema elija un día y una hora de esa ventana de tiempo automáticamente. En cualquier caso, el sistema avisa cinco días antes de ejecutar cualquier mantenimiento. El sistema también comunica cuándo se ha iniciado el mantenimiento y cuándo ha finalizado correctamente.
 
Las notificaciones sobre el próximo mantenimiento programado se pueden:
 
* Enviar por correo electrónico a una dirección concreta
* Enviar por correo electrónico a un rol de Azure Resource Manager
* Enviar en un mensaje de texto (SMS) a dispositivos móviles
* Insertar como una notificación en una aplicación de Azure
* Entregar como un mensaje de voz
 
A la hora de especificar las preferencias de la programación de mantenimiento, puede elegir un día de la semana y una ventana de tiempo. Si no lo hace, el sistema elige horas entre las 23:00 y las 7:00 de la hora de la región del servidor. Puede definir diferentes programaciones para cada servidor flexible de la suscripción de Azure. 
 
> [!IMPORTANT]
> Normalmente, hay al menos 30 días entre los eventos de mantenimiento programado correctos de un servidor.
>
> No obstante, en el caso de una actualización de emergencia crítica, como una vulnerabilidad grave, la ventana de notificación podría ser inferior a cinco días. La actualización crítica se puede aplicar al servidor aunque se haya realizado un mantenimiento programado correcto durante los últimos 30 días.

Se puede actualizar la configuración de programación en cualquier momento. Si hay un mantenimiento programado para el Servidor flexible y se actualizan las preferencias de programación, el evento actual continúa según lo programado y el cambio de configuración de programación se aplica cuando finaliza correctamente. 

Si el sistema cancela el evento de mantenimiento o no se completa correctamente, creará una notificación sobre el evento de mantenimiento cancelado o con errores, respectivamente. El siguiente intento de mantenimiento se programa según la configuración de programación actual y se recibirá una notificación al respecto con cinco días de antelación.
 
## <a name="next-steps"></a>Pasos siguientes
 
* Obtenga información sobre cómo [cambiar la programación de mantenimiento](how-to-maintenance-portal.md).
* Obtenga información sobre cómo [obtener notificaciones sobre próximos eventos de mantenimiento](/azure/service-health/service-notifications.md) con Azure Service Health.
* Obtenga información sobre cómo [configurar alertas sobre próximos eventos de mantenimiento programado](/azure/service-health/resource-health-alert-monitor-guide.md).
