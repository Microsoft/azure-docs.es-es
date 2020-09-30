---
title: Direcciones IP de administración de HDInsight de Azure
description: Obtenga información sobre las direcciones IP a las que tiene que permitir el tráfico de entrada, con el fin de configurar correctamente los grupos de seguridad de red y las rutas definidas por el usuario para la red virtual con Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 08/11/2020
ms.openlocfilehash: 4f7db88da646c9787c70d04ff7e3478a27a09275
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401646"
---
# <a name="hdinsight-management-ip-addresses"></a>Direcciones IP de administración de HDInsight

En este artículo se enumeran las direcciones IP que usan los servicios de mantenimiento y administración de Azure HDInsight. Si usa grupos de seguridad de red o rutas definidas por el usuario, es posible que tenga que agregar algunas de estas direcciones IP a la lista de permitidos para el tráfico de red entrante.

## <a name="introduction"></a>Introducción
 
> [!Important]
> En la mayoría de los casos, ahora puede usar las [etiquetas de servicio](hdinsight-service-tags.md) para los grupos de seguridad de red, en lugar de agregar manualmente las direcciones IP. Las direcciones IP no se publicarán para las nuevas regiones de Azure y solo tendrán etiquetas de servicio publicadas. Las direcciones IP estáticas para las direcciones IP de administración quedarán en desuso.

Si usa grupos de seguridad de red (NSG) o rutas definidas por el usuario (UDR) para controlar el tráfico entrante a su clúster de HDInsight, tiene que asegurarse de que el clúster pueda comunicarse con los servicios críticos de mantenimiento y administración de Azure.  Algunas de las direcciones IP de esos servicios son específicas de una región y otra se aplican a todas las regiones de Azure. También es posible que deba permitir el tráfico desde el servicio Azure DNS si no usa DNS personalizado.

Si necesita direcciones IP para una región que no aparece aquí, puede usar la [Service Tag Discovery API](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api-public-preview) para buscar las direcciones IP de su región. Si no puede usar la API, descargue el [archivo JSON de la etiqueta de servicio](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) y busque la región deseada.

En las secciones siguientes se describen las direcciones IP específicas que se deben permitir.

## <a name="azure-dns-service"></a>Servicio de Azure DNS

Si usa el servicio DNS proporcionado por Azure, debe permitir el acceso desde __168.63.129.16__ en el puerto 53. Para más información, vea el documento [Resolución de nombres para las máquinas virtuales e instancias de rol](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md). Si usa DNS personalizado, omita este paso.

## <a name="health-and-management-services-all-regions"></a>Servicios de mantenimiento y administración: Todas las regiones

Permita el tráfico desde las siguientes direcciones IP para los servicios de mantenimiento y administración de Azure HDInsight, esto es aplicable a todas las regiones de Azure:

| Dirección IP de origen | Destination  | Dirección |
| ---- | ----- | ----- |
| 168.61.49.99 | \*:443 | Entrada |
| 23.99.5.239 | \*:443 | Entrada |
| 168.61.48.131 | \*:443 | Entrada |
| 138.91.141.162 | \*:443 | Entrada |

## <a name="health-and-management-services-specific-regions"></a>Servicios de mantenimiento y administración: Regiones específicas

Permita el tráfico desde las direcciones IP enumeradas en los servicios de mantenimiento y administración de Azure HDInsight en la región específica en la que se encuentran los recursos:

> [!IMPORTANT]  
> Si la región de Azure que usa no aparece en la lista, use la característica [etiqueta de servicio](hdinsight-service-tags.md) para los grupos de seguridad de red.

| Country | Region | Direcciones IP de origen permitidas | Destino permitido | Dirección |
| ---- | ---- | ---- | ---- | ----- |
| Asia | Este de Asia | 23.102.235.122</br>52.175.38.134 | \*:443 | Entrada |
| &nbsp; | Sudeste de Asia | 13.76.245.160</br>13.76.136.249 | \*:443 | Entrada |
| Australia | Este de Australia | 104.210.84.115</br>13.75.152.195 | \*:443 | Entrada |
| &nbsp; | Sudeste de Australia | 13.77.2.56</br>13.77.2.94 | \*:443 | Entrada |
| Brasil | Sur de Brasil | 191.235.84.104</br>191.235.87.113 | \*:443 | Entrada |
| Canadá | Este de Canadá | 52.229.127.96</br>52.229.123.172 | \*:443 | Entrada |
| &nbsp; | Centro de Canadá | 52.228.37.66</br>52.228.45.222 |\*: 443 | Entrada |
| China | Norte de China | 42.159.96.170</br>139.217.2.219</br></br>42.159.198.178</br>42.159.234.157 | \*:443 | Entrada |
| &nbsp; | Este de China | 42.159.198.178</br>42.159.234.157</br></br>42.159.96.170</br>139.217.2.219 | \*:443 | Entrada |
| &nbsp; | Norte de China 2 | 40.73.37.141</br>40.73.38.172 | \*:443 | Entrada |
| &nbsp; | Este de China 2 | 139.217.227.106</br>139.217.228.187 | \*:443 | Entrada |
| Europa | Norte de Europa | 52.164.210.96</br>13.74.153.132 | \*:443 | Entrada |
| &nbsp; | Oeste de Europa| 52.166.243.90</br>52.174.36.244 | \*:443 | Entrada |
| Francia | Centro de Francia| 20.188.39.64</br>40.89.157.135 | \*:443 | Entrada |
| Alemania | Centro de Alemania | 51.4.146.68</br>51.4.146.80 | \*:443 | Entrada |
| &nbsp; | Nordeste de Alemania | 51.5.150.132</br>51.5.144.101 | \*:443 | Entrada |
| India | Centro de la India | 52.172.153.209</br>52.172.152.49 | \*:443 | Entrada |
| &nbsp; | Sur de la India | 104.211.223.67<br/>104.211.216.210 | \*:443 | Entrada |
| Japón | Japón Oriental | 13.78.125.90</br>13.78.89.60 | \*:443 | Entrada |
| &nbsp; | Japón Occidental | 40.74.125.69</br>138.91.29.150 | \*:443 | Entrada |
| Corea | Centro de Corea del Sur | 52.231.39.142</br>52.231.36.209 | \*:443 | Entrada |
| &nbsp; | Corea del Sur | 52.231.203.16</br>52.231.205.214 | \*:443 | Entrada
| Reino Unido | Oeste de Reino Unido | 51.141.13.110</br>51.141.7.20 | \*:443 | Entrada |
| &nbsp; | Sur de Reino Unido 2 | 51.140.47.39</br>51.140.52.16 | \*:443 | Entrada |
| Estados Unidos | Centro de EE. UU. | 13.89.171.122</br>13.89.171.124 | \*:443 | Entrada |
| &nbsp; | Este de EE. UU. | 13.82.225.233</br>40.71.175.99 | \*:443 | Entrada |
| &nbsp; | Centro-Norte de EE. UU | 157.56.8.38</br>157.55.213.99 | \*:443 | Entrada |
| &nbsp; | Centro-Oeste de EE. UU. | 52.161.23.15</br>52.161.10.167 | \*:443 | Entrada |
| &nbsp; | Oeste de EE. UU. | 13.64.254.98</br>23.101.196.19 | \*:443 | Entrada |
| &nbsp; | Oeste de EE. UU. 2 | 52.175.211.210</br>52.175.222.222 | \*:443 | Entrada |
| &nbsp; | Norte de Emiratos Árabes Unidos | 65.52.252.96</br>65.52.252.97 | \*:443 | Entrada |

Para más información sobre las direcciones IP que se van a usar para Azure Government, vea el documento [Azure Government Intelligence + Analytics](https://docs.microsoft.com/azure/azure-government/documentation-government-services-intelligenceandanalytics) (Inteligencia y análisis de Azure Government).

Para más información, consulte [Control del tráfico de red](./control-network-traffic.md).

Si está usando rutas definidas por el usuario (UDR), debe especificar una ruta y permitir el tráfico saliente desde la red virtual a las IP anteriores, con el próximo salto configurado en "Internet".

## <a name="next-steps"></a>Pasos siguientes

* [Creación de redes virtuales para clústeres de Azure HDInsight](hdinsight-create-virtual-network.md)
* [Etiquetas de servicio del grupo de seguridad de red (NSG) para Azure HDInsight](hdinsight-service-tags.md)
