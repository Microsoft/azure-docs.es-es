---
title: 'Inicio rápido: creación y administración de recursos en Azure Communication Services'
titleSuffix: An Azure Communication Services quickstart
description: En este inicio rápido, aprenderá a crear y administrar su primer recurso de Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2020
ms.topic: overview
ms.service: azure-communication-services
zone_pivot_groups: acs-plat-azp-net
ms.openlocfilehash: 3ba3d8d4b345c9db2e41c647147021eb9616288e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945939"
---
# <a name="quickstart-create-and-manage-communication-services-resources"></a>Inicio rápido: Creación y administración de recursos de Communication Services

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

Aprovisione su primer recurso de Communication Services a fin de empezar a usar Azure Communication Services. Los recursos de Communication Services se pueden aprovisionar a través de Azure Portal o con la biblioteca cliente de administración de .NET. La biblioteca cliente de administración le permite crear, configurar, actualizar y eliminar sus recursos e interfaces con [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/management/overview), el servicio de implementación y administración de Azure. Todas las funcionalidades disponibles en las bibliotecas cliente están disponibles en Azure Portal. 

> [!WARNING]
> Tenga en cuenta que la disponibilidad de Communication Services está restringida a la geografía de EE. UU. durante la versión preliminar pública. Así mismo, tenga presente que los recursos de comunicación no se pueden transferir a otra suscripción durante la versión preliminar pública.

::: zone pivot="platform-azp"
[!INCLUDE [Azure portal](./includes/create-resource-azp.md)]
::: zone-end

::: zone pivot="platform-net"
[!INCLUDE [.NET](./includes/create-resource-net.md)]
::: zone-end

## <a name="access-your-connection-strings-and-service-endpoints"></a>Acceso a las cadenas de conexión y los puntos de conexión de servicio

Las cadenas de conexión permiten que las bibliotecas cliente de Communication Services se conecten a Azure y se autentiquen en dicho portal. Puede acceder a las cadenas de conexión de Communication Services y a los puntos de conexión de servicio desde Azure Portal o mediante programación con las API de Azure Resource Manager. 

Después de navegar al recurso de Communication Services, seleccione **Claves** en el menú de navegación y copie el valor **Cadena de conexión** o **Punto de conexión** para que lo usen las bibliotecas cliente de Communication Services. Tenga en cuenta que tiene acceso a las claves principal y secundaria. Puede resultarle útil en escenarios en los que quiera proporcionar acceso temporal a los recursos de Communication Services a un entorno o de ensayo o de terceros.

:::image type="content" source="./media/key.png" alt-text="Captura de pantalla de la página de claves de Communication Services.":::

## <a name="store-your-connection-string"></a>Almacenamiento de la cadena de conexión

Las bibliotecas cliente de Communication Services usan cadenas de conexión para autorizar las solicitudes realizadas a Communication Services. Tiene varias opciones para almacenar una cadena de conexión:

* Una aplicación que se ejecuta en el escritorio o en un dispositivo puede almacenar la cadena de conexión en un archivo **app.config** o **web.config**. Agregue la cadena de conexión a la sección **AppSettings** en estos archivos.
* Una aplicación que se ejecuta en una instancia de Azure App Service puede almacenar la cadena de conexión en la [configuración de la aplicación de App Service](https://docs.microsoft.com/azure/app-service/configure-common). Agregue la cadena de conexión a la sección **Cadenas de conexión** de la pestaña Configuración de la aplicación en el portal.
* Puede almacenar la cadena de conexión en [Azure Key Vault](https://docs.microsoft.com/azure/data-factory/store-credentials-in-key-vault).
* Si ejecuta la aplicación localmente, le recomendamos que almacene la cadena de conexión en una variable de entorno.

### <a name="store-your-connection-string-in-an-environment-variable"></a>Almacenamiento de la cadena de conexión en una variable de entorno

Para configurar una variable de entorno, abra una ventana de consola y seleccione el sistema operativo en las pestañas siguientes. Reemplace `<yourconnectionstring>` por la cadena de conexión real.

#### <a name="windows"></a>[Windows](#tab/windows)

Abra una ventana de consola y escriba el siguiente comando:

```console
setx COMMUNICATION_SERVICES_CONNECTION_STRING "<yourconnectionstring>"
```

Después de agregar la variable de entorno, puede que tenga que reiniciar todos los programas en ejecución que necesiten leer la variable de entorno, incluida la ventana de consola. Por ejemplo, si usa Visual Studio como editor, reinícielo antes de ejecutar el ejemplo.

#### <a name="macos"></a>[macOS](#tab/unix)

Edite **.zshrc** y agregue la variable de entorno:

```bash
export COMMUNICATION_SERVICES_CONNECTION_STRING="<yourconnectionstring>"
```

Después de agregar la variable de entorno, ejecute `source ~/.zshrc` desde la ventana de consola para que los cambios surtan efecto. Si ha creado la variable de entorno con el IDE abierto, puede que tenga que cerrar y volver a abrir el editor, el IDE o el shell para acceder a esta variable.

#### <a name="linux"></a>[Linux](#tab/linux)

Edite **.bash_profile** y agregue la variable de entorno:

```bash
export COMMUNICATION_SERVICES_CONNECTION_STRING="<yourconnectionstring>"
```

Después de agregar la variable de entorno, ejecute `source ~/.bash_profile` desde la ventana de consola para que los cambios surtan efecto. Si ha creado la variable de entorno con el IDE abierto, puede que tenga que cerrar y volver a abrir el editor, el IDE o el shell para acceder a esta variable.

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

Si tiene números de teléfono asignados a su recurso cuando lo elimine, estos se liberarán automáticamente del recurso al mismo tiempo. 

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, aprendió a hacer lo siguiente:

> [!div class="checklist"]
> * Crear un recurso de Communication Services
> * Configurar la geografía y las etiquetas de recursos
> * Acceder a las claves de ese recurso
> * Eliminar el recurso

> [!div class="nextstepaction"]
> [Creación del primer token de acceso de usuario](access-tokens.md)
