---
title: Habilitación de la sincronización de hash de contraseñas para Azure AD Domain Services | Microsoft Docs
description: En este tutorial, aprenderá a habilitar la sincronización de hash de contraseñas con Azure AD Connect en un dominio administrado Azure Active Directory Domain Services.
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/06/2020
ms.author: iainfou
ms.openlocfilehash: 02828e0c159e2d30dacc5759f1239dae1268dac5
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87491789"
---
# <a name="tutorial-enable-password-synchronization-in-azure-active-directory-domain-services-for-hybrid-environments"></a>Tutorial: Habilitación de la sincronización de contraseñas en Azure Active Directory Domain Services para entornos híbridos

En entornos híbridos, se puede configurar un inquilino de Azure Active Directory (Azure AD) para que se sincronice con un entorno de Active Directory Domain Services (AD DS) en el entorno local mediante Azure AD Connect. De forma predeterminada, Azure AD Connect no sincroniza los valores hash de contraseñas de NT LAN Manager (NTLM) y Kerberos heredados que son necesarios para Azure Active Directory Domain Services (Azure AD DS).

Para usar Azure AD DS con cuentas sincronizadas desde un entorno de AD DS local, debe configurar Azure AD Connect para sincronizar los valores hash de contraseñas necesarios para la autenticación NTLM y Kerberos. Una vez configurado Azure AD Connect, un evento de cambio de contraseña o de creación de una cuenta en un entorno local también sincroniza los valores hash de contraseñas heredados con Azure AD.

Si usa cuentas solo en la nube sin un entorno de AD DS local, o si usa un *bosque de recursos*, no es necesario realizar estos pasos. En el caso de dominios administrados que usan un bosque de recursos, los hashes de contraseña locales nunca se sincronizan. Para la autenticación de las cuentas locales, se usan las confianzas de bosque devueltas a los propios controladores de dominio de AD DS.

En este tutorial, aprenderá:

> [!div class="checklist"]
> * Por qué se necesitan los valores hash de contraseñas de Kerberos y NTLM heredados
> * Procedimiento para configurar la sincronización de hash de contraseñas heredadas para Azure AD Connect

Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesitará los siguientes recursos:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción que sea sincronizado con un directorio en el entorno local mediante Azure AD Connect.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
    * Si es necesario, [habilite en Azure AD Connect la sincronización de hash de contraseñas][enable-azure-ad-connect].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, [cree y configure un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance].

## <a name="password-hash-synchronization-using-azure-ad-connect"></a>Sincronización de los valores hash de contraseñas con Azure AD Connect

Azure AD Connect se utiliza para sincronizar objetos como cuentas de usuario y grupos desde un entorno de AD DS local en un inquilino de Azure AD. Como parte del proceso, la sincronización de hash de contraseñas permite a las cuentas usar la misma contraseña en el entorno de AD DS local y Azure AD.

Para autenticar a los usuarios en el dominio administrado, Azure AD DS necesita los valores hash de contraseñas en un formato adecuado para la autenticación de NTLM y Kerberos. A menos que habilite Azure AD DS para el inquilino, Azure AD no almacena los valores hash de contraseñas en el formato necesario para la autenticación NTLM o Kerberos. Por motivos de seguridad, Azure AD tampoco almacena las credenciales de contraseñas en forma de texto sin cifrar. Por consiguiente, Azure AD no tiene forma de generar automáticamente estos hash de las contraseñas de NTLM o Kerberos basándose en las credenciales existentes de los usuarios.

Azure AD Connect se puede configurar para sincronizar los valores hash de contraseñas NTLM o Kerberos necesarios para Azure AD DS. Asegúrese de que ha completado los pasos para [habilitar Azure AD Connect para la sincronización de valores hash de contraseñas][enable-azure-ad-connect]. Si tiene una instancia existente de Azure AD Connect, [descargue y actualice a la versión más reciente][azure-ad-connect-download] para asegurarse de que puede sincronizar los valores hash de contraseñas heredados para NTLM y Kerberos. Esta funcionalidad no está disponible en las primeras versiones de Azure AD Connect ni con la herramienta DirSync heredada. Se requiere Azure AD Connect versión *1.1.614.0* o posterior.

> [!IMPORTANT]
> Azure AD Connect solo debe instalarse y configurarse para la sincronización con entornos de AD DS locales. No se admite la instalación de Azure AD Connect en un dominio administrado de Azure AD DS para volver a sincronizar los objetos con Azure AD.

## <a name="enable-synchronization-of-password-hashes"></a>Habilitación de la sincronización de valores hash de contraseñas

Con Azure AD Connect instalado y configurado para sincronizarse con Azure AD, ahora configure la sincronización de hash de contraseñas heredada para NTLM y Kerberos. Se usa un script de PowerShell para configurar las opciones necesarias y, a continuación, iniciar una sincronización de contraseñas completa en Azure AD. Cuando se haya completado el proceso de sincronización de valores hash de contraseñas de Azure AD Connect, los usuarios podrán iniciar sesión en las aplicaciones mediante Azure AD DS que usa valores hash de contraseñas Kerberos o NTLM heredados.

1. En el equipo en el que se ha instalado Azure AD Connect, en el menú Inicio, abra **Azure AD Connect > Servicio de sincronización**.
1. Seleccione la pestaña **Conectores**. Se enumera la información de conexión utilizada para establecer la sincronización entre el entorno de AD DS local y Azure AD.

    El **tipo** indica o *Windows Azure Active Directory (Microsoft)* para el conector de Azure AD o *Active Directory Domain Services* para el conector de AD DS en el entorno local. Anote los nombres de los conectores que se van a usar en el script de PowerShell en el paso siguiente.

    ![Enumeración de los nombres de los conectores en Sync Service Manager](media/tutorial-configure-password-hash-sync/service-sync-manager.png)

    En esta captura de pantalla de ejemplo, se utilizan los siguientes conectores:

    * El conector de Azure AD se denomina *contoso.onmicrosoft.com - AAD*
    * El conector de AD DS local se denomina *onprem.contoso.com*

1. Copie y pegue el siguiente script de PowerShell en el equipo con Azure AD Connect instalado. El script desencadena una sincronización de contraseñas completa que incluye los valores hash de contraseñas heredados. Actualice las variables `$azureadConnector` y `$adConnector` con los nombres de conector del paso anterior.

    Ejecute este script en cada bosque de AD para sincronizar los valores hash de contraseñas Kerberos y NTLM de la cuenta en el entorno local con Azure AD.

    ```powershell
    # Define the Azure AD Connect connector names and import the required PowerShell module
    $azureadConnector = "<CASE SENSITIVE AZURE AD CONNECTOR NAME>"
    $adConnector = "<CASE SENSITIVE AD DS CONNECTOR NAME>"
    
    Import-Module "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync\ADSync.psd1"
    Import-Module "C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\AdSyncConfig.psm1"

    # Create a new ForceFullPasswordSync configuration parameter object then
    # update the existing connector with this new configuration
    $c = Get-ADSyncConnector -Name $adConnector
    $p = New-Object Microsoft.IdentityManagement.PowerShell.ObjectModel.ConfigurationParameter "Microsoft.Synchronize.ForceFullPasswordSync", String, ConnectorGlobal, $null, $null, $null
    $p.Value = 1
    $c.GlobalParameters.Remove($p.Name)
    $c.GlobalParameters.Add($p)
    $c = Add-ADSyncConnector -Connector $c

    # Disable and re-enable Azure AD Connect to force a full password synchronization
    Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $azureadConnector -Enable $false
    Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $azureadConnector -Enable $true
    ```

    En función del tamaño de su directorio en cuanto al número de cuentas y grupos, la sincronización de los valores hash de contraseñas heredados en Azure AD puede tardar algún tiempo. Las contraseñas se sincronizan después con el dominio administrado una vez que se han sincronizado con Azure AD.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido:

> [!div class="checklist"]
> * Por qué se necesitan los valores hash de contraseñas de Kerberos y NTLM heredados
> * Procedimiento para configurar la sincronización de hash de contraseñas heredadas para Azure AD Connect

> [!div class="nextstepaction"]
> [Aprenda cómo funciona la sincronización en un dominio administrado de Azure AD Domain Services](synchronization.md)

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[enable-azure-ad-connect]: ../active-directory/hybrid/how-to-connect-install-express.md

<!-- EXTERNAL LINKS -->
[azure-ad-connect-download]: https://www.microsoft.com/download/details.aspx?id=47594
