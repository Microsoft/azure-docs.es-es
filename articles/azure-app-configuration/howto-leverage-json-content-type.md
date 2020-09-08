---
title: Uso de un tipo de contenido JSON para valores clave
titleSuffix: Azure App Configuration
description: Aprenda a usar un tipo de contenido JSON para los pares clave-valor
services: azure-app-configuration
author: avanigupta
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: azurecli
ms.topic: how-to
ms.date: 08/03/2020
ms.author: avgupta
ms.openlocfilehash: 725beb50e55852e35ee4434539ff158f082059df
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88121963"
---
# <a name="leverage-content-type-to-store-json-key-values-in-app-configuration"></a>Aproveche el tipo de contenido para almacenar los pares clave-valor JSON en App Configuration

Los datos se almacenan en App Configuration como pares clave-valor, donde se tratan de forma predeterminada como el tipo de cadena. Sin embargo, puede especificar un tipo personalizado aprovechando la propiedad de tipo de contenido (content-type) asociada a cada par clave-valor, para que pueda conservar el tipo original de los datos o hacer que la aplicación se comporte de forma diferente en función del tipo de contenido.


## <a name="overview"></a>Información general

En App Configuration, puede usar el tipo de elemento multimedia JSON como el tipo de contenido de los pares clave-valor para obtener las siguientes ventajas:
- **Una administración de datos más sencilla**: la administración de los pares clave-valor, como la de las matrices, es mucho más fácil en Azure Portal.
- **Una exportación de datos mejorada**: los tipos primitivos, las matrices y los objetos JSON se conservan durante la exportación de datos.
- **Compatibilidad nativa con el proveedor de App Configuration**: los pares clave-valor con el tipo de contenido JSON funcionan correctamente cuando los usan las bibliotecas de proveedores de App Configuration en las aplicaciones.

#### <a name="valid-json-content-type"></a>Tipo de contenido JSON válido

Los tipos de elementos multimedia, tal como se definen [aquí](https://www.iana.org/assignments/media-types/media-types.xhtml), se pueden asignar al tipo de contenido asociado con cada par clave-valor.
Un tipo de elemento multimedia consta de un tipo y un subtipo. Si el tipo es `application` y el subtipo (o sufijo) es `json`, el tipo de elemento multimedia se considerará un tipo de contenido JSON válido.
Algunos ejemplos de tipos de contenido JSON válidos son:

- application/json
- application/activity+json
- application/vnd.foobar+json;charset=utf-8

#### <a name="valid-json-values"></a>Valores válidos de JSON

Cuando un par clave-valor tiene un tipo de contenido JSON, su valor debe tener un formato JSON válido para que los clientes lo puedan procesar correctamente. De lo contrario, los clientes pueden generar un error o retroceder y tratarlo como formato de cadena.
Algunos ejemplos de valores válidos de JSON son:

- "John Doe"
- 723
- False
- null
- "2020-01-01T12:34:56.789Z"
- [1, 2, 3, 4]
- {"ObjectSetting":{"Targeting":{"Default":true,"Level":"Information"}}}

> [!NOTE]
> En el resto de este artículo, cualquier par clave-valor de App Configuration que tenga un tipo de contenido JSON válido y un valor JSON válido se denominará **valor clave JSON**. 

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Crear pares clave-valor JSON en App Configuration.
> * Importar los pares clave-valor JSON desde un archivo JSON.
> * Exportar los pares clave-valor JSON a un archivo JSON.
> * Consumir pares clave-valor JSON en las aplicaciones.


## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/).
- La versión más reciente de la CLI de Azure (2.10.0 o posterior). Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si usa la CLI de Azure, primero debe iniciar sesión con `az login`. También puede usar Azure Cloud Shell.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]


## <a name="create-an-app-configuration-store"></a>Creación de un almacén de App Configuration

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]


## <a name="create-json-key-values-in-app-configuration"></a>Creación de pares clave-valor JSON en App Configuration

Los pares clave-valor JSON se pueden crear mediante Azure Portal, la CLI de Azure o importándolos desde un archivo JSON. En esta sección, encontrará instrucciones sobre la creación de los mismos pares clave-valor JSON mediante los tres métodos.

### <a name="create-json-key-values-using-azure-portal"></a>Creación de pares clave-valor JSON mediante Azure Portal

Vaya al almacén de App Configuration y seleccione **Explorador de configuración** > **Crear** > **Clave-valor** para agregar los siguientes pares clave-valor:

| Clave | Value | Tipo de contenido |
|---|---|---|
| Settings:BackgroundColor | "Verde" | application/json |
| Settings:FontSize | 24 | application/json |
| Settings:UseDefaultRouting | false | application/json |
| Settings:BlockedUsers | null | application/json |
| Settings:ReleaseDate | "2020-08-04T12:34:56.789Z" | application/json |
| Settings:RolloutPercentage | [25,50,75,100] | application/json |
| Settings:Logging | {"Test":{"Level":"Debug"},"Prod":{"Level":"Warning"}} | application/json |

Deje el valor de **Etiqueta** vacío y seleccione **Aplicar**.

### <a name="create-json-key-values-using-azure-cli"></a>Creación de pares clave-valor JSON mediante la CLI de Azure

Los siguientes comandos crearán pares clave-valor JSON en el almacén de App Configuration. Reemplace `<appconfig_name>` por el nombre del almacén de App Configuration.

```azurecli-interactive
appConfigName=<appconfig_name>
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:BackgroundColor --value \"Green\"
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:FontSize --value 24
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:UseDefaultRouting --value false
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:BlockedUsers --value null
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:ReleaseDate --value \"2020-08-04T12:34:56.789Z\"
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:RolloutPercentage --value [25,50,75,100]
az appconfig kv set -n $appConfigName --content-type application/json --key Settings:Logging --value {\"Test\":{\"Level\":\"Debug\"},\"Prod\":{\"Level\":\"Warning\"}}
```

> [!IMPORTANT]
> Si usa la CLI de Azure o Azure Cloud Shell para crear pares clave-valor JSON, el valor proporcionado debe escapar mediante una cadena JSON.

### <a name="import-json-key-values-from-a-file"></a>Importación de pares clave-valor JSON desde un archivo

Cree un archivo JSON llamado `Import.json` con el siguiente contenido e impórtelo en forma de pares clave-valor en App Configuration:

```json
{
  "Settings": {
    "BackgroundColor": "Green",
    "BlockedUsers": null,
    "FontSize": 24,
    "Logging": {"Test":{"Level":"Debug"},"Prod":{"Level":"Warning"}},
    "ReleaseDate": "2020-08-04T12:34:56.789Z",
    "RolloutPercentage": [25,50,75,100],
    "UseDefaultRouting": false
  }
}
```

```azurecli-interactive
az appconfig kv import -s file --format json --path "~/Import.json" --content-type "application/json" --separator : --depth 2
```

> [!Note]
> El argumento `--depth` se usa para eliminar los datos jerárquicos de un archivo en pares clave-valor. En este tutorial, se especifica la profundidad para demostrar que también puede almacenar los objetos JSON como valores en App Configuration. Si no se especifica la profundidad, los objetos JSON se reducirán al nivel más profundo de forma predeterminada.

Los pares clave-valor JSON que creó deben tener una apariencia similar a la siguiente en App Configuration:

![Almacén de configuración que contiene los pares clave-valor JSON](./media/create-json-settings.png)


## <a name="export-json-key-values-to-a-file"></a>Exportación de pares clave-valor JSON a un archivo

Una de las principales ventajas de usar los pares clave-valor JSON es la capacidad de conservar el tipo de datos original de los valores durante la exportación. Si un par clave-valor en App Configuration no tiene su propiedad de tipo de contenido (content-type) JSON, su valor se tratará como una cadena. 

Considere estos pares clave-valor sin el tipo de contenido JSON:

| Clave | Value | Tipo de contenido |
|---|---|---|
| Settings:FontSize | 24 | |
| Settings:UseDefaultRouting | false | |

Al exportar estos pares clave-valor a un archivo JSON, los valores se exportarán como cadenas:
```json
{
  "Settings": {
    "FontSize": "24",
    "UseDefaultRouting": "false"
  }
}
```

Sin embargo, al exportar los pares clave-valor JSON a un archivo, todos los valores conservarán su tipo de datos original. Para comprobarlo, exporte los pares clave-valor desde App Configuration a un archivo JSON. Verá que el archivo exportado tiene el mismo contenido que el archivo `Import.json` que importó anteriormente.

```azurecli-interactive
az appconfig kv export -d file --format json --path "~/Export.json" --separator :
```

> [!NOTE]
> Si el almacén de App Configuration tiene algunos pares clave-valor sin tipo de contenido JSON, también se exportarán al mismo archivo en formato de cadena. Si solo desea exportar los pares clave-valor JSON, asígneles una etiqueta o un prefijo único y use el filtrado de etiquetas o prefijos durante la exportación.


## <a name="consuming-json-key-values-in-applications"></a>Consumo de pares clave-valor JSON en aplicaciones

La forma más fácil de consumir pares clave-valor JSON en una aplicación es mediante bibliotecas de proveedores de App Configuration. Con las bibliotecas de proveedores, no es necesario implementar un tratamiento especial de los pares clave-valor JSON en la aplicación. Siempre se deserializan para la aplicación de la misma manera que otras bibliotecas de proveedor de configuración de JSON. 

> [!Important]
> La compatibilidad nativa con los pares clave-valor JSON está disponible en la versión 4.0.0 del proveedor de configuración de .NET (o posterior). Para más información, consulte [*Siguientes pasos*](#next-steps).

Si usa el SDK o la API REST para leer los pares clave-valor desde App Configuration, según el tipo de contenido, la aplicación será responsable de deserializar el valor de un par clave-valor JSON mediante cualquier deserializador JSON estándar.


## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo trabajar con pares clave-valor JSON en el almacén de App Configuration, cree una aplicación para consumir estos pares clave-valor:

* [Inicio rápido de ASP.NET Core](./quickstart-aspnet-core-app.md)
    * Requisito previo: Paquete [Microsoft.Azure.AppConfiguration.AspNetCore](https://www.nuget.org/packages/Microsoft.Azure.AppConfiguration.AspNetCore) v4.0.0 o posterior.

* [Inicio rápido de .NET Core](./quickstart-dotnet-core-app.md)
    * Requisito previo: Paquete [Microsoft.Extensions.Configuration.AzureAppConfiguration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureAppConfiguration) v4.0.0 o posterior.
