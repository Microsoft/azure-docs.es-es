---
title: Objetos de aplicación y de entidad de servicio de Azure Active Directory
description: Conozca la relación entre los objetos de aplicación y de entidad de servicio en Azure Active Directory.
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
services: active-directory
editor: ''
ms.assetid: adfc0569-dc91-48fe-92c3-b5b4833703de
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/13/2019
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: sureshja
ms.collection: M365-identity-device-management
ms.openlocfilehash: 47626cbd8d5b8a5ddf7cc5c6ea42fb9453e5bfb2
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540681"
---
# <a name="application-and-service-principal-objects-in-azure-active-directory"></a>Objetos de aplicación y de entidad de servicio de Azure Active Directory

En ocasiones, puede malinterpretarse el significado del término "aplicación" cuando se utiliza en el contexto de Azure Active Directory (Azure AD). En este artículo se aclaran aspectos concretos y conceptuales sobre la integración de aplicaciones de Azure AD con un ejemplo de registro y consentimiento para una [aplicación multiinquilino](developer-glossary.md#multi-tenant-application).

## <a name="overview"></a>Información general

Una aplicación que se ha integrado con Azure AD tiene implicaciones que van más allá de los aspectos de software. "Aplicación" se usa con frecuencia como un término conceptual que hace referencia no solo a la aplicación de software, sino también a su registro de Azure AD y rol en “conversaciones” de autenticación o autorización en tiempo de ejecución.

Por definición, una aplicación puede funcionar en estos roles:

- Rol de [cliente](developer-glossary.md#client-application) (que consume un recurso).
- Rol de [servidor de recursos](developer-glossary.md#resource-server) (que expone las API a los clientes).
- Rol de cliente y rol de servidor de recursos.

Un [flujo de concesión de autorizaciones de OAuth 2.0](developer-glossary.md#authorization-grant) define el protocolo de conversación, que permite que el cliente o recurso acceda o proteja los datos de un recurso, respectivamente.

En las siguientes secciones veremos cómo el modelo de aplicaciones de Azure AD representa una aplicación en tiempo de diseño y en tiempo de ejecución.

## <a name="application-registration"></a>Registro de aplicaciones

Al registrar una aplicación de Azure AD en [Azure Portal][AZURE-Portal], se crean dos objetos en el inquilino de Azure AD:

- Un objeto de aplicación.
- Un objeto de entidad de servicio.

### <a name="application-object"></a>Objeto de aplicación

Una aplicación de Azure AD se define por su único objeto de aplicación, que reside en el inquilino de Azure AD donde se registró la aplicación, conocido como inquilino "principal" de la aplicación. Microsoft Graph [entidad de aplicación] [ MS-Graph-App-Entity] define el esquema de propiedades de un objeto de aplicación.

### <a name="service-principal-object"></a>Objeto de entidad de servicio

Para acceder a los recursos que están protegidos por un inquilino de Azure AD, la entidad que requiere acceso debe estar representada por una entidad de seguridad. Esto es cierto para los usuarios (entidad de seguridad de usuario) y para las aplicaciones (entidad de servicio).

La entidad de seguridad define la directiva de acceso y los permisos para el usuario o aplicación de ese inquilino de Azure AD. Esto habilita características básicas como la autenticación del usuario o de la aplicación durante el inicio de sesión y la autorización durante el acceso a los recursos.

Cuando una aplicación tiene permiso para acceder a los recursos de un inquilino (tras el registro o [consentimiento](developer-glossary.md#consent)), se crea un objeto de entidad de seguridad de servicio. Microsoft Graph [entidad ServicePrincipal] [ MS-Graph-Sp-Entity] define el esquema de propiedades de un objeto entidad de servicio.

### <a name="application-and-service-principal-relationship"></a>Relación entre la aplicación y la entidad de servicio

Considere el objeto de aplicación como una representación *global* de la aplicación para su uso en todos los inquilinos y la entidad de servicio como una representación *local* para su uso en un inquilino específico.

El objeto de aplicación actúa como la plantilla a partir de la cual se *derivan* las propiedades comunes y predeterminadas para su uso en la creación de objetos de entidad de servicio correspondientes. Por tanto, un objeto de aplicación tiene una relación 1:1 con la aplicación de software y una relación 1:muchos con sus objetos de entidad de servicio correspondientes.

Debe crearse una entidad de servicio en cada inquilino donde se usa la aplicación. Esto permite establecer una identidad para el inicio de sesión o el acceso a los recursos que va a proteger el inquilino. Una aplicación de inquilino único tendrá solo una entidad de servicio (en su inquilino principal), que normalmente se crea y se consiente para su uso durante el registro de la aplicación. Una API o aplicación web multiinquilino también tiene una entidad de servicio creada en cada inquilino donde un usuario de ese inquilino ha dado su consentimiento para su uso.

> [!NOTE]
> Los cambios que realice en el objeto de aplicación también se reflejan en el objeto de entidad de servicio, solo en el inquilino principal de la aplicación (es decir, el inquilino en donde se registró). Para aplicaciones multiinquilino, los cambios realizados en el objeto de aplicación no se reflejan en los objetos de entidad de servicio del inquilino consumidor hasta que el acceso se quita a través del [panel de acceso de la aplicación](https://myapps.microsoft.com) y se concede de nuevo.
>
> Tenga en cuenta también que las aplicaciones nativas se registran como multiinquilino de forma predeterminada.

## <a name="example"></a>Ejemplo

En el siguiente se muestra la relación entre un objeto de aplicación de la aplicación y los objetos de entidad de servicio correspondientes, en el contexto de una aplicación multiinquilino de ejemplo denominada **HR app**. En este escenario de ejemplo hay tres inquilinos de Azure AD:

- **Adatum**: el inquilino usado por la empresa que desarrolló la **aplicación de recursos humanos**.
- **Contoso**: el inquilino usado por la organización Contoso, que es un consumidor de la **aplicación de recursos humanos**.
- **Fabrikam**: el inquilino usado por la organización Fabrikam, que también consume la **aplicación de recursos humanos**.

![Relación entre un objeto de aplicación y un objeto de entidad de servicio](./media/app-objects-and-service-principals/application-objects-relationship.svg)

En este escenario de ejemplo:

| Paso | DESCRIPCIÓN |
|------|-------------|
| 1    | Es el proceso de creación de los objetos de aplicación y de entidad de servicio del inquilino principal de la aplicación. |
| 2    | Cuando los administradores de Contoso y Fabrikam completan el consentimiento, se crea un objeto de entidad de servicio en el inquilino de Azure AD de la empresa y se asignan los permisos que concede el administrador. Tenga en cuenta también que la aplicación de recursos humanos podría estar configurada o diseñada para dar consentimiento a los usuarios para el uso individual. |
| 3    | Los inquilinos consumidores de la aplicación HR (Contoso y Fabrikam) tienen cada uno de ellos su propio objeto de entidad de servicio. Cada uno representa su uso de una instancia de la aplicación en tiempo de ejecución, que se rige por los permisos que consiente el administrador correspondiente. |

## <a name="next-steps"></a>Pasos siguientes

- Puede usar el [Microsoft Graph Explorer](https://developer.microsoft.com/graph/graph-explorer) para consultar la aplicación y objetos de entidad de servicio.
- Puede tener acceso a objetos de aplicación de la aplicación mediante Microsoft Graph API, el [del portal de Azure] [ AZURE-Portal] editor de manifiestos de aplicación, o [cmdlets de PowerShell de Azure AD](https://docs.microsoft.com/powershell/azure/overview?view=azureadps-2.0) , tal como está representada por su OData [entidad de aplicación][MS-Graph-App-Entity].
- Puede tener acceso a objeto de entidad de servicio de la aplicación a través de Microsoft Graph API o [cmdlets de PowerShell de Azure AD](https://docs.microsoft.com/powershell/azure/overview?view=azureadps-2.0), tal como está representada por su OData [entidad ServicePrincipal] [ MS-Graph-Sp-Entity].

<!--Image references-->

<!--Reference style links -->
[MS-Graph-App-Entity]: https://docs.microsoft.com/graph/api/resources/application
[MS-Graph-Sp-Entity]: https://docs.microsoft.com/graph/api/resources/serviceprincipal
[AZURE-Portal]: https://portal.azure.com
