---
title: 'Incorporación de una cuenta profesional o educativa a la aplicación Microsoft Authenticator: Azure AD'
description: Agregue su cuenta profesional o educativa a la aplicación Microsoft Authenticator para comprobar su identidad al usar la verificación en dos fases.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 01/24/2019
ms.author: curtand
ms.reviewer: olhaun
ms.openlocfilehash: e003c45aa1e7d75b709b7fbf99532fb1302fcbb9
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797656"
---
# <a name="add-your-work-or-school-account-to-the-microsoft-authenticator-app"></a>Agregar la cuenta profesional o educativa a la aplicación Microsoft Authenticator

Si su organización usa la verificación en dos pasos, puede configurar su cuenta profesional o educativa para usar la aplicación Microsoft Authenticator como uno de los métodos de verificación.

>[!Important]
>Antes de agregar su cuenta, debe descargar e instalar la aplicación Microsoft Authenticator. Si no lo ha hecho aún, siga los pasos descritos en el artículo para [descargar e instalar la aplicación](user-help-auth-app-download-install.md).

## <a name="add-your-work-or-school-account"></a>Agregar una cuenta profesional o educativa

1. En el equipo, vaya a la página [Comprobación de seguridad adicional](https://account.activedirectory.windowsazure.com/proofup.aspx?proofup=1).

    >[!Note]
    >Si no ve la página **Comprobación de seguridad adicional**, es posible que el administrador haya activado la experiencia de información de seguridad (versión preliminar). Si es así, debe seguir las instrucciones de la sección [Configuración de la información de seguridad para usar una aplicación autenticadora](security-info-setup-auth-app.md). Si no es así, deberá ponerse en contacto con el departamento de soporte técnico de su organización para obtener ayuda. Para más información sobre seguridad, consulte [Introducción a la información de seguridad (versión preliminar)](./security-info-setup-signin.md).

2. Active la casilla situada junto a **Aplicación Authenticator** y luego seleccione **Configurar**.

    Aparece la página **Configurar aplicación móvil**.

    ![Pantalla que proporciona el código QR](./media/user-help-auth-app-download-install/auth-app-barcode.png)

3. Abra la aplicación Microsoft Authenticator, seleccione **Agregar cuenta** en el icono **Personalizar y controlar** en la esquina superior derecha y, a continuación, seleccione **Cuenta profesional o educativa**.

    >[!Note]
    >Si es la primera vez que configura la aplicación Microsoft Authenticator, es posible que reciba un mensaje en el que se le pregunta si quiere permitir que la aplicación acceda a la cámara (iOS) o permitir que la aplicación tome fotografías y grabe vídeo (Android). Debe seleccionar **Permitir** para que la aplicación de autenticación pueda acceder a la cámara y tomar una fotografía del código QR en el paso siguiente. Aunque no otorgue permiso a la cámara, podrá configurar la aplicación de autenticación, pero tendrá que agregar la información de código manualmente. Para más información sobre cómo agregar el código manualmente, consulte [Agregar manualmente una cuenta a la aplicación](user-help-auth-app-add-account-manual.md).

4. Use la cámara del dispositivo para detectar el código QR desde la pantalla **Configurar aplicación móvil** del equipo y, después, elija **Listo**.

    >[!Note]
    >Si la cámara no puede capturar el código QR, puede agregar manualmente la información de su cuenta a la aplicación Microsoft Authenticator para realizar la verificación en dos fases. Para obtener más información y saber cómo hacerlo, consulte [Manually add your account](user-help-auth-app-add-account-manual.md) (Adición manual de la cuenta).

5. Revise la pantalla **Cuentas** de la aplicación en el dispositivo para asegurarse de que la cuenta es correcta y de que hay un código de verificación de seis dígitos asociado. Para mayor seguridad, el código de verificación cambia cada 30 segundos para impedir que se pueda usar el mismo código varias veces.

    ![Pantalla Cuentas](./media/user-help-auth-app-download-install/auth-app-accounts.png)

## <a name="next-steps"></a>Pasos siguientes

- Después de agregar las cuentas a la aplicación, puede iniciar sesión con la aplicación Authenticator en su dispositivo. Para más información, consulte cómo [iniciar sesión con la aplicación](user-help-auth-app-sign-in.md).

- En cuanto a los dispositivos con iOS, también puede hacer una copia de seguridad en la nube de las credenciales de su cuenta y de la configuración relacionada de la aplicación, como el orden de las cuentas. Para obtener más información, consulte [Copia de seguridad y recuperación de las credenciales de cuenta con la aplicación Microsoft Authenticator](user-help-auth-app-backup-recovery.md).