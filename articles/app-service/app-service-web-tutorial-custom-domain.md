---
title: 'Tutorial: Asignación de un nombre DNS personalizado existente'
description: Aprenda a agregar un nombre de dominio DNS personalizado (dominio personal) a una aplicación web, un back-end de una aplicación móvil o una aplicación de API en Azure App Service.
keywords: app service, azure app service, domain mapping, domain name, existing domain, hostname
ms.assetid: dc446e0e-0958-48ea-8d99-441d2b947a7c
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 08/25/2020
ms.custom: mvc, seodec18
ms.openlocfilehash: 4a2c65e2685dada6412adf8c8ad9c63f472b91e8
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962288"
---
# <a name="tutorial-map-an-existing-custom-dns-name-to-azure-app-service"></a>Tutorial: Asignación de un nombre DNS personalizado existente a Azure App Service

[Azure App Service](overview.md) proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático. En este tutorial se muestra cómo asignar un nombre DNS personalizado existente a Azure App Service.

![Navegación en el portal a la aplicación de Azure](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Asignar un subdominio (por ejemplo, `www.contoso.com`) mediante el uso de un registro CNAME
> * Asignar un dominio raíz (por ejemplo, `contoso.com`) mediante el uso de un registro D
> * Asignar un dominio con comodín (por ejemplo, `*.contoso.com`) mediante el uso de un registro CNAME
> * Redireccionamiento de una dirección URL predeterminada a un directorio personalizado
> * Automatizar la asignación de dominio con scripts

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial:

* [Cree una aplicación de App Service](./index.yml) o use alguna aplicación que haya creado para otro tutorial.
* Adquiera un nombre de dominio y asegúrese de que tiene acceso al registro de DNS de su proveedor de dominio (por ejemplo, GoDaddy).

  Por ejemplo, para agregar entradas DNS para `contoso.com` y `www.contoso.com`, debe poder configurar las opciones de DNS del dominio raíz de `contoso.com`.

  > [!NOTE]
  > Si no tiene un nombre de dominio, considere la posibilidad de [comprar un dominio mediante Azure Portal](manage-custom-dns-buy-domain.md).

## <a name="prepare-the-app"></a>Preparación de la aplicación

Para asignar un nombre DNS personalizado a una aplicación web, el [plan de App Service](https://azure.microsoft.com/pricing/details/app-service/) de dicha aplicación debe ser un nivel de pago (**Compartido**, **Básico**, **Estándar**, **Premium** o **Consumo** para Azure Functions). En este paso, asegúrese de que la aplicación de App Service se encuentra en el plan de tarifa compatible.

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Abra [Azure Portal](https://portal.azure.com) e inicie sesión con su cuenta de Azure.

### <a name="select-the-app-in-the-azure-portal"></a>Selección de la aplicación en Azure Portal

Busque y seleccione **App Services**.

![Selección de App Services](./media/app-service-web-tutorial-custom-domain/app-services.png)

En la página **App Services**, seleccione el nombre de la aplicación de Azure.

![Navegación en el portal a la aplicación de Azure](./media/app-service-web-tutorial-custom-domain/select-app.png)

Consulte la página de administración de la aplicación de App Service.  

<a name="checkpricing" aria-hidden="true"></a>

### <a name="check-the-pricing-tier"></a>Comprobar el plan de tarifa

En el panel de navegación izquierdo de la página de la aplicación, desplácese hasta la sección **Configuración** y seleccione **Escalar verticalmente (plan de App Service)** .

![Menú Escalar verticalmente](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

El nivel actual de la aplicación aparece resaltado con un cuadro azul. Asegúrese de que la aplicación web no está en el nivel **F1**. No se admiten DNS personalizados en el nivel **F1**. 

![Comprobar plan de tarifa](./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png)

Si el plan de App Service no es **F1**, cierre la página **Escalar verticalmente** y vaya directamente a [Asignar un registro CNAME](#cname).

<a name="scaleup" aria-hidden="true"></a>

### <a name="scale-up-the-app-service-plan"></a>Escalado vertical del plan de App Service

Seleccione cualquiera de los niveles no gratuitos (**D1**, **B1**, **B2**, **B3**, o cualquier nivel de la categoría **Producción**). Para ver opciones adicionales, haga clic en **Ver opciones adicionales**.

Haga clic en **Aplicar**.

![Comprobar plan de tarifa](./media/app-service-web-tutorial-custom-domain/choose-pricing-tier.png)

Cuando vea la siguiente notificación, significará que la operación de escalado se habrá completado.

![Confirmación de la operación de escalado](./media/app-service-web-tutorial-custom-domain/scale-notification.png)

<a name="cname" aria-hidden="true"></a>

## <a name="get-domain-verification-id"></a>Obtención del identificador de comprobación de dominio

Para agregar un dominio personalizado a la aplicación, tiene que confirmar la propiedad del dominio agregando un identificador de comprobación como un registro TXT con el proveedor de dominio. En el panel de navegación izquierdo de la página de la aplicación, haga clic en **Dominios personalizados**. Copie el valor de **Custom Domain Verification ID** (Id. de verificación del dominio personalizado) en la página **Custom Domains** (Dominios personalizados) para realizar el siguiente paso.

![Obtención del identificador de verificación del dominio personalizado:](./media/app-service-web-tutorial-custom-domain/get-custom-domain-verification-id.png)

> [!WARNING]
> Agregar identificadores de verificación de dominio al dominio personalizado puede impedir las entradas DNS pendientes y las adquisiciones de subdominios. Para más información sobre esta amenaza común de alta gravedad, consulte [Adquisición de subdominios](../security/fundamentals/subdomain-takeover.md).

## <a name="map-your-domain"></a>Asignación del dominio

Puede usar un **registro CNAME** o un **registro D** para asignar un nombre DNS personalizado a App Service. Siga los pasos correspondientes:

- [Asignar un registro CNAME](#map-a-cname-record)
- [Asignar un registro D](#map-an-a-record)
- [Asignar un dominio con comodín (con un registro CNAME)](#map-a-wildcard-domain)

> [!NOTE]
> Debe utilizar registros CNAME para todos los nombres de DNS personalizados, excepto los dominios raíz (por ejemplo, `contoso.com`). Para los dominios raíz, utilice registros A.

### <a name="map-a-cname-record"></a>Asignar un registro CNAME

En el ejemplo del tutorial, agregue un registro CNAME para el subdominio `www` (por ejemplo, `www.contoso.com`).

Si tiene un subdominio distinto de `www`, reemplace `www` por el subdominio (por ejemplo, con `sub` si el dominio personalizado es `sub.constoso.com`).

#### <a name="access-dns-records-with-domain-provider"></a>Acceso a los registros DNS con el proveedor de dominios

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-cname-record"></a>Crear un registro CNAME

Asigne un subdominio al nombre de dominio predeterminado de la aplicación (`<app-name>.azurewebsites.net`, donde `<app-name>` es el nombre de la aplicación). Para crear una asignación CNAME para el subdominio `www`, cree dos registros:

| Tipo de registro | Host | Value | Comentarios |
| - | - | - |
| CNAME | `www` | `<app-name>.azurewebsites.net` | La propia asignación de dominio. |
| TXT | `asuid.www` | [El identificador de comprobación que obtuvo anteriormente](#get-domain-verification-id). | App Service accede al registro TXT `asuid.<subdomain>` para comprobar la propiedad del dominio personalizado. |

Después de agregar los registros CNAME y TXT, la página de registros DNS es como la del ejemplo siguiente:

![Navegación en el portal a la aplicación de Azure](./media/app-service-web-tutorial-custom-domain/cname-record.png)

#### <a name="enable-the-cname-record-mapping-in-azure"></a>Habilitación de la asignación de registros CNAME en Azure

1. En el panel de navegación izquierdo de la página de la aplicación en Azure Portal, seleccione **Dominios personalizados**.

    ![Menú Dominio personalizado](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

1. En la página **Dominios personalizados** de la aplicación, agregue el nombre DNS personalizado completo (`www.contoso.com`) a la lista.

1. Seleccione **Agregar dominio personalizado**.

    ![Agregar nombre de host](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

1. Escriba el nombre de dominio completo para el que ha agregado un registro CNAME, como `www.contoso.com`.

1. Seleccione **Validar**. Se abre la página **Agregar dominio personalizado**.

1. Asegúrese de que en **Tipo de registro de nombre de host** está seleccionado **CNAME (www\.example.com o cualquier subdominio)** . Seleccione **Agregar dominio personalizado**.

    ![Agregar nombre DNS a la aplicación](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname.png)

    El nuevo dominio personalizado puede tardar algo en reflejarse en la página **Dominios personalizados** de la aplicación. Intente actualizar el explorador para actualizar los datos.

    ![Registro CNAME agregado](./media/app-service-web-tutorial-custom-domain/cname-record-added.png)

    > [!NOTE]
    > Si un dominio personalizado contiene una etiqueta de advertencia, significa que aún no está enlazado a un certificado TLS/SSL y que todas las solicitudes HTTPS de un explorador que lleguen a dicho dominio personalizado recibirán un error o una advertencia, en función del explorador que se use. Para agregar un enlace TLS, consulte [Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service](configure-ssl-bindings.md).

    Si se olvidó de un paso o cometió un error tipográfico en alguna parte anteriormente, verá un error de comprobación en la parte inferior de la página.

    ![Error de comprobación](./media/app-service-web-tutorial-custom-domain/verification-error-cname.png)

<a name="a" aria-hidden="true"></a>

### <a name="map-an-a-record"></a>Asignar un registro D

En el ejemplo del tutorial, se agrega un registro D al dominio raíz (por ejemplo, `contoso.com`).

<a name="info"></a>

#### <a name="copy-the-apps-ip-address"></a>Copiar la dirección IP de la aplicación

Para asignar un registro D, se necesita la dirección IP externa de la aplicación. Dicha dirección IP se puede encontrar en la página **Dominios personalizados** de la aplicación en Azure Portal.

En el panel de navegación izquierdo de la página de la aplicación en Azure Portal, seleccione **Dominios personalizados**.

![Menú Dominio personalizado](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

En la página **Dominios personalizados**, copie la dirección IP de la aplicación.

![Navegación en el portal a la aplicación de Azure](./media/app-service-web-tutorial-custom-domain/mapping-information.png)

#### <a name="access-dns-records-with-domain-provider"></a>Acceso a los registros DNS con el proveedor de dominios

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-a-record"></a>Crear el registro D

Para asignar un registro D a una aplicación, normalmente al dominio raíz, cree dos registros:

| Tipo de registro | Host | Value | Comentarios |
| - | - | - |
| A | `@` | D www Dirección IP de [Copiar la dirección IP de la aplicación](#info) | La propia asignación de dominio (`@` normalmente representa el dominio raíz). |
| TXT | `asuid` | [El identificador de comprobación que obtuvo anteriormente](#get-domain-verification-id). | App Service accede al registro TXT `asuid.<subdomain>` para comprobar la propiedad del dominio personalizado. Para el dominio raíz, use `asuid`. |

> [!NOTE]
> Para agregar un subdominio (como `www.contoso.com`) con un registro de dirección en lugar de un [registro CNAME](#map-a-cname-record) recomendado, el registro de dirección y el registro TXT tendrán un aspecto similar al de la tabla siguiente:
>
> | Tipo de registro | Host | Value |
> | - | - | - |
> | A | `www` | D www Dirección IP de [Copiar la dirección IP de la aplicación](#info) |
> | TXT | `asuid.www` | `<app-name>.azurewebsites.net` |
>

Cuando se agregan los registros, la página de registros DNS es como la del ejemplo siguiente:

![Página de registros DNS](./media/app-service-web-tutorial-custom-domain/a-record.png)

<a name="enable-a" aria-hidden="true"></a>

#### <a name="enable-the-a-record-mapping-in-the-app"></a>Habilitar la asignación de registros A en la aplicación

De vuelta a la página **Dominios personalizados** de la aplicación en Azure Portal, agregue el nombre DNS personalizado completo (por ejemplo, `contoso.com`) a la lista.

1. Seleccione **Agregar dominio personalizado**.

    ![Agregar nombre de host](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

1. Escriba el nombre de dominio completo para el que ha configurado el registro A, como `contoso.com`. 

1. Seleccione **Validar**. Se abre la página **Agregar dominio personalizado**.

1. Asegúrese de que el **tipo de registro de nombre de host** esté establecido en el **registro D (ejemplo.com)** . Seleccione **Agregar dominio personalizado**.

    ![Agregar nombre DNS a la aplicación](./media/app-service-web-tutorial-custom-domain/validate-domain-name.png)

    El nuevo dominio personalizado puede tardar algo en reflejarse en la página **Dominios personalizados** de la aplicación. Intente actualizar el explorador para actualizar los datos.

    ![Registro D agregado](./media/app-service-web-tutorial-custom-domain/a-record-added.png)

    > [!NOTE]
    > Si un dominio personalizado contiene una etiqueta de advertencia, significa que aún no está enlazado a un certificado TLS/SSL y que todas las solicitudes HTTPS de un explorador que lleguen a dicho dominio personalizado recibirán un error o una advertencia, en función del explorador que se use. Para agregar un enlace TLS, consulte [Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service](configure-ssl-bindings.md).
    
    Si se olvidó de un paso o cometió un error tipográfico en alguna parte anteriormente, verá un error de comprobación en la parte inferior de la página.
    
    ![Error de comprobación](./media/app-service-web-tutorial-custom-domain/verification-error.png)
    
<a name="wildcard" aria-hidden="true"></a>

### <a name="map-a-wildcard-domain"></a>Asignar un dominio con caracteres comodín

En el ejemplo del tutorial, asigne un [nombre DNS con caracteres comodín](https://en.wikipedia.org/wiki/Wildcard_DNS_record) (por ejemplo, `*.contoso.com`) a la aplicación de App Service mediante la adición de un registro CNAME.

#### <a name="access-dns-records-with-domain-provider"></a>Acceso a los registros DNS con el proveedor de dominios

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-cname-record"></a>Crear un registro CNAME

Asigne un nombre con caracteres comodín `*` al nombre de dominio predeterminado de la aplicación (`<app-name>.azurewebsites.net`, donde `<app-name>` es el nombre de la aplicación). Para asignar el nombre con caracteres comodín, cree dos registros:

| Tipo de registro | Host | Value | Comentarios |
| - | - | - |
| CNAME | `*` | `<app-name>.azurewebsites.net` | La propia asignación de dominio. |
| TXT | `asuid` | [El identificador de comprobación que obtuvo anteriormente](#get-domain-verification-id). | App Service accede al registro TXT `asuid` para comprobar la propiedad del dominio personalizado. |

En el dominio `*.contoso.com` del ejemplo, el registro CNAME asignará el nombre `*` a `<app-name>.azurewebsites.net`.

Cuando se agrega CNAME, la página de registros DNS es como la del ejemplo siguiente:

![Navegación en el portal a la aplicación de Azure](./media/app-service-web-tutorial-custom-domain/cname-record-wildcard.png)

#### <a name="enable-the-cname-record-mapping-in-the-app"></a>Habilitación de la asignación del registro CNAME en la aplicación

Ahora puede agregar cualquier subdominio que coincida con el nombre con caracteres comodín en la aplicación (por ejemplo, `sub1.contoso.com`, `sub2.contoso.com` y `*.contoso.com` coinciden ambos con `*.contoso.com`).

1. En el panel de navegación izquierdo de la página de la aplicación en Azure Portal, seleccione **Dominios personalizados**.

    ![Menú Dominio personalizado](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

1. Seleccione **Agregar dominio personalizado**.

    ![Agregar nombre de host](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

1. Escriba un nombre de dominio completo que coincida con el dominio con caracteres comodín (por ejemplo, `sub1.contoso.com`) y, después, seleccione **Validar**.

    El botón **Agregar dominio personalizado** está activado.

1. Asegúrese de que en **Tipo de registro de nombre de host** está seleccionado **Registro CNAME (www\.example.com o cualquier subdominio)** . Seleccione **Agregar dominio personalizado**.

    ![Agregar nombre DNS a la aplicación](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname-wildcard.png)

    El nuevo dominio personalizado puede tardar algo en reflejarse en la página **Dominios personalizados** de la aplicación. Intente actualizar el explorador para actualizar los datos.

1. Vuelva a seleccionar el icono **+** de nuevo para agregar otro dominio personalizado que coincida con el dominio con caracteres comodín. Por ejemplo, agregue `sub2.contoso.com`.

    ![Registro CNAME agregado](./media/app-service-web-tutorial-custom-domain/cname-record-added-wildcard-2.png)

    > [!NOTE]
    > Si un dominio personalizado contiene una etiqueta de advertencia, significa que aún no está enlazado a un certificado TLS/SSL y que todas las solicitudes HTTPS de un explorador que lleguen a dicho dominio personalizado recibirán un error o una advertencia, en función del explorador que se use. Para agregar un enlace TLS, consulte [Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service](configure-ssl-bindings.md).
    
## <a name="test-in-browser"></a>Probar en el explorador

Desplácese a los nombres DNS que configuró anteriormente (por ejemplo, `contoso.com`, `www.contoso.com`, `sub1.contoso.com` y `sub2.contoso.com`).

![Navegación en el portal a la aplicación de Azure](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

## <a name="resolve-404-not-found"></a>Resolución de error 404 "No se encuentra"

Si recibe un error HTTP 404 (No se encuentra) al ir a la dirección URL de su dominio personalizado, compruebe que su dominio se resuelve en la dirección IP de la aplicación mediante <a href="https://www.whatsmydns.net/" target="_blank">WhatsmyDNS.net</a>. Si no es así, esto podría deberse a uno de los siguientes motivos:

- En el dominio personalizado configurado falta un registro D o un registro CNAME.
- El cliente del explorador ha almacenado en caché la dirección IP antigua del dominio. Borre la caché y pruebe la resolución DNS de nuevo. En un equipo Windows, borre la memoria caché con `ipconfig /flushdns`.

<a name="virtualdir" aria-hidden="true"></a>

## <a name="migrate-an-active-domain"></a>Migración de un dominio activo

Para migrar un sitio en vivo y su nombre de dominio DNS a App Service sin tiempo de inactividad, consulte [Migración de un nombre DNS activo a Azure App Service](manage-custom-dns-migrate-domain.md).

## <a name="redirect-to-a-custom-directory"></a>Redirección a un directorio personalizado

De forma predeterminada, App Service dirige las solicitudes web al directorio raíz del código de la aplicación. Sin embargo, algunos marcos web no se inician en el directorio raíz. Por ejemplo, [Laravel](https://laravel.com/) se inicia en el subdirectorio `public`. Para continuar con el ejemplo de DNS `contoso.com`, se podría acceder a una aplicación en `http://contoso.com/public`, pero, en su lugar, puede que desee redirigir `http://contoso.com` al directorio `public`. Este paso no conlleva la resolución de DNS, pero sí es necesario personalizar el directorio virtual.

Para ello, seleccione **Configuración de la aplicación** en el panel de navegación izquierdo de la página de la aplicación web. 

En la parte inferior de la página, el directorio virtual raíz `/` apunta a `site\wwwroot` de forma predeterminada, que es el directorio raíz del código de la aplicación. Cambie esta configuración para que, en su lugar, apunte, por ejemplo, a `site\wwwroot\public` y después guarde los cambios.

![Personalización del directorio virtual](./media/app-service-web-tutorial-custom-domain/customize-virtual-directory.png)

Una vez completada la operación, la aplicación debe devolver la página correcta en la ruta de acceso raíz (por ejemplo, `http://contoso.com`).

## <a name="automate-with-scripts"></a>Automatizar con scripts

Puede automatizar la administración de dominios personalizados con scripts, mediante la [CLI de Azure](/cli/azure/install-azure-cli) o [Azure PowerShell](/powershell/azure/). 

### <a name="azure-cli"></a>Azure CLI 

El comando siguiente agrega un nombre DNS personalizado configurado a una aplicación de App Service. 

```bash 
az webapp config hostname add \
    --webapp-name <app-name> \
    --resource-group <resource_group_name> \
    --hostname <fully_qualified_domain_name>
``` 

Para más información, consulte [Asignación de un dominio personalizado a una aplicación web](scripts/cli-configure-custom-domain.md).

### <a name="azure-powershell"></a>Azure PowerShell 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

El comando siguiente agrega un nombre DNS personalizado configurado a una aplicación de App Service.

```powershell  
Set-AzWebApp `
    -Name <app-name> `
    -ResourceGroupName <resource_group_name> ` 
    -HostNames @("<fully_qualified_domain_name>","<app-name>.azurewebsites.net")
```

Para obtener más información, vea [Asignación de un dominio personalizado a una aplicación web](scripts/powershell-configure-custom-domain.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Asignar un subdominio mediante el uso de un registro CNAME
> * Asignar un dominio raíz mediante el uso de un registro D
> * Asignar un dominio con comodín mediante el uso de un registro CNAME
> * Redireccionamiento de una dirección URL predeterminada a un directorio personalizado
> * Automatizar la asignación de dominio con scripts

Pase al siguiente tutorial para aprender a enlazar un certificado TLS/SSL personalizado a una aplicación web.

> [!div class="nextstepaction"]
> [Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service](configure-ssl-bindings.md)