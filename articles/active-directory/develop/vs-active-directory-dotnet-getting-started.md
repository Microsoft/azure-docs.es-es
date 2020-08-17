---
title: Introducción a Azure AD en proyectos .NET MVC | Azure
description: Cómo empezar a usar Azure Active Directory en proyectos .NET MVC después de crear una instancia de Azure AD mediante los servicios conectados de Visual Studio, o de establecer una conexión con ella
author: ghogen
manager: jillfra
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: how-to
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: devx-track-csharp, aaddev, vs-azure
ms.openlocfilehash: 15a7c873e4d1e5c962a89b03f2a5cafc88843192
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165471"
---
# <a name="getting-started-with-azure-active-directory-aspnet-mvc-projects"></a>Introducción a Azure Active Directory (proyectos ASP.NET MVC)

> [!div class="op_single_selector"]
> - [Introducción](vs-active-directory-dotnet-getting-started.md)
> - [¿Qué ha ocurrido?](vs-active-directory-dotnet-what-happened.md)

En este artículo se proporcionan instrucciones adicionales después de haber agregado Active Directory a un proyecto ASP.NET MVC mediante el comando **Proyecto > Servicios conectados** de Visual Studio. Si aún no ha agregado el servicio al proyecto, puede hacerlo en cualquier momento.

Para ver los cambios realizados en el proyecto al agregar el servicio conectado, consulte [¿Qué le ha ocurrido a mi proyecto MVC?](vs-active-directory-dotnet-what-happened.md)

## <a name="requiring-authentication-to-access-controllers"></a>Requerimiento de autenticación para obtener acceso a los controladores

Todos los controladores de su proyecto cuentan ahora con el atributo `[Authorize]`. Este atributo requerirá que el usuario se autentique antes de tener acceso a estos controladores. Para permitir el acceso anónimo al controlador, quite este atributo del controlador. Si desea establecer los permisos a un nivel más detallado, aplique el atributo a cada método que requiere autorización en vez de aplicarlo a la clase del controlador.

## <a name="adding-signin--signout-controls"></a>Incorporación de controles SignIn / SignOut

Para agregar los controles SignIn/SignOut a su vista, puede usar la vista parcial `_LoginPartial.cshtml` para agregar la funcionalidad a una de sus vistas. Este es un ejemplo de la funcionalidad agregada a la vista estándar `_Layout.cshtml`. Observe el último elemento de la sección div con la clase navbar-collapse:

```html
<!DOCTYPE html>
 <html>
 <head>
     <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@ViewBag.Title - My ASP.NET Application</title>
    @Styles.Render("~/Content/css")
    @Scripts.Render("~/bundles/modernizr")
</head>
<body>
    <div class="navbar navbar-inverse navbar-fixed-top">
        <div class="container">
            <div class="navbar-header">
                <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                @Html.ActionLink("Application name", "Index", "Home", new { area = "" }, new { @class = "navbar-brand" })
            </div>
            <div class="navbar-collapse collapse">
                <ul class="nav navbar-nav">
                    <li>@Html.ActionLink("Home", "Index", "Home")</li>
                    <li>@Html.ActionLink("About", "About", "Home")</li>
                    <li>@Html.ActionLink("Contact", "Contact", "Home")</li>
                </ul>
                @Html.Partial("_LoginPartial")
            </div>
        </div>
    </div>
    <div class="container body-content">
        @RenderBody() 
        <hr />
        <footer>
            <p>&copy; @DateTime.Now.Year - My ASP.NET Application</p>
        </footer>
    </div>
    @Scripts.Render("~/bundles/jquery")
    @Scripts.Render("~/bundles/bootstrap")
    @RenderSection("scripts", required: false)
</body>
</html>
```

## <a name="next-steps"></a>Pasos siguientes

- [Escenarios de autenticación para Azure Active Directory](./authentication-vs-authorization.md)
- [Adición de inicio de sesión con Microsoft a una aplicación web ASP.NET](quickstart-v2-aspnet-webapp.md)