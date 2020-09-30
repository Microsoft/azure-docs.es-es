---
title: Inicio rápido de la biblioteca cliente de C# para el Lector inmersivo
titleSuffix: Azure Cognitive Services
description: En este inicio rápido, va a crear una aplicación web desde cero y a agregar la funcionalidad de la API de Lector inmersivo.
services: cognitive-services
author: nitinme
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 05/20/2020
ms.author: nitinme
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: f3d694a1e1eb368a97d994ebe9885c279ff44463
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89505422"
---
El [Lector inmersivo](https://www.onenote.com/learningtools) es una herramienta diseñada de forma inclusiva que implementa técnicas demostradas para mejorar la comprensión lectora.

En este inicio rápido, se va a crear una aplicación web desde cero y se integrará el Lector inmersivo mediante la biblioteca cliente de esta herramienta. Se puede encontrar un completo ejemplo funcional [aquí](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/quickstart-csharp).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads)
* Un recurso del Lector inmersivo configurado para la autenticación de Azure Active Directory. Siga [estas instrucciones](../../how-to-create-immersive-reader.md) para realizar la configuración. Al configurar las propiedades del proyecto de ejemplo, necesitará algunos de los valores creados aquí. Guarde la salida de la sesión en un archivo de texto para futuras referencias.

## <a name="create-a-web-app-project"></a>Creación de un proyecto de aplicación web

Cree un proyecto en Visual Studio mediante la plantilla de aplicación web de ASP.NET Core con Modelo-Vista-Controlador integrado y ASP.NET Core 2.1. Asigne al proyecto el nombre "QuickstartSampleWebApp".

![Nuevo proyecto](../../media/quickstart-csharp/1-createproject.png)

![Configuración del nuevo proyecto](../../media/quickstart-csharp/2-configureproject.png)

![Nueva aplicación web de ASP.NET Core](../../media/quickstart-csharp/3-createmvc.png)

## <a name="set-up-authentication"></a>Configuración de la autenticación

### <a name="configure-authentication-values"></a>Configuración de los valores de autenticación

Haga clic con el botón derecho en el proyecto en el _Explorador de soluciones_ y elija **Administrar secretos de usuario**. Se abrirá un archivo denominado _secrets.json_. Este archivo no está protegido bajo control de código fuente. Obtenga más información [aquí](https://docs.microsoft.com/aspnet/core/security/app-secrets?view=aspnetcore-3.1&tabs=windows). Reemplace el contenido de _secrets. json_ con lo siguiente, y proporcione los valores especificados al crear el recurso del Lector inmersivo.

```json
{
  "TenantId": "YOUR_TENANT_ID",
  "ClientId": "YOUR_CLIENT_ID",
  "ClientSecret": "YOUR_CLIENT_SECRET",
  "Subdomain": "YOUR_SUBDOMAIN"
}
```

### <a name="add-the-microsoftidentitymodelclientsactivedirectory-nuget-package"></a>Adición del paquete NuGet Microsoft.IdentityModel.Clients.ActiveDirectory

El código siguiente usa objetos del paquete NuGet **Microsoft.IdentityModel.Clients.ActiveDirectory**, por lo que tendrá que agregar una referencia a ese paquete en el proyecto.

Abra la Consola del Administrador de paquetes NuGet en **Herramientas -> Administrador de paquetes NuGet -> Consola del Administrador de paquetes** y ejecute el siguiente comando:

```powershell
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Version 5.2.0
```

### <a name="update-the-controller-to-acquire-the-token"></a>Actualización del controlador para adquirir el token 

Abra _Controllers\HomeController.cs_ y agregue el código siguiente después de las instrucciones _using_ en la parte superior del archivo.

```csharp
using Microsoft.IdentityModel.Clients.ActiveDirectory;
```

Ahora, configuraremos el controlador para obtener los valores de Azure AD de _secrets.json_. En la parte superior de la clase _HomeController_, después de ```public class HomeController : Controller {``` agregue el código siguiente.

```csharp
private readonly string TenantId;     // Azure subscription TenantId
private readonly string ClientId;     // Azure AD ApplicationId
private readonly string ClientSecret; // Azure AD Application Service Principal password
private readonly string Subdomain;    // Immersive Reader resource subdomain (resource 'Name' if the resource was created in the Azure portal, or 'CustomSubDomain' option if the resource was created with Azure CLI Powershell. Check the Azure portal for the subdomain on the Endpoint in the resource Overview page, for example, 'https://[SUBDOMAIN].cognitiveservices.azure.com/')

public HomeController(Microsoft.Extensions.Configuration.IConfiguration configuration)
{
    TenantId = configuration["TenantId"];
    ClientId = configuration["ClientId"];
    ClientSecret = configuration["ClientSecret"];
    Subdomain = configuration["Subdomain"];

    if (string.IsNullOrWhiteSpace(TenantId))
    {
        throw new ArgumentNullException("TenantId is null! Did you add that info to secrets.json?");
    }

    if (string.IsNullOrWhiteSpace(ClientId))
    {
        throw new ArgumentNullException("ClientId is null! Did you add that info to secrets.json?");
    }

    if (string.IsNullOrWhiteSpace(ClientSecret))
    {
        throw new ArgumentNullException("ClientSecret is null! Did you add that info to secrets.json?");
    }

    if (string.IsNullOrWhiteSpace(Subdomain))
    {
        throw new ArgumentNullException("Subdomain is null! Did you add that info to secrets.json?");
    }
}

/// <summary>
/// Get an Azure AD authentication token
/// </summary>
private async Task<string> GetTokenAsync()
{
    string authority = $"https://login.windows.net/{TenantId}";
    const string resource = "https://cognitiveservices.azure.com/";

    AuthenticationContext authContext = new AuthenticationContext(authority);
    ClientCredential clientCredential = new ClientCredential(ClientId, ClientSecret);

    AuthenticationResult authResult = await authContext.AcquireTokenAsync(resource, clientCredential);

    return authResult.AccessToken;
}

[HttpGet]
public async Task<JsonResult> GetTokenAndSubdomain()
{
    try
    {
        string tokenResult = await GetTokenAsync();

        return new JsonResult(new { token = tokenResult, subdomain = Subdomain });
    }
    catch (Exception e)
    {
        string message = "Unable to acquire Azure AD token. Check the debugger for more information.";
        Debug.WriteLine(message, e);
        return new JsonResult(new { error = message });
    }
}
```

## <a name="add-sample-content"></a>Adición de contenido de ejemplo
En primer lugar, abra _Views\Shared\Layout.cshtml_. Antes de la línea ```</head>```, agregue el código siguiente:

```html
@RenderSection("Styles", required: false)
```

Ahora, vamos a agregar contenido de ejemplo a esta aplicación web. Abra _Views\Home\Index.cshtml_ y reemplace el código generado automáticamente por este ejemplo:

```html
@{
    ViewData["Title"] = "Immersive Reader C# Quickstart";
}

@section Styles {
    <style type="text/css">
        .immersive-reader-button {
            background-color: white;
            margin-top: 5px;
            border: 1px solid black;
            float: right;
        }
    </style>
}

<div class="container">
    <button class="immersive-reader-button" data-button-style="iconAndText" data-locale="en"></button>

    <h1 id="ir-title">About Immersive Reader</h1>
    <div id="ir-content" lang="en-us">
        <p>
            Immersive Reader is a tool that implements proven techniques to improve reading comprehension for emerging readers, language learners, and people with learning differences.
            The Immersive Reader is designed to make reading more accessible for everyone. The Immersive Reader
            <ul>
                <li>
                    Shows content in a minimal reading view
                </li>
                <li>
                    Displays pictures of commonly used words
                </li>
                <li>
                    Highlights nouns, verbs, adjectives, and adverbs
                </li>
                <li>
                    Reads your content out loud to you
                </li>
                <li>
                    Translates your content into another language
                </li>
                <li>
                    Breaks down words into syllables
                </li>
            </ul>
        </p>
        <h3>
            The Immersive Reader is available in many languages.
        </h3>
        <p lang="es-es">
            El Lector inmersivo está disponible en varios idiomas.
        </p>
        <p lang="zh-cn">
            沉浸式阅读器支持许多语言
        </p>
        <p lang="de-de">
            Der plastische Reader ist in vielen Sprachen verfügbar.
        </p>
        <p lang="ar-eg" dir="rtl" style="text-align:right">
            يتوفر \"القارئ الشامل\" في العديد من اللغات.
        </p>
    </div>
</div>
```

Tenga en cuenta que todo el texto tiene un atributo **lang**, que describe los idiomas del texto. Este atributo ayuda al Lector inmersivo a proporcionar características de idioma y gramática pertinentes.

## <a name="add-javascript-to-handle-launching-the-immersive-reader"></a>Incorporación de JavaScript para administrar el inicio del Lector inmersivo

La biblioteca del Lector inmersivo proporciona funcionalidades como el inicio del Lector inmersivo y la representación de sus botones. Obtenga más información [aquí](https://docs.microsoft.com/azure/cognitive-services/immersive-reader/reference).

En la parte inferior de _Views\Home\Index.cshtml_, agregue el siguiente código:

```html
@section Scripts
{
    <script src="https://contentstorage.onenote.office.net/onenoteltir/immersivereadersdk/immersive-reader-sdk.1.0.0.js"></script>
    <script>
        function getTokenAndSubdomainAsync() {
            return new Promise(function (resolve, reject) {
                $.ajax({
                    url: "@Url.Action("GetTokenAndSubdomain", "Home")",
                    type: "GET",
                    success: function (data) {
                        if (data.error) {
                            reject(data.error);
                        } else {
                            resolve(data);
                        }
                    },
                    error: function (err) {
                        reject(err);
                    }
                });
            });
        }
    
        $(".immersive-reader-button").click(function () {
            handleLaunchImmersiveReader();
        });
    
        function handleLaunchImmersiveReader() {
            getTokenAndSubdomainAsync()
                .then(function (response) {
                    const token = response["token"];
                    const subdomain = response["subdomain"];
    
                    // Learn more about chunk usage and supported MIME types https://docs.microsoft.com/en-us/azure/cognitive-services/immersive-reader/reference#chunk
                    const data = {
                        title: $("#ir-title").text(),
                        chunks: [{
                            content: $("#ir-content").html(),
                            mimeType: "text/html"
                        }]
                    };
    
                    // Learn more about options https://docs.microsoft.com/en-us/azure/cognitive-services/immersive-reader/reference#options
                    const options = {
                        "onExit": exitCallback,
                        "uiZIndex": 2000
                    };
    
                    ImmersiveReader.launchAsync(token, subdomain, data, options)
                        .catch(function (error) {
                            alert("Error in launching the Immersive Reader. Check the console.");
                            console.log(error);
                        });
                })
                .catch(function (error) {
                    alert("Error in getting the Immersive Reader token and subdomain. Check the console.");
                    console.log(error);
                });
        }
    
        function exitCallback() {
            console.log("This is the callback function. It is executed when the Immersive Reader closes.");
        }
    </script>
}
```

## <a name="build-and-run-the-app"></a>Compilación y ejecución de la aplicación

En la barra de menús, seleccione **Depurar > Iniciar depuración** o presione **F5** para iniciar la depuración.

En el explorador, verá:

![Aplicación de ejemplo](../../media/quickstart-csharp/4-buildapp.png)

## <a name="launch-the-immersive-reader"></a>Inicio del Lector inmersivo

Al hacer clic en el botón "Lector inmersivo", verá que se inicia dicha herramienta con el contenido de la página.

![Lector inmersivo](../../media/quickstart-csharp/5-viewimmersivereader.png)

## <a name="next-steps"></a>Pasos siguientes

* Consulte el [inicio rápido de Node.js](../../tutorial-nodejs.md) para ver qué más puede hacer con la biblioteca cliente del Lector inmersivo con Node.js.
* Consulte el [tutorial de Android](../../tutorial-android.md) para ver qué más puede hacer con el SDK del Lector inmersivo con Java o Kotlin para Android.
* Consulte el [tutorial de iOS](../../tutorial-ios.md) para ver qué más puede hacer con el SDK del Lector inmersivo con Swift para iOS.
* Consulte el [tutorial de Python](../../tutorial-python.md) para ver qué más puede hacer con la biblioteca cliente del Lector inmersivo con Python.
* Explorar el [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk) y agregar la [Referencia del SDK del Lector inmersivo](../../reference.md)