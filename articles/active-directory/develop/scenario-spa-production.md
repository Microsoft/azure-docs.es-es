---
title: 'Paso a producción de una aplicación de página única: Plataforma de identidad de Microsoft | Azure'
description: Aprenda a compilar una aplicación de página única (paso a producción)
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: nacanuma
ms.custom: aaddev
ms.openlocfilehash: 21ba0193c3f1e19ffc74452aaceee34759c7e606
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88949021"
---
# <a name="single-page-application-move-to-production"></a>Aplicación de página única: Paso a producción

Ahora que sabe cómo adquirir un token para llamar a las API web, conozca cómo pasarlo a producción.

## <a name="improve-your-app"></a>Mejorar la aplicación

[Habilite el registro](msal-logging.md) para tener la aplicación lista para producción.

## <a name="test-your-integration"></a>Probar la integración

Pruebe la integración siguiendo la [lista de comprobación de integración de la Plataforma de identidad de Microsoft](identity-platform-integration-checklist.md).

## <a name="deploy-your-app"></a>Implementación de la aplicación

Vea un [ejemplo de implementación](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter3) para aprender a implementar los proyectos de SPA y API web con Azure Storage y Azure App Services, respectivamente. 

## <a name="next-steps"></a>Pasos siguientes

Profundice en el ejemplo del inicio rápido, donde se explica el código para iniciar la sesión de los usuarios y obtener un token de acceso para llamar a **Microsoft Graph API** mediante **MSAL.js**:

> [!div class="nextstepaction"]
> [Tutorial de SPA de JavaScript](./tutorial-v2-javascript-spa.md)

Ejemplo en el que se muestra cómo obtener tokens para una API web de back-end (ASP.NET Core) propia mediante **MSAL.js**:

> [!div class="nextstepaction"]
> [SPA con un back-end de ASP.NET](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnetcore-webapi)

Ejemplo en el que se muestra cómo validar tokens de acceso para la API web de back-end (Node.js) mediante **passport-azure-ad**.

> [!div class="nextstepaction"]
> [API web de Node.js (Azure AD)](https://github.com/Azure-Samples/active-directory-javascript-nodejs-webapi-v2)

Ejemplo en el que se muestra cómo usar **MSAL.js** para iniciar la sesión de los usuarios en una aplicación registrada en **Azure Active Directory B2C** (Azure AD B2C):

> [!div class="nextstepaction"]
> [SPA con Azure AD B2C](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp)

Ejemplo en el que se muestra cómo usar **passport-azure-ad** para validar tokens de acceso para aplicaciones registradas en **Azure Active Directory B2C** (Azure AD B2C)

> [!div class="nextstepaction"]
> [API web de Node.js (Azure AD B2C)](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi)
