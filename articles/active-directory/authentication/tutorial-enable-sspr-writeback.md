---
title: Habilitación de la escritura diferida de contraseñas de Azure Active Directory
description: En este tutorial, aprenderá a habilitar la escritura diferida del autoservicio de restablecimiento de contraseña mediante Azure AD Connect para sincronizar los cambios de nuevo en un entorno local de Active Directory Domain Services.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: tutorial
ms.date: 07/13/2020
ms.author: iainfou
author: iainfoulds
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.custom: contperfq4
ms.openlocfilehash: f4687a98bfb58db8ed5e8a853fa0db2207f435e8
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87419655"
---
# <a name="tutorial-enable-azure-active-directory-self-service-password-reset-writeback-to-an-on-premises-environment"></a>Tutorial: Habilitación de la escritura diferida del autoservicio de restablecimiento de contraseña de Azure Active Directory en un entorno local

Con el autoservicio de restablecimiento de contraseña de Azure Active Directory (Azure AD), los usuarios pueden actualizar sus contraseñas o desbloquear la cuenta mediante un explorador web. En un entorno híbrido en el que se conecta Azure AD a un entorno local de Active Directory Domain Services (AD DS), este escenario puede hacer que las contraseñas sean diferentes entre los dos directorios.

La escritura diferida de contraseñas se puede utilizar para sincronizar los cambios de contraseñas en Azure AD de vuelta al entorno local de AD DS. Azure AD Connect proporciona un mecanismo seguro para enviar los cambios de contraseñas de vuelta a un directorio local existente desde Azure AD.

> [!IMPORTANT]
> En este tutorial se muestra a un administrador cómo habilitar el restablecimiento de contraseña de autoservicio en un entorno local. Los usuarios finales registrados para el restablecimiento de contraseña de autoservicio que necesiten volver a su cuenta deben ir a https://aka.ms/sspr.
>
> Si el equipo de TI no ha habilitado la capacidad para restablecer su propia contraseña, póngase en contacto con el departamento de soporte técnico para obtener ayuda adicional.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar los permisos requeridos para la escritura diferida de contraseñas
> * Habilitar la opción de escritura diferida de contraseñas en Azure AD Connect
> * Habilitar la escritura diferida de contraseñas en SSPR de Azure AD

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesitará los siguientes recursos y privilegios:

* Un inquilino de Azure AD activo con al menos una licencia de Azure AD Premium P1 o de prueba habilitada.
    * Si es preciso, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
    * Para más información, consulte [Requisitos de concesión de licencias del autoservicio de restablecimiento de contraseña de Azure AD](concept-sspr-licensing.md).
* Una cuenta con privilegios de *administrador global*.
* Azure AD configurado para el autoservicio de restablecimiento de contraseña.
    * Si es necesario, [complete el tutorial anterior para habilitar SSPR de Azure AD](tutorial-enable-sspr.md).
* Un entorno de AD DS local existente configurado con una versión actual de Azure AD Connect.
    * Si es necesario, configure Azure AD Connect con la configuración [rápida](../hybrid/how-to-connect-install-express.md) o [personalizada](../hybrid/how-to-connect-install-custom.md).
    * Para usar la escritura diferida de contraseñas, los controladores de dominio deben ser Windows Server 2012 o posteriores.

## <a name="configure-account-permissions-for-azure-ad-connect"></a>Configuración de los permisos de cuenta para Azure AD Connect

Azure AD Connect le permite sincronizar usuarios, grupos y credenciales entre un entorno de AD DS local y Azure AD. Normalmente, se instala Azure AD Connect en un equipo con Windows Server 2012 o posterior que esté unido al dominio de AD DS local.

Para trabajar correctamente con la escritura diferida del autoservicio de restablecimiento de contraseña, la cuenta especificada en Azure AD Connect debe tener establecidos los permisos y las opciones correspondientes. Si no está seguro de qué cuenta está actualmente en uso, abra Azure AD Connect y seleccione la opción **Ver la configuración actual**. La cuenta a la que necesita agregar los permisos se muestra en **Directorios sincronizados**. Se deben establecer los siguientes permisos y opciones en la cuenta:

* **Restablecimiento de contraseñas**
* **Permisos de escritura** en `lockoutTime`
* **Permisos de escritura** en `pwdLastSet`
* **Derechos extendidos** para "Contraseña no expirada" en el objeto raíz de *cada* dominio de ese bosque, si aún no se ha establecido.

Si no asigna estos permisos, la escritura diferida puede parecer estar configurada correctamente, pero los usuarios encuentran errores al intentar administrar sus contraseñas locales desde la nube. Para que aparezca "contraseña sin expiración", los permisos deben aplicarse a **Este objeto y todos los descendientes**.  

> [!TIP]
>
> Si las contraseñas de algunas cuentas de usuario no se escriben de nuevo en el directorio local, asegúrese de que la herencia no esté deshabilitada para la cuenta en el entorno de AD DS local. Los permisos de escritura para las contraseñas se deben aplicar a los objetos descendientes para que la característica funcione correctamente.

Para configurar los permisos adecuados para que se realice la escritura diferida de contraseñas, complete los pasos siguientes:

1. En el entorno local de AD DS, abra **Usuarios y equipos de Active Directory** con una cuenta que tenga los permisos de *administración de dominio* adecuados.
1. En el menú **Ver**, asegúrese de que la opción **Características avanzadas** esté activada.
1. En el panel izquierdo, seleccione con el botón derecho el objeto que representa la raíz del dominio y elija **Propiedades** > **Seguridad** > **Avanzado**.
1. En la pestaña **Permisos**, haga clic en **Agregar**.
1. En **Entidad de seguridad**, seleccione la cuenta a la que se deben aplicar los permisos (la cuenta que usa Azure AD Connect).
1. En la lista desplegable **Se aplica a**, seleccione **Descendent User objects** (Objetos del usuario descendientes).
1. En *Permisos*, active la casilla para la siguiente opción:
    * **Restablecimiento de contraseñas**
1. En *Propiedades*, active las casillas de las siguientes opciones: Desplácese por la lista para encontrar estas opciones, que ya se pueden establecer de forma predeterminada:
    * **Escribir lockoutTime**
    * **Escribir pwdLastSet**

    [ ![Establecimiento de los permisos adecuados en usuarios y equipos activos para la cuenta usada por Azure AD Connect](media/tutorial-enable-sspr-writeback/set-ad-ds-permissions-cropped.png) ](media/tutorial-enable-sspr-writeback/set-ad-ds-permissions.png#lightbox)

1. Cuando esté preparado, haga clic en **Aplicar o Aceptar** para aplicar los cambios y salir de los cuadros de diálogo abiertos.

Cuando actualiza los permisos, pueden tardar una hora (o más) en replicarse en todos los objetos del directorio.

Las directivas de contraseñas en el entorno de AD DS local pueden impedir que se procesen correctamente los restablecimientos de contraseña. Para que la escritura diferida de contraseñas funcione de forma más eficaz, la directiva de grupo para la *Vigencia mínima de la contraseña* debe establecerse en 0. Esta configuración se puede encontrar en **Configuración del equipo > Directivas > Configuración de Windows > Configuración de seguridad > Directivas de cuenta** en `gpedit.msc`.

Si actualiza la directiva de grupo, espere a que la directiva actualizada se replique o use el comando `gpupdate /force`.

> [!Note]
> Para que las contraseñas se cambien inmediatamente, la escritura diferida de contraseñas debe establecerse en 0. Sin embargo, si los usuarios se adhieren a las directivas locales y el campo *Vigencia mínima de la contraseña* se establece en un valor mayor que cero, la escritura diferida de contraseñas sigue funcionando después de que se evalúen las directivas locales.

## <a name="enable-password-writeback-in-azure-ad-connect"></a>Habilitación de la Escritura diferida de contraseñas en Azure AD Connect

Una de las opciones de configuración de Azure AD Connect es para la escritura diferida de contraseñas. Cuando esta opción está habilitada, los eventos de cambio de contraseña hacen que Azure AD Connect vuelva a sincronizar las credenciales actualizadas con el entorno de AD DS local.

Para habilitar la reescritura de SSPR, habilite primero la opción de reescritura en Azure AD Connect. Desde el servidor de Azure AD Connect, realice los siguientes pasos:

1. Inicie sesión en el servidor de Azure AD Connect e inicie el asistente de configuración de **Azure AD Connect**.
1. En la página **principal**, seleccione **Configurar**.
1. En la página **Tareas adicionales**, seleccione **Personalizar las opciones de sincronización** y haga clic en **Siguiente**.
1. En la página **Conectar con Azure AD**, escriba una credencial de administrador global para el inquilino de Azure y, después, seleccione **Siguiente**.
1. En las páginas de filtrado **Conectar directorios** y **Dominio/Unidad organizativa**, seleccione **Siguiente**.
1. En la página **Características opcionales**, active la casilla junto a **Escritura diferida de contraseñas** y haga clic en **Siguiente**.

    ![Configuración de Azure AD Connect para escritura diferida de contraseñas](media/tutorial-enable-sspr-writeback/enable-password-writeback.png)

1. En la página **Listo para configurar**, haga clic en **Configurar** y espere a que se complete el proceso.
1. Cuando finalice la configuración, seleccione **Salir**.

## <a name="enable-password-writeback-for-sspr"></a>Habilitación de la escritura diferida de contraseñas para el autoservicio de restablecimiento de contraseña

Con la escritura diferida de contraseñas habilitada en Azure AD Connect, configure ahora el autoservicio de restablecimiento de contraseña de Azure AD para la escritura diferida. Al habilitar el autoservicio de restablecimiento de contraseña para que use la escritura diferida de contraseñas, los usuarios que cambien o restablezcan su contraseña tendrán la contraseña actualizada sincronizada de nuevo en el entorno de AD DS local.

Para habilitar la escritura diferida de contraseñas en SSPR, complete los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta de administrador global.
1. Vaya a **Azure Active Directory**, seleccione **Restablecer contraseña** y, después, elija **Integración local**.
1. Establezca la opción **¿Quiere habilitar Escritura diferida de contraseñas en el directorio local?** en *Sí*.
1. Establezca la opción **¿Quiere permitir que los usuarios desbloqueen las cuentas sin restablecer la contraseña?** en *Sí*.

    ![Habilitación del autoservicio de restablecimiento de contraseña de Azure AD para la escritura diferida de contraseñas](media/tutorial-enable-sspr-writeback/enable-sspr-writeback.png)

1. Cuando esté preparado, seleccione **Guardar**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si decide que ya no desea utilizar la funcionalidad de escritura diferida del autoservicio de restablecimiento de contraseña que se ha configurado como parte de este tutorial, realice los siguientes pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque y seleccione **Azure Active Directory**, haga clic en **Restablecimiento de contraseña** y, después, elija **Integración local**.
1. Establezca la opción **¿Quiere habilitar Escritura diferida de contraseñas en el directorio local?** en *Sí*.
1. Establezca la opción **¿Quiere permitir que los usuarios desbloqueen las cuentas sin restablecer la contraseña?** en *Sí*.

Si ya no desea usar ninguna funcionalidad de contraseña, siga estos pasos desde el servidor de Azure AD Connect:

1. Inicie sesión en el servidor de Azure AD Connect e inicie el asistente de configuración de **Azure AD Connect**.
1. En la página **principal**, seleccione **Configurar**.
1. En la página **Tareas adicionales**, seleccione **Personalizar las opciones de sincronización** y haga clic en **Siguiente**.
1. En la página **Conectar con Azure AD**, escriba una credencial de administrador global para el inquilino de Azure y, después, seleccione **Siguiente**.
1. En las páginas de filtrado **Conectar directorios** y **Dominio/Unidad organizativa**, seleccione **Siguiente**.
1. En la página **Características opcionales**, desactive la casilla junto a **Escritura diferida de contraseñas** y haga clic en **Siguiente**.
1. En la página **Listo para configurar**, haga clic en **Configurar** y espere a que se complete el proceso.
1. Cuando finalice la configuración, seleccione **Salir**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha habilitado la escritura diferida del autoservicio de restablecimiento de contraseña de Azure AD en un entorno de AD DS local. Ha aprendido a:

> [!div class="checklist"]
> * Configurar los permisos necesarios para la escritura diferida de contraseñas
> * Habilitar la opción de escritura diferida de contraseñas en Azure AD Connect
> * Habilitar la escritura diferida de contraseñas en el autoservicio de restablecimiento de contraseña de Azure AD

> [!div class="nextstepaction"]
> [Evaluación del riesgo en el inicio de sesión](tutorial-risk-based-sspr-mfa.md)
