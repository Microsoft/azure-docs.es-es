---
title: Implementación de redes de contenedor de Azure Virtual Network| Microsoft Docs
description: Obtenga información sobre cómo implementar el complemento de interfaz de red de contenedores (CNI) de Azure Virtual Network para clústeres de Kubernetes.
services: virtual-network
documentationcenter: na
author: aanandr
manager: NarayanAnnamalai
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 9/18/2018
ms.author: aanandr
ms.custom: ''
ms.openlocfilehash: 09a0574666441138c143932e843080e8745f1b40
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289584"
---
# <a name="deploy-the-azure-virtual-network-container-network-interface-plug-in"></a>Implementación del complemento de interfaz de red de contenedor de Azure Virtual Network

El complemento de interfaz de red de contenedor (CNI) de Azure Virtual Network se instala en una máquina virtual de Azure y ofrece funcionalidades de red virtual a los pods de Kubernetes y los contenedores de Docker. Para más información sobre el complemento, consulte [Enable containers to use Azure Virtual Network capabilities](container-networking-overview.md) (Habilitación de contenedores para que puedan usar las funcionalidades de Azure Virtual Network). Además, el complemento se puede usar con Azure Kubernetes Service (AKS); para ello, elija la opción [Advanced Networking](../aks/networking-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (Redes avanzadas), que coloca automáticamente los contenedores de AKS en una red virtual.

## <a name="deploy-plug-in-for-acs-engine-kubernetes-cluster"></a>Implementación del complemento para un clúster de Kubernetes con ACS-Engine

ACS-Engine implementa un clúster de Kubernetes con una plantilla de Azure Resource Manager. La configuración del clúster se especifica en un archivo JSON que se pasa a la herramienta al generar la plantilla. Para ver la lista completa de las configuraciones de clúster compatibles y sus descripciones, consulte [Microsoft Azure Container Service Engine - definición de clúster](https://github.com/Azure/acs-engine/blob/master/docs/clusterdefinition.md) (Motor de Microsoft Azure Container Service: definiciones de clúster). Este es el complemento de red predeterminado para los clústeres creados con ACS-Engine. Las siguientes opciones de configuración de red son importantes a la hora de configurar el complemento:

  | Configuración                              | Descripción                                                                                                           |
  |--------------------------------------|------------------------------------------------------------------------------------------------------                 |
  | firstConsecutiveStaticIP             | Dirección IP que se asigna al nodo maestro. Es una opción de configuración obligatoria.                                     |
  | clusterSubnet en kubernetesConfig | CIDR de la subred de red virtual donde se implementa el clúster y desde la cuál se asignan las direcciones IP a los pods.   |
  | vnetSubnetId en masterProfile     | Especifica el identificador de recurso de Azure Resource Manager de la subred en la que el clúster se va a implementar.                    |
  | vnetCidr                             | CIDR de la red virtual en la que se va a implementar el clúster.                                                             |
  | max-Pods en kubeletConfig         | Número máximo de pods en cada máquina virtual de agente. Para el complemento, el valor predeterminado es 30. Puede especificar hasta 250.  |

### <a name="example-configuration"></a>Ejemplo de configuración

El siguiente ejemplo JSON es para un clúster con las siguientes propiedades:
-   Un nodo maestro y dos nodos agente 
-   Se implementa en una subred llamada *KubeClusterSubnet* (10.0.0.0/20) y los nodos maestro y agente residen en ella.

```json
{
  "apiVersion": "vlabs",
  "properties": {
    "orchestratorProfile": {
      "orchestratorType": "Kubernetes",
      "kubernetesConfig": {
        "clusterSubnet": "10.0.0.0/20" --> Subnet allocated for the cluster
      }
    },
    "masterProfile": {
      "count": 1,
      "dnsPrefix": "ACSKubeMaster",
      "vmSize": "Standard_A2",
      "vnetSubnetId": "/subscriptions/<subscription ID>/resourceGroups/<Resource Group Name>/providers/Microsoft.Network/virtualNetworks/<Vnet Name>/subnets/KubeClusterSubnet",
      "firstConsecutiveStaticIP": "10.0.1.50", --> IP address allocated to the Master node
"vnetCidr": "10.0.0.0/16" --> Virtual network address space
    },
    "agentPoolProfiles": [
      {
        "name": "k8sagentpoo1",
        "count": 2,
        "vmSize": "Standard_A2_v2",
"vnetSubnetId": "/subscriptions/<subscription ID>/resourceGroups/<Resource Group Name>/providers/Microsoft.Network/virtualNetworks/<VNet Name>/subnets/KubeClusterSubnet",
        "availabilityProfile": "AvailabilitySet"
      }
    ],
    "linuxProfile": {
      "adminUsername": "KubeServerAdmin",
      "ssh": {
        "publicKeys": [
          {…}
        ]
      }
    },
    "servicePrincipalProfile": {
      "clientId": "dd438987-aa12-4754-b47d-375811889714",
      "secret": "azure123"
    }
  }
}
```

## <a name="deploy-plug-in-for-a-kubernetes-cluster"></a>Implementación del complemento para un clúster de Kubernetes

Complete los pasos siguientes para instalar el complemento en cada máquina virtual de Azure en un clúster de Kubernetes:

1. [Descarga e instalación del complemento](#download-and-install-the-plug-in)
2. Asigne previamente un grupo de direcciones IP de red virtual a cada máquina virtual desde la cual se asignarán las direcciones IP a los pods. Cada máquina virtual de Azure viene con una dirección IP privada de red virtual principal en cada interfaz de red. Las direcciones IP del grupo se agregan como direcciones secundarias (*ipconfigs*) en la interfaz de red de la máquina virtual, con una de las siguientes opciones:

   - **CLI**:  [asignación de varias direcciones IP mediante la CLI de Azure](virtual-network-multiple-ip-addresses-cli.md)
   - **PowerShell**:  [asignación de varias direcciones IP mediante PowerShell](virtual-network-multiple-ip-addresses-powershell.md)
   - **Portal**:  [asignación de varias direcciones IP mediante Azure Portal](virtual-network-multiple-ip-addresses-portal.md)
   - **Plantilla de Azure Resource Manager**:  [asignación de varias direcciones IP mediante plantillas](virtual-network-multiple-ip-addresses-template.md)

   Asegúrese de agregar suficientes direcciones IP para todos los pods que se espera que aparezcan en la máquina virtual.

3. Seleccione el complemento para proporcionar conexión de red al clúster; para ello, pase al kubelet la opción de línea de comandos `–network-plugin=cni` durante la creación del clúster. De forma predeterminada, Kubernetes busca el complemento y el archivo de configuración en los directorios donde ya están instalados.
4. Si desea que los pods tengan acceso a Internet, agregue la siguiente regla *iptables* en las máquinas virtuales Linux para aplicar NAT de origen al tráfico de Internet. En el ejemplo siguiente, el intervalo IP especificado es 10.0.0.0/8.

   ```bash
   iptables -t nat -A POSTROUTING -m iprange ! --dst-range 168.63.129.16 -m
   addrtype ! --dst-type local ! -d 10.0.0.0/8 -j MASQUERADE
   ```

   La regla aplica NAT al tráfico que no está destinado a los intervalos IP especificados. Se da por hecho que todo el tráfico fuera de los intervalos anteriores es tráfico de Internet. Puede elegir especificar los intervalos IP de la red virtual de la máquina virtual, los de la red virtual emparejada y los de las redes locales.

   Las máquinas virtuales Windows aplican NAT automáticamente al tráfico cuyo destino está fuera de la subred a la que pertenece la máquina virtual. No es posible especificar intervalos IP personalizados.

Después de completar los pasos anteriores, a los pods que aparecen en las máquinas virtuales de agente de Kubernetes se les asignan automáticamente direcciones IP privadas de la red virtual.

## <a name="deploy-plug-in-for-docker-containers"></a>Implementación del complemento para contenedores de Docker

1. [Descarga e instalación del complemento](#download-and-install-the-plug-in)
2. Cree contenedores de Docker con el siguiente comando:

   ```
   ./docker-run.sh \<container-name\> \<container-namespace\> \<image\>
   ```

Los contenedores comienzan a recibir automáticamente las direcciones IP del grupo asignado. Si desea cargar equilibrar el tráfico a los contenedores de Docker, se debe colocar detrás de un equilibrador de carga de software, y debe configurar un sondeo de equilibrador de carga, de la misma manera que se crea una directiva y los sondeos para una máquina virtual.

### <a name="cni-network-configuration-file"></a>Archivo de configuración de red de CNI

El archivo de configuración de red de CNI se describe en formato JSON. De forma predeterminada, se encuentra en `/etc/cni/net.d` para Linux y `c:\cni\netconf` para Windows. El archivo especifica la configuración del complemento y es diferente para Windows y Linux. El código JSON siguiente es un archivo de configuración de Linux de ejemplo, seguido de una explicación de algunas de las opciones de configuración clave. No es necesario hacer ningún cambio en el archivo:

```json
{
       "cniVersion":"0.3.0",
       "name":"azure",
       "plugins":[
          {
             "type":"azure-vnet",
             "mode":"bridge",
             "bridge":"azure0",
             "ipam":{
                "type":"azure-vnet-ipam"
             }
          },
          {
             "type":"portmap",
             "capabilities":{
                "portMappings":true
             },
             "snat":true
          }
       ]
}
```

#### <a name="settings-explanation"></a>Explicación de las opciones de configuración

- **cniVersion**: los complementos de CNI de Azure Virtual Network son compatibles con las versiones 0.3.0 y 0.3.1 de la  [especificación CNI](https://github.com/containernetworking/cni/blob/master/SPEC.md).
- **name**: nombre de la red. Esta propiedad se puede establecer en un valor único.
- **type**: nombre del complemento de red. Se establece en *azure-vnet*.
- **mode**: modo de funcionamiento. Este campo es opcional. El único modo admitido es "bridge". Para más información, consulte los  [modos de funcionamiento](https://github.com/Azure/azure-container-networking/blob/master/docs/network.md).
- **bridge**: nombre del puente que se usará para conectar los contenedores a una red virtual. Este campo es opcional. Si se omite, el complemento elige automáticamente un nombre único, basado en el índice de la interfaz maestra.
- **ipam type**: nombre del complemento de IPAM. Siempre se establece en *azure-vnet-ipam*.

## <a name="download-and-install-the-plug-in"></a>Descarga e instalación del complemento

Descargue el complemento de [GitHub](https://github.com/Azure/azure-container-networking/releases). Descargue la versión más reciente para la plataforma que esté usando:

- **Linux**: [azure-vnet-cni-linux-amd64-\<version no.\>.tgz](https://github.com/Azure/azure-container-networking/releases/download/v1.0.12-rc3/azure-vnet-cni-linux-amd64-v1.0.12-rc3.tgz)
- **Windows**: [azure-vnet-cni-windows-amd64-\<version no.\>.zip](https://github.com/Azure/azure-container-networking/releases/download/v1.0.12-rc3/azure-vnet-cni-windows-amd64-v1.0.12-rc3.zip)

Copie el script de instalación para [Linux](https://github.com/Azure/azure-container-networking/blob/master/scripts/install-cni-plugin.sh) o [Windows](https://github.com/Azure/azure-container-networking/blob/master/scripts/Install-CniPlugin.ps1) en su equipo. Guarde el script en un directorio `scripts` del equipo y asigne al archivo el nombre `install-cni-plugin.sh` para Linux o `install-cni-plugin.ps1` para Windows. Para instalar el complemento, ejecute el script adecuado para su plataforma y especifique la versión del complemento que está usando. Por ejemplo, podría especificar *v1.0.12-rc3*:

   ```bash
   \$scripts/install-cni-plugin.sh [version]
   ```

   ```powershell
   scripts\\ install-cni-plugin.ps1 [version]
   ```

El script instala el complemento en `/opt/cni/bin` para Linux y en `c:\cni\bin` para Windows. El complemento instalado incluye un archivo de configuración de red simple que funciona después de la instalación. No es necesario actualizarlo. Para más información sobre las opciones de configuración del archivo, consulte [Archivo de configuración de red de CNI](#cni-network-configuration-file).