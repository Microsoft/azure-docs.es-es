---
title: Uso de puntos de conexión privados con cuentas de Azure Batch
description: Aprenda cómo conectarse de forma privada a una cuenta de Azure Batch mediante puntos de conexión privados.
ms.topic: how-to
ms.date: 08/07/2020
ms.custom: references_regions
ms.openlocfilehash: 0fd16e4e11d0b3f08a7ba0e2f425785e3cce7927
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88814117"
---
# <a name="use-private-endpoints-with-azure-batch-accounts"></a>Uso de puntos de conexión privados con cuentas de Azure Batch

De forma predeterminada, las [cuentas de Azure Batch](accounts.md) tienen un punto de conexión público y son accesibles públicamente. El servicio Batch ofrece la posibilidad de crear cuentas de Batch privadas, deshabilitando el acceso a la red pública.

Al usar [Azure Private Link](../private-link/private-link-overview.md), puede conectarse a una cuenta de Azure Batch mediante un [punto de conexión privado](../private-link/private-endpoint-overview.md). El punto de conexión privado es un conjunto de direcciones IP privadas en una subred dentro de la red virtual. A continuación, puede limitar el acceso a una cuenta de Azure Batch a través de direcciones IP privadas.

Private Link permite a los usuarios obtener acceso a una cuenta de Azure Batch desde dentro de la red virtual o cualquier red virtual emparejada. También se puede acceder a los recursos asignados a Private Link en el entorno local a través de un emparejamiento privado a través de VPN o [Azure ExpressRoute](../expressroute/expressroute-introduction.md). Puede conectarse a una cuenta de Azure Bacth configurada con Private Link mediante el [método de aprobación automático o manual](../private-link/private-endpoint-overview.md#access-to-a-private-link-resource-using-approval-workflow).

> [!IMPORTANT]
> La compatibilidad con la conectividad privada en Azure Batch está disponible actualmente en las siguientes regiones de Azure: Centro de EE. UU., Centro-norte de EE. UU., Centro-sur de EE. UU., Centro-oeste de EE. UU., Este de EE. UU., Este de EE. UU. 2, Oeste de EE. UU., Oeste de EE. UU. 2, Este de Asia, Centro de Francia, Sur de Reino Unido, Norte de Europa, Oeste de Europa, Este de Japón, Oeste de Japón, Este de Australia y todas las regiones de US Gov y US DoD.

En este artículo se describen los pasos para crear una cuenta de Batch privada y cómo acceder a ella mediante un punto de conexión privado.

## <a name="azure-portal"></a>Azure portal

Siga estos pasos para crear una cuenta de Batch privada con Azure Portal:

1. En el panel **Crear un recurso**, seleccione **Servicio batch** y, a continuación, seleccione **Crear**.
2. Escriba la suscripción, el grupo de recursos, la región y el nombre de la cuenta de Batch en la pestaña **Aspectos básicos** y, después, seleccione **Siguiente: Avanzado**.
3. En la pestaña **Avanzado**, establezca **Public network access** (Acceso de red público) en **Deshabilitado**.
4. En **Configuración**, seleccione **Conexiones de punto de conexión privado** y, a continuación, seleccione **+ Punto de conexión privado**.
   :::image type="content" source="media/private-connectivity/private-endpoint-connections.png" alt-text="Conexiones de punto de conexión privado":::
5. En el panel **Aspectos básicos**, escriba o seleccione la suscripción, el grupo de recursos, el nombre del recurso de punto de conexión privado y los detalles de la región y, a continuación, seleccione **Siguiente: Resource** (Siguiente: Recurso).
6. En el panel **Recurso**, establezca **Tipo de recurso** en **Microsoft.Batch/batchAccounts**. Seleccione la cuenta de Batch privada a la que desee acceder y, a continuación, seleccione **Siguiente: Configuration** (Siguiente: Configuración).
   :::image type="content" source="media/private-connectivity/create-private-endpoint.png" alt-text="Creación de un punto de conexión privado: panel Recurso":::
7. En el panel **Configuración**, escriba o seleccione esta información:
   - **Red virtual**: Seleccione la red virtual.
   - **Subred**: Seleccionar la subred.
   - **Integración con una zona DNS privada**:   Seleccione **Sí**. Para conectar de forma privada con el punto de conexión privado, necesita un registro DNS. Se recomienda integrar el punto de conexión privado con una zona DNS privada. También se pueden usar los servidores DNS propios o bien crear registros DNS con los archivos de host de las máquinas virtuales.
   - **Zona DNS privada**:  Seleccione privatelink.<region>.batch.azure.com. La zona DNS privada se determina automáticamente. No puede cambiarla con Azure Portal.
8. Seleccione **Revisar y crear** y espere a que Azure valide la configuración.
9. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**.

Después de aprovisionar el punto de conexión privado, puede acceder a la cuenta de Batch desde las máquinas virtuales de la misma red virtual mediante el punto de conexión privado. Para ver las direcciones IP desde Azure Portal:

1. Seleccione **Todos los recursos**.
2. Busque el punto de conexión privado que creó anteriormente.
3. Seleccione la pestaña **Información general** para ver la configuración de DNS y las direcciones IP.

:::image type="content" source="media/private-connectivity/access-private.png" alt-text="Configuración DNS y direcciones IP del punto de conexión privado":::

## <a name="azure-resource-manager-template"></a>Plantilla del Administrador de recursos de Azure

Al [crear una cuenta de Batch con una plantilla de Azure Resource Manager](quick-create-template.md), modifique la plantilla para establecer **publicNetworkAccess** en **Deshabilitado**, como se muestra a continuación.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "batchAccountName": {
      "type": "string",
      },
    "location": {
      "type": "string",
    }
  },
  "resources": [
    {
      "name": "[parameters('batchAccountName')]",
      "type": "Microsoft.Batch/batchAccounts",
      "apiVersion": "2020-03-01-preview",
      "location": "[parameters('location')]",
      "dependsOn": []
      "properties": {
        "poolAllocationMode": "BatchService"
        "publicNetworkAccess": "Disabled"
      }
    }
  ]
}
```

## <a name="configure-dns-zones"></a>Configuración de zonas DNS

Use una [zona DNS privada](../dns/private-dns-privatednszone.md) dentro de la subred en la que ha creado el punto de conexión privado. Configure los puntos de conexión de modo que cada dirección IP privada se asigne a una entrada DNS

Al crear el punto de conexión privado, puede integrarlo con una [zona DNS privada](../dns/private-dns-privatednszone.md) en Azure. Si, en su lugar, elige usar un [dominio personalizada](../dns/dns-custom-domain.md), debe configurarlo para agregar registros DNS para todas las direcciones IP privadas reservadas para el punto de conexión privado.

## <a name="pricing"></a>Precios

Para más información sobre los costos relacionados con los puntos de conexión privados, consulte [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link/).

## <a name="current-limitations-and-best-practices"></a>Limitaciones actuales y procedimientos recomendados

Cuando cree la cuenta de Batch privada, recuerde lo siguiente:

- Los recursos del punto de conexión privado deben crearse en la misma suscripción que la cuenta de Batch.
- Para eliminar la conexión privada, debe eliminar el recurso del punto de conexión privado.
- Una vez que se crea una cuenta de Batch con acceso de red público, no se puede cambiar a solo acceso privado.
- Los registros DNS de la zona DNS privada no se quitan automáticamente cuando se elimina un punto de conexión privado o se quita una región de la cuenta de Azure Batch. Debe quitar manualmente los registros DNS antes de agregar un nuevo punto de conexión privado vinculado a esta zona DNS privada. Si no se limpian los registros DNS, pueden producirse problemas inesperados en el plano de datos, como la interrupción de datos en las regiones agregadas después de la eliminación de puntos de conexión privados o de la eliminación de regiones.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [crear grupos de Batch en redes virtuales](batch-virtual-network.md).
- Obtenga información sobre cómo [crear grupos de Batch con direcciones IP públicas especificadas](create-pool-public-ip.md).
- Obtenga información sobre [Azure Private Link](../private-link/private-link-overview.md).
