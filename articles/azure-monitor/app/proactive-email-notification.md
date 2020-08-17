---
title: 'Cambio de notificación de detección inteligente: Azure Application Insights'
description: Cambie a los destinatarios de notificación predeterminados de la detección inteligente. La detección inteligente le permite supervisar los seguimientos de las aplicaciones con Azure Application Insights para detectar patrones poco habituales de telemetría de seguimiento.
ms.topic: conceptual
author: harelbr
ms.author: harelbr
ms.date: 03/13/2019
ms.reviewer: mbullwin
ms.openlocfilehash: 8e2bf4e451ebc3c9ebba2c01dae6703fc79aa606
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324138"
---
# <a name="smart-detection-e-mail-notification-change"></a>Cambio de las notificaciones por correo electrónico de detección inteligente

Según los comentarios de los clientes, el 1 de abril de 2019 vamos a cambiar los roles predeterminados de quienes reciben notificaciones por correo electrónico de detección inteligente.

## <a name="what-is-changing"></a>¿Qué está cambiando?

Actualmente, las notificaciones por correo electrónico de detección inteligente se envían de forma predeterminada a los roles de _propietario de la suscripción_, _colaborador de la suscripción_ y _lector de la suscripción_. Estos roles suelen incluir usuarios que no participan activamente en la supervisión, lo que hace que muchos de estos usuarios reciban notificaciones innecesariamente. Para mejorar esta experiencia, estamos haciendo un cambio para que las notificaciones por correo electrónico solo vayan, de manera predeterminada, a los roles de [lector de supervisión ](../../role-based-access-control/built-in-roles.md#monitoring-reader) y [colaborador de supervisión](../../role-based-access-control/built-in-roles.md#monitoring-contributor).

## <a name="scope-of-this-change"></a>Ámbito de este cambio

Este cambio afectará a todas las reglas de detección inteligente, excepto los siguientes:

* Las reglas de detección inteligente marcadas como versión preliminar. Estas reglas de detección inteligente no admiten notificaciones por correo electrónico en este momento.

* Reglas de anomalías de error. Esta regla comenzará a dirigir a los nuevos roles predeterminados una vez que se migre de una alerta clásica a la plataforma de alertas unificada (hay más información disponible [aquí](../platform/monitoring-classic-retirement.md)).

## <a name="how-to-prepare-for-this-change"></a>¿Cómo prepararse para este cambio?

Para garantizar que las notificaciones por correo electrónico de detección inteligente se envíen a los usuarios pertinentes, estos usuarios deben tener asignados los roles de [lector de supervisión](../../role-based-access-control/built-in-roles.md#monitoring-reader) y [colaborador de supervisión](../../role-based-access-control/built-in-roles.md#monitoring-contributor) de la suscripción.

Para asignar usuarios a los roles de lector de supervisión o de colaborador de supervisión mediante Azure Portal, siga los pasos descritos en el artículo [Adición de una asignación de roles](../../role-based-access-control/role-assignments-portal.md#add-a-role-assignment). Asegúrese de seleccionar _lector de supervisión_ o _colaborador de supervisión_ como el rol al que se asignan los usuarios.

> [!NOTE]
> Los destinatarios específicos de las notificaciones de detección inteligente, configurados mediante la opción _Destinatarios de correo electrónico adicionales_ en la configuración de la regla, no se verán afectados por este cambio. Estos destinatarios seguirá recibiendo las notificaciones por correo electrónico.

Si tiene alguna pregunta o duda acerca de este cambio, no dude en [ponerse en contacto con nosotros](mailto:smart-alert-feedback@microsoft.com).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la detección inteligente:

- [Anomalías de error](./proactive-failure-diagnostics.md)
- [Fugas de memoria](./proactive-potential-memory-leak.md)
- [Anomalías de rendimiento](./proactive-performance-diagnostics.md)

