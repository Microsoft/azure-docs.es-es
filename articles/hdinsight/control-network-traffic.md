---
title: Control del tráfico de red en Azure HDInsight
description: Aprenda técnicas para controlar el tráfico entrante y saliente en los clústeres de Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 09/02/2020
ms.openlocfilehash: a33bc5816ded7cdca75737b02add0a6ca8821700
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400201"
---
# <a name="control-network-traffic-in-azure-hdinsight"></a>Control del tráfico de red en Azure HDInsight

El tráfico de red en instancias de Azure Virtual Network se puede controlar mediante los métodos siguientes:

* Los **Grupos de seguridad de red** (NSG) permiten filtrar el tráfico de entrada y salida de la red. Para más información, vea el documento [Filtrar el tráfico de red con grupos de seguridad de red](../virtual-network/security-overview.md).

* Las **Aplicaciones virtuales de red** (NVA) solo se pueden utilizar con el tráfico saliente. Las aplicaciones virtuales de red replican la funcionalidad de dispositivos como enrutadores y firewalls. Para más información, vea el documento [Dispositivos de red](https://azure.microsoft.com/solutions/network-appliances).

Como un servicio administrado, HDInsight requiere acceso sin restricciones a los servicios de administración y mantenimiento de HDInsight para el tráfico entrante y saliente de la red virtual. Al usar grupos de seguridad de red, debe asegurarse de que estos servicios pueden seguir comunicándose con el clúster de HDInsight.

![Diagrama de entidades de HDInsight creadas en una red virtual personalizada de Azure](./media/control-network-traffic/hdinsight-vnet-diagram.png)

## <a name="hdinsight-with-network-security-groups"></a>HDInsight con grupos de seguridad de red

Si planifica usar **grupos de seguridad de red** para controlar el tráfico de red, realice las siguientes acciones antes de instalar HDInsight:

1. Identificar la región de Azure que va a usar para HDInsight.

2. Identifique las etiquetas de servicio que requiere HDInsight para su región. Existen varias maneras de obtener estas etiquetas de servicio:
    1. Consulte la lista de etiquetas de servicio publicadas en [Etiquetas de servicio del grupo de seguridad de red (NSG) para Azure HDInsight](hdinsight-service-tags.md). 
    2. Si su región no está presente en la lista, use [Service Tag Discovery API](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api-public-preview) para buscar una etiqueta de servicio para la región.
    3. Si no puede usar la API, descargue el [archivo JSON de la etiqueta de servicio](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) y busque la región deseada.


3. Cree o modifique los grupos de seguridad de red de la subred en la que tiene previsto instalar HDInsight.

    * __Grupos de seguridad de red__: permita tráfico de __entrada__ en el puerto __443__ desde las direcciones IP. Esto garantizará que los servicios de administración de HDInsight puedan comunicarse con el clúster desde fuera de la red virtual. En el caso de los clústeres habilitados para el __proxy REST de Kafka__, permita también el tráfico __entrante__ en el puerto __9400__. Esto garantizará que el proxy REST de Kafka sea accesible.

Para más información sobre los grupos de seguridad de red, consulte [Introducción a los grupos de seguridad de red](../virtual-network/security-overview.md).

## <a name="controlling-outbound-traffic-from-hdinsight-clusters"></a>Control del tráfico saliente desde clústeres de HDInsight

Para más información sobre cómo controlar el tráfico saliente de los clústeres de HDInsight, consulte [Configuración de la restricción del tráfico de red saliente para clústeres de Azure HDInsight](hdinsight-restrict-outbound-traffic.md).

### <a name="forced-tunneling-to-on-premises"></a>Tunelización forzada a una ubicación local

La tunelización forzada es una configuración de enrutamiento definida por el usuario por la que todo el tráfico de una subred se fuerza a una red o ubicación específica, como la red local o el firewall. La tunelización forzada de toda transferencia de datos de vuelta al entorno local _no_ se recomienda debido a los grandes volúmenes de la transferencia de datos y a un posible impacto en el rendimiento.

Los clientes interesados en configurar la tunelización forzada deben usar [metastores personalizadas](./hdinsight-use-external-metadata-stores.md) y configurar la conectividad adecuada desde la subred del clúster o la red local a estas metastores personalizadas.

Para obtener un ejemplo de la configuración de UDR con Azure Firewall, vea [Configuración de la restricción del tráfico de red saliente para clústeres de Azure HDInsight](hdinsight-restrict-outbound-traffic.md).

## <a name="required-ports"></a>Puertos necesarios

Si planifica usar un **firewall** para obtener acceso al clúster fuera de determinados puertos, debe permitir el tráfico en esos puertos, ya que son necesarios para su escenario. De manera predeterminada, no se necesita una lista blanca especial de puertos, siempre que el tráfico de administración de Azure que se explica en la sección anterior tenga permitido llegar al clúster en el puerto 443.

Para una lista de puertos para servicios específicos, consulte el documento [Puertos utilizados por los servicios Apache Hadoop en HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Para más información sobre las reglas de firewall para aplicaciones virtuales, vea el documento [Escenario de aplicación virtual](../virtual-network/virtual-network-scenario-udr-gw-nva.md).

## <a name="next-steps"></a>Pasos siguientes

* Para ver ejemplos de código y ejemplos de creación de redes virtuales de Azure, consulte [Creación de redes virtuales para clústeres de Azure HDInsight](hdinsight-create-virtual-network.md).
* Para obtener un ejemplo integral de la configuración de HDInsight para conectarse a una red local, vea [Conexión de HDInsight a la red local](./connect-on-premises-network.md).
* Para más información sobre las redes virtuales de Azure, vea la [información general sobre Azure Virtual Network](../virtual-network/virtual-networks-overview.md).
* Para más información sobre los grupos de seguridad de red, vea [Grupos de seguridad de red](../virtual-network/security-overview.md).
* Para obtener más información sobre las rutas definidas por el usuario, consulte [Rutas definidas por el usuario y reenvío de IP](../virtual-network/virtual-networks-udr-overview.md).
* Para más información sobre las redes virtuales, consulte [Planeación de redes virtuales para HDInsight](./hdinsight-plan-virtual-network-deployment.md).
