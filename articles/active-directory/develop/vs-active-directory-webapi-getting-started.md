---
title: Introducción a Azure AD en proyectos de WebApi de Visual Studio
description: Cómo empezar a usar Azure Active Directory en los proyectos de WebApi después de crear un Azure AD usando los servicios conectados de Visual Studio o de conectarse a él
author: ghogen
manager: jillfra
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: how-to
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: devx-track-csharp, aaddev, vs-azure
ms.openlocfilehash: fb8c1f25c8a22c96679a0310353f474f6b9d5ba5
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165437"
---
# <a name="get-started-with-azure-active-directory-webapi-projects"></a>Introducción a Azure Active Directory (proyectos de WebApi)

> [!div class="op_single_selector"]
> - [Introducción](vs-active-directory-webapi-getting-started.md)
> - [¿Qué ha ocurrido?](vs-active-directory-webapi-what-happened.md)

En este artículo se proporcionan instrucciones adicionales después de haber agregado Active Directory a un proyecto de WebAPI para ASP.NET mediante el comando **Proyecto > Servicios conectados** de Visual Studio. Si aún no ha agregado el servicio al proyecto, puede hacerlo en cualquier momento.

Para ver los cambios realizados en el proyecto al agregar el servicio conectado, consulte [¿Qué le ha ocurrido a mi proyecto de WebAPI?](vs-active-directory-webapi-what-happened.md)

## <a name="requiring-authentication-to-access-controllers"></a>Requerimiento de autenticación para obtener acceso a los controladores

Todos los controladores de su proyecto cuentan ahora con el atributo `[Authorize]`. Este atributo requiere que el usuario se autentique antes de tener acceso a las API definidas por estos controladores. Para permitir el acceso anónimo al controlador, quite este atributo del controlador. Si desea establecer los permisos a un nivel más detallado, aplique el atributo a cada método que requiere autorización en vez de aplicarlo a la clase del controlador.

## <a name="next-steps"></a>Pasos siguientes

- [Escenarios de autenticación para Azure Active Directory](./authentication-vs-authorization.md)
- [Adición de inicio de sesión con Microsoft a una aplicación web ASP.NET](quickstart-v2-aspnet-webapp.md)