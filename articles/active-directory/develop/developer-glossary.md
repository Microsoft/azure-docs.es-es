---
title: Guía del desarrollador de Microsoft identity platform | Azure
description: Una lista de términos frecuente de las características y conceptos para desarrolladores de Microsoft identity platform.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 551512df-46fb-4219-a14b-9c9fc23998ba
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/13/2019
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: jmprieur, saeeda, jesakowi, nacanuma, dadobali
ms.collection: M365-identity-device-management
ms.openlocfilehash: 89ea1dba09173b20d11a5022e6666e6c865ead62
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540097"
---
# <a name="microsoft-identity-platform-developer-glossary"></a>Guía del desarrollador de plataforma de identidad de Microsoft

En este artículo contiene las definiciones de algunos de los conceptos clave para desarrolladores y la terminología, que son útiles para obtener información sobre el desarrollo de aplicaciones con la plataforma Microsoft identity.

## <a name="access-token"></a>token de acceso

Un tipo de [token de seguridad](#security-token) emitido por un [servidor de autorización](#authorization-server) y usado por una [aplicación cliente](#client-application) para acceder a un [servidor de recursos protegidos](#resource-server). Normalmente en forma de [JSON Web Token (JWT)][JWT], el token personifica la autorización concedida al cliente por el [propietario del recurso](#resource-owner) para un nivel de acceso solicitado. El token contiene todas las [notificaciones](#claim) aplicables sobre el sujeto, lo que permite a la aplicación cliente utilizarlo como forma de credencial al acceder a un recurso determinado. Esto también elimina la necesidad de que el propietario del recurso exponga sus credenciales al cliente.

A veces se conoce a los tokens de acceso como "Aplicación y usuario" o "Solo aplicación", según las credenciales que se va a representar. Por ejemplo, cuando una aplicación cliente usa:

* La [concesión de autorización del "código de autorización"](#authorization-grant), el usuario final se autentica primero como propietario del recurso, delegando la autorización al cliente para acceder al recurso. Después, el cliente se autentica al obtener el token de acceso. El token a veces se conoce más específicamente como token "Aplicación y usuario", ya que representa tanto al usuario que ha autorizado la aplicación cliente como a la aplicación.
* [La concesión de autorización de "Credenciales de cliente"](#authorization-grant), el cliente proporciona la autenticación única, que funciona sin la autorización o autenticación del propietario del recurso, por lo que a veces el token puede conocerse como un token "Solo aplicación".

Consulte [plataforma Microsoft identity: referencia de tokens] [ AAD-Tokens-Claims] para obtener más detalles.

## <a name="application-id-client-id"></a>Id. de aplicación (Id. de cliente)

El identificador único que Azure AD emite para un registro de la aplicación que identifica una aplicación específica y las configuraciones asociadas. Este identificador de aplicación ([Id. de cliente](https://tools.ietf.org/html/rfc6749#page-15)) se usa al realizar la autenticación de solicitudes y es proporciona a las bibliotecas de autenticación en tiempo de desarrollo. El identificador de aplicación (Id. de cliente) no es un secreto.

## <a name="application-manifest"></a>manifiesto de aplicación

Una característica proporcionada por [Azure Portal][AZURE-portal], que genera una representación JSON de la configuración de identidad de la aplicación, utilizada como un mecanismo para actualizar sus entidades [Application][AAD-Graph-App-Entity] y [ServicePrincipal][AAD-Graph-Sp-Entity] asociadas. Para más información, consulte [Descripción del manifiesto de aplicación de Azure Active Directory][AAD-App-Manifest].

## <a name="application-object"></a>objeto de aplicación

Cuando se registra o se actualiza una aplicación en [Azure Portal][AZURE-portal], el portal crea o actualiza un objeto de aplicación y un [objeto de entidad de servicio](#service-principal-object) correspondiente para dicho inquilino. El objeto de aplicación *define* globalmente la configuración de identidad de la aplicación (en todos los inquilinos a los que tiene acceso), lo que proporciona una plantilla de la que *proceden* los objetos de entidad de servicio correspondientes para su uso local en tiempo de ejecución (en un inquilino específico).

Para más información, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory (Azure AD)][AAD-App-SP-Objects].

## <a name="application-registration"></a>registro de la aplicación

Para permitir que una aplicación integre y delegue las funciones de administración de identidades y acceso a Azure AD, debe registrarse con el [inquilino](#tenant)de Azure AD. Al registrar la aplicación con Azure AD, se proporciona una configuración de identidad para la aplicación, lo que le permite integrarla con Azure AD y usar características tales como:

* Administración sólida de inicio de sesión único mediante la administración de identidades de Azure AD y la implementación del protocolo [OpenID Connect][OpenIDConnect]
* Acceso con intermediación a [recursos protegidos](#resource-server) por [las aplicaciones cliente](#client-application), a través de OAuth 2.0 [servidor de autorización](#authorization-server)
* [Marco de consentimiento](#consent) para administrar el acceso del cliente a los recursos protegidos, en función de la autorización del propietario del recurso.

Para más información, consulte [Integración de aplicaciones con Azure Active Directory][AAD-Integrating-Apps].

## <a name="authentication"></a>autenticación

El acto de solicitar a un usuario credenciales legítimas, que proporciona la base para la creación de una entidad de seguridad que se utilizará para el control de identidades y de acceso. Por ejemplo, durante una [concesión de autorización de OAuth2](#authorization-grant), el usuario que se autentica está cumpliendo el rol de [propietario del recurso](#resource-owner) o de [aplicación cliente](#client-application), en función de la concesión usada.

## <a name="authorization"></a>authorization

El acto de conceder un permiso a la entidad de seguridad autenticada para hacer algo. Hay dos casos de uso principales en el modelo de programación de Azure AD:

* Durante un flujo de [concesión de autorización de OAuth2](#authorization-grant): cuando el [propietario del recurso](#resource-owner) autoriza a la [aplicación cliente](#client-application), lo que permite al cliente acceder a los recursos del propietario del recurso.
* Durante el acceso a los recursos por el cliente: cuando lo implementa el [servidor de recursos](#resource-server), el uso de los valores de [notificación](#claim) presentes en el [token de acceso](#access-token) para tomar decisiones de control de acceso basadas en ellos.

## <a name="authorization-code"></a>código de autorización

Un "token" de corta duración proporcionado a una [aplicación cliente](#client-application) por el [punto de conexión de autorización](#authorization-endpoint), como parte del flujo de "código de autorización", una de las cuatro [concesiones de autorización](#authorization-grant) de OAuth2. El código se devuelve a la aplicación cliente como respuesta a la autenticación de un [propietario del recurso](#resource-owner), lo que indica que este propietario ha delegado la autorización para acceder a los recursos solicitados. Como parte del flujo, el código se canjea después por un [token de acceso](#access-token).

## <a name="authorization-endpoint"></a>punto de conexión de autorización

Uno de los puntos de conexión implementados por el [servidor de autorización](#authorization-server), que se usa para interactuar con el [propietario del recurso](#resource-owner) con el fin de proporcionar una [concesión de autorización](#authorization-grant) durante un flujo de concesión de autorización de OAuth2. Según el flujo de concesión de autorización que se utilice, puede variar la concesión real proporcionada, incluido un [código de autorización](#authorization-code) o un [token de seguridad](#security-token).

Consulte las secciones sobre los [tipos de concesión de autorización][OAuth2-AuthZ-Grant-Types] de la especificación OAuth2 y el [punto de conexión de autorización][OAuth2-AuthZ-Endpoint] y la [especificación de OpenIDConnect][OpenIDConnect-AuthZ-Endpoint] para más información.

## <a name="authorization-grant"></a>concesión de autorización

Una credencial que representa la [autorización](#authorization) del [propietario del recurso](#resource-owner) para acceder a sus recursos protegidos, concedida a una [aplicación cliente](#client-application). Una aplicación cliente puede utilizar uno de los [cuatro tipos de concesión definidos por la plataforma de autorización de OAuth2][OAuth2-AuthZ-Grant-Types] para obtener una concesión, según los requisitos o tipo de cliente: "concesión de código de autorización", "concesión de credenciales de cliente", "concesión implícita" y "concesión de credenciales de contraseña del propietario de recursos". La credencial que se devuelve al cliente es un [token de acceso](#access-token) o un [código de autorización](#authorization-code) (que se intercambia después por un token de acceso), según el tipo de concesión de autorización usado.

## <a name="authorization-server"></a>servidor de autorización

Tal como se define en la [plataforma de autorización de OAuth2][OAuth2-Role-Def], el servidor responsable de emitir los tokens de acceso al [cliente](#client-application) después de autenticar correctamente al [propietario del recurso](#resource-owner) y obtener su autorización. Un [aplicación cliente](#client-application) interactúa con el servidor de autorización en tiempo de ejecución por medio de sus puntos de conexión de [autorización](#authorization-endpoint) y [token](#token-endpoint), con arreglo a las [concesiones de autorización](#authorization-grant) definidas en OAuth2.

En el caso de integración de aplicaciones de plataforma de identidad de Microsoft, la plataforma de identidad de Microsoft implementa el rol de servidor de autorización para aplicaciones de Azure AD y el servicio de Microsoft de las API, por ejemplo [Microsoft Graph API] [Microsoft-Graph].

## <a name="claim"></a>notificación

Un [token de seguridad](#security-token) contiene notificaciones, que proporcionan aserciones acerca de una entidad (como una [aplicación cliente](#client-application) o un [propietario del recurso](#resource-owner)) a otra entidad (como el [servidor de recursos](#resource-server)). Las notificaciones son pares de nombre/valor que retransmiten datos sobre el asunto del token (por ejemplo, la entidad de seguridad que autenticó el [servidor de autorización](#authorization-server)). Las notificaciones presentes en cualquier token dependen de varias variables, como el tipo de token, el tipo de credencial que se usa para autenticar al usuario y la configuración de la aplicación, entre otras.

Consulte [referencia de token de plataforma de identidad de Microsoft] [ AAD-Tokens-Claims] para obtener más detalles.

## <a name="client-application"></a>aplicación cliente

Tal como se define en la [plataforma de autorización de OAuth2][OAuth2-Role-Def], una aplicación que realiza solicitudes de recursos protegidos en nombre del [propietario del recurso](#resource-owner). El término "cliente" no implica ninguna característica de implementación de hardware determinada (por ejemplo, si la aplicación se ejecuta en un servidor, un equipo de escritorio o en otros dispositivos).

Una aplicación cliente solicita [autorización](#authorization) a un propietario del recurso para participar en un flujo de [concesión de autorización de OAuth2](#authorization-grant) y puede acceder a los datos y las API en nombre del propietario del recurso. La plataforma de autorización de OAuth2 [define dos tipos de clientes][OAuth2-Client-Types], "confidencial" y "público", en función de la capacidad del cliente para mantener la confidencialidad de sus credenciales. Las aplicaciones pueden implementar un [cliente web (confidencial)](#web-client) que se ejecuta en un servidor web, un [cliente nativo (público)](#native-client) instalado en un dispositivo y un [cliente basado en agente usuario (público)](#user-agent-based-client) que se ejecuta en el explorador de un dispositivo.

## <a name="consent"></a>consentimiento

El proceso de un [propietario de recursos](#resource-owner) que concede autorización a una [aplicación cliente](#client-application), para acceder a recursos protegidos bajo [permisos](#permissions) específicos, en nombre del propietario del recurso. Según los permisos solicitados por el cliente, se le solicitará al administrador o usuario su consentimiento para permitir el acceso a los datos de la organización o individuales, respectivamente. Tenga en cuenta que en un escenario [multiinquilino](#multi-tenant-application), la [entidad de servicio](#service-principal-object) de la aplicación también se registra en el inquilino del usuario que concede el consentimiento.

Consulte el [marco de consentimiento](consent-framework.md) para más información.

## <a name="id-token"></a>token de identificador

Un [token de seguridad](#security-token) de [OpenID Connect][OpenIDConnect-ID-Token] proporcionado por un [punto de conexión de autorización](#authorization-endpoint) del [servidor de autorización](#authorization-server), que contiene las [notificaciones](#claim) que pertenecen a la autenticación de un [propietario de recursos](#resource-owner) de usuario final. Al igual que un token de acceso, los tokens de identificador también se representan como [JSON Web Token (JWT)][JWT] firmados digitalmente. Sin embargo, a diferencia de un token de acceso, las notificaciones de token de identificador no se usan para fines relacionados con el acceso a los recursos y específicamente con el control de acceso.

Consulte [referencia de token de plataforma de identidad de Microsoft] [ AAD-Tokens-Claims] para obtener más detalles.

## <a name="microsoft-identity-platform"></a>Plataforma de identidad de Microsoft

La plataforma de identidad de Microsoft es una evolución de la plataforma de desarrolladores y de servicio de identidad de Azure Active Directory (Azure AD). Permite a los desarrolladores crear aplicaciones que inicien sesión en todas las identidades de Microsoft, obtener tokens para llamar a Microsoft Graph, otras API de Microsoft o API que los desarrolladores hayan creado. Es una plataforma completa que consiste en un servicio de autenticación, bibliotecas, registro y configuración de aplicaciones, documentación completa para desarrolladores, ejemplos de código y otros contenidos para desarrolladores. La plataforma de identidad de Microsoft admite los protocolos estándar del sector como OAuth 2.0 y OpenID Connect. Consulte [Acerca de la plataforma de identidad de Microsoft](about-microsoft-identity-platform.md) para más información.

## <a name="multi-tenant-application"></a>aplicación multiinquilino

Una clase de aplicación que permite iniciar sesión y [consentir](#consent) a usuarios aprovisionados en un [inquilino](#tenant) de Azure AD, incluidos inquilinos que no son aquel con el que el cliente está registrado. De forma predeterminada, las aplicaciones [cliente nativas](#native-client) son aplicaciones multiinquilino, mientras que las aplicaciones [cliente web](#web-client) y [recurso web/API](#resource-server) tienen la capacidad de seleccionar entre uno y varios inquilinos. Por el contrario, una aplicación web registrada como único inquilino solo permite inicios de sesión desde cuentas de usuario aprovisionadas en el mismo inquilino que donde se registra la aplicación.

Consulte [Inicio de sesión de cualquier usuario de Azure Active Directory (AD) mediante el patrón de aplicación multiinquilino][AAD-Multi-Tenant-Overview] para más información.

## <a name="native-client"></a>cliente nativo

Un tipo de [aplicación cliente](#client-application) que se instala de forma nativa en un dispositivo. Como todo el código se ejecuta en un dispositivo, se considera un cliente "público" debido a su incapacidad para almacenar credenciales de manera privada o confidencial. Consulte los [tipos de cliente y perfiles de OAuth2][OAuth2-Client-Types] para más información.

## <a name="permissions"></a>permisos

Una [aplicación cliente](#client-application) obtiene acceso a un [servidor de recursos](#resource-server) mediante la declaración de las solicitudes de permiso. Existen dos tipos:

* Permisos "delegados", que especifican un acceso [basado en el ámbito](#scopes) con autorización delegada del [propietario del recurso](#resource-owner) que ha iniciado sesión y se presentan al recurso en tiempo de ejecución como [notificaciones "scp"](#claim) en el [token de acceso](#access-token) del cliente.
* Permisos de "aplicación", que especifican acceso [basado en rol](#roles) bajo las credenciales o la identidad de la aplicación cliente y se presentan al recurso en tiempo de ejecución como [notificaciones de "roles"](#claim) en el token de acceso del cliente.

También se revelan durante el proceso de [consentimiento](#consent) , ya que proporciona al administrador o al propietario del recurso la oportunidad de conceder o denegar el acceso de cliente a los recursos en su inquilino.

Las solicitudes de permisos se configuran en la pestaña "Aplicaciones"/"Configuración" de [Azure Portal][AZURE-portal], bajo "Permisos necesarios", al seleccionar los "Permisos delegados" y "Permisos de la aplicación" deseados (el último requiere la pertenencia al rol de administrador global). Dado que un [cliente público](#client-application) no puede mantener credenciales con seguridad, solo puede solicitar permisos delegados, mientras que un [cliente confidencial](#client-application) tiene la capacidad de solicitar permisos tanto delegados como de aplicación. El [objeto de aplicación](#application-object) de cliente almacena los permisos declarados en su [propiedad requiredResourceAccess][AAD-Graph-App-Entity].

## <a name="resource-owner"></a>propietario del recurso

De acuerdo con la [plataforma de autorización de OAuth2][OAuth2-Role-Def], una entidad capaz de conceder acceso a un recurso protegido. Cuando el propietario del recurso es una persona, se conoce como usuario final. Por ejemplo, cuando una [aplicación cliente](#client-application) desea acceder al buzón del usuario a través de [Microsoft Graph API][Microsoft-Graph], requiere el permiso del propietario de los recursos del buzón de correo.

## <a name="resource-server"></a>servidor de recursos

Tal como se ha definido por la [plataforma de autorización de OAuth2][OAuth2-Role-Def], un servidor que hospeda recursos protegidos, capaz de aceptar y responder a las solicitudes de recursos protegidos de las [aplicaciones cliente](#client-application) que presentan un [token de acceso](#access-token). También se conoce como servidor de recursos protegidos, o aplicación de recursos.

Un servidor de recursos expone las API y exige el acceso a sus recursos protegidos mediante [ámbitos](#scopes) y [roles](#roles), con el marco de autorización de OAuth 2.0. Por ejemplo, Graph API de Azure AD que proporciona acceso a los datos del inquilino de Azure AD y las API de Office 365 que proporcionan acceso a datos, como el correo electrónico y el calendario. Ambos también son accesibles a través de [Microsoft Graph API][Microsoft-Graph].

Al igual que una aplicación cliente, se establece la configuración de la identidad de la aplicación de recursos a través de [Registro](#application-registration) en un inquilino de Azure AD, que proporciona tanto el objeto de aplicación y como el de entidad de servicio. Algunas API proporcionadas por Microsoft, como Graph API de Azure AD, han registrado previamente entidades de servicio, que están disponibles en todos los inquilinos durante el aprovisionamiento.

## <a name="roles"></a>roles

Como los [ámbitos](#scopes), los roles proporcionan una forma para que un [servidor de recursos](#resource-server) controle el acceso a los recursos protegidos. Hay dos tipos: un rol de "usuario" implementa el control de acceso basado en roles para usuarios o grupos que requieren acceso al recurso, mientras que un rol de "aplicación" implementa lo mismo para las [aplicaciones cliente](#client-application) que requieren acceso.

Los roles son cadenas definidas por recursos (por ejemplo, "Aprobador de gastos", "Solo lectura", "Directory.ReadWrite.All"), administradas en [Azure Portal][AZURE-portal] mediante el [manifiesto de aplicación](#application-manifest) del recurso, y almacenadas en la [propiedad appRoles][AAD-Graph-Sp-Entity] del recurso. Azure Portal también se usa para asignar usuarios a roles de "usuario" y configurar los [permisos de la aplicación](#permissions) del cliente para acceder a un rol de "aplicación".

Para ver una explicación detallada de los roles de aplicación expuestos por Graph API de Azure AD, consulte los [ámbitos de permisos de Graph API][AAD-Graph-Perm-Scopes]. Para obtener un ejemplo de implementación paso a paso, consulte [Administración del acceso mediante RBAC y Azure Portal][AAD-RBAC].

## <a name="scopes"></a>ámbitos

Como los [roles](#roles), los ámbitos proporcionan una forma para que un [servidor de recursos](#resource-server) controle el acceso a los recursos protegidos. Los ámbitos se utilizan para implementar el control de acceso [basado en ámbitos][OAuth2-Access-Token-Scopes] para una [aplicación cliente](#client-application) a la que el propietario haya otorgado acceso delegado al recurso.

Los ámbitos son cadenas definidas por recursos (por ejemplo, "Mail.Read" o "Directory.ReadWrite.All"), administradas en [Azure Portal][AZURE-portal] mediante el [manifiesto de aplicación](#application-manifest) del recurso, y almacenadas en la [propiedad oauth2Permissions][AAD-Graph-Sp-Entity] del recurso. Azure Portal también se utiliza para configurar los [permisos delegados](#permissions) de la aplicación cliente para acceder a un ámbito.

Como procedimiento recomendado para la convención de nomenclatura, utilice un formato "resource.operation.constraint". Para ver una explicación detallada de los ámbitos expuestos por Graph API de Azure AD, consulte los [ámbitos de permisos de Graph API][AAD-Graph-Perm-Scopes]. Para los ámbitos expuestos por los servicios de Office 365, consulte [Office 365 API permissions reference][O365-Perm-Ref] (Referencia a los permisos de la API de Office 365).

## <a name="security-token"></a>token de seguridad

Un documento firmado con notificaciones, como un token OAuth2 o una aserción SAML 2.0. En el caso de una [concesión de autorización](#authorization-grant) de OAuth2, un [token de acceso](#access-token) (OAuth2) y un [token de identificador](https://openid.net/specs/openid-connect-core-1_0.html#IDToken) son tipos de token de seguridad, que se implementan como un [JSON Web Token (JWT)][JWT].

## <a name="service-principal-object"></a>objeto de entidad de servicio

Cuando se registra o se actualiza una aplicación en [Azure Portal][AZURE-portal], el portal crea o actualiza un [objeto de aplicación](#application-object) y un objeto de entidad de servicio correspondiente para dicho inquilino. El objeto de aplicación *define* globalmente la configuración de identidad de la aplicación (en todos los inquilinos a los que se le ha concedido acceso a la aplicación asociada) y es la plantilla de la que *proceden* los objetos de entidad de servicio correspondientes para su uso local en tiempo de ejecución (en un inquilino específico).

Para más información, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory (Azure AD)][AAD-App-SP-Objects].

## <a name="sign-in"></a>inicio de sesión

Proceso de una [aplicación cliente](#client-application) que inicia la autenticación del usuario final y captura el estado relacionado con el fin de adquirir un [token de seguridad](#security-token) y establecer el ámbito de la sesión de la aplicación en ese estado. El estado puede incluir artefactos, como información de perfil de usuario, e información derivada de las notificaciones de tokens.

Normalmente, la función de inicio de sesión de una aplicación se utiliza para implementar el inicio de sesión único (SSO). También puede ir precedido por una función de "suscripción", como punto de entrada para que un usuario final obtenga acceso a una aplicación (al primer inicio de sesión). La función de suscripción se usa para recopilar y conservar el estado adicional específico del usuario y puede requerir el [consentimiento del usuario](#consent).

## <a name="sign-out"></a>cierre de sesión

Proceso de cancelación de la autenticación de un usuario final para desasociar el estado de usuario asociado a la sesión de la [aplicación cliente](#client-application) durante el [inicio de sesión](#sign-in)

## <a name="tenant"></a>tenant

Una instancia de un directorio de Azure AD se conoce como inquilino de Azure AD. Proporciona varias características, como:

* Un servicio de registro de aplicaciones integradas
* Autenticación de cuentas de usuario y aplicaciones registradas
* Puntos de conexión de REST necesarios para admitir diversos protocolos, como OAuth2 y SAML, incluidos el [punto de conexión de autorización](#authorization-endpoint), el [punto de conexión de token](#token-endpoint) y el punto de conexión "común" utilizado por [aplicaciones multiinquilino](#multi-tenant-application).

Los inquilinos de Azure AD se crean/asocian con suscripciones de Azure y Office 365 durante el inicio de sesión, lo que proporciona características de Administración de identidad y acceso para la suscripción. Los administradores de la suscripción de Azure también pueden crear inquilinos de Azure AD adicionales través de Azure Portal. Consulte [Obtención de un inquilino de Azure Active Directory][AAD-How-To-Tenant] para más información sobre las diversas maneras de acceder a un inquilino. Consulte [Asociación de las suscripciones de Azure con Azure Active Directory][AAD-How-Subscriptions-Assoc] para más información sobre la relación entre las suscripciones y un inquilino de Azure AD.

## <a name="token-endpoint"></a>punto de conexión de token

Uno de los puntos de conexión implementados por el [servidor de autorización](#authorization-server) para admitir las [concesiones de autorización](#authorization-grant) de OAuth2. En función de la concesión, se puede utilizar para adquirir un [token de acceso](#access-token) (y el token de "actualización" relacionado) a un [cliente](#client-application), o un [token de identificador](#id-token) cuando se usa con el protocolo [OpenID Connect][OpenIDConnect].

## <a name="user-agent-based-client"></a>cliente basada en agente usuario

Un tipo de [aplicación cliente](#client-application) que descarga código desde un servidor web y se ejecuta dentro de un agente usuario (por ejemplo, un explorador web), como una aplicación de página única (SPA). Como todo el código se ejecuta en un dispositivo, se considera un cliente "público" debido a su incapacidad para almacenar credenciales de manera privada o confidencial. Consulte los [tipos de cliente y perfiles de OAuth2][OAuth2-Client-Types] para más información.

## <a name="user-principal"></a>entidad de seguridad de usuario

Similar a cómo se usa un objeto de entidad de servicio para representar una instancia de aplicación, un objeto de entidad de seguridad de usuario es otro tipo de entidad de seguridad, que representa a un usuario. La [entidad usuario][AAD-Graph-User-Entity] de Azure AD Graph define el esquema de un objeto de usuario, incluidas las propiedades relacionadas con el usuario, como el nombre y apellidos, el nombre principal del usuario, la pertenencia al rol de directorio, etc. Esto proporciona la configuración de la identidad de usuario para que Azure AD establezca una entidad de seguridad del usuario en tiempo de ejecución. La entidad de seguridad del usuario se utiliza para representar a un usuario autenticado para el inicio de sesión único, al registrar la delegación del [consentimiento](#consent) o tomar decisiones de control de acceso, etc.

## <a name="web-client"></a>cliente web

Un tipo de [aplicación cliente](#client-application) que ejecuta todo el código en un servidor web y puede funcionar como un cliente de "confidencial" al almacenar de forma segura sus credenciales en el servidor. Consulte los [tipos de cliente y perfiles de OAuth2][OAuth2-Client-Types] para más información.

## <a name="next-steps"></a>Pasos siguientes

El [Guía del desarrollador de Microsoft identity platform] [ AAD-Dev-Guide] es la página de inicio que se usará para todos los Microsoft identity platform relacionadas con el desarrollo temas, incluida una descripción general de [aplicación integración] [ AAD-How-To-Integrate] y los conceptos básicos de [autenticación de plataforma de identidad de Microsoft y escenarios de autenticación admitidos][AAD-Auth-Scenarios]. También puede encontrar ejemplos de código y tutoriales sobre cómo empezar a trabajar en [GitHub](https://github.com/azure-samples?utf8=%E2%9C%93&q=active%20directory&type=&language=).

Use la siguiente sección de comentarios para proporcionar comentarios y ayudarnos a mejorar y a dar forma a nuestro contenido. Puede incluir solicitudes de nuevas definiciones o de actualización de las existentes.

<!--Image references-->

<!--Reference style links -->
[AAD-App-Manifest]:reference-app-manifest.md
[AAD-App-SP-Objects]:app-objects-and-service-principals.md
[AAD-Auth-Scenarios]:authentication-scenarios.md
[AAD-Dev-Guide]:azure-ad-developers-guide.md
[AAD-Graph-Perm-Scopes]: https://msdn.microsoft.com/library/azure/ad/graph/howto/azure-ad-graph-api-permission-scopes
[AAD-Graph-App-Entity]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity
[AAD-Graph-Sp-Entity]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#serviceprincipal-entity
[AAD-Graph-User-Entity]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#user-entity
[AAD-How-Subscriptions-Assoc]:../fundamentals/active-directory-how-subscriptions-associated-directory.md
[AAD-How-To-Integrate]: ./active-directory-how-to-integrate.md
[AAD-How-To-Tenant]:quickstart-create-new-tenant.md
[AAD-Integrating-Apps]:quickstart-v1-integrate-apps-with-azure-ad.md
[AAD-Multi-Tenant-Overview]:howto-convert-app-to-be-multi-tenant.md
[AAD-Security-Token-Claims]: ./active-directory-authentication-scenarios/#claims-in-azure-ad-security-tokens
[AAD-Tokens-Claims]:access-tokens.md
[AZURE-portal]: https://portal.azure.com
[AAD-RBAC]: ../../role-based-access-control/role-assignments-portal.md
[JWT]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32
[Microsoft-Graph]: https://developer.microsoft.com/graph
[O365-Perm-Ref]: https://msdn.microsoft.com/office/office365/howto/application-manifest
[OAuth2-Access-Token-Scopes]: https://tools.ietf.org/html/rfc6749#section-3.3
[OAuth2-AuthZ-Endpoint]: https://tools.ietf.org/html/rfc6749#section-3.1
[OAuth2-AuthZ-Grant-Types]: https://tools.ietf.org/html/rfc6749#section-1.3
[OAuth2-Client-Types]: https://tools.ietf.org/html/rfc6749#section-2.1
[OAuth2-Role-Def]: https://tools.ietf.org/html/rfc6749#page-6
[OpenIDConnect]: https://openid.net/specs/openid-connect-core-1_0.html
[OpenIDConnect-AuthZ-Endpoint]: https://openid.net/specs/openid-connect-core-1_0.html#AuthorizationEndpoint
[OpenIDConnect-ID-Token]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken
