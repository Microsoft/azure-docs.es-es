---
title: Experiencias de usuario con Azure AD Identity Protection
description: Experiencia de usuario de Azure AD Identity Protection
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
ms.openlocfilehash: 7d3315a8d6bca9406461d0082ed85bbefc9459f4
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88950211"
---
# <a name="user-experiences-with-azure-ad-identity-protection"></a>Experiencias de usuario con Azure AD Identity Protection

Con Azure Active Directory Identity Protection, puede:

* Exigir que todos los usuarios se registren para la autenticación multifactor (MFA)
* Automatizar la corrección de inicios de sesión conflictivos y usuarios en peligro

Todas las directivas de Identity Protection afectan a la experiencia de inicio de sesión de los usuarios. El hecho de permitir a los usuarios registrarse para el uso de herramientas como Azure MFA y el autoservicio de restablecimiento de contraseña puede reducir el impacto. Estas herramientas, junto con las opciones de directiva adecuadas, proporcionan a los usuarios una opción de corrección automática cuando lo necesitan.

## <a name="multi-factor-authentication-registration"></a>Registro de la autenticación multifactor

Al habilitar la directiva de Identity Protection que requiere el registro de autenticación multifactor y se dirige a todos los usuarios, se asegurará de que estos tienen la capacidad de usar Azure MFA para solucionar sus problemas por sí mismos en el futuro. La configuración de esta directiva proporciona a los usuarios un período de 14 días para registrarse y, si no lo han hecho al finalizar este plazo, se les exigirá el registro. A continuación se describe la experiencia de los usuarios. Puede encontrar más información en la documentación del usuario final, en el artículo [Información general de la verificación en dos fases y la cuenta profesional o educativa](../user-help/multi-factor-authentication-end-user-first-time.md).

### <a name="registration-interrupt"></a>Interrupción de registro

1. En el inicio de sesión en cualquier aplicación integrada de Azure AD, el usuario recibe una notificación sobre la necesidad de configurar la cuenta para la autenticación multifactor. Esta directiva también se desencadena en la configuración rápida de Windows 10 para nuevos usuarios con un nuevo dispositivo.
   
    ![Se necesita más información](./media/concept-identity-protection-user-experience/identity-protection-experience-more-info-mfa.png)

1. Complete los pasos guiados para registrarse en Azure Multi-Factor Authentication y completar el inicio de sesión.

## <a name="risky-sign-in-remediation"></a>Solución de inicios de sesión peligrosos

Cuando un administrador ha configurado una directiva de riesgo de inicio de sesión, se informa de ello a los usuarios afectados cuando intentan iniciar sesión y desencadenar el nivel de riesgo de las directivas. 

### <a name="risky-sign-in-self-remediation"></a>Autocorrección de inicios de sesión peligrosos

1. Se informa al usuario de que se ha detectado algún aspecto inusual en su inicio de sesión, como que se haya realizado desde una nueva ubicación, dispositivo o aplicación.
   
    ![Aviso de algo inusual](./media/concept-identity-protection-user-experience/120.png)

1. El usuario debe demostrar su identidad completando el proceso Azure MFA con uno de sus métodos registrados anteriormente. 

### <a name="risky-sign-in-administrator-unblock"></a>Desbloqueo de administrador de inicio de sesión peligroso

Los administradores pueden optar por bloquear a los usuarios al iniciar sesión en función del nivel de riesgo. Para ser desbloqueados, los usuarios finales deben ponerse en contacto con el departamento de soporte técnico, o bien pueden intentar iniciar sesión desde una ubicación o dispositivo conocidos. La autocorrección por medio de la autenticación multifactor no es una opción en este caso.

![Bloqueo por directiva de riesgo de inicio de sesión](./media/concept-identity-protection-user-experience/200.png)

El personal de TI puede seguir las instrucciones de la sección para [desbloquear usuarios](howto-identity-protection-remediate-unblock.md#unblocking-based-on-sign-in-risk) a fin de permitir que los usuarios vuelvan a iniciar sesión.

## <a name="risky-user-remediation"></a>Corrección de usuarios de riesgo

Cuando se ha configurado una directiva de riesgo de usuario, los usuarios que cumplen la probabilidad del nivel de riesgo de usuario deben pasar por el flujo de recuperación del peligro antes de poder iniciar sesión. 

### <a name="risky-user-self-remediation"></a>Autocorrección de usuarios de riesgo

1. Se informa al usuario que la seguridad de su cuenta está en peligro debido a actividad sospechosa o credenciales con fugas.
   
    ![Corrección](./media/concept-identity-protection-user-experience/101.png)

1. El usuario debe demostrar su identidad completando el proceso Azure MFA con uno de sus métodos registrados anteriormente. 
1. Por último, el usuario debe cambiar su contraseña mediante el autoservicio de restablecimiento de contraseña, ya que es posible que otra persona haya tenido acceso a su cuenta.

## <a name="risky-sign-in-administrator-unblock"></a>Desbloqueo de administrador de inicio de sesión peligroso

Los administradores pueden optar por bloquear a los usuarios al iniciar sesión en función del nivel de riesgo. Para que se les desbloquee, los usuarios finales deben ponerse en contacto con el personal de TI. La autocorrección por medio de la autenticación multifactor y el autoservicio de restablecimiento de contraseña no es una opción en este caso.

![Bloque por directiva de riesgo de usuario](./media/concept-identity-protection-user-experience/104.png)

El personal de TI puede seguir las instrucciones de la sección para [desbloquear usuarios](howto-identity-protection-remediate-unblock.md#unblocking-based-on-user-risk) a fin de permitir que los usuarios vuelvan a iniciar sesión.

## <a name="see-also"></a>Consulte también

- [Corregir riesgos y desbloquear usuarios](howto-identity-protection-remediate-unblock.md)

- [Azure Active Directory Identity Protection](./overview-identity-protection.md)