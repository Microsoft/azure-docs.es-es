---
title: Habilitación del servicio de perfil de usuario de SharePoint con Azure AD DS | Microsoft Docs
description: Aprenda a configurar un dominio administrado de Azure Active Directory Domain Services para admitir la sincronización de perfiles para SharePoint Server
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
ms.openlocfilehash: aa66f7150e5c3ac4560d7f97cb73b1effb8eec89
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88749628"
---
# <a name="configure-azure-active-directory-domain-services-to-support-user-profile-synchronization-for-sharepoint-server"></a>Configuración de Azure Active Directory Domain Services para admitir la sincronización de perfiles de usuario para SharePoint Server

SharePoint Server incluye un servicio para sincronizar perfiles de usuario. Esta característica permite que los perfiles de usuario se almacenen en una ubicación central y estén accesibles en varios sitios y granjas de servidores de SharePoint. Para configurar el servicio de perfiles de usuario de SharePoint Server, se deben conceder los permisos adecuados en un dominio administrado de Azure Active Directory Domain Services (Azure AD DS). Para más información, consulte el artículo de [sincronización de perfiles de usuario en SharePoint Server](/SharePoint/administration/user-profile-service-administration).

En él se muestra cómo configurar Azure AD DS para permitir el servicio de sincronización de perfiles de usuario de SharePoint Server.

## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, complete el tutorial para [crear y configurar un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Una máquina virtual de administración de Windows Server que esté unida al dominio administrado de Azure AD DS.
    * Si es necesario, complete el tutorial para [crear una máquina virtual de administración][tutorial-create-management-vm].
* Una cuenta de usuario que sea miembro del grupo de *administradores de Azure AD DC* en el inquilino de Azure AD.
* Una cuenta de servicio de SharePoint para el servicio de sincronización de perfiles de usuario.
    * Si es necesario, consulte [Plan de cuentas administrativas y de servicio en SharePoint Server][sharepoint-service-account].

## <a name="service-accounts-overview"></a>Introducción a las cuentas de servicio

En un dominio administrado, existe un grupo de seguridad denominado *AAD DC Service Accounts* (Cuentas de servicio de controlador de dominio de AAD) como parte de la unidad organizativa *Usuarios*. A los miembros de este grupo de seguridad se les delegan los privilegios siguientes:

- El privilegio **Replicate Directory Changes** (Replicación de cambios de directorio) en el atributo DSE raíz.
- El privilegio **Replicate Directory Changes** (Replicación de cambios de directorio) en el contexto de nomenclatura de *Configuración* (contenedor `cn=configuration`).

El grupo de seguridad *AAD DC Service Accounts* (Cuentas de servicio de controlador de dominio de AAD) también es un miembro del grupo integrado *Acceso de compatible con versiones anteriores de Windows 2000*.

Al agregarla a este grupo de seguridad, a la cuenta de servicio para el servicio de sincronización de perfiles de usuario de SharePoint Server se le conceden los privilegios necesarios para funcionar correctamente.

## <a name="enable-support-for-sharepoint-server-user-profile-sync"></a>Habilitación de la compatibilidad para la sincronización de perfiles de usuario de SharePoint Server

La cuenta de servicio de SharePoint Server necesita los privilegios adecuados para replicar los cambios en el directorio y permitir que la sincronización de perfiles de usuario de SharePoint Server funcione correctamente. Para otorgar estos privilegios, agregue la cuenta de servicio usada para la sincronización de perfiles de usuario de SharePoint al grupo *AAD DC Service Accounts* (Cuentas de servicio de controlador de dominio de AAD).

Desde la máquina virtual de administración de Azure AD DS, realice los siguientes pasos:

> [!NOTE]
> Para editar la pertenencia a un grupo en un dominio administrado, debe haber iniciado sesión en una cuenta de usuario que sea miembro del grupo *Administradores del controlador de dominio de AAD*.

1. En la pantalla Inicio, seleccione **Herramientas administrativas**. Se muestra una lista de las herramientas de administración disponibles que se instalaron en el tutorial para [crear una máquina virtual de administración][tutorial-create-management-vm].
1. Para administrar la pertenencia a un grupo, seleccione **Centro de administración de Active Directory** de la lista de herramientas administrativas.
1. En el panel izquierdo, elija el dominio administrado, como *aaddscontoso.com*. Aparecerá una lista de las unidades organizativas y los recursos existentes.
1. Seleccione la unidad organizativa **Usuarios** y elija el grupo de seguridad *AAD DC Service Accounts* (Cuentas de servicio del controlador de dominio de AAD).
1. Seleccione **Miembros** y elija **Agregar...**
1. Escriba el nombre de la cuenta de servicio de SharePoint y seleccione **Aceptar**. En el ejemplo siguiente, la cuenta de servicio de SharePoint se denomina *spadmin*:

    ![Incorporación de la cuenta de servicio de SharePoint al grupo de seguridad AAD DC Service Accounts (Cuentas de servicio del controlador de dominio de AAD)](./media/deploy-sp-profile-sync/add-member-to-aad-dc-service-accounts-group.png)

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte [Administración de la sincronización de perfiles de usuario en SharePoint Server](/SharePoint/administration/manage-profile-synchronization).

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md

<!-- EXTERNAL LINKS -->
[sharepoint-service-account]: /sharepoint/security-for-sharepoint-server/plan-for-administrative-and-service-accounts