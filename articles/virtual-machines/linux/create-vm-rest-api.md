---
title: Creación de una máquina virtual Linux mediante la API REST
description: Aprenda a crear una máquina virtual Linux e Azure que use Managed Disks y autenticación de SSH con la API REST de Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 06/05/2018
ms.author: cynthn
ms.openlocfilehash: d6e5195f43991f4d40af57c1ab4b87aaca475b64
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373409"
---
# <a name="create-a-linux-virtual-machine-that-uses-ssh-authentication-with-the-rest-api"></a>Creación de una máquina virtual Linux que usa la autenticación de SSH con la API REST

Una máquina virtual Linux en Azure consta de varios recursos como discos e interfaces de red y define parámetros como la ubicación, el tamaño y la imagen del sistema operativo, así como la configuración de autenticación.

Puede crear una máquina virtual Linux mediante Azure Portal, la CLI de Azure 2.0, muchos SDK de Azure, plantillas de Azure Resource Manager y muchas herramientas de terceros como Ansible o Terraform. Todas estas herramientas utilizan en última instancia la API REST para crear la máquina virtual Linux.

En este artículo se muestra cómo usar la API REST para crear una máquina virtual Linux que ejecuta Ubuntu 18.04-LTS con discos administrados y autenticación de SSH.

## <a name="before-you-start"></a>Antes de comenzar

Antes de crear y enviar la solicitud, necesitará:

* El parámetro `{subscription-id}` para su suscripción
  * Si tiene varias suscripciones, consulte [Trabajo con varias suscripciones](/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest).
* Un parámetro `{resourceGroupName}` que ha creado de antemano
* Una [interfaz de red virtual](../../virtual-network/virtual-network-network-interface.md) en el mismo grupo de recursos
* Un par de claves SSH (puede [generar una nueva](mac-create-ssh-keys.md) si no tiene ninguna)

## <a name="request-basics"></a>Conceptos básicos de la solicitud

Para crear o actualizar una máquina virtual, use la siguiente operación *PUT*:

``` http
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2017-12-01
```

Además de los parámetros `{subscription-id}` y `{resourceGroupName}`, tendrá que especificar el parámetro `{vmName}` (`api-version` es opcional, pero este artículo se ha probado con `api-version=2017-12-01`).

Los siguientes encabezados son obligatorios:

| Encabezado de solicitud   | Descripción |
|------------------|-----------------|
| *Content-Type:*  | Necesario. Establézcalo en `application/json`. |
| *Authorization:* | Necesario. Establézcalo en un [token de acceso](/rest/api/azure/#authorization-code-grant-interactive-clients) `Bearer` válido. |

Para obtener información general sobre cómo trabajar con solicitudes de la API REST, consulte [Components of a REST API request/response](/rest/api/azure/#components-of-a-rest-api-requestresponse) (Componentes de una solicitud/respuesta de la API REST).

## <a name="create-the-request-body"></a>Creación del cuerpo de la solicitud

Para crear el cuerpo de la solicitud, se usan las siguientes definiciones comunes:

| Nombre                       | Obligatorio | Tipo                                                                                | Descripción  |
|----------------------------|----------|-------------------------------------------------------------------------------------|--------------|
| ubicación                   | True     | string                                                                              | Ubicación del recurso |
| name                       |          | string                                                                              | Escriba un nombre para la máquina virtual. |
| properties.hardwareProfile |          | [HardwareProfile](/rest/api/compute/virtualmachines/createorupdate#hardwareprofile) | Especifica la configuración de hardware de la máquina virtual. |
| properties.storageProfile  |          | [StorageProfile](/rest/api/compute/virtualmachines/createorupdate#storageprofile)   | Especifica la configuración de almacenamiento de los discos de máquina virtual. |
| properties.osProfile       |          | [OSProfile](/rest/api/compute/virtualmachines/createorupdate#osprofile)             | Especifica la configuración del sistema operativo de la máquina virtual. |
| properties.networkProfile  |          | [NetworkProfile](/rest/api/compute/virtualmachines/createorupdate#networkprofile)   | Especifica las interfaces de red de la máquina virtual. |

A continuación se muestra un ejemplo de cuerpo de la solicitud. Asegúrese de especificar el nombre de la máquina virtual en los parámetros `{computerName}` y `{name}`, el nombre de la interfaz de red que ha creado en `networkInterfaces`, su nombre de usuario en `adminUsername` y `path`, y la parte *pública* de su par de claves SSH (ubicada, por ejemplo, en `~/.ssh/id_rsa.pub`) en `keyData`. Otros parámetros que puede querer modificar son `location` y `vmSize`.  

```json
{
  "location": "eastus",
  "name": "{vmName}",
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_DS1_v2"
    },
    "storageProfile": {
      "imageReference": {
        "sku": "18.04-LTS",
        "publisher": "Canonical",
        "version": "latest",
        "offer": "UbuntuServer"
      },
      "osDisk": {
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Premium_LRS"
        },
        "name": "myVMosdisk",
        "createOption": "FromImage"
      }
    },
    "osProfile": {
      "adminUsername": "{your-username}",
      "computerName": "{vmName}",
      "linuxConfiguration": {
        "ssh": {
          "publicKeys": [
            {
              "path": "/home/{your-username}/.ssh/authorized_keys",
              "keyData": "ssh-rsa AAAAB3NzaC1{snip}mf69/J1"
            }
          ]
        },
        "disablePasswordAuthentication": true
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/{subscription-id}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkInterfaces/{existing-nic-name}",
          "properties": {
            "primary": true
          }
        }
      ]
    }
  }
}
```

Para obtener una lista completa de las definiciones disponibles en el cuerpo de la solicitud, consulte las [definiciones del cuerpo de la solicitud de creación o actualización de máquinas virtuales](/rest/api/compute/virtualmachines/createorupdate#definitions).

## <a name="sending-the-request"></a>Envío de la solicitud

Puede usar al cliente de su preferencia para enviar esta solicitud HTTP. También puede usar una [herramienta en el explorador](/rest/api/compute/virtualmachines/createorupdate) al hacer clic en el botón **Pruébelo**.

### <a name="responses"></a>Respuestas

Hay dos respuestas correctas a la operación para crear o actualizar una máquina virtual:

| Nombre        | Tipo                                                                              | Descripción |
|-------------|-----------------------------------------------------------------------------------|-------------|
| 200 OK      | [VirtualMachine](/rest/api/compute/virtualmachines/createorupdate#virtualmachine) | Aceptar          |
| 201 Creado | [VirtualMachine](/rest/api/compute/virtualmachines/createorupdate#virtualmachine) | Creado     |

Una respuesta *201 Creado* condensada del cuerpo de solicitud de ejemplo anterior que crea una máquina virtual muestra que se ha asignado *vmId* y que *provisioningState* es *Creando*:

```json
{
    "vmId": "e0de9b84-a506-4b95-9623-00a425d05c90",
    "provisioningState": "Creating"
}
```

Para más información sobre las respuestas de API REST, consulte [Process the response message](/rest/api/azure/#process-the-response-message) (Procesamiento del mensaje de respuesta).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las API REST de Azure u otras herramientas de administración como la CLI de Azure o Azure PowerShell, consulte lo siguiente:

- [API REST del proveedor de Azure Compute](/rest/api/compute/)
- [Get started with Azure REST API](/rest/api/azure/) (Introducción a la API REST de Azure)
- [CLI de Azure](/cli/azure/)
- [Módulo de Azure PowerShell](/powershell/azure/)
