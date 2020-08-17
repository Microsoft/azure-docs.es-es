---
title: Protección de un dispositivo con restricciones de entrada con Azure AD y las API REST de Azure Maps
titleSuffix: Azure Maps
description: Configuración de una aplicación sin explorador que admita el inicio de sesión en Azure AD y llame a las API REST de Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: 91d73ad14cac77e4b00e90ec11791ef141436b7e
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87126758"
---
# <a name="secure-an-input-constrained-device-with-azure-ad-and-azure-maps-rest-apis"></a>Protección de un dispositivo con restricciones de entrada con Azure AD y las API REST de Azure Maps

En esta guía se describe cómo proteger aplicaciones o dispositivos públicos que no pueden almacenar secretos de forma segura ni aceptar entradas del explorador. Estos tipos de aplicaciones se encuentran en la categoría de IoT o Internet de las cosas. Algunos ejemplos de estas aplicaciones pueden incluir: Dispositivos de televisión inteligente o aplicaciones que emiten datos de sensores. 

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

## <a name="create-an-application-registration-in-azure-ad"></a>Creación de un registro de aplicación en Azure AD

> [!NOTE]
> * **Requisitos de lectura:** [Escenario: aplicación de escritorio que llama a las API web](https://docs.microsoft.com/azure/active-directory/develop/scenario-desktop-overview)
> * En el siguiente escenario se usa el flujo de código de dispositivo, que no necesita un explorador web para adquirir un token.

Cree la aplicación basada en el dispositivo en Azure AD para habilitar el inicio de sesión de Azure AD. A esta aplicación se le concederá acceso a las API REST de Azure Maps.

1. En Azure Portal, en la lista de servicios de Azure, seleccione **Azure Active Directory** > **Registros de aplicaciones** > **Nuevo registro**.  

    > [!div class="mx-imgBorder"]
    > ![Registro de aplicaciones](./media/how-to-manage-authentication/app-registration.png)

2. Escriba un **Nombre** y elija **Solo las cuentas de este directorio organizativo.** como **Tipo de cuenta admitido**. En **URI de redirección**, especifique **Cliente público/Nativo (móvil y escritorio)** y, a continuación, agregue `https://login.microsoftonline.com/common/oauth2/nativeclient` al valor. Para más información de Azure AD, consulte [Aplicación de escritorio que llama a las API web: registro de aplicación](https://docs.microsoft.com/azure/active-directory/develop/scenario-desktop-app-registration). A continuación, debe **Registrar** la aplicación.

    > [!div class="mx-imgBorder"]
    > ![Agregar detalles del registro de aplicación para el nombre y el identificador URI de redirección](./media/azure-maps-authentication/devicecode-app-registration.png)

3. Vaya a **Autenticación** y habilite **Tratar la aplicación como un cliente público**. Esto habilitará la autenticación de código de dispositivo con Azure AD.
    
    > [!div class="mx-imgBorder"]
    > ![Habilitación del registro de aplicación como cliente público](./media/azure-maps-authentication/devicecode-public-client.png)

4.  Para asignar permisos de API delegados a Azure Maps, vaya a la aplicación. A continuación, seleccione **Permisos de API** > **Agregar un permiso**. En **API usadas en mi organización**, busque y seleccione **Azure Maps**.

    > [!div class="mx-imgBorder"]
    > ![Adición de permisos de API a la aplicación](./media/how-to-manage-authentication/app-permissions.png)

5. Active la casilla situada junto a **Access Azure Maps** (Acceder a Azure Maps) y seleccione **Agregar permisos**.

    > [!div class="mx-imgBorder"]
    > ![Selección de permisos de API de la aplicación](./media/how-to-manage-authentication/select-app-permissions.png)

6. Configure el control de acceso basado en roles de Azure para usuarios o grupos. Consulte [Concesión a Azure Maps del control de acceso basado en roles](#grant-role-based-access-for-users-to-azure-maps).

7. Agregue el código del flujo de adquisición de tokens a la aplicación. Para obtener información detallada sobre la implementación, consulte [Flujo de código de dispositivo](https://docs.microsoft.com/azure/active-directory/develop/scenario-desktop-acquire-token#device-code-flow). Al adquirir los tokens, haga referencia al ámbito: `user_impersonation`, que se seleccionó en los pasos anteriores.

> [!Tip]
> Use la Biblioteca de autenticación de Microsoft (MSAL) para adquirir los tokens de acceso. Consulte las recomendaciones de [Aplicación de escritorio que llama a las API web: configuración del código](https://docs.microsoft.com/azure/active-directory/develop/scenario-desktop-app-configuration)

8. Cree la solicitud HTTP con el token adquirido desde Azure AD y envíe la solicitud con un cliente HTTP válido.

### <a name="sample-request"></a>Solicitud de ejemplo
El siguiente es un ejemplo de cuerpo de la solicitud para cargar una geovalla sencilla que se representa como una geometría circular mediante un punto central y un radio.

```http
POST /mapData/upload?api-version=1.0&dataFormat=geojson
Host: atlas.microsoft.com
x-ms-client-id: 30d7cc….9f55
Authorization: Bearer eyJ0e….HNIVN
```

 El cuerpo de la solicitud de ejemplo siguiente está en formato GeoJSON:
```json
{
    "type": "FeatureCollection",
    "features": [{
        "type": "Feature",
        "geometry": {
            "type": "Point",
            "coordinates": [-122.126986, 47.639754]
        },
        "properties": {
            "geometryId": "001",
            "radius": 500
        }
    }]
}
```

### <a name="sample-response"></a>Respuesta de ejemplo:

Encabezados:
```http
Location: https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0
Access-Control-Expose-Headers: Location
```

Cuerpo:
```json
{
  "operationId": "{operationId}",
  "status": "Succeeded",
  "created": "2020-01-02 1:02:03 AM +00:00",
  "resourceLocation": "https://atlas.microsoft.com/mapData/metadata/{resourceId}?api-version=1.0"
}
```

[!INCLUDE [grant role access to users](./includes/grant-rbac-users.md)]

## <a name="next-steps"></a>Pasos siguientes

Descubra las métricas de uso de API de la cuenta de Azure Maps:
> [!div class="nextstepaction"]
> [Visualización de métricas de uso](how-to-view-api-usage.md)
