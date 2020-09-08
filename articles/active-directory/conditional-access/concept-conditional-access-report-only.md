---
title: ¿Qué es el modo de solo informe de acceso condicional? - Azure Active Directory
description: Cómo puede ayudar el modo de solo informe con la implementación de directivas de acceso condicional
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 05/01/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: dawoo
ms.collection: M365-identity-device-management
ms.openlocfilehash: df25278d37e518e99bc315a315e017b98c7f4e58
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89049390"
---
# <a name="what-is-conditional-access-report-only-mode"></a>¿Qué es el modo de solo informe de acceso condicional?

Nuestros clientes usan ampliamente el acceso condicional para mantenerse seguros aplicando los controles de acceso adecuados en las circunstancias adecuadas. Sin embargo, uno de los desafíos con la implementación de una directiva de acceso condicional en su organización es determinar el impacto en los usuarios finales. Puede ser difícil prever el número y los nombres de los usuarios afectados por iniciativas de implementación habituales como el bloqueo de la autenticación heredada, la necesidad de autenticación multifactor para una población de usuarios o la implementación de directivas de riesgo de inicio de sesión. 

El modo de solo informe es un nuevo estado de la directiva de acceso condicional que permite a los administradores evaluar el impacto de las directivas de acceso condicional antes de habilitarlas en su entorno.  Con la versión del modo de solo informe:

- Las directivas de acceso condicional se pueden habilitar en modo de solo informe.
- Durante el inicio de sesión, las directivas en modo de solo informe se evalúan, pero no se aplican.
- Los resultados se registran en las pestañas **Acceso condicional** y **Solo informe** de los detalles de registro de inicio de sesión.
- Los clientes con una suscripción de Azure Monitor pueden supervisar el impacto de sus directivas de acceso condicional  mediante el libro Conditional Access Insights.

> [!WARNING]
> Las directivas en modo de solo informe que requieren dispositivos compatibles pueden solicitar a los usuarios de Mac, iOS y Android que seleccionen un certificado de dispositivo durante la evaluación de directiva, aunque el cumplimiento de dispositivos no se fuerce. Estos mensajes pueden repetirse hasta que el dispositivo sea compatible. Para evitar que los usuarios finales reciban mensajes durante el inicio de sesión, excluya las plataformas de dispositivo Mac, iOS y Android de las directivas de solo informe que realizan comprobaciones de cumplimiento de dispositivos. Tenga en cuenta que el modo de solo informe no es aplicable a las directivas de CA con el ámbito "acciones del usuario".

![Pestaña Solo informe del registro de inicio de sesión de Azure AD](./media/concept-conditional-access-report-only/report-only-detail-in-sign-in-log.png)

## <a name="policy-results"></a>Resultados de directivas

Cuando se evalúa una directiva en modo de solo informe para un inicio de sesión especificado, hay cuatro nuevos valores de resultado posibles:

| Resultado | Descripción |
| --- | --- |
| Solo informe: Correcto | Se cumplieron todas las condiciones de directiva configuradas, los controles de concesión no interactivos requeridos y los controles de sesión. Por ejemplo, una notificación de MFA ya presente en el token cumple un requisito de autenticación multifactor o una directiva de dispositivo compatible se cumple realizando una comprobación de dispositivo en un dispositivo compatible. |
| Solo informe: Error | Se cumplieron todas las condiciones de directiva configuradas, pero no se cumplieron todos los controles de concesión no interactivos requeridos ni los controles de sesión. Por ejemplo, una directiva se aplica a un usuario donde se configura un control de bloqueo, o un dispositivo produce un error en una directiva de dispositivo compatible. |
| Solo informe: Se requiere una acción del usuario | Se cumplieron todas las condiciones de directiva configuradas, pero se requiere la acción del usuario para cumplir los controles de concesión requeridos o los controles de sesión. Con el modo de solo informe, no se solicita al usuario que cumpla los controles requeridos. Por ejemplo, no se solicita a los usuarios los desafíos de autenticación multifactor ni los términos de uso.   |
| Solo informe: No aplicado | No se cumplieron todas las condiciones de directiva configuradas. Por ejemplo, el usuario se excluye de la directiva o la directiva solo se aplica a determinadas ubicaciones con nombre de confianza. |

## <a name="conditional-access-insights-workbook"></a>Libro Conditional Access Insights

Los administradores pueden crear varias directivas en modo de solo informe, por lo que es necesario comprender tanto el impacto individual de cada directiva como el impacto combinado de varias directivas evaluadas de forma conjunta. El nuevo libro Conditional Access Insights permite a los administradores visualizar consultas de acceso condicional y supervisar el impacto de una directiva durante un intervalo de tiempo determinado, el conjunto de aplicaciones y los usuarios. 
 
## <a name="next-steps"></a>Pasos siguientes

[Configuración del modo de solo informe en una directiva de acceso condicional](howto-conditional-access-insights-reporting.md)
