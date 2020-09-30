---
title: Creación de una instancia de Azure HPC Cache
description: Creación de una instancia de Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 09/03/2020
ms.author: v-erkel
ms.openlocfilehash: 5b1062556f1f971690f835274be15c11b072eca9
ms.sourcegitcommit: f845ca2f4b626ef9db73b88ca71279ac80538559
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89612064"
---
# <a name="create-an-azure-hpc-cache"></a>Creación de una instancia de Azure HPC Cache

Use Azure Portal o la CLI de Azure para crear la memoria caché.

![captura de pantalla de información general de caché en Azure Portal, con el botón crear en la parte inferior](media/hpc-cache-home-page.png)

Haga clic en la imagen siguiente para ver una [demostración en vídeo](https://azure.microsoft.com/resources/videos/set-up-hpc-cache/) sobre cómo crear una caché y agregar un destino de almacenamiento.

[![Miniatura de vídeo: Azure HPC Cache: Instalación (haga clic para visitar la página del vídeo)](media/video-4-setup.png)](https://azure.microsoft.com/resources/videos/set-up-hpc-cache/)

## <a name="portal"></a>[Portal](#tab/azure-portal)

## <a name="define-basic-details"></a>Definición de los detalles básicos

![captura de pantalla de la página de detalles del proyecto en Azure Portal](media/hpc-cache-create-basics.png)

En **Detalles del proyecto**, seleccione la suscripción y el grupo de recursos que hospedará la caché.<!-- Make sure the subscription is on the [access](hpc-cache-prerequisites.md#azure-subscription) list.  -->

En **Detalles del servicio**, establezca el nombre de la memoria caché y estos otros atributos:

* Ubicación: seleccione alguna de las [regiones admitidas](hpc-cache-overview.md#region-availability).
* Red virtual: puede optar por crear una nueva red virtual o seleccionar una existente.
* Subred: elija o cree una subred con al menos 64 direcciones IP (/24). Esta subred solo se debe usar para esta instancia de Azure HPC Cache.

## <a name="set-cache-capacity"></a>Establecimiento de la capacidad de la memoria caché
<!-- referenced from GUI - update aka.ms link if you change this header text -->

En la página **Caché**, debe establecer la capacidad de la memoria caché. Estos valores determinan la cantidad de datos que puede contener la memoria caché y la rapidez con la que puede atender las solicitudes de cliente.

La capacidad también afecta al costo de la caché.

Para elegir la capacidad, establezca estos dos valores:

* La velocidad de transferencia de datos máxima para la memoria caché (rendimiento), en GB/segundo
* La cantidad de almacenamiento asignado a los datos en caché, en TB

Elija uno de los valores de rendimiento disponibles y los tamaños de almacenamiento en caché.

Tenga en cuenta que la velocidad de transferencia de datos real depende de la carga de trabajo, las velocidades de red y el tipo de destinos de almacenamiento. Los valores que elija establecen el rendimiento máximo para todo el sistema de caché, pero una parte se usa para tareas de sobrecarga. Por ejemplo, si un cliente solicita un archivo que aún no está almacenado en la memoria caché, o si el archivo está marcado como obsoleto, la memoria caché utiliza parte de su capacidad de proceso para recuperarlo del almacenamiento back-end.

Azure HPC Cache administra qué archivos se almacenan en caché y se cargan previamente para maximizar las tasas de aciertos de caché. El contenido de la caché se evalúa continuamente y los archivos se mueven al almacenamiento a largo plazo cuando se accede a ellos con menos frecuencia. Elija un tamaño de almacenamiento en caché que pueda contener el conjunto activo de archivos de trabajo con espacio adicional para los metadatos y otras sobrecargas.

![captura de pantalla de la página de tamaño de caché](media/hpc-cache-create-capacity.png)

## <a name="enable-azure-key-vault-encryption-optional"></a>Habilitación del cifrado de Azure Key Vault (opcional)

Si su memoria caché se encuentra en una región que admite claves de cifrado administradas por el cliente, aparece la página **Claves de cifrado de disco** entre las pestañas **Caché** y **Etiquetas**. Lea [Disponibilidad regional](hpc-cache-overview.md#region-availability) para obtener más información sobre la compatibilidad con regiones.

Si desea administrar las claves de cifrado usadas para el almacenamiento en caché, proporcione la información de Azure Key Vault en la página **Claves de cifrado de disco**. El almacén de claves debe estar en la misma región y en la misma suscripción que la memoria caché.

Puede omitir esta sección si no necesita claves administradas por el cliente. De manera predeterminada, Azure cifra los datos con claves administradas por Microsoft. Para obtener más información, lea [Cifrado de Azure Storage](../storage/common/storage-service-encryption.md).

> [!NOTE]
>
> * No puede cambiar entre claves administradas por Microsoft y claves administradas por el cliente después de crear la memoria caché.
> * Una vez creada la memoria caché, debe autorizarla para que tenga acceso al almacén de claves. Haga clic en el botón **Habilitar cifrado** en la página **Información general** de la memoria caché para activar el cifrado. Complete este paso en un plazo de 90 minutos a partir de la creación de la memoria caché.
> * Los discos de caché se crean después de esta autorización. Esto significa que el tiempo de creación de la memoria caché inicial es breve, pero la memoria caché no estará lista para usarse durante diez minutos o más después de que autorice el acceso.

Para obtener una explicación completa del proceso de cifrado con claves administradas por el cliente, lea [Uso de claves de cifrado administradas por el cliente para Azure HPC Cache](customer-keys.md).

![Captura de pantalla de la página Claves de cifrado con "Administrado por el cliente" y los campos del almacén de claves](media/create-encryption.png)

Seleccione **Administrado por el cliente** para elegir el cifrado con claves administradas por el cliente. Aparecen los campos de especificación del almacén de claves. Seleccione la instancia de Azure Key Vault que se va a usar y, a continuación, seleccione la clave y la versión que se usarán para esta caché. Debe ser una clave RSA de 2048 bits. Puede crear un nuevo almacén de claves, clave o versión de clave desde esta página.

Después de crear la memoria caché, debe autorizarla para que use el servicio de almacén de claves. Para obtener más información, lea [Autorización del cifrado de Azure Key Vault desde la memoria caché](customer-keys.md#3-authorize-azure-key-vault-encryption-from-the-cache).

## <a name="add-resource-tags-optional"></a>Incorporación de etiquetas de recursos (opcional)

La página **Etiquetas** permiten agregar [etiquetas de recursos](https://go.microsoft.com/fwlink/?linkid=873112) a Azure HPC Cache.

## <a name="finish-creating-the-cache"></a>Terminar de crear la memoria caché

Después de configurar la nueva memoria caché, haga clic en la pestaña **Revisar y crear**. El portal valida las selecciones y le permite revisar las opciones. Si todo es correcto, haga clic en **Crear**.

La creación de la memoria caché tarda unos 10 minutos. Puede realizar el seguimiento del progreso en las notificaciones de Azure Portal.

![Captura de pantalla de las páginas de "implementación en curso" y "notificaciones" de creación de la memoria caché en el portal](media/hpc-cache-deploy-status.png)

Cuando finaliza la creación, aparece una notificación con un vínculo a la nueva instancia de Azure HPC Cache y la memoria caché aparece en la lista **Recursos** de la suscripción.

![captura de pantalla de la instancia de Azure HPC Cache en Azure Portal](media/hpc-cache-new-overview.png)

> [!NOTE]
> Si la memoria caché usa claves de cifrado administradas por el cliente, la memoria caché podría aparecer en la lista de recursos antes de que el estado de implementación cambie a Completo. Tan pronto como el estado de la memoria caché sea **Waiting for key** (Esperando la clave), puede [autorizarla](customer-keys.md#3-authorize-azure-key-vault-encryption-from-the-cache) para usar el almacén de claves.

## <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

## <a name="create-the-cache-with-azure-cli"></a>Creación de la memoria caché con la CLI de Azure

[!INCLUDE [cli-reminder.md](includes/cli-reminder.md)]

> [!NOTE]
> Actualmente, la CLI de Azure no admite la creación de una caché con claves de cifrado administradas por el cliente. Use Azure Portal.

Use el comando [az hpc-cache create](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-create) para crear una nueva instancia de Azure HPC Cache.

Especifique estos valores:

* Nombre del grupo de recursos de la caché
* Nombre de la caché
* Región de Azure
* Subred de la caché, con este formato:

  ``--subnet "/subscriptions/<subscription_id>/resourceGroups/<cache_resource_group>/providers/Microsoft.Network/virtualNetworks/<virtual_network_name>/sub
nets/<cache_subnet_name>"``

  La subred de la caché necesita al menos 64 direcciones IP (/24) y no puede hospedar ningún otro recurso.

* Capacidad de la caché. Dos valores establecen el rendimiento máximo de Azure HPC Cache:

  * El tamaño de la caché (en GB)
  * La SKU de las máquinas virtuales que se usan en la infraestructura de caché

  [az hpc-cache skus list](/cli/azure/ext/hpc-cache/hpc-cache/skus) muestra las SKU disponibles y las opciones de tamaño de caché válidas para cada una. Las opciones de tamaño de caché van de 3 TB a 48 TB, pero solo se admiten algunos valores.

  En este gráfico se muestra qué combinaciones de tamaño de caché y SKU son válidas en el momento en que se ha preparado este documento (julio de 2020).

  | Tamaño de memoria caché | Standard_2G | Standard_4G | Standard_8G |
  |------------|-------------|-------------|-------------|
  | 3072 GB    | sí         | no          | No          |
  | 6144 GB    | sí         | sí         | No          |
  | 12288 GB   | sí         | sí         | sí         |
  | 24576 GB   | No          | sí         | sí         |
  | 49152 GB   | No          | No          | sí         |

  Lea la sección **Establecimiento de la capacidad de la memoria caché** en la pestaña de instrucciones del portal para obtener información importante sobre los precios, el rendimiento y la definición del tamaño adecuado de la caché para su flujo de trabajo.

Ejemplo de creación de una caché:

```azurecli
az hpc-cache create --resource-group doc-demo-rg --name my-cache-0619 \
    --location "eastus" --cache-size-gb "3072" \
    --subnet "/subscriptions/<subscription-ID>/resourceGroups/doc-demo-rg/providers/Microsoft.Network/virtualNetworks/vnet-doc0619/subnets/default" \
    --sku-name "Standard_2G"
```

El proceso de creación tarda varios minutos. Si se ejecuta correctamente, el comando create devuelve una salida similar a la siguiente:

```azurecli
{
  "cacheSizeGb": 3072,
  "health": {
    "state": "Healthy",
    "statusDescription": "The cache is in Running state"
  },
  "id": "/subscriptions/<subscription-ID>/resourceGroups/doc-demo-rg/providers/Microsoft.StorageCache/caches/my-cache-0619",
  "location": "eastus",
  "mountAddresses": [
    "10.3.0.17",
    "10.3.0.18",
    "10.3.0.19"
  ],
  "name": "my-cache-0619",
  "provisioningState": "Succeeded",
  "resourceGroup": "doc-demo-rg",
  "sku": {
    "name": "Standard_2G"
  },
  "subnet": "/subscriptions/<subscription-ID>/resourceGroups/doc-demo-rg/providers/Microsoft.Network/virtualNetworks/vnet-doc0619/subnets/default",
  "tags": null,
  "type": "Microsoft.StorageCache/caches",
  "upgradeStatus": {
    "currentFirmwareVersion": "5.3.42",
    "firmwareUpdateDeadline": "0001-01-01T00:00:00+00:00",
    "firmwareUpdateStatus": "unavailable",
    "lastFirmwareUpdate": "2020-04-01T15:19:54.068299+00:00",
    "pendingFirmwareVersion": null
  }
}
```

El mensaje incluye alguna información útil, incluidos estos elementos:

* Direcciones de montaje del cliente: Use estas direcciones IP cuando esté listo para conectar clientes a la memoria caché. Para obtener más información, lea [Montaje de la instancia de Azure HPC Cache](hpc-cache-mount.md).
* Estado de actualización: Cuando se publique una actualización de software, este mensaje cambiará. Puede [actualizar el software de la caché](hpc-cache-manage.md#upgrade-cache-software) manualmente en un momento conveniente, o bien se aplicará automáticamente después de varios días.

---

## <a name="next-steps"></a>Pasos siguientes

Después de que la memoria caché aparezca en la lista **Recursos**, puede avanzar al paso siguiente.

* [Defina los destinos de almacenamiento](hpc-cache-add-storage.md) para otorgar a la memoria caché acceso a los orígenes de datos.
* Si usa claves de cifrado administradas por el cliente, debe [autorizar el cifrado de Azure Key Vault](customer-keys.md#3-authorize-azure-key-vault-encryption-from-the-cache) en la página de información general de la memoria caché para completar la configuración de la memoria caché. Debe completar este paso para poder agregar almacenamiento. Para obtener más información, lea [Uso de claves de cifrado administradas por el cliente](customer-keys.md).
