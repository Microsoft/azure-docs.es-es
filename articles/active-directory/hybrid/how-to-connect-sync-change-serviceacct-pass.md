---
title: 'Sincronización de Azure AD Connect:  Cambiar la cuenta de servicio de ADSync | Microsoft Docs'
description: En este documento del tema se describe la clave de cifrado y cómo abandonarla una vez cambiada la contraseña.
services: active-directory
keywords: Cuenta del servicio Sincronización de Azure AD, contraseña
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 76b19162-8b16-4960-9e22-bd64e6675ecc
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/02/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: ce4c64f0be61c2fe28a102674929333235ee29c8
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87385099"
---
# <a name="changing-the-adsync-service-account-password"></a>Cambiar la contraseña de la cuenta de servicio de ADSync
Si cambia la contraseña de la cuenta de servicio de ADSync, el servicio de sincronización no podrá iniciarse correctamente hasta que haya abandonado la clave de cifrado y reinicializado la contraseña de la cuenta de servicio de ADSync. 

Azure AD Connect, como parte de los servicios de sincronización, usa una clave de cifrado para almacenar las contraseñas de las cuentas de los servicios de conector de AD DS y la cuenta de servicio de ADSync.  Estas cuentas se cifran antes de almacenarse en la base de datos. 

La clave de cifrado usada se protege mediante la [API de protección de datos de Windows (DPAPI)](https://msdn.microsoft.com/library/ms995355.aspx). Para proteger la clave de cifrado, DPAPI usa la **cuenta de servicio ADSync**. 

Si tiene que cambiar la contraseña de la cuenta de servicio, puede seguir para ello los procedimientos descritos en [Abandonar la clave de cifrado de la cuenta de servicio de ADSync](#abandoning-the-adsync-service-account-encryption-key).  También debe seguir estos procedimientos si por algún motivo tiene que abandonar la clave de cifrado.

## <a name="issues-that-arise-from-changing-the-password"></a>Problemas que surgen al cambiar la contraseña
Es preciso hacer dos cosas al cambiar la contraseña de la cuenta del servicio.

En primer lugar, tiene que cambiar la contraseña en el Administrador de control de servicios de Windows.  Hasta que se solucione el problema, verá los siguientes errores:


- Si intenta iniciar el servicio de sincronización en el Administrador de control de servicios de Windows, recibirá el error "**Windows no pudo iniciar el servicio Sincronización de Microsoft Azure AD en el equipo local**". **Error 1069: No se puede iniciar el servicio debido a un error en el inicio de sesión.** "
- En el Visor de eventos de Windows, el registro de eventos del sistema contiene un error con el **identificador de evento 7038** y el mensaje "**The ADSync service was unable to log on as with the currently configured password due to the following error: The user name or password is incorrect.** " (El servicio ADSync no puede iniciar sesión con la contraseña configurada actualmente debido al siguiente error: El nombre de usuario o la contraseña es incorrecto).

En segundo lugar, en determinadas condiciones, si la contraseña se actualiza, el servicio de sincronización ya no podrá recuperar la clave de cifrado a través de DPAPI. Sin la clave de cifrado, el servicio de sincronización no puede descifrar la contraseña necesaria para sincronizar con o desde AD y Azure AD locales.
Verá errores como los siguientes:

- En el Administrador de control de servicios de Windows, si intenta iniciar el servicio de sincronización y este no puede recuperar la clave de cifrado, se produce el error “<strong>Windows no pudo iniciar el servicio Sincronización de Microsoft Azure AD en el equipo local. Para más información, revise el registro de eventos del sistema. Si este no es un servicio de Microsoft, póngase en contacto con el proveedor del servicio y haga referencia al código de error específico del servicio -21451857952</strong>”.
- En el Visor de eventos de Windows, el registro de eventos de la aplicación contiene un error con **id. de evento 6028** y el mensaje de error *"The server encryption key cannot be accessed."* (No se puede acceder a la clave de cifrado del servidor.)

Para asegurarse de que no recibe estos errores, siga los procedimientos descritos en [Abandonar la clave de cifrado de la cuenta de servicio de ADSync](#abandoning-the-adsync-service-account-encryption-key) al cambiar la contraseña.
 
## <a name="abandoning-the-adsync-service-account-encryption-key"></a>Abandonar la clave de cifrado de la cuenta de servicio ADSync
>[!IMPORTANT]
>Los procedimientos siguientes solo se aplican a Azure AD Connect compilación 1.1.443.0 o anterior. No se puede usar para las versiones más recientes de Azure AD Connect.

Use los procedimientos siguientes para abandonar la clave de cifrado.

### <a name="what-to-do-if-you-need-to-abandon-the-encryption-key"></a>Qué hacer si tiene que abandonar la clave de cifrado

Si tiene que abandonar la clave de cifrado, use para ello los procedimientos siguientes.

1. [Detener el servicio de sincronización](#stop-the-synchronization-service)

1. [Abandonar la clave de cifrado existente](#abandon-the-existing-encryption-key)

2. [Especificar la contraseña de la cuenta de conector de AD DS](#provide-the-password-of-the-ad-ds-connector-account)

3. [Reinicializar la contraseña de la cuenta de servicio de ADSync](#reinitialize-the-password-of-the-adsync-service-account)

4. [Iniciar el servicio de sincronización](#start-the-synchronization-service)

#### <a name="stop-the-synchronization-service"></a>Detener el servicio de sincronización
En primer lugar, puede detener el servicio en el Administrador de control de servicios de Windows.  Asegúrese de que el servicio no esté en ejecución cuando intente detenerlo.  Si lo está, espere a que termine y después deténgalo.


1. Vaya a Administrador de control de servicios de Windows (INICIO → Servicios).
2. Seleccione **Sincronización de Microsoft Azure AD** y haga clic en Detener.

#### <a name="abandon-the-existing-encryption-key"></a>Abandonar la clave de cifrado existente
Abandone la clave de cifrado existente para poder crear otra clave de cifrado:

1. Inicie sesión como administrador en el servidor de Azure AD Connect.

2. Inicie una nueva sesión de PowerShell.

3. Acceda a la carpeta: `'$env:ProgramFiles\Microsoft Azure AD Sync\bin\'`

4. Ejecute el comando: `./miiskmu.exe /a`

![Utilidad de clave de cifrado de sincronización de Azure AD Connect](./media/how-to-connect-sync-change-serviceacct-pass/key5.png)

#### <a name="provide-the-password-of-the-ad-ds-connector-account"></a>Especificar la contraseña de la cuenta de conector de AD DS
Cuando las contraseñas existentes almacenadas en la base de datos ya no se pueden descifrar, tiene que proporcionar el servicio de sincronización con la contraseña de la cuenta de conector de AD DS. El servicio de sincronización cifra las contraseñas con la nueva clave de cifrado:

1. Inicie el Synchronization Service Manager (INICIO → Synchronization Service).
</br>![Sync Service Manager](./media/how-to-connect-sync-change-serviceacct-pass/startmenu.png)  
2. Vaya a la pestaña **Conectores**.
3. Seleccione el **AD Connector** (conector de AD) que corresponda a su AD local. Si tiene más de un conector de AD, repita los pasos siguientes para cada uno de ellos.
4. En **Acciones**, seleccione **Propiedades**.
5. En el cuadro de diálogo emergente, seleccione **Connect to Active Directory Forest** (Conectar con el bosque de Active Directory):
6. Escriba la contraseña de la cuenta de AD DS en el cuadro de texto **Contraseña**. Si no conoce la contraseña, debe establecerla en un valor conocido antes de realizar este paso.
7. Haga clic en **OK** (Aceptar) para guardar la nueva contraseña y cerrar el cuadro de diálogo emergente.
![Utilidad de clave de cifrado de sincronización de Azure AD Connect](./media/how-to-connect-sync-change-serviceacct-pass/key6.png)

#### <a name="reinitialize-the-password-of-the-adsync-service-account"></a>Reinicializar la contraseña de la cuenta de servicio de ADSync
No se puede especificar directamente la contraseña de la cuenta del servicio Azure AD en el servicio de sincronización. En su lugar, tendrá que usar el cmdlet **Add-ADSyncAADServiceAccount** para reinicializar la cuenta del servicio Azure AD. El cmdlet restablece la contraseña de la cuenta y la pone a disposición de servicio de sincronización:

1. Inicie una nueva sesión de PowerShell en el servidor de Azure AD Connect.
2. Ejecute el cmdlet `Add-ADSyncAADServiceAccount`.
3. En el cuadro de diálogo emergente, especifique las credenciales de administrador global de Azure AD para el inquilino de Azure AD.
![Utilidad de clave de cifrado de sincronización de Azure AD Connect](./media/how-to-connect-sync-change-serviceacct-pass/key7.png)
4. Si se realiza correctamente, verá el símbolo del sistema de PowerShell.

#### <a name="start-the-synchronization-service"></a>Iniciar el servicio de sincronización
Ahora que el servicio de sincronización tiene acceso a la clave de cifrado y a todas las contraseñas que necesita, puede reiniciar el servicio en el Administrador de control de servicios de Windows:


1. Vaya a Administrador de control de servicios de Windows (INICIO → Servicios).
2. Seleccione **Sincronización de Microsoft Azure AD** y haga clic en Reiniciar.

## <a name="next-steps"></a>Pasos siguientes
**Temas de introducción**

* [Sincronización de Azure AD Connect: comprender y personalizar la sincronización](how-to-connect-sync-whatis.md)

* [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md)
