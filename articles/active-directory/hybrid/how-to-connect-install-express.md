---
title: 'Azure AD Connect: introducción al uso de la configuración rápida | Microsoft Docs'
description: Obtenga información acerca de cómo descargar, instalar y ejecutar el asistente para instalación de Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
editor: curtand
ms.assetid: b6ce45fd-554d-4f4d-95d1-47996d561c9f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/28/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0a655f355bb77d937f4daff2f8987769416ebd8c
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279676"
---
# <a name="getting-started-with-azure-ad-connect-using-express-settings"></a>Introducción a Azure AD Connect mediante la configuración rápida
La **configuración rápida** de Azure AD Connect se usa cuando se tiene una topología de bosque único y una [sincronización de hash de contraseñas](how-to-connect-password-hash-synchronization.md) para la autenticación. **Configuración rápida** es la opción predeterminada y se utiliza en los escenarios implementados más habituales. Ya solo le faltan uno cuantos clics para extender su directorio local a la nube.

Antes de empezar a instalarlo, debe [descargar Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771) y completar los pasos de los requisitos previos que se señalan en [Requisitos previos de Azure AD Connect](how-to-connect-install-prerequisites.md).

Si la configuración rápida no coincide con su topología, consulte la [documentación relacionada](#related-documentation) sobre otros escenarios.

## <a name="express-installation-of-azure-ad-connect"></a>Instalación rápida de Azure AD Connect
Puede ver estos pasos de acción en la sección [Vídeos](#videos) .

1. Inicie sesión como administrador local en el servidor en el que quiere instalar Azure AD Connect. Debe ser el servidor que quiere que sea el servidor de sincronización.
2. Navegue hasta **AzureADConnect.msi**y haga doble clic en él.
3. En la pantalla de bienvenida, active la casilla que acepta los términos de licencia y haga clic en **Continuar**.  
4. En la pantalla Configuración rápida, haga clic en **Usar configuración rápida**.  
   ![Bienvenida a Azure AD Connect](./media/how-to-connect-install-express/express.png)
5. En la pantalla Conectar a Azure AD, escriba el nombre de usuario y la contraseña de un administrador global de Azure AD. Haga clic en **Next**.  
   ![Conectarse a Azure AD](./media/how-to-connect-install-express/connectaad.png)  
   Si aparece un error y tiene problemas de conectividad, consulte [Solución de problemas de conectividad con Azure AD Connect](tshoot-connect-connectivity.md).
6. En la pantalla Conectar a AD DS, escriba el nombre de usuario y la contraseña de una cuenta de administrador de empresa. Puede escribir la parte del dominio con el formato NetBios o FQDN, es decir, FABRIKAM\administrator o fabrikam.com\administrator. Haga clic en **Next**.  
   ![Conectarse a AD DS](./media/how-to-connect-install-express/connectad.png)
7. Solo se mostrará la página [**Configuración de inicio de sesión de Azure AD**](plan-connect-user-signin.md#azure-ad-sign-in-configuration) si no completó la [comprobación de dominios](../fundamentals/add-custom-domain.md) en los [requisitos previos](how-to-connect-install-prerequisites.md).
   ![Dominios sin comprobar](./media/how-to-connect-install-express/unverifieddomain.png)  
   Si ve esta página, revise los dominios marcados como **Not Added** (Sin agregar) y **Not Verified** (Sin comprobar). Asegúrese de que los dominios que usa se han comprobado en Azure AD. Cuando haya comprobado los dominios, haga clic en el símbolo de actualización.
8. En la pantalla Listo para configurar, haga clic en **Instalar**.
   * En la página Listo para configurar, puede desactivar la casilla **Inicie el proceso de sincronización en cuanto se complete la configuración**. Si quiere realizar alguna configuración adicional, por ejemplo, el [filtrado](how-to-connect-sync-configure-filtering.md), desactive esta casilla. Si desactiva esta opción, el asistente configura la sincronización, pero deja el programador deshabilitado, y no se ejecuta hasta que lo habilite manualmente [volviendo a ejecutar el Asistente para la instalación](how-to-connect-installation-wizard.md).
   * Si deja la casilla **Inicie el proceso de sincronización cuando se complete la configuración** habilitada, se desencadenará de inmediato una sincronización completa en Azure AD de todos los usuarios, grupos y contactos.
   * Si tiene Exchange en su entorno de Active Directory local, también dispone de la opción para habilitar una [**implementación híbrida de Exchange**](/exchange/exchange-hybrid). Habilite esta opción si planea tener buzones de Exchange tanto en la nube como locales de manera simultánea.
     ![Listo para configurar Azure AD Connect](./media/how-to-connect-install-express/readytoconfigure.png)
9. Una vez completada la instalación, haga clic en **Salir**.
10. Una vez completada la instalación, cierre la sesión e inicie de sesión de nuevo antes de utilizar Synchronization Service Manager o el Editor de reglas de sincronización.

## <a name="videos"></a>Vídeos
Para ver un vídeo sobre el uso de la instalación rápida, consulte:

> [!VIDEO https://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/Azure-Active-Directory-Connect-Express-Settings/player]
>
>

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha instalado Azure AD Connect, puede [comprobar la instalación y asignar licencias](how-to-connect-post-installation.md).

Para aprender más acerca de estas características que se habilitaron con la instalación, consulte la información sobre: [actualización automática](how-to-connect-install-automatic-upgrade.md), [cómo evitar eliminaciones accidentales](how-to-connect-sync-feature-prevent-accidental-deletes.md) y [Azure AD Connect Health](how-to-connect-health-sync.md).

Obtenga información acerca de estos temas habituales: [el programador y cómo desencadenar la sincronización](how-to-connect-sync-feature-scheduler.md).

Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).

## <a name="related-documentation"></a>documentación relacionada

| Tema | Vínculo |
| --- | --- |
| Información general de Azure AD Connect | [Integración de los directorios locales con Azure Active Directory](whatis-hybrid-identity.md)
| Instalación mediante configuración personalizada | [Instalación personalizada de Azure AD Connect](how-to-connect-install-custom.md) |
| Actualización desde DirSync | [Actualización desde la herramienta de sincronización de Azure AD (DirSync)](how-to-dirsync-upgrade-get-started.md)|
| Cuentas usadas para la instalación | [Más información sobre las credenciales y los permisos de Azure AD Connect](reference-connect-accounts-permissions.md) |