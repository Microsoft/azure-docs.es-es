---
title: 'Tutorial de la aplicación web: escritura de aplicación web'
description: En este tutorial se explica un ejemplo para implementar una aplicación web sencilla. En esta sección del tutorial se explica cómo escribir la aplicación web.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 01/03/2020
ms.openlocfilehash: bb9c206a17a11f0cf710ac4ee3ac7ccae0fa9eb9
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87848033"
---
# <a name="write-azure-web-application-to-read-fhir-data"></a>Escritura de una aplicación web de Azure para leer datos de FHIR
Ahora que puede conectarse al servidor de FHIR y publicar datos, está preparado para escribir una aplicación web que leerá datos de FHIR. En este paso final del tutorial, se explicarán la escritura y el acceso a la aplicación web.

## <a name="create-web-application"></a>Creación de una aplicación web
En Azure, seleccione **Crear un recurso** y, luego, **Aplicación web**. Asegúrese de asignar un nombre a la aplicación web especificada en el URI de redirección de la aplicación cliente o vuelva y actualice el URI de redirección con el nuevo nombre. 

![Creación de una aplicación web](media/tutorial-web-app/create-web-app.png)

Una vez que la aplicación web esté disponible, haga clic en **Ir al recurso**. Seleccione **Editor de App Service (versión preliminar)** en Herramientas de desarrollo a la derecha y, a continuación, **Ir**. Al seleccionar Ir, se abrirá el Editor de App Service. Haga clic con el botón derecho en el espacio gris en *Explorar* y cree un archivo denominado **index.html**.

A continuación se muestra el código que puede escribir en **index.html**. Deberá actualizar los siguientes elementos:
* **clientId**: actualícelo con el identificador de la aplicación cliente. Este identificador será el mismo identificador que extrajo al recuperar el token.
* **authority**: actualícelo con el identificador de inquilino de Azure AD.
* **FHIRendpoint**: actualice FHIRendpoint para que tenga el nombre del servicio de FHIR.
* **scopes**: actualícelo para que refleje la dirección URL completa de la audiencia.

``` HTML

<!DOCTYPE html>
<html>

<head>
    <title>FHIR Patient browser sample app</title>
    <script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.0/js/msal.js"></script>
</head>

<body>
    <div class="leftContainer">
        <p id="WelcomeMessage">Welcome to the FHIR Patient browsing sample Application</p>
        <button id="SignIn" onclick="signIn()">Sign In</button>
    </div>

    <div id="patientTable">
    </div>

    <script>
        var msalConfig = {
            auth: {
                clientId: '<CLIENT-ID>',
                authority: "https://login.microsoftonline.com/<AZURE-AD-TENANT-ID>"
            },
            cache: {
                cacheLocation: "localStorage",
                storeAuthStateInCookie: true
            }
        }

        var FHIRConfig = {
            FHIRendpoint: "https://<FHIR-SERVER-NAME>.azurehealthcareapis.com"
        }
        var requestObj = {
            scopes: ["https://<FHIR-SERVER-NAME>.azurehealthcareapis.com/user_impersonation"]
        }

        function authRedirectCallBack(error, response) {
            if (error) {
                console.log(error);
            } else {
                if (response.tokenType === "access_token") {
                    callFHIRServer(FHIRConfig.FHIRendpoint + '/Patient', 'GET', null, response.accessToken, FHIRCallback);
                }
            }
        }

        var myMSALObj = new Msal.UserAgentApplication(msalConfig);
        myMSALObj.handleRedirectCallback(authRedirectCallBack);

        function signIn() {
            myMSALObj.loginPopup(requestObj).then(function (loginResponse) {
                showWelcomeMessage();
                acquireTokenPopupAndCallFHIRServer();
            }).catch(function (error) {
                console.log(error);
            })
        }

        function showWelcomeMessage() {
            var divWelcome = document.getElementById('WelcomeMessage');
            divWelcome.innerHTML = "Welcome " + myMSALObj.getAccount().userName + " to FHIR Patient Browsing App";
            var loginbutton = document.getElementById('SignIn');
            loginbutton.innerHTML = 'Sign Out';
            loginbutton.setAttribute('onclick', 'signOut()')
        }

        function signOut() {
            myMSALObj.logout();
        }

        function acquireTokenPopupAndCallFHIRServer() {
            myMSALObj.acquireTokenSilent(requestObj).then(function (tokenResponse) {
                callFHIRServer(FHIRConfig.FHIRendpoint + '/Patient', 'GET', null, tokenResponse.accessToken, FHIRCallback);
            }).catch(function (error) {
                console.log(error);
                if (requiresInteraction(error.errorCode)) {
                    myMSALObj.acquireTokenPopup(requestObj).then(function (tokenResponse) {
                        callFHIRServer(FHIRConfig.FHIRendpoint + '/Patient', 'GET', null, tokenResponse.accessToken, FHIRCallback);
                    }).catch(function (error) {
                        console.log(error);
                    })
                }
            });
        }

        function callFHIRServer(theUrl, method, message, accessToken, callBack) {
            var xmlHttp = new XMLHttpRequest();
            xmlHttp.onreadystatechange = function () {
                if (this.readyState == 4 && this.status == 200)
                    callBack(JSON.parse(this.responseText));
            }
            xmlHttp.open(method, theUrl, true);
            xmlHttp.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
            xmlHttp.setRequestHeader('Authorization', 'Bearer ' + accessToken);
            xmlHttp.send(message);
        }

        function FHIRCallback(data) {
            patientListHtml = '<ol>';
            data.entry.forEach(function(e) {
                patientListHtml += '<li>' + e.resource.name[0].family + ', ' + e.resource.name[0].given + ' (' + e.resource.id + ')';
            });
            patientListHtml += '</ol>';
            document.getElementById("patientTable").innerHTML = patientListHtml;
        }
    </script>
</body>

</html>
```

Desde aquí, puede volver al recurso de la aplicación web y abrir la dirección URL que se encuentra en la página de información general. Inicie sesión para ver al paciente James Tiberious Kirk que ha creado anteriormente.

## <a name="next-steps"></a>Pasos siguientes
Ha implementado correctamente Azure API for FHIR, ha registrado una aplicación cliente pública, ha probado el acceso y ha creado una pequeña aplicación web. Consulte las características admitidas de Azure API for FHIR como paso siguiente.

>[!div class="nextstepaction"]
>[Características admitidas](fhir-features-supported.md)





