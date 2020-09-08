---
title: Autorización del acceso a Azure App Configuration con Azure Active Directory
description: Habilitar RBAC para autorizar el acceso a la instancia de Azure App Configuration
author: lisaguthrie
ms.author: lcozzens
ms.date: 02/13/2020
ms.topic: conceptual
ms.service: azure-app-configuration
ms.openlocfilehash: 8889e7270127aa3991adb3c0575a4bce96090db2
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87830078"
---
# <a name="authorize-access-to-azure-app-configuration-using-azure-active-directory"></a>Autorización del acceso a Azure App Configuration con Azure Active Directory
Además de usar el Código de autenticación de mensajes basado en hash (HMAC), Azure App Configuration admite el uso de Azure Active Directory (Azure AD) para autorizar solicitudes a instancias de App Configuration.  Azure AD permite usar el control de acceso basado en roles (RBAC) para conceder permisos a una entidad de seguridad.  Una entidad de seguridad puede ser un usuario, una [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md) o una [entidad de servicio de aplicación](../active-directory/develop/app-objects-and-service-principals.md).  Para más información sobre los roles y las asignaciones de roles, consulte [Descripción de los distintos roles](../role-based-access-control/overview.md).

## <a name="overview"></a>Información general
Las solicitudes realizadas por una entidad de seguridad para acceder a un recurso de App Configuration deben ser autorizadas. Con Azure AD, el acceso a un recurso es un proceso de dos pasos:
1. Se autentica la identidad de la entidad de seguridad y se devuelve un token de OAuth 2.0.  El nombre de recurso para solicitar un token es `https://login.microsoftonline.com/{tenantID}`, donde `{tenantID}` coincide con el identificador de inquilino de Azure Active Directory al que pertenece la entidad de servicio.
2. El token se pasa como parte de una solicitud al servicio App Configuration para autorizar el acceso al recurso especificado.

El paso de autenticación exige que una solicitud de aplicación contenga un token de acceso de OAuth 2.0 en tiempo de ejecución.  Si una aplicación se ejecuta dentro de una entidad de Azure, como una aplicación de Azure Functions, una aplicación web de Azure o una máquina virtual de Azure, puede usar una identidad administrada para acceder a los recursos.  Para más información sobre cómo autenticar solicitudes realizadas por una identidad administrada a Azure App Configuration, consulte [Autenticación del acceso a recursos de Azure App Configuration con Azure Active Directory e identidades administradas para los recursos de Azure](howto-integrate-azure-managed-service-identity.md).

El paso de autorización exige que se asignen uno o varios roles de Azure a la entidad de seguridad. Azure App Configuration proporciona roles de Azure que abarcan conjuntos de permisos para recursos de App Configuration. Los roles que se asignan a una entidad de seguridad determinan los permisos que se proporcionan a la entidad de seguridad. Para más información sobre los roles de Azure, consulte [Roles integrados de Azure para Azure App Configuration](#azure-built-in-roles-for-azure-app-configuration). 

## <a name="assign-azure-roles-for-access-rights"></a>Asignación de roles de Azure para derechos de acceso
Azure Active Directory (Azure AD) autoriza derechos de acceso a recursos protegidos mediante el [control de acceso basado en rol de Azure (RBAC de Azure)](../role-based-access-control/overview.md).

Cuando un rol de Azure se asigna a una entidad de seguridad de Azure AD, Azure concede a esa entidad de seguridad acceso a esos recursos. El acceso tiene el ámbito del recurso de App Configuration. Una entidad de seguridad de Azure AD puede ser un usuario, una entidad de servicio de aplicación o una [identidad de servicio administrada para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="azure-built-in-roles-for-azure-app-configuration"></a>Roles integrados de Azure para Azure App Configuration
Azure proporciona los siguientes roles integrados de Azure para autorizar el acceso a los datos de App Configuration con Azure AD y OAuth:

- **Propietario de los datos de App Configuration**: use este rol para proporcionar acceso de lectura, escritura o eliminación a los datos de App Configuration. Esto no concede acceso al recurso de App Configuration.
- **Lector de los datos de App Configuration**: use este rol para proporcionar acceso de lectura a los datos de App Configuration. Esto no concede acceso al recurso de App Configuration.
- **Colaborador**: use este rol para administrar el recurso de App Configuration. Aunque se puede acceder a los datos de App Configuration mediante las claves de acceso, este rol no concede acceso directo a los datos mediante Azure AD.
- **Lector**: use este rol para proporcionar acceso de lectura al recurso de App Configuration. Esto no concede acceso a las claves de acceso del recurso ni a los datos almacenados en App Configuration.

> [!NOTE]
> Actualmente, Azure Portal y la CLI solo admiten la autenticación HMAC para tener acceso a los datos de App Configuration. No se admite la autenticación de Azure AD. Por lo tanto, los usuarios de Azure Portal y de la CLI requieren el rol *Colaborador* para recuperar las claves de acceso del recurso de App Configuration. La concesión de los roles *Lector de datos de App Configuration* o *Propietario de los datos de App Configuration* no tiene ningún impacto en el acceso a través del portal y la CLI.

## <a name="next-steps"></a>Pasos siguientes
Más información sobre el uso de [identidades administradas](howto-integrate-azure-managed-service-identity.md) para administrar el servicio App Configuration.
