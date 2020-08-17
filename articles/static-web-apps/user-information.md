---
title: Acceso a la información del usuario en Azure Static Web Apps
description: Obtenga información sobre cómo leer los datos de usuario devueltos por el proveedor de autorización.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: cshoe
ms.custom: devx-track-javascript
ms.openlocfilehash: 7e1f56fc4601b271bf4a0718a944741016509ce4
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87430527"
---
# <a name="accessing-user-information-in-azure-static-web-apps-preview"></a>Acceso a la información del usuario en la versión preliminar de Azure Static Web Apps

Azure Static Web Apps proporciona información de usuario relacionada con la autenticación a través de un [punto de conexión de acceso directo](#direct-access-endpoint) y [funciones de API](#api-functions).

Muchas interfaces de usuario dependen en gran medida de los datos de autenticación del usuario. El punto de conexión de acceso directo es una API de utilidad que expone información de usuario sin necesidad de implementar una función personalizada. Además de ser práctico, el punto de conexión de acceso directo no está sujeto a retrasos por arranque en frío asociados a las arquitecturas sin servidor.

## <a name="client-principal-data"></a>Datos de entidad de seguridad del cliente

El objeto de datos de la entidad de seguridad de cliente expone información de identificación del usuario a la aplicación. Las siguientes propiedades están incluidas en el objeto de entidad de seguridad de cliente:

| Propiedad  | Descripción |
|-----------|---------|
| `identityProvider` | Nombre del [proveedor de identidades](authentication-authorization.md). |
| `userId` | Identificador único específico de Azure Static Web Apps para el usuario. <ul><li>El valor es único para cada aplicación. Por ejemplo, el mismo usuario devuelve un valor `userId` diferente en un recurso de Static Web Apps diferente.<li>El valor se conserva durante la duración de un usuario. Si elimina y vuelve a agregar el mismo usuario a la aplicación, se genera un nuevo `userId`.</ul>|
| `userDetails` | Nombre de usuario o dirección de correo electrónico del usuario. Algunos proveedores devuelven la [dirección de correo electrónico del usuario](authentication-authorization.md), mientras que otros envían el [identificador de usuario](authentication-authorization.md). |
| `userRoles`     | Matriz de los [roles asignados del usuario](authentication-authorization.md). |

El ejemplo siguiente es una muestra de un objeto de entidad de seguridad de cliente:

```json
{
  "identityProvider": "facebook",
  "userId": "d75b260a64504067bfc5b2905e3b8182",
  "userDetails": "user@example.com",
  "userRoles": [ "anonymous", "authenticated" ]
}
```

## <a name="direct-access-endpoint"></a>Punto de conexión de acceso directo

Puede enviar una solicitud `GET` a la ruta `/.auth/me` y recibir acceso directo a los datos de la entidad de seguridad del cliente. Cuando el estado de la vista dependa de los datos de autorización, use este método para obtener el mejor rendimiento.

En el caso de los usuarios que han iniciado sesión, la respuesta contiene un objeto JSON de entidad de seguridad de cliente. Las solicitudes de los usuarios sin autenticar devuelven `null`.

Con la API [Fetch](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch)<sup>1</sup>, puede tener acceso a los datos de la entidad de seguridad del cliente con la siguiente sintaxis.

```javascript
async function getUserInfo() {
  const response = await fetch("/.auth/me");
  const payload = await response.json();
  const { clientPrincipal } = payload;
  return clientPrincipal;
}

console.log(getUserInfo());
```

## <a name="api-functions"></a>Funciones de API

Los datos de la entidad de seguridad del cliente se pasan a las funciones de API en el encabezado de solicitud `x-ms-client-principal`. Los datos de la entidad de seguridad del cliente se envían como una cadena con codificación [Base64](https://www.wikipedia.org/wiki/Base64) que contiene un objeto JSON serializado.

En la función de ejemplo siguiente se muestra cómo leer y devolver la información del usuario.

```javascript
module.exports = async function (context, req) {
  const header = req.headers["x-ms-client-principal"];
  const encoded = Buffer.from(header, "base64");
  const decoded = encoded.toString("ascii");

  context.res = {
    body: {
      clientPrincipal: JSON.parse(decoded)
    }
  };
};
```

Suponiendo que la función anterior se denomine `user`, puede usar la API [Fetch](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch)<sup>1</sup> del explorador para tener acceso a la respuesta de la API mediante la siguiente sintaxis.

```javascript
async function getUser() {
  const response = await fetch("/api/user");
  const payload = await response.json();
  const { clientPrincipal } = payload;
  return clientPrincipal;
}

console.log(getUser());
```

<sup>1</sup> La API [Fetch](https://caniuse.com/#feat=fetch) y el operador [Await](https://caniuse.com/#feat=mdn-javascript_operators_await) no son compatibles con Internet Explorer.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de aplicaciones](application-settings.md)
