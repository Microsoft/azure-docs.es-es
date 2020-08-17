---
title: Uso de Shared Image Gallery para crear un grupo de imágenes personalizadas
description: Los grupos de imágenes personalizadas son una manera eficaz de configurar los nodos de proceso para ejecutar las cargas de trabajo de Batch.
ms.topic: conceptual
ms.date: 07/01/2020
ms.custom: devx-track-python
ms.openlocfilehash: aad8b279ce821496d4c947bc7f9c707243468f07
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852419"
---
# <a name="use-the-shared-image-gallery-to-create-a-custom-image-pool"></a>Uso de Shared Image Gallery para crear un grupo de imágenes personalizadas

Al crear un grupo en Azure Batch con Configuración de máquina virtual, se especifica una imagen de máquina virtual (VM) que proporciona el sistema operativo para cada nodo de proceso en el grupo. Puede crear un grupo de máquinas virtuales con una imagen de Azure Marketplace compatible o crear una imagen personalizada con una [imagen de Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md).

## <a name="benefits-of-the-shared-image-gallery"></a>Ventajas de Shared Image Gallery

Al usar Shared Image Gallery para la imagen personalizada, tiene el control sobre la configuración y el tipo del sistema operativo, así como el tipo de discos de datos. La imagen de Shared Image puede incluir los datos de referencia y las aplicaciones que están disponibles en los nodo de grupo de Batch en cuanto se aprovisionan.

También puede tener varias versiones de una imagen según sea necesario para su entorno. Cuando se usa una versión de la imagen para crear una VM, la versión de la imagen se usa para crear nuevos discos para la VM.

El uso de una imagen de Shared Image le ahorra tiempo a la hora de preparar los nodos de ejecución del grupo para ejecutar la carga de trabajo de Batch. Se puede usar una imagen de Azure Marketplace e instalar el software en cada nodo de ejecución una vez que se haya aprovisionado, pero el uso de una imagen de Shared Image podría ser más eficiente. Además, puede especificar varias réplicas para la imagen de Shared Image, por lo que al crear grupos con muchas VM (más de 600), ahorrará tiempo en la creación del grupo.

El uso de una imagen de Shared Image configurada para su escenario puede proporcionar varias ventajas:

- **Uso de las mismas imágenes en todas las regiones.** Puede crear réplicas de imágenes de Shared Image en distintas regiones para que todos los grupos usen la misma imagen.
- **Configuración del sistema operativo (SO).** La configuración del disco de sistema operativo de la imagen se puede personalizar.
- **Preinstalar las aplicaciones.** Preinstalar las aplicaciones en el disco del sistema operativo es más eficaz y menos propenso a errores que instalar las aplicaciones después de aprovisionar los nodos de ejecución con una tarea de inicio.
- **Copia de grandes cantidades de datos una vez.** Convierta en estáticos parte de los datos de la imagen de Shared Image administrada copiándolos en los discos de datos de una imagen administrada. Solo debe hacerse una vez y los datos estarán disponibles para cada nodo del grupo.
- **Aumento de los grupos a tamaños más grandes.** Con Shared Image Gallery puede crear grupos más grandes con sus imágenes personalizadas junto con más réplicas de Shared Image.
- **Mejor rendimiento que el uso de una imagen administrada como imagen personalizada.** Para un grupo de imágenes personalizadas de Shared Image, el tiempo necesario para alcanzar el estado estable es hasta un 25 % más rápido y la latencia de inactividad de la máquina virtual es hasta un 30 % más breve.
- **Control de versiones y agrupación de imágenes para facilitar su administración.** La definición de la agrupación de imágenes contiene información acerca de por qué se creó la imagen, para qué sistema operativo sirve e información acerca del uso de la imagen. La agrupación de imágenes permite una administración más sencilla de las imágenes. Para más información, consulte [Definiciones de imagen](../virtual-machines/windows/shared-image-galleries.md#image-definitions).

## <a name="prerequisites"></a>Requisitos previos

> [!NOTE]
> Debe autenticarse mediante Azure AD. Si usa shared-key-auth, obtendrá un error de autenticación.  

- **Una cuenta de Azure Batch** Para crear una cuenta de Batch, consulte las guías de inicio rápido de Batch con [Azure Portal](quick-create-portal.md) o la [CLI de Azure](quick-create-cli.md).

- **Una imagen de Shared Image Gallery**. Para crear una imagen de Shared Image Gallery, debe tener o crear un recurso de imagen administrada. La imagen debe crearse desde instantáneas del disco del sistema operativo de la máquina virtual y, opcionalmente, de sus discos de datos conectados.

> [!NOTE]
> La imagen de Shared Image debe estar en la misma suscripción que la cuenta de Batch. La imagen puede estar en distintas regiones, siempre que tenga réplicas en la misma región que la cuenta de Batch.

Si usa una aplicación de Azure AD para crear un grupo de imágenes personalizadas con una imagen de Shared Image Gallery, se debe haber concedido a la aplicación un [rol integrado de Azure](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) que le proporcione acceso a Shared Image. Para conceder este acceso en Azure Portal, navegue a la imagen compartida, seleccione **Control de acceso (IAM)** y agregue una asignación de roles para la aplicación.

## <a name="prepare-a-shared-image"></a>Preparación de una imagen compartida

En Azure, puede preparar una imagen compartida a partir de una imagen administrada, que se puede crear a partir de:

- Instantáneas del sistema operativo y los discos de datos de una máquina virtual de Azure
- Una máquina virtual de Azure generalizada con discos administrados
- Un disco duro virtual local generalizado cargado en la nube

> [!NOTE]
> Actualmente, Batch solo admite imágenes compartidas generalizadas. En este momento no se puede crear un grupo de imágenes personalizado a partir de una imagen compartida especializada.

Consulte los siguientes pasos para preparar la máquina virtual, tome una instantánea y cree una imagen a partir de ella.

### <a name="prepare-a-vm"></a>Preparación de la máquina virtual

Si va a crear una máquina virtual para la imagen, use una imagen propia de Azure Marketplace admitida por Batch como imagen base para la imagen administrada. Solo pueden usarse imágenes propias como imagen base. Para ver una lista completa de las referencias de imagen de Azure Marketplace compatibles con Azure Batch, consulte la operación [List node agent SKUs](/java/api/com.microsoft.azure.batch.protocol.accounts.listnodeagentskus).

> [!NOTE]
> No se puede usar una imagen de terceros que tenga licencias adicionales y términos de compra como imagen base. Para más información sobre estas imágenes de Azure Marketplace, consulte las instrucciones para las máquinas virtuales [Linux](../virtual-machines/linux/cli-ps-findimage.md#deploy-an-image-with-marketplace-terms) o [Windows](../virtual-machines/windows/cli-ps-findimage.md#deploy-an-image-with-marketplace-terms).

- Asegúrese de que la máquina virtual se crea con un disco administrado. Se trata de la configuración de almacenamiento predeterminada cuando se crea una máquina virtual.
- No instale extensiones de Azure, como la extensión de script personalizado, en la máquina virtual. Si la imagen contiene una extensión preinstalada, Azure podría experimentar problemas al implementar el grupo de Batch.
- Cuando use discos de datos conectados, debe montar y dar formato a los discos desde una máquina virtual para usarlos.
- Asegúrese de que la imagen del sistema operativo base que proporcione usa la unidad temporal predeterminada. El agente de nodo de Batch actualmente espera la unidad temporal predeterminada.
- Una vez que la máquina virtual está en ejecución, conéctese a ella a través de RDP (para Windows) o SSH (para Linux). Instale el software necesario o copie los datos deseados.  

### <a name="create-a-vm-snapshot"></a>Creación de una instantánea de máquina virtual

Una instantánea es una copia completa de solo lectura de un disco duro virtual. Para crear una instantánea del sistema operativo de una máquina virtual o de los discos de datos, puede usar Azure Portal o herramientas de la línea de comandos. En cuanto a pasos y opciones de creación de una instantánea, consulte la guía para máquinas virtuales [Linux](../virtual-machines/linux/snapshot-copy-managed-disk.md) o [Windows](../virtual-machines/windows/snapshot-copy-managed-disk.md).

### <a name="create-an-image-from-one-or-more-snapshots"></a>Creación de una imagen a partir de una o varias instantáneas

Para crear una imagen administrada a partir de una instantánea, use las herramientas de la línea de comandos de Azure, como el comando [az image create](/cli/azure/image). Cree una imagen mediante la especificación de una instantánea del disco del sistema operativo y, opcionalmente, una o varias instantáneas de disco de datos.

### <a name="create-a-shared-image-gallery"></a>Creación de una instancia de Shared Image Gallery

Una vez que haya creado correctamente la imagen administrada, debe crear una instancia de Shared Image Gallery para que la imagen personalizada esté disponible. Para más información acerca de cómo crear una instancia de Shared Image Gallery para sus imágenes, consulte [Creación de una imagen de Shared Image Gallery con la CLI de Azure](../virtual-machines/shared-images-cli.md) o [Creación de una imagen de Shared Image Gallery mediante Azure Portal](../virtual-machines/linux/shared-images-portal.md).

## <a name="create-a-pool-from-a-shared-image-using-the-azure-cli"></a>Creación de un grupo a partir de una imagen de Shared Image con la CLI de Azure

Para crear un grupo a partir de la imagen de Shared Image mediante la CLI de Azure, use el comando `az batch pool create`. Especifique el identificador de la imagen de Shared Image en el campo `--image`. Asegúrese de que el tipo de sistema operativo y la SKU coinciden con las versiones especificadas por `--node-agent-sku-id`.

> [!NOTE]
> Debe autenticarse mediante Azure AD. Si usa shared-key-auth, obtendrá un error de autenticación.  

```azurecli
az batch pool create \
    --id mypool --vm-size Standard_A1_v2 \
    --target-dedicated-nodes 2 \
    --image "/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/galleries/{gallery name}/images/{image definition name}/versions/{version id}" \
    --node-agent-sku-id "batch.node.ubuntu 16.04"
```

## <a name="create-a-pool-from-a-shared-image-using-c"></a>Creación de un grupo a partir de una imagen de Shared Image mediante C#

Como alternativa, puede crear un grupo a partir de una imagen de Shared Image mediante el SDK de C#.

```csharp
private static VirtualMachineConfiguration CreateVirtualMachineConfiguration(ImageReference imageReference)
{
    return new VirtualMachineConfiguration(
        imageReference: imageReference,
        nodeAgentSkuId: "batch.node.windows amd64");
}

private static ImageReference CreateImageReference()
{
    return new ImageReference(
        virtualMachineImageId: "/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/galleries/{gallery name}/images/{image definition name}/versions/{version id}");
}

private static void CreateBatchPool(BatchClient batchClient, VirtualMachineConfiguration vmConfiguration)
{
    try
    {
        CloudPool pool = batchClient.PoolOperations.CreatePool(
            poolId: PoolId,
            targetDedicatedComputeNodes: PoolNodeCount,
            virtualMachineSize: PoolVMSize,
            virtualMachineConfiguration: vmConfiguration);

        pool.Commit();
    }
    ...
}
```

## <a name="create-a-pool-from-a-shared-image-using-python"></a>Creación de un grupo a partir de una imagen compartida mediante Python

También puede crear un grupo a partir de una imagen compartida mediante el SDK de Python: 

```python
# Import the required modules from the
# Azure Batch Client Library for Python
import azure.batch as batch
import azure.batch.models as batchmodels
from azure.common.credentials import ServicePrincipalCredentials

# Specify Batch account and service principal account credentials
account = "{batch-account-name}"
batch_url = "{batch-account-url}"
ad_client_id = "{sp-client-id}"
ad_tenant = "{tenant-id}"
ad_secret = "{sp-secret}"

# Pool settings
pool_id = "LinuxNodesSamplePoolPython"
vm_size = "STANDARD_D2_V3"
node_count = 1

# Initialize the Batch client with Azure AD authentication
creds = ServicePrincipalCredentials(
    client_id=ad_client_id,
    secret=ad_secret,
    tenant=ad_tenant,
    resource="https://batch.core.windows.net/"
)
client = batch.BatchServiceClient(creds, batch_url)

# Configure the start task for the pool
start_task = batchmodels.StartTask(
    command_line="printenv AZ_BATCH_NODE_STARTUP_DIR"
)
start_task.run_elevated = True

# Create an ImageReference which specifies the image from
# Shared Image Gallery to install on the nodes.
ir = batchmodels.ImageReference(
    virtual_machine_image_id="/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/galleries/{gallery name}/images/{image definition name}/versions/{version id}"
)

# Create the VirtualMachineConfiguration, specifying
# the VM image reference and the Batch node agent to
# be installed on the node.
vmc = batchmodels.VirtualMachineConfiguration(
    image_reference=ir,
    node_agent_sku_id="batch.node.ubuntu 18.04"
)

# Create the unbound pool
new_pool = batchmodels.PoolAddParameter(
    id=pool_id,
    vm_size=vm_size,
    target_dedicated_nodes=node_count,
    virtual_machine_configuration=vmc,
    start_task=start_task
)

# Create pool in the Batch service
client.pool.add(new_pool)
```

## <a name="create-a-pool-from-a-shared-image-using-the-azure-portal"></a>Creación de una imagen compartida mediante Azure Portal

Use los pasos siguientes para crear un grupo a partir de una imagen compartida en Azure Portal.

1. Abra [Azure Portal](https://portal.azure.com).
1. Vaya a **Cuentas de Batch** y seleccione su cuenta.
1. Seleccione **Grupos** y, luego, **Agregar** para crear un grupo.
1. En la sección **Tipo de imagen**, seleccione **Shared Image Gallery**.
1. Complete el resto de secciones con información sobre la imagen administrada.
1. Seleccione **Aceptar**.

![Cree un grupo a partir de una imagen compartida con el portal.](media/batch-sig-images/create-custom-pool.png)

## <a name="considerations-for-large-pools"></a>Consideraciones sobre grupos grandes

Si tiene previsto crear un grupo con cientos o miles de VM o más mediante una imagen de Shared Image, use la siguiente guía.

- **Números de réplicas de Shared Image Gallery.**  Para cada grupo con hasta 600 instancias, se recomienda conservar al menos una réplica. Por ejemplo, si va a crear un grupo con 3000 VM, debe mantener al menos cinco réplicas de la imagen. Siempre se recomienda mantener más réplicas que las indicadas por los requisitos mínimos para un mejor rendimiento.

- **Tiempo de espera del cambio de tamaño.** Si el grupo contiene un número de nodos fijo (no se escala automáticamente), aumente la propiedad `resizeTimeout` del grupo en función de su tamaño. Por cada 1000 VM, el tiempo de espera del cambio de tamaño recomendado es de al menos 15 minutos. Por ejemplo, el tiempo de espera del cambio de tamaño recomendado para un grupo con 2000 VM es de al menos 30 minutos.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general detallada sobre Batch, consulte [Recursos y flujo de trabajo del servicio de Batch](batch-service-workflow-features.md).
- Obtenga información sobre [Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md).
