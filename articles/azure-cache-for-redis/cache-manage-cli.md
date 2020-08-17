---
title: Administración de Azure Cache for Redis mediante la CLI de Azure clásica
description: En este tema se describe cómo instalar la CLI de Azure clásica en cualquier plataforma, cómo usarla para conectarse a la cuenta de Azure y cómo crear y administrar una instancia de Azure Cache for Redis desde la CLI clásica.
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 01/23/2017
ms.author: yegu
ms.custom: devx-track-azurecli
ms.openlocfilehash: db55ac4ce868f2e6ce2afbfbf014aac67653ce4a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500511"
---
# <a name="how-to-create-and-manage-azure-cache-for-redis-using-the-azure-classic-cli"></a>Creación y administración de Azure Cache for Redis mediante la CLI de Azure clásica
> [!div class="op_single_selector"]
> * [PowerShell](cache-how-to-manage-redis-cache-powershell.md)
> * [CLI de Azure clásica](cache-manage-cli.md)
>

La CLI de Azure clásica es una excelente manera de administrar la infraestructura de Azure desde cualquier plataforma. En este artículo se muestra cómo crear y administrar las instancias de Azure Cache for Redis usando la CLI de Azure clásica.

[!INCLUDE [outdated-cli-content](../../includes/contains-classic-cli-content.md)]
> [!NOTE]
> Para conocer los scripts de ejemplo más recientes de la CLI de Azure, consulte [Azure CLI Azure Cache for Redis samples](cli-samples.md) (Ejemplos de instancias de Azure Cache for Redis).

## <a name="prerequisites"></a>Requisitos previos
Para crear y administrar instancias de Azure Cache for Redis mediante la CLI de Azure clásica, debe realizar los pasos siguientes.

* Debe tener una cuenta de Azure. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos momentos.
* [Instale la CLI de Azure clásica](../cli-install-nodejs.md).
* Conecte su instalación de CLI de Azure con una cuenta personal de Azure o con una cuenta de Azure profesional o educativa, e inicie sesión desde la CLI clásica mediante el comando `azure login`.
* Antes de ejecutar cualquiera de los comandos siguientes, cambie la CLI clásica al modo de Administrador de recursos mediante la ejecución del comando `azure config mode arm`. Para más información, consulte [Uso de la CLI de Azure clásica para administrar los recursos y grupos de recursos de Azure](../xplat-cli-azure-resource-manager.md).

## <a name="azure-cache-for-redis-properties"></a>Propiedades de Azure Redis Cache
Las siguientes propiedades se utilizan al crear y actualizar instancias de Azure Cache for Redis.

| Propiedad | Switch | Descripción |
| --- | --- | --- |
| name |-n,--name |Nombre de la instancia de Azure Cache for Redis. |
| resource group |-g, --resource-group |Nombre del grupo de recursos. |
| ubicación |-l, --location |Ubicación donde crear la caché. |
| tamaño |-z, --size |Tamaño de la instancia de Azure Cache for Redis. Valores válidos: [C0, C1, C2, C3, C4, C5, C6, P1, P2, P3, P4] |
| sku |-x, --sku |SKU de Redis. Debe ser uno de estos valores: [Basic, Standard, Premium] |
| EnableNonSslPort |-e, --enable-non-ssl-port |Propiedad EnableNonSslPort de la instancia Azure Cache for Redis. Agregue esta marca si desea habilitar el puerto que no es TLS/SSL de la memoria caché |
| Configuración de Redis |-c, --redis-configuration |Configuración de Redis. Escriba una cadena con formato JSON de valores y claves de configuración aquí. Formato:"{"":"","":""}" |
| Configuración de Redis |-f, --redis-configuration-file |Configuración de Redis. Escriba la ruta de acceso de un archivo que contiene valores y claves de configuración aquí. Formato de la entrada del archivo: {"":"","":""} |
| Número de particiones |-r, --shard-count |Número de particiones que se creará en una caché de clúster premium con la agrupación de clústeres. |
| Virtual Network |-v, --virtual-network |Si hospeda la memoria caché en una red virtual, especifica el id. de recurso de ARM exacto de la red virtual en la que se va a implementar la instancia de Azure Cache for Redis. Formato de ejemplo: /subscriptions/{subid}/resourceGroups/{resourceGroupName}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1 |
| key type |-t, --key-type |Tipo de la clave que renovar. Valores válidos: [Primary, Secondary] |
| StaticIP |-p, --static-ip \<static-ip\> |Si hospeda la memoria caché en una red virtual, especifica una dirección IP única en la subred de la memoria caché. Si no se ofrece, elija una para usted en la subred. |
| Subnet |t, --subnet \<subnet\> |Si hospeda la memoria caché en una red virtual, especifica el nombre de la subred en la que se va a implementar la memoria caché. |
| VirtualNetwork |-v, --virtual-network \<virtual-network\> |Si hospeda la memoria caché en una red virtual, especifica el id. de recurso de ARM exacto de la red virtual en la que se va a implementar la instancia de Azure Cache for Redis. Formato de ejemplo: /subscriptions/{subid}/resourceGroups/{resourceGroupName}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1 |
| Suscripción |-s, --subscription |Identificador de la suscripción. |

## <a name="see-all-azure-cache-for-redis-commands"></a>Ver todos los comandos de Azure Cache for Redis
Para ver todos los comandos de Azure Cache for Redis y sus parámetros, use el comando `azure rediscache -h`.

```azurecli
C:\>azure rediscache -h
help:    Commands to manage your Azure Cache for Redis(s)
help:
help:    Create an Azure Cache for Redis
help:      rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
help:
help:    Delete an existing Azure Cache for Redis
help:      rediscache delete [--name <name> --resource-group <resource-group> ]
help:
help:    List all Azure Cache for Redis within your Subscription or Resource Group
help:      rediscache list [options]
help:
help:    Show properties of an existing Azure Cache for Redis
help:      rediscache show [--name <name> --resource-group <resource-group>]
help:
help:    Change settings of an existing Azure Cache for Redis
help:      rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
help:
help:    Renew the authentication key for an existing Azure Cache for Redis
help:      rediscache renew-key [--name <name> --resource-group <resource-group> ]
help:
help:    Lists Primary and Secondary key of an existing Azure Cache for Redis
help:      rediscache list-keys [--name <name> --resource-group <resource-group>]
help:
help:    Options:
help:      -h, --help  output usage information
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="create-an-azure-cache-for-redis"></a>Creación de una instancia de Azure Redis Cache
Para crear una instancia de Azure Cache for Redis, use el comando siguiente:

```azurecli
    azure rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
```

Para más información sobre este comando, ejecute el comando `azure rediscache create -h`.

```azurecli
C:\>azure rediscache create -h
help:    Create an Azure Cache for Redis
help:
help:    Usage: rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
help:
help:    Options:
help:      -h, --help                                               output usage information
help:      -v, --verbose                                            use verbose output
help:      -vv                                                      more verbose with debug output
help:      --json                                                   use json output
help:      -n, --name <name>                                        Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>                    Name of the Resource Group
help:      -l, --location <location>                                Location to create cache.
help:      -z, --size <size>                                        Size of the Azure Cache for Redis. Valid values: [C0, C1, C2, C3, C4, C5, C6, P1, P2, P3, P4]
help:      -x, --sku <sku>                                          Redis SKU. Should be one of : [Basic, Standard, Premium]
help:      -e, --enable-non-ssl-port                                EnableNonSslPort property of the Azure Cache for Redis. Add this flag if you want to enable the non-TLS/SSL Port for your cache
help:      -c, --redis-configuration <redis-configuration>          Redis Configuration. Enter a JSON formatted string of configuration keys and values here. Format:"{"<key1>":"<value1>","<key2>":"<value2>"}"
help:      -f, --redis-configuration-file <redisConfigurationFile>  Redis Configuration. Enter the path of a file containing configuration keys and values here. Format for the file entry: {"<key1>":"<value1>","<key2>":"<value2>"}
help:      -r, --shard-count <shard-count>                          Number of Shards to create on a Premium Cluster Cache
help:      -v, --virtual-network <virtual-network>                  The exact ARM resource ID of the virtual network to deploy the Azure Cache for Redis in. Example format: /subscriptions/{subid}/resourceGroups/{resourceGroupName}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1
help:      -t, --subnet <subnet>                                    Required when deploying an Azure Cache for Redis inside an existing Azure Virtual Network
help:      -p, --static-ip <static-ip>                              Required when deploying an Azure Cache for Redis inside an existing Azure Virtual Network
help:      -s, --subscription <id>                                  the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="delete-an-existing-azure-cache-for-redis"></a>Eliminación de una instancia de Azure Cache for Redis
Para eliminar una instancia de Azure Cache for Redis, use el comando siguiente:

```azurecli
    azure rediscache delete [--name <name> --resource-group <resource-group> ]
```

Para más información sobre este comando, ejecute el comando `azure rediscache delete -h`.

```azurecli
C:\>azure rediscache delete -h
help:    Delete an existing Azure Cache for Redis
help:
help:    Usage: rediscache delete [--name <name> --resource-group <resource-group> ]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group under which the cache exists
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="list-all-azure-cache-for-redis-within-your-subscription-or-resource-group"></a>Lista de todas las instancias de Azure Cache for Redis de su suscripción o del grupo de recursos
Para enumerar todas las instancias de Azure Cache for Redis incluidas en su suscripción o en el grupo de recursos, use el comando siguiente:

```azurecli
    azure rediscache list [options]
```

Para más información sobre este comando, ejecute el comando `azure rediscache list -h`.

```azurecli
C:\>azure rediscache list -h
help:    List all Azure Cache for Redis within your Subscription or Resource Group
help:
help:    Usage: rediscache list [options]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -g, --resource-group <resource-group>  Name of the Resource Group
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="show-properties-of-an-existing-azure-cache-for-redis"></a>Presentación de las propiedades de una instancia existente de Azure Cache for Redis
Para mostrar las propiedades de una instancia de Azure Cache for Redis, use el comando siguiente:

```azurecli
    azure rediscache show [--name <name> --resource-group <resource-group>]
```

Para más información sobre este comando, ejecute el comando `azure rediscache show -h`.

```azurecli
C:\>azure rediscache show -h
help:    Show properties of an existing Azure Cache for Redis
help:
help:    Usage: rediscache show [--name <name> --resource-group <resource-group>]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

<a name="scale"></a>

## <a name="change-settings-of-an-existing-azure-cache-for-redis"></a>Modifique la configuración de una instancia existente de Azure Cache for Redis
Para modificar la configuración de una instancia de Azure Cache for Redis, use el comando siguiente:

```azurecli
    azure rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
```

Para más información sobre este comando, ejecute el comando `azure rediscache set -h`.

```azurecli
C:\>azure rediscache set -h
help:    Change settings of an existing Azure Cache for Redis
help:
help:    Usage: rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
help:
help:    Options:
help:      -h, --help                                               output usage information
help:      -v, --verbose                                            use verbose output
help:      -vv                                                      more verbose with debug output
help:      --json                                                   use json output
help:      -n, --name <name>                                        Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>                    Name of the Resource Group
help:      -c, --redis-configuration <redis-configuration>          Redis Configuration. Enter a JSON formatted string of configuration keys and values here.
help:      -f, --redis-configuration-file <redisConfigurationFile>  Redis Configuration. Enter the path of a file containing configuration keys and values here.
help:      -s, --subscription <subscription>                        the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="renew-the-authentication-key-for-an-existing-azure-cache-for-redis"></a>Renovación de la clave de autenticación para una instancia de Azure Cache for Redis
Para renovar la clave de autenticación para una instancia de Azure Cache for Redis, use el comando siguiente:

```azurecli
    azure rediscache renew-key [--name <name> --resource-group <resource-group> --key-type <key-type>]
```

Especifique `Primary` o `Secondary` para `key-type`.

Para más información sobre este comando, ejecute el comando `azure rediscache renew-key -h`.

```azurecli
C:\>azure rediscache renew-key -h
help:    Renew the authentication key for an existing Azure Cache for Redis
help:
help:    Usage: rediscache renew-key [--name <name> --resource-group <resource-group> ]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group under which cache exists
help:      -t, --key-type <key-type>              type of key to renew. Valid values are: 'Primary', 'Secondary'.
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="list-primary-and-secondary-keys-of-an-existing-azure-cache-for-redis"></a>Lista de las claves principal y secundaria de una instancia existente de Azure Cache for Redis
Para enumerar las claves Principal y Secundaria de una instancia existente de Azure Cache for Redis, use el comando siguiente:

```azurecli
    azure rediscache list-keys [--name <name> --resource-group <resource-group>]
```

Para más información sobre este comando, ejecute el comando `azure rediscache list-keys -h`.

```azurecli
C:\>azure rediscache list-keys -h
help:    Lists Primary and Secondary key of an existing Azure Cache for Redis
help:
help:    Usage: rediscache list-keys [--name <name> --resource-group <resource-group>]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group under which Cache exists
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```
