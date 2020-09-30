---
title: Escalado y reducción vertical de un grupo de servidores de Hiperescala de Azure Database for PostgreSQL mediante la CLI (azdata o kubectl)
description: Escalado y reducción vertical de un grupo de servidores de Hiperescala de Azure Database for PostgreSQL mediante la CLI (azdata o kubectl)
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: b96f38d04fe3e3cb59fa75424ae588fe0e38f510
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932028"
---
# <a name="scale-up-and-down-an-azure-database-for-postgresql-hyperscale-server-group-using-cli-azdata-or-kubectl"></a>Escalado y reducción vertical de un grupo de servidores de Hiperescala de Azure Database for PostgreSQL mediante la CLI (azdata o kubectl)



En algunas ocasiones, es posible que necesite cambiar las características o la definición de un grupo de servidores. Por ejemplo:

- Para escalar o reducir verticalmente el número de núcleos virtuales que usa cada nodo de coordinación o de trabajo.
- Para escalar o reducir verticalmente la memoria que usa cada nodo de coordinación o de trabajo.

En esta guía se explica cómo escalar los núcleos virtuales y la memoria.

El escalado o la reducción vertical de la configuración de la memoria o los núcleos virtuales del grupo de servidores significa que puede establecer un valor mínimo o máximo para cada una de las opciones de configuración de memoria y de núcleos virtuales. Si quiere configurar el grupo de servidores para que use un número específico de núcleos virtuales o una cantidad determinada de memoria, debe establecer los mismos valores mínimos y máximos.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="show-the-current-definition-of-the-server-group"></a>Visualización de la definición actual del grupo de servidores

Para mostrar la definición actual del grupo de servidores y ver cuál es la configuración de los núcleos virtuales y de la memoria en este momento, ejecute cualquiera de los siguientes comandos:

### <a name="cli-with-azdata"></a>CLI con azdata

```console
azdata arc postgres server show -n <server group name>
```
### <a name="cli-with-kubectl"></a>CLI con kubectl

```console
kubectl describe postgresql-12/<server group name> [-n <namespace name>]
```
> [!NOTE]
> Si ha creado un grupo de servidores de la versión 11 de PostgreSQL, ejecute `kubectl describe postgresql-11/<server group name>` en su lugar.

Devuelve la configuración del grupo de servidores. Si ha creado el grupo de servidores con la configuración predeterminada, debería ver una definición como la siguiente:

```console
"scheduling": {
      "default": {
        "resources": {
          "requests": {
            "memory": "256Mi"
          }
        }
      }
    },
```

## <a name="interpret-the-definition-of-the-server-group"></a>Interpretación de la definición del grupo de servidores

En la definición de un grupo de servidores, la sección que contiene la configuración de los núcleos virtuales mínimos y máximos y de la memoria mínima y máxima por nodo es la sección **"scheduling"** (programación). En esta sección, los valores máximos se conservan en una subsección denominada **"limits"** (límites), y los mínimos, en una subsección denominada **"requests"** (solicitudes).

Si establece un valor mínimo distinto del máximo, esta configuración garantizará que al grupo de servidores se le asignan los recursos solicitados que necesita. No superará los límites establecidos.

Los recursos (los núcleos virtuales y la memoria) que usará realmente el grupo de servidores dependerán de los valores máximos configurados y de las cargas de trabajo y los recursos disponibles en el clúster. Si no se limitan los valores con un máximo, el grupo de servidores puede llegar a usar todos los recursos que el clúster de Kubernetes asigna a los nodos de Kubernetes en los que está programado el grupo de servidores.

Esos valores de los núcleos virtuales y de la memoria se aplican a cada uno de los nodos (tanto de coordinación como de trabajo) de Hiperescala de PostgreSQL. Todavía no es posible establecer las definiciones del nodo de coordinación y de los nodos de trabajo por separado.

En una configuración predeterminada, solo la memoria mínima se establece en 256 MiB, ya que es la cantidad mínima de memoria que se recomienda para ejecutar Hiperescala de PostgreSQL.

> [!NOTE]
> La configuración de un valor mínimo no significa que el grupo de servidores usará ese mínimo. Significa que, si el grupo de servidores lo necesita, seguro que se le asignará al menos este mínimo. Por ejemplo, imaginemos que hemos establecido `--minCpu 2`. Eso no significa que el grupo de servidores usará al menos dos núcleos virtuales en todo momento, sino que puede empezar a usar menos de dos núcleos virtuales si no necesita más, y garantiza que se le asignarán al menos dos núcleos virtuales si los necesita más adelante. Esto implica que el clúster de Kubernetes asigna los recursos a otras cargas de trabajo de forma que pueda asignar esos dos núcleos virtuales al grupo de servidores si es necesario.

>[!NOTE]
>Antes de modificar la configuración del sistema, asegúrese de familiarizarse con el modelo de recursos de Kubernetes [aquí](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities).

## <a name="scale-up-the-server-group"></a>Escalado vertical del grupo de servidores

Tenga en cuenta la configuración especificada para el clúster de Kubernetes a la hora de establecer los valores mínimos y máximos. Asegúrese de que no está estableciendo valores que el clúster de Kubernetes no puede satisfacer. Esto podría provocar errores o un comportamiento imprevisible. Por ejemplo, si el estado del grupo de servidores permanece como _actualizando_ durante mucho tiempo después de cambiar la configuración, esto puede indicar que ha establecido los parámetros siguientes en valores que el clúster de Kubernetes no puede satisfacer. En ese caso, revierta el cambio o consulte la sección de solución de problemas.

Supongamos que quiere escalar verticalmente la definición del grupo de servidores a lo siguiente:

- Núcleos virtuales mínimos = 2
- Núcleos virtuales máximos = 4
- Memoria mínima = 512 MB
- Memoria máxima = 1 GB

Puede usar cualquiera de los enfoques siguientes:

### <a name="cli-with-azdata"></a>CLI con azdata

```console
azdata arc postgres server edit -n <name of your server group> --cores-request 2  --cores-limit 4  --memory-request 512Mi  --memory-limit 1024Mi
```

El comando se ejecuta correctamente cuando se muestra lo siguiente:

```console
<name of your server group> is Ready
```

> [!NOTE]
> Para obtener más información sobre estos parámetros, ejecute `azdata arc postgres server edit --help`.

### <a name="cli-with-kubectl"></a>CLI con kubectl

```console
kubectl edit postgresql-12/<server group name> [-n <namespace name>]
```

Esto le llevará al editor vi, donde podrá navegar y cambiar la configuración. Use lo siguiente para asignar el valor deseado al nombre del campo en la especificación:

- Núcleos virtuales mínimos = 2-> scheduling\default\resources\requests\cpu
- Núcleos virtuales máximos = 4-> scheduling\default\resources\limits\cpu
- Memoria mínima = 512 MB -> scheduling\default\resources\requests\cpu
- Memoria máxima = 1 GB -> scheduling\default\resources\limits\cpu

Si no está familiarizado con el editor vi, puede ver una descripción de los comandos que puede necesitar [aquí](https://www.computerhope.com/unix/uvi.htm):
- Modo de edición: `i`
- Desplazarse: flechas
- Detener la edición: `esc`
- Salir sin guardar: `:qa!`
- Salir después de guardar: `:qw!`


## <a name="show-the-scaled-up-definition-of-the-server-group"></a>Visualización de la definición del escalado vertical del grupo de servidores

Vuelva a ejecutar el comando para mostrar la definición del grupo de servidores y comprobar que está configurado como quiere:

### <a name="cli-with-azdata"></a>CLI con azdata

```console
azdata arc postgres server show -n <the name of your server group>
```
### <a name="cli-with-kubectl"></a>CLI con kubectl

```console
kubectl describe postgresql-12/<server group name>  [-n <namespace name>]
```
> [!NOTE]
> Si ha creado un grupo de servidores de la versión 11 de PostgreSQL, ejecute `kubectl describe postgresql-11/<server group name>` en su lugar.


Se mostrará la nueva definición del grupo de servidores:

```console
"scheduling": {
      "default": {
        "resources": {
          "limits": {
            "cpu": "4",
            "memory": "1024Mi"
          },
          "requests": {
            "cpu": "2",
            "memory": "512Mi"
          }
        }
      }
    },
```

## <a name="scale-down-the-server-group"></a>Reducción vertical del grupo de servidores

Para reducir verticalmente el grupo de servidores, ejecute el mismo comando, pero establezca valores inferiores para las configuraciones que quiere reducir verticalmente. Para quitar las solicitudes o los límites, especifique su valor como una cadena vacía.

## <a name="next-steps"></a>Pasos siguientes

- [Escalado horizontal del grupo de servidores de Hiperescala de Azure Database for PostgreSQL](scale-out-postgresql-hyperscale-server-group.md)
- [Conceptos de almacenamiento de Kubernetes y configuración de almacenamiento](storage-configuration.md)
- [Expansión de notificaciones de volúmenes persistentes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)
- [Modelo de recursos de Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)
