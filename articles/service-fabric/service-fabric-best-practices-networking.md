---
title: Procedimientos recomendados de red de Azure Service Fabric
description: Reglas y consideraciones de diseño para administrar la conectividad de red mediante Azure Service Fabric.
author: chrpap
ms.topic: conceptual
ms.date: 01/23/2019
ms.author: chrpap
ms.openlocfilehash: 0f25627c852befb03c2c32d741b8fe9b64cd4dc2
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88948970"
---
# <a name="networking"></a>Redes

Al crear y administrar clústeres de Azure Service Fabric, proporcionará conectividad de red para los nodos y aplicaciones. Los recursos de red incluyen intervalos de direcciones IP, redes virtuales, equilibradores de carga y grupos de seguridad de red. En este artículo, conocerá los procedimientos recomendados para estos recursos.

Revise los [patrones de redes de Service Fabric](./service-fabric-patterns-networking.md) de Azure para aprender a crear clústeres que usen las siguientes características: red virtual o subred existentes, dirección IP pública estática, equilibrador de carga solo interno y equilibrador de carga solo externo.

## <a name="infrastructure-networking"></a>Redes de infraestructura
Maximice el rendimiento de la máquina virtual con redes aceleradas; para ello, declare la propiedad *enableAcceleratedNetworking* en la plantilla de Resource Manager. El siguiente fragmento de código corresponde al elemento NetworkInterfaceConfigurations de un conjunto de escalado de máquinas virtuales que habilita las redes aceleradas:

```json
"networkInterfaceConfigurations": [
  {
    "name": "[concat(variables('nicName'), '-0')]",
    "properties": {
      "enableAcceleratedNetworking": true,
      "ipConfigurations": [
        {
        <snip>
        }
      ],
      "primary": true
    }
  }
]
```
El clúster de Service Fabric se puede aprovisionar en [Linux con redes aceleradas](../virtual-network/create-vm-accelerated-networking-cli.md) y [Windows con redes aceleradas](../virtual-network/create-vm-accelerated-networking-powershell.md).

Las redes aceleradas se admiten para las SKU de la serie de máquina virtual de Azure: D/DSv2, D/DSv3, E/ESv3, F/FS, FSv2 y Ms/Mms. Las redes aceleradas se probaron correctamente con la SKU Standard_DS8_v3 el 23 de enero de 2019 en un clúster de Windows de Service Fabric y con Standard_DS12_v2 el 29 de enero de 2019 en un clúster de Linux de Service Fabric.

Para habilitar las redes aceleradas en un clúster de Service Fabric existente, primero debe [escalar un clúster de Service Fabric horizontalmente mediante la incorporación de un conjunto de escalado de máquinas virtuales](./virtual-machine-scale-set-scale-node-type-scale-out.md), para realizar lo siguiente:
1. Aprovisionar un elemento NodeType con las redes aceleradas habilitadas
2. Migrar los servicios y su estado al elemento NodeType aprovisionado con las redes aceleradas habilitadas

Para habilitar las redes aceleradas en un clúster existente, es necesario escalar horizontalmente la infraestructura dado que si se habilitan tal y como está, se produciría tiempo de inactividad puesto que todas las máquinas virtuales de un conjunto de disponibilidad se tendrían que [detener y desasignar antes de habilitarlas en una NIC existente](../virtual-network/create-vm-accelerated-networking-cli.md#enable-accelerated-networking-on-existing-vms).

## <a name="cluster-networking"></a>Redes de clúster

* Los clústeres de Service Fabric se pueden implementar en una red virtual existente mediante los pasos descritos en [Patrones de redes de Service Fabric](./service-fabric-patterns-networking.md).

* Se recomiendan grupos de seguridad de red (NSG) para que los tipos de nodo restrinjan el tráfico entrante y saliente en su clúster. Asegúrese de que los puertos necesarios estén abiertos en el NSG. 

* El tipo de nodo principal, que contiene los servicios del sistema de Service Fabric, no debe exponerse a través del equilibrador de carga externo y puede exponerse mediante un [equilibrador de carga interno](./service-fabric-patterns-networking.md#internal-only-load-balancer).

* Use una [dirección IP pública estática](./service-fabric-patterns-networking.md#static-public-ip-address-1) para el clúster.

## <a name="network-security-rules"></a>Reglas de seguridad de red

Las reglas básicas aquí son las mínimas para un bloqueo de seguridad de un clúster de Service Fabric administrado de Azure. Si no se abren los puertos siguientes o no se incluye la dirección IP o URL en la lista blanca, se evita que el clúster funcione correctamente y es posible que no se admita. Con este conjunto de reglas, es estrictamente necesario usar [actualizaciones automáticas de imágenes del sistema operativo](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md); de lo contrario, es necesario abrir puertos adicionales.

### <a name="inbound"></a>Entrante 
|Priority   |Nombre               |Port        |Protocolo  |Source             |Destination       |Acción   
|---        |---                |---         |---       |---                |---               |---
|3900       |Azure              |19080       |TCP       |Internet           |VirtualNetwork    |Allow
|3910       |Remoto             |19000       |TCP       |Internet           |VirtualNetwork    |Allow
|3920       |Clúster            |1025-1027   |TCP       |VirtualNetwork     |VirtualNetwork    |Allow
|3930       |Efímero          |49152-65534 |TCP       |VirtualNetwork     |VirtualNetwork    |Allow
|3940       |Application        |20000-30000 |TCP       |VirtualNetwork     |VirtualNetwork    |Allow
|3950       |SMB                |445         |TCP       |VirtualNetwork     |VirtualNetwork    |Allow
|3960       |RDP                |3389-3488   |TCP       |Internet           |VirtualNetwork    |Denegar
|3970       |SSH                |22          |TCP       |Internet           |VirtualNetwork    |Denegar
|3980       |Punto de conexión personalizado    |80          |TCP       |Internet           |VirtualNetwork    |Allow
|4100       |Bloquear entrada      |443         |Any       |Any                |Any               |Allow

Más información sobre las reglas de seguridad de entrada:

* **Azure**. Service Fabric Explorer usa este puerto para examinar y administrar el clúster; el proveedor de recursos de Service Fabric también lo usa para consultar información sobre el clúster a fin de mostrarla en el Portal de administración de Azure. Si este puerto no es accesible desde el proveedor de recursos de Service Fabric, se ve un mensaje como "Nodos no encontrados" o "UpgradeServiceNotReachable" en Azure Portal y el nodo y la lista de aplicaciones aparecen vacíos. Esto significa que si quiere tener visibilidad del clúster en el Portal de administración de Azure, el equilibrador de carga debe exponer una dirección IP pública y el NSG debe permitir el tráfico entrante de 19080.  

* **Cliente**. Punto de conexión de cliente para API como REST, PowerShell o CLI. 

* **Clúster**. Se usa para la comunicación entre nodos; nunca se debe bloquear.

* **Efímero**. Service Fabric usará una parte de estos puertos como puertos de aplicación, y el resto estará disponible para el sistema operativo. También asigna este rango al rango existente en el sistema operativo, por lo que puede usar los rangos de este ejemplo para todos los fines. Asegúrese de que la diferencia entre los puertos de inicio y finalización es al menos de 255. Puede encontrarse con conflictos si esta diferencia es demasiado baja, ya que este intervalo se comparte con el sistema operativo. Para ver el rango de puertos dinámicos configurado, ejecute *netsh int ipv4 show dynamic port tcp*. Estos puertos no son necesarios para clústeres de Linux.

* **Aplicación**. El intervalo de puertos de las aplicaciones debería ser lo suficientemente amplio como para contemplar el requisito de punto de conexión de estas. Este intervalo debería quedar excluido del intervalo de puertos dinámicos de la máquina, es decir, el intervalo ephemeralPorts tal y como se establece en la configuración. Service Fabric usa estos puertos siempre que se requieren nuevos puertos y se ocupa de la apertura del firewall para ellos en los nodos.

* **SMB**. El servicio ImageStore usa el protocolo SMB en dos escenarios. Este puerto es necesario para que los nodos descarguen los paquetes de ImageStore y para replicarlos entre las réplicas. 

* **RDP**. Opcional, si se requiere RDP desde Internet o VirtualNetwork para escenarios de jumpbox. 

* **SSH**. Opcional, si se requiere SSH desde Internet o VirtualNetwork para escenarios de jumpbox.

* **Punto de conexión personalizado**. Ejemplo para que la aplicación habilite un punto de conexión accesible de Internet.

### <a name="outbound"></a>Saliente

|Priority   |Nombre               |Port        |Protocolo  |Source             |Destination       |Acción   
|---        |---                |---         |---       |---                |---               |---
|3900       |Red            |Any         |TCP       |VirtualNetwork     |VirtualNetwork    |Allow
|3910       |Proveedor de recursos  |443         |TCP       |VirtualNetwork     |ServiceFabric     |Allow
|3920       |Actualizar            |443         |TCP       |VirtualNetwork     |Internet          |Allow
|3950       |Bloquear salida     |Any         |Any       |Any                |Any               |Denegar

Más información sobre las reglas de seguridad de salida:

* **Red**. Canal de comunicación para subredes y para otras redes virtuales.

* **Proveedor de recursos**. Conexión por parte de UpgradeService para ejecutar todas las implementaciones de ARM del proveedor de recursos de Service Fabric.

* **Actualización**. Servicio de actualización que usa la dirección download.microsoft.com para obtener los bits, es necesario para la instalación, para volver a crear imágenes y para las actualizaciones del entorno de ejecución. El servicio funciona con direcciones IP dinámicas. En el escenario de un equilibrador de carga "solo interno", debe agregarse un equilibrador de carga externo adicional a la plantilla con una regla que permita el tráfico de salida del puerto 443. Opcionalmente, este puerto se puede bloquear después de una instalación correcta, pero en este caso el paquete de actualización debe distribuirse a los nodos o el puerto tiene que estar abierto durante ese breve período de tiempo; después es necesaria una actualización manual.

Use Azure Firewall con [registro de flujo de grupos de seguridad de red](../network-watcher/network-watcher-nsg-flow-logging-overview.md) y [análisis de tráfico](../network-watcher/traffic-analytics.md) para realizar el seguimiento de los problemas del bloqueo de seguridad. La plantilla de ARM [Service Fabric con NSG](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure-NSG) es un buen ejemplo para empezar. 


## <a name="application-networking"></a>Redes de aplicación

* Para ejecutar cargas de trabajo de contenedor de Windows, use el [modo de red abierto](./service-fabric-networking-modes.md#set-up-open-networking-mode) para facilitar la comunicación de servicio a servicio.

* Use un proxy inverso, como [Traefik](https://docs.traefik.io/v1.6/configuration/backends/servicefabric/) o el [proxy inverso de Service Fabric](./service-fabric-reverseproxy.md) para exponer los puertos de aplicación comunes, como 80 o 443.

* En el caso de los contenedores de Windows hospedados en máquinas que no tienen una conexión física que no pueden extraer los niveles de base del almacenamiento en la nube de Azure, anule el comportamiento del nivel externo con el uso de la marca [--allow-nondistributable-artifacts](/virtualization/windowscontainers/about/faq#how-do-i-make-my-container-images-available-on-air-gapped-machines) en el demonio de Docker.

## <a name="next-steps"></a>Pasos siguientes

* Creación de un clúster en máquinas virtuales o equipos que ejecutan Windows Server: [Creación de un clúster independiente con Windows Server](service-fabric-cluster-creation-for-windows-server.md)
* Creación de un clúster en máquinas virtuales o equipos que ejecutan Linux: [Crear un clúster Linux](service-fabric-cluster-creation-via-portal.md)
* Más información sobre las [opciones de soporte técnico de Service Fabric](service-fabric-support.md)

[NSGSetup]: ./media/service-fabric-best-practices/service-fabric-nsg-rules.png
