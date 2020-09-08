---
title: 'Identity Protection y usuarios de B2B: Azure Active Directory'
description: 'Uso de Identity Protection con usuarios de B2B: cómo funciona y limitaciones conocidas'
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: conceptual
ms.date: 10/18/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: c9ec9e110c3a476c9096ae3e216c9780da0e0f49
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88949203"
---
# <a name="identity-protection-and-b2b-users"></a>Protección de identidades y usuarios de Colaboración B2B

Con la Colaboración B2B de Azure AD, las organizaciones pueden exigir directivas basadas en riesgos para usuarios de B2B mediante Identity Protection. Estas directivas se configuran de dos maneras:

- Los administradores pueden configurar las directivas basadas en riesgos de Identity Protection integradas, que se aplican a todas las aplicaciones, y que incluyen a los usuarios invitados.
- Los administradores pueden configurar sus directivas de acceso condicional mediante el riesgo de inicio de sesión como una condición, que incluye a los usuarios invitados.

## <a name="how-is-risk-evaluated-for-b2b-collaboration-users"></a>Evaluación del riesgo para los usuarios de Colaboración B2B

El riesgo del usuario para los usuarios de Colaboración B2B se evalúa en su directorio particular. El riesgo de inicio de sesión en tiempo real para estos usuarios se evalúa en el directorio de recursos cuando intentan acceder al recurso.

## <a name="limitations-of-identity-protection-for-b2b-collaboration-users"></a>Limitaciones de Identity Protection para los usuarios de Colaboración B2B

Existen limitaciones en la implementación de Identity Protection para los usuarios de Colaboración B2B en un directorio de recursos debido a su identidad existente en su directorio particular. Las principales limitaciones son las siguientes:

- Si un usuario invitado desencadena la directiva de riesgo de usuario de Identity Protection para forzar el restablecimiento de la contraseña, **se bloqueará**. Este bloqueo se debe a la imposibilidad de restablecer contraseñas en el directorio de recursos.
- **Los usuarios invitados no aparecen en el informe de usuarios de riesgo**. Esta pérdida de visibilidad se debe a la evaluación del riesgo que se produce en el directorio particular del usuario de B2B.
- Los administradores **no pueden descartar ni corregir un usuario de Colaboración B2B de riesgo** en su directorio de recursos. Esta pérdida de funcionalidad se debe a que los administradores del directorio de recursos no tienen acceso al directorio particular del usuario de B2B.

### <a name="why-cant-i-remediate-risky-b2b-collaboration-users-in-my-directory"></a>¿Por qué no puedo corregir a los usuarios de Colaboración B2B de riesgo en mi directorio?

La evaluación y la corrección de riesgos para los usuarios de B2B se producen en su directorio particular. Debido a este hecho, los usuarios invitados no aparecen en el informe de usuarios de riesgo en el directorio de recursos y los administradores del directorio de recursos no pueden forzar un restablecimiento de contraseña seguro para estos usuarios.

### <a name="what-do-i-do-if-a-b2b-collaboration-user-was-blocked-due-to-a-risk-based-policy-in-my-organization"></a>¿Qué puedo hacer si un usuario de Colaboración B2B se ha bloqueado debido a una directiva basada en riesgos de la organización?

Si un usuario de B2B en riesgo del directorio está bloqueado por la directiva basada en riesgos, el usuario tendrá que corregir ese riesgo en su directorio particular. Los usuarios pueden corregir su riesgo si realizan un restablecimiento de contraseña seguro en su directorio particular. Si no tienen habilitado el autoservicio de restablecimiento de contraseña en su directorio particular, tendrán que ponerse en contacto con el personal de TI de su propia organización para que un administrador descarte manualmente el riesgo o restablezca su contraseña.

### <a name="how-do-i-prevent-b2b-collaboration-users-from-being-impacted-by-risk-based-policies"></a>¿Cómo se evita que las directivas basadas en riesgos afecten a los usuarios de Colaboración B2B?

La exclusión de los usuarios de B2B de las directivas de acceso condicional basadas en riesgos de la organización impedirá que se vean afectados o bloqueados por su evaluación de riesgos. Para excluir estos usuarios de B2B, cree un grupo en Azure AD que contenga todos los usuarios invitados de la organización. Después, agregue este grupo como una exclusión para las directivas de riesgo de inicio de sesión y riesgo de usuario de Identity Protection integradas, así como para las directivas de acceso condicional con el riesgo de inicio de sesión como condición.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos sobre la colaboración de B2B de Azure AD:

- [¿Qué es la colaboración B2B de Azure AD?](../external-identities/what-is-b2b.md)