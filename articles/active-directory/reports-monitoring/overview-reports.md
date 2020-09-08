---
title: ¿Qué son los informes de Azure Active Directory? | Microsoft Docs
description: Proporciona una visión general sobre los informes de Azure Active Directory.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 6141a333-38db-478a-927e-526f1e7614f4
ms.service: active-directory
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5c07500ec432d176b2cf74aa95f0a1c943348f9f
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89229523"
---
# <a name="what-are-azure-active-directory-reports"></a>¿Qué son los informes de Azure Active Directory?

Los informes de Azure Active Directory (Azure AD) proporcionan una vista completa de la actividad en su entorno. Los datos proporcionados le permiten:

- Determinar cómo utilizan los usuarios las aplicaciones y servicios
- Detectar posibles riesgos que afectan al estado de su entorno
- Solucionar problemas que impiden a los usuarios finalizar su trabajo  

La arquitectura de informes se basa en dos pilares principales:

- [Informes de seguridad](#security-reports)
- [Informes de actividad](#activity-reports)

![Notificación](./media/overview-reports/01.png)


## <a name="security-reports"></a>Informes de seguridad

Los informes de seguridad ayudan a proteger las identidades de su organización. Hay dos tipos de informes de seguridad:

- **Usuarios marcados en riesgo**: en el [informe de seguridad de usuarios marcados en riesgo](../identity-protection/overview-identity-protection.md) puede obtener una visión general de las cuentas de usuario que pueden haber estado en peligro.

- **Inicios de sesión de riesgo**: en el [informe de seguridad de inicios de sesión de riesgo](../identity-protection/overview-identity-protection.md) puede obtener un indicador de un intento de inicio de sesión que puede haber realizado alguien que no es el propietario legítimo de una cuenta de usuario. 

### <a name="what-azure-ad-license-do-you-need-to-access-a-security-report"></a>¿Qué licencia de Azure AD se necesita para acceder a un informe de seguridad?  

Todas las ediciones de Azure AD le proporcionan informes de usuarios marcados en riesgo e informes de inicios de sesión de riesgo. Sin embargo, el nivel de granularidad del informe varía según la edición: 

- En las **ediciones gratuita y básica de Azure Active Directory**, puede obtener una lista de usuarios marcados en riesgo y de inicios de sesión de riesgo. 

- En la edición **Azure Active Directory Premium 1** se extiende este modelo, con lo que también puede examinar algunos de las detecciones de riesgo subyacentes que se han detectado para cada informe. 

- La edición **Azure Active Directory Premium 2** le proporciona la información más detallada acerca de las detecciones de riesgo subyacentes y también le permite configurar directivas de seguridad que responden automáticamente a los niveles de riesgo configurados.


## <a name="activity-reports"></a>Informes de actividad

Los informes de actividad le ayudan a comprender el comportamiento de los usuarios de su organización. Hay dos tipos de informes de actividad en Azure AD:

- **Registros de auditoría**: el [informe de actividad de registros de auditoría](concept-audit-logs.md) le proporciona acceso al historial de todas las tareas llevadas a cabo en el inquilino.

- **Inicios de sesión**: con el [informe de actividad de inicios de sesión](concept-sign-ins.md), puede determinar quién ha realizado las tareas notificadas en el informe de registros de auditoría.


### <a name="audit-logs-report"></a>Informe de registros de auditoría 

Los [informes de registros de auditoría](concept-audit-logs.md) le proporcionan registros de las actividades del sistema en relación con el cumplimiento normativo. Estos datos le permiten abordar cuestiones comunes como:

- Algún usuario de mi inquilino obtuvo acceso a un grupo de administración. ¿Quién les dio acceso? 

- Quiero conocer la lista de usuarios que inician sesión en una aplicación específica ya que la puse en marcha recientemente y quiero conocer si está funcionando correctamente.

- Quiero saber cuántos restablecimientos de contraseña se producen en mi inquilino


#### <a name="what-azure-ad-license-do-you-need-to-access-the-audit-logs-report"></a>¿Qué licencia de Azure AD se necesita para acceder al informe de registros de auditoría?  

El informe de registros de auditoría está disponible para aquellas características para las que dispone de licencia. Si tiene una licencia para una característica determinada, también tiene acceso a la información del registro de auditoría de dicha característica. Para ver una comparación detallada de las características según los [diferentes tipos de licencias](../fundamentals/active-directory-whatis.md#what-are-the-azure-ad-licenses), consulte la [página de precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/). Para más información, consulte [Características y funcionalidades de Azure Active Directory](../fundamentals/active-directory-whatis.md#which-features-work-in-azure-ad).

### <a name="sign-ins-report"></a>Informe de inicios de sesión

El [informe de inicios de sesión](concept-sign-ins.md) le permite encontrar respuestas a preguntas como:

- ¿Cuál es el patrón de inicio de sesión de un usuario?
- ¿Cuántos usuarios tienen usuarios que han iniciado sesión durante una semana?
- ¿Cuál es el estado de estos inicios de sesión?

#### <a name="what-azure-ad-license-do-you-need-to-access-the-sign-ins-activity-report"></a>¿Qué licencia de Azure AD se necesita para acceder a los informes de actividades de inicio de sesión?  

El inquilino debe tener una licencia de Azure AD Premium asociada para acceder al informe de actividades de inicio de sesión.

## <a name="programmatic-access"></a>Acceso mediante programación

Además de la interfaz de usuario, Azure AD también proporciona [acceso mediante programación](concept-reporting-api.md) a los datos de informes, mediante un conjunto de API basadas en REST. Estas API pueden llamarse desde una variedad de lenguajes de programación y herramientas. 

## <a name="next-steps"></a>Pasos siguientes

- [Informe de inicios de sesión de riesgo](../identity-protection/overview-identity-protection.md)
- [Informe de registros de auditoría](concept-audit-logs.md)
- [Informe de registros de inicios de sesión](concept-sign-ins.md)