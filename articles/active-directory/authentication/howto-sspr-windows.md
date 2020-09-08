---
title: 'Autoservicio de restablecimiento de contraseña para dispositivos Windows: Azure Active Directory'
description: Obtenga información sobre cómo habilitar el autoservicio de restablecimiento de contraseña de Azure Active Directory en la pantalla de inicio de sesión de Windows.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/17/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: a51d8c45f652173e5b2b0731d64a8e6f14ee46c7
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88717360"
---
# <a name="enable-azure-active-directory-self-service-password-reset-at-the-windows-sign-in-screen"></a>Habilitación del autoservicio de restablecimiento de contraseña de Azure Active Directory en la pantalla de inicio de sesión de Windows

El autoservicio de restablecimiento de contraseña (SSPR) ofrece a los usuarios de Azure Active Directory (Azure AD) la posibilidad de cambiar o restablecer su contraseña sin necesidad de que intervenga el administrador o el departamento de soporte técnico. Normalmente, los usuarios abren un explorador web en otro dispositivo para acceder al [portal de SSPR](https://aka.ms/sspr). Para mejorar la experiencia en equipos que ejecutan Windows 7, 8, 8.1 y 10, puede permitir que los usuarios restablezcan su contraseña en la pantalla de inicio de sesión de Windows.

![Pantallas de inicio de sesión de Windows 7 y 10 de ejemplo en las que se muestra el vínculo de SSPR](./media/howto-sspr-windows/windows-reset-password.png)

> [!IMPORTANT]
> En este tutorial se muestra a un administrador cómo habilitar SSPR para dispositivos Windows en una empresa.
>
> Si el equipo de TI no ha habilitado la capacidad de usar SSPR desde un dispositivo Windows o si tiene problemas para iniciar sesión, póngase en contacto con el departamento de soporte técnico para obtener ayuda adicional.

## <a name="general-limitations"></a>Limitaciones generales

Las siguientes limitaciones se aplican al uso de SSPR desde la pantalla de inicio de sesión de Windows:

- El restablecimiento de contraseña no se admite actualmente desde Escritorio remoto ni sesiones mejoradas de Hyper-V.
- Se sabe que algunos proveedores de credenciales externos provocan problemas con esta característica.
- Se sabe que la deshabilitación de UAC mediante la modificación de la [clave del registro EnableLUA](/openspecs/windows_protocols/ms-gpsb/958053ae-5397-4f96-977f-b7700ee461ec) provoca problemas.
- Esta característica no funciona para las redes con la autenticación de red 802.1x implementada y la opción "Realizar inmediatamente antes de que el usuario inicie sesión". Para las redes con la autenticación de red 802.1X implementada se recomienda usar la autenticación del equipo para habilitar esta característica.
- Los equipos unidos a Azure AD híbrido deben tener una línea de visión de conectividad de red a un controlador de dominio para usar las nuevas credenciales en caché de contraseña y actualización. Esto significa que los dispositivos deben estar en la red interna de la organización o en una VPN con acceso de red a un controlador de dominio local.
- Si se usa una imagen, antes de ejecutar sysprep, asegúrese de que se borra la caché web para la cuenta predefinida de administrador antes de realizar el paso de CopyProfile. Puede encontrar más información sobre este paso en el artículo de soporte técnico [Rendimiento deficiente cuando se usa el perfil de usuario predeterminado personalizado](https://support.microsoft.com/help/4056823/performance-issue-with-custom-default-user-profile).
- Se sabe que los valores siguientes interfieren con la capacidad de usar y restablecer contraseñas en dispositivos con Windows 10:
    - Si en versiones de Windows 10 anteriores a la 1809 la directiva requiere Ctrl+Alt+Supr, no funcionará **Restablecer contraseña**.
    - Si las notificaciones de la pantalla de bloqueo están desactivadas, tampoco funcionará **Restablecer contraseña**.
    - *HideFastUserSwitching* está establecido en habilitado o 1.
    - *DontDisplayLastUserName* está establecido en habilitado o 1.
    - *NoLockScreen* está establecido en habilitado o 1.
    - *EnableLostMode* se establece en el dispositivo.
    - Explorer.exe se reemplaza por un shell personalizado
- La combinación de las tres configuraciones específicas siguientes puede hacer que esta característica no funcione.
    - Inicio de sesión interactivo: No requiere CTRL + ALT + SUPR = Deshabilitado
    - *DisableLockScreenAppNotifications* = 1 o Habilitado
    - La SKU de Windows no es Home o Professional Edition.

## <a name="windows-10-password-reset"></a>Restablecimiento de contraseña de Windows 10

Para configurar un dispositivo con Windows 10 para SSPR en la pantalla de inicio de sesión, revise los requisitos previos y pasos de configuración a continuación.

### <a name="windows-10-prerequisites"></a>Requisitos previos de Windows 10

- Un administrador [debe habilitar el autoservicio de restablecimiento de contraseña de Azure AD desde Azure Portal](tutorial-enable-sspr.md).
- Los usuarios deben registrarse para SSPR antes de usar esta característica en [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup)
    - No solo en el uso de SSPR desde la pantalla de inicio de sesión de Windows, todos los usuarios deben proporcionar la información de contacto de autenticación para poder restablecer su contraseña.
- Requisitos del proxy de red:
    - Puerto 443 para `passwordreset.microsoftonline.com` y `ajax.aspnetcdn.com`
    - Los dispositivos con Windows 10 solo admiten la configuración de proxy de nivel de equipo.
- Ejecute al menos la actualización de abril de 2018 (v1803) de Windows 10, y los dispositivos deben estar:
    - Unido a Azure AD
    - Híbrido unido a Azure AD

### <a name="enable-for-windows-10-using-intune"></a>Habilitación para Windows 10 con Intune

La implementación de cambios de configuración para habilitar SSPR desde la pantalla de inicio de sesión mediante Intune es el método más flexible. Intune le permite implementar el cambio de configuración en un grupo específico de máquinas que defina. Este método requiere la inscripción en Intune del dispositivo.

#### <a name="create-a-device-configuration-policy-in-intune"></a>Creación de una directiva de configuración de dispositivo en Intune

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y selección **Intune**.
1. Cree un nuevo perfil de configuración de dispositivo; para ello, vaya a **Configuración del dispositivo** > **Perfiles**, a continuación, seleccione **+ Crear perfil**.
   - En **Plataforma**, elija *Windows 10 y versiones posteriores*.
   - En **Tipo de perfil**, elija *Personalizado.*
1. Seleccione **Crear** y escriba un nombre descriptivo para el perfil, como *SSPR en la pantalla de inicio de sesión de Windows 10*.

    Opcionalmente, escriba una descripción detallada del perfil y seleccione **Siguiente**.
1. En *Opciones de configuración*, seleccione **Agregar** y proporcione la siguiente configuración OMA-URI para habilitar el vínculo de restablecimiento de contraseña:
      - Escriba un nombre descriptivo para explicar lo que hace la opción, como *Agregar vínculo de SSPR*.
      - De manera opcional, escriba una descripción detallada de la opción.
      - **OMA-URI** establecido en `./Vendor/MSFT/Policy/Config/Authentication/AllowAadPasswordReset`
      - **Tipo de datos** establecido en **Entero**
      - **Valor** establecido en **1**

    Seleccione **Agregar** y, luego, **Siguiente**.
1. La directiva se puede asignar a usuarios, dispositivos o grupos específicos. Asigne primero el perfil que desee para su entorno, idealmente a un grupo de prueba de dispositivos y, a continuación, seleccione **Siguiente**.

    Para más información, consulte [Asignación de perfiles de usuario y de dispositivo en Microsoft Intune](/mem/intune/configuration/device-profile-assign).

1. Configure las reglas de aplicabilidad como desee para su entorno, por ejemplo, para *Assign profile if OS edition is Windows 10 Enterprise* (Asignar perfil si la edición del SO es Windows 10 Enterprise) y, después, seleccione **Siguiente**.
1. Revise el perfil y seleccione **Crear**.

### <a name="enable-for-windows-10-using-the-registry"></a>Habilitación para Windows 10 con el Registro

Para habilitar SSPR en la pantalla de inicio de sesión mediante una clave del registro, siga los pasos a continuación:

1. Inicie sesión en el equipo Windows con credenciales administrativas
1. Presione **Windows** + **R** para abrir el cuadro de diálogo *Ejecutar* y, a continuación, ejecute **regedit** como administrador.
1. Establezca la siguiente clave del Registro:

    ```cmd
    HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\AzureADAccount
       "AllowPasswordReset"=dword:00000001
    ```

#### <a name="troubleshooting-windows-10-password-reset"></a>Solución de problemas de restablecimiento de contraseña de Windows 10

Si tiene problemas con el uso de SSPR desde la pantalla de inicio de sesión de Windows, el registro de auditoría de Azure AD incluye información sobre la dirección IP y *ClientType* donde se produjo el restablecimiento de la contraseña, como se muestra en la siguiente salida de ejemplo:

![Ejemplo de restablecimiento de contraseña de Windows 7 en el registro de auditoría de Azure AD](media/howto-sspr-windows/windows-7-sspr-azure-ad-audit-log.png)

Cuando los usuarios restablecen su contraseña desde la pantalla de inicio de sesión de un dispositivo con Windows 10, se crea una cuenta temporal con pocos privilegios denominada `defaultuser1`. Esta cuenta se usa para proteger el proceso de restablecimiento de contraseña.

La propia cuenta tiene una contraseña generada aleatoriamente, no se muestra en el inicio de sesión del dispositivo y se elimina automáticamente después de que el usuario restablece su contraseña. Pueden existir varios perfiles `defaultuser`, pero se pueden omitir con seguridad.

## <a name="windows-7-8-and-81-password-reset"></a>Restablecimiento de contraseña de Windows 7, 8 y 8.1

Para configurar un dispositivo con Windows 7, 8 u 8.1 para SSPR en la pantalla de inicio de sesión, revise los requisitos previos y pasos de configuración a continuación.

### <a name="windows-7-8-and-81-prerequisites"></a>Requisitos previos de Windows 7, 8 y 8.1

- Un administrador [debe habilitar el autoservicio de restablecimiento de contraseña de Azure AD desde Azure Portal](tutorial-enable-sspr.md).
- Los usuarios deben registrarse para SSPR antes de usar esta característica en [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup)
    - No solo en el uso de SSPR desde la pantalla de inicio de sesión de Windows, todos los usuarios deben proporcionar la información de contacto de autenticación para poder restablecer su contraseña.
- Requisitos del proxy de red:
    - Puerto 443 para `passwordreset.microsoftonline.com`
- Sistema operativo Windows 7 o Windows 8.1 revisado.
- TLS 1.2 habilitada mediante las instrucciones que se encuentran en [Configuración del registro de TLS](/windows-server/security/tls/tls-registry-settings#tls-12).
- Si en el equipo hay más de un proveedor de credenciales de terceros habilitado, los usuarios ven más de un perfil de usuario en la pantalla de inicio de sesión.

> [!WARNING]
> Se debe habilitar TLS 1.2, no hay solo que establecerla en negociación automática.

### <a name="install"></a>Instalar

En Windows 7, 8 y 8,1, se debe instalar un componente pequeño en la máquina para habilitar SSPR en la pantalla de inicio de sesión. Para instalar este componente de SSPR, complete los pasos siguientes:

1. Descargue el instalador adecuado para la versión de Windows que desea habilitar.

    El instalador de software está disponible en el centro de descarga de Microsoft en [https://aka.ms/sspraddin](https://aka.ms/sspraddin)
1. Inicie sesión en la máquina donde desea instalar y ejecutar el instalador.
1. Después de la instalación, se recomienda encarecidamente reiniciar el equipo.
1. Después del reinicio, en la pantalla de inicio de sesión, elija un usuario y seleccione "¿Olvidó la contraseña?" para iniciar el flujo de trabajo de restablecimiento de contraseña.
1. Complete el flujo de trabajo siguiendo los pasos que aparecen en la pantalla para restablecer su contraseña.

![Ejemplo de Windows 7 después de hacer clic en "¿Olvidó la contraseña?" Flujo de SSPR](media/howto-sspr-windows/windows-7-sspr.png)

#### <a name="silent-installation"></a>Instalación silenciosa

El componente de SSPR se puede instalar o desinstalar sin avisos mediante los siguientes comandos:

- Para una instalación silenciosa, use el comando "msiexec /i SsprWindowsLogon.PROD.msi /qn".
- Para una desinstalación silenciosa, use el comando "msiexec /x SsprWindowsLogon.PROD.msi /qn".

#### <a name="troubleshooting-windows-7-8-and-81-password-reset"></a>Solución de problemas de restablecimiento de contraseña de Windows 7, 8 y 8.1

Si tiene problemas con el uso de SSPR desde la pantalla de inicio de sesión de Windows, los eventos se registran tanto en la máquina como en Azure AD. Los eventos de Azure AD incluyen información sobre la dirección IP y el ClientType donde se produjo el restablecimiento de contraseña, como se muestra en la salida de ejemplo siguiente:

![Ejemplo de restablecimiento de contraseña de Windows 7 en el registro de auditoría de Azure AD](media/howto-sspr-windows/windows-7-sspr-azure-ad-audit-log.png)

Si se requiere un registro adicional, se puede cambiar una clave del Registro en la máquina para habilitar el registro detallado. Habilite el registro detallado para la solución de problemas únicamente con el siguiente valor de clave del Registro:

```cmd
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Providers\{86D2F0AC-2171-46CF-9998-4E33B3D7FD4F}
```

- Para habilitar el registro detallado, cree un `REG_DWORD: "EnableLogging"` y establézcalo en 1.
- Para deshabilitar el registro detallado, cambie `REG_DWORD: "EnableLogging"` a 0.

## <a name="what-do-users-see"></a>Lo que ven los usuarios

Con SSPR configurado para los dispositivos Windows, ¿qué cambia para el usuario? ¿Cómo sabe que puede restablecer su contraseña en la pantalla de inicio de sesión? Las capturas de pantallas de ejemplo siguientes muestran las opciones adicionales para que un usuario restablezca su contraseña mediante SSPR:

![Pantallas de inicio de sesión de Windows 7 y 10 de ejemplo en las que se muestra el vínculo de SSPR](./media/howto-sspr-windows/windows-reset-password.png)

Cuando los usuarios intenten iniciar sesión, verán un vínculo **Restablecer contraseña** o **¿Olvidó la contraseña?** que abre la experiencia de autoservicio de restablecimiento de contraseña en la pantalla de inicio de sesión. Esta funcionalidad permite a los usuarios restablecer su contraseña sin tener que usar otro dispositivo para acceder a un explorador web.

Los usuarios pueden encontrar más información sobre el uso de esta característica en [Restablecimiento de la contraseña profesional o educativa](../user-help/active-directory-passwords-update-your-own-password.md).

## <a name="next-steps"></a>Pasos siguientes

Para simplificar la experiencia de registro de usuarios, puede [rellenar previamente la información de contacto de autenticación de usuario para SSPR](howto-sspr-authenticationdata.md).