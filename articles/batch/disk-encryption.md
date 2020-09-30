---
title: Creación de un grupo con el cifrado de disco habilitado
description: Aprenda a usar la configuración del cifrado de disco para cifrar los nodos con una clave administrada por la plataforma.
author: pkshultz
ms.topic: how-to
ms.date: 08/25/2020
ms.author: peshultz
ms.custom: references_regions
ms.openlocfilehash: 9b0f7f9963ee0edd3986f7ec808a8a4060d857f8
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267058"
---
# <a name="create-a-pool-with-disk-encryption-enabled"></a>Creación de un grupo con el cifrado de disco habilitado

Al crear un grupo de Azure Batch con la configuración de la máquina virtual, puede cifrar los nodos de proceso del grupo con una clave administrada por la plataforma especificando la configuración del cifrado de disco.

En este artículo se explica cómo crear un grupo de Batch con el cifrado de discos habilitado.

## <a name="why-use-a-pool-with-disk-encryption-configuration"></a>¿Por qué usar un grupo con la configuración de cifrado de disco?

Con un grupo de Batch, puede obtener acceso y almacenar datos en el sistema operativo y los discos temporales del nodo de proceso. El cifrado del disco del servidor con una clave administrada por la plataforma protegerá estos datos con una baja sobrecarga y comodidad.  

Batch aplicará una de estas tecnologías de cifrado de discos en nodos de proceso, en función de la configuración del grupo y la compatibilidad regional.

- [Cifrado en reposo con claves administradas por la plataforma](../virtual-machines/windows/disk-encryption.md#platform-managed-keys)
- [Cifrado en el host con una clave administrada por la plataforma](../virtual-machines/windows/disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data)
- [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md)

> [!IMPORTANT]
> La compatibilidad con el cifrado en el host mediante una clave administrada por la plataforma en Azure Batch se encuentra actualmente en versión preliminar pública para las regiones este de EE. UU., Oeste de EE. UU. 2, Centro-sur de EE. UU., US Gov Virginia y US Gov Arizona.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

No podrá especificar qué método de cifrado se aplicará a los nodos del grupo. En su lugar, debe proporcionar los discos de destino que quiere cifrar en sus nodos y Batch puede elegir el método de cifrado adecuado, asegurándose de que los discos especificados se cifren en el nodo de proceso.
 
## <a name="azure-portal"></a>Azure portal 

Al crear un grupo de Batch en Azure Portal, seleccione **TemporaryDisk** o **OsAndTemporaryDisk** en **Configuración de cifrado de disco**.

:::image type="content" source="media/disk-encryption/portal-view.png" alt-text="Captura de pantalla de la opción de configuración de cifrado de disco en Azure Portal":::

Una vez creado el grupo, puede ver los destinos de configuración del cifrado de disco en la sección **Propiedades**.

:::image type="content" source="media/disk-encryption/configuration-target.png" alt-text="Captura de pantalla que muestra la configuración de cifrado de disco en Azure Portal":::

## <a name="examples"></a>Ejemplos

En los siguientes ejemplos se muestra cómo cifrar el sistema operativo y los discos temporales de un grupo de Batch mediante el SDK de .NET para Batch, la API REST de Batch y la CLI de Azure.

### <a name="batch-net-sdk"></a>SDK de Batch para .NET

```csharp
pool.VirtualMachineConfiguration.DiskEncryptionConfiguration = new DiskEncryptionConfiguration(
    targets: new List<DiskEncryptionTarget> { DiskEncryptionTarget.OsDisk, DiskEncryptionTarget.TemporaryDisk }
    );
```

### <a name="batch-rest-api"></a>REST de Batch

URL de la API REST:
```
POST {batchURL}/pools?api-version=2020-03-01.11.0
client-request-id: 00000000-0000-0000-0000-000000000000
```
Cuerpo de la solicitud:
```
"pool": {
    "id": "pool2",
    "vmSize": "standard_a1",
    "virtualMachineConfiguration": {
        "imageReference": {
            "publisher": "Canonical",
            "offer": "UbuntuServer",
            "sku": "18.04-LTS"
        },
        "diskEncryptionConfiguration": {
            "targets": [
                "OsDisk",
                "TemporaryDisk"
            ]
        }
        "nodeAgentSKUId": "batch.node.ubuntu 18.04"
    },
    "resizeTimeout": "PT15M",
    "targetDedicatedNodes": 5,
    "targetLowPriorityNodes": 0,
    "maxTasksPerNode": 3,
    "enableAutoScale": false,
    "enableInterNodeCommunication": false
}
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az batch pool create \
    --id diskencryptionPool \
    --vm-size Standard_DS1_V2 \
    --target-dedicated-nodes 2 \
    --image canonical:ubuntuserver:18.04-LTS \
    --node-agent-sku-id "batch.node.ubuntu 18.04" \
    --disk-encryption-targets OsDisk TemporaryDisk
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre el [cifrado del lado servidor de Azure Disk Storage](../virtual-machines/windows/disk-encryption.md).
- Para obtener información general detallada sobre Batch, consulte [Flujo de trabajo y recursos del servicio Batch](batch-service-workflow-features.md).
