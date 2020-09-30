---
title: 'Escenario de aplicaciones de página única de JavaScript: plataforma de identidad de Microsoft | Azure'
description: Aprenda a compilar una aplicación de página única (información general del escenario) mediante la plataforma de identidad de Microsoft.
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: nacanuma
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 60e4ca80faa2c8787a13d87ab06cad9243299e50
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291964"
---
# <a name="scenario-single-page-application"></a>Escenario: Aplicación de una sola página

Aprenda lo necesario para crear una aplicación de página única (SPA)

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [Prerequisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="getting-started"></a>Introducción

Puede crear su primera aplicación siguiendo la guía de inicio rápido de SPA de JavaScript:

> [!div class="nextstepaction"]
> [Inicio rápido: Aplicación de página única](./quickstart-v2-javascript.md)

## <a name="overview"></a>Información general

Muchas aplicaciones web modernas se compilan como aplicaciones de página única del lado cliente. Los desarrolladores las escriben mediante JavaScript o un marco SPA, como Angular, Vue y React. Estas aplicaciones se ejecutan en un explorador web y tienen características de autenticación diferentes a las de las aplicaciones web del lado servidor tradicionales. 

La Plataforma de identidad de Microsoft ofrece **dos** opciones para que las aplicaciones de página única inicien la sesión de los usuarios y obtengan tokens para acceder a los servicios de back-end o a las API web:

- [Flujo de códigos de autorización de OAuth 2.0 (con PKCE)](./v2-oauth2-auth-code-flow.md) El flujo del código de autorización permite a la aplicación intercambiar un código de autorización para los tokens de **identificador** para representar al usuario autenticado y también los tokens de **acceso** necesarios para llamar a las API protegidas. Además, devuelve tokens de **actualización** que proporcionan acceso a largo plazo a los recursos en nombre de los usuarios sin necesidad de interacción con estos. Este es el enfoque **recomendado**.

![Aplicaciones de página única: autorización](./media/scenarios/spa-app-auth.svg)

- [Flujo implícito de OAuth 2.0](./v2-oauth2-implicit-grant-flow.md). El flujo de concesión implícita permite a la aplicación obtener tokens de **identificador** y de **acceso**. A diferencia del flujo del código de autorización, el flujo de concesión implícita no devuelve un **token de actualización**.

![Aplicaciones de página única: implícito](./media/scenarios/spa-app.svg)

Este flujo de autenticación no incluye escenarios de aplicaciones que usan marcos de JavaScript multiplataforma, como Electron y React Native, ya que requieren más funcionalidades para la interacción con las plataformas nativas.

## <a name="specifics"></a>Características específicas

Para habilitar este escenario en la aplicación, necesita lo siguiente:

* Registro de aplicaciones con Azure Active Directory (Azure AD). Los pasos para el registro difieren entre el flujo de concesión implícita y el flujo de código de autorización.
* La configuración de la aplicación con las propiedades de aplicación registradas, como el identificador de aplicación.
* Usar la biblioteca de autenticación de Microsoft para JavaScript (MSAL.js) para completar el flujo de autenticación a fin de iniciar sesión y adquirir tokens.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Registro de aplicaciones](scenario-spa-app-registration.md)
