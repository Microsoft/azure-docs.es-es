---
title: Proporcionar comentarios sobre riesgos de Azure Active Directory Identity Protection
description: Cómo y por qué se deberían proporcionar comentarios sobre las detecciones de riesgo de Identity Protection.
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: how-to
ms.date: 06/05/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 939ffc9e2a8fb8ce3a25dc212e3df34dc6bb2ec1
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88950364"
---
# <a name="how-to-give-risk-feedback-in-azure-ad-identity-protection"></a>Instrucciones: Enviar comentarios sobre riesgo en Azure AD Identity Protection

Azure AD Identity Protection le permite enviar comentarios sobre su evaluación de riesgos. En el documento siguiente se enumeran los escenarios en los que puede enviar comentarios sobre la evaluación del riesgo de Azure AD Identity Protection y se proporciona información para incorporarlos.

## <a name="what-is-a-detection"></a>¿Qué es una detección?

La detección de Identity Protection es un indicador de actividad sospechosa desde una perspectiva de riesgo de identidad. Estas actividades sospechosas se llaman detecciones de riesgo. Estas detecciones basadas en identidades pueden basarse en heurística, aprendizaje automático o pueden proceder de productos de asociados. Estas detecciones se usan para determinar el riesgo de inicio de sesión y el riesgo del usuario.

* El riesgo de usuario representa la probabilidad de que una identidad esté en peligro.
* El riesgo de inicio de sesión representa la probabilidad de que un inicio de sesión esté en peligro (por ejemplo, el inicio de sesión no está autorizado por el propietario de la identidad).

## <a name="why-should-i-give-risk-feedback-to-azure-ads-risk-assessments"></a>¿Por qué debo enviar comentarios sobre riesgo a las evaluaciones de riesgo de Azure AD? 

Hay varias razones por las que debería enviar comentarios sobre el riesgo de Azure AD:

- **Ha encontrado una evaluación de riesgo de inicio de sesión o usuario de Azure AD incorrecta**. Por ejemplo, un inicio de sesión que se muestra en el informe "Inicios de sesión de riesgo" era inofensivo y todas las detecciones en ese inicio de sesión eran falsos positivos.
- **Ha validado que la evaluación de riesgo de inicio de sesión o usuario de Azure AD era correcta**. Por ejemplo, un inicio de sesión que se muestra en el informe "Inicios de sesión de riesgo" era realmente malintencionado y quiere que Azure AD sepa que todas las detecciones en ese inicio de sesión eran verdaderos positivos.
- **Ha corregido el riesgo de ese usuario fuera de Azure AD Identity Protection** y quiere que el nivel de riesgo del usuario se actualice.

## <a name="how-does-azure-ad-use-my-risk-feedback"></a>¿Cómo usa Azure AD mis comentarios sobre riesgo?

Azure AD usa sus comentarios para actualizar el riesgo del usuario o inicio de sesión subyacentes y la precisión de estos eventos. Estos comentarios le ayudan a proteger al usuario final. Por ejemplo, después de confirmar que un inicio de sesión está en peligro, Azure AD aumenta inmediatamente el nivel de riesgo de usuario y riesgo agregado de inicio de sesión (no en tiempo real) a Alto. Si este usuario está incluido en la directiva de riesgo de usuario para forzar a los usuarios de alto riesgo a restablecer sus contraseñas de forma segura, el usuario se corregirá automáticamente la próxima vez que inicie sesión.

## <a name="how-should-i-give-risk-feedback-and-what-happens-under-the-hood"></a>¿Cómo debo enviar comentarios sobre riesgo y qué sucede en segundo plano?

Estos son los escenarios y los mecanismos para enviar comentarios sobre riesgo a Azure AD.

| Escenario | Cómo enviar comentarios | Qué sucede en segundo plano | Notas |
| --- | --- | --- | --- |
| **Inicio de sesión que no está en peligro (falso positivo)** <br> En el informe "Inicios de sesión de riesgo" se muestra un inicio de sesión de riesgo [Estado de riesgo = En riesgo], pero ese inicio de sesión no estaba en peligro. | Seleccione el inicio de sesión y haga clic en "Confirmar inicio de sesión seguro". | Azure AD moverá el riesgo agregado de inicio de sesión a Ninguno [Estado de riesgo = Confirmado seguro; Nivel de riesgo (agregado) = -] e invertirá su impacto sobre el riesgo de usuario. | Actualmente, la opción "Confirmar inicio de sesión seguro" solo está disponible en el informe "Inicios de sesión de riesgo". |
| **Inicio de sesión en peligro (verdadero positivo)** <br> En el informe "Inicios de sesión de riesgo" se muestra un inicio de sesión de riesgo [Estado de riesgo = En riesgo] con nivel bajo de riesgo [Nivel de riesgo (agregado) = Bajo] y que ese inicio de sesión estaba en peligro en realidad. | Seleccione el inicio de sesión y haga clic en "Confirmar inicio de sesión en peligro". | Azure AD moverá el nivel de riesgo agregado de inicio de sesión y el riesgo de usuario a Alto [Estado de riesgo = Confirmado en peligro; Nivel de riesgo = Alto]. | Actualmente, la opción "Confirmar inicio de sesión en peligro" solo está disponible en el informe "Inicios de sesión de riesgo". |
| **Usuario en peligro (verdadero positivo)** <br> En el informe "Usuarios de riesgo" se muestra un usuario de riesgo [Estado de riesgo = En riesgo] con nivel bajo de riesgo [Nivel de riesgo = Bajo] y que ese usuario estaba en peligro en realidad. | Seleccione el usuario y haga clic en "Confirmar usuario en peligro". | Azure AD moverá el nivel del usuario de riesgo a Alto [Estado de riesgo = Confirmado en peligro; Nivel de riesgo = Alto] y agregará una nueva detección "Vulneración de identidad de usuario confirmada por el administrador". | Actualmente, la opción "Confirmar usuario en peligro" solo está disponible en el informe "Usuarios de riesgo". <br> La detección "Vulneración de identidad de usuario confirmada por el administrador" se muestra en la pestaña "Detecciones de riesgo no vinculadas a un inicio de sesión" en el informe "Usuarios de riesgo". |
| **Usuario corregido fuera de Azure AD Identity Protection (verdadero positivo + corregido)** <br> En el informe "Usuarios de riesgo" se muestra un usuario en riesgo y, posteriormente, he corregido el usuario fuera de Azure AD Identity Protection. | 1. Seleccione el usuario y haga clic en "Confirmar usuario en peligro". (Este proceso confirma a Azure AD que el usuario estaba en peligro realmente). <br> 2. Espere a que el "nivel de riesgo" del usuario pase a Alto. (Esto le ofrece a Azure AD el tiempo necesario para tener en cuenta los comentarios anteriores para el motor de riesgo). <br> 3. Seleccione el usuario y haga clic en "Descartar el riesgo del usuario". (Este proceso confirma a Azure AD que el usuario ya no está en peligro). |  Azure AD mueve el riesgo del usuario a Ninguno [Estado de riesgo = Descartado; Nivel de riesgo = -] y cierra el riesgo en todos los inicios de sesión existentes con riesgo activo. | Al hacer clic en "Descartar el riesgo del usuario", se cerrarán todos los riesgos del usuario y de los inicios de sesión pasados. Esta operación no se puede deshacer. |
| **Usuario que no está en peligro (falso positivo)** <br> En el informe "Usuarios de riesgo" se muestra el usuario en riesgo, pero este no está en peligro. | Seleccione el usuario y haga clic en "Descartar el riesgo del usuario". (Este proceso confirma a Azure AD que el usuario no está en peligro). | Azure AD mueve el riesgo del usuario a Ninguno [Estado de riesgo = Descartado; Nivel de riesgo = -]. | Al hacer clic en "Descartar el riesgo del usuario", se cerrarán todos los riesgos del usuario y de los inicios de sesión pasados. Esta operación no se puede deshacer. |
| Quiero cerrar el riesgo del usuario pero no estoy seguro de si el usuario está en peligro o seguro. | Seleccione el usuario y haga clic en "Descartar el riesgo del usuario". (Este proceso confirma a Azure AD que el usuario ya no está en peligro). | Azure AD mueve el riesgo del usuario a Ninguno [Estado de riesgo = Descartado; Nivel de riesgo = -]. | Al hacer clic en "Descartar el riesgo del usuario", se cerrarán todos los riesgos del usuario y de los inicios de sesión pasados. Esta operación no se puede deshacer. Se recomienda corregir el usuario haciendo clic en "Restablecer contraseña" o solicitar al usuario que restablezca o cambie sus credenciales de forma segura. |

Los comentarios sobre las detecciones de riesgo de usuario en Identity Protection se procesan sin conexión y pueden tardar algún tiempo en actualizarse. La columna de estado de procesamiento de riesgo proporcionará el estado actual del procesamiento de comentarios.

![Estado de procesamiento de riesgo para el informe de usuarios en riesgo](./media/howto-identity-protection-risk-feedback/risky-users-provide-feedback.png)

## <a name="next-steps"></a>Pasos siguientes

- [Referencia sobre detecciones de riesgo de Azure Active Directory Identity Protection](./concept-identity-protection-risks.md)