---
title: Cloud Shell en una instancia de Azure Virtual Network
description: Implementación de Cloud Shell en una instancia de Azure Virtual Network
services: Azure
documentationcenter: ''
author: maertendMSFT
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/15/2020
ms.author: damaerte
ms.openlocfilehash: 722d935c242a51ddfc01377676f026b71a8951b8
ms.sourcegitcommit: 4feb198becb7a6ff9e6b42be9185e07539022f17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89468545"
---
# <a name="deploy-cloud-shell-into-an-azure-virtual-network"></a>Implementación de Cloud Shell en una instancia de Azure Virtual Network
> [!NOTE]
> Esta funcionalidad está en versión preliminar pública.

Una sesión de Cloud Shell normal se ejecuta en un contenedor de una red de Microsoft independiente de los recursos. Es decir, que los comandos que se ejecutan dentro del contenedor no pueden acceder a los recursos a los que solo se puede acceder desde una red virtual específica. Por ejemplo, no puede usar SSH para conectarse desde Cloud Shell a una máquina virtual que solo tiene una dirección IP privada, ni usar kubectl para conectarse a un clúster de Kubernetes que ha bloqueado el acceso. 

Esta característica opcional se encarga de estas limitaciones y permite implementar Cloud Shell en una red virtual de Azure bajo su control. Desde allí, el contenedor puede interactuar con los recursos dentro de la red virtual que seleccione.  

A continuación puede ver la arquitectura de recursos que se implementará y usará en este escenario.

![Ilustración de la arquitectura de red virtual aislada de Cloud Shell.](media/private-vnet/data-diagram.png)

Antes de que pueda usar Cloud Shell en su propia instancia de Azure Virtual Network, deberá crear varios recursos para admitir esta funcionalidad. En este artículo se muestra cómo configurar los recursos necesarios mediante una plantilla de Resource Manager.

> [!NOTE]
> Estos recursos solo deben configurarse una vez para la red virtual. A continuación, se pueden compartir con todos los administradores que tengan acceso a la red virtual.

## <a name="required-network-resources"></a>Recursos de red necesarios

### <a name="virtual-network"></a>Virtual network
Una red virtual define el espacio de direcciones en el que se crean una o varias subredes.

Es necesario identificar la red virtual que se quiere usar para Cloud Shell. Normalmente, se trata de una red virtual existente que contiene los recursos que quiere administrar, o bien una red que se conecta con las redes que contienen los recursos.

### <a name="subnet"></a>Subnet
Dentro de la red virtual seleccionada, se debe usar una subred dedicada para los contenedores de Cloud Shell. Esta subred se delega al servicio de Azure Container Instances (ACI).  Cuando un usuario solicita un contenedor de Cloud Shell en una red virtual, Cloud Shell usa ACI para crear un contenedor que se encuentre en esta subred delegada.  No se puede crear ningún otro recurso en esta subred.

### <a name="network-profile"></a>Perfil de red
Un perfil de red es una plantilla de configuración de red para los recursos de Azure que especifica determinadas propiedades de red para un recurso.

### <a name="azure-relay"></a>Azure Relay
Una instancia de [Azure Relay](../azure-relay/relay-what-is-it.md) permite que se comuniquen dos puntos de conexión que no pueden comunicarse directamente. En este caso, se usa para permitir que el explorador del administrador se comunique con el contenedor de la red privada.

La instancia de Azure Relay que se use para Cloud Shell puede configurarse para controlar qué redes pueden acceder a los recursos del contenedor: 
- Accesible desde la red pública de Internet: en esta configuración, Cloud Shell ofrece una forma de comunicarse con los recursos internos desde fuera. 
- Accesible desde redes especificadas: en esta configuración, los administradores tendrán que acceder a Azure Portal desde un equipo que ejecute en la red correspondiente para usar Cloud Shell.

## <a name="storage-requirements"></a>Requisitos de almacenamiento
Como en la instancia estándar de Cloud Shell, se requiere una cuenta de almacenamiento para usar Cloud Shell en una red virtual. Cada administrador necesita un recurso compartido de archivos para almacenar sus archivos.  Debe poder acceder a la cuenta de almacenamiento desde la red virtual que usa Cloud Shell. 

## <a name="virtual-network-deployment-limitations"></a>Limitaciones de la implementación de redes virtuales
* Debido a los recursos de red adicionales que intervienen, iniciar Cloud Shell en una red virtual suele ser más lento que en una sesión estándar de Cloud Shell.

* Durante la versión preliminar, se admiten menos regiones para Cloud Shell en una red virtual. Actualmente se está limitado a: WestUS y WestCentralUS.

* [Azure Relay](../azure-relay/relay-what-is-it.md) no es un servicio gratuito, consulte los [precios](https://azure.microsoft.com/pricing/details/service-bus/) correspondientes. En el escenario de Cloud Shell, se usa una conexión híbrida para cada administrador mientras usan Cloud Shell. La conexión se apagará automáticamente una vez completada la sesión de Cloud Shell.

## <a name="register-the-resource-provider"></a>Registrar el proveedor de recursos

Es necesario registrar el proveedor de recursos Microsoft.ContainerInstances en la suscripción que contiene la red virtual que quiere usar. Seleccione la suscripción adecuada con `Set-AzContext -Subscription {subscriptionName}` y después ejecute lo siguiente:

```powershell
PS> Get-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance | select ResourceTypes,RegistrationState

ResourceTypes                             RegistrationState
-------------                             -----------------
{containerGroups}                         Registered
...
```

Si **RegistrationState** es `Registered`, no es necesario realizar ninguna acción. Si, por el contrario, es `NotRegistered`, ejecute `Register-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance`. 

## <a name="deploy-network-resources"></a>Implementación de recursos de red
 
### <a name="create-a-resource-group-and-virtual-network"></a>Creación de un grupo de recursos y red virtual
Si ya tiene una red virtual a la que le gustaría conectarse, omita esta sección.

En Azure Portal, o mediante la CLI de Azure, Azure PowerShell u otros, cree un grupo de recursos y una red virtual en el nuevo grupo de recursos, **ambos deben estar en la misma región**.

> [!NOTE]
> Si usa la versión preliminar pública, el grupo de recursos y la red virtual deben encontrarse en WestCentralUS o WestUS.

### <a name="arm-templates"></a>Plantillas de ARM
Use la [plantilla de inicio rápido de Azure](https://aka.ms/cloudshell/docs/vnet/template) para crear los recursos de Cloud Shell en una red virtual y la otra [plantilla de inicio rápido de Azure](https://aka.ms/cloudshell/docs/vnet/template/storage) para crear el almacenamiento necesario. Tome nota de los nombres de los recursos, en especial del nombre de recurso compartido de archivos.

### <a name="open-relay-firewall"></a>Apertura del firewall de retransmisión
Desplácese a la retransmisión que se creó con la plantilla anterior, seleccione "Redes" en la configuración y habilite el acceso de la red del explorador a la retransmisión. De forma predeterminada, solo puede acceder a la retransmisión desde la red virtual en la que se creó. 

### <a name="configuring-cloud-shell-to-use-a-virtual-network"></a>Configuración de Cloud Shell para usar una red virtual
> [!NOTE]
> Este paso se debe completar para todos los administradores que usarán Cloud Shell.

Después de implementar los pasos anteriores, diríjase a Cloud Shell en Azure Portal o en https://shell.azure.com. Una de estas experiencias debe usarse cada vez que quiera conectarse a una experiencia de Cloud Shell aislada.

> [!NOTE]
> Si ha usado Cloud Shell en el pasado, el clouddrive existente debe desmontarse. Para ello, ejecute `clouddrive unmount` desde una sesión de Cloud Shell activa y actualice la página.

Conéctese a Cloud Shell, se mostrará la primera experiencia de ejecución. Seleccione la experiencia de shell que prefiera, seleccione "Mostrar la configuración avanzada" y seleccione la casilla "Show VNET isolation settings" (Mostrar configuración de aislamiento de red virtual). Rellene todos los campos en el elemento emergente.  La mayoría de los campos se rellenan automáticamente con los recursos disponibles que se pueden asociar con Cloud Shell en una red virtual.  El usuario tendrá que rellenar el nombre del recurso compartido de archivos.


![Ilustración de la configuración de la primera experiencia de red virtual aislada de Cloud Shell.](media/private-vnet/vnet-settings.png)

## <a name="next-steps"></a>Pasos siguientes
[Más información sobre Azure Virtual Network](../virtual-network/virtual-networks-overview.md)