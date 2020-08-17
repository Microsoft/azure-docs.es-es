---
title: 'Inicio rápido: Diagnóstico de problemas de filtrado del tráfico de red de una máquina virtual: CLI de Azure'
titleSuffix: Azure Network Watcher
description: En esta guía de inicio rápido, aprenderá a diagnosticar los problemas que surgen al filtrar el tráfico de las máquinas virtuales utilizando la funcionalidad Comprobación del flujo de IP de Azure Network Watcher.
services: network-watcher
documentationcenter: network-watcher
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose a virtual machine (VM) network traffic filter problem that prevents communication to and from a VM.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: kumud
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 9fcc26d17b9bb1d67d85a1775c4df191fe3524f0
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502062"
---
# <a name="quickstart-diagnose-a-virtual-machine-network-traffic-filter-problem---azure-cli"></a>Inicio rápido: Diagnóstico de problemas de filtrado del tráfico de red de una máquina virtual: CLI de Azure

En esta guía de inicio rápido, va a implementar una máquina virtual y a comprobar después las comunicaciones de entrada y salida con una dirección IP y una dirección URL. Además, va a determinar la causa de un error de comunicación y cómo puede resolverlo.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI de Azure localmente, para este inicio rápido es preciso ejecutar la CLI de Azure, versión 2.0.28 o posterior. Ejecute `az --version` para ver cuál es la versión instalada. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Después de verificar la versión de la CLI de Azure, ejecute `az login` para crear una conexión con Azure. Los comandos de la CLI de Azure que aparecen en esta guía de inicio rápido tienen un formato que permite ejecutarlos en un shell de Bash.

## <a name="create-a-vm"></a>Crear una VM

Para poder crear una máquina virtual, debe crear un grupo de recursos que contenga la máquina virtual. Cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Cree la máquina virtual con [az vm create](/cli/azure/vm). Si las claves SSH no existen en la ubicación de claves predeterminada, el comando las crea. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`. En el ejemplo siguiente, se crea una máquina virtual llamada *myVm*:

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image UbuntuLTS \
  --generate-ssh-keys
```

La máquina virtual tarda en crearse unos minutos. No continúe con los pasos restantes hasta que la máquina virtual se haya creado y la CLI de Azure devuelva la salida.

## <a name="test-network-communication"></a>Prueba de la comunicación de red

Para probar la comunicación de red con Network Watcher, primero debe habilitar un monitor de red en la región en la que quiere probar la máquina virtual y, a continuación, usar la funcionalidad Comprobación del flujo de IP de Network Watcher para probar la comunicación.

### <a name="enable-network-watcher"></a>Habilitación de Network Watcher

Si ya dispone de un monitor de red habilitado en la región Este de EE. UU., sáltese la sección [Uso de la funcionalidad Comprobación del flujo de IP](#use-ip-flow-verify). Utilice el comando [az network watcher configure](/cli/azure/network/watcher#az-network-watcher-configure) para crear un monitor de red en la región EastUS:

```azurecli-interactive
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations eastus \
  --enabled
```

### <a name="use-ip-flow-verify"></a>Uso de la funcionalidad Comprobación del flujo de IP

Cuando se crea una máquina virtual, Azure permite y deniega el tráfico de entrada y salida de la máquina virtual conforme a unos valores predeterminados. Si lo desea, puede invalidar después los valores predeterminados de Azure para permitir o denegar otros tipos de tráfico. Para comprobar si se permite o deniega el tráfico dirigido a distintos destinos o el tráfico procedente de una dirección IP, utilice el comando [az network watcher test-ip-flow](/cli/azure/network/watcher#az-network-watcher-test-ip-flow).

Pruebe la comunicación de salida entre la máquina virtual y una de las direcciones IP de www.bing.com:

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 13.107.21.200:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

Después de unos segundos, el resultado devuelto le informa de que una regla de seguridad llamada **AllowInternetOutbound** ha permitido el acceso.

Pruebe la comunicación de salida entre la máquina virtual y 172.31.0.100:

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 172.31.0.100:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

El resultado devuelto le informa de que una regla de seguridad llamada **DefaultOutboundDenyAll** ha denegado el acceso.

Pruebe la comunicación de entrada que llega a la máquina virtual desde 172.31.0.100:

```azurecli-interactive
az network watcher test-ip-flow \
  --direction inbound \
  --local 10.0.0.4:80 \
  --protocol TCP \
  --remote 172.31.0.100:60000 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

El resultado devuelto le informa de que una regla de seguridad llamada **DefaultInboundDenyAll** ha denegado el acceso. Ahora que sabe qué reglas de seguridad están permitiendo o denegando el tráfico de entrada y salida en una máquina virtual, puede determinar cómo deben resolverse los problemas.

## <a name="view-details-of-a-security-rule"></a>Ver detalles de una regla de seguridad

Para determinar por qué las reglas de [Uso de la funcionalidad Comprobación del flujo de IP](#use-ip-flow-verify) permiten o deniegan la comunicación, revise las reglas de seguridad que están en vigor en la interfaz de red con el comando [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg):

```azurecli-interactive
az network nic list-effective-nsg \
  --resource-group myResourceGroup \
  --name myVmVMNic
```

La salida devuelta contiene el siguiente texto de la regla **AllowInternetOutbound**, que permite el acceso de salida a www.bing.com en un paso anterior de [Uso de la funcionalidad Comprobación del flujo de IP](#use-ip-flow-verify):

```
{
 "access": "Allow",
 "additionalProperties": {},
 "destinationAddressPrefix": "Internet",
 "destinationAddressPrefixes": [
  "Internet"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": [
  "1.0.0.0/8",
  "2.0.0.0/7",
  "4.0.0.0/6",
  "8.0.0.0/7",
  "11.0.0.0/8",
  "12.0.0.0/6",
  ...
 ],
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/AllowInternetOutBound",
 "priority": 65001,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

Puede ver en la salida anterior que **destinationAddressPrefix** es **Internet**. Sin embargo, no está muy claro qué relación tiene 13.107.21.200 con **Internet**. Verá que aparecen varios prefijos de direcciones bajo **expandedDestinationAddressPrefix**. Uno de los prefijos de la lista es **12.0.0.0/6**, que comprende el intervalo de direcciones IP de 12.0.0.1 a 15.255.255.254. Como 13.107.21.200 está dentro de ese intervalo, la regla **AllowInternetOutBound** permite el tráfico de salida. Asimismo, no aparece ninguna regla con una prioridad mayor (número más bajo) en la salida anterior que invalide esta regla. Para denegar la comunicación de salida hacia una dirección IP, podría agregar una regla de seguridad con una prioridad mayor que denegara la salida del puerto 80 hacia la dirección IP.

Cuando ejecutó el comando `az network watcher test-ip-flow` para probar la comunicación de salida hacia 172.131.0.100 en [Uso de la funcionalidad Comprobación del flujo de IP](#use-ip-flow-verify), la salida le informó de que la regla **DefaultOutboundDenyAll** había denegado la comunicación. La regla **DefaultOutboundDenyAll** es igual que la regla **DenyAllOutBound** que aparece en la siguiente salida del comando `az network nic list-effective-nsg`:

```
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllOutBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
}
```

La regla muestra **0.0.0.0/0** como **destinationAddressPrefix**. La regla deniega la comunicación de salida hacia 172.131.0.100 porque la dirección no está incluida en el valor **destinationAddressPrefix** de ninguna otra regla de salida en los resultados del comando `az network nic list-effective-nsg`. Para permitir la comunicación de salida, podría agregar una regla de seguridad con una prioridad mayor que permitiera el tráfico hacia el puerto 80 en 172.131.0.100.

Cuando ejecutó el comando `az network watcher test-ip-flow` para probar la comunicación de entrada procedente de 172.131.0.100 en [Uso de la funcionalidad Comprobación del flujo de IP](#use-ip-flow-verify), la salida le informó de que la regla **DefaultInboundDenyAll** había denegado la comunicación. La regla **DefaultInboundDenyAll** es igual que la regla **DenyAllInBound** que aparece en la siguiente salida del comando `az network nic list-effective-nsg`:

```
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Inbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllInBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

La regla **DenyAllInBound** se aplica porque, tal y como puede verse en la salida, no existe ninguna otra regla con una prioridad mayor en la salida del comando `az network nic list-effective-nsg` que permita la entrada en el puerto 80 del tráfico dirigido a la máquina virtual procedente de 172.131.0.100. Para permitir la comunicación de entrada, podría agregar una regla de seguridad con una prioridad mayor que permitiera el tráfico de entrada en el puerto 80 del tráfico procedente de 172.131.0.100.

Las pruebas de esta guía de inicio rápido permiten comprobar la configuración de Azure. Si estas pruebas devuelven resultados esperados pero sigue teniendo problemas de red, asegúrese de que no hay un firewall entre la máquina virtual y el punto de conexión con el que se está comunicando y que el sistema operativo de la máquina virtual no tiene un firewall que permita o deniegue la comunicación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede utilizar [az group delete](/cli/azure/group) para eliminar el grupo de recursos y todos los recursos que contenga:

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado una máquina virtual y diagnosticado los filtros del tráfico de entrada y de salida. Ha aprendido que las reglas de grupo de seguridad de red permiten o deniegan el tráfico de entrada y salida de una máquina virtual. Más información acerca de las [reglas de seguridad](../virtual-network/security-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) y [cómo crearlas](../virtual-network/manage-network-security-group.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-security-rule).

Incluso con la aplicación de los filtros de tráfico adecuados, la comunicación con una máquina virtual puede seguir experimentando errores debido a la configuración del enrutamiento. Para más información acerca de cómo diagnosticar los problemas de enrutamiento de red de la máquina virtual, consulte [Diagnose VM routing problems](diagnose-vm-network-routing-problem-cli.md) (Diagnóstico de problemas de enrutamiento de máquina virtual). Para diagnosticar problemas con los filtros de tráfico, la latencia o el enrutamiento de salida mediante una herramienta, consulte [Solución de problemas de conexiones](network-watcher-connectivity-cli.md).
