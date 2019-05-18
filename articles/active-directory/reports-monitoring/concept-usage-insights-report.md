---
title: Informe de uso e información en el portal de Azure Active Directory | Microsoft Docs
description: Introducción a los informes de uso e información en el portal de Azure Active Directory
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 3fba300d-18fc-4355-9924-d8662f563a1f
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 05/13/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.openlocfilehash: 3fe1e72d277bffd4bc9b38ac377e33b341967e17
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65806362"
---
# <a name="usage-and-insights-report-in-the-azure-active-directory-portal"></a>Informe de uso e información en el portal de Azure Active Directory

Con el informe de uso y insights, puede obtener una vista centrada en la aplicación de los datos de inicio de sesión. Puede encontrar respuestas a las preguntas siguientes:

*   ¿Cuáles son la parte superior, usa aplicaciones en mi organización?
*   ¿Qué aplicaciones tienen los inicios de sesión con errores más? 
*   ¿Cuáles son los errores de inicio de sesión principales para cada aplicación?

## <a name="prerequisites"></a>Requisitos previos 

Para obtener acceso a los datos del informe de uso y perspectivas, necesita:

* Un inquilino de Azure AD
* Una licencia de Azure AD premium (P1 y P2) para ver los datos de inicio de sesión
* Un usuario en el administrador global, Administrador de seguridad, lector de seguridad o roles de lector de informes. Además, cualquier usuario (no administradores) puede tener acceso a sus propios inicios de sesión. 

## <a name="access-the-usage-and-insights-report"></a>Tener acceso al informe de uso y perspectivas

1. Acceda a [Azure Portal](https://portal.azure.com).
2. Seleccione el directorio correcto y luego seleccione **Azure Active Directory** y elija **aplicaciones empresariales**.
3. Desde el **actividad** sección, seleccione **uso & insights** para abrir el informe. 

![Informe de uso y perspectivas](./media/concept-usage-insights-report/main-menu.png)
                                     

## <a name="use-the-report"></a>Utilizar el informe

El informe de uso e información muestra la lista de aplicaciones con inicio de sesión de uno o más en el intento y permite ordenar por el número de inicios de sesión correctos, inicios de sesión erróneos y la tasa de éxito.

Al hacer clic en cargar más en la parte inferior de la lista le permite ver aplicaciones adicionales en la página. Puede seleccionar el intervalo de fechas para ver todas las aplicaciones que se han usado dentro del intervalo.

También puede establecer el foco en una aplicación específica. Seleccione **ver la actividad de inicio de sesión** para ver el inicio de sesión en la actividad con el tiempo para la aplicación, así como los errores principales.  

Cuando se selecciona un día en el gráfico de uso de la aplicación, obtener una lista detallada de las actividades de inicio de sesión para la aplicación.  

![Informe de uso y perspectivas](./media/concept-usage-insights-report/usage-and-insights-report.png)

## <a name="next-steps"></a>Pasos siguientes

* [Informe de inicios de sesión](concept-sign-ins.md)