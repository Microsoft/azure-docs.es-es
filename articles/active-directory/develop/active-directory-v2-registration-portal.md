---
title: Referencia del portal de registro de aplicaciones | Azure
titleSuffix: Microsoft identity platform
description: Descripción de las características del portal de registro de aplicaciones de Microsoft.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 08/13/2019
ms.author: ryanwi
ms.reviewer: lenalepa
ms.custom: aaddev
ms.openlocfilehash: 0574eb0b8ab0b5d4de671ad44a45a85e434393f1
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88115584"
---
# <a name="app-registration-reference"></a>Referencia del registro de aplicaciones

En este documento se proporciona el contexto y las descripciones de las distintas características que se encuentran en la experiencia de [Registros de aplicaciones](https://aka.ms/appregistrations) en Azure Portal.

## <a name="my-applications-or-converged-applications"></a>Mis aplicaciones o aplicaciones convergentes

En esta lista se incluyen todas las aplicaciones que se registran para su uso con el punto de conexión de la Plataforma de identidad de Microsoft (v2.0). Estas aplicaciones tienen la capacidad de permitir el inicio de sesión de los usuarios tanto con cuentas personales de Microsoft como con cuentas profesionales o educativas de Azure Active Directory. Para más información sobre el punto de conexión de la plataforma de identidad, consulte la [introducción a v2.0](./v2-overview.md). Estas aplicaciones también se pueden usar para la integración con el punto de conexión de autenticación de la cuenta Microsoft, `https://login.live.com`.

## <a name="azure-ad-only-applications"></a>Aplicaciones que solo usan Azure AD

Esta lista contiene todas las aplicaciones que se registran para su uso con el punto de conexión de Azure AD v1.0. Estas aplicaciones solo tienen la capacidad de iniciar la sesión de usuarios con cuentas profesionales o educativas de Azure Active Directory. Esta lista incluye las aplicaciones que se registraron mediante **Registros de aplicaciones** de [Azure Portal](https://portal.azure.com).

## <a name="live-sdk-applications"></a>Aplicaciones de SDK de Live

Esta lista contiene todas las aplicaciones registradas para su uso únicamente con la cuenta Microsoft. No están habilitadas para su uso con Azure Active Directory. Aquí es donde encontrará todas las aplicaciones que se registraron previamente en el portal de desarrolladores de MSA en `https://account.live.com/developers/applications`. Todas las funciones que realizaba anteriormente en `https://account.live.com/developers/applications` ahora se pueden realizar en [Registros de aplicaciones](https://aka.ms/appregistrations).

## <a name="application-secrets"></a>Secretos de aplicación

Los secretos de aplicación son credenciales que permiten que la aplicación realice una [autenticación de cliente](https://tools.ietf.org/html/rfc6749#section-2.3) confiable con la Plataforma de identidad de Microsoft. En OAuth y OpenID Connect, un secreto de aplicación se conoce comúnmente como `client_secret`. En el protocolo v2.0, cualquier aplicación que reciba un token de seguridad en una ubicación direccionable web (mediante un esquema `https`) debe usar un secreto de aplicación para identificarse en la Plataforma de identidad de Microsoft en el momento del canje de ese token de seguridad. Además, cualquier cliente nativo que reciba tokens en un dispositivo tendrá prohibido el uso de un secreto de aplicación para realizar la autenticación de cliente. De esta forma, se evita el almacenamiento de secretos en entornos no seguros.

Cada aplicación puede contener dos secretos de aplicación válidos en un momento determinado. Al mantener dos secretos, tiene la posibilidad de realizar una sustitución de claves periódica en todo el entorno de la aplicación. Una vez que haya migrado la totalidad de la aplicación a un nuevo secreto, podrá eliminar el antiguo secreto y aprovisionar uno nuevo.

En este momento, solo se permiten dos tipos de secretos de aplicación en el portal de registro de aplicaciones. Al elegir **Generar nueva contraseña**, se genera y almacena un secreto compartido en el almacén de datos correspondiente, que podrá utilizar en su aplicación. Al elegir **Generar nuevo par de claves**, se crea un nuevo par de claves pública y privada que puede descargar y usar para la autenticación de cliente en la Plataforma de identidad de Microsoft. Al elegir **Cargar clave pública**, podrá usar su propio par de claves pública y privada.
Debe cargar un certificado que incluya una clave pública.

## <a name="profile"></a>Perfil

La sección de perfil del portal de registro de aplicaciones se puede utilizar para personalizar la página de inicio de sesión de la aplicación. En este momento puede modificar el logotipo de la aplicación de la página de inicio de sesión, la dirección URL de los términos de servicio y la dirección URL de la declaración de privacidad. El logotipo debe ser una imagen transparente de 48 x 48 o 50 x 50 píxeles en un archivo de formato GIF, PNG o JPEG que tenga un tamaño máximo de 15 KB. Pruebe a cambiar los valores y ver la página de inicio de sesión resultante.

## <a name="live-sdk-support"></a>Compatibilidad con SDK de Live

Cuando habilita la "Compatibilidad con SDK de Live", los secretos de la aplicación que cree se aprovisionarán en los almacenes de datos tanto de Azure AD como de la cuenta Microsoft. Esto permite que la aplicación se integre directamente con el servicio de la cuenta de Microsoft (login.live.com). Si quiere crear una aplicación con la cuenta de Microsoft directamente (en lugar de usar el punto de conexión v2.0), debe asegurarse de que esté habilitada la compatibilidad con Live SDK.

Si deshabilita la compatibilidad con SDK de Live, se asegura de que el secreto de la aplicación solo se escriba en el almacén de datos de Azure AD. El almacén de datos de Azure AD incorpora las normas empresariales que le que permiten satisfacer ciertas regulaciones, como el cumplimiento de FISMA. Si habilita la compatibilidad con el SDK de Live, es posible que la aplicación no pueda lograr el cumplimiento con algunas de estos regulaciones.

Si solo piensa usar el punto de conexión v2.0 de Azure AD, puede deshabilitar sin riesgos la compatibilidad de Live SDK.