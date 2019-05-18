---
title: Temas de ayuda del portal de registro de aplicaciones | Microsoft Docs
description: Descripción de las distintas características del portal de registro de aplicaciones de Microsoft.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: f0507c28-9464-4d3e-bd53-de9053fd5278
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/28/2018
ms.author: ryanwi
ms.reviewer: lenalepa
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: ec615e1c6229539958f66d0dca15cf7eb788e597
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546094"
---
# <a name="app-registration-reference"></a>Referencia del registro de aplicaciones
En este documento se proporciona el contexto y las descripciones de las distintas características que se encuentran en el [Portal de registro de aplicaciones](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/).

> [!NOTE]
> Ya no dará soporte registrar y administrar las aplicaciones convergentes y Azure AD en el [Portal de registro de aplicación](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/) a partir de mayo de 2019. Se recomienda administrar las aplicaciones existentes y permite registrar nuevas aplicaciones mediante el uso de la [registros de aplicaciones](https://aka.ms/appregistrations) experiencia en el portal de Azure.

## <a name="my-applications-or-converged-applications"></a>Mis aplicaciones o aplicaciones convergentes
Esta lista contiene todas las aplicaciones que se registran para su uso con el punto de conexión v2.0 de Azure AD. Estas aplicaciones tienen la capacidad de permitir el inicio de sesión de los usuarios tanto con cuentas personales de Microsoft como con cuentas profesionales o educativas de Azure Active Directory. Para obtener más información sobre el punto de conexión v2.0 de Azure AD, consulte la [información general sobre v2.0](active-directory-appmodel-v2-overview.md). Estas aplicaciones también se pueden usar para la integración con el punto de conexión de autenticación de la cuenta Microsoft, `https://login.live.com`.

## <a name="azure-ad-only-applications"></a>Aplicaciones que solo usan Azure AD
Esta lista contiene todas las aplicaciones que se registran para su uso con el punto de conexión de Azure AD v1.0. Estas aplicaciones solo tienen la capacidad de iniciar la sesión de usuarios con cuentas profesionales o educativas de Azure Active Directory. Esta lista incluye las aplicaciones que se registraron mediante los **registros de aplicaciones** de [Azure Portal](https://portal.azure.com).

## <a name="live-sdk-applications"></a>Aplicaciones de SDK de Live
Esta lista contiene todas las aplicaciones registradas para su uso únicamente con la cuenta Microsoft. No están habilitadas para su uso con Azure Active Directory. Aquí es donde encontrará todas las aplicaciones que se registraron previamente en el portal de desarrolladores de MSA en `https://account.live.com/developers/applications`. Todas las funciones que realizaba anteriormente en `https://account.live.com/developers/applications` ahora se pueden realizar en este nuevo portal, `https://apps.dev.microsoft.com`.

## <a name="application-secrets"></a>Secretos de aplicación
Los secretos de aplicación son credenciales que permiten que la aplicación realice una [autenticación de cliente](https://tools.ietf.org/html/rfc6749#section-2.3) confiable con Azure AD. En OAuth y OpenID Connect, un secreto de aplicación se conoce comúnmente como `client_secret`. En el protocolo v2.0, cualquier aplicación que reciba un token de seguridad en una ubicación direccionable web (mediante un esquema `https` ) debe usar un secreto de aplicación para identificarse en Azure AD en el momento del canje de ese token de seguridad. Además, cualquier cliente nativo que reciba tokens en un dispositivo tendrá prohibido el uso de un secreto de aplicación para realizar la autenticación de cliente. De esta forma, se evita el almacenamiento de secretos en entornos no seguros.

Cada aplicación puede contener dos secretos de aplicación válidos en un momento determinado. Al mantener dos secretos, tiene la posibilidad de realizar una sustitución de claves periódica en todo el entorno de la aplicación. Una vez que haya migrado la totalidad de la aplicación a un nuevo secreto, podrá eliminar el antiguo secreto y aprovisionar uno nuevo.

En este momento, solo se permiten dos tipos de secretos de aplicación en el portal de registro de aplicaciones. Al elegir **Generar nueva contraseña**, se genera y almacena un secreto compartido en el almacén de datos correspondiente, que podrá utilizar en su aplicación. Al elegir **Generar nuevo par de claves**, se crea un nuevo par de claves pública y privada que puede descargar y usar para la autenticación de cliente en Azure AD. Al elegir **Cargar clave pública**, podrá usar su propio par de claves pública y privada.
Debe cargar un certificado que incluya una clave pública.

## <a name="profile"></a>Perfil
La sección de perfil del portal de registro de aplicaciones se puede utilizar para personalizar la página de inicio de sesión de la aplicación. En este momento puede modificar el logotipo de la aplicación de la página de inicio de sesión, la dirección URL de los términos de servicio y la dirección URL de la declaración de privacidad. El logotipo debe ser una imagen transparente de 48 x 48 o 50 x 50 píxeles en un archivo GIF, PNG o JPEG de hasta 15 KB. Pruebe a cambiar los valores y ver la página de inicio de sesión resultante.

## <a name="live-sdk-support"></a>Compatibilidad con SDK de Live
Cuando habilita la "Compatibilidad con SDK de Live", los secretos de la aplicación que cree se aprovisionarán en los almacenes de datos tanto de Azure AD como de la cuenta Microsoft. Esto permite que la aplicación se integre directamente con el servicio de la cuenta de Microsoft (login.live.com). Si quiere compilar una aplicación usando directamente la cuenta de Microsoft (en lugar de usar el punto de conexión v2.0 de Azure AD), debe asegurarse de que esté habilitada la compatibilidad con SDK de Live.

Si deshabilita la compatibilidad con SDK de Live, se asegura de que el secreto de la aplicación solo se escriba en el almacén de datos de Azure AD. El almacén de datos de Azure AD incorpora las normas empresariales que le que permiten satisfacer ciertas regulaciones, como el cumplimiento de FISMA. Si habilita la compatibilidad con el SDK de Live, es posible que la aplicación no pueda lograr el cumplimiento con algunas de estos regulaciones.

Si solo piensa usar el punto de conexión v2.0 de Azure AD, puede deshabilitar sin riesgos la compatibilidad del SDK de Live.

