---
title: 'Solución del error: No se puede acceder a Azure Functions Runtime'
description: Aprenda a solucionar los problemas con una cuenta de almacenamiento no válida.
ms.topic: article
ms.date: 09/05/2018
ms.openlocfilehash: c46ca214ab6c0798fdc39ead575fb2873b8c51c8
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87385864"
---
# <a name="troubleshoot-error-azure-functions-runtime-is-unreachable"></a>Solución del error: "No se puede acceder a Azure Functions Runtime"

Este artículo ayuda a solucionar la cadena de error siguiente que aparece en Azure Portal:

> "Error: No se puede acceder a Azure Functions Runtime. Haga clic aquí para obtener detalles sobre la configuración de almacenamiento".

Este problema se produce cuando no se puede iniciar Azure Functions Runtime. La razón más común de este error es que la aplicación de funciones haya perdido el acceso a su cuenta de almacenamiento. Para más información, consulte [Requisitos de la cuenta de almacenamiento](./functions-create-function-app-portal.md#storage-account-requirements).

Las demás secciones de este artículo le ayudarán a solucionar las siguientes causas del error, así como a identificar y a resolver cada caso.

## <a name="storage-account-was-deleted"></a>Se ha eliminado la cuenta de almacenamiento

Cada aplicación de función requiere una cuenta de almacenamiento para funcionar. Si se elimina la cuenta, la función no funcionará.

Empiece por buscar el nombre de la cuenta de almacenamiento en la configuración de la aplicación. `AzureWebJobsStorage` o `WEBSITE_CONTENTAZUREFILECONNECTIONSTRING` contienen el nombre de la cuenta de almacenamiento incluida en una cadena de conexión. Para más información, consulte [Referencia de configuración de aplicación para Azure Functions](./functions-app-settings.md#azurewebjobsstorage).

Busque la cuenta de almacenamiento en Azure Portal para ver si sigue existiendo. Si se ha eliminado, vuelva a crear la cuenta de almacenamiento y reemplace las cadenas de conexión de almacenamiento. El código de función se pierde y debe volver a implementarlo.

## <a name="storage-account-application-settings-were-deleted"></a>Se ha eliminado la configuración de la aplicación de la cuenta de almacenamiento

En el paso anterior, si no encontraba una cadena de conexión de la cuenta de almacenamiento, es probable que se eliminara o se sobrescribiera. La configuración de la aplicación se suele eliminar cuando se usan ranuras de implementación o scripts de Azure Resource Manager para la configuración de la aplicación.

### <a name="required-application-settings"></a>Configuración de aplicación necesaria

* Requerido:
    * [`AzureWebJobsStorage`](./functions-app-settings.md#azurewebjobsstorage)
* Obligatorio para las funciones del plan de consumo:
    * [`WEBSITE_CONTENTAZUREFILECONNECTIONSTRING`](./functions-app-settings.md)
    * [`WEBSITE_CONTENTSHARE`](./functions-app-settings.md)

Para más información, consulte [Referencia de configuración de aplicación para Azure Functions](./functions-app-settings.md).

### <a name="guidance"></a>Guía

* No seleccione "Configuración de ranuras" en ninguna de estas opciones. Si se intercambian las ranuras de implementación, la aplicación de funciones se interrumpe.
* No modifique esta configuración durante las implementaciones automatizadas.
* Esta configuración debe ser válida y proporcionarse en el momento de la creación. Si una implementación automatizada no tuviera esta configuración, la aplicación de funciones no se ejecutaría, ni siquiera aunque esta configuración se agregara más adelante.

## <a name="storage-account-credentials-are-invalid"></a>Las credenciales de la cuenta de almacenamiento no son válidas

Al volver a generar las claves de almacenamiento deben actualizarse las cadenas de conexión de la cuenta de almacenamiento que se han comentado anteriormente. Para más información sobre la administración de las claves de almacenamiento, consulte [Creación de una cuenta de Azure Storage](../storage/common/storage-account-create.md).

## <a name="storage-account-is-inaccessible"></a>No se puede acceder a la cuenta de almacenamiento

Es necesario que la aplicación de funciones pueda acceder a la cuenta de almacenamiento. Estos son los problemas comunes de bloqueo del acceso de la aplicación de funciones a la cuenta de almacenamiento:

* La aplicación de funciones está implementada en App Service Environment sin las reglas de red adecuadas para permitir el tráfico de entrada y salida de la cuenta de almacenamiento.

* El firewall de la cuenta de almacenamiento está habilitado, pero no está configurado para permitir el tráfico de entrada y salida de Functions. Para más información, vea [Configuración de Firewalls y redes virtuales de Azure Storage](../storage/common/storage-network-security.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

## <a name="daily-execution-quota-is-full"></a>Se ha alcanzado la cuota de ejecución diaria

Si tiene configurada una cuota de ejecución diaria, la aplicación de funciones se deshabilitará temporalmente y muchos de los controles del portal dejarán de estar disponibles. 

Para comprobar la cuota en [Azure Portal](https://portal.azure.com), seleccione **Platform Features** (Características de la plataforma)  > **Configuración de aplicación de funciones** en la aplicación de funciones. Si supera el valor establecido en **Cuota de uso diario**, aparecerá el siguiente mensaje:

  > "La aplicación de funciones ha alcanzado la cuota de uso diaria y se ha detenido hasta que comience el siguiente período de 24 horas".

Para resolver este problema, quite o aumente la cuota diaria y reinicie la aplicación. De lo contrario, la ejecución de la aplicación se bloqueará hasta el día siguiente.

## <a name="app-is-behind-a-firewall"></a>Aplicación detrás de un firewall

Es posible que no se pueda acceder al entorno en tiempo de ejecución de la función por alguna de las razones siguientes:

* La aplicación de funciones se hospeda en una [instancia de App Service Environment con equilibrio de carga interno](../app-service/environment/create-ilb-ase.md) y está configurada para bloquear el tráfico entrante de Internet.

* La aplicación de funciones tiene [restricciones de IP de entrada](functions-networking-options.md#inbound-ip-restrictions) que están configuradas para bloquear el acceso a Internet. 

Azure Portal realiza llamadas directamente a la aplicación en ejecución para obtener la lista de funciones y realiza llamadas HTTP al punto de conexión de Kudu. La configuración en el nivel de plataforma de la pestaña **Platform Features** (Características de la plataforma) sigue estando disponible.

Para comprobar la configuración de App Service Environment:
1. Vaya al grupo de seguridad de red de la subred en la que reside la instancia de App Service Environment.
1. Valide las reglas de entrada para permitir el tráfico procedente de la dirección IP pública del equipo desde donde se accede a la aplicación. 
   
También puede usar el portal desde un equipo conectado a la red virtual que ejecuta la aplicación o una máquina virtual que se ejecuta en la red virtual. 

Para más información acerca de la configuración de reglas de entrada, consulte la sección "Grupos de seguridad de red" de [Consideraciones de red para una instancia de App Service Environment](../app-service/environment/network-info.md#network-security-groups).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la supervisión de las aplicaciones de funciones:

> [!div class="nextstepaction"]
> [Monitor Azure Functions](functions-monitoring.md)
