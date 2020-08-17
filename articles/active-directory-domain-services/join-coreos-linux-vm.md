---
title: Unión de una máquina virtual CoreOS a Azure AD Domain Services | Microsoft Docs
description: Aprenda a configurar y unir una máquina virtual CoreOS a un dominio administrado de Azure AD Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 5db65f30-bf69-4ea3-9ea5-add1db83fdb8
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/13/2020
ms.author: iainfou
ms.openlocfilehash: 93f16629b74ab76d7b46603d84d52cff4bf1ca13
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87005113"
---
# <a name="join-a-coreos-virtual-machine-to-an-azure-active-directory-domain-services-managed-domain"></a>Unión de una máquina virtual CoreOS a un dominio administrado de Azure Active Directory Domain Services

Para que los usuarios puedan iniciar sesión en máquinas virtuales (VM) en Azure con un único conjunto de credenciales, puede unir VM a un dominio administrado de Azure Active Directory Domain Services (Azure AD DS). Al unir una máquina virtual a un dominio administrado de Azure AD DS, se pueden usar las cuentas de usuario y las credenciales del dominio para iniciar sesión y administrar servidores. También se aplican las pertenencias a grupos del dominio administrado para permitirle controlar el acceso a los archivos o servicios de la VM.

Este artículo muestra cómo unir una VM CoreOS a un dominio administrado.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, el primer tutorial [crea y configura un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Una cuenta de usuario que forme parte del dominio administrado.

## <a name="create-and-connect-to-a-coreos-linux-vm"></a>Creación y conexión a una máquina virtual CoreOS Linux

Si ya tiene una máquina virtual CoreOS Linux en Azure, conéctese a ella mediante SSH y luego continúe con el paso siguiente para [empezar a configurar la máquina virtual](#configure-the-hosts-file).

Si necesita crear una máquina virtual CoreOS Linux o desea crear una máquina virtual de prueba para usarla con este artículo, puede utilizar uno de los métodos siguientes:

* [Azure Portal](../virtual-machines/linux/quick-create-portal.md)
* [CLI de Azure](../virtual-machines/linux/quick-create-cli.md)
* [Azure PowerShell](../virtual-machines/linux/quick-create-powershell.md)

Al crear la VM, preste atención a la configuración de la red virtual para asegurarse de que la VM puede comunicarse con el dominio administrado:

* Implemente la máquina virtual en la misma red virtual, o en otra emparejada, en la que haya habilitado Azure AD Domain Services.
* Implemente la VM en una subred diferente a la del dominio administrado de Azure AD Domain Services.

Una vez implementada la máquina virtual, siga los pasos para conectarse a la máquina virtual mediante SSH.

## <a name="configure-the-hosts-file"></a>Configuración del archivo hosts

Para asegurarse de que el nombre de host de la máquina virtual está configurado correctamente para el dominio administrado, edite el archivo */etc/hosts* y establezca el nombre de host:

```console
sudo vi /etc/hosts
```

En el archivo *hosts*, actualice la dirección *localhost*. En el ejemplo siguiente:

* *aaddscontoso.com* es el nombre de dominio DNS del dominio administrado.
* *coreos* es el nombre de host de la máquina virtual CoreOS que se va a unir al dominio administrado.

Actualice estos nombres con sus propios valores:

```console
127.0.0.1 coreos coreos.aaddscontoso.com
```

Cuando haya terminado, guarde y salga del archivo *hosts* mediante el comando `:wq` del editor.

## <a name="configure-the-sssd-service"></a>Configuración del servicio SSSD

Actualice la configuración de SSSD en */etc/sssd/sssd.conf*.

```console
sudo vi /etc/sssd/sssd.conf
```

Especifique su propio nombre de dominio administrado para los siguientes parámetros:

* *domains* con TODAS LAS LETRAS EN MAYÚSCULAS
* *[domain/AADDSCONTOSO]* , donde  AADDSCONTOSO tiene TODAS LETRAS EN MAYÚSCULAS
* *ldap_uri*
* *ldap_search_base*
* *krb5_server*
* *krb5_realm* con TODAS LAS LETRAS EN MAYÚSCULAS

```console
[sssd]
config_file_version = 2
services = nss, pam
domains = AADDSCONTOSO.COM

[domain/AADDSCONTOSO]
id_provider = ad
auth_provider = ad
chpass_provider = ad

ldap_uri = ldap://aaddscontoso.com
ldap_search_base = dc=aaddscontoso,dc=com
ldap_schema = rfc2307bis
ldap_sasl_mech = GSSAPI
ldap_user_object_class = user
ldap_group_object_class = group
ldap_user_home_directory = unixHomeDirectory
ldap_user_principal = userPrincipalName
ldap_account_expire_policy = ad
ldap_force_upper_case_realm = true
fallback_homedir = /home/%d/%u

krb5_server = aaddscontoso.com
krb5_realm = AADDSCONTOSO.COM
```

## <a name="join-the-vm-to-the-managed-domain"></a>Unión de la máquina virtual al dominio administrado

Con el archivo de configuración de SSSD actualizado, una la máquina virtual al dominio administrado.

1. En primer lugar, use el comando `adcli info` para comprobar que puede ver información sobre el dominio administrado. En el ejemplo siguiente se obtiene información para el dominio *AADDSCONTOSO.COM*. Especifique su propio nombre de dominio administrado CON TODAS LAS LETRAS EN MAYÚSCULAS:

    ```console
    sudo adcli info AADDSCONTOSO.COM
    ```

   Si el comando `adcli info` no se puede encontrar en el dominio administrado, repase los siguientes pasos de solución de problemas:

    * Asegúrese de que el dominio sea accesible desde la máquina virtual. Pruebe `ping aaddscontoso.com` para ver si se devuelve una respuesta positiva.
    * Compruebe que la VM se ha implementado en la misma red virtual (o en otra emparejada) en la que el dominio administrado está disponible.
    * Confirme que se ha actualizado la configuración del servidor DNS de la red virtual para que apunte a los controladores de dominio del dominio administrado.

1. Ahora, una laVM al dominio administrado con el comando `adcli join`. Especifique un usuario que forme parte del dominio administrado. Si es necesario, [agregue una cuenta de usuario a un grupo en Azure AD](../active-directory/fundamentals/active-directory-groups-members-azure-portal.md).

    Una vez más, el nombre del dominio administrado se debe escribir CON TODAS LAS LETRAS EN MAYÚSCULAS. En el ejemplo siguiente, la cuenta denominada `contosoadmin@aaddscontoso.com` se usa para inicializar Kerberos. Introduzca su cuenta de usuario que forma parte del dominio administrado.

    ```console
    sudo adcli join -D AADDSCONTOSO.COM -U contosoadmin@AADDSCONTOSO.COM -K /etc/krb5.keytab -H coreos.aaddscontoso.com -N coreos
    ```

    El comando `adcli join` no devuelve ninguna información cuando la VM se ha unido correctamente al dominio administrado.

1. Para aplicar la configuración de unión a dominio, inicie el servicio SSSD:
  
    ```console
    sudo systemctl start sssd.service
    ```

## <a name="sign-in-to-the-vm-using-a-domain-account"></a>Inicio de sesión en la máquina virtual mediante una cuenta de dominio

Para comprobar que la VM se ha unido correctamente al dominio administrado, inicie una nueva conexión SSH con una cuenta de usuario de dominio. Confirme que se ha creado un directorio particular y que se ha aplicado la pertenencia a grupos del dominio.

1. Cree una nueva conexión SSH desde la consola. Use una cuenta de dominio que pertenezca al dominio administrado mediante el comando `ssh -l` como, por ejemplo, `contosoadmin@aaddscontoso.com` y, a continuación, escriba la dirección de la máquina virtual, por ejemplo: *coreos.aaddscontoso.com*. Si usa Azure Cloud Shell, use la dirección IP pública de la máquina virtual en lugar del nombre DNS interno.

    ```console
    ssh -l contosoadmin@AADDSCONTOSO.com coreos.aaddscontoso.com
    ```

1. Ahora, compruebe que las pertenencias a grupos se están resolviendo correctamente:

    ```console
    id
    ```

    Debería ver las pertenencias a grupos del dominio administrado.

## <a name="next-steps"></a>Pasos siguientes

Si tiene problemas para conectar la VM al dominio administrado o al iniciar sesión con una cuenta de dominio, consulte [Solución de problemas de unión al dominio](join-windows-vm.md#troubleshoot-domain-join-issues).

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
