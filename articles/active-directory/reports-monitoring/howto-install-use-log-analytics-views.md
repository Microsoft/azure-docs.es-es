---
title: Instalación y uso de las vistas de análisis de registros | Microsoft Docs
description: Aprenda a instalar y usar las vistas de Log Analytics para Azure Active Directory.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 2290de3c-2858-4da0-b4ca-a00107702e26
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 56eee39a9e99e9d6752d1b4f6eb7182043ff14d6
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89230592"
---
# <a name="install-and-use-the-log-analytics-views-for-azure-active-directory"></a>Instalación y uso de las vistas de Log Analytics para Azure Active Directory

Las vistas de Log Analytics para Azure Active Directory le ayudan a analizar y a buscar los registros de actividad de Azure AD en su inquilino de Azure AD. Los registros de actividad de Azure AD incluyen:

* Registros de auditoría: el [informe de actividad de registros de auditoría](concept-audit-logs.md) le proporciona acceso al historial de todas las tareas llevadas a cabo en el inquilino.
* Registros de inicio de sesión: Con el [informe de actividad de inicios de sesión](concept-sign-ins.md), puede determinar quién ha realizado las tareas notificadas en el informe de registros de auditoría.

## <a name="prerequisites"></a>Prerrequisitos

Para usar las vistas de Log Analytics, necesita:

* Un área de trabajo de Log Analytics en la suscripción a Azure. Aprenda a [crear un área de trabajo de Log Analytics](../../azure-monitor/learn/quick-create-workspace.md).
* Primero, complete los pasos para [enrutar los registros de actividad de Azure AD al área de trabajo de Log Analytics](howto-integrate-activity-logs-with-log-analytics.md).
* Descargue las vistas del [repositorio de GitHub](https://aka.ms/AADLogAnalyticsviews) al equipo local.

## <a name="install-the-log-analytics-views"></a>Instalación de las vistas de Log Analytics

1. Vaya al área de trabajo de Log Analytics. Para ello, vaya primero a [Azure Portal](https://portal.azure.com) y seleccione **Todos los servicios**. Escriba **Log Analytics** en el cuadro de texto y seleccione **Áreas de trabajo de Log Analytics**. Seleccione el área de trabajo a la que se han enviado los registros de actividad, como parte de los requisitos previos.
2. Seleccione **View Designer** (Diseñador de vistas), seleccione **Import** (Importar) y, después, seleccione **Choose File** (Elegir archivo) para importar las vistas desde el equipo local.
3. Seleccione las vistas que ha descargado de los requisitos previos y seleccione **Save** (Guardar) para guardar la importación. Realice este paso en la vista **Azure AD Account Provisioning Events** (Eventos de aprovisionamiento de la cuenta de AD) y la vista **Sign-ins Events** (Eventos de inicio de sesión).

## <a name="use-the-views"></a>Uso de las vistas

1. Vaya al área de trabajo de Log Analytics. Para ello, vaya primero a [Azure Portal](https://portal.azure.com) y seleccione **Todos los servicios**. Escriba **Log Analytics** en el cuadro de texto y seleccione **Áreas de trabajo de Log Analytics**. Seleccione el área de trabajo a la que se han enviado los registros de actividad, como parte de los requisitos previos.

2. Cuando se encuentre en el área de trabajo, seleccione **Workspace Summary** (Resumen del área de trabajo). Debería ver las tres vistas siguientes:

    * **Azure AD Account Provisioning Events** (Eventos de aprovisionamiento de la cuenta de Azure AD): Esta vista muestra informes relacionados con la auditoría de la actividad de aprovisionamiento, como el número de nuevos usuarios aprovisionados y errores de aprovisionamiento, el número de usuarios actualizados y errores de actualización y el número de usuarios desaprovisionados y los errores correspondientes.    
    * **Sign-ins Events** (Eventos de inicio de sesión): Esta vista muestra los informes más importantes relacionados con la supervisión de la actividad de inicio de sesión, como los inicios de sesión por aplicación, usuario y dispositivo, así como una vista resumida que muestra el número de inicios de sesión a lo largo del tiempo.

3. Seleccione cualquiera de estas vistas para pasar a los informes individuales. También puede establecer alertas sobre cualquiera de los parámetros del informe. Por ejemplo, vamos a establecer una alerta para cada vez que se produzca un error de inicio de sesión. Para ello, seleccione primero la vista **Sign-ins Events** (Eventos de inicio de sesión), seleccione **Sign-in errors over time** (Errores de inicio de sesión a lo largo del tiempo) y, a continuación, seleccione **Analytics** (Análisis) para abrir la página de detalles, con la consulta real detrás del informe. 

    ![Detalles](./media/howto-install-use-log-analytics-views/details.png)


4. Seleccione **Set alert** (Establecer alerta) y, después, seleccione la lógica **Whenever the Custom log search is &lt; (Siempre que la búsqueda de registros personalizados sea) sin definir&gt;** en la sección **Alert criteria** (Criterios de alerta). Como queremos emitir una alerta cuando haya un error de inicio de sesión, establezca el **umbral** de la lógica de alerta predeterminada en **1** y, a continuación, seleccione **Hecho**. 

    ![Configuración de la lógica de señal](./media/howto-install-use-log-analytics-views/configure-signal-logic.png)

5. Escriba un nombre y una descripción para la alerta y establezca la gravedad en **Advertencia**.

    ![Creación de una regla](./media/howto-install-use-log-analytics-views/create-rule.png)

6. Seleccione el grupo de acciones al que desea alertar. En general, puede tratarse de un equipo al que desea notificar por correo electrónico o mensaje de texto, o puede ser una tarea automatizada que utiliza webhooks, runbooks, funciones, aplicaciones lógicas o soluciones ITSM externas. Aprenda a [crear y administrar grupos de acciones en Azure Portal](../../azure-monitor/platform/action-groups.md).

7. Seleccione **Crear regla de alerta** para crear la alerta. Ahora se le notificará cada vez que haya un error de inicio de sesión.

## <a name="next-steps"></a>Pasos siguientes

* [Análisis de registros de actividad con registros de Azure Monitor](howto-analyze-activity-logs-log-analytics.md)
* [Introducción a los registros de Azure Monitor en Azure Portal](../../azure-monitor/log-query/get-started-portal.md)