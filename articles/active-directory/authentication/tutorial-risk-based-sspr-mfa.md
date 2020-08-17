---
title: Protección del inicio de sesión de usuario basada en el riesgo en Azure Active Directory
description: En este tutorial, aprenderá a habilitar Azure Identity Protection para proteger a los usuarios cuando se detecta un comportamiento de inicio de sesión con riesgo en su cuenta.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: tutorial
ms.date: 07/13/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: e008091b3d0b450384cb7a672a62c786c33bfeab
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87419638"
---
# <a name="tutorial-use-risk-detections-for-user-sign-ins-to-trigger-azure-multi-factor-authentication-or-password-changes"></a>Tutorial: Uso de las detecciones de riesgos en los inicios de sesión de usuario para desencadenar la autenticación multifactor de Azure y cambios de contraseña

Para proteger a los usuarios, puede configurar directivas basadas en el riesgo en Azure Active Directory (Azure AD) que respondan automáticamente a comportamientos de riesgo. Las directivas de Azure AD Identity Protection pueden bloquear automáticamente un intento de inicio de sesión o requerir una acción adicional, como requerir un cambio de contraseña o solicitar la autenticación multifactor de Azure. Estas directivas funcionan con las directivas existentes de acceso condicional de Azure AD como una capa de protección adicional para la organización. Los usuarios podrían no desencadenar nunca un comportamiento de riesgo en una de estas directivas, pero la organización está protegida si se produce un intento de poner en peligro la seguridad.

> [!IMPORTANT]
> Este tutorial muestra al administrador cómo habilitar Azure Multi-Factor Authentication basada en riesgos.
>
> Si el equipo de TI no ha habilitado la función para usar Azure Multi-Factor Authentication o si tiene problemas para iniciar sesión, póngase en contacto con el departamento de soporte técnico para obtener ayuda adicional.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Descripción de las directivas disponibles para Azure AD Identity Protection
> * Habilitación del registro de Azure Multi-Factor Authentication
> * Habilitación de los cambios de contraseña en función del riesgo
> * Habilitación de la autenticación multifactor en función del riesgo
> * Prueba de las directivas basadas en riesgos para los intentos de inicio de sesión de usuario

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará los siguientes recursos y privilegios:

* Un inquilino de Azure AD activo con al menos una licencia de Azure AD Premium P2 o de prueba habilitada.
    * Si es preciso, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Una cuenta con privilegios de *Administrador global*.
* Azure AD configurado para el autoservicio de restablecimiento de contraseña y Azure Multi-Factor Authentication.
    * Si es necesario, [complete el tutorial para habilitar SSPR de Azure AD](tutorial-enable-sspr.md).
    * Si es necesario, [complete el tutorial para habilitar Azure Multi-Factor Authentication](tutorial-enable-azure-mfa.md).

## <a name="overview-of-azure-ad-identity-protection"></a>Introducción a Azure AD Identity Protection

Cada día, Microsoft recopila y analiza billones de señales anónimas como parte de los intentos de inicio de sesión de usuario. Estas señales ayudan a crear patrones de un comportamiento de inicio de sesión de usuario correcto e identifican posibles intentos de inicio de sesión con riesgo. Azure AD Identity Protection puede revisar los intentos de inicio de sesión de los usuarios y tomar medidas adicionales si se produce un comportamiento sospechoso:

Algunas de las acciones siguientes pueden desencadenar la detección de riesgos de Azure AD Identity Protection:

* Usuarios con credenciales filtradas.
* Inicios de sesión desde direcciones IP anónimas.
* Viaje imposible a ubicaciones inusuales.
* Inicios de sesión desde dispositivos infectados.
* Inicios de sesión desde direcciones IP con actividad sospechosa.
* Inicios de sesión desde ubicaciones desconocidas.

Las tres directivas siguientes están disponibles en Azure AD Identity Protection para proteger a los usuarios y responder frente a actividades sospechosas. Puede optar por activar o desactivar la aplicación de directivas, seleccionar los usuarios o grupos a los que se aplicará la directiva y decidir si desea bloquear el acceso en el inicio de sesión o solicitar una acción adicional.

* Directiva de riesgo de usuario
    * Identifica y responde en caso de cuentas de usuario que pudieran tener credenciales en peligro. Puede solicitar al usuario que cree una nueva contraseña.
* Directiva de riesgo de inicio de sesión
    * Identifica y responde en caso de intentos de inicio de sesión sospechosos. Puede solicitar al usuario que proporcione otras formas de verificación mediante Azure Multi-Factor Authentication.
* Directiva de registro de MFA
    * Se asegura de que los usuarios se hayan registrado en Azure Multi-Factor Authentication. Si una directiva de riesgo de inicio de sesión solicita MFA, el usuario ya debe estar registrado en Azure Multi-Factor Authentication.

Cuando habilita una directiva de riesgo de usuario o de riesgo de inicio de sesión, también puede elegir el umbral para el nivel de riesgo: *bajo y superiores*, *medio y superiores* o *alto*. Esta flexibilidad le permite decidir cómo desea que se apliquen los controles a los eventos de inicio de sesión sospechosos.

Para más información acerca de Azure AD Identity Protection, consulte [¿Qué es Azure AD Identity Protection?](../identity-protection/overview-identity-protection.md)

## <a name="enable-mfa-registration-policy"></a>Habilitación de la directiva de registro de MFA

Azure AD Identity Protection incluye una directiva predeterminada que puede ayudar a que los usuarios se registren en Azure Multi-Factor Authentication. Si usa directivas adicionales para proteger los eventos de inicio de sesión, necesitará que los usuarios ya estén registrados en MFA. Cuando se habilita esta directiva, no es necesario que los usuarios realicen la autenticación multifactor en cada evento de inicio de sesión. La directiva solo comprueba el estado del registro de un usuario y le pide que se registre previamente si es necesario.

Se recomienda habilitar la directiva de registro de MFA para aquellos usuarios para los que se van a habilitar directivas adicionales de Azure AD Identity Protection. Para habilitar esta directiva, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta de administrador global.
1. Busque y seleccione **Azure Active Directory**, seleccione **Seguridad** y, a continuación, elija **Identity Protection** en el encabezado de menú *Proteger*.
1. Seleccione **Directiva de registro de MFA** en el menú de la izquierda.
1. De forma predeterminada, la directiva se aplica a *Todos los usuarios*. Si lo desea, seleccione **Asignaciones** y, a continuación, elija los usuarios o grupos a los que desea aplicar la directiva.
1. En *Controles*, seleccione **Acceso**. Asegúrese de que la opción *Requerir registro de Azure MFA* está activada y, a continuación, elija **Seleccionar**.
1. Establezca **Aplicar directiva** en *Activado* y, a continuación, seleccione **Guardar**.

    ![Captura de pantalla sobre cómo requerir que los usuarios se registren en MFA en Azure Portal](./media/tutorial-risk-based-sspr-mfa/enable-mfa-registration.png)

## <a name="enable-user-risk-policy-for-password-change"></a>Habilitación de la directiva de riesgo de usuario para el cambio de contraseña

Microsoft trabaja con los investigadores, las autoridades judiciales, varios equipos de seguridad de Microsoft y otros orígenes de confianza para buscar pares de nombre de usuario y contraseña. Cuando uno de estos pares coincide con una cuenta de su entorno, se puede solicitar un cambio de contraseña basado en el riesgo. Esta directiva y acción requieren que el usuario actualice su contraseña para poder iniciar sesión con el fin de asegurarse de que las credenciales expuestas previamente ya no funcionan.

Para habilitar esta directiva, siga estos pasos:

1. Seleccione **Directiva de riesgo de usuario** en el menú de la izquierda.
1. De forma predeterminada, la directiva se aplica a *Todos los usuarios*. Si lo desea, seleccione **Asignaciones** y, a continuación, elija los usuarios o grupos a los que desea aplicar la directiva.
1. En *Condiciones*, elija **Seleccionar condiciones > Seleccionar un nivel de riesgo** y, a continuación, elija *Medio y superior*.
1. Elija **Seleccionar** y, a continuación, elija **Listo**.
1. En *Acceso*, seleccione **Acceso**. Asegúrese de que las opciones **Permitir acceso** y *Requerir cambio de contraseña* están activadas y, a continuación, elija **Seleccionar**.
1. Establezca **Aplicar directiva** en *Activado* y, a continuación, seleccione **Guardar**.

    ![Captura de pantalla sobre cómo habilitar la directiva de riesgo de usuario en Azure Portal](./media/tutorial-risk-based-sspr-mfa/enable-user-risk-policy.png)

## <a name="enable-sign-in-risk-policy-for-mfa"></a>Habilitación de la directiva de riesgo de inicio de sesión para MFA

La mayoría de los usuarios tienen un comportamiento normal del que se puede realizar un seguimiento. Cuando se encuentran fuera de esta norma, podría ser arriesgado permitirles iniciar sesión correctamente. En su lugar, es posible que desee bloquear al usuario o pedirle que realice una autenticación multifactor. Si el usuario completa correctamente el desafío de MFA, puede considerarlo un intento de inicio de sesión válido y conceder acceso a la aplicación o servicio.

Para habilitar esta directiva, siga estos pasos:

1. Seleccione **Directiva de riesgo de inicio de sesión** en el menú de la izquierda.
1. De forma predeterminada, la directiva se aplica a *Todos los usuarios*. Si lo desea, seleccione **Asignaciones** y, a continuación, elija los usuarios o grupos a los que desea aplicar la directiva.
1. En *Condiciones*, elija **Seleccionar condiciones > Seleccionar un nivel de riesgo** y, a continuación, elija *Medio y superior*.
1. Elija **Seleccionar** y, a continuación, elija **Listo**.
1. En *Acceso*, seleccione **Seleccionar un control**. Asegúrese de que las opciones **Permitir acceso** y *Requerir autenticación multifactor* están activadas y, a continuación, elija **Seleccionar**.
1. Establezca **Aplicar directiva** en *Activado* y, a continuación, seleccione **Guardar**.

    ![Captura de pantalla sobre cómo habilitar la directiva de riesgo de inicio de sesión en Azure Portal](./media/tutorial-risk-based-sspr-mfa/enable-sign-in-risk-policy.png)

## <a name="test-risky-sign-events"></a>Prueba de los eventos de inicio de sesión con riesgo

La mayoría de los eventos de inicio de sesión de usuario no desencadenarán las directivas basadas en riesgos configuradas en los pasos anteriores. Es posible que un usuario no vea nunca una solicitud de MFA adicional o para restablecer su contraseña. Si sus credenciales permanecen seguras y su comportamiento es coherente, los eventos de inicio de sesión serán correctos.

Para probar las directivas de Azure AD Identity Protection creadas en los pasos anteriores, necesita una forma de simular comportamientos de riesgo o posibles ataques. Los pasos para realizar estas pruebas varían en función de la directiva de Azure AD Identity Protection que desea validar. Para más información sobre escenarios y pasos, consulte [Simulación de detecciones de riesgos en Azure AD Identity Protection](../identity-protection/howto-identity-protection-simulate-risk.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha completado las pruebas y ya no desea tener habilitadas las directivas basadas en riesgos, vuelva a cada directiva que desee deshabilitar y establezca **Aplicar directiva** en *Desactivado*.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha habilitado las directivas de usuario basadas en riesgos para Azure AD Identity Protection. Ha aprendido a:

> [!div class="checklist"]
> * Descripción de las directivas disponibles para Azure AD Identity Protection
> * Habilitación del registro de Azure Multi-Factor Authentication
> * Habilitación de los cambios de contraseña en función del riesgo
> * Habilitación de la autenticación multifactor en función del riesgo
> * Prueba de las directivas basadas en riesgos para los intentos de inicio de sesión de usuario

> [!div class="nextstepaction"]
> [Más información sobre Azure AD Identity Protection](../identity-protection/overview-identity-protection.md)
