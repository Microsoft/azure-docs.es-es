---
title: 'Proporcionar comentarios sobre los eventos de riesgo en Azure AD Identity Protection: Azure Active Directory'
description: Cómo y por qué ¿debería proporcionar comentarios sobre los eventos de riesgo de Identity Protection.
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: conceptual
ms.date: 05/13/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 66d53590e89afb1a903b22ff60e32871a1502ada
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827911"
---
# <a name="how-to-give-risk-feedback-in-azure-ad-identity-protection"></a>Instrucciones: Comentarios de riesgo en Azure AD Identity Protection

Azure AD Identity Protection le permite enviar comentarios sobre su evaluación de riesgos. El documento siguiente enumera los escenarios donde desea enviar comentarios sobre la evaluación del riesgo de Azure AD Identity Protection y cómo se incorporan.

## <a name="what-is-a-detection"></a>¿Qué es una detección?

La detección de Identity Protection es un indicador de actividad sospechosa desde una perspectiva de riesgo de identidad. Estas actividades sospechosas se denominan eventos de riesgo. Estas detecciones basada en identidades pueden basarse en heurística, aprendizaje automático o pueden proceder de productos de socios. Estas detecciones se usan para determinar el riesgo de inicio de sesión y riesgo del usuario,

* Riesgo de usuario representa la probabilidad de que una identidad está en peligro.
* Inicio de sesión de riesgo representa la probabilidad de inicio de sesión se pone en peligro (por ejemplo, el inicio de sesión no está autorizado por el propietario de la identidad).

## <a name="why-should-i-give-risk-feedback-to-azure-ads-risk-assessments"></a>¿Por qué debo proporcionar comentarios de riesgo a evaluaciones de riesgo de Azure AD? 

Hay varias razones por qué debería dar a los comentarios de riesgo de Azure AD:

1. **Ha encontrado una evaluación del riesgo de inicio de sesión o usuario de Azure AD incorrecto**. Por ejemplo, un inicio de sesión que se muestra en el informe 'Inicios de sesión' era benigno y todas las detecciones en ese inicio de sesión eran los falsos positivos.
1. **Valida los usuarios de Azure AD o era correcta, la evaluación del riesgo de inicio de sesión**. Por ejemplo, un inicio de sesión que se muestra en el informe 'Inicios de sesión' era realmente malintencionado y desea que Azure AD saber que todas las detecciones en ese inicio de sesión fueron verdaderos positivos.
1. **Corregir el riesgo de ese usuario fuera de Azure AD Identity Protection** y desea que el nivel de riesgo del usuario para actualizarse.

## <a name="how-does-azure-ad-use-my-risk-feedback"></a>¿Cómo usa Azure AD mis comentarios de riesgo?

Azure AD usa sus comentarios para actualizar el riesgo de que el subyacente usuario o inicio de sesión. Estos comentarios le ayuda a proteger el usuario final. Por ejemplo, después de confirmar el que inicio de sesión está en peligro, Azure AD inmediatamente aumenta el riesgo del usuario y el agregado riesgo (no en tiempo real) inicio de sesión del complemento a gran. Si este usuario está incluido en la directiva de riesgo de usuario para forzar a los usuarios de alto riesgo para restablecer sus contraseñas de forma segura, el usuario corregirá automáticamente sí misma la próxima vez que inicien sesión en.

## <a name="how-should-i-give-risk-feedback-and-what-happens-under-the-hood"></a>¿Cómo debo conceder riesgo comentarios y lo que sucede en segundo plano?

Estos son los escenarios y los mecanismos de comentarios de riesgo a Azure AD.

| Escenario | ¿Cómo enviar comentarios? | ¿Qué sucede en segundo plano? | Notas |
| --- | --- | --- | --- |
| **Inicio de sesión no pone en peligro (falso positivo)** <br> Muestra el informe 'Inicios de sesión' en un inicio de sesión en riesgo [estado de riesgo = expuestos] pero ese inicio de sesión no está en peligro. | Seleccione el inicio de sesión y haga clic en 'Confirmar inicio de sesión seguro para'. | Azure AD moverá riesgo agregado del inicio de sesión en none [estado de riesgo = confirmado seguro; (Agregado) de nivel de riesgo =-] e invertirá su impacto del riesgo del usuario. | Actualmente, la opción 'Confirmar inicio de sesión seguro para' solo está disponible en el informe 'Inicios de sesión'. |
| **Inicio de sesión en peligro (falso positivo)** <br> Muestra el informe 'Inicios de sesión' en un inicio de sesión en riesgo [estado de riesgo = expuestos] con bajo riesgo [(agregado) de nivel de riesgo = baja] y ese inicio de sesión en realidad está en peligro. | Seleccione el inicio de sesión y haga clic en"Confirmar inicio de sesión en riesgo". | Azure AD moverá el riesgo de usuario y riesgo agregado del inicio de sesión al alto [estado de riesgo = confirmado en peligro. Nivel de riesgo = alto]. | Actualmente, la opción 'Confirmar inicio de sesión de peligro' solo está disponible en el informe 'Inicios de sesión'. |
| **Usuario en peligro (falso positivo)** <br> Informe "Usuarios de riesgo" muestra un usuario en riesgo [estado de riesgo = expuestos] con bajo riesgo [nivel de riesgo = baja] y que el usuario realmente está en peligro. | Seleccione el usuario y haga clic en "User de confirmar en peligro". | Azure AD moverá el riesgo del usuario al alto [estado de riesgo = confirmado en peligro. Nivel de riesgo = alto] y agregará una nueva detección 'Admin confirmado en peligro el usuario'. | Actualmente, la opción "User de confirmar en peligro" solo está disponible en el informe "Usuarios de riesgo". <br> La detección "Admin confirmado usuario en peligro" se muestra en la ficha "No vinculados a un inicio de sesión de eventos de riesgo" en el informe "Usuarios de riesgo". |
| **Usuario corrige fuera de Azure AD Identity Protection (verdadero positivo + Remediated)** <br> Informe "Usuarios de riesgo" muestra un usuario en riesgo y he corregido posteriormente el usuario fuera de Azure AD Identity Protection. | 1. Seleccione el usuario y haga clic en "User de confirmar en peligro". (Este proceso se confirma en Azure AD que realmente se ha puesto en peligro el usuario.) <br> 2. Espere a que el nivel del usuario"riesgo" para ir a alto. (Esta vez le ofrece Azure AD el tiempo necesario para seguir los comentarios anteriores para el motor de riesgo.) <br> 3. Seleccione el usuario y haga clic en 'Descartar el riesgo del usuario'. (Este proceso se confirma en Azure AD que el usuario ya no está en peligro.) |  Azure AD mueve el riesgo del usuario en none [estado de riesgo = descartadas; Nivel de riesgo =-] y el riesgo en todos los existentes inicios de sesión con riesgo activo se cierra. | Al hacer clic en 'Descartar riesgo del usuario' cerrará todos los riesgos en el usuario y más allá de los inicios de sesión. Esta acción no se puede deshacer. |
| **Usuario no pone en peligro (falso positivo)** <br> Informe "Usuarios de riesgo" se muestra al usuario en riesgo, pero el usuario no está en peligro. | Seleccione el usuario y haga clic en 'Descartar el riesgo del usuario'. (Este proceso se confirma en Azure AD que el usuario no está en peligro.) | Azure AD mueve el riesgo del usuario en none [estado de riesgo = descartadas; Nivel de riesgo =-]. | Al hacer clic en 'Descartar riesgo del usuario' cerrará todos los riesgos en el usuario y más allá de los inicios de sesión. Esta acción no se puede deshacer. |
| Deseo cerrar el riesgo del usuario pero no estoy seguro de si el usuario está en peligro / safe. | Seleccione el usuario y haga clic en 'Descartar el riesgo del usuario'. (Este proceso se confirma en Azure AD que el usuario ya no está en peligro.) | Azure AD mueve el riesgo del usuario en none [estado de riesgo = descartadas; Nivel de riesgo =-]. | Al hacer clic en 'Descartar riesgo del usuario' cerrará todos los riesgos en el usuario y más allá de los inicios de sesión. Esta acción no se puede deshacer. Se recomienda corregir el usuario haciendo clic en "Restablecer contraseña" o solicitar al usuario para restablecer o cambiar de forma segura sus credenciales. |

Comentarios sobre los eventos de riesgo del usuario en Identity Protection se procesan sin conexión y pueden tardar algún tiempo en actualizar. El procesamiento de la columna de estado de riesgo proporcionará el estado actual de procesamiento de comentarios.

![Estado de procesamiento de informes de usuario en peligro de riesgo](./media/howto-provide-risk-event-feedback/risky-users-provide-feedback.png)

## <a name="next-steps"></a>Pasos siguientes

[Referencia de eventos de riesgo de Azure Active Directory Identity Protection](risk-events-reference.md)