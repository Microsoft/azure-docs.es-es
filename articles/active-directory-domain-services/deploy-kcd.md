---
title: Delegación restringida de Kerberos para Azure AD Domain Services | Microsoft Docs
description: Aprenda a habilitar la delegación restringida de Kerberos (KCD) basada en recursos en un dominio administrado de Azure Active Directory Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/06/2020
ms.author: iainfou
ms.openlocfilehash: 7f1640f74c3fca27e1d992f3e2cc538f6e2c7fef
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723000"
---
# <a name="configure-kerberos-constrained-delegation-kcd-in-azure-active-directory-domain-services"></a>Configuración de la delegación restringida de Kerberos (KCD) en Azure Active Directory Domain Services

A medida que se ejecutan las aplicaciones, puede haber una necesidad de que accedan a los recursos en el contexto de un usuario diferente. Active Directory Domain Services (AD DS) admite un mecanismo denominado *delegación de Kerberos* que permite este caso de uso. La delegación *restringida* de Kerberos (KCD) se basa entonces en este mecanismo para definir los recursos específicos a los que se puede acceder en el contexto del usuario.

Los dominios administrados de Azure Active Directory Domain Services (Azure DS) se bloquean de forma más segura que los entornos de AD DS locales tradicionales, así que use una delegación restringida de Kerberos más segura *basada en recursos*.

En este artículo se muestra cómo configurar la delegación restringida de Kerberos basada en recursos en un dominio administrado de Azure AD DS.

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesita los siguientes recursos:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, [cree y configure un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Una máquina virtual de administración de Windows Server que esté unida al dominio administrado de Azure AD DS.
    * Si es necesario, realice el tutorial para [crear una máquina virtual de Windows Server y unirla a un dominio administrado][create-join-windows-vm] y luego [instalar las herramientas de administración de AD DS][tutorial-create-management-vm].
* Una cuenta de usuario que sea miembro del grupo de *administradores de Azure AD DC* en el inquilino de Azure AD.

## <a name="kerberos-constrained-delegation-overview"></a>Introducción a la delegación restringida de Kerberos

La delegación de Kerberos permite que una cuenta suplante a otra cuenta para acceder a los recursos. Por ejemplo, una aplicación web que accede a un componente web de back-end puede suplantarse con una cuenta de usuario diferente cuando realiza la conexión back-end. La delegación de Kerberos no es segura ya que no limita los recursos a los que puede acceder la cuenta de suplantación.

La delegación *restringida* de Kerberos (KCD) restringe los servicios o recursos a los que un servidor o una aplicación especificados puede conectarse al suplantar otra identidad. La KCD tradicional requiere privilegios de administrador de dominio para configurar una cuenta de dominio de un servicio y restringe la cuenta a la ejecución en un solo dominio.

Además, presenta algunos problemas. Por ejemplo, en versiones anteriores del sistema operativo, el administrador de servicios no tenía ninguna manera útil de saber qué servicios de front-end se delegaban en los servicios de recurso que poseía. Cualquier servicio de front-end que podía delegarse en un servicio de recursos representaba un punto de ataque en potencia. Si un servidor que hospedaba un servicio de front-end configurado para delegarse en servicios de recursos se ponía en peligro, dichos servicios de recursos también se podían poner en peligro.

En un dominio administrado, no tiene privilegios de administrador de dominio. Como consecuencia, la KCD tradicional basada en la cuenta no se puede configurar en un dominio administrado. En su lugar, se puede usar la KCD basada en recursos, que también es más segura.

### <a name="resource-based-kcd"></a>KCD basada en recursos

Windows Server 2012 y versiones posteriores ofrece a los administradores la posibilidad de configurar la delegación restringida para su servicio. Este modelo se conoce como KCD basada en recursos. En este modelo, el administrador de servicios de back-end puede permitir o denegar servicios de front-end específicos mediante KCD.

La KCD basada en recursos se configura mediante PowerShell. Se usan los cmdlets [Set-ADComputer][Set-ADComputer] o [Set-ADUser][Set-ADUser], dependiendo de si la cuenta de suplantación es una cuenta de equipo o una cuenta de servicio o usuario.

## <a name="configure-resource-based-kcd-for-a-computer-account"></a>Configuración de la KCD basada en recursos para una cuenta de equipo

En este escenario, supongamos que tiene una aplicación web que se ejecuta en el equipo denominado *contoso-webapp.aaddscontoso.com*.

La aplicación web necesita acceder a una API web que se ejecuta en el equipo denominado *contoso-api.aaddscontoso.com* en el contexto de los usuarios del dominio.

Haga lo siguiente para configurar este escenario:

1. [Cree una UO personalizada](create-ou.md). Puede delegar los permisos para administrar esta UO personalizada para los usuarios dentro del dominio administrado.
1. [Una las máquinas virtuales][create-join-windows-vm], tanto la que ejecuta la aplicación web como la que ejecuta la API web, al dominio administrado. Cree estas cuentas de equipo en la unidad organizativa personalizada del paso anterior.

    > [!NOTE]
    > Las cuentas de equipo de la aplicación web y de la API web deben estar en una unidad organizativa personalizada en la que tenga permisos para configurar la KCD basada en recursos. No puede configurar la KCD basada en recursos para una cuenta de equipo en el contenedor integrado *Equipos de controlador de dominio de AAD*.

1. Por último, configure la KCD basada en recursos mediante el cmdlet [Set-ADComputer][Set-ADComputer] de PowerShell.

    En la máquina virtual de administración unida a un dominio y en la que se ha iniciado sesión con una cuenta de usuario miembro del grupo *Administradores de Azure AD DC*, ejecute los siguientes cmdlets. Proporcione sus propios nombres de equipo según sea necesario:
    
    ```powershell
    $ImpersonatingAccount = Get-ADComputer -Identity contoso-webapp.aaddscontoso.com
    Set-ADComputer contoso-api.aaddscontoso.com -PrincipalsAllowedToDelegateToAccount $ImpersonatingAccount
    ```

## <a name="configure-resource-based-kcd-for-a-user-account"></a>Configuración de la KCD basada en recursos para una cuenta de usuario

En este escenario, supongamos que tiene una aplicación web que se ejecuta con una cuenta de servicio denominada *appsvc*. La aplicación web necesita acceder a una API web que se ejecute con una cuenta de servicio denominada *backendsvc* en el contexto de los usuarios del dominio. Haga lo siguiente para configurar este escenario:

1. [Cree una UO personalizada](create-ou.md). Puede delegar los permisos para administrar esta UO personalizada para los usuarios dentro del dominio administrado.
1. [Una las máquinas virtuales][create-join-windows-vm] que ejecutan la API o el recurso web de back-end al dominio administrado. Cree su cuenta de equipo dentro de la UO personalizada.
1. Cree la cuenta de servicio (por ejemplo, *appsvc*) usada para ejecutar la aplicación web dentro de la UO personalizada.

    > [!NOTE]
    > De nuevo, la cuenta de equipo de la máquina virtual de API web y la cuenta de servicio de la aplicación web deben estar en una unidad organizativa personalizada en la que tenga permisos para configurar la KCD basada en recursos. No se puede configurar la KCD basada en recursos para las cuentas de los contenedores integrados *Equipos de controlador de dominio de AAD* o *Usuarios de controlador de dominio de AAD*. Esto significa que no puede usar cuentas de usuario sincronizadas desde Azure AD para configurar la KCD basada en recursos. Debe crear y usar cuentas de servicio creadas específicamente en Azure AD DS.

1. Por último, configure la KCD basada en recursos mediante el cmdlet de [Set-ADUser][Set-ADUser] de PowerShell.

    En la máquina virtual de administración unida a un dominio y en la que se ha iniciado sesión con una cuenta de usuario miembro del grupo *Administradores de Azure AD DC*, ejecute los siguientes cmdlets. Proporcione sus propios nombres de servicio según sea necesario:

    ```powershell
    $ImpersonatingAccount = Get-ADUser -Identity appsvc
    Set-ADUser backendsvc -PrincipalsAllowedToDelegateToAccount $ImpersonatingAccount
    ```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo funciona la delegación en Active Directory Domain Services, consulte [Introducción a la delegación restringida de Kerberos][kcd-technet].

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[create-join-windows-vm]: join-windows-vm.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md
[Set-ADComputer]: /powershell/module/addsadministration/set-adcomputer
[Set-ADUser]: /powershell/module/addsadministration/set-aduser

<!-- EXTERNAL LINKS -->
[kcd-technet]: /previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj553400(v=ws.11)