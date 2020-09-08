---
title: Opciones de red del servicio Azure Image Builder
description: Descripción de las opciones de red al implementar el servicio Azure VM Image Builder
author: danielsollondon
ms.author: danis
ms.date: 08/10/2020
ms.topic: article
ms.service: virtual-machines
ms.subservice: imaging
ms.openlocfilehash: dfd0929ea03cd99033482f71579e91aaf6fc131c
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88068035"
---
# <a name="azure-image-builder-service-networking-options"></a>Opciones de red del servicio Azure Image Builder

Debe elegir implementar Azure Image Builder con o sin una red virtual existente.

## <a name="deploy-without-specifying-an-existing-vnet"></a>Implementación sin especificar una red virtual existente

Si no especifica una red virtual existente, Azure Image Builder crea una red virtual y una subred en el grupo de recursos de almacenamiento provisional. Un recurso de dirección IP pública se usa con un grupo de seguridad de red para restringir el tráfico entrante al servicio Azure Image Builder. La dirección IP pública facilita el canal para los comandos de Azure Image Builder durante la creación de la imagen. Una vez completada la creación, se eliminan la máquina virtual, la dirección IP pública, los discos y la red virtual. Para usar esta opción, no especifique ninguna propiedad de la red virtual.

## <a name="deploy-using-an-existing-vnet"></a>Implementación mediante una red virtual existente

Si especifica una red virtual y una subred, Azure Image Builder implementa la máquina virtual de compilación en la red virtual elegida. Puede acceder a los recursos que son accesibles en la red virtual. También puede crear una red virtual en silos que no esté conectada a ninguna otra red virtual. Si especifica una red virtual, Azure Image Builder no usa una dirección IP pública. La comunicación desde el servicio Azure Image Builder a la máquina virtual de creación usa la tecnología Azure Private Link.

Para más información, consulte uno de estos ejemplos:

* [Uso de Azure Image Builder para máquinas virtuales Linux que permiten acceder a una red virtual de Azure existente](../windows/image-builder-vnet.md)
* [Uso de Azure Image Builder para máquinas virtuales Linux que permiten acceder a una red virtual de Azure existente](image-builder-vnet.md)

### <a name="what-is-azure-private-link"></a>¿Qué es Azure Private Link?

Azure Private Link proporciona conectividad privada desde una red virtual a la plataforma como servicio (PaaS) de Azure, propiedad del cliente o servicios de asociados de Microsoft. Simplifica la arquitectura de red y protege la conexión entre los puntos de conexión de Azure mediante la eliminación de la exposición de los datos a la red pública de Internet. Para obtener más información, consulte la [documentación de Private Link](https://docs.microsoft.com/azure/private-link).

### <a name="required-permissions-for-an-existing-vnet"></a>Permisos necesarios para una red virtual existente

Azure Image Builder requiere permisos específicos para usar una red virtual existente. Para más información, consulte [Configuración de los permisos del Azure Image Builder la mediante CLI de Azure](image-builder-permissions-cli.md) o [Configuración de los permisos del servicio Azure Image Builder mediante PowerShell](image-builder-permissions-powershell.md).

### <a name="what-is-deployed-during-an-image-build"></a>¿Qué se implementa durante una creación de imagen?

El uso de una red virtual existente significa que Azure Image Builder implementa una máquina virtual (máquina virtual de proxy) adicional y una instancia de Azure Load Balancer (ALB) que está conectada a Private Link. El tráfico del servicio AIB pasa por el vínculo privado a ALB. ALB se comunica con la máquina virtual de proxy mediante el puerto 60001 para el sistema operativo Linux o 60000 para el sistema operativo Windows. El proxy reenvía comandos a la máquina virtual de creación mediante el puerto 22 para el sistema operativo Linux o 5986 para el sistema operativo Windows.

> [!NOTE]
> La red virtual debe estar en la misma región que la región de servicio Azure Image Builder.
> 

### <a name="why-deploy-a-proxy-vm"></a>¿Por qué implementar una máquina virtual de proxy?

Cuando una máquina virtual sin una dirección IP pública está detrás de una instancia de Load Balancer interna, no tiene acceso a Internet. La instancia de Azure Load Balancer que se usa para la red virtual es interna. La máquina virtual de proxy permite el acceso a Internet para la máquina virtual de creación durante las creaciones. Los grupos de seguridad de red asociados se pueden usar para restringir el acceso a la máquina virtual de creación.

El tamaño de la máquina virtual de proxy implementado es A1_v2 estándar además de la máquina virtual de creación. La máquina virtual de proxy se usa para enviar comandos entre el servicio de Azure Image Builder y la máquina virtual de creación. No se pueden cambiar las propiedades de la máquina virtual del proxy, incluido el tamaño o el sistema operativo. No se pueden cambiar las propiedades de la máquina virtual de proxy para creaciones de imágenes de Windows y Linux.

### <a name="image-template-parameters-to-support-vnet"></a>Parámetros de plantilla de imagen para admitir la red virtual
```json
"VirtualNetworkConfig": {
        "name": "",
        "subnetName": "",
        "resourceGroupName": ""
        },
```

| Configuración | Descripción |
|---------|---------|
| name | (Opcional) Nombre de una red virtual preexistente. |
| subnetName | Nombre de la subred dentro de la red virtual especificada. Debe especificarse si y solo si se especifica *name*. |
| resourceGroupName | Nombre del grupo de recursos que contiene la red virtual especificada. Debe especificarse si y solo si se especifica *name*. |

El servicio Private Link requiere una dirección IP de la red virtual y la subred dadas. Actualmente, Azure no admite directivas de red en estas direcciones IP. Por lo tanto, es necesario deshabilitar las directivas de red en la subred. Para obtener más información, consulte la [documentación de Private Link](https://docs.microsoft.com/azure/private-link).

### <a name="checklist-for-using-your-vnet"></a>Lista de comprobación para usar la red virtual

1. Permitir a Azure Load Balancer (ALB) para comunicarse con la máquina virtual de proxy en un grupo de seguridad de red
    * [Ejemplo de la CLI de Azure](image-builder-vnet.md#add-network-security-group-rule):
    * [Ejemplo de PowerShell](../windows/image-builder-vnet.md#add-network-security-group-rule)
2. Deshabilitar la directiva de servicio privada en la subred
    * [Ejemplo de la CLI de Azure](image-builder-vnet.md#disable-private-service-policy-on-subnet):
    * [Ejemplo de PowerShell](../windows/image-builder-vnet.md#disable-private-service-policy-on-subnet)
3. Permitir que Azure Image Builder cree un ALB y agregue máquinas virtuales a la red virtual
    * [Ejemplo de la CLI de Azure](image-builder-permissions-cli.md#existing-vnet-azure-role-example):
    * [Ejemplo de PowerShell](image-builder-permissions-powershell.md#permission-to-customize-images-on-your-vnets)
4. Permitir que Azure Image Builder lea y escriba imágenes de origen y cree imágenes
    * [Ejemplo de la CLI de Azure](image-builder-permissions-cli.md#custom-image-azure-role-example):
    * [Ejemplo de PowerShell](image-builder-permissions-powershell.md#custom-image-azure-role-example)
5. Asegúrese de estar usando la red virtual en la misma región que la región de servicio de Azure Image Builder.


## <a name="next-steps"></a>Pasos siguientes

Para más información, vea [Introducción a Azure Image Builder](image-builder-overview.md).