---
title: Etiquetas de servicio del grupo de seguridad de red (NSG) para Azure HDInsight
description: Use etiquetas de servicio de HDInsight para permitir el tráfico entrante al clúster desde los nodos de servicios de mantenimiento y administración, sin agregar direcciones IP a los grupos de seguridad de red.
author: hrasheed-msft
ms.author: hrasheed
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 08/11/2020
ms.openlocfilehash: 112f915f533627ccdc0ac6efe38caacc80b254bc
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399963"
---
# <a name="nsg-service-tags-for-azure-hdinsight"></a>Etiquetas de servicio de grupos de seguridad de red para Azure HDInsight

Las etiquetas de servicio de Azure HDInsight de los grupos de seguridad de red (NSG) son grupos de direcciones IP para los servicios de mantenimiento y administración. Estos grupos ayudan a minimizar la complejidad de la creación de reglas de seguridad. Las [etiquetas de servicio](../virtual-network/security-overview.md#service-tags) permiten el tráfico de entrada desde direcciones IP específicas sin escribir cada una de las [direcciones IP de administración](hdinsight-management-ip-addresses.md) en los grupos de seguridad de red.

El servicio HDInsight administra estas etiquetas de servicio. No puede crear su propia etiqueta de servicio ni modificar una etiqueta existente. Microsoft administra los prefijos de direcciones que coinciden con la etiqueta de servicio y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

Si quiere usar una región determinada y la etiqueta de servicio aún no está documentada en esta página, puede usar la [Service Tag Discovery API](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api-public-preview) para buscar la etiqueta de servicio. También puede descargar el [archivo JSON de la etiqueta de servicio](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) y buscar la región deseada.

## <a name="get-started-with-service-tags"></a>Introducción a las etiquetas de servicio

Tiene dos opciones para usar etiquetas de servicio en los grupos de seguridad de red:

- **Usar una sola etiqueta de servicio de HDInsight global**: esta opción abre la red virtual a todas las direcciones IP que usa el servicio HDInsight para supervisar los clústeres en todas las regiones. Aunque es el método más sencillo, podría no ser adecuado si tiene requisitos de seguridad restrictivos.

- **Usar varias etiquetas de servicio regionales**: esta opción abre la red virtual solo a las direcciones IP que usa HDInsight en esa región específica. No obstante, si usa varias regiones, tendrá que agregar varias etiquetas de servicio a la red virtual.

## <a name="use-a-single-global-hdinsight-service-tag"></a>Usar una sola etiqueta de servicio de HDInsight global

La manera más fácil de empezar a usar etiquetas de servicio con el clúster de HDInsight es agregar la etiqueta global `HDInsight` a una regla de grupo de seguridad de red.

1. Desde [Azure Portal](https://portal.azure.com/), seleccione su grupo de seguridad de red.

1. En **Configuración**, seleccione **Reglas de seguridad de entrada** y, a continuación, seleccione **+ Agregar**.

1. En la lista desplegable **Origen**, seleccione **Etiqueta de servicio**.

1. En la lista desplegable **Etiqueta de servicio de origen**, seleccione **HDInsight**.

    ![Adición de una etiqueta de servicio desde Azure Portal](./media/hdinsight-service-tags/azure-portal-add-service-tag.png)

Esta etiqueta contiene las direcciones IP de los servicios de mantenimiento y administración de todas las regiones donde está disponible HDInsight. La etiqueta garantizará que el clúster pueda comunicarse con los servicios de mantenimiento y administración necesarios, independientemente de dónde se cree.

## <a name="use-regional-hdinsight-service-tags"></a>Uso de etiquetas de servicio de HDInsight regionales

Si la opción de etiqueta global no funciona porque se necesitan permisos más restrictivos, solo podrá permitir las etiquetas de servicio aplicables a su región. Puede haber varias etiquetas de servicio aplicables, en función de la región en la que se cree el clúster.

Para averiguar qué etiquetas de servicio agregar a su región, lea las siguientes secciones del artículo.

### <a name="use-a-single-regional-service-tag"></a>Uso de una sola etiqueta de servicio regional

Si el clúster se encuentra en una región incluida en esta tabla, solo tiene que agregar una etiqueta de servicio regional única al grupo de seguridad de red.

| Country | Region | Etiqueta de servicio |
| ---- | ---- | ---- |
| Australia | Este de Australia | HDInsight.AustraliaEast |
| &nbsp; | Sudeste de Australia | HDInsight.AustraliaSoutheast |
| &nbsp; | Centro de Australia | HDInsight.AustraliaCentral |
| China | Este de China 2 | HDInsight.ChinaEast2 |
| &nbsp; | Norte de China 2 | HDInsight.ChinaNorth2 |
| Estados Unidos | Centro-Norte de EE. UU | HDInsight.NorthCentralUS |
| &nbsp; | Oeste de EE. UU. 2 | HDInsight.WestUS2 |
| &nbsp; | Centro-Oeste de EE. UU. | HDInsight.WestCentralUS |
| Canadá | Este de Canadá | HDInsight.CanadaEast |
| Brasil | Sur de Brasil | HDInsight.BrazilSouth |
| Corea | Centro de Corea del Sur | HDInsight.KoreaCentral |
| &nbsp; | Corea del Sur | HDInsight.KoreaSouth |
| India | Centro de la India | HDInsight.CentralIndia |
| &nbsp; | Sur de la India | HDInsight.SouthIndia |
| Japón | Japón Occidental | HDInsight.JapanWest |
| Francia | Centro de Francia| HDInsight.FranceCentral |
| Reino Unido | Sur de Reino Unido 2 | HDInsight.UKSouth |
| Azure Government | Departamento de Defensa de centro de EE. UU. | HDInsight.USDoDCentral |
| &nbsp; | USGov: Texas | HDInsight.USGovTexas |
| &nbsp; | Departamento de Defensa de este de EE. UU. | HDInsight.USDoDEast |
| &nbsp; | USGov: Arizona | HDInsight.USGovArizona |

### <a name="use-multiple-regional-service-tags"></a>Uso de varias etiquetas de servicio regional

Si la región en la que se creó el clúster no aparece en la tabla anterior, debe permitir varias etiquetas de servicio regional. La necesidad de usar más de una es debido a las diferencias en la organización de los proveedores de recursos para las distintas regiones.

Las regiones restantes se dividen en grupos en función de las etiquetas de servicio regionales que usan.

#### <a name="group-1"></a>Grupo 1

Si el clúster se crea en una de las regiones de la tabla siguiente, permita las etiquetas de servicio `HDInsight.WestUS` y `HDInsight.EastUS`. Además, se muestra la etiqueta de servicio regional. Las regiones de esta sección requieren tres etiquetas de servicio.

Por ejemplo, si el clúster se crea en la región `East US 2`, tendrá que agregar las siguientes etiquetas de servicio al grupo de seguridad de red:

- `HDInsight.EastUS2`
- `HDInsight.WestUS`
- `HDInsight.EastUS`

| Country | Region | Etiqueta de servicio |
| ---- | ---- | ---- |
| Estados Unidos | Este de EE. UU. 2 | HDInsight.EastUS2 |
| &nbsp; | Centro de EE. UU. | HDInsight.CentralUS |
| &nbsp; | Centro-Norte de EE. UU. | HDInsight. NorthCentralUS |
| &nbsp; | Centro-sur de EE. UU. | HDInsight.SouthCentralUS |
| &nbsp; | Este de EE. UU. | HDInsight.EastUS |
| &nbsp; | Oeste de EE. UU. | HDInsight.WestUS |
| Japón | Japón Oriental | HDInsight.JapanEast |
| Europa | Norte de Europa | HDInsight.NorthEurope |
| &nbsp; | Oeste de Europa| HDInsight.WestEurope |
| Asia | Este de Asia | HDInsight.EastAsia |
| &nbsp; | Sudeste de Asia | HDInsight.SoutheastAsia |
| Australia | Este de Australia | HDInsight.AustraliaEast |

#### <a name="group-2"></a>Grupo 2

Los clústeres de las regiones de *Norte de China* y *Este de China* deben permitir dos etiquetas de servicio: `HDInsight.ChinaNorth` y `HDInsight.ChinaEast`.

#### <a name="group-3"></a>Grupo 3

Los clústeres de las regiones de *US Gov Iowa* y *US Gov Virginia* deben permitir dos etiquetas de servicio: `HDInsight.USGovIowa` y `HDInsight.USGovVirginia`.

#### <a name="group-4"></a>Grupo 4

Los clústeres de las regiones de *Centro de Alemania* y *Nordeste de Alemania* deben permitir dos etiquetas de servicio: `HDInsight.GermanyCentral` y `HDInsight.GermanyNortheast`.

## <a name="next-steps"></a>Pasos siguientes

- [Grupos de seguridad de red: etiquetas de servicio](../virtual-network/security-overview.md#security-rules)
- [Creación de redes virtuales para clústeres de Azure HDInsight](hdinsight-create-virtual-network.md)
