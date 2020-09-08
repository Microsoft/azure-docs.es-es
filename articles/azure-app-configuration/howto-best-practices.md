---
title: Procedimientos recomendados para Azure App Configuration | Microsoft Docs
description: Obtenga información sobre procedimientos recomendados al usar Azure App Configuration. Entre los temas tratados se incluyen las agrupaciones de claves, las composiciones de pares clave-valor, el arranque de App Configuration, etc.
services: azure-app-configuration
documentationcenter: ''
author: lisaguthrie
manager: maiye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: lcozzens
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: d532b8aab87840f4b6ad90daedba743597f4fe43
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88588065"
---
# <a name="azure-app-configuration-best-practices"></a>Procedimientos recomendados para Azure App Configuration

En este artículo se describen patrones comunes y procedimientos recomendados cuando se usa Azure App Configuration.

## <a name="key-groupings"></a>Agrupaciones de claves

App Configuration proporciona dos opciones para organizar las claves:

* Prefijos de clave
* Etiquetas

Puede usar una o ambas opciones para agrupar sus claves.

*Los prefijos de clave* son lo que va al principio de la clave. Puede agrupar lógicamente un conjunto de claves utilizando el mismo prefijo en sus nombres. Los prefijos pueden contener varios componentes conectados por un delimitador, como `/`, de forma similar a una ruta de dirección de URL, para formar un espacio de nombres. Estas jerarquías son útiles cuando se están almacenando las claves para muchas aplicaciones, servicios de componentes y entornos en un almacén de App Configuration.

Una cuestión importante que debe tener en cuenta es que las claves son a lo que el código de aplicación hace referencia para recuperar los valores de la configuración correspondiente. Las claves no deberían cambiar o bien tendrá que modificar el código cada vez que suceda.

*Las etiquetas* son un atributo en las claves. Se usan para crear variantes de una clave. Por ejemplo, puede asignar etiquetas a varias versiones de una clave. Una versión podría ser una iteración, un entorno o cualquier otra información contextual. La aplicación puede solicitar un conjunto completamente distinto de los valores de clave al especificar otra etiqueta. Como resultado, todas las referencias de clave permanecen sin cambios en el código.

## <a name="key-value-compositions"></a>Composiciones de pares clave-valor

App Configuration trata todas las claves que se almacenan con él como entidades independientes. App Configuration no intenta deducir ninguna relación entre las claves o heredar valores de clave en función de su jerarquía. Sin embargo, es posible agregar varios conjuntos de claves mediante el uso de etiquetas junto con la configuración correcta del apilamiento en el código de la aplicación.

Veamos un ejemplo. Suponga que tiene una opción denominada **Asset1** (Recurso1), cuyo valor puede variar según el entorno de desarrollo. Cree una clave denominada "Asset1" con una etiqueta vacía y una etiqueta denominada "Development" (Desarrollo). En la primera etiqueta, coloca el valor predeterminado de **Asset1** y debe incluir un valor específico para "Development" en la última.

En el código, primero recupera los valores de clave sin ninguna etiqueta y, a continuación, recupera el mismo conjunto de valores de clave con la etiqueta "Development". Cuando se recuperan los valores la segunda vez, se sobrescriben los valores anteriores de las claves. El sistema de configuración de .NET Core permite "apilar" varios conjuntos de datos de configuración en la parte superior entre sí. Si una clave existe en más de un conjunto, se utiliza el último conjunto que la contiene. Con un marco de programación moderno, como .NET Core, obtiene esta funcionalidad de apilamiento de forma gratuita si usa un proveedor de configuración nativo para tener acceso a App Configuration. El fragmento de código siguiente le muestra cómo puede implementar el apilamiento en una aplicación .NET Core:

```csharp
// Augment the ConfigurationBuilder with Azure App Configuration
// Pull the connection string from an environment variable
configBuilder.AddAzureAppConfiguration(options => {
    options.Connect(configuration["connection_string"])
           .Select(KeyFilter.Any, LabelFilter.Null)
           .Select(KeyFilter.Any, "Development");
});
```

[Uso de etiquetas para habilitar diferentes configuraciones para distintos entornos](./howto-labels-aspnet-core.md) proporciona un ejemplo completo.

## <a name="app-configuration-bootstrap"></a>Arranque de App Configuration

Para acceder a un almacén de App Configuration, puede usar su cadena de conexión, que está disponible en Azure Portal. Dado que las cadenas de conexión contienen información de credenciales, se consideran secretos. Estos secretos deben almacenarse en Azure Key Vault y el código debe autenticarse en Key Vault para recuperarlos.

Una mejor opción es usar la característica de las identidades administradas en Azure Active Directory. Con las identidades administradas, solo necesita la dirección URL del punto de conexión de App Configuration para el acceso de arranque a su almacén de App Configuration. Puede insertar la dirección URL en el código de aplicación (por ejemplo, en el archivo *appsettings.json*). Consulte [Integración con las identidades administradas de Azure](howto-integrate-azure-managed-service-identity.md) para obtener detalles.

## <a name="app-or-function-access-to-app-configuration"></a>Acceso a funciones o aplicaciones para App Configuration

Puede proporcionar acceso a App Configuration para las aplicaciones web o las funciones mediante alguno de los métodos siguientes:

* En Azure Portal, escriba la cadena de conexión para el almacén de App Configuration en la configuración de la aplicación de App Service.
* Almacene la cadena de conexión para su almacén de App Configuration en Key Vault y [haga referencia a ella desde App Service](https://docs.microsoft.com/azure/app-service/app-service-key-vault-references).
* Use las identidades administradas de Azure para acceder al almacén de App Configuration. Para más información, consulte [Integración con identidades administradas de Azure](howto-integrate-azure-managed-service-identity.md).
* Inserte la configuración de App Configuration en App Service. App Configuration proporciona una función de exportación (en Azure Portal y la CLI de Azure) que envía los datos directamente a App Service. Con este método, no es necesario cambiar el código de la aplicación.

## <a name="reduce-requests-made-to-app-configuration"></a>Reducción de las solicitudes realizadas a App Configuration

Una cantidad excesiva de solicitudes a App Configuration puede dar lugar a cargos por superar el límite de ancho de banda o el de uso. Para reducir el número de solicitudes realizadas:

* Aumente el tiempo de espera de la actualización, especialmente si los valores de configuración no cambian con frecuencia. Especifique un nuevo tiempo de espera de actualización mediante el [método `SetCacheExpiration`](/dotnet/api/microsoft.extensions.configuration.azureappconfiguration.azureappconfigurationrefreshoptions.setcacheexpiration).

* Vea una sola *clave de Sentinel*, en lugar de ver claves individuales. Actualice toda la configuración solo si cambia la clave de Sentinel. Consulte [Uso de la configuración dinámica en una aplicación de ASP.NET Core](enable-dynamic-configuration-aspnet-core.md) para obtener un ejemplo.

* Use Azure Event Grid para recibir notificaciones cuando cambie la configuración, en lugar de realizar un sondeo constante de los cambios. Para más información, consulte [Enrutamiento de eventos de Azure App Configuration a un punto de conexión web con la CLI de Azure](./howto-app-configuration-event.md).

## <a name="importing-configuration-data-into-app-configuration"></a>Importación de datos de configuración en App Configuration

App Configuration ofrece la opción de [importación](https://aka.ms/azconfig-importexport1) masiva de las opciones de los archivos de configuración actuales mediante Azure Portal o la CLI de Azure. También puede usar las mismas opciones para exportar valores de App Configuration, por ejemplo, entre almacenes relacionados. Si desea configurar una sincronización continua con el repositorio de GitHub, puede usar nuestra [acción de GitHub](https://aka.ms/azconfig-gha2) para poder seguir usando las prácticas de control de código fuente existentes a la vez que obtiene las ventajas de App Configuration.

## <a name="multi-region-deployment-in-app-configuration"></a>Implementación en varias regiones en App Configuration

App Configuration es el servicio regional. En el caso de las aplicaciones con distintas configuraciones por región, el almacenamiento de estas configuraciones en una instancia puede crear un único punto de error. La implementación de una instancia de App Configuration por región en varias regiones puede ser mejor opción. Puede ayudar con la recuperación ante desastres regional, el rendimiento y el silo de seguridad. La configuración por región también mejora la latencia y usa cuotas de limitación independientes, ya que la limitación es por instancia. Para aplicar la mitigación de recuperación ante desastres, puede utilizar [varios almacenes de configuración](./concept-disaster-recovery.md). 

## <a name="next-steps"></a>Pasos siguientes

* [Claves y valores](./concept-key-value.md)
