---
title: 'Actualización del esquema de Análisis de tráfico de Azure: marzo de 2020 | Microsoft Docs'
description: Consultas de ejemplo con nuevos campos en el esquema de Análisis de tráfico.
services: network-watcher
documentationcenter: na
author: vinigam
manager: agummadi
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/06/2020
ms.author: vinigam
ms.openlocfilehash: 31d0de63185c56eafda8c42efbe44d8e7ffcaf1f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87022470"
---
# <a name="sample-queries-with-new-fields-in-the-traffic-analytics-schema-august-2019-schema-update"></a>Consultas de ejemplo con nuevos campos en el esquema de Análisis de tráfico (actualización del esquema de agosto de 2019)

El [esquema de registro del Análisis de tráfico](https://docs.microsoft.com/azure/network-watcher/traffic-analytics-schema) incluye estos campos nuevos: **SrcPublicIPs_s**, **DestPublicIPs_s**, **NSGRule_s**. Los nuevos campos proporcionan información sobre las direcciones IP de origen y de destino y simplifican las consultas.

En los próximos meses, los siguientes campos más antiguos van a quedar en desuso: **VMIP_s**, **Subscription_g**, **Region_s**, **NSGRules_s**, **Subnet_s**, **VM_s**, **NIC_s**, **PublicIPs_s**, **FlowCount_d**.

En los tres ejemplos siguientes se muestra cómo reemplazar los campos anteriores por los nuevos.

## <a name="example-1-vmip_s-subscription_g-region_s-subnet_s-vm_s-nic_s-and-publicips_s-fields"></a>Ejemplo 1: campos VMIP_s, Subscription_g, Region_s, Subnet_s, VM_s, NIC_s y PublicIPs_s

No es necesario inferir los casos de origen y destino del campo **FlowDirection_s** para los flujos AzurePublic y ExternalPublic. También puede resultar inadecuado usar el campo **FlowDirection_s** para una aplicación virtual de red.

```Old Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "1"
| extend isAzureOrExternalPublicFlows = FlowType_s in ("AzurePublic", "ExternalPublic")
| extend SourceAzureVM = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', VM_s, "N/A"), VM1_s),
SourceAzureVMIP = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', VM_s, "N/A"), SrcIP_s),
SourceAzureVMSubscription = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', Subscription_g, "N/A"), Subscription1_g),
SourceAzureRegion = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', Region_s, "N/A"), Region1_s),
SourceAzureSubnet = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', Subnet_s, "N/A"), Subnet1_s),
SourceAzureNIC = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', NIC_s, "N/A"), NIC1_s),
DestAzureVM = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', VM_s, "N/A"), VM2_s),
DestAzureVMIP = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', VM_s, "N/A"), DestIP_s),
DestAzureVMSubscription = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', Subscription_g, "N/A"), Subscription2_g),
DestAzureRegion = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', Region_s, "N/A"), Region2_s),
DestAzureSubnet = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', Subnet_s, "N/A"), Subnet2_s),
DestAzureNIC = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', NIC_s, "N/A"), NIC2_s),
SourcePublicIPsAggregated = iif(isAzureOrExternalPublicFlows and FlowDirection_s == 'I', PublicIPs_s, "N/A"),
DestPublicIPsAggregated = iif(isAzureOrExternalPublicFlows and FlowDirection_s == 'O', PublicIPs_s, "N/A")
```


```New Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "2"
| extend SourceAzureVM = iif(isnotempty(VM1_s), VM1_s, "N/A"),
SourceAzureVMIP = iif(isnotempty(SrcIP_s), SrcIP_s, "N/A"),
SourceAzureVMSubscription = iif(isnotempty(Subscription1_g), Subscription1_g, "N/A"),
SourceAzureRegion = iif(isnotempty(Region1_s), Region1_s, "N/A"),
SourceAzureSubnet = iif(isnotempty(Subnet1_s), Subnet1_s, "N/A"),
SourceAzureNIC = iif(isnotempty(NIC1_s), NIC1_s, "N/A"),
DestAzureVM = iif(isnotempty(VM2_s), VM2_s, "N/A"),
DestAzureVMIP = iif(isnotempty(DestIP_s), DestIP_s, "N/A"),
DestAzureVMSubscription = iif(isnotempty(Subscription2_g), Subscription2_g, "N/A"),
DestAzureRegion = iif(isnotempty(Region2_s), Region2_s, "N/A"),
DestAzureSubnet = iif(isnotempty(Subnet2_s), Subnet2_s, "N/A"),
DestAzureNIC = iif(isnotempty(NIC2_s), NIC2_s, "N/A"),
SourcePublicIPsAggregated = iif(isnotempty(SrcPublicIPs_s), SrcPublicIPs_s, "N/A"),
DestPublicIPsAggregated = iif(isnotempty(DestPublicIPs_s), DestPublicIPs_s, "N/A")
```

## <a name="example-2-nsgrules_s-field"></a>Ejemplo 2: Campo NSGRules_s

El campo anterior usaba el formato:

`<Index value 0)>|<NSG_ RuleName>|<Flow Direction>|<Flow Status>|<FlowCount ProcessedByRule>`

Ya no se agregan datos a través de un grupo de seguridad de red (NSG). En el esquema actualizado, **NSGList_s** contiene solo un NSG. Además, **NSGRules** solo contiene una regla. Hemos quitado el formato complicado aquí y en otros campos, tal y como se muestra en el ejemplo.

```Old Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "1"
| extend NSGRuleComponents = split(NSGRules_s, "|")
| extend NSGName = NSGList_s // remains same
| extend NSGRuleName = NSGRuleComponents[1],
         FlowDirection = NSGRuleComponents[2],
         FlowStatus = NSGRuleComponents[3],
         FlowCountProcessedByRule = NSGRuleComponents[4]
| project NSGName, NSGRuleName, FlowDirection, FlowStatus, FlowCountProcessedByRule
```

```New Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "2"
| extend NSGRuleComponents = split(NSGRules_s, "|")
| project NSGName = NSGList_s,
NSGRuleName = NSGRule_s ,
FlowDirection = FlowDirection_s,
FlowStatus = FlowStatus_s,
FlowCountProcessedByRule = AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d
```

## <a name="example-3-flowcount_d-field"></a>Ejemplo 3: Campo FlowCount_d

Dado que no agrupamos los datos en el NSG, el campo **FlowCount_d** es, simplemente:

**AllowedInFlows_d** + **DeniedInFlows_d** + **AllowedOutFlows_d** + **DeniedOutFlows_d**

Solo uno de los cuatro campos será distinto de cero. Los otros tres campos tendrán un valor de cero. Estos campos se completan para indicar el estado y el recuento en la NIC donde se ha capturado el flujo.

Para ilustrar estas condiciones:

- Si se ha permitido el flujo, se rellena uno de los campos con el prefijo "Allowed".
- Si se ha denegado el flujo, se rellena uno de los campos con el prefijo "Denied".
- Si el flujo era de entrada, se rellena uno de los campos con el sufijo "InFlows_d".
- Si el flujo era de salida, se rellena uno de los campos con el sufijo "OutFlows_d".

En función de las condiciones, sabemos cuál de los cuatro campos se rellenará.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener respuestas a las preguntas más frecuentes, consulte [Preguntas más frecuentes de Análisis de tráfico](traffic-analytics-faq.md).
- Para ver detalles acerca de la funcionalidad, consulte la [documentación de Análisis de tráfico](traffic-analytics.md).
