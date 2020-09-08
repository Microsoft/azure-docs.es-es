---
title: Creación de un grupo con direcciones IP públicas
description: Obtenga información sobre cómo crear un grupo de Batch que use sus propias direcciones IP públicas.
ms.topic: how-to
ms.date: 07/20/2020
ms.openlocfilehash: 158facaf1fd5052c3626f065a69bfbd134ca4c3e
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89146494"
---
# <a name="create-an-azure-batch-pool-with-specified-public-ip-addresses"></a>Creación de un grupo de Azure Batch con direcciones IP públicas especificadas

Al crear un grupo de Azure Batch, puede aprovisionarlo en una subred de una [red virtual de Azure](batch-virtual-network.md) que especifique. A las máquinas virtuales del grupo de Batch se accede a través de direcciones IP públicas creadas por Batch. Estas direcciones IP públicas pueden cambiar mientras dure el grupo, lo que significa que la configuración de red puede quedar obsoleta si no se actualizan las direcciones IP.

Puede crear una lista de direcciones IP públicas estáticas para usarlas con las máquinas virtuales del grupo. De este modo, puede controlar la lista de direcciones IP públicas y asegurarse de que no cambiarán de forma inesperada. Esto puede resultar especialmente útil si está trabajando con cualquier servicio externo, como una base de datos, que restringe el acceso a determinadas direcciones IP.

Para información sobre cómo crear grupos sin direcciones IP públicas, consulte [Creación de un grupo de Azure Batch sin direcciones IP públicas](./batch-pool-no-public-ip-address.md).

## <a name="prerequisites"></a>Requisitos previos

- **Autenticación**. Para usar una red dirección IP pública, la API de cliente de Batch debe usar la [autenticación de Azure Active Directory (AD)](batch-aad-auth.md).

- **Una red virtual de Azure**. Debe usar una [red virtual](batch-virtual-network.md) de la misma suscripción de Azure en la que va a crear el grupo y sus direcciones IP. Solo se pueden usar redes virtuales basadas en Azure Resource Manager. Asegúrese de que la red virtual cumple todos los [requisitos generales](batch-virtual-network.md#vnet-requirements).

- **Al menos una dirección IP pública de Azure**. Para crear una o varias direcciones IP públicas, puede usar [Azure Portal](../virtual-network/virtual-network-public-ip-address.md#create-a-public-ip-address), la [interfaz de la línea de comandos (CLI) de Azure](/cli/azure/network/public-ip#az-network-public-ip-create) o [Azure PowerShell](/powershell/module/az.network/new-azpublicipaddress). Asegúrese de seguir los requisitos que se indican a continuación.

> [!NOTE]
> Batch asigna automáticamente los recursos de red adicionales del grupo de recursos que contiene las direcciones IP públicas. Por cada 100 nodos dedicados, Batch generalmente asigna un grupo de seguridad de red (NSG) y un equilibrador de carga. Estos recursos están limitados por las cuotas de recursos de la suscripción. Para los grupos grandes puede que deba [solicitar un aumento de cuota](batch-quota-limit.md#increase-a-quota) para uno o varios de estos recursos.

## <a name="public-ip-address-requirements"></a>Acerca de los requisitos de las direcciones IP

Al crear las direcciones IP públicas, tenga en cuenta los siguientes requisitos:

- Las direcciones IP públicas deben estar en la misma región y suscripción que la cuenta de Batch que se utiliza para crear el grupo.
- La **asignación de dirección IP** debe estar configurada en **Estático**.
- **SKU** se debe establecer en **Estándar**.
- Se debe especificar un nombre de DNS.
- Las direcciones IP públicas deben usarse solo para los grupos de configuración de máquina virtual. Ningún otro recurso debe usar estas direcciones IP, o el grupo puede experimentar errores de asignación.
- Ninguna directiva de seguridad bloqueo de recursos debe restringir el acceso de un usuario a la dirección IP pública.
- El número de direcciones IP públicas especificado para el grupo debe ser lo suficientemente grande como para admitir el número de máquinas virtuales destinadas al grupo. Debe ser al menos la suma de las propiedades **targetDedicatedNodes** y **targetLowPriorityNodes** del grupo. Si no hay suficientes direcciones IP, el grupo asigna parcialmente los nodos de proceso y se producirá un error de cambio de tamaño. Actualmente, Batch usa una dirección IP pública para cada 100 máquinas virtuales.
- Siempre tiene un búfer adicional de direcciones IP públicas. Se recomienda agregar al menos una dirección IP pública adicional, o aproximadamente el 10 % del total de direcciones IP públicas que agregue a un grupo, lo que sea mayor. Este búfer adicional ayudará a Batch con su optimización interna al reducir verticalmente, así como a permitir un escalado más rápido después de un escalado o una reducción vertical con error.
- Una vez creado el grupo, no puede agregar ni cambiar la lista de direcciones IP públicas que usa el grupo. Si necesita modificar la lista, debe eliminar el grupo y volver a crearlo.

## <a name="create-a-batch-pool-with-public-ip-addresses"></a>Creación de un grupo de Batch con direcciones IP públicas

En el ejemplo siguiente se muestra cómo usar la [API REST de servicios de Azure Batch](/rest/api/batchservice/pool/add) para crear un grupo que use direcciones IP públicas.

### <a name="batch-service-rest-api"></a>API REST de servicios de Batch

URI DE LA API REST

```http
POST {batchURL}/pools?api-version=2020-03-01.11.0
client-request-id: 00000000-0000-0000-0000-000000000000
```

Cuerpo de la solicitud

```json
"pool": {
      "id": "pool2",
      "vmSize": "standard_a1",
      "virtualMachineConfiguration": {
        "imageReference": {
          "publisher": "Canonical",
          "offer": "UbuntuServer",
          "sku": "16.04.0-LTS"
        },
        "nodeAgentSKUId": "batch.node.ubuntu 16.04"
      },
"networkConfiguration": {
          "subnetId": "/subscriptions/<subId>/resourceGroups/<rgId>/providers/Microsoft.Network/virtualNetworks/<vNetId>/subnets/<subnetId>",
          "publicIPAddressConfiguration": {
            "provision": "usermanaged",
            "ipAddressIds": [
              "/subscriptions/<subId>/resourceGroups/<rgId>/providers/Microsoft.Network/publicIPAddresses/<publicIpId>"
          ]
        },

       "resizeTimeout":"PT15M",
      "targetDedicatedNodes":5,
      "targetLowPriorityNodes":0,
      "maxTasksPerNode":3,
      "taskSchedulingPolicy": {
        "nodeFillType":"spread"
      }, 
      "enableAutoScale":false,
      "enableInterNodeCommunication":true,
      "metadata": [ {
        "name":"myproperty",
        "value":"myvalue"
      } ]
    }
```

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [flujo de trabajo y los recursos principales del servicio Batch](batch-service-workflow-features.md), como grupos, nodos, trabajos y tareas.
- Obtenga información sobre la [creación un grupo en una subred de una red virtual de Azure](batch-virtual-network.md).
- Obtenga información sobre la [creación de un grupo de Azure Batch sin direcciones IP públicas](./batch-pool-no-public-ip-address.md).

