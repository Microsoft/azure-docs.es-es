---
title: Graph API de Azure Active Directory
description: Una guía de información general e inicio rápido para Graph API de Azure AD que permite el acceso mediante programación a Azure AD a través de los puntos de conexión de la API REST.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/26/2019
ms.author: ryanwi
ms.reviewer: dkershaw, sureshja
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: da99468b1582c4acab192ad3b96761172aa69580
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89068667"
---
# <a name="azure-active-directory-graph-api"></a>Graph API de Azure Active Directory

> [!IMPORTANT]
> Le recomendamos encarecidamente que utilice [Microsoft Graph](https://developer.microsoft.com/graph) en lugar de Graph API de Azure AD para tener acceso a recursos de Azure Active Directory (Azure AD). Nuestros esfuerzos de desarrollo ahora se centran en Microsoft Graph y no están previstas realizar mejoras adicionales para Graph API de Azure AD. Hay un número muy limitado de escenarios para los que Graph API de Azure AD todavía podría ser adecuado. Para obtener más información, consulte la entrada de blog [Microsoft Graph o Graph de Azure AD](https://developer.microsoft.com/office/blogs/microsoft-graph-or-azure-ad-graph/) y [Migrar aplicaciones de Graph de Azure AD a Microsoft Graph](/graph/migrate-azure-ad-graph-planning-checklist).

Este artículo se aplica a Azure AD Graph API. Para obtener información similar relacionada con Microsoft Graph API, consulte [Usar la API de Microsoft Graph](/graph/use-the-api).

Graph API de Azure Active Directory proporciona acceso mediante programación a Azure AD a través de los extremos de la API de REST. Las aplicaciones pueden usar Graph API de Azure AD para ejecutar operaciones de creación, lectura, actualización y eliminación (CRUD) en objetos y datos de directorio. Por ejemplo, Graph API de Azure AD admite las siguientes operaciones comunes en un objeto de usuario:

* Crear un usuario nuevo en un directorio
* Obtener las propiedades detalladas de un usuario, como sus grupos
* Actualizar las propiedades de un usuario, como su ubicación y número de teléfono o cambiar su contraseña
* Consultar la pertenencia a grupos de un usuario para el acceso basado en roles
* Deshabilitar la cuenta de un usuario o eliminarla por completo

Además, puede realizar las mismas operaciones en otros objetos, como grupos y aplicaciones. Para llamar a Graph API de Azure AD en un directorio, la aplicación debe registrarse con Azure AD. La aplicación también se debe conceder acceso a Graph API de Azure AD. Este acceso se logra normalmente mediante un flujo de consentimiento de usuario o administrador.

Para empezar a usar Graph API de Azure Active Directory, consulte la [Guía de inicio rápido de Graph API de Azure AD](./microsoft-graph-intro.md) o la [documentación interactiva de referencia sobre Graph API de Azure AD](/previous-versions/azure/ad/graph/api/api-catalog).

## <a name="features"></a>Características

Graph API de Azure AD ofrece las siguientes características:

* **Puntos de conexión de API REST**: Azure AD Graph API es un servicio RESTful que se compone de puntos de conexión a los que se accede con solicitudes HTTP estándar. Graph API de Azure AD admite tipos de contenido XML o notación de objetos JavaScript (JSON) en solicitudes y respuestas. Para obtener más información, consulte [Referencia de la API Graph de REST de Azure AD](/previous-versions/azure/ad/graph/api/api-catalog).
* **Autenticación con Azure AD**: Todas las solicitudes enviadas a Azure AD Graph API se tienen que autenticar anexando un token web JSON (JWT) al encabezado Authorization de la solicitud. Para conseguir este token, es necesario enviar una solicitud al punto de conexión de token de Azure AD y facilitar unas credenciales válidas. Puede usar el flujo de credenciales de cliente OAuth 2.0 o el flujo de concesión de código de autorización para conseguir un token con el que llamar a Graph. Para obtener más información, vea [OAuth 2.0 en Azure AD](/previous-versions/azure/dn645545(v=azure.100)).
* **Autorización basada en roles (RBAC)** : En Azure AD Graph API se usan grupos de seguridad para llevar a cabo la RBAC. Por ejemplo, si quiere saber si un usuario tiene acceso a un recurso determinado, la aplicación puede llamar a la operación [Comprobar pertenencia a grupos (transitiva)](/previous-versions/azure/ad/graph/api/functions-and-actions#checkMemberGroups) , que devuelve true o false.
* **Consulta diferencial**: Una consulta diferencial le permite realizar el seguimiento de los cambios hechos en un directorio entre dos períodos de tiempo, sin tener que hacer consultas frecuentes a Azure AD Graph API. Este tipo de solicitud solo devolverá los cambios hechos entre la solicitud de consulta diferencial anterior y la solicitud actual. Para más información, vea [Consulta diferencial de Graph API de Azure AD](/previous-versions/azure/ad/graph/howto/azure-ad-graph-api-differential-query).
* **Extensiones de directorio**: Puede agregar propiedades personalizadas a los objetos de directorio sin necesidad de un almacén de datos externo. Por ejemplo, si su aplicación necesita una propiedad de usuario de Skype para cada usuario, puede registrar la nueva propiedad en el directorio para que esté disponible para su uso en todos los objetos de usuario. Para obtener más información, consulte [Extensiones de esquema de directorio de Graph API de Azure AD](/previous-versions/azure/ad/graph/howto/azure-ad-graph-api-directory-schema-extensions).
* **Protegido por ámbitos de permiso**: Azure AD Graph API expone los ámbitos de permiso que permiten el acceso seguro a los datos de Azure AD mediante OAuth 2.0. Admite una variedad de tipos de aplicación cliente, entre ellos:
  
  * Interfaces de usuario que reciben acceso delegado a los datos a través de una autorización del usuario con sesión iniciada (delegado).
  * Aplicaciones de servicio o demonio que funcionan en segundo plano sin un usuario que haya iniciado sesión presente y utilicen el control de acceso basado en rol definidos por la aplicación.
    
    Tanto los permisos delegados y de aplicación representan un privilegio que se expone mediante Graph API de Azure AD, y las aplicaciones cliente pueden solicitarlos a través de las características de permisos de registro de aplicación en [Azure Portal](https://portal.azure.com). Los [ámbitos de permiso de Graph API de Azure AD](/previous-versions/azure/ad/graph/howto/azure-ad-graph-api-permission-scopes) proporcionan información sobre lo que está disponible para su uso por la aplicación cliente.

## <a name="scenarios"></a>Escenarios

Graph API de Azure AD admite muchos escenarios de aplicación. Los siguientes escenarios son los más comunes:

* **Aplicación de línea de negocio (un solo inquilino)** : En este escenario, un desarrollador empresarial trabaja para una organización que tiene una suscripción a Office 365. El desarrollador crea una aplicación web que interactúa con Azure AD para realizar tareas tales como la asignación de una licencia a un usuario. Esta tarea requiere acceso a Graph API de Azure AD, por eso el desarrollador registra la aplicación de inquilino único en Azure AD y configura los permisos de lectura y escritura para Graph API de Azure AD. A continuación, la aplicación se configura para que use sus propias credenciales o las del usuario que tenga iniciada la sesión para conseguir un token con el que llamar a Graph API de Azure AD.
* **Aplicación de Software como servicio (multiinquilino)** : En este escenario, un fabricante de software independiente (ISV) desarrolla una aplicación web multiinquilino hospedada que ofrece características de administración de usuarios para otras organizaciones que usan Azure AD. Estas características requieren acceso a objetos de directorio, por lo que la aplicación necesita llamar a Graph API de Azure AD. El desarrollador registra la aplicación en Azure AD, la configura para requerir permisos de lectura y escritura para Graph API de Azure AD, y habilita el acceso externo para que las demás organizaciones puedan permitir usar la aplicación en su directorio. Cuando se autentica un usuario de otra organización en la aplicación por primera vez, se le muestra un cuadro de diálogo de consentimiento con los permisos que solicita la aplicación. La concesión de consentimiento proporciona a la aplicación los permisos solicitados para Graph API de Azure AD en el directorio del usuario. Para obtener más información acerca del marco de consentimiento, consulte [Información general sobre el marco de consentimiento](consent-framework.md).

## <a name="next-steps"></a>Pasos siguientes

Para empezar a usar Azure Active Directory Graph API, consulte los temas siguientes:

* [Inicio rápido para Graph API de Azure AD](./microsoft-graph-intro.md)
* [Documentación de Graph de REST de Azure AD](/previous-versions/azure/ad/graph/api/api-catalog)
