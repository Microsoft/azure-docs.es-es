---
title: Servicio de metadatos de instancia de Azure
description: Interfaz RESTful para obtener información sobre proceso, red y próximos eventos de mantenimiento de la máquina virtual.
services: virtual-machines
author: KumariSupriya
manager: paulmey
ms.service: virtual-machines
ms.subservice: monitoring
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 04/29/2020
ms.author: sukumari
ms.reviewer: azmetadatadev
ms.openlocfilehash: 4f0e9d057c92f1907bb77ee0767c7bb07f0f4c62
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836996"
---
# <a name="azure-instance-metadata-service-imds"></a>Azure Instance Metadata Service (IMDS)

Instance Metadata Service (IMDS) de Azure proporciona información sobre instancias de máquina virtual actualmente en ejecución y se puede usar para administrar y configurar las máquinas virtuales.
Esta información incluye las SKU, almacenamiento, configuraciones de red y próximos eventos de mantenimiento. Para una lista completa de los datos que están disponibles, consulte [API de metadatos](#metadata-apis).
Instance Metadata Service está disponible para las instancias de máquinas virtuales y los conjuntos de escalado de máquinas virtuales. Solo está disponible para ejecutar máquinas virtuales creadas o administradas mediante [Azure Resource Manager](/rest/api/resources/).

IMDS de Azure es un punto de conexión REST que está disponible en una dirección IP no enrutable conocida (`169.254.169.254`); solo se puede acceder desde dentro de la máquina virtual. La comunicación entre la máquina virtual y IMDS nunca sale del host.
Se recomienda que los clientes HTTP omitan los servidores proxy web dentro de la VM al consultar a IMDS y consideran que `169.254.169.254` es lo mismo que [`168.63.129.16`](../../virtual-network/what-is-ip-address-168-63-129-16.md).

## <a name="security"></a>Seguridad

El punto de conexión del servicio de metadatos de instancia solo es accesible desde la instancia de máquina virtual en ejecución en una dirección IP no enrutable. Además, el servicio rechaza cualquier solicitud que tenga un encabezado `X-Forwarded-For`.
Las solicitudes tienen que incluir también un encabezado `Metadata: true` para garantizar que la solicitud actual esté dirigida directamente y no como parte de un redireccionamiento accidental.

> [!IMPORTANT]
> Instance Metadata Service no es un canal para datos confidenciales. El punto de conexión está abierto a todos los procesos de la máquina virtual. La información expuesta a través de este servicio se debe considerar como información compartida con todas las aplicaciones que se ejecutan dentro de la máquina virtual.

## <a name="usage"></a>Uso

### <a name="accessing-azure-instance-metadata-service"></a>Acceso a Instance Metadata Service de Azure

Para tener acceso a Instance Metadata Service, cree una máquina virtual desde [Azure Resource Manager](/rest/api/resources/) o [Azure Portal](https://portal.azure.com) y siga los ejemplos de abajo.
Puede encontrar más ejemplos de cómo realizar consultas en IMDS en la página de [ejemplos de Instance Metadata Service de Azure](https://github.com/microsoft/azureimds).

A continuación se muestra el código de ejemplo para recuperar todos los metadatos de una instancia. Para acceder a un origen de datos específico, consulte la sección de [API de metadatos](#metadata-apis). 

**Solicitud**

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance?api-version=2019-06-01"
```

**Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

```json
{
  "compute": {
    "azEnvironment": "AzurePublicCloud",
    "customData": "",
    "location": "centralus",
    "name": "negasonic",
    "offer": "lampstack",
    "osType": "Linux",
    "placementGroupId": "",
    "plan": {
        "name": "5-6",
        "product": "lampstack",
        "publisher": "bitnami"
    },
    "platformFaultDomain": "0",
    "platformUpdateDomain": "0",
    "provider": "Microsoft.Compute",
    "publicKeys": [],
    "publisher": "bitnami",
    "resourceGroupName": "myrg",
    "resourceId": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/myrg/providers/Microsoft.Compute/virtualMachines/negasonic",
    "sku": "5-6",
    "storageProfile": {
        "dataDisks": [
          {
            "caching": "None",
            "createOption": "Empty",
            "diskSizeGB": "1024",
            "image": {
              "uri": ""
            },
            "lun": "0",
            "managedDisk": {
              "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampledatadiskname",
              "storageAccountType": "Standard_LRS"
            },
            "name": "exampledatadiskname",
            "vhd": {
              "uri": ""
            },
            "writeAcceleratorEnabled": "false"
          }
        ],
        "imageReference": {
          "id": "",
          "offer": "UbuntuServer",
          "publisher": "Canonical",
          "sku": "16.04.0-LTS",
          "version": "latest"
        },
        "osDisk": {
          "caching": "ReadWrite",
          "createOption": "FromImage",
          "diskSizeGB": "30",
          "diffDiskSettings": {
            "option": "Local"
          },
          "encryptionSettings": {
            "enabled": "false"
          },
          "image": {
            "uri": ""
          },
          "managedDisk": {
            "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampleosdiskname",
            "storageAccountType": "Standard_LRS"
          },
          "name": "exampleosdiskname",
          "osType": "Linux",
          "vhd": {
            "uri": ""
          },
          "writeAcceleratorEnabled": "false"
        }
    },
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "Department:IT;Environment:Prod;Role:WorkerRole",
    "version": "7.1.1902271506",
    "vmId": "13f56399-bd52-4150-9748-7190aae1ff21",
    "vmScaleSetName": "",
    "vmSize": "Standard_A1_v2",
    "zone": "1"
  },
  "network": {
    "interface": [
      {
        "ipv4": {
          "ipAddress": [
            {
              "privateIpAddress": "10.1.2.5",
              "publicIpAddress": "X.X.X.X"
            }
          ],
          "subnet": [
            {
              "address": "10.1.2.0",
              "prefix": "24"
            }
          ]
        },
        "ipv6": {
          "ipAddress": []
        },
        "macAddress": "000D3A36DDED"
      }
    ]
  }
}
```

### <a name="data-output"></a>Salida de datos

De forma predeterminada, el servicio de metadatos de instancia devuelve datos en formato JSON (`Content-Type: application/json`). Sin embargo, algunas API pueden devolver datos en formatos distinto si se solicita.
En la tabla siguiente se muestra una referencia de otros formatos de datos que las API pueden admitir.

API | Formato predeterminado de datos | Otros formatos
--------|---------------------|--------------
/attested | json | None
/identity | json | None
/instance | json | text
/scheduledevents | json | None

Para acceder a un formato de respuesta que no sea el predeterminado, especifique el formato solicitado como un parámetro de cadena de consulta en la solicitud. Por ejemplo:

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance?api-version=2017-08-01&format=text"
```

> [!NOTE]
> En el caso de los nodos hoja de /metadata/instance, el elemento `format=json` no funciona. Para estas consultas, `format=text` debe especificarse explícitamente ya que el formato predeterminado es JSON.

### <a name="versioning"></a>Control de versiones

Instance Metadata Service tiene versiones y es obligatorio especificar la versión de la API en la solicitud HTTP.

A continuación se indican las versiones de servicio admitidas: 2017-04-02, 2017-08-01, 2017-12-01, 2018-02-01, 2018-04-02, 2018-10-01, 2019-02-01, 2019-03-11, 2019-04-30, 2019-06-01, 2019-06-04, 2019-08-01, 2019-08-15.

Tenga en cuenta que cuando se lanza una nueva versión, se tarda un tiempo en implementarse en todas las regiones. La versión 2019-11-01 se está implementando actualmente y puede que no esté disponible en todas las regiones.

A medida que se agreguen versiones más recientes, todavía se podrá acceder a las versiones anteriores por motivos de compatibilidad si los scripts tienen dependencias en formatos de datos específicos.

Cuando no se especifica ninguna versión, se devuelve un error con una lista de las versiones más recientes que son compatibles.

> [!NOTE]
> La respuesta es una cadena JSON. En el ejemplo siguiente se indica la condición de error cuando no se especifica la versión, la respuesta aparece con formato de sangría para mejorar la legibilidad.

**Solicitud**

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance"
```

**Respuesta**

```json
{
    "error": "Bad request. api-version was not specified in the request. For more information refer to aka.ms/azureimds",
    "newest-versions": [
        "2018-10-01",
        "2018-04-02",
        "2018-02-01"
    ]
}
```

## <a name="metadata-apis"></a>API de metadatos

Metadata Service contiene varias API que representan distintos orígenes de datos.

API | Descripción | Versión de introducción
----|-------------|-----------------------
/attested | Consulte [Datos atestiguados](#attested-data) | 2018-10-01
/identity | Consulte el artículo sobre cómo [obtener un token de acceso](../../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md) | 2018-02-01
/instance | Vea [API de instancia](#instance-api). | 2017-04-02
/scheduledevents | Consulte [Scheduled Events](scheduled-events.md). | 2017-08-01

## <a name="instance-api"></a>API de instancia

La API de instancia expone los metadatos importantes de las instancias de máquina virtual, como la máquina virtual, la red y el almacenamiento. Se puede acceder a las siguientes categorías a través de la instancia o el proceso:

data | Descripción | Versión de introducción
-----|-------------|-----------------------
azEnvironment | Entorno de Azure donde se está ejecutando la máquina virtual | 2018-10-01
customData | Esta funcionalidad actualmente está deshabilitada. Esta documentación se actualizará cuando esté disponible | 01-02-2019
ubicación | La región de Azure donde se ejecuta la máquina virtual | 2017-04-02
name | Nombre de la máquina virtual | 2017-04-02
offer | Ofrece información de la imagen de la máquina virtual y solo está presente para imágenes implementadas desde la galería de imágenes de Azure. | 2017-04-02
osType | Linux o Windows | 2017-04-02
placementGroupId | [Grupo de selección de ubicación](../../virtual-machine-scale-sets/virtual-machine-scale-sets-placement-groups.md) de su conjunto de escalado de máquina virtual | 2017-08-01
plan | [Plan](/rest/api/compute/virtualmachines/createorupdate#plan) que contiene el nombre, el producto y el editor de una máquina virtual si es una imagen de Azure Marketplace | 2018-04-02
platformUpdateDomain |  El [dominio de actualización](manage-availability.md) en que se ejecuta la máquina virtual. | 2017-04-02
platformFaultDomain | El [dominio de error](manage-availability.md) en que se ejecuta la máquina virtual. | 2017-04-02
provider | Proveedor de la máquina virtual | 2018-10-01
publicKeys | [Colección de claves públicas](/rest/api/compute/virtualmachines/createorupdate#sshpublickey) asignada a la máquina virtual y rutas de acceso | 2018-04-02
publisher | Publicador de la imagen de VM | 2017-04-02
resourceGroupName | [Grupo de recursos](../../azure-resource-manager/management/overview.md) para su máquina virtual | 2017-08-01
resourceId | El identificador [completo](/rest/api/resources/resources/getbyid) del recurso | 2019-03-11
sku | SKU específica de la imagen de VM | 2017-04-02
storageProfile | Consulte [Perfil de almacenamiento](#storage-metadata) | 2019-06-01
subscriptionId | Suscripción de Azure para la máquina virtual | 2017-08-01
etiquetas | [Etiquetas](../../azure-resource-manager/management/tag-resources.md) para su máquina virtual  | 2017-08-01
tagsList | Etiquetas con formato de matriz de JSON para facilitar el análisis mediante programación  | 2019-06-04
version | Versión de la imagen de máquina virtual | 2017-04-02
vmId | [Identificador único](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/) para la máquina virtual. | 2017-04-02
vmScaleSetName | [Nombre del conjunto de escalado de máquina virtual](../../virtual-machine-scale-sets/overview.md) del conjunto de escalado de la máquina virtual | 2017-12-01
vmSize | [Tamaño de VM](../sizes.md) | 2017-04-02
zona | [Zona de disponibilidad](../../availability-zones/az-overview.md) de la máquina virtual | 2017-12-01

### <a name="sample-1-tracking-vm-running-on-azure"></a>Muestra 1: Seguimiento de una máquina virtual que se ejecuta en Azure

Como proveedor de servicios, es posible que necesite hacer seguimiento de la cantidad de máquinas virtuales que ejecutan su software o que tenga agentes que deban hacer seguimiento de la unicidad de la máquina virtual. Para poder obtener un identificador único para una máquina virtual, use el campo `vmId` del servicio de metadatos de instancia.

**Solicitud**

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/vmId?api-version=2017-08-01&format=text"
```

**Respuesta**

```text
5c08b38e-4d57-4c23-ac45-aca61037f084
```

### <a name="sample-2-placement-of-containers-data-partitions-based-faultupdate-domain"></a>Ejemplo 2: Ubicación de los contenedores y el dominio de error/actualización basado en particiones de datos

Para ciertos escenarios, la ubicación de las distintas réplicas de datos es de máxima importancia. Por ejemplo, para [la ubicación de réplicas de HDFS](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Replica_Placement:_The_First_Baby_Steps) o la ubicación de contenedores a través de un [orquestador](https://kubernetes.io/docs/user-guide/node-selection/) se debe saber en qué `platformFaultDomain` y `platformUpdateDomain` se ejecuta la máquina virtual.
También puede usar las [zonas de disponibilidad](../../availability-zones/az-overview.md) para las instancias para tomar estas decisiones.
Puede consultar directamente estos datos a través del servicio de metadatos de instancia.

**Solicitud**

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/platformFaultDomain?api-version=2017-08-01&format=text"
```

**Respuesta**

```text
0
```

### <a name="sample-3-getting-more-information-about-the-vm-during-support-case"></a>Ejemplo 3: Obtención de más información sobre la VM durante el caso de soporte técnico

Como proveedor de servicios, es posible que reciba una llamada de soporte técnico en la que le gustaría tener más información sobre la máquina virtual. Pedirle al cliente que comparta los metadatos del equipo puede proporcionar información básica para que el profesional de soporte técnico conozca la variante de máquina virtual en Azure.

**Solicitud**

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute?api-version=2019-06-01"
```

**Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

```json
{
    "azEnvironment": "AzurePublicCloud",
    "customData": "",
    "location": "centralus",
    "name": "negasonic",
    "offer": "lampstack",
    "osType": "Linux",
    "placementGroupId": "",
    "plan": {
        "name": "5-6",
        "product": "lampstack",
        "publisher": "bitnami"
    },
    "platformFaultDomain": "0",
    "platformUpdateDomain": "0",
    "provider": "Microsoft.Compute",
    "publicKeys": [],
    "publisher": "bitnami",
    "resourceGroupName": "myrg",
    "resourceId": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/myrg/providers/Microsoft.Compute/virtualMachines/negasonic",
    "sku": "5-6",
    "storageProfile": {
        "dataDisks": [
          {
            "caching": "None",
            "createOption": "Empty",
            "diskSizeGB": "1024",
            "image": {
              "uri": ""
            },
            "lun": "0",
            "managedDisk": {
              "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampledatadiskname",
              "storageAccountType": "Standard_LRS"
            },
            "name": "exampledatadiskname",
            "vhd": {
              "uri": ""
            },
            "writeAcceleratorEnabled": "false"
          }
        ],
        "imageReference": {
          "id": "",
          "offer": "UbuntuServer",
          "publisher": "Canonical",
          "sku": "16.04.0-LTS",
          "version": "latest"
        },
        "osDisk": {
          "caching": "ReadWrite",
          "createOption": "FromImage",
          "diskSizeGB": "30",
          "diffDiskSettings": {
            "option": "Local"
          },
          "encryptionSettings": {
            "enabled": "false"
          },
          "image": {
            "uri": ""
          },
          "managedDisk": {
            "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampleosdiskname",
            "storageAccountType": "Standard_LRS"
          },
          "name": "exampleosdiskname",
          "osType": "Linux",
          "vhd": {
            "uri": ""
          },
          "writeAcceleratorEnabled": "false"
        }
    },
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "Department:IT;Environment:Test;Role:WebRole",
    "version": "7.1.1902271506",
    "vmId": "13f56399-bd52-4150-9748-7190aae1ff21",
    "vmScaleSetName": "",
    "vmSize": "Standard_A1_v2",
    "zone": "1"
}
```

### <a name="sample-4-getting-azure-environment-where-the-vm-is-running"></a>Ejemplo 4: Obtención del entorno de Azure donde se está ejecutando la máquina virtual

Azure tiene varias nubes soberanas, como [Azure Government](https://azure.microsoft.com/overview/clouds/government/). En ocasiones, necesitará el entorno de Azure para tomar algunas decisiones acerca del tiempo de ejecución. En el siguiente ejemplo se muestra cómo lograr este comportamiento.

**Solicitud**

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/azEnvironment?api-version=2018-10-01&format=text"
```

**Respuesta**

```text
AzurePublicCloud
```

Aquí se muestran las nubes y los valores del entorno de Azure.

 Nube   | Entorno de Azure
---------|-----------------
[Todas las regiones globales de Azure disponibles con carácter general](https://azure.microsoft.com/regions/)     | AzurePublicCloud
[Azure Government](https://azure.microsoft.com/overview/clouds/government/)              | AzureUSGovernmentCloud
[Azure China 21Vianet](https://azure.microsoft.com/global-infrastructure/china/)         | AzureChinaCloud
[Azure Alemania](https://azure.microsoft.com/overview/clouds/germany/)                    | AzureGermanCloud

## <a name="network-metadata"></a>Metadatos de red 

Los metadatos de red forman parte de la API de instancia. Las siguientes categorías de redes están disponibles a través del punto de conexión de red o de instancia.

data | Descripción | Versión de introducción
-----|-------------|-----------------------
ipv4/privateIpAddress | Dirección IPv4 local de la máquina virtual | 2017-04-02
ipv4/publicIpAddress | Dirección IPv4 pública de la máquina virtual | 2017-04-02
subnet/address | Dirección de subred de la máquina virtual | 2017-04-02
subnet/prefix | Prefijo de la subred, ejemplo, 24 | 2017-04-02
ipv6/ipAddress | Dirección IPv6 local de la máquina virtual | 2017-04-02
macAddress | Dirección de MAC de la VM | 2017-04-02

> [!NOTE]
> Todas las respuestas de las API son cadenas JSON. Todas las respuestas de ejemplo siguientes se han imprimido correctamente para mejorar la legibilidad.

#### <a name="sample-1-retrieving-network-information"></a>Muestra 1: Recuperación de información de red

**Solicitud**

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/network?api-version=2017-08-01"
```

**Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

```json
{
  "interface": [
    {
      "ipv4": {
        "ipAddress": [
          {
            "privateIpAddress": "10.1.0.4",
            "publicIpAddress": "X.X.X.X"
          }
        ],
        "subnet": [
          {
            "address": "10.1.0.0",
            "prefix": "24"
          }
        ]
      },
      "ipv6": {
        "ipAddress": []
      },
      "macAddress": "000D3AF806EC"
    }
  ]
}

```

#### <a name="sample-2-retrieving-public-ip-address"></a>Ejemplo 2: Recuperación de dirección IP pública

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-08-01&format=text"
```

## <a name="storage-metadata"></a>Metadatos de almacenamiento

Los metadatos de almacenamiento forman parte de la API de instancia en el punto de conexión instance/compute/storageProfile.
Proporciona detalles sobre los discos de almacenamiento asociados a la máquina virtual. 

El perfil de almacenamiento de una máquina virtual se divide en tres categorías: referencia de imagen, disco del sistema operativo y discos de datos.

El objeto de referencia de imagen contiene la información siguiente sobre la imagen del sistema operativo:

data    | Descripción
--------|-----------------
id      | Id. de recurso
offer   | Oferta de la plataforma o imagen de Marketplace
publisher | Editor de la imagen
sku     | SKU de la imagen
version | Versión de la plataforma o imagen de Marketplace

El objeto de disco del sistema operativo contiene la información siguiente sobre el disco del sistema operativo que usa la máquina virtual:

data    | Descripción
--------|-----------------
caching | Requisitos de almacenamiento en caché
createOption | Información sobre cómo se ha creado la máquina virtual
diffDiskSettings | Configuración de discos efímeros
diskSizeGB | Tamaño del disco en GB
imagen   | Disco duro virtual de la imagen de usuario de origen
lun     | Número de unidad lógica del disco
managedDisk | Parámetros de disco administrado
name    | Nombre del disco
vhd     | Disco duro virtual
writeAcceleratorEnabled | Si writeAccelerator está habilitado o no en el disco

La matriz de discos de datos contiene una lista de discos de datos conectados a la máquina virtual. Cada objeto de disco de datos contiene la información siguiente:

data    | Descripción
--------|-----------------
caching | Requisitos de almacenamiento en caché
createOption | Información sobre cómo se ha creado la máquina virtual
diffDiskSettings | Configuración de discos efímeros
diskSizeGB | Tamaño del disco en GB
encryptionSettings | Configuración de cifrado para el disco
imagen   | Disco duro virtual de la imagen de usuario de origen
managedDisk | Parámetros de disco administrado
name    | Nombre del disco
osType  | Tipo de sistema operativo incluido en el disco
vhd     | Disco duro virtual
writeAcceleratorEnabled | Si writeAccelerator está habilitado o no en el disco

El ejemplo siguiente muestra cómo consultar la información de almacenamiento de la máquina virtual.

**Solicitud**

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/storageProfile?api-version=2019-06-01"
```

**Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

```json
{
    "dataDisks": [
      {
        "caching": "None",
        "createOption": "Empty",
        "diskSizeGB": "1024",
        "image": {
          "uri": ""
        },
        "lun": "0",
        "managedDisk": {
          "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampledatadiskname",
          "storageAccountType": "Standard_LRS"
        },
        "name": "exampledatadiskname",
        "vhd": {
          "uri": ""
        },
        "writeAcceleratorEnabled": "false"
      }
    ],
    "imageReference": {
      "id": "",
      "offer": "UbuntuServer",
      "publisher": "Canonical",
      "sku": "16.04.0-LTS",
      "version": "latest"
    },
    "osDisk": {
      "caching": "ReadWrite",
      "createOption": "FromImage",
      "diskSizeGB": "30",
      "diffDiskSettings": {
        "option": "Local"
      },
      "encryptionSettings": {
        "enabled": "false"
      },
      "image": {
        "uri": ""
      },
      "managedDisk": {
        "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampleosdiskname",
        "storageAccountType": "Standard_LRS"
      },
      "name": "exampleosdiskname",
      "osType": "Linux",
      "vhd": {
        "uri": ""
      },
      "writeAcceleratorEnabled": "false"
    }
}
```

## <a name="vm-tags"></a>Etiquetas de máquina virtual

Las etiquetas de máquina virtual se incluyen en la API de instancia en el punto de conexión instance/compute/tags.
Es posible que se hayan aplicado etiquetas a las máquinas virtuales de Azure para organizarlas de forma lógica en una taxonomía. Las etiquetas asignadas a una máquina virtual se pueden recuperar mediante esta solicitud.

**Solicitud**

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/tags?api-version=2018-10-01&format=text"
```

**Respuesta**

```text
Department:IT;Environment:Test;Role:WebRole
```

El campo `tags` es una cadena con las etiquetas delimitadas por puntos y coma. Esta salida puede ser un problema si se usan puntos y coma en las propias etiquetas. Si se escribe un analizador para extraer mediante programación las etiquetas, debe basarse en el campo `tagsList`. El campo `tagsList` es una matriz JSON sin delimitadores y, por tanto, es más fácil de analizar.

**Solicitud**

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/tagsList?api-version=2019-06-04"
```

**Respuesta**

```json
[
  {
    "name": "Department",
    "value": "IT"
  },
  {
    "name": "Environment",
    "value": "Test"
  },
  {
    "name": "Role",
    "value": "WebRole"
  }
]
```

## <a name="attested-data"></a>Datos atestiguados

Parte del escenario que sirve Instance Metadata Service está destinado a garantizar que los datos proporcionados provienen de Azure. Firmamos una parte de esta información para que las imágenes de Marketplace puedan tener la seguridad de que su imagen se ejecuta en Azure.

### <a name="sample-1-getting-attested-data"></a>Muestra 1: Obtención de datos de atestación

> [!NOTE]
> Todas las respuestas de las API son cadenas JSON. Las siguientes respuestas de ejemplo se han impreso correctamente para facilitar su lectura.

**Solicitud**

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/attested/document?api-version=2018-10-01&nonce=1234567890"
```

Api-version es un campo obligatorio. Consulte la [sección de uso](#usage) para ver las versiones de API compatibles.
Nonce es una cadena opcional de 10 dígitos. Si no se proporciona, IMDS devuelve en su lugar la marca de tiempo UTC actual.

> [!NOTE]
> Debido al mecanismo de almacenamiento en caché de IMDS, es posible que se devuelva un valor de nonce previamente almacenado en caché.

**Respuesta**

> [!NOTE]
> La respuesta es una cadena JSON. La respuesta de ejemplo siguiente se ha impreso correctamente para mejorar la legibilidad.

```json
{
 "encoding":"pkcs7","signature":"MIIEEgYJKoZIhvcNAQcCoIIEAzCCA/8CAQExDzANBgkqhkiG9w0BAQsFADCBugYJKoZIhvcNAQcBoIGsBIGpeyJub25jZSI6IjEyMzQ1NjY3NjYiLCJwbGFuIjp7Im5hbWUiOiIiLCJwcm9kdWN0IjoiIiwicHVibGlzaGVyIjoiIn0sInRpbWVTdGFtcCI6eyJjcmVhdGVkT24iOiIxMS8yMC8xOCAyMjowNzozOSAtMDAwMCIsImV4cGlyZXNPbiI6IjExLzIwLzE4IDIyOjA4OjI0IC0wMDAwIn0sInZtSWQiOiIifaCCAj8wggI7MIIBpKADAgECAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBBAUAMCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tMB4XDTE4MTEyMDIxNTc1N1oXDTE4MTIyMDIxNTc1NlowKzEpMCcGA1UEAxMgdGVzdHN1YmRvbWFpbi5tZXRhZGF0YS5henVyZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAML/tBo86ENWPzmXZ0kPkX5dY5QZ150mA8lommszE71x2sCLonzv4/UWk4H+jMMWRRwIea2CuQ5RhdWAHvKq6if4okKNt66fxm+YTVz9z0CTfCLmLT+nsdfOAsG1xZppEapC0Cd9vD6NCKyE8aYI1pliaeOnFjG0WvMY04uWz2MdAgMBAAGjYDBeMFwGA1UdAQRVMFOAENnYkHLa04Ut4Mpt7TkJFfyhLTArMSkwJwYDVQQDEyB0ZXN0c3ViZG9tYWluLm1ldGFkYXRhLmF6dXJlLmNvbYIQZ8VuSofHbJRAQNBNpiASdDANBgkqhkiG9w0BAQQFAAOBgQCLSM6aX5Bs1KHCJp4VQtxZPzXF71rVKCocHy3N9PTJQ9Fpnd+bYw2vSpQHg/AiG82WuDFpPReJvr7Pa938mZqW9HUOGjQKK2FYDTg6fXD8pkPdyghlX5boGWAMMrf7bFkup+lsT+n2tRw2wbNknO1tQ0wICtqy2VqzWwLi45RBwTGB6DCB5QIBATA/MCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBCwUAMA0GCSqGSIb3DQEBAQUABIGAld1BM/yYIqqv8SDE4kjQo3Ul/IKAVR8ETKcve5BAdGSNkTUooUGVniTXeuvDj5NkmazOaKZp9fEtByqqPOyw/nlXaZgOO44HDGiPUJ90xVYmfeK6p9RpJBu6kiKhnnYTelUk5u75phe5ZbMZfBhuPhXmYAdjc7Nmw97nx8NnprQ="
}
```

El blob de firma es una versión con la firma [pkcs7](https://aka.ms/pkcs7) del documento. Contiene el certificado que se usa para la firma junto con detalles de la máquina virtual como vmld, sku, nonce, subscriptionId y timeStamp para la creación y expiración del documento, y la información del plan sobre la imagen. La información del plan solo se rellena para las imágenes de Azure Marketplace. El certificado se puede extraer de la respuesta y usarse para validar que la respuesta es válida y viene de Azure.
El documento contiene estos campos:

data | Descripción
-----|------------
valor de seguridad | Una cadena que se puede proporcionar de manera opcional con la solicitud. Si no se suministró ninguna clave nonce, se usa la marca de tiempo UTC actual
plan | El [plan de imagen de Azure Marketplace](/rest/api/compute/virtualmachines/createorupdate#plan). Contiene el identificador de plan (nombre), la oferta o imagen de producto (producto) y el identificador de publicador (publicador).
timestamp/createdOn | La marca de tiempo UTC para el momento en que se creó el documento firmado
timestamp/expiresOn | La marca de tiempo UTC para el momento en que expira el documento firmado
vmId |  [Identificador único](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/) para la máquina virtual.
subscriptionId | Suscripción de Azure para la máquina virtual publicada con fecha `2019-04-30`
sku | SKU específica de la imagen de VM, presentada en `2019-11-01`

### <a name="sample-2-validating-that-the-vm-is-running-in-azure"></a>Ejemplo 2: Validación de que la máquina virtual se ejecuta en Azure

Los proveedores de Marketplace desean asegurarse de que su software tiene licencia para ejecutarse solo en Azure. Si alguien copia el disco duro virtual en un entorno local, debería disponer de un capacidad para detectarlo. Mediante una llamada al servicio de metadatos de instancia, los proveedores pueden obtener datos firmados que garantizan la respuesta únicamente de Azure.

> [!NOTE]
> Requiere jq para instalarse.

**Solicitud**

```bash
# Get the signature
curl --silent -H Metadata:True --noproxy "*" "http://169.254.169.254/metadata/attested/document?api-version=2019-04-30" | jq -r '.["signature"]' > signature
# Decode the signature
base64 -d signature > decodedsignature
# Get PKCS7 format
openssl pkcs7 -in decodedsignature -inform DER -out sign.pk7
# Get Public key out of pkc7
openssl pkcs7 -in decodedsignature -inform DER  -print_certs -out signer.pem
# Get the intermediate certificate
curl -s -o intermediate.cer "$(openssl x509 -in signer.pem -text -noout | grep " CA Issuers -" | awk -FURI: '{print $2}')"
openssl x509 -inform der -in intermediate.cer -out intermediate.pem
# Verify the contents
openssl smime -verify -in sign.pk7 -inform pem -noverify
```

**Respuesta**

```json
Verification successful
{
  "nonce": "20181128-001617",
  "plan":
    {
      "name": "",
      "product": "",
      "publisher": ""
    },
  "timeStamp":
    {
      "createdOn": "11/28/18 00:16:17 -0000",
      "expiresOn": "11/28/18 06:16:17 -0000"
    },
  "vmId": "d3e0e374-fda6-4649-bbc9-7f20dc379f34",
  "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
  "sku": "RS3-Pro"
}
```

Compruebe que la firma proviene de Microsoft Azure y si hay errores en la cadena de certificados.

```bash
# Verify the subject name for the main certificate
openssl x509 -noout -subject -in signer.pem
# Verify the issuer for the main certificate
openssl x509 -noout -issuer -in signer.pem
#Validate the subject name for intermediate certificate
openssl x509 -noout -subject -in intermediate.pem
# Verify the issuer for the intermediate certificate
openssl x509 -noout -issuer -in intermediate.pem
# Verify the certificate chain, for Azure China 21Vianet the intermediate certificate will be from DigiCert Global Root CA
openssl verify -verbose -CAfile /etc/ssl/certs/Baltimore_CyberTrust_Root.pem -untrusted intermediate.pem signer.pem
```

> [!NOTE]
> Debido al mecanismo de almacenamiento en caché de IMDS, es posible que se devuelva un valor de nonce previamente almacenado en caché.

Se puede comparar la clave nonce del documento firmado si proporcionó un parámetro nonce en la solicitud inicial.

> [!NOTE]
> El certificado para la nube pública y la nube soberana serán diferentes.

Nube | Certificado
------|------------
[Todas las regiones globales de Azure disponibles con carácter general](https://azure.microsoft.com/regions/) | *.metadata.azure.com
[Azure Government](https://azure.microsoft.com/overview/clouds/government/)          | *.metadata.azure.us
[Azure China 21Vianet](https://azure.microsoft.com/global-infrastructure/china/)     | *.metadata.azure.cn
[Azure Alemania](https://azure.microsoft.com/overview/clouds/germany/)                | *.metadata.microsoftazure.de

> [!NOTE]
> Existe un problema conocido con respecto al certificado usado para firmar. Es posible que los certificados no tengan una coincidencia exacta de `metadata.azure.com` para la nube pública. Por lo tanto, la validación de la certificación debe permitir un nombre común de cualquier subdominio `.metadata.azure.com`.

En aquellos casos en los que el certificado intermedio no se puede descargar debido a restricciones de red durante la validación, es posible anclarlo. No obstante, Azure sustituirá los certificados según el procedimiento PKI estándar. Los certificados anclados deberán actualizarse cuando se produzca la sustitución. Cada vez que se planee un cambio para actualizar el certificado intermedio, se actualizará el blog de Azure y se notificará a los clientes de Azure. Encontrará los certificados intermedios [aquí](https://www.microsoft.com/pki/mscorp/cps/default.htm). Los certificados intermedios para cada una de las regiones pueden ser diferentes.

> [!NOTE]
> El certificado intermedio para Azure China 21Vianet será de la entidad de certificación raíz global de DigiCert en lugar de Baltimore.
Además, si había anclado los certificados intermedios para Azure China como parte del cambio de la entidad de la cadena raíz, los certificados intermedios tendrán que actualizarse.

## <a name="managed-identity-via-metadata-service"></a>Identidad administrada a través de Metadata Service

Una identidad administrada asignada por el sistema puede estar habilitada en la máquina virtual o una o varias identidades administradas asignadas por el usuario se pueden asignar a la máquina virtual.
A continuación, se pueden solicitar tokens para identidades administradas desde Instance Metadata Service. Estos tokens se pueden usar para autenticarse con otros servicios de Azure, como Azure Key Vault.

Para ver pasos detallados sobre cómo habilitar esta característica, consulte cómo [adquirir un token de acceso](../../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md).

## <a name="scheduled-events-via-metadata-service"></a>Scheduled Events a través de Metadata Service
Puede obtener el estado de los eventos programados mediante el servicio de metadatos y, a continuación, el usuario puede especificar un conjunto de acciones para ejecutarse en estos eventos.  Consulte la página de [eventos programados](scheduled-events.md) para obtener más información. 

## <a name="regional-availability"></a>Disponibilidad regional

El servicio está **disponible con carácter general** en todas las nubes de Azure.

## <a name="sample-code-in-different-languages"></a>Código de ejemplo en diferentes lenguajes

Ejemplos de llamadas al servicio de metadatos mediante lenguajes diferentes dentro de la máquina virtual:

Idioma      | Ejemplo
--------------|----------------
Bash          | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.sh
C#            | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.cs
Go            | https://github.com/Microsoft/azureimds/blob/master/imdssample.go
Java          | https://github.com/Microsoft/azureimds/blob/master/imdssample.java
NodeJS        | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.js
Perl          | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.pl
PowerShell    | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.ps1
Puppet        | https://github.com/keirans/azuremetadata
Python        | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.py
Ruby          | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.rb

## <a name="error-and-debugging"></a>Error y depuración

Si no se encuentra un elemento de datos o hay una solicitud con formato incorrecto, el servicio de metadatos de instancia devuelve errores HTTP estándar. Por ejemplo:

Código de estado HTTP | Motivo
----------------|-------
200 OK |
400 - Solicitud incorrecta | Falta el encabezado `Metadata: true` o el parámetro `format=json` al consultar un nodo hoja
404 No encontrado | El elemento solicitado no existe
405 Método no permitido | Solo se admiten solicitudes `GET`
410 Ya no existe | Reintente después de un tiempo durante un máximo de 70 segundos
429 Demasiadas solicitudes | La API es compatible actualmente con un máximo de cinco consultas por segundo
500 Error de servicio     | Vuelva a intentarlo más tarde

### <a name="known-issues-and-faq"></a>Problemas conocidos y preguntas más frecuentes

1. Obtengo el error `400 Bad Request, Required metadata header not specified`. ¿Qué significa?
   * El servicio de metadatos de instancia requiere que el encabezado `Metadata: true` se transmita en la solicitud. Transmitir este encabezado en la llamada de REST permite tener acceso al servicio de metadatos de instancia.
1. ¿Por qué no recibo información de proceso de mi máquina virtual?
   * Actualmente el servicio de metadatos de instancia solo admite instancias creadas con Azure Resource Manager. En el futuro, es posible que se agregue compatibilidad para máquinas virtuales de servicio en la nube.
1. Creé mi máquina virtual mediante Azure Resource Manager hace un tiempo. ¿Por qué no veo la información de metadatos de proceso?
   * En el caso de las máquinas virtuales que se crearon después de septiembre de 2016, agregue una [etiqueta](../../azure-resource-manager/management/tag-resources.md) para empezar a ver los metadatos de proceso. En el caso de máquinas virtuales anteriores (creadas antes de septiembre de 2016), agregue o quite extensiones o discos de datos a las instancias de la máquina virtual para actualizar los metadatos.
1. No veo todos los datos rellenados para la nueva versión
   * En el caso de las máquinas virtuales que se crearon después de septiembre de 2016, agregue una [etiqueta](../../azure-resource-manager/management/tag-resources.md) para empezar a ver los metadatos de proceso. En el caso de máquinas virtuales anteriores (creadas antes de septiembre de 2016), agregue o quite extensiones o discos de datos a las instancias de la máquina virtual para actualizar los metadatos.
1. ¿Por qué recibo el error `500 Internal Server Error` o `410 Resource Gone`?
   * Vuelva a intentar la solicitud en función del sistema de interrupción exponencial u otros métodos que se describen en [Control de errores transitorios](/azure/architecture/best-practices/transient-faults). Si el problema persiste, cree un problema de soporte técnico en Azure Portal para la máquina virtual.
1. ¿Esto funciona para las instancias de conjunto de escalado de máquinas virtuales?
   * Sí, el servicio de metadatos está disponible para las instancias del conjunto de escalado.
1. Actualicé mis etiquetas en Virtual Machine Scale Sets, pero no aparecen en las instancias, a diferencia de las máquinas virtuales de instancia única.
   * Actualmente, las etiquetas para Scale Sets solo se muestran a la máquina virtual durante un reinicio, restablecimiento de imagen o cambio de disco en la instancia.
1. ¿Por qué se agotó el tiempo de espera de solicitud para mi llamada al servicio?
   * Las llamadas de metadatos se deben hacer desde la dirección IP principal asignada a la tarjeta de red principal de la máquina virtual. Además, en caso de que haya cambiado las rutas, debe haber una ruta para la dirección 169.254.169.254/32 en la tabla de rutas local de la máquina virtual.
   * <details>
        <summary>Comprobación de la tabla de rutas</summary>

        1. Vuelque la tabla de rutas local con un comando como `netstat -r` y busque la entrada IMDS (por ejemplo):
            ```console
            ~$ netstat -r
            Kernel IP routing table
            Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
            default         _gateway        0.0.0.0         UG        0 0          0 eth0
            168.63.129.16   _gateway        255.255.255.255 UGH       0 0          0 eth0
            169.254.169.254 _gateway        255.255.255.255 UGH       0 0          0 eth0
            172.16.69.0     0.0.0.0         255.255.255.0   U         0 0          0 eth0
            ```
        1. Compruebe que existe una ruta para `169.254.169.254` y anote la interfaz de red correspondiente (por ejemplo, `eth0`).
        1. Vuelque la configuración de la interfaz para la interfaz correspondiente en la tabla de rutas (tenga en cuenta que el nombre exacto del archivo de configuración puede variar).
            ```console
            ~$ cat /etc/netplan/50-cloud-init.yaml
            network:
            ethernets:
                eth0:
                    dhcp4: true
                    dhcp4-overrides:
                        route-metric: 100
                    dhcp6: false
                    match:
                        macaddress: 00:0d:3a:e4:c7:2e
                    set-name: eth0
            version: 2
            ```
        1. Si usa una dirección IP dinámica, anote la dirección MAC. Si usa una dirección IP estática, puede anotar las direcciones IP o la dirección MAC que aparecen.
        1. Confirme que la interfaz corresponde a la NIC principal y la dirección IP principal de la máquina virtual. Para encontrar la NIC/dirección IP principal, consulte la configuración de red en Azure Portal o búsquela [con la CLI de Azure](/cli/azure/vm/nic?view=azure-cli-latest#az-vm-nic-show). Anote las direcciones IP públicas y privadas (y la dirección MAC, si usa la CLI). Ejemplo de la CLI de PowerShell:
            ```powershell
            $ResourceGroup = '<Resource_Group>'
            $VmName = '<VM_Name>'
            $NicNames = az vm nic list --resource-group $ResourceGroup --vm-name $VmName | ConvertFrom-Json | Foreach-Object { $_.id.Split('/')[-1] }
            foreach($NicName in $NicNames)
            {
                $Nic = az vm nic show --resource-group $ResourceGroup --vm-name $VmName --nic $NicName | ConvertFrom-Json
                Write-Host $NicName, $Nic.primary, $Nic.macAddress
            }
            # Output: ipexample606 True 00-0D-3A-E4-C7-2E
            ```
        1. Si no coinciden, actualice la tabla de rutas para que la NIC/dirección IP principal estén dirigidas.
    </details>

## <a name="support-and-feedback"></a>Asistencia y comentarios

Envíe sus comentarios sobre https://feedback.azure.com.

Para obtener soporte técnico para el servicio, cree un problema de compatibilidad en Azure Portal en la máquina virtual en la que no obtiene respuesta de metadatos después de reintentos prolongados.
Use el tipo de problema de `Management` y seleccione `Instance Metadata Service` como categoría.

![Soporte técnico de Instance Metadata](./media/instance-metadata-service/InstanceMetadata-support.png "Captura de pantalla: Apertura de un caso de soporte técnico al tener problemas con Instance Metadata Service")

## <a name="next-steps"></a>Pasos siguientes

Más información sobre:
1. [Obtener un token de acceso para la máquina virtual](../../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md)
1. [Eventos programados](scheduled-events.md)