---
title: Cómo y por qué se añaden aplicaciones a Azure Active Directory
description: ¿Por qué se añade una aplicación a Azure AD y cómo llega hasta ahí?
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 3321d130-f2a8-4e38-b35e-0959693f3576
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/18/2018
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: elisol, lenalepa
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8cc9d0a951ac6f7ed18ad6558ae9edb2d1f9c8f4
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544653"
---
# <a name="how-and-why-applications-are-added-to-azure-ad"></a>Cómo y por qué se agregan aplicaciones a Azure AD

En Azure AD existen dos conceptos de aplicación: 
* [Objetos de aplicación](app-objects-and-service-principals.md#application-object): aunque hay [excepciones](#notes-and-exceptions), los objetos de aplicación pueden considerarse como la definición de una aplicación.
* [Entidades de servicio](app-objects-and-service-principals.md#service-principal-object): pueden considerarse como una instancia de una aplicación. Las entidades de servicio suelen hacer referencia a un objeto de aplicación. Asimismo, varias entidades de servicio de diversos directorios pueden hacer referencia a un mismo objeto de aplicación.

## <a name="what-are-application-objects-and-where-do-they-come-from"></a>¿Qué son los objetos de aplicación y de dónde provienen?
Puede administrar los [objetos de aplicación](app-objects-and-service-principals.md#application-object) en Azure Portal a través de la experiencia de [Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ApplicationsListBlade). Los objetos de aplicación describen la aplicación para Azure AD y pueden considerarse como su definición, que permite al servicio saber cómo emitir tokens para la aplicación en función de su configuración. El objeto de aplicación solo está presente en su directorio principal, incluso si se trata de una aplicación multiinquilino que admite entidades de servicio en otros directorios. El objeto de aplicación puede incluir cualquiera de los siguientes datos (así como información adicional que no se indica aquí):
* Nombre, logotipo y editor
* URL de respuesta
* Secretos (claves simétricas o asimétricas utilizadas para autenticar la aplicación)
* Dependencias de API (OAuth)
* API/recursos/ámbitos publicados (OAuth)
* Roles de aplicación (RBAC)
* Configuración y metadatos SSO
* Configuración y metadatos de aprovisionamiento de usuarios
* Configuración y metadatos proxy

Los objetos de aplicación se pueden crear de varias maneras, incluidas las siguientes:
* Registros de aplicación en Azure Portal.
* Crear una nueva aplicación con Visual Studio y configurarla para que use la autenticación de Azure AD.
* Cuando un administrador añade una aplicación desde la galería de aplicaciones (esto también creará una entidad de servicio).
* Usar Microsoft Graph API, Graph API de Azure AD o PowerShell para crear una nueva aplicación.
* Muchas más, incluidas varias experiencias de desarrollador en Azure y experiencias del explorador de API en centros de desarrollador.

## <a name="what-are-service-principals-and-where-do-they-come-from"></a>¿Qué son las entidades de servicio y de dónde provienen?
Puede administrar las [entidades de servicio](app-objects-and-service-principals.md#service-principal-object) en Azure Portal a través de la experiencia de [Aplicaciones empresariales](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps/menuId/). Las entidades de servicio son las que realmente rigen una aplicación que se conecta a Azure AD y pueden considerarse la instancia de la aplicación en el directorio. Cualquier aplicación determinada puede tener como máximo un objeto de aplicación (registrado en un directorio principal) y uno o varios objetos de entidad de servicio que representan instancias de la aplicación en cada directorio en donde actúa. 

La entidad de servicio puede incluir:

* Una referencia a un objeto de aplicación a través de la propiedad ID de la aplicación.
* Registros de asignaciones de rol de aplicación de grupos y de usuarios locales.
* Registros de permisos de administrador y usuarios locales concedidos a la aplicación.
  * Por ejemplo: permiso de la aplicación para acceder al correo electrónico de un usuario determinado.
* Registros de directivas locales, incluida la directiva de acceso condicional.
* Registros de configuración local alternativa para una aplicación.
  * Reclama las reglas de transformación
  * Asignaciones de atributos (aprovisionamiento de usuarios)
  * Roles de aplicación específicos del directorio (si la aplicación admite roles personalizados).
  * Nombre o logotipo específico del directorio.

Al igual que los objetos de aplicación, las entidades de servicio se pueden crear de varias maneras, incluidas las siguientes:

* Cuando los usuarios inician sesión en una aplicación de terceros integrada con Azure AD.
  * Durante el inicio de sesión, se solicita a los usuarios que proporcionen permiso a la aplicación para acceder a su perfil y otros permisos. La primera persona que dé su consentimiento hará que una entidad de servicio que representa la aplicación se añada al directorio.
* Cuando los usuarios inician sesión en servicios en línea de Microsoft, como [Office 365](https://products.office.com/).
  * Al suscribirse a Office 365 o probar una versión de prueba, se crean una o más entidades de servicio en el directorio que representa los distintos servicios que se usan para ofrecer toda la funcionalidad asociada a Office 365.
  * Algunos servicios de Office 365 como SharePoint crean entidades de servicio de forma continua para permitir una comunicación segura entre los componentes que incluyen los flujos de trabajo.
* Cuando un administrador añade una aplicación desde la galería de aplicaciones (esto también creará un objeto de aplicación subyacente).
* Al añadir una aplicación para usar el [Azure Active Directory Application Proxy](https://msdn.microsoft.com/library/azure/dn768219.aspx).
* Al conectar una aplicación para inicio de sesión único utilizando SAML o inicio de sesión único (SSO) de contraseña.
* Mediante programación con Graph API de Azure AD o PowerShell

## <a name="how-are-application-objects-and-service-principals-related-to-each-other"></a>¿Qué relación tienen los objetos de aplicación y las entidades de servicio?
Una aplicación tiene un objeto de aplicación en su directorio principal al que una o varias entidades de servicio hacen referencia en cada uno de los directorios donde opera (incluido el directorio principal de la aplicación).
![Un diagrama que muestra cómo los objetos de aplicación y las entidades de servicio interactúan entre sí y con instancias de Azure AD.][apps_service_principals_directory]

En el diagrama anterior, Microsoft mantiene dos directorios internamente (a la izquierda) que usa para publicar aplicaciones:

* Uno para Microsoft Apps (directorio de servicios de Microsoft)
* Uno para aplicaciones preintegradas de terceros (directorio de galería de aplicaciones)

Se necesitan publicadores/proveedores de aplicaciones que se integren con Azure AD para disponer de un directorio de publicación (indicado a la derecha como "Some SaaS Directory" [Un directorio SaaS]).

Entre las aplicaciones que añade personalmente (representadas como **App (yours)** [Aplicación (suya)] en el diagrama) se incluyen:

* Aplicaciones que ha desarrollado (integradas con Azure AD)
* Aplicaciones conectadas para el inicio de sesión único
* Aplicaciones que ha publicado mediante el proxy de aplicación de Azure AD

### <a name="notes-and-exceptions"></a>Notas y excepciones
* No todas las entidades de servicio señalan a un objeto de aplicación. Cuando se diseñó originalmente Azure AD, los servicios proporcionados a las aplicaciones eran más limitados y la entidad de servicio era suficiente para establecer una identidad de aplicación. La entidad de seguridad de servicio original era más cercana en cuanto a la forma a la cuenta de servicio de Windows Server Active Directory. Por este motivo, aún es posible crear entidades de servicio de otras maneras, como con Azure AD PowerShell, sin crear primero un objeto de aplicación. Graph API de Azure AD requiere un objeto de aplicación antes de crear una entidad de servicio.
* Actualmente, no toda la información que se ha descrito anteriormente se expone mediante programación. Lo siguiente solo está disponible en la interfaz de usuario:
  * Reclama las reglas de transformación
  * Asignaciones de atributos (aprovisionamiento de usuarios)
* Para más información sobre los objetos de aplicación y la entidad de servicio, consulte la documentación de referencia de Graph API de Azure AD.
  * [Aplicación](https://msdn.microsoft.com/library/azure/ad/graph/api/entity-and-complex-type-reference#application-entity)
  * [Entidad de seguridad de servicio](https://msdn.microsoft.com/library/azure/ad/graph/api/entity-and-complex-type-reference#serviceprincipal-entity)

## <a name="why-do-applications-integrate-with-azure-ad"></a>¿Por qué se integran las aplicaciones con Azure AD?
Las aplicaciones se añaden a Azure AD para aprovechar uno o varios de los servicios que proporciona, como los siguientes:

* Autenticación y autorización de aplicaciones.
* Autenticación y autorización de usuarios.
* Inicio de sesión único (SSO) mediante federación o contraseña.
* Aprovisionamiento y sincronización de usuarios.
* Control de acceso basado en roles. Utilice el directorio para definir los roles de aplicación para realizar comprobaciones de autorización basadas en roles en una aplicación.
* Servicios de autorización OAuth. Usados por Office 365 y otras aplicaciones Microsoft para autorizar el acceso a los recursos y las API.
* Publicación de aplicaciones y proxy. Publique una aplicación desde una red privada en Internet.

## <a name="who-has-permission-to-add-applications-to-my-azure-ad-instance"></a>¿Quién tiene permiso para agregar aplicaciones a la instancia de Azure AD?
Aunque hay algunas tareas que solo los administradores globales pueden realizar (por ejemplo, añadir aplicaciones desde la galería de aplicaciones y configurar una aplicación para usar el proxy de aplicación), de forma predeterminada todos los usuarios del directorio tienen permisos para registrar objetos de aplicación que estén desarrollando y para decidir qué aplicaciones comparten o a cuáles permitirán acceder a sus datos de la organización mediante autorización. Si una persona es el primer usuario del directorio en iniciar sesión en una aplicación y conceder autorización, creará una entidad de servicio en el inquilino; en caso contrario, la información de concesión de autorización se almacenará en la entidad de servicio existente.

Es posible que la idea de permitir a los usuarios registrar y autorizar aplicaciones pueda parecer inquietante en un principio, pero debe tenerse en cuenta lo siguiente:


* Las aplicaciones han podido aprovechar Windows Server Active Directory para la autenticación de usuarios durante muchos años sin necesidad de que la aplicación se registre en el directorio. Ahora la organización contará con una visibilidad mejorada para conocer exactamente cuántas aplicaciones están usando el directorio y para qué.
* Delegar estas responsabilidades en los usuarios elimina la necesidad de un registro de la aplicación y un proceso de publicación controlados por el administrador. Con Servicios de federación de Active Directory (AD FS) es probable que un administrador tuviera que añadir una aplicación como usuario de confianza en nombre de los desarrolladores. Ahora los desarrolladores pueden hacer uso del autoservicio.
* El inicio de sesión de usuarios en aplicaciones con cuentas de organización para fines empresariales es una buena opción. Si posteriormente dejan la organización, perderán automáticamente el acceso a su cuenta en la aplicación que utilizaban.
* Disponer de un registro sobre qué datos se compartieron con qué aplicación es algo positivo. Los datos son más transportables que nunca y tener un registro claro de quién compartió qué datos con qué aplicaciones resulta útil.
* Los propietarios de API que usan Azure AD para OAuth deciden exactamente qué permisos pueden conceder los usuarios a las aplicaciones y qué permisos requieren la aceptación de un administrador. Solo los administradores pueden dar su consentimiento a ámbitos más amplios y permisos más significativos, mientras que el consentimiento de los usuarios se limita a los propios datos y funcionalidades de los usuarios.
* Cuando un usuario añade una aplicación o permite que esta acceda a sus datos, el evento puede auditarse, con lo que podrá ver los informes de auditoría en Azure Portal para determinar cómo se añadió una aplicación al directorio.

Si aún desea impedir que los usuarios del directorio registren aplicaciones e inicien sesión en aplicaciones sin la aprobación del administrador, hay dos opciones de configuración que puede cambiar para desactivar estas funcionalidades:

* Para impedir que los usuarios autoricen aplicaciones en su propio nombre:
  1. En Azure Portal, vaya a la sección [Configuración de usuario](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/UserSettings/menuId/) en Aplicaciones empresariales.
  2. Cambie **Los usuarios pueden permitir que las aplicaciones accedan a los datos de la compañía en su nombre** a **No**. 
     
     > [!NOTE]
     > Si decide desactivar el consentimiento del usuario, se necesitará que un administrador dé su consentimiento a cualquier nueva aplicación que un usuario necesite usar.    
* Para impedir que los usuarios registren sus propias aplicaciones:
  1. En Azure Portal, vaya a la sección [Configuración de usuario](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/UserSettings) en Azure Active Directory.
  2. Cambie **Los usuarios pueden registrar aplicaciones** a **No**.

> [!NOTE]
> Incluso Microsoft usa la configuración predeterminada con usuarios que pueden registrar aplicaciones y dar su consentimiento a aplicaciones en su propio nombre.

<!--Image references-->
[apps_service_principals_directory]:../media/active-directory-how-applications-are-added/HowAppsAreAddedToAAD.jpg

