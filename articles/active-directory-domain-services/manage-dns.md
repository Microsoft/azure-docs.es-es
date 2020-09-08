---
title: Administración de DNS para Azure AD Domain Services | Microsoft Docs
description: Obtenga información sobre cómo instalar las herramientas del servidor DNS para administrar DNS y crear reenviadores condicionales para un dominio administrado de Azure Active Directory Domain Services.
author: iainfoulds
manager: daveba
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/06/2020
ms.author: iainfou
ms.openlocfilehash: af321f4e9ef9e340026852a759bb70b9f03c77f6
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88722949"
---
# <a name="administer-dns-and-create-conditional-forwarders-in-an-azure-active-directory-domain-services-managed-domain"></a>Administración de DNS y creación de reenviadores condicionales en un dominio administrado de Azure Active Directory Domain Services

En Azure Active Directory Domain Services (Azure AD DS), un componente clave es DNS (resolución de nombres de dominio). Azure AD DS incluye un servidor DNS que proporciona la resolución de nombres para el dominio administrado. Este servidor DNS incluye registros y actualizaciones de DNS integrados para los componentes clave que permiten la ejecución del servicio.

Cuando ejecute sus propias aplicaciones y servicios, podría tener que crear registros de DNS para las máquinas que no están unidas al dominio, configurar direcciones IP virtuales para los equilibradores de carga o configurar reenviadores DNS externos. Los usuarios que pertenecen al grupo *Administradores del controlador de dominio de AAD* reciben privilegios de administración de DNS en el dominio administrado con Azure AD DS y pueden crear y editar registros de DNS personalizados.

En un entorno híbrido, las zonas DNS y los registros configurados en otros espacios de nombres DNS, como un entorno de AD DS local, no se sincronizan con el dominio administrado. Para resolver los recursos con nombre en otros espacios de nombres DNS, cree y use reenviadores condicionales que apunten a los servidores DNS existentes de su entorno.

En este artículo se muestra cómo se instalan las herramientas del servidor DNS y cómo se usa la consola DNS para administrar registros y crear reenviadores condicionales en Azure AD DS.

## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, complete el tutorial para [crear y configurar un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Conectividad desde la red virtual de Azure AD DS a donde se hospedan sus otros espacios de nombres DNS.
    * Esta conectividad se puede proporcionar con una conexión de [Azure ExpressRoute][expressroute] o [Azure VPN Gateway][vpn-gateway].
* Una máquina virtual de administración de Windows Server que está unida al dominio administrado.
    * Si es necesario, complete el tutorial para [crear una máquina virtual de Windows Server y unirla a un dominio administrado][create-join-windows-vm].
* Una cuenta de usuario que sea miembro del grupo de *administradores de Azure AD DC* en el inquilino de Azure AD.

## <a name="install-dns-server-tools"></a>Instalación de las herramientas del servidor DNS

Para crear y modificar registros DNS en un dominio administrado, debe instalar las herramientas del servidor DNS. Estas herramientas se pueden instalar como una característica en Windows Server. Para obtener más información sobre cómo instalar las herramientas administrativas en un cliente de Windows, consulte cómo se instalan las [herramientas de administración remota del servidor (RSAT)][install-rsat].

1. Inicie sesión en la máquina virtual de administración. Si quiere conocer los pasos para conectarse mediante Azure Portal, consulte [Conexión a una máquina virtual de Windows Server][connect-windows-server-vm].
1. Si **Administrador del servidor** no se abre de forma predeterminada al iniciar sesión en la máquina virtual, seleccione el menú **Inicio** y, a continuación, elija **Administrador del servidor**.
1. En el *Panel de información* de la ventana **Administrador del servidor**, seleccione **Agregar roles y características**.
1. En la página **Antes de comenzar** del *Asistente para agregar roles y características*, seleccione **Siguiente**.
1. En *Tipo de instalación*, deje activada la opción **Instalación basada en características o en roles** y seleccione **Siguiente**.
1. En la página **Selección de servidor**, elija la máquina virtual actual del grupo de servidores, por ejemplo *mivm.aaddscontoso.com*, y seleccione **Siguiente**.
1. En la página **Roles de servidor**, haga clic en **Siguiente**.
1. En la página **Características**, expanda el nodo **Herramientas de administración remota del servidor** y el nodo **Herramientas de administración de roles**. Seleccione la característica **Herramientas del servidor DNS** en la lista de herramientas de administración de roles.

    ![Selección para instalar las herramientas del servidor DNS en la lista de herramientas de administración de roles disponibles](./media/manage-dns/install-dns-tools.png)

1. En la página **Confirmación**, seleccione **Instalar**. Puede tardar un minuto o dos en instalar las herramientas del servidor DNS.
1. Cuando finalice la instalación de la característica, haga clic en **Cerrar** para salir del Asistente para **Agregar roles y características**.

## <a name="open-the-dns-management-console-to-administer-dns"></a>Apertura de la consola de administración de DNS para administrar DNS

Con las herramientas del servidor DNS instaladas, puede administrar los registros de DNS en el dominio administrado.

> [!NOTE]
> Para administrar DNS en un dominio administrado, debe haber iniciado sesión en una cuenta de usuario que sea miembro del grupo *Administradores del controlador de dominio de AAD*.

1. En la pantalla Inicio, seleccione **Herramientas administrativas**. Se muestra una lista de las herramientas de administración disponibles, incluido **DNS**, instalado en la sección anterior. Seleccione **DNS** para iniciar la consola de administración de DNS.
1. En el cuadro de diálogo **Conectar a servidor DNS**, seleccione **El siguiente equipo** y escriba el nombre de dominio DNS del dominio administrado, como *aaddscontoso.com*:

    ![Conexión al dominio administrado en la consola DNS](./media/manage-dns/connect-dns-server.png)

1. La consola DNS se conecta al dominio administrado que se ha especificado. Expanda las **Zonas de búsqueda directa** o las **Zonas de búsqueda inversa** para crear las entradas DNS necesarias, o bien edite los registros existentes según sea necesario.

    ![Consola DNS: administración del dominio](./media/manage-dns/dns-manager.png)

> [!WARNING]
> Cuando administre registros con las herramientas del servidor DNS, asegúrese de no eliminar ni modificar los registros DNS integrados que usa Azure AD DS. Los registros DNS integrados incluyen registros DNS de dominio, registros de servidor de nombres y otros registros usados para la ubicación del controlador de dominio. Si modifica estos registros, se interrumpirán los servicios de dominio en la red virtual.

## <a name="create-conditional-forwarders"></a>Creación de reenviadores condicionales

Una zona DNS de Azure AD DS solo debe contener la zona y los registros para el propio dominio administrado. No cree zonas adicionales en el dominio administrado para resolver recursos con nombre en otros espacios de nombres DNS. En su lugar, utilice reenviadores condicionales en el dominio administrado para indicar al servidor DNS a dónde debe ir para resolver las direcciones de esos recursos.

Un reenviador condicional es una opción de configuración en un servidor DNS que le permite definir un dominio DNS, por ejemplo, *contoso.com*, a donde reenviar las consultas. En lugar de que el servidor DNS local intente resolver las consultas para los registros de ese dominio, las consultas de DNS se reenvían al DNS configurado para ese dominio. Esta configuración garantiza que se devuelvan los registros DNS correctos, ya que no se crea una zona DNS local con registros duplicados en el dominio administrado para reflejar esos recursos.

Para crear un reenviador condicional en su dominio administrado, complete los siguientes pasos:

1. Seleccione la zona DNS, por ejemplo, *aaddscontoso.com*.
1. Seleccione **Reenviadores condicionales**, haga clic con el botón derecho y seleccione **Nuevo reenviador condicional…**
1. Escriba el otro **Dominio DNS**, por ejemplo, *contoso.com*, y escriba las direcciones IP de los servidores DNS para ese espacio de nombres, tal como se muestra en el ejemplo siguiente:

    ![Adición y configuración de un reenviador condicional para el servidor DNS](./media/manage-dns/create-conditional-forwarder.png)

1. Active la casilla **Almacenar este reenviador condicional en Active Directory y replicarlo como sigue**, a continuación, seleccione la opción *Todos los servidores DNS en este dominio*, como se muestra en el ejemplo siguiente:

    ![Consola DNS: seleccione Todos los servidores DNS en este dominio.](./media/manage-dns/store-in-domain.png)

    > [!IMPORTANT]
    > Si el reenviador condicional está almacenado en el *bosque* en lugar de en el *dominio*, se produce un error en el reenviador condicional.

1. Para crear el reenviador condicional, seleccione **Aceptar**.

La resolución de nombres de los recursos en otros espacios de nombres de las VM conectadas al dominio administrado deberían resolverse correctamente. Las consultas para el dominio DNS configurado en el reenviador condicional se pasan a los servidores DNS correspondientes.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la administración de DNS, consulte el artículo [Herramientas de DNS](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753579(v=ws.11))de TechNet.

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[expressroute]: ../expressroute/expressroute-introduction.md
[vpn-gateway]: ../vpn-gateway/vpn-gateway-about-vpngateways.md
[create-join-windows-vm]: join-windows-vm.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md
[connect-windows-server-vm]: join-windows-vm.md#connect-to-the-windows-server-vm

<!-- EXTERNAL LINKS -->
[install-rsat]: /windows-server/remote/remote-server-administration-tools#BKMK_Thresh