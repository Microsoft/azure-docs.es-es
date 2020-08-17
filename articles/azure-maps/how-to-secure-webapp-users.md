---
title: Protección de una aplicación web con el inicio de sesión único interactivo
titleSuffix: Azure Maps
description: Configuración de una aplicación web que admite el inicio de sesión único de Azure AD con el SDK web de Azure Maps mediante el protocolo OpenID Connect.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: devx-track-javascript
ms.openlocfilehash: b86a8f726c039e3fa909cdc6f3f7b33b7c9c01ff
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87279753"
---
# <a name="secure-a-web-application-with-user-sign-in"></a>Protección de una aplicación web con el inicio de sesión de usuario

La siguiente guía pertenece a una aplicación que se hospeda en servidores web, mantiene varios escenarios empresariales y se implementa en servidores web. La aplicación tiene el requisito de proporcionar recursos protegidos solo a usuarios de Azure AD. El objetivo del escenario es permitir que la aplicación web se autentique en Azure AD y llame a las API REST de Azure Maps en nombre del usuario.

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

## <a name="create-an-application-registration-in-azure-ad"></a>Creación de un registro de aplicación en Azure AD

Debe crear la aplicación web en Azure AD para que los usuarios inicien sesión. Esta aplicación web delegará el acceso de los usuarios a las API REST de Azure Maps.

1. En Azure Portal, en la lista de servicios de Azure, seleccione **Azure Active Directory** > **Registros de aplicaciones** > **Nuevo registro**.  

    > [!div class="mx-imgBorder"]
    > ![Registro de aplicaciones](./media/how-to-manage-authentication/app-registration.png)

2. Escriba un **nombre**, elija un **tipo de cuenta de soporte técnico** y proporcione un URI de redirección que represente la dirección URL en la que Azure AD emitirá el token y que sea la dirección URL donde se hospede el control de mapa. Para más información, consulte el [Escenario: Aplicación web que permite iniciar sesión a los usuarios](https://docs.microsoft.com/azure/active-directory/develop/scenario-web-app-sign-user-overview) de Azure AD. Complete los pasos proporcionados en el escenario de Azure AD.  

3. Tras completar el registro de la aplicación, confirme que el inicio de sesión de la aplicación funciona para los usuarios. Una vez que el inicio de sesión funcione, a la aplicación se le puede conceder acceso delegado a las API REST de Azure Maps.
    
4.  Para asignar permisos de API delegados a Azure Maps, vaya a la aplicación. A continuación, seleccione **Permisos de API** > **Agregar un permiso**. En **API usadas en mi organización**, busque y seleccione **Azure Maps**.

    > [!div class="mx-imgBorder"]
    > ![Adición de permisos de API a la aplicación](./media/how-to-manage-authentication/app-permissions.png)

5. Active la casilla situada junto a **Access Azure Maps** (Acceder a Azure Maps) y seleccione **Agregar permisos**.

    > [!div class="mx-imgBorder"]
    > ![Selección de permisos de API de la aplicación](./media/how-to-manage-authentication/select-app-permissions.png)

6. Habilite la aplicación web para llamar a API REST de Azure Maps mediante la configuración del registro de la aplicación con un secreto de aplicación. Para obtener pasos detallados, consulte [Aplicación web que llama a las API web: registro de aplicación](https://docs.microsoft.com/azure/active-directory/develop/scenario-web-app-call-api-app-registration). Se requiere un secreto para autenticarse en Azure AD en nombre del usuario. El certificado o el secreto de registro de la aplicación debe almacenarse en un almacén seguro para que la aplicación web lo recupere para autenticarse en Azure AD. 
   
   * Si la aplicación ya tiene configurado un secreto y un registro de la aplicación de Azure AD, este paso se puede omitir.

> [!Tip]
> Si la aplicación se hospeda en un entorno de Azure, se recomienda el uso de [identidades administradas para recursos de Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) y una instancia de Azure Key Vault para acceder a los secretos mediante la [adquisición de un token de acceso](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token) para acceder a secretos o certificados de Azure Key Vault. Para conectarse a Azure Key Vault para recuperar secretos, consulte el [tutorial para conectarse mediante identidad administrada](https://docs.microsoft.com/azure/key-vault/general/tutorial-net-create-vault-azure-web-app).
   
7. Implemente un punto de conexión de token seguro para que el SDK web de Azure Maps acceda a un token. 
   
   * Para ver un controlador de token de ejemplo, consulte [Ejemplos de Azure AD de Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/blob/master/src/OpenIdConnect/AzureMapsOpenIdConnectv1/AzureMapsOpenIdConnect/Controllers/TokenController.cs). 
   * Para una implementación que no sea de AspNetCore o de otro tipo, consulte [Adquisición del token para la aplicación](https://docs.microsoft.com/azure/active-directory/develop/scenario-web-app-call-api-acquire-token) en la documentación de Azure AD.
   * El punto de conexión de token seguro es responsable de devolver un token de acceso para que el usuario autenticado y autorizado llame a las API REST de Azure Maps.

8. Configure el control de acceso basado en roles de Azure para usuarios o grupos. Consulte [Concesión de acceso basado en roles para usuarios](#grant-role-based-access-for-users-to-azure-maps).

9. Configure la página de la aplicación web con el SDK web de Azure Maps para acceder al punto de conexión de token seguro. 

```javascript
var map = new atlas.Map("map", {
        center: [-122.33, 47.64],
        zoom: 12,
        language: "en-US",
        authOptions: {
            authType: "anonymous",
            clientId: "<insert>",  // azure map account client id
            getToken: function (resolve, reject, map) {
                var xhttp = new XMLHttpRequest();
                xhttp.open("GET", "/api/token", true); // the url path maps to the token endpoint.
                xhttp.onreadystatechange = function () {
                    if (this.readyState === 4 && this.status === 200) {
                        resolve(this.responseText);
                    } else if (this.status !== 200) {
                        reject(this.responseText);
                    }
                };

                xhttp.send();
            }
        }
    });
    map.events.add("tokenacquired", function () {
        console.log("token acquired");
    });
    map.events.add("error", function (err) {
        console.log(JSON.stringify(err.error));
    });
```

[!INCLUDE [grant role access to users](./includes/grant-rbac-users.md)]

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el escenario de aplicación web:
> [!div class="nextstepaction"]
> [Escenario: Aplicación web que permite iniciar sesión a los usuarios](https://docs.microsoft.com/azure/active-directory/develop/scenario-web-app-sign-user-overview)

Descubra las métricas de uso de API de la cuenta de Azure Maps:
> [!div class="nextstepaction"]
> [Visualización de métricas de uso](how-to-view-api-usage.md)

Explore ejemplos acerca de cómo integrar Azure AD con Azure Maps:
> [!div class="nextstepaction"]
> [Ejemplos de aplicación web de Azure AD de Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/tree/master/src/OpenIdConnect)
