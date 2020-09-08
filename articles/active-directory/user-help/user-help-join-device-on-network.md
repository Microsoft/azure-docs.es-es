---
title: 'Unión del dispositivo de trabajo a la red de la organización: AD'
description: Aprenda a unir su dispositivo de trabajo a la red de su organización.
services: active-directory
author: curtand
manager: daveba
ms.assetid: 54e1b01b-03ee-4c46-bcf0-e01affc0419d
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 08/03/2018
ms.author: curtand
ms.reviewer: jairoc
ms.openlocfilehash: 17c0b7bd802d4a50343f88b9517313489943e10c
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799492"
---
# <a name="join-your-work-device-to-your-organizations-network"></a>Una su dispositivo de trabajo a la red de su organización
Una su dispositivo Windows 10 de trabajo a la red de su organización para poder tener acceso a recursos potencialmente restringidos.

## <a name="what-happens-when-you-join-your-device"></a>Qué ocurre al unir su dispositivo
Durante la unión de su dispositivo Windows 10 a la red de su organización, se producirán las acciones siguientes:

- Windows registra su dispositivo en la red de su organización, lo que le permite tener acceso a sus recursos con su cuenta personal. Una vez registrado su dispositivo, Windows une su dispositivo a la red para que pueda usar el nombre de usuario y la contraseña de su organización para iniciar sesión y tener acceso a recursos restringidos.

- Opcionalmente, en función de las opciones de su organización, puede que se le pida configurar una verificación en dos pasos a través de [Multi-Factor Authentication](multi-factor-authentication-end-user-first-time.md) o [información de seguridad](./security-info-setup-signin.md).

- Opcionalmente, en función de las opciones de su organización, puede que se le inscriba automáticamente en la administración de dispositivos móviles, como Microsoft Intune. Para obtener más información sobre cómo inscribirse en Microsoft Intune, consulte [Enroll your device in Intune](/intune-user-help/enroll-your-device-in-intune-all) (Inscribir su dispositivo en Intune).

- Seguirá el proceso de inicio de sesión, haciendo uso del inicio de sesión automático con la cuenta de su organización.

## <a name="to-join-a-brand-new-windows-10-device"></a>Para unir un nuevo dispositivo Windows 10
Si su dispositivo es nuevo y no se ha configurado aún, puede seguir el proceso de experiencia inmediata (OOBE) de Windows para unir su dispositivo a la red.

1. Configure su nuevo dispositivo e inicie el proceso de experiencia inmediata.

2. En la pantalla **Iniciar sesión con Microsoft**, escriba su dirección de correo electrónico laboral o académico.

    ![Pantalla de inicio de sesión con dirección de correo electrónico](./media/user-help-join-device-on-network/join-device-oobe-signin.png)

3. En la pantalla **Enter your password** (Escribir su contraseña), escriba su contraseña.

    ![Pantalla Enter your password (Escribir su contraseña)](./media/user-help-join-device-on-network/join-device-oobe-password.png)

4. En su dispositivo móvil, apruebe su dispositivo para que pueda tener acceso a su cuenta. 

    ![Pantalla Mobile notification (Notificación móvil)](./media/user-help-join-device-on-network/join-device-oobe-mobile.png)

5. Complete el proceso de experiencia inmediata, incluidos el establecimiento de su configuración de privacidad y la configuración de Windows Hello (si es necesario).

    Ahora su dispositivo se encuentra unido a la red de su organización.

## <a name="to-make-sure-youre-joined"></a>Para asegurarse de que se ha unido
Puede asegurarse de que se ha unido fijándose en su configuración.

1. Abra **Configuración** y, a continuación, seleccione **Cuentas**.

    ![Cuentas en la pantalla Configuración](./media/user-help-join-device-on-network/join-device-settings-accounts.png)

2. Seleccione **Obtener acceso a trabajo o escuela** y asegúrese de que ve texto que indica algo como **Conectado a Azure AD de *\<your_organization>***.

    ![Pantalla Obtener acceso a trabajo o escuela con cuenta Contoso conectada](./media/user-help-join-device-on-network/join-device-oobe-verify.png)


## <a name="to-join-an-already-configured-windows-10-device"></a>Para unir un dispositivo Windows 10 ya configurado
Si ha tenido su dispositivo un tiempo y ya está configurado, puede seguir estos pasos para unir su dispositivo a la red.

1. Abra **Configuración** y, a continuación, seleccione **Cuentas**.

2. Seleccione **Obtener acceso a trabajo o escuela** y, a continuación, seleccione **Conectar**.

    ![Vínculos Obtener acceso a trabajo o escuela y Conectar](./media/user-help-join-device-on-network/join-device-access-work-school-connect.png)

3. En la pantalla **Configurar una cuenta profesional o educativa**, seleccione **Unir este dispositivo a Azure Active Directory**.

    ![Pantalla Configurar una cuenta profesional o educativa](./media/user-help-join-device-on-network/join-device-setup-join-aad.png)

4. En la pantalla **Vamos a iniciar su sesión**, escriba su dirección de correo electrónico (por ejemplo, alain@contoso.com) y, a continuación, seleccione **Siguiente**.

    ![Pantalla Vamos a iniciar su sesión](./media/user-help-join-device-on-network/join-device-setup-get-signed-in.png)

5. En la pantalla **Escribir contraseña**, escriba la contraseña y después seleccione **Iniciar sesión**.

    ![Escribir contraseña](./media/user-help-join-device-on-network/join-device-setup-password.png)

6. En su dispositivo móvil, apruebe su dispositivo para que pueda tener acceso a su cuenta. 

    ![Pantalla Mobile notification (Notificación móvil)](./media/user-help-join-device-on-network/join-device-setup-mobile.png)

7. En la pantalla  **Asegúrate de que esta es tu organización**, revise la información para asegurarse de que es correcta y, a continuación, seleccione **Combinar**.

    ![Pantalla de comprobación Asegúrate de que esta es tu organización](./media/user-help-join-device-on-network/join-device-setup-confirm.png)

8. En la pantalla **Está todo listo**, haga clic en **Listo**.

    ![Pantalla Está todo listo](./media/user-help-join-device-on-network/join-device-setup-finish.png)

## <a name="to-make-sure-youre-joined"></a>Para asegurarse de que se ha unido
Puede asegurarse de que se ha unido fijándose en su configuración.

1. Abra **Configuración** y, a continuación, seleccione **Cuentas**.

    ![Cuentas en la pantalla Configuración](./media/user-help-join-device-on-network/join-device-settings-accounts.png)

2. Seleccione **Obtener acceso a trabajo o escuela** y asegúrese de que ve texto que indica algo como **Conectado a Azure AD de *\<your_organization>***.

    ![Pantalla Obtener acceso a trabajo o escuela con cuenta Contoso conectada](./media/user-help-join-device-on-network/join-device-setup-verify.png)

## <a name="next-steps"></a>Pasos siguientes
Después de unir su dispositivo a la red de su organización, debe poder tener acceso a todos sus recursos con la información de su cuenta profesional o educativa.

- Si su organización quiere que registre su dispositivo personal, por ejemplo, su teléfono, consulte [Register your personal device on your organization's network](user-help-register-device-on-network.md) (Registre su dispositivo personal en la red de su organización).

- Si su organización se administra con Microsoft Intune y tiene alguna pregunta sobre la inscripción, el inicio de sesión o cualquier otro problema relacionado con Intune, consulte el [contenido de la Ayuda de usuario de Intune](/intune-user-help/use-managed-devices-to-get-work-done).