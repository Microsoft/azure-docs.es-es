---
title: 'Tutorial: Iniciar Lector inmersivo mediante Node.js'
titleSuffix: Azure Cognitive Services
description: En este tutorial, creará una aplicación de Node.js que inicia el Lector inmersivo.
services: cognitive-services
author: metanMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: tutorial
ms.date: 01/14/2020
ms.author: metan
ms.custom: devx-track-javascript
ms.openlocfilehash: c64fc95c44aad81087cada23639e3b13bf478056
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407190"
---
# <a name="tutorial-launch-the-immersive-reader-nodejs"></a>Tutorial: Inicio del Lector inmersivo (Node.js)

En la [introducción](./overview.md), se indica lo que es el Lector inmersivo y cómo implementa técnicas demostradas para mejorar la comprensión lectora de nuevos lectores, alumnos de idiomas y alumnos con diferencias de aprendizaje. En este tutorial, se explica cómo crear una aplicación web de Node.js que inicia el Lector inmersivo. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una aplicación web de Node.js con Express
> * Adquisición de un token de acceso
> * Iniciar el Lector inmersivo con contenido de muestra
> * Especificar el idioma del contenido
> * Especificar el idioma de la interfaz del Lector inmersivo
> * Iniciar el Lector inmersivo con contenido de matemáticas

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

* Un recurso del Lector inmersivo configurado para la autenticación de Azure Active Directory. Siga [estas instrucciones](./how-to-create-immersive-reader.md) para realizar la configuración. Necesitará algunos de los valores que se crean aquí cuando configure las propiedades del entorno. Guarde la salida de la sesión en un archivo de texto para futuras referencias.
* [Node.js](https://nodejs.org/) y [Yarn](https://yarnpkg.com)
* Un IDE como [Visual Studio Code](https://code.visualstudio.com/)

## <a name="create-a-nodejs-web-app-with-express"></a>Crear una aplicación web de Node.js con Express

Cree una aplicación web de Node.js con la herramienta `express-generator`.

```bash
npm install express-generator -g
express --view=pug myapp
cd myapp
```

Instale las dependencias de yarn y agregue las dependencias `request` y `dotenv`, que se utilizarán más adelante en el tutorial.

```bash
yarn
yarn add request
yarn add dotenv
```

## <a name="acquire-an-azure-ad-authentication-token"></a>Adquisición de un token de autenticación de Azure AD

Después, escriba una API de back-end para recuperar un token de autenticación de Azure AD.

Necesita algunos valores del paso del requisito previo de configuración de la autenticación de Azure AD para esta parte. Vuelva a consultar el archivo de texto que ha guardado de esa sesión.

````text
TenantId     => Azure subscription TenantId
ClientId     => Azure AD ApplicationId
ClientSecret => Azure AD Application Service Principal password
Subdomain    => Immersive Reader resource subdomain (resource 'Name' if the resource was created in the Azure portal, or 'CustomSubDomain' option if the resource was created with Azure CLI Powershell. Check the Azure portal for the subdomain on the Endpoint in the resource Overview page, for example, 'https://[SUBDOMAIN].cognitiveservices.azure.com/')
````

Una vez que tenga estos valores, cree un nuevo archivo llamado _.env_ y pegue el siguiente código en él, proporcionando sus valores de propiedad personalizados desde arriba. No incluya comillas ni los caracteres "{" y "}".

```text
TENANT_ID={YOUR_TENANT_ID}
CLIENT_ID={YOUR_CLIENT_ID}
CLIENT_SECRET={YOUR_CLIENT_SECRET}
SUBDOMAIN={YOUR_SUBDOMAIN}
```

Asegúrese de no confirmar este archivo en el control de código fuente, ya que contiene secretos que no deben hacerse públicos.

Después, abra el archivo _app.js_ y agregue lo siente en la parte superior. Esto carga las propiedades definidas en el archivo .env como variables de entorno en el nodo.

```javascript
require('dotenv').config();
```

Abra el archivo _routes\index.js_y reemplace su contenido por el código siguiente.

Este código crea un punto de conexión de la API que adquiere un token de autenticación de Azure AD mediante la contraseña de la entidad de servicio. También recupera el subdominio. Luego, devuelve un objeto que contiene el token y el subdominio.

```javascript
var request = require('request');
var express = require('express');
var router = express.Router();

router.get('/getimmersivereaderlaunchparams', function(req, res) {
    request.post ({
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
        function(err, resp, tokenResponse) {
                if (err) {
                    return res.status(500).send('CogSvcs IssueToken error');
                }

                const token = JSON.parse(tokenResponse).access_token;
                const subdomain = process.env.SUBDOMAIN;
                return res.send({token: token, subdomain: subdomain});
        }
  );
});

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});

module.exports = router;

```

El punto de conexión de la API **getimmersivereaderlaunchparams** debe estar protegido mediante algún tipo de autenticación (por ejemplo, [OAuth](https://oauth.net/2/)) para evitar que usuarios no autorizados obtengan tokens para utilizarlos contra el servicio Lector inmersivo y su facturación; este trabajo está fuera del ámbito de este tutorial.

## <a name="launch-the-immersive-reader-with-sample-content"></a>Iniciar el Lector inmersivo con contenido de muestra

1. Abra _views\layout.pug_ y agregue el siguiente código bajo la etiqueta `head`, antes de la etiqueta `body`. Estas etiquetas `script` cargan el [DSK de Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk) y jQuery.

    ```pug
    script(src='https://contentstorage.onenote.office.net/onenoteltir/immersivereadersdk/immersive-reader-sdk.0.0.2.js')
    script(src='https://code.jquery.com/jquery-3.3.1.min.js')
    ```

2. Abra el archivo _views\index.pug_ y reemplace el contenido por el código siguiente. Este código rellena la página con algún contenido de muestra y agrega un botón que inicia el Lector inmersivo.

    ```pug
    extends layout

    block content
          h2(id='title') Geography
          p(id='content') The study of Earth's landforms is called physical geography. Landforms can be mountains and valleys. They can also be glaciers, lakes or rivers.
          div(class='immersive-reader-button' data-button-style='iconAndText' data-locale='en-US' onclick='launchImmersiveReader()')
          script.

            function getImmersiveReaderLaunchParamsAsync() {
                    return new Promise((resolve, reject) => {
                        $.ajax({
                                url: '/getimmersivereaderlaunchparams',
                                type: 'GET',
                                success: data => {
                                        resolve(data);
                                },
                                error: err => {
                                        console.log('Error in getting token and subdomain!', err);
                                        reject(err);
                                }
                        });
                    });
            }

            async function launchImmersiveReader() {
                    const content = {
                            title: document.getElementById('title').innerText,
                            chunks: [{
                                    content: document.getElementById('content').innerText + '\n\n',
                                    lang: 'en'
                            }]
                    };

                    const launchParams = await getImmersiveReaderLaunchParamsAsync();
                    const token = launchParams.token;
                    const subdomain = launchParams.subdomain;

                    ImmersiveReader.launchAsync(token, subdomain, content);
            }
    ```

3. Nuestra aplicación web ya está lista. Inicie la aplicación; para ello, ejecute:

    ```bash
    npm start
    ```

4. Abra el explorador web y vaya a _http://localhost:3000_ . Debería ver el contenido anterior en la página. Haga clic en el botón **Lector inmersivo** para iniciar el Lector inmersivo con su contenido.

## <a name="specify-the-language-of-your-content"></a>Especificar el idioma del contenido

El Lector inmersivo es compatible con numerosos lenguajes diferentes. Para especificar el idioma del contenido, puede seguir estos pasos.

1. Abra _views\index.pug_ y agregue el código siguiente bajo la etiqueta `p(id=content)` que agregó en el paso anterior. Este código agrega contenido en español a la página.

    ```pug
    p(id='content-spanish') El estudio de las formas terrestres de la Tierra se llama geografía física. Los accidentes geográficos pueden ser montañas y valles. También pueden ser glaciares, lagos o ríos.
    ```

2. En el código JavaScript, agregue lo siguiente encima de la llamada a `ImmersiveReader.launchAsync`. Este código pasa el contenido en español al Lector inmersivo.

    ```pug
    content.chunks.push({
      content: document.getElementById('content-spanish').innerText + '\n\n',
      lang: 'es'
    });
    ```

3. Vaya de nuevo a _http://localhost:3000_ . Debería ver el texto en español en la página y, al hacer clic en **Lector inmersivo**, se mostrará en el Lector inmersivo también.

## <a name="specify-the-language-of-the-immersive-reader-interface"></a>Especificar el idioma de la interfaz del Lector inmersivo

De forma predeterminada, el idioma de la interfaz del Lector inmersivo coincide con la configuración de idioma del explorador. También puede especificar el idioma de la interfaz del Lector inmersivo con el código siguiente.

1. En _views\index.pug_, reemplace la llamada a `ImmersiveReader.launchAsync(token, subdomain, content)` por el código siguiente.

    ```javascript
    const options = {
        uiLang: 'fr',
    }
    ImmersiveReader.launchAsync(token, subdomain, content, options);
    ```

2. Vaya a _http://localhost:3000_ . Al iniciar el Lector inmersivo, se mostrará la interfaz en francés.

## <a name="launch-the-immersive-reader-with-math-content"></a>Iniciar el Lector inmersivo con contenido de matemáticas

Puede incluir contenido de matemáticas en el Lector inmersivo mediante [MathML](https://developer.mozilla.org/en-US/docs/Web/MathML).

1. Modifique _views\index.pug_ para incluir el código siguiente encima de la llamada a `ImmersiveReader.launchAsync`:

    ```javascript
    const mathML = '<math xmlns="https://www.w3.org/1998/Math/MathML" display="block"> \
      <munderover> \
        <mo>∫</mo> \
        <mn>0</mn> \
        <mn>1</mn> \
      </munderover> \
      <mrow> \
        <msup> \
          <mi>x</mi> \
          <mn>2</mn> \
        </msup> \
        <mo>ⅆ</mo> \
        <mi>x</mi> \
      </mrow> \
    </math>';

    content.chunks.push({
      content: mathML,
      mimeType: 'application/mathml+xml'
    });
    ```

2. Vaya a _http://localhost:3000_ . Al iniciar el Lector inmersivo y desplazarse hasta la parte inferior, verá la fórmula matemática.

## <a name="next-steps"></a>Pasos siguientes

* Explorar el [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk) y agregar la [Referencia del SDK del Lector inmersivo](./reference.md)
* Ver muestras de código en [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/advanced-csharp)
