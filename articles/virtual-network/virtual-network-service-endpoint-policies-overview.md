---
title: Directivas de punto de conexión de servicio de red virtual de Azure | Microsoft Docs
description: Aprenda a filtrar el tráfico de la red virtual para los recursos de servicio de Azure con las directivas de punto de conexión de servicio.
services: virtual-network
documentationcenter: na
author: RDhillon
ms.service: virtual-network
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/21/2020
ms.author: rdhillon
ms.openlocfilehash: 3c0b0f9f39620996245614b53c7ec274ec965d5b
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87921205"
---
# <a name="virtual-network-service-endpoint-policies-for-azure-storage"></a>Directivas de punto de conexión de servicio de red virtual para Azure Storage

Las directivas de punto de conexión de servicio de red virtual (VNet) permiten filtrar el tráfico de red virtual de salida a cuentas de Azure Storage a través de un punto de conexión de servicio y habilitan la filtración de datos únicamente a cuentas específicas de Azure Storage. Las directivas de punto de conexión ofrecen un control de acceso pormenorizado para el tráfico de red virtual a Azure Storage al conectarse a través de un punto de conexión de servicio.

![Protección del tráfico de salida de red virtual a cuentas de Azure Storage](./media/virtual-network-service-endpoint-policies-overview/vnet-service-endpoint-policies-overview.png)

Esta característica está disponible con carácter general para __Azure Storage__ en __todas las regiones del mundo de Azure__.

## <a name="key-benefits"></a>Ventajas principales

Las directivas de puntos de conexión de servicio de red virtual proporcionan las ventajas siguientes:

- __Seguridad mejorada para el tráfico de red virtual a Azure Storage__

  Las [etiquetas de servicio de Azure para grupos de seguridad de red](https://aka.ms/servicetags) permiten restringir el tráfico de salida de red virtual a regiones concretas de Azure Storage. Pero esto permite el tráfico a cualquier cuenta dentro de la región de Azure Storage seleccionada.
  
  Las directivas de punto de conexión permiten especificar las cuentas de Azure Storage a las que se permite el acceso de salida de red virtual y restringir el acceso a todas las demás cuentas de almacenamiento. Esto proporciona un control de seguridad mucho más pormenorizado para proteger el filtrado de datos de la red virtual.

- __Directivas escalables y altamente disponibles para filtrar el tráfico del servicio de Azure__

   Las directivas de punto de conexión proporcionan una solución horizontal escalable y de alta disponibilidad para filtrar el tráfico de servicio de Azure desde las redes virtuales, a través de los puntos de extremo de servicio. No se requiere ninguna sobrecarga adicional para mantener los dispositivos de red central para este tráfico en las redes virtuales.

## <a name="json-object-for-service-endpoint-policies"></a>Objeto JSON para directivas de punto de conexión de servicio
Vamos a echar un vistazo rápido al objeto de directiva de punto de conexión de servicio.

```json
"serviceEndpointPolicyDefinitions": [
    {
            "description": null,
            "name": "MySEP-Definition",
            "resourceGroup": "MySEPDeployment",
            "service": "Microsoft.Storage",
            "serviceResources": [ 
                    "/subscriptions/subscriptionID/resourceGroups/MySEPDeployment/providers/Microsoft.Storage/storageAccounts/mystgacc"
            ],
            "type": "Microsoft.Network/serviceEndpointPolicies/serviceEndpointPolicyDefinitions"
    }
]
```

## <a name="configuration"></a>Configuración

-   Puede configurar las directivas de punto de conexión de modo que se restrinja el tráfico de red virtual a cuentas específicas de Azure Storage.
-   La directiva de puntos de conexión se configura en una subred de una red virtual. Los puntos de conexión de servicio de Azure Storage deben estar habilitados en la subred para aplicar la directiva.
-   La directiva de punto de conexión permite agregar cuentas concretas de Azure Storage a una lista de permitidos mediante el formato resourceID. Puede restringir el acceso a
    - todas las cuentas de almacenamiento de una suscripción<br>
      `E.g. /subscriptions/subscriptionId`

    - todas las cuentas de almacenamiento de un grupo de recursos<br>
      `E.g. subscriptions/subscriptionId/resourceGroups/resourceGroupName`
     
    - una cuenta de almacenamiento individual al indicar el resourceId correspondiente de Azure Resource Manager. Esto trata el tráfico a blobs, tablas, colas, archivos y a Azure Data Lake Storage Gen2. <br>
    `E.g. /subscriptions/subscriptionId/resourceGroups/resourceGroupName/providers/Microsoft.Storage/storageAccounts/storageAccountName`
-   De forma predeterminada, si no hay directivas asociadas a una subred con puntos de conexión, puede acceder a todas las cuentas de almacenamiento del servicio. Una vez configurada una directiva en esa subred, solo se puede acceder a los recursos especificados en la directiva desde las instancias de proceso de esa subred. Se deniega el acceso a todas las demás cuentas de almacenamiento.
-   Al aplicar directivas de punto de conexión de servicio en una subred, el *ámbito del punto de conexión de servicio* de Azure Storage se actualiza de regional a **global**. Esto significa que todo el tráfico a Azure Storage se protege a través del punto de conexión de servicio a partir de entonces. Las directivas de punto de conexión de servicio también se pueden aplicar globalmente, por lo que se deniega el acceso a cualquier cuenta de almacenamiento que no se haya permitido de forma explícita. 
-   Puede aplicar varias directivas a una subred. Cuando hay varias directivas asociadas a la subred, se permite el tráfico de red virtual a los recursos especificados en cualquiera de estas directivas. Se denegará el acceso a todos los demás recursos del servicio, que no se estén especificados en ninguna de las directivas.

    > [!NOTE]  
    > Las directivas de punto de conexión de servicio son **directivas de permiso**, por lo que, aparte de los recursos especificados, se restringen todos los demás recursos. Asegúrese de que todas las dependencias de recursos de servicio de las aplicaciones estén identificadas e indicadas en la directiva.

- Solo las cuentas de almacenamiento que utilizan el modelo de recursos de Azure pueden especificarse en la directiva de punto de conexión. Las cuentas de Azure Storage clásicas no son compatibles con las directivas de punto de conexión de servicio de Azure.
- El acceso secundario a RA-GRS se permitirá automáticamente si la cuenta principal aparece en la lista.
- Las cuentas de almacenamiento pueden estar en la misma suscripción o en una diferente o en un inquilino de Azure Active Directory que la red virtual.

## <a name="scenarios"></a>Escenarios

- **Redes virtuales emparejadas, conectadas o múltiples**: para filtrar el tráfico en redes virtuales emparejadas, las directivas de punto de conexión deben aplicarse individualmente a estas redes virtuales.
- **Filtrado del tráfico de Internet con dispositivos de red o Azure Firewall**: filtre el tráfico del servicio Azure con directivas, mediante puntos de conexión, y filtre el resto del tráfico de Internet o Azure a través de dispositivos o de Azure Firewall.
- **Filtrado del tráfico en servicios de Azure implementados en redes virtuales**: en este momento, las directivas de punto de conexión de servicio de Azure no son compatibles con ningún servicio administrado de Azure implementado en la red virtual. 
- **Filtrado del tráfico a los servicios de Azure desde el entorno local**: las directivas de punto de conexión de servicio solo se aplican al tráfico de las subredes asociadas a las directivas. Para permitir el acceso a recursos de servicio específicos de Azure desde el entorno local, el tráfico debe filtrarse utilizando aplicaciones de redes virtuales o firewalls.

## <a name="logging-and-troubleshooting"></a>Registro y solución de problemas
No se dispone de un registro centralizado para las directivas de punto de conexión de servicio. Para los registros de recursos de servicio, consulte [Registro de puntos de conexión de servicio](virtual-network-service-endpoints-overview.md#logging-and-troubleshooting).

### <a name="troubleshooting-scenarios"></a>Escenarios de solución de problemas
- Acceso denegado a las cuentas de almacenamiento que funcionaban en la versión preliminar (no en región emparejada geográficamente)
  - Al actualizar Azure Storage para usar etiquetas de servicio globales, el ámbito del punto de conexión de servicio y, por tanto, las directivas de punto de conexión de servicio, ahora es Global. Por lo tanto, todo el tráfico a Azure Storage se cifra mediante puntos de conexión de servicio y solo se permite el acceso a las cuentas de almacenamiento que aparecen explícitamente en la directiva.
  - Permita de forma explícita en la lista todas las cuentas de almacenamiento necesarias para restaurar el acceso.  
  - Póngase en contacto con el equipo de soporte técnico de Azure.
- Se niega el acceso a las cuentas enumeradas en las directivas de punto de conexión.
  - Los grupos de seguridad de red o el filtrado de firewall podrían estar bloqueando el acceso.
  - Si al quitar o volver a aplicar la directiva se produce una pérdida de conectividad:
    - Valide si el servicio Azure está configurado para permitir el acceso desde la red virtual a través de puntos de conexión, o que la directiva predeterminada del recurso esté establecida en *Permitir todo*.
    - Valide que los diagnósticos de servicio muestran el tráfico sobre los puntos de conexión.
    - Compruebe si los registros de flujo de grupo de seguridad de red muestran el acceso y que los registros de almacenamiento muestran el acceso, como se esperaba, a los puntos de conexión de servicio.
    - Póngase en contacto con el equipo de soporte técnico de Azure.
- Se niega el acceso a las cuentas no enumeradas en las directivas de punto de conexión.
  - Valide si Azure Storage está configurado para permitir el acceso desde la red virtual a través de puntos de conexión, o si la directiva predeterminada del recurso está establecida en *Permitir todo*.
  - Asegúrese de que las cuentas no sean **cuentas de almacenamiento clásicas** con directivas de punto de conexión de servicio en la subred.
- Un servicio de Azure administrado ha dejado de funcionar después de aplicar una directiva de punto de conexión de servicio a través de la subred
  - Los servicios administrados no son compatibles con las directivas de punto de conexión de servicio de momento. *No pierda de vista este espacio para ver si hay actualizaciones*.

## <a name="provisioning"></a>Aprovisionamiento

Las directivas de punto de conexión de servicio pueden configurarse en subredes por un usuario con acceso de escritura a una red virtual. Obtenga más información sobre los [roles integrados](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Azure y la asignación de permisos específicos a [roles personalizados](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Las redes virtuales y las cuentas de Azure Storage pueden encontrarse en la misma o en diferentes suscripciones, o en inquilinos de Azure Active Directory.

## <a name="limitations"></a>Limitaciones

- Solo puede implementar las directivas de punto de conexión de servicio en las redes virtuales implementadas con el modelo de implementación de Azure Resource Manager.
- Las redes virtuales deben estar en la misma región que la directiva de punto de conexión de servicio.
- Solo puede aplicar la directiva de punto de conexión de servicio en una subred si los puntos de conexión de servicio están configurados para los servicios de Azure enumerados en la directiva.
- No puede utilizar las directivas de punto de conexión de servicio para el tráfico de la red local a los servicios de Azure.
- Los servicios administrados de Azure no admiten actualmente directivas de punto de conexión. Esto incluye los servicios administrados implementados en las subredes compartidas (por ejemplo, *Azure HDInsight, Azure Batch, Azure AD DS, Azure Application Gateway, Azure VPN Gateway, Azure Firewall*) o en las subredes dedicadas (por ejemplo, *Azure App Service Environment, Azure Redis Cache, Azure API Management, Instancia administrada de Azure SQL, servicios administrados clásicos*).

 > [!WARNING]
 > Los servicios de Azure implementados en la red virtual, como Azure HDInsight, acceden a otros servicios de Azure, como Azure Storage, para saber los requisitos de infraestructura. Restringir la directiva de punto de conexión a recursos específicos podría interrumpir el acceso a estos recursos de infraestructura para los servicios de Azure implementados en la red virtual.

- no se admiten las cuentas de almacenamiento clásicas en las directivas de punto de conexión. Las directivas denegarán el acceso a todas las cuentas de almacenamiento clásicas de forma predeterminada. Si su aplicación necesita acceso a Azure Resource Manager y a las cuentas de almacenamiento clásicas, las directivas de punto de conexión no deben usarse para este tráfico.

## <a name="pricing-and-limits"></a>Precios y límites

No hay ningún cargo adicional para el uso de directivas de punto de conexión de servicio. El modelo de precios vigente para los servicios de Azure (como Azure Storage) se aplica tal cual, sobre los punto de conexión de servicio.

Los límites siguientes se aplican en las directivas de punto de conexión de servicio: 

 |Resource | Límite predeterminado |
 |---------|---------------|
 |ServiceEndpointPoliciesPerSubscription |500 |
 |ServiceEndpointPoliciesPerSubnet|100 |
 |ServiceResourcesPerServiceEndpointPolicyDefinition|200 |

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [configurar las directivas de puntos de conexión de servicio de red virtual](virtual-network-service-endpoint-policies-portal.md).
- Aprenda más sobre los [puntos de conexión de servicio de red virtual](virtual-network-service-endpoints-overview.md).
