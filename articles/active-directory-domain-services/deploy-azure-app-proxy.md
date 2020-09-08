---
title: Implementación de Azure AD Application Proxy en Azure AD Domain Services | Microsoft Docs
description: Obtenga información sobre cómo proporcionar a los trabajadores remotos acceso seguro a aplicaciones internas mediante la implementación y configuración de Azure Active Directory Application Proxy en un dominio administrado de Azure Active Directory Domain Services
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: iainfou
ms.openlocfilehash: f02f8c0c5aabc48e7b9966898028485a4687f413
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723085"
---
# <a name="deploy-azure-ad-application-proxy-for-secure-access-to-internal-applications-in-an-azure-active-directory-domain-services-managed-domain"></a>Implementación de Azure AD Application Proxy para el acceso seguro a las aplicaciones internas en un dominio administrado de Azure Active Directory Domain Services

Con Azure AD Domain Services (Azure AD DS), puede migrar mediante lift-and-shift las aplicaciones heredadas que se ejecutan de manera local en Azure. Azure Active Directory (AD) Application Proxy lo ayuda a admitir trabajadores remotos mediante la publicación de manera segura de esas aplicaciones internas que forman parte de un dominio administrado de Azure AD DS para que sean accesibles a través de Internet.

Si no está familiarizado con Azure AD Application Proxy y quiere más información, consulte [cómo proporcionar acceso remoto seguro a aplicaciones internas](../active-directory/manage-apps/application-proxy.md).

En este artículo se muestra cómo crear y configurar un conector de Azure AD Application Proxy para proporcionar acceso seguro a las aplicaciones en un dominio administrado.

## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
    * Se necesita una **licencia de Azure AD Premium** para usar Azure AD Application Proxy.
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, [cree y configure un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance].

## <a name="create-a-domain-joined-windows-vm"></a>Creación de una máquina virtual Windows unida a un dominio

Para enrutar el tráfico a las aplicaciones que se ejecutan en su entorno, puede instalar el componente del conector de Azure AD Application Proxy. Este conector de Azure AD Application Proxy se debe instalar en una máquina virtual (VM) con Windows Server que estén unida al dominio administrado. En el caso de algunas aplicaciones, puede implementar varios servidores que tengan instalado el conector. Esta opción de implementación ofrece mayor disponibilidad y lo ayuda a administrar cargas de autenticación más intensas.

La máquina virtual que ejecuta el conector de Azure AD Application Proxy debe estar en la misma red virtual (o en una emparejada) que el dominio administrado. Las máquinas virtuales que hospedan las aplicaciones que se publican con Application Proxy también se deben implementar en la misma red virtual de Azure.

Para crear una máquina virtual para el conector de Azure AD Application Proxy, complete los pasos siguientes:

1. [Cree una UO personalizada](create-ou.md). Puede delegar los permisos para administrar esta UO personalizada para los usuarios dentro del dominio administrado. Las máquinas virtuales para Azure AD Application Proxy y que ejecutan las aplicaciones deben formar parte de la unidad organizativa personalizada y no de la unidad organizativa *Equipos de DC de AAD*.
1. [Una las máquinas virtuales][create-join-windows-vm], tanto la que ejecuta el conector de Azure AD Application Proxy como las que ejecutan las aplicaciones, al dominio administrado Cree estas cuentas de equipo en la unidad organizativa personalizada del paso anterior.

## <a name="download-the-azure-ad-application-proxy-connector"></a>Descarga del conector de Azure AD Application Proxy

Realice los pasos siguientes para descargar el conector de Azure AD Application Proxy. El archivo de instalación que descargue se copiará en la máquina virtual de Application Proxy en la sección siguiente.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta de usuario que tiene los permisos de *Administrador de empresa* en Azure AD.
1. Busque y seleccione **Azure Active Directory** en la parte superior del portal y, luego, elija **Aplicaciones empresariales**.
1. Seleccione **Proxy de aplicación** en el menú de la izquierda. Para crear el primer conector y habilitar el proxy de aplicación, seleccione el vínculo para **descargar un conector**.
1. En la página de descarga, acepte los términos de licencia y el contrato de privacidad y, luego, seleccione **Accept terms & Download** (Aceptar términos y descargar).

    ![Descarga del conector de Azure AD App Proxy](./media/app-proxy/download-app-proxy-connector.png)

## <a name="install-and-register-the-azure-ad-application-proxy-connector"></a>Instalación y registro del conector de Azure AD Application Proxy

Con una máquina virtual lista para usarla como el conector de Azure AD Application Proxy, copie y ejecute el archivo de instalación descargado desde Azure Portal.

1. Copie el archivo de instalación del conector de Azure AD Application Proxy en la máquina virtual.
1. Ejecute el archivo de instalación, como *AADApplicationProxyConnectorInstaller.exe*. Acepte los términos de licencia del software.
1. Durante la instalación, se le pedirá que registre el conector en Application Proxy en el directorio de Azure AD.
   * Proporcione las credenciales de un administrador global en el directorio de Azure AD. Las credenciales de administrador global de Azure AD pueden ser distintas de las credenciales de Azure en el portal.

        > [!NOTE]
        > La cuenta de administrador global que se usa para registrar el conector debe pertenecer al mismo directorio donde haya habilitado el servicio Application Proxy.
        >
        > Por ejemplo, si el dominio de Azure AD es *contoso.com*, el administrador global debe ser `admin@contoso.com` u otro alias válido en ese dominio.

   * Si la configuración de seguridad mejorada de Internet Explorer está activada en la máquina virtual donde instala el conector, la pantalla de registro podría bloquearse. Para permitir el acceso, siga las instrucciones del mensaje de error o desactive la seguridad mejorada de Internet Explorer durante el proceso de instalación.
   * Si el registro del conector no funciona, consulte [Solución de problemas de Proxy de aplicación](../active-directory/manage-apps/application-proxy-troubleshoot.md).
1. Al final de la instalación, se muestra una nota para los entornos con un proxy de salida. Para configurar el conector de Azure AD Application Proxy para que funcione a través del proxy de salida, ejecute el script proporcionado, como `C:\Program Files\Microsoft AAD App Proxy connector\ConfigureOutBoundProxy.ps1`.
1. En la página Proxy de aplicación de Azure Portal, el conector nuevo aparece con estado *Activo*, tal como se muestra en el ejemplo siguiente:

    ![El nuevo conector de Azure AD Application Proxy se muestra como activo en Azure Portal](./media/app-proxy/connected-app-proxy.png)

> [!NOTE]
> Para proporcionar alta disponibilidad a las aplicaciones que se autentican a través de Azure AD Application Proxy, puede instalar conectores en varias máquinas virtuales. Repita los mismos pasos indicados en la sección anterior para instalar el conector en otros servidores unidos al dominio administrado.

## <a name="enable-resource-based-kerberos-constrained-delegation"></a>Habilitación de la delegación restringida de Kerberos basada en recursos

Si quiere usar el inicio de sesión único en sus aplicaciones mediante la autenticación integrada de Windows (IWA), conceda a los conectores de Azure AD Application Proxy permiso para suplantar a los usuarios y enviar y recibir tokens en su nombre. Para conceder estos permisos, puede configurar la delegación restringida de Kerberos (KCD) para que el conector acceda a los recursos en el dominio administrado. Como no tiene privilegios de administrador de dominio en un dominio administrado, no se puede configurar el KCD de nivel de cuenta tradicional en un dominio administrado. En su lugar, use KCD basadas en recursos.

Para más información, consulte [Configuración de la delegación restringida de Kerberos (KCD) en Azure Active Directory Domain Services](deploy-kcd.md).

> [!NOTE]
> Debe haber iniciado sesión en una cuenta de usuario que sea miembro del grupo de *administradores de control de dominio de Azure AD* en el inquilino de Azure AD para ejecutar los cmdlets de PowerShell siguientes.
>
> Las cuentas de equipo de la máquina virtual del conector de Application Proxy y las máquinas virtuales de aplicación deben estar en una unidad organizativa personalizada en la que tenga permisos para configurar la KCD basada en recursos. No puede configurar la KCD basada en recursos para una cuenta de equipo en el contenedor integrado *Equipos de controlador de dominio de AAD*.

Use el cmdlet [Get-ADComputer][Get-ADComputer] para recuperar la configuración del equipo donde está instalado el conector de Azure AD Application Proxy. En la máquina virtual de administración unida a un dominio y en la que se ha iniciado sesión con una cuenta de usuario miembro del grupo *Administradores de Azure AD DC*, ejecute los siguientes cmdlets.

En el ejemplo siguiente se obtiene información sobre la cuenta de equipo denominada *appproxy.aaddscontoso.com*. Proporcione su propio nombre de equipo para la máquina virtual de Azure AD Application Proxy configurada en los pasos anteriores.

```powershell
$ImpersonatingAccount = Get-ADComputer -Identity appproxy.aaddscontoso.com
```

Para cada servidor de aplicaciones que ejecuta las aplicaciones detrás de Azure AD Application Proxy, use el cmdlet [Set-ADComputer][Set-ADComputer] de PowerShell para configurar KCD basada en recursos. En el ejemplo siguiente, el conector de Azure AD Application Proxy recibe permisos para usar el equipo de *appserver.aaddscontoso.com*:

```powershell
Set-ADComputer appserver.aaddscontoso.com -PrincipalsAllowedToDelegateToAccount $ImpersonatingAccount
```

Si implementa varios conectores de Azure AD Application Proxy, debe configurar una KCD basada en recursos para cada instancia del conector.

## <a name="next-steps"></a>Pasos siguientes

Con Azure AD Application Proxy integrado con Azure AD DS, publique aplicaciones para que los usuarios tengan acceso. Para más información, consulte [Publicación de aplicaciones mediante Azure AD Application Proxy](../active-directory/manage-apps/application-proxy-add-on-premises-application.md).

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[create-join-windows-vm]: join-windows-vm.md
[azure-bastion]: ../bastion/bastion-create-host-portal.md
[Get-ADComputer]: /powershell/module/addsadministration/get-adcomputer
[Set-ADComputer]: /powershell/module/addsadministration/set-adcomputer