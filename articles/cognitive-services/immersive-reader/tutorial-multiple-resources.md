---
title: 'Tutorial: Integración de varios recursos del Lector inmersivo'
titleSuffix: Azure Cognitive Services
description: En este tutorial, creará una aplicación de Node.js que inicia el Lector inmersivo mediante el uso de varios recursos del Lector inmersivo.
author: skamal
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: tutorial
ms.date: 01/14/2020
ms.author: skamal
ms.custom: devx-track-javascript
ms.openlocfilehash: 1abd1052c33346d949809d7c35a99ef97bde2c03
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88516289"
---
# <a name="tutorial-integrate-multiple-immersive-reader-resources"></a>Tutorial: Integración de varios recursos del Lector inmersivo

En la [introducción](./overview.md), se indica lo que es el Lector inmersivo y cómo implementa técnicas demostradas para mejorar la comprensión lectora de nuevos lectores, alumnos de idiomas y alumnos con diferencias de aprendizaje. En el [inicio rápido de Node. js](./quickstarts/client-libraries.md?pivots=programming-language-nodejs), ha aprendido a usar el Lector inmersivo con un solo recurso. En este tutorial se explica cómo integrar varios recursos del Lector inmersivo en la misma aplicación. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear varios recursos del Lector inmersivo en un grupo de recursos existente
> * Iniciar el Lector inmersivo mediante el uso de varios recursos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

* Siga el [inicio rápido](./quickstarts/client-libraries.md?pivots=programming-language-nodejs) para crear una aplicación web que inicie el Lector inmersivo con NodeJS. En ese inicio rápido, se configura un único recurso del Lector inmersivo. En este tutorial trabajaremos a partir de esto.

## <a name="create-the-immersive-reader-resources"></a>Creación de recursos del Lector inmersivo

Siga [estas instrucciones](./how-to-create-immersive-reader.md) para crear cada recurso del Lector inmersivo. El script **Create-ImmersiveReaderResource** tiene los siguientes parámetros: `ResourceName`, `ResourceSubdomain` y `ResourceLocation`. Estos deben ser únicos para cada recurso que se va a crear. Los parámetros restantes deben ser los mismos que los usados al configurar el primer recurso del Lector inmersivo. De esta manera, cada recurso se puede vincular al mismo grupo de recursos de Azure y la aplicación de Azure AD.

En el ejemplo siguiente se muestra cómo crear dos recursos, uno en WestUS y otro en EastUS. Observe los valores únicos de `ResourceName`, `ResourceSubdomain` y `ResourceLocation`.

```azurepowershell-interactive
Create-ImmersiveReaderResource
  -SubscriptionName <SUBSCRIPTION_NAME> `
  -ResourceName Resource_name_wus `
  -ResourceSubdomain resource-subdomain-wus `
  -ResourceSKU <RESOURCE_SKU> `
  -ResourceLocation westus `
  -ResourceGroupName <RESOURCE_GROUP_NAME> `
  -ResourceGroupLocation <RESOURCE_GROUP_LOCATION> `
  -AADAppDisplayName <AAD_APP_DISPLAY_NAME> `
  -AADAppIdentifierUri <AAD_APP_IDENTIFIER_URI> `
  -AADAppClientSecret <AAD_APP_CLIENT_SECRET>

Create-ImmersiveReaderResource
  -SubscriptionName <SUBSCRIPTION_NAME> `
  -ResourceName Resource_name_eus `
  -ResourceSubdomain resource-subdomain-eus `
  -ResourceSKU <RESOURCE_SKU> `
  -ResourceLocation eastus `
  -ResourceGroupName <RESOURCE_GROUP_NAME> `
  -ResourceGroupLocation <RESOURCE_GROUP_LOCATION> `
  -AADAppDisplayName <AAD_APP_DISPLAY_NAME> `
  -AADAppIdentifierUri <AAD_APP_IDENTIFIER_URI> `
  -AADAppClientSecret <AAD_APP_CLIENT_SECRET>
```

## <a name="add-resources-to-environment-configuration"></a>Incorporación de recursos a la configuración del entorno

En el inicio rápido, ha creado un archivo de configuración de entorno que contiene los parámetros `TenantId`, `ClientId`, `ClientSecret` y `Subdomain`. Dado que todos los recursos usan la misma aplicación de Azure AD, se pueden usar los mismos valores para `TenantId`, `ClientId` y `ClientSecret`. El único cambio necesario es la enumeración de cada subdominio para cada recurso.

El nuevo archivo __.env__ debería tener ahora un aspecto similar al siguiente ejemplo:

```text
TENANT_ID={YOUR_TENANT_ID}
CLIENT_ID={YOUR_CLIENT_ID}
CLIENT_SECRET={YOUR_CLIENT_SECRET}
SUBDOMAIN_WUS={YOUR_WESTUS_SUBDOMAIN}
SUBDOMAIN_EUS={YOUR_EASTUS_SUBDOMAIN}
```

Asegúrese de no confirmar este archivo en el control de código fuente, ya que contiene secretos que no deben hacerse públicos.

A continuación, vamos a modificar el archivo _routes\index.js_ que hemos creado para admitir los múltiples recursos. Sustituya su contenido por el código a continuación.

Como ya ha ocurrido anteriormente, este código crea un punto de conexión de la API que adquiere un token de autenticación de Azure AD mediante la contraseña de la entidad de servicio. Esta vez, permite al usuario especificar una ubicación de recursos y la pasa como un parámetro de consulta. Luego, devuelve un objeto que contiene el token y el subdominio correspondiente.

```javascript
var express = require('express');
var router = express.Router();
var request = require('request');

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});

router.get('/GetTokenAndSubdomain', function(req, res) {
    try {
        request.post({
            headers: {
                'content-type': 'application/x-www-form-urlencoded'
            },
            url: `https://login.windows.net/${process.env.TENANT_ID}/oauth2/token`,
            form: {
                grant_type: 'client_credentials',
                client_id: process.env.CLIENT_ID,
                client_secret: process.env.CLIENT_SECRET,
                resource: 'https://cognitiveservices.azure.com/'
            }
        },
        function(err, resp, tokenResult) {
            if (err) {
                console.log(err);
                return res.status(500).send('CogSvcs IssueToken error');
            }

            var tokenResultParsed = JSON.parse(tokenResult);

            if (tokenResultParsed.error) {
                console.log(tokenResult);
                return res.send({error :  "Unable to acquire Azure AD token. Check the debugger for more information."})
            }

            var token = tokenResultParsed.access_token;

            var subdomain = "";
            var region = req.query && req.query.region;
            switch (region) {
                case "eus":
                    subdomain = process.env.SUBDOMAIN_EUS
                    break;
                case "wus":
                default:
                    subdomain = process.env.SUBDOMAIN_WUS
            }

            return res.send({token, subdomain});
        });
    } catch (err) {
        console.log(err);
        return res.status(500).send('CogSvcs IssueToken error');
    }
});

module.exports = router;
```

El punto de conexión de la API **getimmersivereaderlaunchparams** debe estar protegido mediante algún tipo de autenticación (por ejemplo, [OAuth](https://oauth.net/2/)) para evitar que usuarios no autorizados obtengan tokens para utilizarlos contra el servicio Lector inmersivo y su facturación; este trabajo está fuera del ámbito de este tutorial.

## <a name="launch-the-immersive-reader-with-sample-content"></a>Iniciar el Lector inmersivo con contenido de muestra

1. Abra el archivo _views\index.pug_ y reemplace el contenido por el código siguiente. Este código rellena la página con contenido de ejemplo y agrega dos botones que inician el Lector inmersivo. Uno de ellos es para iniciar el Lector inmersivo para el recurso EastUS, y el otro para el recurso WestUS.

    ```pug
    doctype html
    html
        head
            title Immersive Reader Quickstart Node.js

            link(rel='stylesheet', href='https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css')

            // A polyfill for Promise is needed for IE11 support.
            script(src='https://cdn.jsdelivr.net/npm/promise-polyfill@8/dist/polyfill.min.js')

            script(src='https://contentstorage.onenote.office.net/onenoteltir/immersivereadersdk/immersive-reader-sdk.1.0.0.js')
            script(src='https://code.jquery.com/jquery-3.3.1.min.js')

            style(type="text/css").
                .immersive-reader-button {
                background-color: white;
                margin-top: 5px;
                border: 1px solid black;
                float: right;
                }
        body
            div(class="container")
                button(class="immersive-reader-button" data-button-style="icon" data-locale="en" onclick='handleLaunchImmersiveReader("wus")') WestUS Immersive Reader
                button(class="immersive-reader-button" data-button-style="icon" data-locale="en" onclick='handleLaunchImmersiveReader("eus")') EastUS Immersive Reader

                h1(id="ir-title") About Immersive Reader
                div(id="ir-content" lang="en-us")
                p Immersive Reader is a tool that implements proven techniques to improve reading comprehension for emerging readers, language learners, and people with learning differences. The Immersive Reader is designed to make reading more accessible for everyone. The Immersive Reader

                    ul
                        li Shows content in a minimal reading view
                        li Displays pictures of commonly used words
                        li Highlights nouns, verbs, adjectives, and adverbs
                        li Reads your content out loud to you
                        li Translates your content into another language
                        li Breaks down words into syllables

                h3 The Immersive Reader is available in many languages.

                p(lang="es-es") El Lector inmersivo está disponible en varios idiomas.
                p(lang="zh-cn") 沉浸式阅读器支持许多语言
                p(lang="de-de") Der plastische Reader ist in vielen Sprachen verfügbar.
                p(lang="ar-eg" dir="rtl" style="text-align:right") يتوفر \"القارئ الشامل\" في العديد من اللغات.

    script(type="text/javascript").
    function getTokenAndSubdomainAsync(region) {
            return new Promise(function (resolve, reject) {
                $.ajax({
                    url: "/GetTokenAndSubdomain",
                    type: "GET",
                    data: {
                        region: region
                    },
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

        function handleLaunchImmersiveReader(region) {
            getTokenAndSubdomainAsync(region)
                .then(function (response) {
                    const token = response["token"];
                    const subdomain = response["subdomain"];
                    // Learn more about chunk usage and supported MIME types https://docs.microsoft.com/azure/cognitive-services/immersive-reader/reference#chunk
                    const data = {
                        title: $("#ir-title").text(),
                        chunks: [{
                            content: $("#ir-content").html(),
                            mimeType: "text/html"
                        }]
                    };
                    // Learn more about options https://docs.microsoft.com/azure/cognitive-services/immersive-reader/reference#options
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
    ```

3. Nuestra aplicación web ya está lista. Inicie la aplicación; para ello, ejecute:

    ```bash
    npm start
    ```

4. Abra el explorador web y vaya a `http://localhost:3000`. Debería ver el contenido anterior en la página. Haga clic en el botón del **Lector inmersivo de EastUS** o en el botón del **Lector inmersivo de WestUS** para iniciar el Lector inmersivo con uno de los respectivos recursos.

## <a name="next-steps"></a>Pasos siguientes

* Explorar el [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk) y agregar la [Referencia del SDK del Lector inmersivo](./reference.md)
* Ver muestras de código en [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/advanced-csharp)