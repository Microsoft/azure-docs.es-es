---
title: Rutas en Azure Static Web Apps
description: Obtenga información acerca de las reglas de enrutamiento del back-end y cómo proteger las rutas con roles.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: e6653f8f26f90b6ea7f911efab40ec7a3e0c2a60
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906783"
---
# <a name="routes-in-azure-static-web-apps-preview"></a>Rutas en la versión preliminar de Azure Static Web Apps

El enrutamiento en Azure Static Web Apps define las reglas de enrutamiento del back-end y el comportamiento de autorización del contenido estático y las API<sup>1</sup>. Se definen como una matriz de reglas en el archivo _routes.json_.

- El archivo _routes.json_ debe encontrarse en la raíz de la carpeta de artefactos de compilación de la aplicación.
- Las reglas se ejecutan en el orden en que aparecen en la matriz `routes`.
- La evaluación de la regla se detiene en la primera coincidencia. Las reglas de enrutamiento no están conectadas entre sí.
- Los roles se definen en el archivo _routes.json_ y los usuarios se asocian a los roles mediante [invitaciones](authentication-authorization.md).
- Tiene control total sobre los nombres de los roles.

El tema sobre el enrutamiento se superpone significativamente a los conceptos de autenticación y autorización. Asegúrese de leer la guía de [autenticación y autorización](authentication-authorization.md) junto con este artículo.

Consulte el [archivo de ruta de ejemplo](#example-route-file) para ver los detalles.

## <a name="location"></a>Location

El archivo _routes.json_ debe encontrarse en la raíz de la carpeta de artefactos de compilación de la aplicación. Si la aplicación web incluye un paso de compilación que copia los archivos compilados de una carpeta específica a la carpeta de artefactos de compilación, el archivo _routes.json_ debe encontrarse en esa carpeta específica.

En la tabla siguiente se muestra la ubicación correcta a fin de colocar el archivo _routes.json_ para varios marcos y bibliotecas JavaScript de front-end.

|Marco o biblioteca | Location  |
|---------|----------|
| Angular | _assets_   |
| React   | _public_  |
| Svelte  | _public_   |
| Vue     | _public_ |
| Blazor  | _wwwroot_ |

La tabla anterior solo es representativa de algunos marcos y bibliotecas compatibles con Azure Static Web Apps. Consulte [Configuración de bibliotecas y marcos de front-end](./front-end-frameworks.md) para obtener más información.

## <a name="defining-routes"></a>Definición de rutas

Las rutas se definen en el archivo _routes.json_ como una matriz de reglas de ruta en la propiedad `routes`. Cada una se compone de un patrón de ruta, junto con una o varias de las propiedades de regla opcionales. Consulte el [archivo de ruta de ejemplo](#example-route-file) para ver ejemplos de uso.

| Propiedad de regla  | Obligatorio | Valor predeterminado | Comentario                                                      |
| -------------- | -------- | ------------- | ------------------------------------------------------------ |
| `route`        | Sí      | N/D          | Patrón de ruta solicitado por el autor de la llamada.<ul><li>Los [caracteres comodín](#wildcards) se admiten al final de las rutas. Por ejemplo, la ruta _admin/\*_ coincide con cualquier ruta que se encuentre en la ruta de acceso _admin_.<li>El archivo predeterminado de una ruta es _index.html_.</ul>|
| `serve`        | No       | N/D          | Define el archivo o la ruta de acceso que devuelve la solicitud. La ruta de acceso y el nombre del archivo pueden ser distintos de la ruta de acceso solicitada. Si no se define un valor `serve`, se usa la ruta de acceso solicitada. No se admiten parámetros de cadena de consulta; los valores `serve` deben apuntar a archivos reales.  |
| `allowedRoles` | No       | anonymous     | Matriz de nombres de roles. <ul><li>Los caracteres válidos incluyen `a-z`, `A-Z`, `0-9` y `_`.<li>El rol integrado `anonymous` se aplica a todos los usuarios sin autenticar.<li>El rol integrado `authenticated` se aplica a todos los usuarios que han iniciado sesión.<li>Los usuarios deben pertenecer al menos a un rol.<li>Los roles se evalúan según el operador _OR_. Si un usuario tiene cualquiera de los roles enumerados, se le concede acceso.<li>Los usuarios individuales se asocian a los roles mediante [invitaciones](authentication-authorization.md).</ul> |
| `statusCode`   | No       | 200           | Respuesta de [código de estado HTTP](https://wikipedia.org/wiki/List_of_HTTP_status_codes) de la solicitud. |

## <a name="securing-routes-with-roles"></a>Protección de rutas mediante roles

Las rutas están protegidas mediante la adición de uno o varios nombres de rol a la matriz `allowedRoles` de una regla. Consulte el [archivo de ruta de ejemplo](#example-route-file) para ver ejemplos de uso.

De forma predeterminada, todos los usuarios pertenecen al rol de `anonymous` integrado, y todos los usuarios que han iniciado sesión son miembros del rol `authenticated`. Por ejemplo, para restringir una ruta solo a los usuarios autenticados, agregue el rol `authenticated` integrado a la matriz `allowedRoles`.

```json
{
  "route": "/profile",
  "allowedRoles": ["authenticated"]
}
```

Puede crear nuevos roles según sea necesario en la matriz `allowedRoles`. Para restringir una ruta a solo los administradores, puede definir un rol `administrator` en la matriz `allowedRoles`.

```json
{
  "route": "/admin",
  "allowedRoles": ["administrator"]
}
```

- Tiene control total sobre los nombres de rol, ya que no hay ninguna lista maestra que los roles deban seguir.
- Los usuarios individuales se asocian a los roles a través de [invitaciones](authentication-authorization.md).

## <a name="wildcards"></a>Caracteres comodín

Las reglas de caracteres comodín coinciden con todas las solicitudes de un patrón de ruta determinado. Si define un valor `serve` en la regla, el archivo o la ruta de acceso mencionados se devuelven como respuesta.

Por ejemplo, para implementar las rutas de una aplicación de calendario, puede asignar todas las direcciones URL que se encuentran en la ruta _calendar_ para que proporcionen un único archivo.

```json
{
  "route": "/calendar/*",
  "serve": "/calendar.html"
}
```

El archivo _calendar.html_ puede usar el enrutamiento del lado cliente para generar una vista diferente para las variantes de las direcciones URL, como `/calendar/january/1`, `/calendar/2020` y `/calendar/overview`.

También puede proteger las rutas con caracteres comodín. En el ejemplo siguiente, cualquier archivo solicitado en la ruta de acceso _admin_ requiere de un usuario autenticado que sea miembro del rol de _administrador_.

```json
{
  "route": "/admin/*",
  "allowedRoles": ["administrator"]
}
```

> [!NOTE]
> Las solicitudes de archivos a los que hace referencia un archivo proporcionado en servidor solo se evalúan para las comprobaciones de autenticación. Por ejemplo, las solicitudes a un archivo CSS en una ruta de acceso con carácter comodín se proporcionan el archivo CSS, en lugar de lo que está definido como archivo `serve`. El archivo CSS se proporciona siempre que el usuario cumple los requisitos de autenticación necesarios.

## <a name="fallback-routes"></a>Rutas de reserva

Las aplicaciones de una sola página, tanto si usan marcos de JavaScript de front-end, bibliotecas o plataformas WebAssembly como Blazor, suelen depender del enrutamiento del lado cliente para la navegación por la aplicación web. Estas reglas de enrutamiento del lado cliente actualizan la ubicación de la ventana del explorador sin realizar solicitudes al servidor. Si actualiza la página o navega directamente a las ubicaciones generadas por las reglas de enrutamiento del lado cliente, se necesita una ruta de reserva del lado servidor para proporcionar la página HTML adecuada.

En el siguiente ejemplo se muestra una ruta de reserva común:

```json
{
  "routes": [
    {
      "route": "/*",
      "serve": "/index.html",
      "statusCode": 200
    }
  ]
}
```

La ruta de reserva debe aparecer en último lugar en la lista de reglas de enrutamiento, ya que se encarga de todas las solicitudes que no hayan detectado las reglas definidas anteriormente.

## <a name="redirects"></a>Redireccionamientos

Puede usar los códigos de estado HTTP [301](https://en.wikipedia.org/wiki/HTTP_301) y [302](https://en.wikipedia.org/wiki/HTTP_302) para redirigir las solicitudes de una ruta a otra.

Por ejemplo, la siguiente regla crea un redireccionamiento 301 de _old-page.html_ a _new-page.html_.

```json
{
  "route": "/old-page.html",
  "serve": "/new-page.html",
  "statusCode": 301
}
```

Los redireccionamientos también funcionan con rutas que no definen a archivos diferentes.

```json
{
  "route": "/about-us",
  "serve": "/about",
  "statusCode": 301
}
```

## <a name="custom-error-pages"></a>Páginas de error personalizadas

Los usuarios pueden encontrar varias situaciones en las que se puede producir un error. Mediante el uso de la matriz `platformErrorOverrides`, puede proporcionar una experiencia personalizada en respuesta a estos errores. Consulte el [archivo de ruta de ejemplo](#example-route-file) para la selección de ubicación de la matriz en el archivo _routes.json_.

> [!NOTE]
> Cuando una solicitud la convierte en el nivel de invalidaciones de plataforma, las reglas de ruta no vuelven a ejecutarse.

En esta tabla se muestran las anulaciones de errores de plataforma que hay disponibles:

| Tipo de error  | Código de estado HTTP | Descripción |
|---------|---------|---------|
| `NotFound` | 404  | No se encuentra una página en el servidor. |
| `Unauthenticated` | 401 | El usuario no ha iniciado sesión con un [proveedor de autenticación](authentication-authorization.md). |
| `Unauthorized_InsufficientUserInformation` | 401 | La cuenta del usuario en el proveedor de autenticación no está configurada para exponer los datos necesarios. Este error puede producirse en situaciones en las que la aplicación pide al proveedor de autenticación la dirección de correo electrónico del usuario, pero el usuario decidió restringir el acceso a esta. |
| `Unauthorized_InvalidInvitationLink` | 401 | Una invitación expiró o el usuario usó un vínculo de invitación generado para otro destinatario.  |
| `Unauthorized_MissingRoles` | 401 | El usuario no es miembro del rol requerido. |
| `Unauthorized_TooManyUsers` | 401 | El sitio ha alcanzado el número máximo de usuarios y el servidor está limitando las adiciones. Este error se expone al cliente porque no hay ningún límite en el número de [invitaciones](authentication-authorization.md) que puede generar, y es posible que algunos usuarios nunca acepten su invitación.|
| `Unauthorized_Unknown` | 401 | Problema desconocido al intentar autenticar al usuario. Una causa de este error puede ser que el usuario no se reconoce, ya que no dio su consentimiento a la aplicación.|

## <a name="custom-mime-types"></a>Tipos MIME personalizados

El objeto `mimeTypes`, que se muestra en el mismo nivel que la matriz `routes`, permite asociar [tipos MIME](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types) con extensiones de archivo.

```json
{
    "routes": [],
    "mimeTypes": {
        "custom": "text/html"
    }
}
```

En el ejemplo anterior, todos los archivos con la extensión `.custom` se sirven con el tipo MIME `text/html`.

Las siguientes consideraciones son importantes cuando se trabaja con tipos MIME:

- Las claves no pueden ser nulas, estar vacías ni tener más de 50 caracteres.
- Los valores no pueden ser nulos, estar vacíos ni tener más de 1000 caracteres.

> [!NOTE]
> Static Web Apps entiende las aplicaciones de Blazor y los tipos MIME previstos para los archivos WASM y DLL; no es necesario agregar asignaciones para ellos.

## <a name="default-headers"></a>Encabezados predeterminados

El objeto `defaultHeaders`, que se muestra en el mismo nivel que la matriz `routes`, permite agregar, modificar o quitar [encabezados de respuesta](https://developer.mozilla.org/docs/Web/HTTP/Headers).

Al proporcionar un valor para un encabezado, se agrega o se modifica el encabezado. Al proporcionar un valor vacío, se quita el encabezado y no se sirve al cliente.

```json
{
    "routes": [],
    "defaultHeaders": {
      "content-security-policy": "default-src https: 'unsafe-eval' 'unsafe-inline'; object-src 'none'",
      "cache-control": "must-revalidate, max-age=6000",
      "x-dns-prefetch-control": ""
    }
}
```

En el ejemplo anterior, se agrega un nuevo encabezado `content-security-policy`, el objeto `cache-control` modifica el valor predeterminado del servidor y se quita el encabezado `x-dns-prefectch-control`.

Las siguientes consideraciones son importantes cuando se trabaja con encabezados:

- Las claves no puede ser nulas ni estar vacías.
- Los valores nulos o vacíos quitan un encabezado del procesamiento.
- Las claves o los valores no pueden superar los 8000 caracteres.
- Los encabezados definidos se sirven con todas las solicitudes.
- Los encabezados definidos en _routes.json_ solo se aplican al contenido estático. Puede personalizar los encabezados de respuesta de un punto de conexión de API en el código de la función.

## <a name="example-route-file"></a>Archivo de ruta de ejemplo

En el ejemplo siguiente se muestra cómo crear reglas de ruta para las API y el contenido estático en un archivo _routes.json_. Algunas rutas usan la [carpeta del sistema _/.auth_](authentication-authorization.md) que accede a los puntos de conexión relacionados con la autenticación.

```json
{
  "routes": [
    {
      "route": "/profile",
      "allowedRoles": ["authenticated"]
    },
    {
      "route": "/admin/*",
      "allowedRoles": ["administrator"]
    },
    {
      "route": "/api/admin",
      "allowedRoles": ["administrator"]
    },
    {
      "route": "/customers/contoso",
      "allowedRoles": ["administrator", "customers_contoso"]
    },
    {
      "route": "/login",
      "serve": "/.auth/login/github"
    },
    {
      "route": "/.auth/login/twitter",
      "statusCode": "404"
    },
    {
      "route": "/logout",
      "serve": "/.auth/logout"
    },
    {
      "route": "/calendar/*",
      "serve": "/calendar.html"
    },
    {
      "route": "/specials",
      "serve": "/deals",
      "statusCode": 301
    }
  ],
  "platformErrorOverrides": [
    {
      "errorType": "NotFound",
      "serve": "/custom-404.html"
    },
    {
      "errorType": "Unauthenticated",
      "statusCode": "302",
      "serve": "/login"
    }
  ],
  "defaultHeaders": {
    "content-security-policy": "default-src https: 'unsafe-eval' 'unsafe-inline'; object-src 'none'"
  },
  "mimeTypes": {
      "custom": "text/html"
  }
}
```

En los siguientes ejemplos se describe lo que ocurre cuando una solicitud coincide con una regla.

| Enviar una solicitud a... | Hace que... |
|--|--|--|
| _/profile_ | Los usuarios autenticados reciban el archivo _/profile/index.html_. Los usuarios sin autenticar se redirigen a _/login_. |
| _/admin/reports_ | Los usuarios autenticados en el rol _Administradores_ reciban el archivo _/admin/reports/index.html_. Los usuarios autenticados que no tienen el rol _Administradores_ reciben un error 401<sup>2</sup>. Los usuarios sin autenticar se redirigen a _/login_. |
| _/api/admin_ | Las solicitudes de los usuarios autenticados en el rol _Administradores_ se envíen a la API. Los usuarios autenticados que no tienen el rol _Administradores_ y los que no se han autenticado reciben un error 401. |
| _/customers/contoso_ | Los usuarios autenticados que pertenecen a los roles _Administradores_ o _customers\_contoso_ reciben el archivo _/customers/contoso/index.html_<sup>2</sup>. Los usuarios autenticados que no tienen los roles _Administradores_ o _customers\_contoso_ reciben un error 401. Los usuarios sin autenticar se redirigen a _/login_. |
| _/login_ | Se pide a los usuarios sin autenticar que lo hagan con GitHub. |
| _/.auth/login/twitter_ | La autorización con Twitter está deshabilitada. El servidor responde con un error 404. |
| _/logout_ | Se cierra la sesión de los usuarios con cualquier proveedor de autenticación. |
| _/calendar/2020/01_ | El explorador proporciona el archivo _/calendar.html_. |
| _/specials_ | El explorador se redirige a _/deals_. |
| _/unknown-folder_ | Se proporciona el archivo _/custom-404.html_. |
| Archivos con la extensión `.custom` | Se sirven con el tipo MIME `text/html`. |

Todas las respuestas incluyen los encabezados `content-security-policy` con un valor de `default-src https: 'unsafe-eval' 'unsafe-inline'; object-src 'none'`.

<sup>1</sup> Las reglas de enrutamiento para las funciones de API solo admiten [redirecciones](#redirects) y la [protección de rutas con roles](#securing-routes-with-roles).

<sup>2</sup> Puede proporcionar una página de error personalizada al definir una regla `Unauthorized_MissingRoles` en la matriz `platformErrorOverrides`.

## <a name="restrictions"></a>Restricciones

- El archivo _routes.json_ no puede tener un tamaño superior a 100 KB.
- El archivo _routes.json_ admite un máximo de 50 roles distintos.

Consulte el [artículo sobre cuotas](quotas.md) para conocer las restricciones y limitaciones generales.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de la autenticación y autorización](authentication-authorization.md)
