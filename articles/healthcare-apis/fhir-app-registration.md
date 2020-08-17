---
title: Registro de aplicaciones de Azure Active Directory para Azure API for FHIR
description: En este tutorial se explica qué aplicaciones deben registrarse para Azure API for FHIR y FHIR Server for Azure.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: overview
ms.reviewer: dseven
ms.author: matjazl
author: matjazl
ms.date: 10/13/2019
ms.openlocfilehash: 22f31cf3911b5ea24e8798fb226e389071fadd0b
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87848985"
---
# <a name="register-the-azure-active-directory-apps-for-azure-api-for-fhir"></a>Registro de aplicaciones de Azure Active Directory para Azure API for FHIR

Tiene varias opciones de configuración para elegir al configurar Azure API for FHIR o FHIR Server for Azure (OSS). Para código abierto, tendrá que crear su propio registro de aplicación de recursos. En el caso de Azure API for FHIR, esta aplicación de recursos se crea automáticamente.

## <a name="application-registrations"></a>Registros de aplicación

Para que una aplicación interactúe con Azure AD, es necesario registrarla. En el contexto de FHIR Server, hay dos tipos de registros de aplicaciones que se deben analizar:

1. Registros de aplicaciones de recursos.
1. Registros de aplicaciones cliente.

Las **aplicaciones de recursos** son representaciones en Azure AD de una API o recurso que está protegido con Azure AD; un ejemplo concreto sería Azure API for FHIR. Se creará automáticamente una aplicación de recursos para Azure API for FHIR al aprovisionar el servicio, pero si va a usar el servidor de código abierto, tendrá que [registrar una aplicación de recursos](register-resource-azure-ad-client-app.md) en Azure AD. Esta aplicación de recursos tendrá un identificador URI. Se recomienda que este URI sea el mismo que el del servidor de FHIR. Este URI debe usarse como `Audience` para el servidor de FHIR. Una aplicación cliente puede solicitar acceso a este servidor de FHIR cuando solicita un token.

Las *aplicaciones cliente* son registros de los clientes que van a solicitar tokens. A menudo, en OAuth 2.0, se distinguen al menos tres tipos diferentes de aplicaciones:

1. **Clientes confidenciales**, también conocidos como aplicaciones web en Azure AD. Los clientes confidenciales son aplicaciones que usan [flujo de código de autorización](https://docs.microsoft.com/azure/active-directory/develop/v1-protocols-oauth-code) para obtener un token en nombre de un usuario con la sesión iniciada mediante la presentación de credenciales válidas. Se denominan clientes confidenciales porque pueden contener un secreto y presentarán este secreto a Azure AD al intercambiar el código de autenticación de un token. Dado que los clientes confidenciales pueden autenticarse a sí mismos mediante el secreto de cliente, son de mayor confianza que los clientes públicos y pueden tener tokens de mayor duración y se les puede conceder un token de actualización. Lea los detalles sobre cómo [registrar un cliente confidencial](register-confidential-azure-ad-client-app.md). Tenga en cuenta que es importante registrar la dirección URL de respuesta en la que el cliente va a recibir el código de autorización.
1. **Clientes públicos**. Se trata de clientes que no pueden contener un secreto. Normalmente se trata de una aplicación de dispositivo móvil o una aplicación de página única de JavaScript, donde un usuario podría detectar un secreto en el cliente. Los clientes públicos también usan el flujo de código de autorización, pero no se les permite presentar un secreto al obtener un token y pueden tener tokens de menor duración y ningún token de actualización. Lea los detalles sobre cómo [registrar un cliente público](register-public-azure-ad-client-app.md).
1. Clientes de servicios. Estos clientes obtienen tokens en nombre de ellos mismos (no en nombre de un usuario) mediante el [flujo de credenciales de cliente](https://docs.microsoft.com/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow). Normalmente representan aplicaciones que acceden al servidor de FHIR de forma no interactiva. Un ejemplo podría ser un proceso de ingesta. Cuando se usa un cliente de servicio, no es necesario iniciar el proceso de obtención de un token con una llamada al punto de conexión de `/authorize`. Un cliente de servicio puede ir directamente al punto de conexión de `/token` y presentar el identificador y el secreto de cliente para obtener un token. Lea los detalles sobre cómo [registrar un cliente de servicio](register-service-azure-ad-client-app.md).

## <a name="next-steps"></a>Pasos siguientes

En esta información general, ha visto los tipos de registros de aplicaciones que puede necesitar para trabajar con una API de FHIR.

En función de la configuración, consulte las guías de procedimientos para registrar las aplicaciones.

* [Registro de una aplicación de recursos](register-resource-azure-ad-client-app.md)
* [Registro de una aplicación cliente confidencial](register-confidential-azure-ad-client-app.md)
* [Registro de una aplicación cliente pública](register-public-azure-ad-client-app.md)
* [Registro de una aplicación de servicio](register-service-azure-ad-client-app.md)

Una vez que haya registrado las aplicaciones, puede implementar Azure API for FHIR.

>[!div class="nextstepaction"]
>[Implementación de Azure API for FHIR](fhir-paas-powershell-quickstart.md)