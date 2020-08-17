---
title: Configuración de un dominio personalizado en Azure Static Web Apps
description: Aprenda a asignar un dominio personalizado a Azure Static Web Apps
services: static-web-apps
author: burkeholland
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: buhollan
ms.openlocfilehash: 61ec96a35fac6a033fe6c8b65cff156ba63e5e58
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87563354"
---
# <a name="setup-a-custom-domain-in-azure-static-web-apps-preview"></a>Configuración de un dominio personalizado en Azure Static Web Apps (versión preliminar)

De forma predeterminada, Azure Static Web Apps proporciona un nombre de dominio generado automáticamente. En este artículo se muestra cómo asignar un nombre de dominio personalizado a una aplicación de Azure Static Web Apps.

## <a name="prerequisites"></a>Requisitos previos

- Un nombre de dominio adquirido
- Acceso a las propiedades de configuración de DNS para el dominio

Al configurar los nombres de dominio, los registros "A" se usan para asignar dominios raíz (por ejemplo, `example.com`) a una dirección IP. Los dominios raíz se deben asignar directamente a una dirección IP, ya que la especificación DNS no permite la asignación de un dominio a otro.

## <a name="dns-configuration-options"></a>Opciones de configuración de DNS

Hay algunos tipos diferentes de configuraciones de DNS disponibles para una aplicación.

| Si desea                                 | Entonces                                                |
| -----------------------------------------------| --------------------------------------------------- |
| Asistencia de `www.example.com` o `blog.example.net`| [Asignar un registro CNAME](#map-a-cname-record)           |
| Se ha agregado compatibilidad con `example.com`.                          | [Configure un dominio raíz](#configure-a-root-domain) |
| Apuntar todos los subdominios a `www.example.com`      | [Asigne un dominio comodín](#map-a-wildcard-domain)            |

## <a name="map-a-cname-record"></a>Asignar un registro CNAME

Un registro CNAME asigna un dominio a otro. Puede usar un registro CNAME para asignar `www.example.com`, `blog.example.com` u otro subdominio al dominio generado automáticamente que proporciona Azure Static Web Apps.

1. Abra [Azure Portal](https://portal.azure.com) e inicie sesión con su cuenta de Azure.

1. Busque y seleccione **Static Web Apps**.

1. En la página _Static Web App_, seleccione el nombre de la aplicación.

1. Haga clic en **Dominios personalizados** en el menú.

1. En la ventana _Dominios personalizados_, copie la dirección URL en el campo **Valor**.

### <a name="configure-dns-provider"></a>Configuración del proveedor de DNS

1. Inicie sesión en el sitio web de su proveedor de dominios.

2. Busque la página de administración de registros DNS. Cada proveedor de dominios tiene su propia interfaz de registros DNS, así que consulte la documentación del proveedor. Busque áreas del sitio etiquetadas como **Nombre de dominio**, **DNS** o **Administración del servidor del nombres**.

3. A menudo, se puede encontrar la página de registros DNS al ver la información de la cuenta y al buscar un vínculo, como **Mis dominios**. Vaya a dicha página y, después, busque un vínculo que se llame algo similar a **Zone file**, **DNS Records** o **Advanced configuration**.

    La captura de pantalla siguiente es un ejemplo de página de registros DNS:

    :::image type="content" source="media/custom-domain/example-record-ui.png" alt-text="Configuración del proveedor de DNS de ejemplo":::

4. Cree un nuevo registro **CNAME** con los siguientes valores:

    | Configuración             | Value                     |
    | ------------------- | ------------------------- |
    | Tipo                | CNAME                     |
    | Host                | www                       |
    | Value               | Pegado desde el Portapapeles |
    | TTL (si corresponde) | Deje el valor predeterminado    |

5. Guarde los cambios en el proveedor de DNS.

### <a name="validate-cname"></a>Validación de CNAME

1. Vuelva a la ventana _Dominios personalizados_ en Azure Portal.

1. Escriba el dominio, incluida la parte `www` en la sección _Validar dominio personalizado_.

1. Haga clic en el botón **Validar**.

Ahora que el dominio personalizado está configurado, el proveedor de DNS puede tardar varias horas en propagar los cambios en todo el mundo. Para comprobar el estado de la propagación, puede ir a [dnspropagation.net](https://dnspropagation.net). Escriba el dominio personalizado, incluido `www`, seleccione CNAME en la lista desplegable y seleccione **Start**.

Si se han rellenado los cambios de DNS, el sitio web devuelve la dirección URL generada automáticamente de la aplicación de Static Web Apps (por ejemplo, _nombre-aleatorio-123456789c.azurestaticapps.net_).

## <a name="configure-a-root-domain"></a>Configuración de un dominio raíz

Los dominios raíz son el dominio menos cualquier subdominio, incluido `www`. Por ejemplo, el dominio raíz de `www.example.com` es `example.com`. Esto también se conoce como dominio "APEX".

Si bien la compatibilidad con el dominio raíz no está disponible durante la versión preliminar, puede ver la entrada de blog sobre la [configuración de dominios raíz en Azure Static Web Apps](https://burkeholland.github.io/posts/static-app-root-domain) para obtener más información sobre cómo configurar la compatibilidad con dominios raíz con una aplicación de Static Web Apps.

## <a name="map-a-wildcard-domain"></a>Asignar un dominio con caracteres comodín

A veces, desea que todo el tráfico que se envía a un subdominio se enrute a otro dominio. Un ejemplo común es asignar todo el tráfico del subdominio a `www.example.com`. De este modo, incluso si alguien escribe `w.example.com` en lugar de `www.example.com`, la solicitud se envía a `www.example.com`.

### <a name="configure-dns-provider"></a>Configuración del proveedor de DNS

1. Inicie sesión en el sitio web de su proveedor de dominios.

2. Busque la página de administración de registros DNS. Cada proveedor de dominios tiene su propia interfaz de registros DNS, así que consulte la documentación del proveedor. Busque áreas del sitio etiquetadas como **Nombre de dominio**, **DNS** o **Administración del servidor del nombres**.

3. A menudo, se puede encontrar la página de registros DNS al ver la información de la cuenta y al buscar un vínculo, como **Mis dominios**. Vaya a dicha página y, después, busque un vínculo que se llame algo similar a **Zone file**, **DNS Records** o **Advanced configuration**.

    La captura de pantalla siguiente es un ejemplo de página de registros DNS:

    :::image type="content" source="media/custom-domain/example-record-ui.png" alt-text="Configuración del proveedor de DNS de ejemplo":::

4. Cree un nuevo registro **CNAME** con los siguientes valores, y reemplace `www.example.com` por el nombre de dominio personalizado.

    | Configuración | Value                  |
    | ------- | ---------------------- |
    | Tipo    | CNAME                  |
    | Host    | \*                     |
    | Value   | www.example.com        |
    | TTL     | Deje el valor predeterminado |

5. Guarde los cambios en el proveedor de DNS.

Ahora que el dominio comodín está configurado, los cambios pueden tardar varias horas en propagarse por todo el mundo. Para comprobar el estado de la propagación, puede ir a [dnspropagation.net](https://dnspropagation.net). Escriba el dominio personalizado con un subdominio (excepto `www`), seleccione CNAME en la lista desplegable y seleccione **Start**.

Si se han rellenado los cambios de DNS, el sitio web devolverá el dominio personalizado configurado para la aplicación de Static Web Apps (por ejemplo, `www.example.com`).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de aplicaciones](application-settings.md)
