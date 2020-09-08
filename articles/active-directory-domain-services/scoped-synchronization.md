---
title: Sincronización con ámbito en Azure AD Domain Services | Microsoft Docs
description: Aprenda a usar Azure Portal para configurar la sincronización con ámbito entre Azure AD y un dominio administrado de Azure Active Directory Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 9389cf0f-0036-4b17-95da-80838edd2225
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/24/2020
ms.author: iainfou
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: fe373b9aa08d6999a9c57dd0008a207071e1dddd
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89068905"
---
# <a name="configure-scoped-synchronization-from-azure-ad-to-azure-active-directory-domain-services-using-the-azure-portal"></a>Configuración de la sincronización con ámbito entre Azure AD y Azure Active Directory Domain Services mediante Azure Portal

Para proporcionar servicios de autenticación, Azure Active Directory Domain Services (Azure AD DS) sincroniza los usuarios y grupos de Azure AD. En un entorno híbrido, los usuarios y grupos de un entorno local de Active Directory Domain Services (AD DS) se pueden sincronizar primero con Azure AD mediante Azure AD Connect y, luego, con un dominio administrado de Azure AD DS.

De forma predeterminada, todos los usuarios y grupos de un directorio de Azure AD se sincronizan con un dominio administrado. Si tiene necesidades específicas, puede optar por sincronizar solo un conjunto definido de usuarios.

En este artículo se muestra cómo configurar la sincronización con ámbito y, luego, cambiar o deshabilitar el conjunto de usuarios con ámbito mediante Azure Portal. También puede [completar estos pasos con PowerShell][scoped-sync-powershell].

## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, complete el tutorial para [crear y configurar un dominio administrado de Azure Active Directory Domain Services][tutorial-create-instance].
* Necesita privilegios de *administrador global* en el inquilino de Azure AD para cambiar el ámbito de sincronización de Azure AD DS.

## <a name="scoped-synchronization-overview"></a>Introducción a la sincronización con ámbito

De forma predeterminada, todos los usuarios y grupos de un directorio de Azure AD se sincronizan con un dominio administrado. Si solo unos cuantos usuarios necesitan acceso al dominio administrado, puede sincronizar únicamente esas cuentas de usuario. Esta sincronización son ámbito se basa en grupos. Al configurar la sincronización con ámbito basada en grupos, solo las cuentas de usuario que pertenecen a los grupos que especifique se sincronizan con el dominio administrado. Los grupos anidados no se sincronizan, solo los grupos específicos que seleccione.

Puede cambiar el ámbito de sincronización al crear el dominio administrado o una vez implementado. También puede cambiar el ámbito de sincronización en un dominio administrado existente sin necesidad de volver a crearlo.

Para más información sobre el proceso de sincronización , consulte [Funcionamiento de la sincronización en Azure AD Domain Services][concepts-sync].

> [!WARNING]
> Al cambiar el ámbito de sincronización el dominio administrado vuelve a sincronizar todos los datos. Se aplican las siguientes consideraciones:
>
>  * Al cambiar el ámbito de sincronización de un dominio administrado, se produce una resincronización completa.
>  * Los objetos que ya no son necesarios en el dominio administrado se eliminan. Se crean objetos en el dominio administrado.

## <a name="enable-scoped-synchronization"></a>Habilitación de la sincronización con ámbito

Para habilitar la sincronización con ámbito en Azure Portal, complete los pasos siguientes:

1. En Azure Portal, busque y seleccione **Azure AD Domain Services**. Elija el dominio administrado como, por ejemplo, *aaddscontoso.com*.
1. Seleccione **Sincronización** en el menú de la izquierda.
1. En *Tipo de sincronización*, seleccione **Ámbito**.
1. Elija **Seleccionar grupos** y después busque y seleccione los grupos que quiere agregar.
1. Cuando se realicen todos los cambios, seleccione **Guardar ámbito de sincronización**.

Al cambiar el ámbito de sincronización el dominio administrado vuelve a sincronizar todos los datos. Los objetos que ya no son necesarios en el dominio administrado se eliminan y la resincronización puede tardar un poco en completarse.

## <a name="modify-scoped-synchronization"></a>Modificación de la sincronización con ámbito

Para modificar la lista de grupos cuyos usuarios deben sincronizarse con el dominio administrado, lleve a cabo los pasos siguientes:

1. En Azure Portal, busque y seleccione **Azure AD Domain Services**. Elija el dominio administrado como, por ejemplo, *aaddscontoso.com*.
1. Seleccione **Sincronización** en el menú de la izquierda.
1. Para agregar un grupo, elija **+ Seleccionar grupos** en la parte superior y, después, seleccione los grupos que quiere agregar.
1. Para quitar un grupo del ámbito de sincronización, selecciónelo en la lista de grupos actualmente sincronizados y elija **Quitar grupos**.
1. Cuando se realicen todos los cambios, seleccione **Guardar ámbito de sincronización**.

Al cambiar el ámbito de sincronización el dominio administrado vuelve a sincronizar todos los datos. Los objetos que ya no son necesarios en el dominio administrado se eliminan y la resincronización puede tardar un poco en completarse.

## <a name="disable-scoped-synchronization"></a>Deshabilitación de la sincronización con ámbito

Para deshabilitar la sincronización con ámbito basada en grupos para un dominio administrado, lleve a cabo los pasos siguientes:

1. En Azure Portal, busque y seleccione **Azure AD Domain Services**. Elija el dominio administrado como, por ejemplo, *aaddscontoso.com*.
1. Seleccione **Sincronización** en el menú de la izquierda.
1. Cambie el *Tipo de sincronización* de **Ámbito** a **Todo** y, luego, seleccione **Guardar ámbito de sincronización**.

Al cambiar el ámbito de sincronización el dominio administrado vuelve a sincronizar todos los datos. Los objetos que ya no son necesarios en el dominio administrado se eliminan y la resincronización puede tardar un poco en completarse.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el proceso de sincronización , consulte [Funcionamiento de la sincronización en Azure AD Domain Services][concepts-sync].

<!-- INTERNAL LINKS -->
[scoped-sync-powershell]: powershell-scoped-synchronization.md
[concepts-sync]: synchronization.md
[tutorial-create-instance]: tutorial-create-instance.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
