---
title: Compatibilidad de zonas de disponibilidad con instancias de App Service Environment
description: Aprenda a implementar instancias de App Service Environment para que las aplicaciones tengan redundancia de zona.
author: ccompy
ms.assetid: 24e3e7eb-c160-49ff-8d46-e947818ef186
ms.topic: article
ms.date: 07/15/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 1e88aac4209f7960b2589cf43f59ead4bd129134
ms.sourcegitcommit: 80b9c8ef63cc75b226db5513ad81368b8ab28a28
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90605080"
---
# <a name="availability-zone-support-for-app-service-environments"></a>Compatibilidad de zonas de disponibilidad con instancias de App Service Environment

Las instancias de App Service Environment (ASE) se pueden implementar en Availability Zones (AZ).  Los clientes pueden implementar instancias de ASE de equilibrador de carga interno (ILB) en una instancia de AZ específica dentro de una región de Azure. Si ancla la instancia de ASE de ILB en una AZ específica, los recursos que usa una instancia de ASE de ILB se anclarán a la AZ especificada o se implementarán con redundancia de zona.  

Una instancia de ASE de ILB que se implementa explícitamente en una AZ se considera un recurso de zona porque la instancia de ASE de ILB está anclada a una zona específica. Las dependencias siguientes de ASE de ILB se anclarán a la zona especificada:

- La dirección IP del equilibrador de carga interno de la instancia de ASE.
- Los recursos de proceso que la instancia de ASE usa para administrar y ejecutar aplicaciones web.

El almacenamiento de archivos remoto para las aplicaciones web implementadas en una instancia de ASE de ILB de zona usa almacenamiento con redundancia de zona (ZRS).

A menos que se sigan los pasos descritos en este artículo, las instancias de ASE de ILB no se implementan automáticamente de forma zonal. No se puede anclar una instancia de ASE externo con una IP pública a una zona de disponibilidad específica. 

Las instancias de ASE de ILB de zona se pueden crear en cualquiera de estas regiones:

- Este de Australia
- Sur de Brasil
- Centro de Canadá
- Centro de EE. UU.
- Este de EE. UU.
- Este de EE. UU. 2
- Este de EE. UU. 2 (EUAP)
- Centro de Francia 
- Centro-oeste de Alemania
- Japón Oriental
- Norte de Europa
- Oeste de Europa
- Sudeste de Asia
- Sur de Reino Unido
- Oeste de EE. UU. 2

Las aplicaciones implementadas en una instancia de ASE de ILB de zona se seguirán ejecutando y atenderán el tráfico en esa instancia de ASE incluso si otras zonas de la misma región sufren una interrupción.  Es posible que los comportamientos no en tiempo de ejecución, incluidos el escalado del plan de servicio de aplicación, la creación de aplicaciones, la configuración de aplicaciones y la publicación de aplicaciones, puedan verse afectados por una interrupción en otras zonas de disponibilidad. La implementación anclada por zona de una instancia de ASE de ILB de zona solo garantiza un tiempo de actividad continuado para las aplicaciones ya implementadas.

## <a name="how-to-deploy-an-app-service-environment-in-an-availability-zone"></a>Implementación de App Service Environment en una zona de disponibilidad ##

Las instancias de ASE de ILB de zona se deben crear con plantillas de Resource Manager. Una vez que se crea una instancia de ASE de ILB de zona a través de una plantilla de Resource Manager, es posible verla e interactuar con ella a través de Azure Portal y la CLI.  Solo se necesita una plantilla de Resource Manager para la creación inicial de una instancia de ASE de ILB de zona.

El único cambio necesario en una plantilla de Resource Manager para especificar una instancia de ASE de ILB de zona es la nueva propiedad ***zones***. La propiedad ***zones*** se debe establecer en un valor de "1", "2" o "3", según la zona de disponibilidad lógica a la que debe estar anclada la instancia de ASE de ILB.

El fragmento de la plantilla de Resource Manager de ejemplo que aparece a continuación muestra la nueva propiedad ***zones*** que especifica que la instancia de ASE de ILB se debe anclar a la zona 2.

```
   "resources": [
      {
         "type": "Microsoft.Web/hostingEnvironments",
         "kind": "ASEV2",
         "name": "yourASENameHere",
         "apiVersion": "2015-08-01",
         "location": "your location here",
         "zones": [
            "2"
         ],
         "properties": {
         "name": "yourASENameHere",
         "location": "your location here",
         "ipSslAddressCount": 0,
         "internalLoadBalancingMode": "3",
         "dnsSuffix": "contoso-internal.com",
         "virtualNetwork": {
             "Id": "/subscriptions/your-subscription-id-here/resourceGroups/your-resource-group-here/providers/Microsoft.Network/virtualNetworks/your-vnet-name-here",
             "Subnet": "yourSubnetNameHere"
          }
         }
      }
    ]
```

Para hacer que la zona de aplicaciones sea redundante, debe implementar dos instancias de ASE de ILB de zona. Las dos instancias de ASE de ILB de zona deben estar en zonas de disponibilidad independientes. Luego debe implementar las aplicaciones en cada una de las instancias de ASE de ILB. Una vez creadas las aplicaciones, debe configurar una solución de equilibrio de carga. La solución recomendada es implementar una instancia ascendente de [Application Gateway con redundancia de zona](../../application-gateway/application-gateway-autoscaling-zone-redundant.md) de las instancias de ASE de ILB de zona. 

## <a name="in-region-data-residency"></a>Residencia de datos en la región ##

Las instancias de ASE de ILB implementadas en una zona de disponibilidad solo almacenarán datos de cliente dentro de la región donde se implementó la instancia de ASE de ILB de zona. Tanto el contenido del archivo del sitio web como la configuración proporcionada por el cliente y los secretos almacenados en App Service permanecen dentro de la región donde está implementada la instancia de ASE de ILB de zona.

Los clientes garantizan la residencia de datos de una sola región si siguen los pasos descritos anteriormente en la sección "Implementación de App Service Environment en una zona de disponibilidad". Al configurar una instancia de App Service Environment de acuerdo con estos pasos, una instancia de App Service Environment implementada en una zona de disponibilidad cumple con los requisitos de residencia de datos en la región, incluidos los especificados en el [Centro de confianza de Azure](https://azuredatacentermap.azurewebsites.net/).

Para validar que hay una instancia de App Service Environment configurada correctamente para almacenar datos en una sola región, los clientes pueden seguir estos pasos: 

1. Con el [Explorador de recursos](https://resources.azure.com), vaya al recurso de Azure Resource Manager correspondiente a la instancia de App Service Environment.  Las instancias de ASE aparecen en *providers/Microsoft.Web/hostingEnvironments*.
2. Si hay una propiedad *zones* en la vista de la sintaxis JSON de ARM y contiene una matriz JZON de un solo valor con un valor de "1", "2" o "3", la instancia de ASE está implementada en la zona y los datos de cliente permanecen en la misma región.
2. Si no existe una propiedad *zones* o la propiedad no tiene un valor de zona válido según lo especificado anteriormente, la instancia de ASE no está implementada en la zona y los datos de cliente no se almacenan de manera exclusiva en la misma región.
