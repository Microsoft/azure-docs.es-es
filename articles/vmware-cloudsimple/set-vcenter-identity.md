---
title: 'Azure VMware Solution by CloudSimple: configuración de orígenes de identidad de vCenter en la nube privada'
description: Se describe cómo configurar vCenter de la nube privada para autenticarse con Active Directory de forma que los administradores de VMware accedan a él.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: f6f3b10219775adb02d47a91da2573ea99f30ac0
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212254"
---
# <a name="set-up-vcenter-identity-sources-to-use-active-directory"></a>Configuración de orígenes de identidad de vCenter para usar Active Directory

## <a name="about-vmware-vcenter-identity-sources"></a>Acerca de los orígenes de identidad de VMware vCenter

VMware vCenter admite distintos orígenes de identidad para la autenticación de los usuarios que acceden a vCenter.  Se puede configurar vCenter de la nube privada de CloudSimple para autenticarse con Active Directory para que sus administradores de VMware accedan a él. Una vez completada la instalación, el usuario **cloudowner** puede agregar usuarios del origen de identidad a vCenter.  

Puede configurar el dominio y los controladores de dominio de Active Directory de cualquiera de las maneras siguientes:

* Dominio y controladores de dominio de Active Directory que se ejecutan localmente
* Dominio y controladores de dominio de Active Directory que se ejecutan en Azure como máquinas virtuales de la suscripción de Azure
* Nuevos dominios y controladores de dominio de Active Directory que se ejecutan en la nube privada
* Servicio de Azure Active Directory

En esta guía se explican las tareas para configurar el dominio y los controladores de dominio de Active Directory que se ejecutan de forma local o como máquinas virtuales de las suscripciones.  Si quiere usar Azure AD como el origen de identidad, consulte [Uso de Azure AD como proveedor de identidades para vCenter en la nube privada de CloudSimple](azure-ad.md) para obtener instrucciones detalladas sobre cómo configurar el origen de identidad.

Antes de [agregar un origen de identidad](#add-an-identity-source-on-vcenter), [escale los privilegios de vCenter](escalate-private-cloud-privileges.md) temporalmente.

> [!CAUTION]
> Los nuevos usuarios solo se deben agregar a *Cloud-Owner-Group*, *Cloud-Global-Cluster-Admin-Group*, *Cloud-Global-Storage-Admin-Group*, *Cloud-Global-Network-Admin-Group* o *Cloud-Global-VM-Admin-Group*.  Los usuarios agregados al grupo *Administradores* se quitarán automáticamente.  Solo se deben agregar cuentas de servicio al grupo *Administradores* y estas cuentas no se deben usar para iniciar sesión en la interfaz de usuario web de vSphere.   


## <a name="identity-source-options"></a>Opciones de origen de identidad

* [Adición de Active Directory local como origen de identidad de inicio de sesión único](#add-on-premises-active-directory-as-a-single-sign-on-identity-source)
* [Configuración de una nueva instancia de Active Directory en una nube privada](#set-up-new-active-directory-on-a-private-cloud)
* [Configuración de Active Directory en Azure](#set-up-active-directory-on-azure)

> [!IMPORTANT]
> **No se admite Active Directory (Autenticación integrada de Windows).** Solo se admite la opción Active Directory a través de LDAP como origen de identidad.

## <a name="add-on-premises-active-directory-as-a-single-sign-on-identity-source"></a>Adición de Active Directory local como origen de identidad de inicio de sesión único

Para configurar la instancia de Active Directory local como un origen de identidad de inicio de sesión único, necesita lo siguiente:

* [Conexión VPN de sitio a sitio](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway) desde el centro de datos local hasta la nube privada.
* Agregar la dirección IP del servidor DNS local a vCenter y al controlador de servicios de plataforma (PSC).

Use la información de la tabla siguiente al configurar el dominio de Active Directory.

| **Opción** | **Descripción** |
|------------|-----------------|
| **Nombre** | Nombre del origen de identidad. |
| **DN base para usuarios** | Nombre distintivo base de los usuarios. |
| **Nombre de dominio** | FDQN del dominio, por ejemplo, ejemplo.com. No proporcione una dirección IP en este cuadro de texto. |
| **Alias de dominio** | Nombre NetBIOS del dominio. Si usa autenticaciones de SSPI, agregue el nombre de NetBIOS del dominio de Active Directory como un alias del origen de identidad. |
| **DN base para grupos** | Nombre distintivo base para los grupos. |
| **URL de servidor principal** | Servidor LDAP del controlador de dominio principal para el dominio.<br><br>Use el formato  `ldap://hostname:port`  o  `ldaps://hostname:port`. Normalmente, el puerto es 389 para las conexiones LDAP y 636 para las conexiones LDAPS. En el caso de las implementaciones de varios controladores de dominio de Active Directory, el puerto suele ser 3268 para LDAP y 3269 para LDAPS.<br><br>Cuando se usa  `ldaps://`  en la URL de LDAP principal o secundaria, se requiere un certificado que establezca la confianza para el punto de conexión LDAPS del servidor de Active Directory. |
| **URL de servidor secundario** | Dirección de un servidor LDAP de controlador de dominio secundario que se usa para la conmutación por error. |
| **Elegir certificado** | Si quiere usar LDAPS con el origen de identidad del servidor LDAP de Active Directory o del servidor de OpenLDAP, aparecerá el botón Elegir certificado cuando escriba  `ldaps://`  en el cuadro de texto de la URL. No se requiere una URL secundaria. |
| **Nombre de usuario** | Identificador de un usuario del dominio que tiene un mínimo de acceso de solo lectura al DN base para usuarios y grupos. |
| **Contraseña** | Contraseña del usuario que se especifica mediante el nombre de usuario. |

Cuando tenga la información de la tabla anterior, puede agregar su instancia de Active Directory local como un origen de identidad de inicio de sesión único en vCenter.

> [!TIP]
> Encontrará más información sobre los orígenes de identidad de inicio de sesión único en la [página de documentación de VMware](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.psc.doc/GUID-B23B1360-8838-4FF2-B074-71643C4CB040.html).

## <a name="set-up-new-active-directory-on-a-private-cloud"></a>Configuración de una nueva instancia de Active Directory en una nube privada

Puede configurar un nuevo dominio de Active Directory en la nube privada y usarlo como origen de identidad para el inicio de sesión único.  El dominio de Active Directory puede formar parte de un bosque existente de Active Directory o se puede configurar como un bosque independiente.

### <a name="new-active-directory-forest-and-domain"></a>Nuevo bosque y dominio de Active Directory

Para configurar un nuevo bosque y dominio de Active Directory, necesita:

* Una o varias máquinas virtuales que ejecuten Microsoft Windows Server para usarlas como controladores de dominio para el nuevo bosque y dominio de Active Directory.
* Una o varias máquinas virtuales que ejecuten el servicio DNS para la resolución de nombres.

Para ver los pasos detallados, consulte [Instalar un nuevo bosque de Active Directory de Windows Server 2012](/windows-server/identity/ad-ds/deploy/install-a-new-windows-server-2012-active-directory-forest--level-200-).

> [!TIP]
> Para lograr una alta disponibilidad de los servicios, se recomienda configurar varios controladores de dominio y servidores DNS.

Después de configurar el bosque y el dominio Active Directory, puede [agregar un origen de identidad en vCenter](#add-an-identity-source-on-vcenter) para la nueva instancia de Active Directory.

### <a name="new-active-directory-domain-in-an-existing-active-directory-forest"></a>Nuevo dominio de Active Directory en un bosque de Active Directory existente

Para configurar un nuevo dominio de Active Directory en un bosque de Active Directory existente, necesita lo siguiente:

* Conexión VPN de sitio a sitio a la ubicación del bosque de Active Directory.
* Servidor DNS para resolver el nombre del bosque de Active Directory existente.

Para ver los pasos detallados, consulte [Instalar un nuevo dominio secundario o de árbol de Active Directory de Windows Server 2012](/windows-server/identity/ad-ds/deploy/install-a-new-windows-server-2012-active-directory-child-or-tree-domain--level-200-).

Después de configurar el dominio de Active Directory, puede [agregar un origen de identidad en vCenter](#add-an-identity-source-on-vcenter) para la nueva instancia de Active Directory.

## <a name="set-up-active-directory-on-azure"></a>Configuración de Active Directory en Azure

Active Directory en Azure es similar a Active Directory en el entorno local.  Para configurar Active Directory en Azure como un origen de identidad de inicio de sesión único en vCenter, el servidor vCenter y el PSC deben tener conectividad de red con la red virtual de Azure donde se ejecutan los servicios de Active Directory.  Puede establecer esta conectividad mediante una [conexión de Azure Virtual Network con ExpressRoute](azure-expressroute-connection.md) desde la red virtual de Azure donde se ejecutan los servicios de Active Directory hasta la nube privada de CloudSimple.

Una vez establecida la conexión de red, siga los pasos descritos en [Incorporación de Active Directory local como un origen de identidad de inicio de sesión único](#add-on-premises-active-directory-as-a-single-sign-on-identity-source) para agregarlo como un origen de identidad.  

## <a name="add-an-identity-source-on-vcenter"></a>Adición de un origen de identidad en vCenter

1. [Escale los privilegios en la nube privada](escalate-private-cloud-privileges.md).

2. Inicie sesión en vCenter para su nube privada.

3. Seleccione **Home > Administration** (Inicio > Administración).

    ![Administración](media/OnPremAD01.png)

4. Seleccione **Single Sign On > Configuration** (inicio de sesión único > Configuración).

    ![Inicio de sesión único](media/OnPremAD02.png)

5. Abra la pestaña **Identity Sources** (Orígenes de identidad) y haga clic en **+** para agregar un nuevo origen de identidad.

    ![Orígenes de identidad](media/OnPremAD03.png)

6. Seleccione **Active Directory as an LDAP Server** (Active Directory como servidor LDAP) y haga clic en **Next** (Siguiente).

    ![Active Directory](media/OnPremAD04.png)

7. Especifique los parámetros de origen de identidad para su entorno y haga clic en **Next** (Siguiente).

    ![Active Directory](media/OnPremAD05.png)

8. Revise la configuración y haga clic en **Finish** (Finalizar).
