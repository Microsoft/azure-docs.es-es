---
title: Configuración de conectividad de máquinas virtuales en SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Configuración de conectividad de máquinas virtuales para usar SAP HANA en Azure (instancias grandes).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/25/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 3231241e2fbc6e02725b5611d05b9ee4f0f1a35a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082178"
---
# <a name="connecting-azure-vms-to-hana-large-instances"></a>Conexión de las máquinas virtuales de Azure a HANA Instancias grandes

En el artículo [¿Qué es SAP HANA en Azure (instancias grandes)?](./hana-overview-architecture.md) se menciona que la implementación mínima de HANA (instancias grandes) con el nivel de aplicación de SAP en Azure es similar a la siguiente:

![Red virtual de Azure conectada a SAP HANA en Azure (Instancias grandes) y localmente](./media/hana-overview-architecture/image1-architecture.png)

Al examinar más de cerca el lado de la red virtual de Azure, observamos que se necesita:

- La definición de una red virtual de Azure en la que se van a implementar las máquinas virtuales del nivel de aplicación de SAP.
- La definición de una subred predeterminada en la red virtual de Azure que es realmente donde se implementan las máquinas virtuales.
- La red virtual de Azure que se crea debe tener como mínimo una subred de VM y una subred de puerta de enlace de la red virtual de Azure ExpressRoute. A estas subredes se les deben asignar los intervalos de direcciones IP, como se especifica y trata en las siguientes secciones.


## <a name="create-the-azure-virtual-network-for-hana-large-instances"></a>Creación de una red virtual de Azure para HANA (instancias grandes)

>[!Note]
>La red virtual de Azure para HANA (instancias grandes) se debe crear mediante el modelo de implementación de Azure Resource Manager. La solución HANA (instancias grandes) no admite el antiguo modelo de implementación de Azure, habitualmente conocido como el modelo de implementación clásica.

Para crear la red virtual, puede usar Azure Portal, PowerShell, una plantilla de Azure o la CLI de Azure. (Para más información, consulte [Creación de una red virtual mediante Azure Portal](../../../virtual-network/manage-virtual-network.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#create-a-virtual-network)). En el ejemplo siguiente, observamos una red virtual creada mediante Azure Portal.

Cuando se habla del **espacio de direcciones** en esta documentación, se hace referencia al espacio de direcciones que puede usar la red virtual de Azure. Este espacio de direcciones también es el intervalo de direcciones que la red virtual usa para la propagación de la ruta BGP. Este **espacio de direcciones** puede verse a continuación:

![El espacio de direcciones de una red virtual de Azure se muestra en Azure Portal](./media/hana-overview-connectivity/image1-azure-vnet-address-space.png)

En el ejemplo anterior, con 10.16.0.0/16, se asignó un intervalo de direcciones IP amplio y grande a la red virtual de Azure. Por tanto, todos los intervalos de direcciones IP de las subredes subsiguientes dentro de esta red virtual pueden tener sus intervalos dentro de ese espacio de direcciones. Por lo general, no se recomienda un intervalo de direcciones tan grande para una sola red virtual en Azure. Pero examinemos las subredes definidas en la red virtual de Azure:

![Subredes de la red virtual de Azure y sus intervalos de direcciones IP](./media/hana-overview-connectivity/image2b-vnet-subnets.png)

Vemos una red virtual con una primera subred de máquina virtual (denominada aquí como "default") y una subred llamada "GatewaySubnet".

En los dos gráficos anteriores, el **espacio de direcciones de la red virtual** abarca tanto el **intervalo de direcciones IP de la subred de la máquina virtual de Azure** como el de la puerta de enlace de red virtual.

Puede restringir el **espacio de direcciones de la red virtual** a los intervalos específicos que cada subred usa. También puede definir el **espacio de direcciones de la red virtual** de una red virtual como varios intervalos específicos, tal como se muestra aquí:

![Espacio de direcciones de la red virtual de Azure con dos espacios](./media/hana-overview-connectivity/image3-azure-vnet-address-space_alternate.png)

En este caso, el **espacio de direcciones de la red virtual** tiene dos espacios definidos. Son iguales que los intervalos de direcciones IP definidos para el intervalo de direcciones IP de la subred de la máquina virtual de Azure y la puerta de enlace de la red virtual. 

Puede usar la nomenclatura que desee para estas subredes de inquilino (subredes de máquina virtual). Pero **siempre debe haber una, y solo una, subred de puerta de enlace para cada red virtual** que se conecte al circuito ExpressRoute de SAP HANA en Azure (instancias grandes). **Esta subred de puerta de enlace se tiene que denominar "GatewaySubnet"** para garantizar que la puerta de enlace de ExpressRoute está correctamente colocada.

> [!WARNING] 
> Es fundamental que la subred de la puerta de enlace siempre se denomine "GatewaySubnet".

Puede usar varias subredes de máquina virtual e intervalos de direcciones no contiguos. Estos intervalos de direcciones los debe cubrir el **espacio de direcciones de la red virtual** de la red virtual. Pueden estar en forma agregada. También pueden estar en una lista de los intervalos exactos de las subredes de máquina virtual y la subred de puerta de enlace.

A continuación, aparece un resumen de los hechos importantes sobre una red virtual de Azure que se conecta a HANA (instancias grandes):

- Debe enviar el **espacio de direcciones de la red virtual** a Microsoft cuando haga una implementación inicial de HANA (Instancias grandes). 
- El **espacio de direcciones de la red virtual** puede ser un intervalo mayor que abarque tanto el intervalo de direcciones IP de la subred de la máquina virtual de Azure como la puerta de enlace de la red virtual.
- O puede enviar varios intervalos que abarquen los distintos intervalos de direcciones IP de la subred de la máquina virtual y el intervalo de direcciones IP de la puerta de enlace de red virtual.
- El **espacio de direcciones de la red virtual** definido se usa para la propagación de enrutamiento BGP.
- El nombre de la subred de la puerta de enlace debe ser: **"GatewaySubnet"** .
- El espacio de direcciones se usa como filtro en el lado de HANA (instancias grandes) para permitir o impedir el tráfico a las unidades de HANA (instancias grandes) desde Azure. La información de enrutamiento BGP de la red virtual de Azure y los intervalos de direcciones IP que están configurados para el filtrado en el lado de HANA (instancias grandes) deben coincidir. En caso contrario, pueden producirse problemas de conectividad.
- Hay algunos detalles sobre la subred de la puerta de enlace que se describen más adelante, en la sección sobre la **conexión de una red virtual a ExpressRoute de HANA (instancias grandes)** .



## <a name="different-ip-address-ranges-to-be-defined"></a>Distintos intervalos de direcciones IP para definir 

Ya se presentaron anteriormente algunos de los intervalos de direcciones IP necesarios para implementar HANA (Instancias grandes). Pero hay más intervalos de direcciones IP que también son importantes. No todos los intervalos de direcciones IP siguientes deben enviarse a Microsoft. Sin embargo, deberá definirlos antes de enviar una solicitud para la implementación inicial:

- **Espacio de direcciones de red virtual**: El **espacio de direcciones de la red virtual** es los intervalos de direcciones IP que asigna al parámetro de espacio de direcciones en las redes virtuales de Azure. Estas redes se conectan con el entorno de SAP HANA (instancias grandes). Se recomienda que este parámetro de espacio de direcciones sea un valor de varias líneas. Debe constar del intervalo de subred de la máquina virtual de Azure y de los intervalos de subred de la puerta de enlace de Azure. Este intervalo de subred se muestra en los gráficos anteriores. No se debe solapar con los intervalos de direcciones locales, ni con los del grupo de direcciones IP de servidor, ni con los de ER-P2P. ¿Cómo se obtienen estos intervalos de direcciones IP? El equipo de red o el proveedor de servicios de la empresa debe proporcionar uno o varios intervalos de direcciones IP que no se usen en la red. Por ejemplo, la subred de la máquina virtual de Azure es 10.0.1.0/24 y la subred de la subred de puerta de enlace de Azure es 10.0.2.0/28.  Se recomienda que el espacio de direcciones de la red virtual de Azure se defina como: 10.0.1.0/24 y 10.0.2.0/28. Aunque se pueden agregar los valores del espacio de direcciones, se recomienda asociarlos con los intervalos de la subred. De esta manera, evita reutilizar accidentalmente los intervalos de direcciones IP no utilizados dentro de espacios de direcciones más grandes en cualquier otra parte de la red. **El espacio de direcciones de red virtual es un intervalo de direcciones IP. Se debe enviar a Microsoft cuando se solicita una implementación inicial**.
- **Intervalo de direcciones IP de la subred de máquina virtual de Azure:** Este intervalo de direcciones IP es el que asigna al parámetro de la subred de red virtual de Azure. Este parámetro se encuentra en la red virtual de Azure y se conecta al entorno de SAP HANA (instancias grandes). Este intervalo de direcciones IP se usa para asignar direcciones IP a las máquinas virtuales de Azure. Las direcciones IP fuera de este intervalo pueden conectarse a los servidores de SAP HANA (Instancias grandes). Si es necesario, puede usar varias subredes de máquina virtual de Azure. Se recomienda un bloque CIDR /24 para cada subred de máquina virtual de Azure. Este intervalo de direcciones debe ser parte de los valores que se usan en el espacio de direcciones de la red virtual de Azure. ¿Cómo se obtiene este intervalo de direcciones IP? El equipo de redes corporativas o el proveedor de servicios debe proporcionar un intervalo de direcciones IP que no se use dentro de la red.
- **Intervalo de direcciones IP de subred de la puerta de enlace de red virtual:** Según las características que va a usar, el tamaño recomendado es:
   - Puerta de enlace de ExpressRoute de ultrarrendimiento: bloque de direcciones /26: se necesita para la clase de tipo II de SKU.
   - Coexistencia con VPN y ExpressRoute mediante una puerta de enlace de red virtual de ExpressRoute de alto rendimiento (o inferior): bloque de direcciones /27.
   - Resto de situaciones: bloque de direcciones /28. Este intervalo de direcciones debe formar parte de los valores utilizados en "Espacio de direcciones de la red virtual". Este intervalo de direcciones debe formar parte de los valores utilizados en los valores del espacio de direcciones de la red virtual de Azure que se envían a Microsoft. ¿Cómo se obtiene este intervalo de direcciones IP? El equipo de redes corporativas o el proveedor de servicios debe proporcionar un intervalo de direcciones IP que actualmente no se usa dentro de la red. 
- **Intervalo de direcciones para conectividad ER-P2P:** Es el intervalo de direcciones IP para la conexión ExpressRoute (ER) P2P de SAP HANA (Instancias grandes). Este intervalo de direcciones IP debe ser un intervalo de direcciones IP CIDR /29. Este intervalo no se debe superponer con los intervalos de direcciones locales ni con otros intervalos de direcciones IP de Azure. Este intervalo de direcciones IP se usa para configurar la conectividad ER desde la puerta de enlace virtual de ExpressRoute a los servidores de SAP HANA (instancias grandes). ¿Cómo se obtiene este intervalo de direcciones IP? El equipo de redes corporativas o el proveedor de servicios debe proporcionar un intervalo de direcciones IP que actualmente no se usa dentro de la red. **Este rango es un intervalo de direcciones IP. Se debe enviar a Microsoft cuando se solicita una implementación inicial**.  
- **Intervalo de direcciones del grupo de direcciones IP de servidor:** Este intervalo de direcciones IP se usa para asignar direcciones IP individuales a los servidores de HANA (instancias grandes). El tamaño recomendado de la subred es un bloque CIDR /24. Si es necesario, puede ser inferior, con tan solo 64 direcciones IP. De este intervalo, las primeras 30 direcciones IP están reservadas para uso de Microsoft. Asegúrese de tener en cuenta este hecho al elegir el tamaño del intervalo. Este intervalo no se debe superponer con las direcciones IP locales ni con otras direcciones IP de Azure. ¿Cómo se obtiene este intervalo de direcciones IP? El equipo de redes corporativas o el proveedor de servicios debe proporcionar un intervalo de direcciones IP que actualmente no se usa dentro de la red.  **Se debe enviar a Microsoft este intervalo de direcciones IP al solicitar una implementación inicial**.

Intervalos de direcciones IP opcionales que a la larga deben enviarse a Microsoft:

- Si elige usar [ExpressRoute Global Reach](../../../expressroute/expressroute-global-reach.md) para permitir el enrutamiento directo desde el entorno local a las unidades de HANA (Instancias grandes), debe reservar otro intervalo de direcciones IP /29. Este intervalo no se puede superponer con ninguno de los otros intervalos de direcciones IP definidos antes.
- Si opta por usar [ExpressRoute Global Reach](../../../expressroute/expressroute-global-reach.md) para permitir el enrutamiento directo entre inquilinos de HANA (Instancias grandes) de diferentes regiones de Azure, debe reservar otro intervalo de direcciones IP /29. Este intervalo no se puede superponer con ninguno de los otros intervalos de direcciones IP definidos antes.

Para más información sobre ExpressRoute Global Reach y su uso en torno a instancias grandes de HANA, consulte los documentos:

- [Arquitectura de red de SAP HANA (instancias grandes)](./hana-network-architecture.md)
- [Conexión de una red virtual a instancias grandes de HANA](./hana-connect-vnet-express-route.md)
 
Deberá definir y planificar los intervalos de direcciones IP que se describieron anteriormente. Sin embargo, no es necesario transmitirlos todos a Microsoft. Los intervalos de direcciones IP que debe notificar a Microsoft son:

- Espacios de direcciones de red virtual de Azure
- Intervalo de direcciones para conectividad ER-P2P
- Intervalo de direcciones del grupo de direcciones IP de servidor

Si agrega las redes virtuales que se deben conectar a HANA (instancias grandes), debe enviar el nuevo espacio de direcciones de la red virtual de Azure que agrega a Microsoft. 

A continuación se muestra un ejemplo de los diferentes intervalos y algunos intervalos de ejemplo que se deben configurar y, en algunos casos, proporcionar a Microsoft. No se agrega el valor para el espacio de direcciones de red virtual de Azure en el primer ejemplo. Sin embargo, se define a partir de los intervalos del primer intervalo de direcciones IP de la subred de máquina virtual de Azure y el intervalo de direcciones IP de la subred de puerta de enlace de la red virtual. 

Puede usar varias subredes de máquina virtual dentro de la red virtual de Azure cuando configura y envía los intervalos de direcciones IP adicionales de las subredes de VM adicionales como parte del espacio de direcciones de la red virtual de Azure.

![Intervalos de direcciones IP necesarios en la implementación mínima de SAP HANA en Azure (Instancias grandes)](./media/hana-overview-connectivity/image4b-ip-addres-ranges-necessary.png)

El gráfico no muestra los intervalos de direcciones IP adicionales que son necesarios para el uso opcional de ExpressRoute Global Reach.

También puede agregar los datos que envía a Microsoft. En ese caso, el espacio de direcciones de la red virtual de Azure solo incluye un espacio. Con los intervalos de direcciones IP del ejemplo anterior, el espacio de direcciones de la red virtual agregado podría tener un aspecto similar a la imagen siguiente:

![Segunda posibilidad de intervalos de direcciones IP necesarios en la implementación mínima de SAP HANA en Azure (Instancias grandes)](./media/hana-overview-connectivity/image5b-ip-addres-ranges-necessary-one-value.png)

En el ejemplo, en lugar de dos intervalos más pequeños que definían el espacio de direcciones de la red virtual de Azure, tenemos un intervalo mayor que abarca 4096 direcciones IP. Una definición de gran tamaño del espacio de direcciones deja algunos intervalos bastante grandes sin usar. Puesto que los valores del espacio de direcciones de la red virtual se usan para la propagación de la ruta BGP, la utilización de los intervalos no usados en instancias locales o en otra parte de la red puede causar problemas de enrutamiento. El gráfico no muestra los intervalos de direcciones IP adicionales que son necesarios para el uso opcional de ExpressRoute Global Reach.

Se recomienda mantener el espacio de direcciones estrechamente alineado con el espacio de direcciones de la subred que realmente se usa. En caso necesario, siempre puede agregar nuevos valores al espacio de direcciones más adelante sin incurrir en tiempo de inactividad en la red virtual.
 
> [!IMPORTANT] 
> Cada intervalo de direcciones IP de ER-P2P, el grupo de direcciones IP del servidor y el espacio de direcciones de la red virtual de Azure **NO** se deben superponer entre sí ni con ningún otro intervalo usado en la red. Cada uno de ellos debe ser discreto. Como muestran los dos gráficos anteriores, tampoco pueden ser una subred de ningún otro intervalo. Si se producen superposiciones entre los intervalos, la red virtual de Azure podría no conectarse al circuito de ExpressRoute.

## <a name="next-steps-after-address-ranges-have-been-defined"></a>Pasos siguientes después de definir los intervalos de direcciones
Después de definir los intervalos de direcciones IP, son necesarios los siguientes pasos:

1. Enviar los intervalos de direcciones IP del espacio de direcciones de la red virtual de Azure, la conectividad de ER-P2P y el intervalo de direcciones del grupo de direcciones IP del servidor, junto con cualquier otro dato que se haya indicado al principio del documento. En este momento, también podría empezar a crear la red virtual y las subredes de la máquina virtual. 
2. Microsoft crea un circuito de ExpressRoute entre la suscripción de Azure y el sello de HANA (instancias grandes).
3. Microsoft crea una red de inquilino en el sello de HANA Instancias grandes.
4. Microsoft configura las redes en la infraestructura de SAP HANA en Azure (instancias grandes) de forma que acepten las direcciones IP del espacio de direcciones de la red virtual de Azure que se comunica con HANA (instancias grandes).
5. Según la SKU concreta de SAP HANA en Azure (instancias grandes) que haya comprado, Microsoft asigna una unidad de proceso en una red de inquilino. También asigna y monta almacenamiento e instala el sistema operativo (SUSE o Red Hat Linux). Las direcciones IP para estas unidades se toman del intervalo de direcciones del grupo de direcciones IP de servidor que envió a Microsoft.

Al final del proceso de implementación, Microsoft le proporciona los siguientes datos:
- Información que necesita para conectar las redes virtuales de Azure al circuito de ExpressRoute que conecta las redes virtuales de Azure con HANA (instancias grandes):
     - Claves de autorización
     - PeerID de ExpressRoute
- Datos para acceder a HANA (instancias grandes) después de establecer el circuito de ExpressRoute y la red virtual de Azure.

También puede buscar la secuencia de conexión de HANA (instancias grandes) en el documento [SAP HANA on Azure (Large Instances) Setup](https://azure.microsoft.com/resources/sap-hana-on-azure-large-instances-setup/) (Configuración de SAP HANA en Azure [instancias grandes]). Muchos de los pasos siguientes se muestran en una implementación de ejemplo en ese documento. 

## <a name="next-steps"></a>Pasos siguientes

- Consulte el artículo sobre la [conexión de una red virtual a ExpressRoute de HANA (instancias grandes)](hana-connect-vnet-express-route.md).
