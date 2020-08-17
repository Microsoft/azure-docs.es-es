---
title: Protección de una aplicación de página única con el inicio de sesión de usuario
titleSuffix: Azure Maps
description: Configuración de una aplicación de página única que admite el inicio de sesión único de Azure AD con el SDK web de Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: devx-track-javascript
ms.openlocfilehash: 72fe4d897c9c202e0c4cd5861525093760036d26
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87285669"
---
# <a name="secure-a-single-page-application-with-user-sign-in"></a>Protección de una aplicación de página única con el inicio de sesión de usuario

La guía siguiente pertenece a una aplicación que se hospeda en un servidor de contenido o que tiene dependencias de servidor web mínimas. La aplicación proporciona recursos protegidos solo a usuarios de Azure AD. El objetivo del escenario es permitir que la aplicación web se autentique en Azure AD y llame a las API REST de Azure Maps en nombre del usuario.

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

## <a name="create-an-application-registration-in-azure-ad"></a>Creación de un registro de aplicación en Azure AD

Cree la aplicación web en Azure AD para que los usuarios inicien sesión. Esta aplicación web delegará el acceso de los usuarios a las API REST de Azure Maps.

1. En Azure Portal, en la lista de servicios de Azure, seleccione **Azure Active Directory** > **Registros de aplicaciones** > **Nuevo registro**.  

    > [!div class="mx-imgBorder"]
    > ![Registro de aplicaciones](./media/how-to-manage-authentication/app-registration.png)

2. Escriba un **nombre**, elija un **tipo de cuenta de soporte técnico** y proporcione un URI de redirección que represente la dirección URL en la que Azure AD emitirá el token y que sea la dirección URL donde se hospede el control de mapa. Para ver un ejemplo detallado, consulte [Ejemplos de Azure AD de Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/tree/master/src/ImplicitGrant). Después, seleccione **Register** (Registrar).  

3. Para asignar permisos de API delegados a Azure Maps, vaya a la aplicación. A continuación, en **Registros de aplicaciones**, seleccione **Permisos de API** > **Add a permission** (Agregar un permiso). En **API usadas en mi organización**, busque y seleccione **Azure Maps**.

    > [!div class="mx-imgBorder"]
    > ![Adición de permisos de API a la aplicación](./media/how-to-manage-authentication/app-permissions.png)

4. Active la casilla situada junto a **Access Azure Maps** (Acceder a Azure Maps) y seleccione **Agregar permisos**.

    > [!div class="mx-imgBorder"]
    > ![Selección de permisos de API de la aplicación](./media/how-to-manage-authentication/select-app-permissions.png)

5. Habilite `oauth2AllowImplicitFlow`. Para ello, en la sección **Manifiesto** del registro de la aplicación, establezca `oauth2AllowImplicitFlow` en `true`.

6. Copie el identificador de la aplicación de Azure AD y el identificador del inquilino de Azure AD del registro de la aplicación que se usarán en el SDK web. Agregue los detalles del registro de la aplicación de Azure AD y `x-ms-client-id` de la cuenta de Azure Map al SDK web.

    ```javascript
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js" />
        <script>
            var map = new atlas.Map("map", {
                center: [-122.33, 47.64],
                zoom: 12,
                language: "en-US",
                authOptions: {
                    authType: "aad",
                    clientId: "<insert>",  // azure map account client id
                    aadAppId: "<insert>",  // azure ad app registration id
                    aadTenant: "<insert>", // azure ad tenant id
                    aadInstance: "https://login.microsoftonline.com/"
                }
            });
        </script>   
    ```

7. Configure el control de acceso basado en roles de Azure para usuarios o grupos. Consulte las [secciones siguientes para habilitar RBAC](#grant-role-based-access-for-users-to-azure-maps).
   
[!INCLUDE [grant role access to users](./includes/grant-rbac-users.md)]

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el escenario de aplicación de página única:
> [!div class="nextstepaction"]
> [Aplicación de página única](https://docs.microsoft.com/azure/active-directory/develop/scenario-spa-overview)

Descubra las métricas de uso de API de la cuenta de Azure Maps:
> [!div class="nextstepaction"]
> [Visualización de métricas de uso](how-to-view-api-usage.md)

Explore ejemplos acerca de cómo integrar Azure AD con Azure Maps:
> [!div class="nextstepaction"]
> [Ejemplos de Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/tree/master/src/ImplicitGrant)
