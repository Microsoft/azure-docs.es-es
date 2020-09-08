---
title: Creación de informes acerca del aprovisionamiento automático de cuentas de usuario para aplicaciones SaaS
description: Aprenda a comprobar el estado de los trabajos de aprovisionamiento automático de cuentas de usuario y a solucionar problemas del aprovisionamiento de usuarios individuales.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: how-to
ms.date: 09/09/2018
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: 2de6c07395a559085db237eb1bc7f885998860d0
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88235084"
---
# <a name="tutorial-reporting-on-automatic-user-account-provisioning"></a>Tutorial: Creación de informes sobre el aprovisionamiento automático de cuentas de usuario

Azure Active Directory (Azure AD) incluye un [servicio de aprovisionamiento de cuentas de usuario](user-provisioning.md) que ayuda a automatizar el aprovisionamiento y el desaprovisionamiento de cuentas de usuario en aplicaciones SaaS y otros sistemas, para la administración del ciclo de vida de las identidades de un extremo a otro. Azure AD admite conectores de aprovisionamiento de usuarios integrados previamente para todas las aplicaciones y sistemas que cuentan con tutoriales de aprovisionamiento de usuarios incluidos [aquí](../saas-apps/tutorial-list.md).

En este artículo se describe cómo comprobar el estado de aprovisionamiento de trabajos una vez configurados y cómo solucionar el aprovisionamiento de usuarios individuales y grupos.

## <a name="overview"></a>Información general

Los conectores de aprovisionamiento se configuran mediante [Azure Portal](https://portal.azure.com), siguiendo la [documentación proporcionada](../saas-apps/tutorial-list.md) para la aplicación admitida. Una vez configurados y en ejecución, es posible notificar trabajos de aprovisionamiento mediante uno de estos dos métodos:

* **Azure Portal**: en este artículo se describe principalmente la recuperación de la información de informes desde [Azure Portal](https://portal.azure.com), que proporciona tanto un informe de resumen de aprovisionamiento como registros de auditoría detallados del aprovisionamiento para una aplicación determinada.
* **API de auditoría**: Azure Active Directory también proporciona una API de auditoría que permite la recuperación mediante programación de registros detallados de auditoría de aprovisionamiento. Consulte [Referencia de la API de auditoría de Azure Active Directory](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/directoryaudit) para ver documentación específica sobre el uso de esta API. Aunque este artículo no trata específicamente del uso de la API, detalla los tipos de eventos de aprovisionamiento que se registran en el registro de auditoría.

### <a name="definitions"></a>Definiciones

En este artículo se utilizan los términos que se definen a continuación:

* **Sistema de origen**: el repositorio de usuarios desde el que se sincroniza el servicio de aprovisionamiento de Azure AD. Azure Active Directory es el sistema de origen para la mayoría de los conectores de aprovisionamiento integrados previamente, sin embargo, hay algunas excepciones (ejemplo: Sincronización de entrada de Workday).
* **Sistema de destino**: el repositorio de usuarios al que se sincroniza el servicio de aprovisionamiento de Azure AD. Esto suele ser una aplicación SaaS (ejemplos: Salesforce, ServiceNow, G Suite, Dropbox Business), pero en algunos casos puede ser un sistema local como Active Directory (ejemplo: Sincronización de entrada de Workday a Active Directory).

## <a name="getting-provisioning-reports-from-the-azure-portal"></a>Obtención de informes de aprovisionamiento desde Azure Portal

Para obtener información de informes de aprovisionamiento de una aplicación determinada, comience por iniciar [Azure Portal](https://portal.azure.com) y **Azure Active Directory** &gt; **Aplicaciones empresariales** &gt; **Registros de aprovisionamiento (versión preliminar)** en la sección **Actividad**. También puede ir a la aplicación empresarial para la que está configurado el aprovisionamiento. Por ejemplo, si va a aprovisionar usuarios para LinkedIn Elevate, la ruta de navegación a los detalles de la aplicación es la siguiente:

**Azure Active Directory &gt; Aplicaciones empresariales &gt; Todas las aplicaciones &gt; LinkedIn Elevate**

Desde aquí, puede acceder a la barra de progreso de aprovisionamiento y a los registros de aprovisionamiento, que se describen a continuación.

## <a name="provisioning-progress-bar"></a>Barra de progreso de aprovisionamiento

La [barra de progreso de aprovisionamiento](application-provisioning-when-will-provisioning-finish-specific-user.md#view-the-provisioning-progress-bar) está visible en la **pestaña Aprovisionamiento** de la aplicación específica. Se encuentra en la sección **Estado actual** bajo **Configuración**, y muestra el estado del ciclo inicial o incremental actual. En esta sección también se muestra lo siguiente:

* El número total de usuarios y grupos que se han sincronizado y están actualmente en el ámbito para el aprovisionamiento entre el sistema de origen y el sistema de destino.
* La última vez que se ejecutó la sincronización. Las sincronizaciones suelen producirse cada 20-40 minutos, una vez que haya finalizado un [ciclo inicial](../app-provisioning/how-provisioning-works.md#provisioning-cycles-initial-and-incremental).
* Tanto si ha finalizado un [ciclo inicial](../app-provisioning/how-provisioning-works.md#provisioning-cycles-initial-and-incremental) como si no.
* Tanto si el proceso de aprovisionamiento se ha puesto en cuarentena como si no, y sea cual sea la razón para el estado de cuarentena (por ejemplo, error al comunicarse con el sistema de destino debido a que las credenciales del administrador no son válidas).

El **Estado actual** debe ser el primer lugar en el que busquen los administradores para comprobar el estado operativo del trabajo de aprovisionamiento.

 ![Informe de resumen](./media/check-status-user-account-provisioning/provisioning-progress-bar-section.png)

## <a name="provisioning-logs-preview"></a>Registros de aprovisionamiento (versión preliminar)

Todas las actividades realizadas por el servicio de aprovisionamiento se registran en los [registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context) de Azure AD. Para acceder a los registros de aprovisionamiento en Azure Portal, seleccione **Azure Active Directory** &gt; **Aplicaciones empresariales** &gt; **Registros de aprovisionamiento (versión preliminar)** en la sección **Actividad**. Puede buscar los datos de aprovisionamiento por el nombre del usuario o el identificador en el sistema de origen o en el sistema de destino. Para más información, consulte [Registros de aprovisionamiento (versión preliminar)](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context). Los tipos de eventos de actividades registradas incluyen:

## <a name="troubleshooting"></a>Solución de problemas

El informe de resumen de aprovisionamiento y los registros de aprovisionamiento desempeñan un papel clave a la hora de ayudar a los administradores a solucionar diversos problemas de aprovisionamiento de cuentas de usuario.

Para ver instrucciones basadas en escenarios sobre cómo solucionar problemas de aprovisionamiento automático de usuarios, consulte [Problemas al configurar y aprovisionar usuarios en una aplicación](../app-provisioning/application-provisioning-config-problem.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)