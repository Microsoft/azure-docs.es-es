---
title: 'Azure VMware Solution by CloudSimple: uso de Azure AD como origen de identidades en la nube privada'
description: En este artículo se describe cómo agregar Azure AD como proveedor de identidades en la nube privada de CloudSimple para autenticar a los usuarios que acceden a CloudSimple desde Azure.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 93922986dfe0b2b4e8ba0923931df601cc12428b
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90532535"
---
# <a name="use-azure-ad-as-an-identity-provider-for-vcenter-on-cloudsimple-private-cloud"></a>Uso de Azure AD como proveedor de identidades para vCenter en la nube privada de CloudSimple

Puede configurar vCenter de la nube privada de CloudSimple para realizar la autenticación con Azure Active Directory (Azure AD) para que sus administradores de VMware accedan a vCenter. Después de configurar el origen de identidades de inicio de sesión único, el usuario **cloudowner** puede agregar usuarios desde el origen de identidades a vCenter.  

Puede configurar el dominio y los controladores de dominio de Active Directory de cualquiera de las maneras siguientes:

* Dominio y controladores de dominio de Active Directory que se ejecutan localmente
* Dominio y controladores de dominio de Active Directory que se ejecutan en Azure como máquinas virtuales de la suscripción de Azure
* Nuevos dominios y controladores de dominio de Active Directory que se ejecutan en la nube privada de CloudSimple
* Servicio de Azure Active Directory

En esta guía se explican las tareas necesarias para configurar Azure AD como origen de identidades.  Para obtener información sobre el uso de una instancia de Active Directory local o una instancia de Active Directory que se ejecuta en Azure, consulte [Configuración de orígenes de identidades de vCenter para usar Active Directory](set-vcenter-identity.md) a fin de obtener instrucciones detalladas sobre cómo configurar el origen de identidades.

## <a name="about-azure-ad"></a>Acerca de Azure AD

Azure AD es el directorio basado en la nube multiinquilino y el servicio de administración de identidades de Microsoft.  Azure AD proporciona un mecanismo de autenticación escalable, coherente y fiable para que los usuarios se autentiquen y accedan a diferentes servicios de Azure.  Además, proporciona servicios LDAP seguros para que los servicios de terceros usen Azure AD como origen de autenticación o de identidades.  Azure AD combina servicios de directorio principales, la gobernanza de identidades avanzada y la administración del acceso a las aplicaciones, que se pueden usar para conceder acceso a la nube privada para los usuarios que administran la nube privada.

Para usar Azure AD como origen de identidades con vCenter, debe configurar Azure AD y los servicios de dominio de Azure AD. Siga estas instrucciones:

1. [Cómo configurar Azure AD y los servicios de dominio de Azure AD](#set-up-azure-ad-and-azure-ad-domain-services)
2. [Cómo configurar un origen de identidades en vCenter de la nube privada](#set-up-an-identity-source-on-your-private-cloud-vcenter)

## <a name="set-up-azure-ad-and-azure-ad-domain-services"></a>Configuración de Azure AD y los servicios de dominio de Azure AD

Antes de empezar, necesitará acceso a su suscripción de Azure con privilegios de administrador global.  En los pasos siguientes encontrará instrucciones generales. Para conocer los detalles, consulte la documentación de Azure.

### <a name="azure-ad"></a>Azure AD

> [!NOTE]
> Si ya tiene Azure AD, puede omitir esta sección.

1. Configure Azure AD en la suscripción como se describe en la [documentación de Azure AD](../active-directory/fundamentals/active-directory-whatis.md).
2. Habilite Azure Active Directory Premium en la suscripción como se describe en [Suscripción a las ediciones Azure Active Directory Premium](../active-directory/fundamentals/active-directory-get-started-premium.md).
3. Configure un nombre de dominio personalizado y compruébelo como se describe en [Incorporación del nombre de dominio personalizado mediante el portal de Azure Active Directory](../active-directory/fundamentals/add-custom-domain.md).
    1. Configure un registro DNS en el registrador de dominios con la información que se proporciona en Azure.
    2. Establezca el nombre de dominio personalizado como dominio principal.

También puede configurar otras características de Azure AD,  aunque son necesarias para habilitar la autenticación de vCenter con Azure AD.

### <a name="azure-ad-domain-services"></a>Azure AD Domain Services

> [!NOTE]
> Este es un paso importante para habilitar Azure AD como origen de identidades para vCenter.  Para evitar problemas, asegúrese de que todos los pasos se realizan correctamente.

1. Habilite Azure AD Domain Services como se describe en [Habilitación de Azure Active Directory Domain Services mediante Azure Portal](../active-directory-domain-services/tutorial-create-instance.md).
2. Configure la red que usará Azure AD Domain Services como se describe en [Habilitación de Azure Active Directory Domain Services mediante Azure Portal](../active-directory-domain-services/tutorial-create-instance.md).
3. Configure el grupo de administradores para administrar Azure AD Domain Services como se describe en [Habilitación de Azure Active Directory Domain Services mediante Azure Portal](../active-directory-domain-services/tutorial-create-instance.md).
4. Actualice la configuración de DNS para Azure AD Domain Services como se describe en [Habilitación de Azure Active Directory Domain Services](../active-directory-domain-services/tutorial-create-instance.md).  Si quiere conectarse a AD a través de Internet, configure el registro DNS para la dirección IP pública de Azure AD Domain Services en el nombre de dominio.
5. Habilite la sincronización de hash de contraseñas para los usuarios.  Este paso habilita la sincronización de los valores hash de las contraseñas necesarios para la autenticación NT LAN Manager (NTLM) y Kerberos en Azure AD Domain Services. Una vez configurada la sincronización de los valores hash de las contraseñas, los usuarios pueden iniciar sesión en el dominio administrado mediante sus credenciales corporativas. Consulte [Habilitación de la sincronización de los valores hash de las contraseñas con Azure Active Directory Domain Services](../active-directory-domain-services/tutorial-create-instance.md).
    1. Si hay presentes usuarios solo de nube, deben cambiar su contraseña mediante el <a href="https://myapps.microsoft.com/" target="_blank">panel de acceso de Azure AD</a> para asegurarse de que los valores hash de las contraseñas se almacenan en el formato requerido por NTLM o Kerberos.  Siga las instrucciones indicadas en [Habilitar la sincronización de los valores hash de las contraseñas con el dominio administrado para las cuentas de usuario solo de nube](../active-directory-domain-services/tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds).  Este paso debe realizarse para los usuarios individuales y para todos los usuarios nuevos que se creen en el directorio de Azure AD con los cmdlets de PowerShell de Azure AD o Azure Portal. Los usuarios que necesitan acceso a Azure AD Domain Services deben usar el <a href="https://myapps.microsoft.com/" target="_blank">panel de acceso de Azure AD</a> y acceder a su perfil para cambiar la contraseña.

        > [!NOTE]
        > Si su organización tiene cuentas de usuario solo de nube, todos los usuarios que tengan que usar Azure Active Directory Domain Services deben cambiar sus contraseñas. Una cuenta de usuario solo de nube es una cuenta creada en su directorio de Azure AD mediante Azure Portal, o bien mediante cmdlets de PowerShell de Azure AD. Estas cuentas de usuario no se sincronizan desde un directorio local.

    2. Si va a sincronizar contraseñas de Active Directory local, siga los pasos que se describen en la [documentación de Active Directory](../active-directory-domain-services/tutorial-configure-password-hash-sync.md).

6.  Configure LDAP seguro en Azure Active Directory Domain Services como se describe en [Configuración de LDAP seguro (LDAPS) para un dominio administrado con Azure AD Domain Services](../active-directory-domain-services/tutorial-configure-ldaps.md).
    1. Cargue un certificado para su uso con LDAP seguro como se describe en el tema de Azure sobre la [obtención de certificados para LDAP seguro](../active-directory-domain-services/tutorial-configure-ldaps.md#create-a-certificate-for-secure-ldap).  CloudSimple recomienda que se use un certificado firmado emitido por una entidad de certificación para asegurarse de que vCenter pueda confiar en el certificado.
    2. Habilite LDAP seguro como se describe en [Configuración de LDAP seguro (LDAPS) para un dominio administrado con Azure AD Domain Services](../active-directory-domain-services/tutorial-configure-ldaps.md).
    3. Guarde la parte pública del certificado (sin la clave privada) en formato .cer para su uso con vCenter mientras configura el origen de identidades.
    4. Si se requiere acceso a Internet para Azure AD Domain Services, habilite la opción "Allow secure access to LDAP over internet" (Permitir el acceso seguro a LDAP a través de Internet).
    5. Agregue la regla de seguridad de entrada para el NSG de Azure AD Domain Services para el puerto TCP 636.

## <a name="set-up-an-identity-source-on-your-private-cloud-vcenter"></a>Configuración de un origen de identidades en vCenter de la nube privada

1. [Escale privilegios](escalate-private-cloud-privileges.md) para vCenter de la nube privada.
2. Recopile los parámetros de configuración necesarios para configurar el origen de identidades.

    | **Opción** | **Descripción** |
    |------------|-----------------|
    | **Nombre** | Nombre del origen de identidad. |
    | **DN base para usuarios** | Nombre distintivo base de los usuarios.  Para Azure AD, use: `OU=AADDC Users,DC=<domain>,DC=<domain suffix>`. Ejemplo: `OU=AADDC Users,DC=cloudsimplecustomer,DC=com`.|
    | **Nombre de dominio** | FDQN del dominio, por ejemplo, ejemplo.com. No proporcione una dirección IP en este cuadro de texto. |
    | **Alias de dominio** | *(Opcional)* Nombre de NetBIOS del dominio. Si usa autenticaciones de SSPI, agregue el nombre de NetBIOS del dominio de Active Directory como un alias del origen de identidad. |
    | **DN base para grupos** | Nombre distintivo base para los grupos. Para Azure AD, use: `OU=AADDC Users,DC=<domain>,DC=<domain suffix>`. Ejemplo: `OU=AADDC Users,DC=cloudsimplecustomer,DC=com`.|
    | **URL de servidor principal** | Servidor LDAP del controlador de dominio principal para el dominio.<br><br>Use el formato `ldaps://hostname:port`. Normalmente, el puerto es el 636 para las conexiones LDAPS. <br><br>Cuando se usa  `ldaps://`  en la URL de LDAP principal o secundaria, se requiere un certificado que establezca la confianza para el punto de conexión LDAPS del servidor de Active Directory. |
    | **URL de servidor secundario** | Dirección de un servidor LDAP de controlador de dominio secundario que se usa para la conmutación por error. |
    | **Elegir certificado** | Si quiere usar LDAPS con el origen de identidad del servidor LDAP de Active Directory o del servidor de OpenLDAP, aparecerá el botón Elegir certificado cuando escriba  `ldaps://`  en el cuadro de texto de la URL. No se requiere una URL secundaria. |
    | **Nombre de usuario** | Identificador de un usuario del dominio que tiene un mínimo de acceso de solo lectura al DN base para usuarios y grupos. |
    | **Contraseña** | Contraseña del usuario que se especifica mediante el nombre de usuario. |

3. Inicie sesión en vCenter de la nube privada una vez que se hayan escalado los privilegios.
4. Siga las instrucciones indicadas en [Agregar un origen de identidades en vCenter](set-vcenter-identity.md#add-an-identity-source-on-vcenter) con los valores del paso anterior para configurar Azure Active Directory como origen de identidades.
5. Agregue usuarios o grupos de Azure AD a los grupos de vCenter como se describe en el tema de VMware [Incorporación de miembros a un grupo de inicio de sesión único de vCenter](https://docs.vmware.com/en/VMware-vSphere/5.5/com.vmware.vsphere.security.doc/GUID-CDEA6F32-7581-4615-8572-E0B44C11D80D.html).

> [!CAUTION]
> Los nuevos usuarios solo se deben agregar a *Cloud-Owner-Group*, *Cloud-Global-Cluster-Admin-Group*, *Cloud-Global-Storage-Admin-Group*, *Cloud-Global-Network-Admin-Group* o *Cloud-Global-VM-Admin-Group*.  Los usuarios agregados al grupo *Administradores* se quitarán automáticamente.  Solo se deben agregar cuentas de servicio al grupo *Administradores*.

## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre el modelo de permisos de la nube privada](learn-private-cloud-permissions.md)
