---
title: Traslado de recursos de Azure a una nueva suscripción o grupo de recursos | Microsoft Docs
description: Use Azure Resource Manager para trasladar recursos a un nuevo grupo de recursos o a una nueva suscripción.
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 05/16/2019
ms.author: tomfitz
ms.openlocfilehash: 076d120d9c02b15837e92b71bc2a015377f54594
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65792695"
---
# <a name="move-resources-to-new-resource-group-or-subscription"></a>Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción

En este artículo se explica cómo trasladar recursos de Azure a otra suscripción de Azure o a otro grupo de recursos en la misma suscripción. Puede usar Azure Portal, Azure PowerShell, la CLI de Azure o la API REST para trasladar recursos.

Tanto el grupo de origen como el grupo de destino se bloquean durante la operación de traslado. Las operaciones de escritura y eliminación están bloqueadas en los grupos de recursos hasta que se completa el movimiento. Este bloqueo significa que no puede agregar, actualizar ni eliminar recursos de los grupos de recursos, pero no que los recursos queden bloqueados. Por ejemplo, si mueve un servidor SQL Server y su base de datos a un nuevo grupo de recursos, una aplicación que utiliza la base de datos no experimenta ningún tiempo de inactividad. Todavía puede leer y escribir en la base de datos.

Si se mueve un recurso, solo se mueve a un nuevo grupo de recursos. La operación de traslado no puede cambiar la ubicación del recurso. El nuevo grupo de recursos puede tener una ubicación diferente, pero eso no cambia la ubicación del recurso.

> [!NOTE]
> En este artículo se describe cómo mover los recursos entre las distintas suscripciones de Azure. Si quiere actualizar la suscripción de Azure (por ejemplo, para cambiar de gratuita a de pago por uso) deberá convertir su suscripción.
> * Para actualizar a una evaluación gratuita, consulte [Actualización de la suscripción de Microsoft Imagine Azure o la prueba gratuita al plan de pago por uso](..//billing/billing-upgrade-azure-subscription.md).
> * Para cambiar a una cuenta de pago por uso, consulte [Cambie la suscripción de pago por uso de Azure a otra oferta](../billing/billing-how-to-switch-azure-offer.md).
> * Si no puede convertir la suscripción, [cree una solicitud de soporte técnico de Azure](../azure-supportability/how-to-create-azure-support-request.md). Seleccione **Administración de suscripciones** para el tipo de problema.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="when-to-call-azure-support"></a>Al llamar al soporte técnico de Azure

Puede trasladar la mayoría de los recursos a través de las operaciones de autoservicio que se muestran en este artículo. Utilice las operaciones de autoservicio para:

* Trasladar recursos de Resource Manager.
* Trasladar recursos clásicos conforme a las [limitaciones de implementación clásica](#classic-deployment-limitations).

Póngase en contacto con [soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) cuando necesite:

* Traslade los recursos a una nueva cuenta de Azure (e inquilino de Azure Active Directory) y si necesita ayuda consulte las instrucciones de la sección anterior.
* Trasladar recursos clásicos, pero que tienen problemas con las limitaciones.

## <a name="services-that-can-be-moved"></a>Servicios que se pueden mover

En la lista siguiente se proporciona un resumen general de servicios de Azure que se pueden mover a un grupo de recursos y una suscripción nuevos. Para obtener una lista de qué recurso tipos admiten el movimiento, consulte [para recursos de compatibilidad con la operación de mover](move-support-resources.md).

* Analysis Services
* API Management
* Aplicaciones de App Service (aplicaciones web) - consulte las [limitaciones de App Service](#app-service-limitations)
* Certificados de App Service: consulte [Limitaciones de App Service Certificate](#app-service-certificate-limitations)
* Automatización: los runbooks deben existir en el mismo grupo de recursos que la cuenta de Automation.
* Azure Active Directory B2C
* Azure Cache for Redis: si la instancia de Azure Cache for Redis está configurada con una red virtual, la instancia no se puede mover a otra suscripción. Vea [Virtual Networks limitations](#virtual-networks-limitations) (Limitaciones de las redes virtuales).
* Azure Cosmos DB
* Explorador de datos de Azure
* Azure Database for MariaDB
* Azure Database for MySQL
* Azure Database for PostgreSQL
* Azure DevOps: siga los pasos para [cambiar la suscripción de Azure usada para la facturación](/azure/devops/organizations/billing/change-azure-subscription?view=azure-devops).
* Azure Maps
* Registros de Azure Monitor
* Azure Relay
* Azure Stack: registros
* Lote
* BizTalk Services
* Servicio de bots
* CDN
* Cloud Services (consulte las [limitaciones de la implementación clásica](#classic-deployment-limitations)
* Cognitive Services
* Container Registry
* Content Moderator
* Administración de costos
* Customer Insights
* Data Catalog
* Data Factory
* Data Lake Analytics
* Data Lake Store
* DNS
* Event Grid
* Event Hubs
* Front Door
* Clústeres de HDInsight: consulte [Limitaciones de HDInsight](#hdinsight-limitations).
* Iot Central
* IoT Hubs
* Key Vault: los almacenes de claves utilizados para el cifrado de discos no se pueden mover a grupos de recursos de la misma suscripción o de varias suscripciones.
* Equilibradores de carga: el equilibrador de carga de SKU básico se puede mover. En cambio, el equilibrador de carga de SKU estándar no se puede mover.
* Logic Apps
* Machine Learning: los servicios de Machine Learning Studio se pueden mover a un grupo de recursos en la misma suscripción, pero no una suscripción diferente. Otros recursos de Machine Learning se pueden mover entre suscripciones.
* Managed Disks: Managed Disks en zonas de disponibilidad no se puede mover a otra suscripción
* Identidad administrada: asignada por el usuario
* Servicios multimedia
* Supervisión: asegúrese de que el cambio a una nueva suscripción no exceda las [cuotas de suscripción](../azure-subscription-service-limits.md#monitor-limits).
* Notification Hubs
* Operational Insights
* Operations Management
* Paneles del portal
* Power BI (tanto Power BI Embedded como Colección de áreas de trabajo de Power BI)
* Dirección IP pública: la dirección IP de SKU básica se puede mover. Las direcciones IP públicas de SKU Estándar no se pueden mover.
* Almacén de Recovery Services: inscríbase en una [versión preliminar](#recovery-services-limitations).
* SAP HANA en Azure
* Programador
* Search: no puede trasladar varios recursos de Search en regiones diferentes en una operación. En su lugar, muévalos en operaciones independientes.
* Azure Service Bus
* Service Fabric
* Service Fabric Mesh
* Servicio SignalR
* Storage: las cuentas de almacenamiento de regiones diferentes no se pueden mover en la misma operación. En su lugar, use operaciones independientes para cada región.
* Storage (clásico); consulte las [limitaciones de la implementación clásica](#classic-deployment-limitations)
* Servicio de sincronización de almacenamiento
* Stream Analytics: los trabajos de Stream Analytics no se pueden mover si se encuentran en estado de ejecución.
* Servidor de SQL Database: la base de datos y el servidor deben residir en el mismo grupo de recursos. Cuando se mueve un servidor SQL Server, se mueven también todas sus bases de datos. Este comportamiento se aplica a las bases de datos de Azure SQL Database y Azure SQL Data Warehouse.
* Time Series Insights
* Traffic Manager
* Vea las máquinas virtuales - [limitaciones de Virtual Machines](#virtual-machines-limitations)
* Virtual Machines (clásico); consulte las [limitaciones de la implementación clásica](#classic-deployment-limitations)
* Conjuntos de escalado de máquinas virtuales; vea [Limitaciones de Virtual Machines](#virtual-machines-limitations).
* Redes virtuales; vea [Limitaciones de las redes virtuales](#virtual-networks-limitations).
* VPN Gateway

### <a name="services-that-cannot-be-moved"></a>Recursos que no se pueden mover

En la lista siguiente se proporciona un resumen general de servicios de Azure que no se pueden mover a un grupo de recursos y una suscripción nuevos. Para obtener más detalles, vea [Traslado del soporte técnico por tipo de recurso](move-support-resources.md).

* Servicios de dominio de AD
* Servicio de mantenimiento híbrido de AD
* Application Gateway
* Azure Database Migration
* Azure Databricks
* Azure Firewall
* Azure Kubernetes Service (AKS)
* Azure Migrate
* Azure NetApp Files
* Certificados: los certificados de App Service se pueden trasladar, pero los certificados cargados tienen [limitaciones](#app-service-limitations).
* Aplicaciones clásicas
* Instancias de contenedor
* Servicio de contenedor
* Data Box
* Dev Spaces
* Dynamics LCS
* ExpressRoute
* Los servicios de laboratorio - laboratorios de clase no se puede mover a un nuevo grupo de recursos o suscripción. DevTest Labs se pueden mover a un nuevo grupo de recursos en la misma suscripción, pero no a través de suscripciones.
* Aplicaciones administradas
* Microsoft Genomics
* Seguridad
* Site Recovery
* Administrador de dispositivos de StorSimple
* Virtual Networks (clásico); consulte las [limitaciones de la implementación clásica](#classic-deployment-limitations)

## <a name="limitations"></a>Limitaciones

La sección proporciona descripciones de cómo tratar escenarios complicados para trasladar recursos. Las limitaciones son las siguientes:

* [Limitaciones de máquinas virtuales](#virtual-machines-limitations)
* [Limitaciones de las redes virtuales](#virtual-networks-limitations)
* [Limitaciones de App Service](#app-service-limitations)
* [Limitaciones de App Service Certificate](#app-service-certificate-limitations)
* [Limitaciones de la implementación clásica](#classic-deployment-limitations)
* [Limitaciones de Recovery Services](#recovery-services-limitations)
* [Limitaciones de HDInsight](#hdinsight-limitations)

### <a name="virtual-machines-limitations"></a>Limitaciones de Virtual Machines

Puede mover máquinas virtuales con los discos administrados, imágenes administradas, instantáneas administradas y conjuntos de disponibilidad con máquinas virtuales que usan discos administrados. No se puede mover discos administrados en las zonas de disponibilidad a una suscripción diferente.

Todavía no se admiten los siguientes escenarios:

* Los recursos de Virtual Machines con certificado almacenados en Key Vault se pueden trasladar al nuevo grupo de recursos en la misma suscripción, pero no entre suscripciones.
* No se puede mover conjuntos de escalado de máquinas virtuales con equilibrador de carga de SKU estándar o IP pública de SKU estándar.
* Las máquinas virtuales creadas a partir de recursos de Marketplace con planes adjuntos no se pueden mover entre suscripciones o grupos de recursos. Desaprovisione el recurso en la suscripción activa y vuelva a implementarlo en la nueva suscripción.

Para mover máquinas virtuales configuradas con la copia de seguridad de Azure, use la siguiente solución alternativa:

* Busque la ubicación de la máquina virtual.
* Busque un grupo de recursos con el patrón de nombres siguiente: `AzureBackupRG_<location of your VM>_1`, por ejemplo, AzureBackupRG_westus2_1
* Si está en Azure Portal, active "Mostrar tipos ocultos"
* Si se encuentra en PowerShell, use el cmdlet `Get-AzResource -ResourceGroupName AzureBackupRG_<location of your VM>_1`
* Si está en la CLI, use `az resource list -g AzureBackupRG_<location of your VM>_1`
* Busque el recurso con el tipo `Microsoft.Compute/restorePointCollections` que tiene el patrón de nombres `AzureBackup_<name of your VM that you're trying to move>_###########`
* Elimine este recurso. Esta operación elimina solo los puntos de recuperación instantáneos, no los datos de copia de seguridad que se encuentran en el almacén.
* Una vez completada la eliminación, podrá mover la máquina virtual. Puede trasladar el almacén y la máquina virtual a la suscripción de destino. Tras el traslado, puede continuar realizando las copias de seguridad sin pérdida de datos.
* Para más información sobre cómo mover los almacenes de Recovery Services para realizar copias de seguridad, consulte [Limitaciones de Recovery Services](#recovery-services-limitations).

### <a name="virtual-networks-limitations"></a>Limitaciones de las redes virtuales

Si mueve una red virtual, también deberá mover sus recursos dependientes. En el caso de las instancias de VPN Gateway, debe mover las direcciones IP, las puertas de enlace de red virtuales y todos los recursos de conexión asociados. Las puertas de enlace de red local pueden estar en otro grupo de recursos.

Para mover una máquina virtual con una tarjeta de interfaz de red, debe mover todos los recursos dependientes. Debe mover la red virtual para la tarjeta de interfaz de red, todas las otras tarjetas de interfaz de red para la red virtual y las puertas de enlace VPN.

Para mover una red virtual emparejada, primero debe deshabilitar el emparejamiento de red virtual. Una vez deshabilitado, puede mover la red virtual. Después de moverla, vuelva a habilitar el emparejamiento de red virtual.

Si una red virtual contiene una subred con vínculos de navegación de los recursos, no se puede mover a otra suscripción. Por ejemplo, si un recurso de Azure Cache for Redis está implementado en una subred, esta contiene un vínculo de navegación de recursos.

### <a name="app-service-limitations"></a>limitaciones de App Service

Las limitaciones para mover recursos de App Service son diferentes en función de si los recursos se mueven dentro de una suscripción o a una nueva. Si su aplicación web usa una instancia de App Service Certificate, consulte [Limitaciones de App Service Certificate](#app-service-certificate-limitations)

#### <a name="moving-within-the-same-subscription"></a>Moverlos dentro de la misma suscripción

Al mover una instancia de Web App _dentro de la misma suscripción_, no se pueden mover los certificados SSL de terceros. Sin embargo, puede mover una instancia de Web App al nuevo grupo de recursos sin mover su certificado de terceros y la funcionalidad SSL de la aplicación seguirá funcionando.

Si desea mover el certificado SSL junto con la instancia de Web App, siga estos pasos:

1. Elimine el certificado de terceros desde Web App, pero guarde una copia de su certificado.
2. Traslade la instancia.
3. Cargue el certificado de terceros en la instancia de Web App trasladada.

#### <a name="moving-across-subscriptions"></a>Moverlos entre suscripciones

Al mover una instancia de Web App _entre suscripciones_, se aplican las limitaciones siguientes:

- El grupo de recursos de destino no debe tener ningún recurso de App Service. Entre los recursos de App Service se incluyen:
    - Web Apps
    - Planes de App Service
    - Certificados SSL cargados o importados
    - Entornos de App Service
- Todos los recursos de App Service del grupo de recursos se deben mover conjuntamente.
- Los recursos de App Service solo se pueden mover del grupo de recursos en el que se crearon originalmente. Si un recurso de App Service ya no se encuentra en su grupo de recursos original, deberá devolverse a este en primer lugar y, a continuación, se podrá mover entre suscripciones.

Si no recuerda el grupo de recursos original, se puede encontrar a través de diagnóstico. Para la aplicación web, seleccione **diagnosticar y resolver problemas**. A continuación, seleccione **configuración y administración**.

![Seleccionar diagnósticos](./media/resource-group-move-resources/select-diagnostics.png)

Seleccione **opciones de migración**.

![Seleccione las opciones de migración](./media/resource-group-move-resources/select-migration.png)

Seleccione la opción para los pasos recomendados mover la aplicación web.

![Seleccione los pasos recomendados](./media/resource-group-move-resources/recommended-steps.png)

Vea las acciones recomendadas que deben realizarse antes de mover los recursos. La información incluye el grupo de recursos original de la aplicación web.

![Recomendaciones](./media/resource-group-move-resources/recommendations.png)

### <a name="app-service-certificate-limitations"></a>Limitaciones de App Service Certificate

Puede mover su instancia de App Service Certificate a un nuevo grupo de recursos o a una nueva suscripción. Si su instancia de App Service Certificate está enlazada a una aplicación web, debe seguir algunos pasos antes de trasladar los recursos a una nueva suscripción. Elimine el enlace SSL y el certificado privado de la aplicación web antes de trasladar los recursos. No es necesario eliminar la instancia de App Service Certificate, solo el certificado privado en la aplicación web.

### <a name="classic-deployment-limitations"></a>limitaciones de la implementación clásica

Las opciones para mover recursos implementados con el modelo clásico varían en función de si traslada los recursos dentro de una misma suscripción o a una nueva suscripción.

#### <a name="same-subscription"></a>Misma suscripción

Al mover recursos de un grupo de recursos a otro dentro de la misma suscripción, se aplican las restricciones siguientes:

* No se pueden mover redes virtuales (clásico).
* Las máquinas virtuales (clásico) se deben mover con el servicio en la nube.
* El servicio en la nube solo se puede mover cuando el traslado incluye todas sus máquinas virtuales.
* Solo se puede mover un servicio en la nube cada vez.
* Solo se puede mover una cuenta de almacenamiento (clásico) cada vez.
* No se puede mover una cuenta de almacenamiento (clásico) con una máquina virtual o un servicio en la nube en la misma operación de traslado.

Para mover recursos clásicos a un grupo de recursos nuevo de la misma suscripción, utilice las operaciones de traslado estándar a través del [portal](#use-portal), Azure PowerShell, la CLI de Azure o la API REST. Utilice las mismas operaciones que utiliza para trasladar recursos de Resource Manager.

#### <a name="new-subscription"></a>Suscripción nueva

Al trasladar recursos a una nueva suscripción, se aplican las restricciones siguientes:

* Todos los recursos clásicos de la suscripción se deben mover en la misma operación.
* La suscripción de destino no debe contener otros recursos clásicos.
* El traslado solo puede solicitarse a través de una API de REST independiente para el traslado de recursos clásicos. Los comandos de movimiento estándar de Resource Manager no funcionan para mover recursos clásicos a una nueva suscripción.

Para trasladar recursos clásicos a una nueva suscripción, use operaciones REST específicas para recursos clásicos. Para usar REST, realice los pasos siguientes:

1. Compruebe si la suscripción de origen puede participar en un movimiento entre suscripciones. Utilice la siguiente operación:

   ```HTTP
   POST https://management.azure.com/subscriptions/{sourceSubscriptionId}/providers/Microsoft.ClassicCompute/validateSubscriptionMoveAvailability?api-version=2016-04-01
   ```

     En el cuerpo de la solicitud, incluya:

   ```json
   {
    "role": "source"
   }
   ```

     La respuesta para la operación de validación está en el formato siguiente:

   ```json
   {
    "status": "{status}",
    "reasons": [
      "reason1",
      "reason2"
    ]
   }
   ```

2. Compruebe si la suscripción de destino puede participar en un movimiento entre suscripciones. Utilice la siguiente operación:

   ```HTTP
   POST https://management.azure.com/subscriptions/{destinationSubscriptionId}/providers/Microsoft.ClassicCompute/validateSubscriptionMoveAvailability?api-version=2016-04-01
   ```

     En el cuerpo de la solicitud, incluya:

   ```json
   {
    "role": "target"
   }
   ```

     La respuesta está en el mismo formato que la validación de la suscripción de origen.
3. Si ambas suscripciones superan la validación, traslade todos los recursos clásicos de una suscripción a otra con la siguiente operación:

   ```HTTP
   POST https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.ClassicCompute/moveSubscriptionResources?api-version=2016-04-01
   ```

    En el cuerpo de la solicitud, incluya:

   ```json
   {
    "target": "/subscriptions/{target-subscription-id}"
   }
   ```

Es posible que esta operación tarde varios minutos.

### <a name="recovery-services-limitations"></a>Limitaciones de Recovery Services

 Para trasladar un almacén de Recovery Services, debe inscribirse en una [versión preliminar pública limitada](../backup/backup-azure-move-recovery-services-vault.md).

Actualmente, puede trasladar un almacén de Recovery Services, por región, cada vez. No puede trasladar aquellos almacenes que realizan copias de seguridad de Azure Files, Azure File Sync o SQL en máquinas virtuales de IaaS.

Si una máquina virtual no se traslada con el almacén, los puntos de recuperación de máquinas virtuales actuales permanecerán en el almacén hasta su expiración. Independientemente de que la máquina virtual se trasladara con el almacén o no, podrá restaurar la máquina virtual desde el historial de copia de seguridad en el almacén.

El almacén de Recovery Services no admite copias de seguridad entre suscripciones. Si traslada un almacén con datos de copia de seguridad de máquinas virtuales entre las suscripciones, deberá trasladar sus máquinas virtuales a la misma suscripción y usar el mismo grupo de recursos de destino para continuar con las copias de seguridad.

Las directivas de copia de seguridad definidas para el almacén se conservan una vez trasladado el almacén. La creación de informes y la supervisión deben configurarse de nuevo para el almacén tras el traslado.

Para trasladar una máquina virtual a una nueva suscripción sin trasladar el almacén de Recovery Services:

 1. Detenga temporalmente la creación de las copias de seguridad
 1. [Eliminar el punto de restauración](#virtual-machines-limitations). Esta operación elimina solo los puntos de recuperación instantáneos, no los datos de copia de seguridad que se encuentran en el almacén.
 1. Traslade las máquinas virtuales a la nueva suscripción
 1. Vuelva a protegerla en un nuevo almacén de esa suscripción

No se admite el traslado para recursos de Storage, Network o Compute que se usan para configurar la recuperación ante desastres de Azure Site Recovery. Por ejemplo, suponga que ha configurado la replicación de las máquinas locales en una cuenta de almacenamiento (Storage1) y desea que la máquina protegida aparezca después de la conmutación por error en Azure como una máquina virtual (VM1) conectada a una red virtual (Network1). No puede mover ninguno de estos recursos de Azure: Storage1, VM1 y Network1, a grupos de recursos dentro de la misma suscripción o entre suscripciones.

### <a name="hdinsight-limitations"></a>Limitaciones de HDInsight

Puede mover clústeres de HDInsight a una nueva suscripción o un nuevo grupo de recursos. Sin embargo, no puede mover entre suscripciones los recursos de red vinculados al clúster de HDInsight (por ejemplo, la red virtual, una NIC o un equilibrador de carga). Además, tampoco se puede mover a un nuevo grupo de recursos una NIC que está conectada a una máquina virtual del clúster.

Al mover un clúster de HDInsight a una nueva suscripción, mueva primero otros recursos (por ejemplo, la cuenta de almacenamiento). Después, mover el clúster de HDInsight.

## <a name="checklist-before-moving-resources"></a>Lista de comprobación antes de mover recursos

Hay algunos pasos importantes que deben realizarse antes de mover un recurso. Puede evitar errores mediante la comprobación de estas condiciones.

1. Las suscripciones de origen y de destino deben estar activas. Si tiene problemas para habilitar una cuenta que se ha deshabilitado, [cree una solicitud de soporte técnico de Azure](../azure-supportability/how-to-create-azure-support-request.md). Seleccione **Administración de suscripciones** para el tipo de problema.

1. Las suscripciones de origen y destino deben existir en el mismo [inquilino de Azure Active Directory](../active-directory/develop/quickstart-create-new-tenant.md). Para comprobar que ambas suscripciones tienen el mismo identificador de inquilino, utilice Azure PowerShell o la CLI de Azure.

   Para Azure PowerShell, use:

   ```azurepowershell-interactive
   (Get-AzSubscription -SubscriptionName <your-source-subscription>).TenantId
   (Get-AzSubscription -SubscriptionName <your-destination-subscription>).TenantId
   ```

   Para la CLI de Azure, utilice:

   ```azurecli-interactive
   az account show --subscription <your-source-subscription> --query tenantId
   az account show --subscription <your-destination-subscription> --query tenantId
   ```

   Si los identificadores de inquilino de las suscripciones de origen y destino no son los mismos, use los siguientes métodos para conciliarlos:

   * [Transferencia de la propiedad de una suscripción de Azure a otra cuenta](../billing/billing-subscription-transfer.md)
   * [Asociación o adición de una suscripción de Azure a Azure Active Directory](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)

1. La suscripción de destino correspondiente al proveedor de recursos del recurso que se traslada debe estar registrada. Si no es así, recibirá un error en el que se indicará que la **suscripción no está registrada para un tipo de recurso**. Podría encontrar este error al mover un recurso a una nueva suscripción que nunca se ha utilizado con ese tipo de recurso.

   En PowerShell, use los siguientes comandos para obtener el estado de registro:

   ```azurepowershell-interactive
   Set-AzContext -Subscription <destination-subscription-name-or-id>
   Get-AzResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
   ```

   Para registrar un proveedor de recursos, use:

   ```azurepowershell-interactive
   Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
   ```

   En la CLI de Azure, use los siguientes comandos para obtener el estado de registro:

   ```azurecli-interactive
   az account set -s <destination-subscription-name-or-id>
   az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
   ```

   Para registrar un proveedor de recursos, use:

   ```azurecli-interactive
   az provider register --namespace Microsoft.Batch
   ```

1. La cuenta que mueve los recursos debe tener al menos los permisos siguientes:

   * **Microsoft.Resources/subscriptions/resourceGroups/moveResources/action** en el grupo de recursos de origen.
   * **Microsoft.Resources/subscriptions/resourceGroups/write** en el grupo de recursos de destino.

1. Antes de mover los recursos, compruebe las cuotas de la suscripción a la que está trasladando los recursos. Si trasladar los recursos significa que la suscripción excederá sus límites, debe revisar si puede solicitar un aumento de la cuota. Para ver una lista de estos límites y cómo solicitar un aumento, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-subscription-service-limits.md).

1. Cuando sea posible, divida las operaciones de movimiento grandes en varias operaciones de movimiento independientes. Resource Manager inmediatamente devolverá un error si hay más de 800 recursos en una sola operación. No obstante, también se puede producir un error por agotamiento del tiempo de espera al mover menos de 800 recursos.

1. El servicio debe permitir la capacidad de traslado de recursos. Para determinar si el traslado se realizará correctamente, [valide la solicitud de movimiento](#validate-move). Consulte las secciones siguientes de este artículo sobre los [servicios que permiten mover recursos](#services-that-can-be-moved) y los [servicios que no permiten mover recursos](#services-that-cannot-be-moved).

## <a name="validate-move"></a>Validar el movimiento

La [operación de validación del movimiento](/rest/api/resources/resources/validatemoveresources) le permite probar el escenario de movimiento sin mover realmente los recursos. Use esta operación para determinar si el movimiento se realizará correctamente. Para ejecutar esta operación, necesita el:

* nombre del grupo de recursos de origen
* identificador de recurso del grupo de recursos de destino
* identificador de recurso de cada recurso que se va a mover
* [token de acceso](/rest/api/azure/#acquire-an-access-token) de la cuenta

Envíe la solicitud siguiente:

```
POST https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<source-group>/validateMoveResources?api-version=2018-02-01
Authorization: Bearer <access-token>
Content-type: application/json
```

Con un cuerpo de la solicitud:

```json
{
 "resources": ["<resource-id-1>", "<resource-id-2>"],
 "targetResourceGroup": "/subscriptions/<subscription-id>/resourceGroups/<target-group>"
}
```

Si la solicitud tiene un formato correcto, la operación devuelve:

```
Response Code: 202
cache-control: no-cache
pragma: no-cache
expires: -1
location: https://management.azure.com/subscriptions/<subscription-id>/operationresults/<operation-id>?api-version=2018-02-01
retry-after: 15
...
```

El código de estado 202 indica que se ha aceptado la solicitud de validación, pero aún no se ha determinado si la operación de movimiento se realizará correctamente. El valor `location` contiene una dirección URL que se usa para comprobar el estado de la operación de ejecución larga.  

Para comprobar el estado, envíe la solicitud siguiente:

```
GET <location-url>
Authorization: Bearer <access-token>
```

Mientras todavía se esté ejecutando la operación, continuará recibiendo el código de estado 202. Espere el número de segundos que se indica en el valor `retry-after` antes de intentarlo de nuevo. Si la operación de movimiento se valida correctamente, recibirá el código de estado 204. Si falla la validación de movimiento, recibirá un mensaje de error, como:

```json
{"error":{"code":"ResourceMoveProviderValidationFailed","message":"<message>"...}}
```

## <a name="move-resources"></a>Mover recursos

### <a name="a-nameuse-portal-by-using-azure-portal"></a><a name="use-portal" />Mediante Azure Portal

Para trasladar recursos, seleccione el grupo de recursos con esos recursos y, después, el botón **Mover**.

![Mover recursos](./media/resource-group-move-resources/select-move.png)

Seleccione si va a mover los recursos a un nuevo grupo de recursos o a una nueva suscripción.

Seleccione los recursos que trasladar y el grupo de recursos de destino. Confirme que tiene que actualizar los scripts para estos recursos y seleccione **Aceptar**. Si ha seleccionado el icono de edición de la suscripción en el paso anterior, también debe seleccionar la suscripción de destino.

![seleccionar destino](./media/resource-group-move-resources/select-destination.png)

En **Notificaciones**, podrá ver que la operación de traslado está en curso.

![mostrar el estado del traslado](./media/resource-group-move-resources/show-status.png)

Cuando haya finalizado, se le notificará del resultado.

![mostrar el resultado del traslado](./media/resource-group-move-resources/show-result.png)

### <a name="by-using-azure-powershell"></a>Mediante Azure PowerShell.

Para mover recursos existentes a otro grupo de recursos o a otra suscripción, use el comando [Move-AzResource](/powershell/module/az.resources/move-azresource). El siguiente ejemplo muestra cómo trasladar varios recursos a un nuevo grupo de recursos.

```azurepowershell-interactive
$webapp = Get-AzResource -ResourceGroupName OldRG -ResourceName ExampleSite
$plan = Get-AzResource -ResourceGroupName OldRG -ResourceName ExamplePlan
Move-AzResource -DestinationResourceGroupName NewRG -ResourceId $webapp.ResourceId, $plan.ResourceId
```

Para moverlos a una nueva suscripción, especifique un valor para el parámetro `DestinationSubscriptionId`.

### <a name="by-using-azure-cli"></a>Mediante la CLI de Azure

Para trasladar recursos existentes a otro grupo de recursos o a otra suscripción, use el comando [az resource move](/cli/azure/resource?view=azure-cli-latest#az-resource-move). Proporcione los identificadores de recursos de los recursos que se van a mover. El siguiente ejemplo muestra cómo trasladar varios recursos a un nuevo grupo de recursos. En el parámetro `--ids`, ofrezca una lista separada por espacios de los identificadores de recurso que se van a trasladar.

```azurecli
webapp=$(az resource show -g OldRG -n ExampleSite --resource-type "Microsoft.Web/sites" --query id --output tsv)
plan=$(az resource show -g OldRG -n ExamplePlan --resource-type "Microsoft.Web/serverfarms" --query id --output tsv)
az resource move --destination-group newgroup --ids $webapp $plan
```

Para mover a una nueva suscripción, proporcione el parámetro `--destination-subscription-id`.

### <a name="by-using-rest-api"></a>Mediante la API REST

Para trasladar recursos existentes a otro grupo de recursos o a una suscripción, ejecute:

```HTTP
POST https://management.azure.com/subscriptions/{source-subscription-id}/resourcegroups/{source-resource-group-name}/moveResources?api-version={api-version}
```

En el cuerpo de la solicitud, especifique el grupo de recursos de destino y los recursos a mover. Para obtener más información acerca de la operación REST de movimiento, consulte [Mover recursos](/rest/api/resources/Resources/MoveResources).

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información acerca de los cmdlets de PowerShell para administrar los recursos, consulte [con Azure PowerShell con Resource Manager](manage-resources-powershell.md).
* Para obtener información acerca de los comandos de CLI de Azure para administrar los recursos, consulte [mediante la CLI de Azure con Resource Manager](manage-resources-cli.md).
* Si desea conocer las funciones del portal que permiten administrar la suscripción, consulte [Uso del Azure Portal para implementar y administrar los recursos de Azure](resource-group-portal.md).
* Para aprender a aplicar una organización lógica a los recursos, consulte [Uso de etiquetas para organizar los recursos de Azure](resource-group-using-tags.md).
