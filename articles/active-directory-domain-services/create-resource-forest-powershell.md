---
title: Creación de un bosque de recursos de Azure AD Domain Services mediante Azure PowerShell | Microsoft Docs
description: En este artículo, aprenderá a crear y configurar un bosque de recursos de Azure Active Directory Domain Services y un bosque de salida para un entorno de Active Directory Domain Services local mediante Azure PowerShell.
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/27/2020
ms.author: iainfou
ms.openlocfilehash: eb627b8069bcd9efd1d56adab5eda45dc34a1a10
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87922003"
---
# <a name="create-an-azure-active-directory-domain-services-resource-forest-and-outbound-forest-trust-to-an-on-premises-domain-using-azure-powershell"></a>Cree un bosque de recursos de Azure Active Directory Domain Services y una confianza de bosque de salida en un dominio local mediante Azure PowerShell

En entornos en los que no se pueden sincronizar los hash de contraseña, o en los que los usuarios inician sesión de forma exclusiva con tarjetas inteligentes y no saben su contraseña, puede usar un bosque de recursos en Azure Active Directory Domain Services (Azure AD DS). Un bosque de recursos usa una confianza de salida unidireccional desde Azure AD DS a uno o varios entornos locales de AD DS. Esta relación de confianza permite que los usuarios, las aplicaciones y los equipos se autentiquen en un dominio local desde el dominio administrado de Azure AD DS. En un bosque de recursos, los hashes de contraseña locales nunca se sincronizan.

![Diagrama de la confianza de bosque desde Azure AD DS a un entorno local de AD DS](./media/concepts-resource-forest/resource-forest-trust-relationship.png)

En este artículo aprenderá a:

> [!div class="checklist"]
> * Crear un bosque de recursos de Azure AD DS mediante Azure PowerShell.
> * Crear una confianza de bosque de salida unidireccional en el dominio administrado mediante Azure PowerShell.
> * Configurar DNS en un entorno de AD DS local para admitir la conectividad del dominio administrado.
> * Crear una confianza de bosque de entrada unidireccional en un entorno local de AD DS
> * Probar y validar la relación de confianza para la autenticación y el acceso a los recursos.

Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

> [!IMPORTANT]
> A día de hoy, los bosques de recursos del dominio administrado no son compatibles con Azure HDInsight ni Azure Files. Los bosques de usuarios predeterminados del dominio administrado admiten ambos servicios adicionales.

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].

* Instale y configure Azure PowerShell.
    * Si es necesario, siga las instrucciones para [instalar el módulo de Azure PowerShell y conectarse a la suscripción de Azure](/powershell/azure/install-az-ps).
    * Asegúrese de que inicia sesión en su suscripción de Azure con el cmdlet [Connect-AzAccount][Connect-AzAccount].
* Instale y configure Azure AD PowerShell.
    * Si es necesario, siga las instrucciones para [instalar el módulo de Azure AD PowerShell y conectarse a Azure AD](/powershell/azure/active-directory/install-adv2).
    * Asegúrese de iniciar sesión en el inquilino de Azure AD mediante el cmdlet [Connect-AzureAD][Connect-AzureAD].
* Necesita privilegios de *administrador global* en el inquilino de Azure AD para habilitar Azure AD DS.
* Necesita privilegios de *colaborador* en la suscripción de Azure para crear los recursos de Azure AD DS necesarios.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

En este artículo, creará y configurará la confianza de bosque de salida desde un dominio administrado mediante Azure Portal. Para empezar a trabajar, primero inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="deployment-process"></a>Proceso de implementación

Este es un proceso compuesto de varias partes para crear un bosque de recursos de dominio administrado y una relación de confianza en una instancia de AD DS local. Los siguientes pasos de alto nivel crean el entorno híbrido de confianza:

1. Cree una entidad de servicio de dominio administrado.
1. Cree un bosque de recursos de dominio administrado.
1. Cree la conectividad de red híbrida con una VPN de sitio a sitio o con Express Route.
1. Cree el lado del dominio administrado de la relación de confianza.
1. Cree el lado AD DS local de la relación de confianza.

Antes de empezar, asegúrese de que comprende las [consideraciones de red, los nombres de bosque y los requisitos de DNS](tutorial-create-forest-trust.md#networking-considerations). No se puede cambiar el nombre del bosque de dominio administrado una vez implementado.

## <a name="create-the-azure-ad-service-principal"></a>Cree la entidad de servicio de Azure AD.

Azure AD DS requiere una entidad de servicio que sincronice los datos de Azure AD. Esta entidad de seguridad debe crearse en el inquilino de Azure AD antes de crear el bosque de recursos del dominio administrado.

Cree una entidad de servicio de Azure AD para Azure AD DS así poder comunicarse y autenticarse. Se usa un identificador de aplicación específico denominado *Servicios de controlador de dominio* con un identificador de *2565bd9d-da50-47d4-8b85-4c97f669dc36*. No cambie este identificador de aplicación.

Cree una entidad de servicio de Azure AD con el cmdlet [New-AzureADServicePrincipal][New-AzureADServicePrincipal]:

```powershell
New-AzureADServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"
```

## <a name="create-a-managed-domain-resource-forest"></a>Creación de un bosque de recursos de dominio administrado

Para crear un bosque de recursos de dominio administrado, use el script `New-AzureAaddsForest`. Este script forma parte de un conjunto más amplio de comandos que admiten la creación y administración de bosques de recursos de dominio administrado, incluida la creación de un bosque enlazado unidireccional que se explica en la sección siguiente. Estos scripts están disponibles en la [Galería de PowerShell](https://www.powershellgallery.com/) y los ha firmado digitalmente el equipo de ingeniería de Azure AD.

1. En primer lugar, cree un grupo de recursos con el cmdlet [New-AzResourceGroup][New-AzResourceGroup]. En el ejemplo siguiente, el grupo de recursos se denomina *myResourceGroup* y se crea en la región *westus*. Use su propio nombre y la región deseada:

    ```azure-powershell
    New-AzResourceGroup `
      -Name "myResourceGroup" `
      -Location "WestUS"
    ```

1. Instale el script `New-AaddsResourceForestTrust` de la [Galería de PowerShell][powershell-gallery] mediante el cmdlet [install-script][Install-Script]:

    ```powershell
    Install-Script -Name New-AaddsResourceForestTrust
    ```

1. Revise los siguientes parámetros necesarios para el script `New-AzureAaddsForest`. Asegúrese de que cumple también los requisitos previos de los módulos de **Azure PowerShell** y de **Azure AD PowerShell**. Asegúrese de que ha planeado los requisitos de la red virtual para proporcionar conectividad local y de la aplicación.

    | Nombre                         | Parámetro de script          | Descripción |
    |:-----------------------------|---------------------------|:------------|
    | Suscripción                 | *-azureSubscriptionId*    | Id. de suscripción que se para la facturación de Azure AD DS. Puede obtener la lista de suscripciones mediante el cmdlet [Get-AzureRMSubscription][Get-AzureRMSubscription]. |
    | Grupo de recursos               | *-aaddsResourceGroupName* | Nombre del grupo de recursos del dominio administrado y los recursos asociados. |
    | Location                     | *-aaddsLocation*          | Región de Azure que se usa para hospedar el dominio administrado. Para ver las regiones disponibles, consulte las [regiones admitidas para Azure AD DS.](https://azure.microsoft.com/global-infrastructure/services/?products=active-directory-ds&regions=all) |
    | Administrador de Azure AD DS    | *-aaddsAdminUser*         | Nombre principal de usuario del primer administrador de dominio administrado. Esta cuenta debe ser una cuenta de usuario de nube existente en Azure Active Directory. El usuario y el usuario que ejecuta el script se agregan al grupo de *administradores de AAD DC*. |
    | Nombre de dominio de Azure AD DS      | *-aaddsDomainName*        | FQDN del dominio administrado, basado en las instrucciones anteriores sobre cómo elegir un nombre de bosque. |

    El script `New-AzureAaddsForest` puede crear la red virtual de Azure y la subred de Azure AD DS si estos recursos no existen todavía. Opcionalmente, el script puede crear subredes de carga de trabajo cuando se especifica:

    | Nombre                              | Parámetro de script                  | Descripción |
    |:----------------------------------|:----------------------------------|:------------|
    | Nombre de la red virtual              | *-aaddsVnetName*                  | Nombre de la red virtual del dominio administrado.|
    | Espacio de direcciones                     | *-aaddsVnetCIDRAddressSpace*      | El intervalo de direcciones de la red virtual en notación CIDR (si se crea la red virtual).|
    | Nombre de subred de Azure AD DS           | *-aaddsSubnetName*                | Nombre de la subred de la red virtual *aaddsVnetName* que hospeda el dominio administrado. No implemente sus propias VM y cargas de trabajo en esta subred. |
    | Intervalo de direcciones de Azure AD DS         | *-aaddsSubnetCIDRAddressRange*    | Intervalo de direcciones de subred en notación CIDR para la instancia de AAD DS, como *192.168.1.0/24*. El intervalo de direcciones debe estar incluido en el intervalo de direcciones de la red virtual y debe ser diferente de otras subredes. |
    | Nombre de la subred de carga de trabajo (opcional)   | *-workloadSubnetName*             | Nombre opcional de una subred en la red virtual *aaddsVnetName* que se va a crear para sus propias cargas de trabajo de la aplicación. En su lugar, las VM y las aplicaciones se conectan a una red virtual de Azure emparejada. |
    | Intervalo de direcciones de la carga de trabajo (opcional) | *-workloadSubnetCIDRAddressRange* | Intervalo opcional de direcciones de subred en la notación CIDR para la carga de trabajo de la aplicación, como *192.168.2.0/24*. El intervalo de direcciones debe estar incluido en el intervalo de direcciones de la red virtual y debe ser diferente de otras subredes.|

1. A continuación deberá crear un bosque de recursos de dominio administrado mediante el script `New-AzureAaaddsForest`. En el ejemplo siguiente se crea un bosque denominado *addscontoso.com* y una subred de carga de trabajo. Proporcione sus propios nombres de parámetro e intervalos de direcciones IP o redes virtuales existentes.

    ```azure-powershell
    New-AzureAaddsForest `
        -azureSubscriptionId <subscriptionId> `
        -aaddsResourceGroupName "myResourceGroup" `
        -aaddsLocation "WestUS" `
        -aaddsAdminUser "contosoadmin@contoso.com" `
        -aaddsDomainName "aaddscontoso.com" `
        -aaddsVnetName "myVnet" `
        -aaddsVnetCIDRAddressSpace "192.168.0.0/16" `
        -aaddsSubnetName "AzureADDS" `
        -aaddsSubnetCIDRAddressRange "192.168.1.0/24" `
        -workloadSubnetName "myWorkloads" `
        -workloadSubnetCIDRAddressRange "192.168.2.0/24"
    ```

    Se tarda bastante tiempo en crear el bosque de recursos del dominio administrado y los recursos de apoyo. Permita que se complete el script. Continúe con la sección siguiente para configurar la conectividad de red local mientras que el bosque de recursos de Azure AD se aprovisiona en segundo plano.

## <a name="configure-and-validate-network-settings"></a>Configuración y validación de los parámetros de configuración de red

A medida que el dominio administrado se sigue implementando, configure y valide la conectividad de red híbrida en el centro de recursos local. También necesita una VM de administración para usarla con el dominio administrado y así poder realizar el mantenimiento periódico. Es posible que algunas de las conexiones híbridas ya existan en su entorno o que necesite trabajar con otras personas del equipo para configurar las conexiones.

Antes de empezar, asegúrese de que comprende las [consideraciones y recomendaciones sobre las redes](tutorial-create-forest-trust.md#networking-considerations).

1. Cree la conectividad híbrida en su red local de Azure mediante una VPN de Azure o una conexión de Azure ExpressRoute. La configuración de red híbrida está fuera del ámbito de esta documentación y puede que ya exista en su entorno. Para obtener más información sobre escenarios específicos, consulte los siguientes artículos:

    * [VPN de sitio a sitio de Azure](/azure/vpn-gateway/vpn-gateway-about-vpngateways).
    * [Información general de Azure ExpressRoute](/azure/expressroute/expressroute-introduction).

    > [!IMPORTANT]
    > Si crea la conexión directamente en la red virtual de su dominio administrado, use una subred de puerta de enlace independiente. No cree la puerta de enlace en la subred del dominio administrado.

1. Para administrar un dominio administrado, cree una VM de administración, únase al dominio administrado e instale las herramientas de administración de AD DS necesarias.

    Mientras se implementa el bosque de recursos del dominio administrado, [cree una VM de Windows Server](https://docs.microsoft.com/azure/active-directory-domain-services/join-windows-vm); a continuación, [instale las herramientas de administración del núcleo de AD DS](https://docs.microsoft.com/azure/active-directory-domain-services/tutorial-create-management-vm) y así poder instalar las herramientas de administración necesarias. Espere a unir la VM de administración al dominio administrado hasta que llegue a uno de los siguientes pasos después de implementar correctamente el dominio.

1. Valide la conectividad de red entre la red local y la red virtual de Azure.

    * Confirme que el controlador de dominio local puede conectarse a la VM administrada mediante `ping` o el escritorio remoto, por ejemplo.
    * Compruebe que la VM de administración puede conectarse a los controladores de dominio locales mediante una utilidad como `ping`.

1. En Azure Portal, busque y seleccione **Azure AD Domain Services**. Elija el dominio administrado, como *aaddscontoso.com* y espere a que el estado aparezca como **En ejecución**.

    Al realizar la ejecución, [actualice la configuración de DNS para Azure Virtual Network](tutorial-create-instance.md#update-dns-settings-for-the-azure-virtual-network) y, a continuación, [habilite las cuentas de usuario para Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) para finalizar las configuraciones del bosque de recursos del dominio administrado.

1. Anote las direcciones DNS que se muestran en la página de información general. Necesitará esas direcciones cuando tenga que configurar el lado local de Active Directory correspondiente a la relación de confianza de la sección siguiente.
1. Reinicie la VM de administración para que reciba la nueva configuración de DNS y, a continuación, [una la VM al dominio administrado](join-windows-vm.md#join-the-vm-to-the-managed-domain).
1. Una vez que la VM de administración esté unida al dominio administrado, vuelva a conectarse mediante el escritorio remoto.

    En el símbolo del sistema, use `nslookup` y el nombre del bosque de recursos del dominio administrado para validar la resolución de nombres del bosque de recursos.

    ```console
    nslookup aaddscontoso.com
    ```

    El comando debe devolver dos direcciones IP para el bosque de recursos.

## <a name="create-the-forest-trust"></a>Creación de la confianza de bosque

La confianza de bosque tiene dos partes: la confianza de bosque de salida unidireccional en el bosque de recursos del dominio administrado y la confianza de bosque de entrada unidireccional en el bosque de AD DS local. Cree manualmente ambos lados de esta relación de confianza. Cuando se crean ambos lados, los usuarios y los recursos pueden autenticarse correctamente mediante la confianza de bosque. Un bosque de recursos de dominio administrado admite hasta cinco bosques de salida unidireccionales en los bosques locales.

### <a name="create-the-managed-domain-side-of-the-trust-relationship"></a>Creación del lado del dominio administrado de la relación de confianza

Use el script `Add-AaddsResourceForestTrust` para crear el lado del dominio administrado de la relación de confianza. En primer lugar, instale el script `Add-AaddsResourceForestTrust` de la [Galería de PowerShell][powershell-gallery] mediante el cmdlet [Install-Script][Install-Script]:

```powershell
Install-Script -Name Add-AaddsResourceForestTrust
```

A continuación, proporcione al script la siguiente información:

| Nombre                               | Parámetro de script     | Descripción |
|:-----------------------------------|:---------------------|:------------|
| Nombre de dominio de Azure AD DS            | *-ManagedDomainFqdn* | FQDN del dominio administrado, como *aaddscontoso.com*. |
| Nombre de dominio de AD DS local      | *-TrustFqdn*         | FQDN del bosque de confianza, como *onprem.contoso.com*. |
| Nombre descriptivo de confianza                | *-TrustFriendlyName* | Nombre descriptivo de la relación de confianza. |
| Direcciones IP de DNS de la instancia de AD DS local | *-TrustDnsIPs*       | Lista delimitada por comas de las direcciones IPv4 del servidor DNS para el dominio de confianza que se muestra. |
| Contraseña de confianza                     | *-TrustPassword*     | Contraseña compleja para la relación de confianza. Esta contraseña también se especifica al crear la confianza de entrada unidireccional en la instancia de AD DS local. |
| Credenciales                        | *-Credentials*       | Credenciales usadas para autenticarse en Azure. El usuario debe estar en el *grupo de administradores de controlador AAD DC*. Si no se proporciona, el script solicita la autenticación. |

En el ejemplo siguiente se crea una relación de confianza denominada *myAzureADDSTrust* para *onprem.contoso.com*. Use sus propios nombres de parámetros y contraseñas.

```azure-powershell
Add-AaddsResourceForestTrust `
    -ManagedDomainFqdn "aaddscontoso.com" `
    -TrustFqdn "onprem.contoso.com" `
    -TrustFriendlyName "myAzureADDSTrust" `
    -TrustDnsIPs "10.0.1.10,10.0.1.11" `
    -TrustPassword <complexPassword>
```

> [!IMPORTANT]
> Recuerde la contraseña de confianza. Debe usar la misma contraseña cuando cree el lado de confianza local.

## <a name="configure-dns-in-the-on-premises-domain"></a>Configuración de DNS en el dominio local

Para resolver correctamente el dominio administrado desde el entorno local, quizá tenga que agregar reenviadores a los servidores DNS existentes. Si no ha configurado el entorno local para comunicarse con el dominio administrado, complete los pasos siguientes desde una estación de trabajo de administración para el dominio de AD DS local:

1. Seleccione **Inicio | Herramientas administrativas | DNS**.
1. Haga clic con el botón derecho en el servidor DNS, como *myAD01*, y seleccione **Propiedades**.
1. Elija **Reenviadores** y, después, **Editar** para agregar reenviadores adicionales.
1. Agregue las direcciones IP del dominio administrado, como *10.0.1.4* y *10.0.1.5*.
1. En un símbolo del sistema local, valide la resolución de nombres mediante el valor **nslookup** correspondiente al nombre de dominio del bosque de recursos del dominio administrado. Por ejemplo, `Nslookup aaddscontoso.com` debe devolver las dos direcciones IP del bosque de recursos del dominio administrado.

## <a name="create-inbound-forest-trust-in-the-on-premises-domain"></a>Creación de una confianza de bosque de entrada en el dominio local

El dominio de AD DS local necesita una confianza de bosque de entrada para el dominio administrado. Esta confianza se debe crear manualmente en el dominio de AD DS local, no se puede crear a partir de Azure Portal.

Para configurar la confianza de entrada en el dominio de AD DS local, complete los pasos siguientes desde una estación de trabajo de administración para el dominio de AD DS local:

1. Seleccione **Inicio | Herramientas administrativas | Dominios y confianzas de Active Directory**.
1. Haga clic derecho en el dominio, como *onprem.contoso.com*, y seleccione **Propiedades**.
1. Elija la pestaña **Confianzas** y, a continuación, **Nueva confianza**.
1. Escriba el nombre del dominio administrado, como *aaddscontoso.com* y, a continuación, seleccione **Siguiente**.
1. Seleccione la opción para crear una **Confianza de bosque** y, a continuación, para crear una confianza **Unidireccional: de entrada**.
1. Elija la opción para crear la confianza **Solo para este dominio**. En el paso siguiente, creará la confianza en Azure Portal para el dominio administrado.
1. Elija usar **Autenticación en todo el bosque** y después escriba y confirme una contraseña de confianza. Esta misma contraseña también se escribe en Azure Portal en la sección siguiente.
1. Deje las opciones predeterminadas de las siguientes ventanas y después elija la opción **No, no confirmar la confianza saliente**. No se puede validar la relación de confianza porque la cuenta de administrador delegada en el bosque de recursos del dominio administrado no tiene los permisos necesarios. Este comportamiento es así por diseño.
1. Seleccione **Finish** (Finalizar).

## <a name="validate-resource-authentication"></a>Validación de autenticación de recursos

Los siguientes escenarios habituales permiten validar que la confianza de bosque autentica correctamente a los usuarios y al acceso a los recursos:

* [Autenticación de usuarios locales desde el bosque de recursos de Azure AD DS](#on-premises-user-authentication-from-the-azure-ad-ds-resource-forest)
* [Acceso a los recursos del bosque de recursos de Azure AD DS mediante el usuario local](#access-resources-in-the-azure-ad-ds-resource-forest-using-on-premises-user)
    * [Habilitación del uso compartido de archivos e impresoras](#enable-file-and-printer-sharing)
    * [Creación de un grupo de seguridad e incorporación de miembros](#create-a-security-group-and-add-members)
    * [Creación de un recurso compartido de archivos para el acceso entre bosques](#create-a-file-share-for-cross-forest-access)
    * [Validación de la autenticación entre bosques en un recurso](#validate-cross-forest-authentication-to-a-resource)

### <a name="on-premises-user-authentication-from-the-azure-ad-ds-resource-forest"></a>Autenticación de usuarios locales desde el bosque de recursos de Azure AD DS

Debe tener una máquina virtual de Windows Server unida al dominio de recursos del dominio administrado. Use esta máquina virtual para probar si el usuario local puede autenticarse en una máquina virtual.

1. Conéctese a la VM de Windows Server unida al bosque de recursos del dominio administrado mediante el Escritorio remoto y sus credenciales de administrador del dominio administrado. Si obtiene un error de Autenticación a nivel de red (NLA), compruebe que la cuenta de usuario que usó no sea una cuenta de usuario de dominio.

    > [!TIP]
    > Para conectarse de forma segura a las máquinas virtuales unidas a Azure AD Domain Services, puede usar el [servicio de host de Azure Bastion](https://docs.microsoft.com/azure/bastion/bastion-overview) en las regiones de Azure admitidas.

1. Abra un símbolo del sistema y use el comando `whoami` para mostrar el nombre distintivo del usuario autenticado actualmente:

    ```console
    whoami /fqdn
    ```

1. Use el comando `runas` para autenticarse como un usuario del dominio local. En el siguiente comando, reemplace `userUpn@trusteddomain.com` por el UPN de un usuario del dominio local de confianza. El comando le solicita la contraseña de usuario:

    ```console
    Runas /u:userUpn@trusteddomain.com cmd.exe
    ```

1. Si la autenticación se realiza correctamente, se abre un nuevo símbolo del sistema. El título del nuevo símbolo del sistema incluye `running as userUpn@trusteddomain.com`.
1. Utilice `whoami /fqdn` en el nuevo símbolo del sistema para ver el nombre distintivo del usuario autenticado del entorno local de Active Directory.

### <a name="access-resources-in-the-azure-ad-ds-resource-forest-using-on-premises-user"></a>Acceso a los recursos del bosque de recursos de Azure AD DS mediante el usuario local

Con la VM de Windows Server unida al bosque de recursos del dominio administrado, puede probar el escenario en el que los usuarios pueden acceder a los recursos hospedados en el bosque de recursos, cuando se autentican desde los equipos del dominio local con los usuarios del dominio local. En los siguientes ejemplos se muestra cómo crear y probar varios escenarios habituales.

#### <a name="enable-file-and-printer-sharing"></a>Habilitación del uso compartido de archivos e impresoras

1. Conéctese a la VM de Windows Server unida al bosque de recursos del dominio administrado mediante el Escritorio remoto y sus credenciales de administrador del dominio administrado. Si obtiene un error de Autenticación a nivel de red (NLA), compruebe que la cuenta de usuario que usó no sea una cuenta de usuario de dominio.

    > [!TIP]
    > Para conectarse de forma segura a las máquinas virtuales unidas a Azure AD Domain Services, puede usar el [servicio de host de Azure Bastion](https://docs.microsoft.com/azure/bastion/bastion-overview) en las regiones de Azure admitidas.

1. Abra **Configuración de Windows** y busque y seleccione **Centro de redes y recursos compartidos**.
1. Elija la opción **Cambiar configuración de uso compartido avanzado**.
1. En **Perfil de dominio**, seleccione **Activar el uso compartido de archivos e impresoras** y, a continuación, **Guardar cambios**.
1. Cierre el **Centro de redes y recursos compartidos**.

#### <a name="create-a-security-group-and-add-members"></a>Creación de un grupo de seguridad e incorporación de miembros

1. Abra **Usuarios y equipos de Active Directory**.
1. Haga clic derecho en el nombre de dominio, elija **Nuevo** y después seleccione **Unidad organizativa**.
1. En el cuadro de nombre, escriba *LocalObjects* y seleccione **Aceptar**.
1. Seleccione y haga clic con el botón derecho en **LocalObjects** en el panel de navegación. Seleccione **Nuevo** y, a continuación, **Grupo**.
1. Escriba *FileServerAccess* en el cuadro **Nombre de grupo**. En **Ámbito de grupo**, seleccione **Dominio local** y, a continuación, elija **Aceptar**.
1. En el panel de contenido, haga doble clic en **FileServerAccess**. Seleccione **Miembros**, elija **Agregar** y, a continuación, seleccione **Ubicaciones**.
1. Seleccione el entorno local de Active Directory en la vista **Ubicación** y, a continuación, elija **Aceptar**.
1. Escriba *Usuarios del dominio* en el cuadro **Escriba los nombres de objeto que desea seleccionar**. Seleccione **Comprobar nombres**, proporcione las credenciales del entorno local de Active Directory y, a continuación, seleccione **Aceptar**.

    > [!NOTE]
    > Debe especificar las credenciales porque la relación de confianza solo es unidireccional. Esto implica que los usuarios del dominio administrado no pueden obtener acceso a los recursos ni buscar usuarios o grupos en el dominio de confianza (local).

1. El grupo **Usuarios del dominio** del entorno local de Active Directory debe ser miembro del grupo **FileServerAccess**. Seleccione **Aceptar** para guardar el grupo y cerrar la ventana.

#### <a name="create-a-file-share-for-cross-forest-access"></a>Creación de un recurso compartido de archivos para el acceso entre bosques

1. En la VM de Windows Server unida al bosque de recursos del dominio administrado, cree una carpeta y proporcione un nombre como *CrossForestShare*.
1. Haga clic con el botón derecho en la carpeta y elija **Propiedades**.
1. Seleccione la pestaña **Seguridad** y después **Editar**.
1. En el cuadro de diálogo *Permisos para CrossForestShare*, seleccione **Agregar**.
1. Escriba *FileServerAccess* en **Escriba los nombres de objeto que desea seleccionar** y, a continuación, seleccione **Aceptar**.
1. Seleccione *FileServerAccess* de la lista **Nombres de grupos o usuarios**. En la lista **Permisos para FileServerAccess**, elija *Permitir* para los permisos **Modificar** y **Escribir** y después seleccione **Aceptar**.
1. Seleccione la pestaña **Compartir** y después elija **Uso compartido avanzado...**
1. Elija **Compartir esta carpeta** y escriba un nombre fácil de recordar para el recurso compartido de archivos en **Nombre del recurso compartido**, como *CrossForestShare*.
1. Seleccione **Permisos**. En la lista **Permisos para todos**, elija **Permitir** para el permiso **Cambiar**.
1. Seleccione **Aceptar** dos veces y, a continuación, **Cerrar**.

#### <a name="validate-cross-forest-authentication-to-a-resource"></a>Validación de la autenticación entre bosques en un recurso

1. Inicie sesión en un equipo Windows unido a su entorno local de Active Directory mediante una cuenta de usuario del entorno local de Active Directory.
1. Con el **Explorador de Windows**, conéctese al recurso compartido que creó. Para ello, use el nombre de host completo y el recurso compartido, como en `\\fs1.aaddscontoso.com\CrossforestShare`.
1. Para validar el permiso de escritura, haga clic con el botón derecho en la carpeta, elija **Nuevo** y después seleccione **Documento de texto**. Use el nombre predeterminado **Nuevo documento de texto**.

    Si los permisos de escritura están configurados correctamente, se crea un nuevo documento de texto. Los pasos siguientes abrirán, editarán y eliminarán el archivo según corresponda.
1. Para validar el permiso de lectura, abra el **Nuevo documento de texto**.
1. Para validar el permiso de modificación, agregue texto al archivo y cierre el **Bloc de notas**. Cuando se le pregunte si quiere guardar los cambios, elija **Guardar**.
1. Para validar el permiso de eliminación, seleccione el **Nuevo documento de texto** y elija **Eliminar**. Elija **Sí** para confirmar la eliminación del archivo.

## <a name="update-or-remove-outbound-forest-trust"></a>Actualización o eliminación de la confianza del bosque de salida

Si necesita actualizar un bosque de salida unidireccional existente en el dominio administrado, puede usar los scripts `Get-AaddsResourceForestTrusts` y `Set-AaddsResourceForestTrust`. Estos scripts le ayudarán en aquellos escenarios en los que quiera actualizar el nombre descriptivo o la contraseña de confianza de un bosque. Para quitar una confianza de salida unidireccional del dominio administrado, puede usar el script `Remove-AaddsResourceForestTrust`. Debe quitar manualmente la confianza de bosque de entrada unidireccional en el bosque de AD DS local asociado.

### <a name="update-a-forest-trust"></a>Actualización de una confianza de bosque

Si funciona normalmente, el bosque de dominio administrado y el bosque local negocian entre ellos un proceso normal de actualización de contraseñas. Esto forma parte del proceso normal de seguridad de la relación de confianza de AD DS. No es necesario rotar manualmente la contraseña de confianza a menos que la relación de confianza haya experimentado un problema y quiera restablecer manualmente la contraseña conocida. Para obtener más información, consulte los [cambios de contraseña de objetos del dominio de confianza](concepts-forest-trust.md#tdo-password-changes).

En los siguientes pasos de ejemplo se muestra cómo actualizar una relación de confianza existente si necesita restablecer manualmente la contraseña de confianza de salida:

1. Instale los scripts `Get-AaddsResourceForestTrusts` y `Set-AaddsResourceForestTrust` de la [Galería de PowerShell][powershell-gallery] mediante el cmdlet [Install-Script][Install-Script]:

    ```powershell
    Install-Script -Name Get-AaddsResourceForestTrusts,Set-AaddsResourceForestTrust
    ```

1. Antes de poder actualizar una confianza existente, obtenga primero el recurso de confianza mediante el script de `Get-AaddsResourceForestTrusts`. En el ejemplo siguiente, la confianza existente se asigna a un objeto denominado *existingTrust*. Especifique su propio nombre de bosque de dominio administrado y el nombre de bosque local que quiere actualizar:

    ```powershell
    $existingTrust = Get-AaddsResourceForestTrust `
        -ManagedDomainFqdn "aaddscontoso.com" `
        -TrustFqdn "onprem.contoso.com" `
        -TrustFriendlyName "myAzureADDSTrust"
    ```

1. Para actualizar la contraseña de confianza existente, use el script `Set-AaddsResourceForestTrust`. Especifique el objeto de confianza existente del paso anterior y, a continuación, escriba una nueva contraseña de relación de confianza. PowerShell no aplica la opción de complejidad de contraseñas, por lo que debe asegurarse de generar y usar una contraseña segura para su entorno.

    ```powershell
    Set-AaddsResourceForestTrust `
        -Trust $existingTrust `
        -TrustPassword <newComplexPassword>
    ```

### <a name="delete-a-forest-trust"></a>Eliminación de una confianza de bosque

Si ya no necesita la confianza de bosque de salida unidireccional desde el dominio administrado hasta un bosque de AD DS local, puede quitarla. Asegúrese de que ninguna aplicación o servicio necesite autenticarse en el bosque local de AD DS antes de quitar la confianza. Asimismo, también debe quitar manualmente la confianza de entrada unidireccional en el bosque de AD DS local asociado.

1. Instale el script `Remove-AaddsResourceForestTrust` de la [Galería de PowerShell][powershell-gallery] mediante el cmdlet [Install-Script][Install-Script]:

    ```powershell
    Install-Script -Name Remove-AaddsResourceForestTrust
    ```

1. A continuación, quite la confianza de bosque mediante el script `Remove-AaddsResourceForestTrust`. En el ejemplo siguiente, se quita la confianza denominada *myAzureADDSTrust* entre el bosque de dominio administrado denominado *aaddscontoso.com* y el bosque local *onprem.contoso.com*. Especifique su propio nombre de bosque de dominio administrado y el nombre de bosque local que quiere quitar:

    ```powershell
    Remove-AaddsResourceForestTrust `
        -ManagedDomainFqdn "aaddscontoso.com" `
        -TrustFqdn "onprem.contoso.com" `
        -TrustFriendlyName "myAzureADDSTrust"
    ```

Para quitar la confianza de entrada unidireccional del bosque de AD DS local, conéctese a un equipo de administración que tenga acceso al bosque de AD DS local y realice los pasos siguientes:

1. Seleccione **Inicio | Herramientas administrativas | Dominios y confianzas de Active Directory**.
1. Haga clic derecho en el dominio, como *onprem.contoso.com*, y seleccione **Propiedades**.
1. Elija la pestaña **Confianzas** y, a continuación, seleccione la confianza entrante existente en el bosque de dominio administrado.
1. Seleccione **Quitar** y confirme que quiere quitar la confianza entrante.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido cómo:

> [!div class="checklist"]
> * Crear un bosque de recursos de dominio administrado mediante Azure PowerShell.
> * Crear una confianza de bosque de salida unidireccional en el dominio administrado mediante Azure PowerShell.
> * Configurar DNS en un entorno de AD DS local para admitir la conectividad del dominio administrado.
> * Crear una confianza de bosque de entrada unidireccional en un entorno local de AD DS
> * Probar y validar la relación de confianza para la autenticación y el acceso a los recursos.

Para obtener más información conceptual sobre los tipos de bosque de Azure AD DS, consulte [¿Qué son los bosques de recursos?][concepts-forest] y [¿Cómo funcionan las confianzas de bosque en Azure AD DS?][concepts-trust]

<!-- INTERNAL LINKS -->
[concepts-forest]: concepts-resource-forest.md
[concepts-trust]: concepts-forest-trust.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance-advanced]: tutorial-create-instance-advanced.md
[Connect-AzAccount]: /powershell/module/Az.Accounts/Connect-AzAccount
[Connect-AzureAD]: /powershell/module/AzureAD/Connect-AzureAD
[New-AzResourceGroup]: /powershell/module/Az.Resources/New-AzResourceGroup
[network-peering]: ../virtual-network/virtual-network-peering-overview.md
[New-AzureADServicePrincipal]: /powershell/module/AzureAD/New-AzureADServicePrincipal
[Get-AzureRMSubscription]: /powershell/module/AzureRM.Profile/Get-AzureRmSubscription
[Install-Script]: /powershell/module/powershellget/install-script

<!-- EXTERNAL LINKS -->
[powershell-gallery]: https://www.powershellgallery.com/