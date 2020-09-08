---
title: Configuración del servicio API Management con Git - Azure | Microsoft Docs
description: Obtenga información sobre cómo guardar y configurar la configuración del servicio de administración de API mediante Git.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: mattfarm
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/12/2019
ms.author: apimpm
ms.openlocfilehash: 18cc42c3447de733447c27db52a9a6d664539464
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400369"
---
# <a name="how-to-save-and-configure-your-api-management-service-configuration-using-git"></a>Guardado y configuración del servicio Administración de API mediante Git

Cada instancia del servicio Administración de API mantiene una base de datos de configuración que contiene información sobre la configuración y los metadatos de la instancia del servicio. Es posible hacer cambios en la instancia del servicio; para ello, modifique un valor en Azure Portal con un cmdlet de PowerShell o realice una llamada de API REST. Pero esto no es todo, también puede administrar la configuración de la instancia del servicio con Git, con lo que se posibilitan escenarios de administración de servicio como los siguientes:

* Control de versiones de configuración: descargue y almacene versiones diferentes de la configuración del servicio
* Cambios masivos en la configuración: realice cambios en varios ajustes de la configuración del servicio en el repositorio local e integre los cambios de nuevo en el servidor con una sola operación
* Cadena de herramientas y flujo de trabajo familiares de Git: use las herramientas y los flujos de trabajo de Git con los que ya está familiarizado

El diagrama siguiente muestra de forma global los distintos modos de configurar la instancia del servicio de API Management.

![Configuración de GIT][api-management-git-configure]

Cuando hace cambios en el servicio mediante Azure Portal, los cmdlets de PowerShell o la API REST, está administrando la base de datos de configuración del servicio mediante el punto de conexión `https://{name}.management.azure-api.net`, tal como se muestra en el lado derecho del diagrama. En el lado izquierdo del diagrama se muestra cómo administrar la configuración del servicio mediante Git y el repositorio de Git para el servicio ubicado en `https://{name}.scm.azure-api.net`.

Los pasos siguientes proporcionan una visión general sobre el proceso de administración de la instancia del servicio de API Management mediante Git.

1. Acceso a la configuración de Git en el servicio
2. Guardar la base de datos de configuración del servicio en el repositorio de Git
3. Clonar el repositorio de Git en el equipo local
4. Desplegar el repositorio más reciente en el equipo local, y confirmar e insertar los cambios de nuevo en el repositorio
5. Implementar los cambios desde el repositorio en la base de datos de configuración del servicio

Este artículo describe cómo habilitar y usar Git para administrar la configuración del servicio y sirve como referencia para los archivos y las carpetas del repositorio Git.

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="access-git-configuration-in-your-service"></a>Acceso a la configuración de Git en el servicio

Para ver y configurar las opciones de configuración Git, puede hacer clic en el menú **Deployment and infrastructure**  (Implementación e Infraestructura) e ir a la pestaña **Repositorio**.

![Habilitar GIT][api-management-enable-git]

> [!IMPORTANT]
> Los secretos que no se definan como valores con nombre se almacenarán en el repositorio y permanecerán en su historial hasta que deshabilite y vuelva a habilitar el acceso de Git. Los valores con nombre proporcionan un lugar seguro para administrar los valores de cadena constante, como los secretos, en toda la configuración y las directivas de API, por lo que no tiene que almacenarlos directamente en las instrucciones de directiva. Para más información, consulte [Cómo usar valores con nombre en las directivas de Azure API Management](api-management-howto-properties.md).
>
>

Para obtener información sobre la habilitación o la deshabilitación del acceso de Git mediante la API de REST, consulte [Enable or disable Git access using the REST API](/rest/api/apimanagement/2019-12-01/tenantaccess?EnableGit)(Habilitación o deshabilitación del acceso de Git mediante la API de REST).

## <a name="to-save-the-service-configuration-to-the-git-repository"></a>Guardado de la configuración del servicio en el repositorio de Git

El primer paso antes de la clonación del repositorio es guardar el estado actual de la configuración del servicio en el repositorio. Haga clic en **Guardar en repositorio**.

Realice los cambios deseados en la pantalla de confirmación y haga clic en **Guardar** para guardarlos.

Transcurridos unos segundos, la configuración se guarda y se muestra el estado de configuración del repositorio, incluidas la fecha y la hora del último cambio de configuración y la última sincronización entre la configuración del servicio y el repositorio.

Una vez que la configuración se guarda en el repositorio, se puede clonar.

Para obtener información acerca de cómo realizar esta operación mediante la API de REST, consulte [Commit configuration snapshot using the REST API](/rest/api/apimanagement/2019-12-01/tenantaccess?CommitSnapshot)(Confirmación de la instantánea de configuración mediante la API de REST).

## <a name="to-clone-the-repository-to-your-local-machine"></a>Para clonar el repositorio en el equipo local

Para clonar un repositorio, necesitará la dirección URL del repositorio, un nombre de usuario y una contraseña. Para obtener el nombre de usuario y otras credenciales, haga clic en **Credenciales de acceso** cerca de la parte superior de la página.

Para generar una contraseña, primero asegúrese de que el campo **Expiración** refleja la fecha y la hora de caducidad deseada y luego haga clic en **Generar**.

> [!IMPORTANT]
> Anote esta contraseña. Una vez que salga de esta página, la contraseña no se volverá a mostrar.
>

En los ejemplos siguientes se usa la herramienta Git Bash desde [Git para Windows](https://www.git-scm.com/downloads) , pero puede usar cualquier herramienta de Git con la que esté familiarizado.

Abra su herramienta Git en la carpeta deseada y ejecute el siguiente comando para clonar el repositorio de Git en el equipo local, usando para ello el comando incluido en Azure Portal.

```
git clone https://{name}.scm.azure-api.net/
```

Proporcione el nombre de usuario y la contraseña cuando se le solicite.

Si recibe algún error, pruebe a modificar su comando `git clone` para incluir el nombre de usuario y la contraseña, como se muestra en el ejemplo siguiente.

```
git clone https://username:password@{name}.scm.azure-api.net/
```

Si de este modo aparece un error, pruebe a codificar como dirección URL la parte de la contraseña del comando. Una manera rápida de hacer esto es abrir Visual Studio y emitir el siguiente comando en la **Ventana Inmediato**. Para abrir la **Ventana Inmediato**, abra cualquier solución o proyecto en Visual Studio (o cree una aplicación de consola vacía) y elija **Ventanas**, **Inmediato** en el menú **Depurar**.

```
?System.Net.WebUtility.UrlEncode("password from the Azure portal")
```

Utilice la contraseña codificada junto con su nombre de usuario y ubicación de repositorio para construir el comando git.

```
git clone https://username:url encoded password@{name}.scm.azure-api.net/
```

Una vez que se clone el repositorio, podrá ver y trabajar con él en el sistema de archivos local. Para más información, consulte [Referencia de estructura de archivo y carpeta del repositorio local de Git](#file-and-folder-structure-reference-of-local-git-repository).

## <a name="to-update-your-local-repository-with-the-most-current-service-instance-configuration"></a>Para actualizar su repositorio local con la configuración de instancia de servicio más reciente

Si realiza cambios en la instancia del servicio de API Management en Azure Portal o mediante la API REST, debe guardar estos cambios en el repositorio para poder actualizar el repositorio local con los cambios más recientes. Para ello, haga clic en **Guardar configuración en repositorio** en la pestaña **Repositorio de configuración** de Azure Portal y emita el siguiente comando en el repositorio local.

```
git pull
```

Antes de ejecutar `git pull` , asegúrese de que se encuentra en la carpeta del repositorio local. Si acaba de completar el comando `git clone` , debe cambiar el directorio a su repositorio ejecutando un comando similar al siguiente.

```
cd {name}.scm.azure-api.net/
```

## <a name="to-push-changes-from-your-local-repo-to-the-server-repo"></a>Para insertar los cambios desde su repositorio local al repositorio del servidor
Para insertar los cambios desde el repositorio local al repositorio del servidor, debe confirmar los cambios y, a continuación, insertarlos en el repositorio del servidor. Para confirmar los cambios, abra la herramienta de comandos de Git, cambie al directorio del repositorio local y emita los siguientes comandos.

```
git add --all
git commit -m "Description of your changes"
```

Para insertar todas las confirmaciones en el servidor, ejecute el siguiente comando.

```
git push
```

## <a name="to-deploy-any-service-configuration-changes-to-the-api-management-service-instance"></a>Para implementar los cambios de la configuración del servicio en la instancia del servicio de API Management

Una vez confirmados los cambios locales e insertados en el repositorio del servidor, puede implementarlos en la instancia del servicio Administración de API.

Para obtener información acerca de cómo realizar esta operación mediante la API de REST, consulte [Deploy Git changes to configuration database using the REST API](/rest/api/apimanagement/2019-12-01/tenantconfiguration)(Implementación de cambios de Git en la base de datos de configuración mediante la API de REST).

## <a name="file-and-folder-structure-reference-of-local-git-repository"></a>Referencia de estructura de archivo y carpeta del repositorio local de Git

Los archivos y carpetas del repositorio local de Git contienen la información de configuración acerca de la instancia de servicio.

| Elemento | Descripción |
| --- | --- |
| carpeta raíz de la administración de API |Contiene la configuración de nivel superior para la instancia de servicio |
| carpeta de API |Contiene la configuración para las API de la instancia de servicio |
| carpeta de grupos |Contiene la configuración para los grupos de la instancia de servicio |
| carpeta de directivas |Contiene las directivas de la instancia de servicio |
| carpeta portalStyles |Contiene la configuración para las personalizaciones del portal para desarrolladores de la instancia de servicio |
| carpeta de productos |Contiene la configuración para los productos de la instancia de servicio |
| carpeta de plantillas |Contiene la configuración para las plantillas de correo electrónico de la instancia de servicio |

Cada carpeta puede contener uno o varios archivos y, en algunos casos, una o varias carpetas; por ejemplo, una carpeta para cada API, producto o grupo. Los archivos dentro de cada carpeta son específicos del tipo de entidad descrito por el nombre de la carpeta.

| Tipo de archivo | Propósito |
| --- | --- |
| json |Información de configuración acerca de la entidad correspondiente |
| html |Descripción de la entidad, a menudo mostrada en el portal para desarrolladores |
| Xml |Policy statements |
| css |Hojas de estilo para la personalización del portal para desarrolladores |

Estos archivos se pueden crear, eliminar, editar y administrar en el sistema de archivos local, y los cambios se pueden implementar de nuevo en la instancia de servicio de API Management.

> [!NOTE]
> Las siguientes entidades no están en el repositorio de Git y no se pueden configurar mediante Git.
>
> * [Usuarios](/rest/api/apimanagement/2019-12-01/user)
> * [Suscripciones](/rest/api/apimanagement/2019-12-01/subscription)
> * Valores con nombre
> * Entidades del portal de desarrolladores distintas de los estilos
>

### <a name="root-api-management-folder"></a>carpeta raíz de la administración de API
La carpeta raíz `api-management` contiene un archivo `configuration.json` con información de nivel superior sobre la instancia de servicio en el formato siguiente.

```json
{
  "settings": {
    "RegistrationEnabled": "True",
    "UserRegistrationTerms": null,
    "UserRegistrationTermsEnabled": "False",
    "UserRegistrationTermsConsentRequired": "False",
    "DelegationEnabled": "False",
    "DelegationUrl": "",
    "DelegatedSubscriptionEnabled": "False",
    "DelegationValidationKey": "",
    "RequireUserSigninEnabled": "false"
  },
  "$ref-policy": "api-management/policies/global.xml"
}
```

Los primeros cuatro valores (`RegistrationEnabled`, `UserRegistrationTerms`, `UserRegistrationTermsEnabled` y `UserRegistrationTermsConsentRequired`) se asignan a la siguiente configuración en la pestaña **Identidades** de la sección **Portal para desarrolladores**.

| Configuración de identidad | Se asigna a |
| --- | --- |
| RegistrationEnabled |Presencia del proveedor de identidades **Nombre de usuario y contraseña** |
| UserRegistrationTerms |**Condiciones de uso del registro de usuario** |
| UserRegistrationTermsEnabled |**Mostrar condiciones de uso en la página de registro** |
| UserRegistrationTermsConsentRequired |**Requerir consentimiento** |
| RequireUserSigninEnabled |**Redirigir a los usuarios anónimos a la página de inicio de sesión** |

La cuatro valores siguientes (`DelegationEnabled`, `DelegationUrl`, `DelegatedSubscriptionEnabled` y `DelegationValidationKey`) se asignan a la siguiente configuración en la pestaña **Delegación** de la sección **Portal para desarrolladores**.

| Configuración de delegación | Se asigna a |
| --- | --- |
| DelegationEnabled |Casilla **Delegar inicio de sesión y registro** |
| DelegationUrl |**Dirección URL del punto de conexión de delegación** |
| DelegatedSubscriptionEnabled |**Delegar suscripción de producto** |
| DelegationValidationKey |**Delegar clave de validación** |

El valor final, `$ref-policy`, se asigna al archivo de instrucciones de directiva global para la instancia de servicio.

### <a name="apis-folder"></a>carpeta de API
La carpeta `apis` contiene una carpeta para cada API de la instancia de servicio, que contiene los elementos siguientes.

* `apis\<api name>\configuration.json` : es la configuración de la API y contiene información acerca de la dirección URL del servicio back-end y las operaciones. Se trata de la misma información que se devolvería si se llamase a [Obtener una API específica](/rest/api/apimanagement/2019-12-01/apis/get) con `export=true` en formato `application/json`.
* `apis\<api name>\api.description.html`: es la descripción de la API y corresponde a la propiedad `description` de la [entidad de API](/java/api/com.microsoft.azure.storage.table.entityproperty).
* `apis\<api name>\operations\`: esta carpeta contiene archivos `<operation name>.description.html` que se asignan a las operaciones de la API. Cada archivo contiene la descripción de una única operación en la API, que se asigna a la propiedad `description` de la [entidad de operación](/rest/api/visualstudio/operations/list#operationproperties) en la API de REST.

### <a name="groups-folder"></a>carpeta de grupos
La carpeta `groups` contiene una carpeta para cada grupo definido en la instancia de servicio.

* `groups\<group name>\configuration.json`: es la configuración para el grupo. Se trata de la misma información que se devolvería si se llamase a la operación [Obtener un grupo específico](/rest/api/apimanagement/2019-12-01/group/get) .
* `groups\<group name>\description.html`: es la descripción del grupo y corresponde a la propiedad `description` de la [entidad de servicio](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-group-entity).

### <a name="policies-folder"></a>carpeta de directivas
La carpeta `policies` contiene las instrucciones de directiva para la instancia de servicio.

* `policies\global.xml` : contiene las directivas definidas en el ámbito global para la instancia de servicio.
* `policies\apis\<api name>\`: si tiene directivas definidas en el ámbito de la API, se encuentran en esta carpeta.
* Carpeta `policies\apis\<api name>\<operation name>\`: si tiene directivas definidas en el ámbito de la operación, se encuentran en esta carpeta en archivos `<operation name>.xml` que se asignan a las instrucciones de directiva para cada operación.
* `policies\products\`: si tiene directivas definidas en el ámbito del producto, se encuentran esta carpeta, que contiene archivos `<product name>.xml` que se asignan a las instrucciones de directiva para cada producto.

### <a name="portalstyles-folder"></a>carpeta portalStyles
La carpeta `portalStyles` contiene la configuración y las hojas de estilo para las personalizaciones del portal para desarrolladores de la instancia de servicio.

* `portalStyles\configuration.json` : contiene los nombres de las hojas de estilos usadas por el portal de desarrolladores
* `portalStyles\<style name>.css`: cada archivo `<style name>.css` contiene estilos para el portal para desarrolladores (`Preview.css` y `Production.css` de forma predeterminada).

### <a name="products-folder"></a>carpeta de productos
La carpeta `products` contiene una carpeta para cada producto que se define en la instancia de servicio.

* `products\<product name>\configuration.json`: es la configuración del producto. Se trata de la misma información que se devolvería si se llamase a la operación [Obtener un producto específico](/rest/api/apimanagement/2019-12-01/product/get) .
* `products\<product name>\product.description.html`: es la descripción del producto y corresponde a la propiedad `description` de la [entidad de producto](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-product-entity) de la API de REST.

### <a name="templates"></a>plantillas
La carpeta `templates` contiene la configuración para las [plantillas de correo electrónico](api-management-howto-configure-notifications.md) de la instancia de servicio.

* `<template name>\configuration.json` : es la configuración de la plantilla de correo electrónico.
* `<template name>\body.html` : es el cuerpo de la plantilla de correo electrónico.

## <a name="next-steps"></a>Pasos siguientes
Para obtener información sobre otras formas de administrar la instancia de servicio, consulte:

* Administrar la instancia de servicio con los siguientes cmdlets de PowerShell
  * [Azure API Management Deployment Management Cmdlets (Cmdlets de administración de la implementación de Administración de API de Azure)](/powershell/module/wds)
  * [Azure API Management Service Management Cmdlets (Cmdlets de administración del servicio Administración de API de Azure)](/powershell/azure/servicemanagement/overview)
* Administrar la instancia de servicio mediante la API de REST
  * [API Management REST API (API de REST de Administración de API)](/rest/api/apimanagement/)


[api-management-enable-git]: ./media/api-management-configuration-repository-git/api-management-enable-git.png
[api-management-git-enabled]: ./media/api-management-configuration-repository-git/api-management-git-enabled.png
[api-management-save-configuration]: ./media/api-management-configuration-repository-git/api-management-save-configuration.png
[api-management-save-configuration-confirm]: ./media/api-management-configuration-repository-git/api-management-save-configuration-confirm.png
[api-management-configuration-status]: ./media/api-management-configuration-repository-git/api-management-configuration-status.png
[api-management-configuration-git-clone]: ./media/api-management-configuration-repository-git/api-management-configuration-git-clone.png
[api-management-generate-password]: ./media/api-management-configuration-repository-git/api-management-generate-password.png
[api-management-password]: ./media/api-management-configuration-repository-git/api-management-password.png
[api-management-git-configure]: ./media/api-management-configuration-repository-git/api-management-git-configure.png
[api-management-configuration-deploy]: ./media/api-management-configuration-repository-git/api-management-configuration-deploy.png
[api-management-identity-settings]: ./media/api-management-configuration-repository-git/api-management-identity-settings.png
[api-management-delegation-settings]: ./media/api-management-configuration-repository-git/api-management-delegation-settings.png
[api-management-git-icon-enable]: ./media/api-management-configuration-repository-git/api-management-git-icon-enable.png
