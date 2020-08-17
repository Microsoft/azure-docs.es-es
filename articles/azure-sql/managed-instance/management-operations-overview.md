---
title: Operaciones de administración
titleSuffix: Azure SQL Managed Instance
description: Más información sobre la duración de las operaciones de administración de Azure SQL Managed Instance y los procedimientos recomendados.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: urosmil
ms.author: urmilano
ms.reviewer: sstein, carlrab, MashaMSFT
ms.date: 07/10/2020
ms.openlocfilehash: 5cff54b1f71d30f7932c4ead722d1dda0a7aec57
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86528197"
---
# <a name="overview-of-azure-sql-managed-instance-management-operations"></a>Introducción a las operaciones de administración de Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

## <a name="what-are-management-operations"></a>¿Qué son las operaciones de administración?
Instancia administrada de SQL proporciona operaciones de administración que puede usar para implementar automáticamente instancias administradas nuevas, actualizar las propiedades de una instancia y eliminar instancias que ya no son necesarias.

Para admitir las [implementaciones dentro de máquinas virtuales de Azure](../../virtual-network/virtual-network-for-azure-services.md) y proporcionar aislamiento y seguridad para los clientes, SQL Managed Instance depende de los [clústeres virtuales](connectivity-architecture-overview.md#high-level-connectivity-architecture). Un clúster virtual representa un conjunto dedicado de máquinas virtuales aisladas implementadas dentro de la subred de la red virtual del cliente. En esencia, cada implementación de instancia administrada en una subred vacía resulta en la creación de un clúster virtual.

Las operaciones subsiguientes en las instancias administradas implementadas pueden tener efecto también en el clúster virtual subyacente. Estas operaciones afectan la duración de las operaciones de administración, debido a que la implementación de las máquinas virtuales adicionales conlleva una sobrecarga que se debe considerar cuando planea implementaciones nuevas o actualizaciones de instancias administradas existentes.

Todas las operaciones de administración se pueden clasificar de la siguiente manera:

- Implementación de una instancia (creación de instancia nueva).
- Actualización de una instancia (cambio de las propiedades de una instancia, como núcleos virtuales o almacenamiento reservado).
- Eliminación de una instancia.

## <a name="management-operations-duration"></a>Duración de las operaciones de administración
Por lo general, las operaciones en clústeres virtuales tardan más. La duración de las operaciones en los clústeres virtuales varía. A continuación, se muestran los valores que puede esperar habitualmente, en función de los datos de telemetría de servicio existentes:

- **Creación de un clúster virtual**:  es un paso sincrónico en las operaciones de administración de una instancia. **El 90 % de las operaciones finaliza en 4 horas**.
- **Cambio de tamaño del clúster virtual (expansión o reducción)** : La expansión es un paso sincrónico, mientras que la reducción se realiza de manera asincrónica (sin afectar la duración de las operaciones de administración de la instancia). **El 90 % de las expansiones de un clúster finaliza en menos de 2,5 horas**.
- **Eliminación de un clúster virtual**: La eliminación es un paso asincrónico, pero también se puede [iniciar manualmente](virtual-cluster-delete.md) en un clúster virtual vacío, en cuyo caso se ejecuta de manera sincrónica. **El 90 % de las eliminaciones de clúster virtual finaliza en 1,5 horas**.

Además, la administración de las instancias también puede incluir una de las operaciones en bases de datos hospedadas, lo que provoca duraciones más largas:

- **Asociación de archivos de base de datos desde Azure Storage**:  un paso sincrónico, como el escalado o la reducción vertical de un proceso (núcleo virtual) o de un almacenamiento en el nivel de servicio De uso general. **El 90 % de estas operaciones finaliza en 5 minutos**.
- **Inicialización de un grupo de disponibilidad AlwaysOn**: un paso sincrónico, como el escalado o la reducción vertical de un proceso (núcleo virtual) o de un almacenamiento en el nivel de servicio Crítico para la empresa, así como en el cambio del nivel de servicio De uso general a Crítico para la empresa (o viceversa). La duración de esta operación es proporcional al tamaño total de la base de datos, así como la actividad de base de datos actual (el número de transacciones activas). La actividad de la base de datos cuando se actualiza una instancia puede introducir una varianza considerable en la duración total. **El 90 % de estas operaciones se ejecuta a 220 GB/hora o más**.

En la tabla siguiente se resumen las operaciones y las duraciones generales típicas:

|Category  |Operación  |Segmento de larga duración  |Duración estimada  |
|---------|---------|---------|---------|
|**Implementación** |Primera instancia en una subred vacía|Creación de un clúster virtual|El 90 % de las operaciones finaliza en 4 horas.|
|Implementación |Primera instancia de otra generación de hardware en una subred no vacía (por ejemplo, primera instancia Gen 5 en una subred con instancias Gen 4)|Creación de un clúster virtual*|El 90 % de las operaciones finaliza en 4 horas.|
|Implementación |Creación de primera instancia de 4 núcleos virtuales en una subred vacía o no vacía|Creación de un clúster virtual**|El 90 % de las operaciones finaliza en 4 horas.|
|Implementación |Creación de instancia subsiguiente dentro de la subred no vacía (segunda instancia, tercera instancia, etc.)|Cambio de tamaño de un clúster virtual|El 90 % de las operaciones finaliza en 2,5 horas.|
|**Actualizar** |Cambio de una propiedad de una instancia (contraseña de administrador, inicio de sesión de Azure AD, marca de Ventaja híbrida de Azure)|N/D|Hasta 1 minuto.|
|Actualizar |Escalado o reducción vertical del almacenamiento de una instancia (nivel de servicio De uso general)|Adjuntar archivos de base de datos|El 90 % de las operaciones finaliza en 5 minutos.|
|Actualizar |Escalado o reducción vertical del almacenamiento de una instancia (nivel de servicio Crítico para la empresa)|- Cambio de tamaño de un clúster virtual<br>- Inicialización de un grupos de disponibilidad AlwaysOn|El 90 % de las operaciones finaliza en 2,5 horas + tiempo para inicializar todas las bases de datos (220 GB/hora).|
|Actualizar |Escalado y reducción vertical del proceso de una instancia (núcleos virtuales) (De uso general)|- Cambio de tamaño de un clúster virtual<br>- Adjuntar archivos de base de datos|El 90 % de las operaciones finaliza en 2,5 horas.|
|Actualizar |Escalado y reducción vertical del proceso de una instancia (núcleos virtuales) (Crítico para la empresa)|- Cambio de tamaño de un clúster virtual<br>- Inicialización de un grupos de disponibilidad AlwaysOn|El 90 % de las operaciones finaliza en 2,5 horas + tiempo para inicializar todas las bases de datos (220 GB/hora).|
|Actualizar |Cambio en el nivel de servicio de una instancia (De uso general a Crítico para la empresa y viceversa)|- Cambio de tamaño de un clúster virtual<br>- Inicialización de un grupos de disponibilidad AlwaysOn|El 90 % de las operaciones finaliza en 2,5 horas + tiempo para inicializar todas las bases de datos (220 GB/hora).|
|**Eliminación**|Eliminación de una instancia|Copia del final del registro para todas las bases de datos|El 90 % de las operaciones finaliza en hasta 1 minuto.<br>Nota: Si se elimina la última instancia de la subred, esta operación programará la eliminación del clúster virtual después de 12 horas.**|
|Eliminación|Eliminación de un clúster virtual (como operación iniciada por el usuario)|Eliminación de un clúster virtual|El 90 % de las operaciones finaliza en hasta 1,5 hora.|

\* El clúster virtual se crea por generación de hardware.

\*\* La configuración actual es de 12 horas, pero eso podría cambiar en el futuro, por lo que no debe depender demasiado de ella. Si necesita eliminar un clúster virtual anterior (por ejemplo, para liberar la subred), consulte [Eliminación de una subred después de eliminar una instancia administrada](virtual-cluster-delete.md).

## <a name="instance-availability-during-management-operations"></a>Disponibilidad de una instancia durante las operaciones de administración

SQL Managed Instance **está disponible durante las operaciones de actualización**, excepto durante un tiempo de inactividad breve provocado por la conmutación por error que se produce al final de la actualización. Normalmente tarda 10 segundos como máximo, incluso en el caso de transacciones de larga duración interrumpidas, gracias a la [recuperación acelerada de bases de datos](../accelerated-database-recovery.md).

> [!IMPORTANT]
> No se recomienda escalar el proceso ni el almacenamiento de Azure SQL Managed Instance ni cambiar el nivel de servicio al mismo tiempo con las transacciones de larga duración (importación de datos, trabajos de procesamiento de datos, recompilación del índice, etc.). La conmutación por error de la base de datos que se realizará al final de la operación cancelará todas las transacciones en curso.

Instancia administrada de SQL no está disponible para las aplicaciones cliente durante las operaciones de implementación y eliminación.

## <a name="management-operations-cross-impact"></a>Impacto de las operaciones de administración

Las operaciones de administración de una instancia administrada pueden afectar a otras operaciones de administración de las instancias colocadas en el mismo clúster virtual:

- Las **operaciones de restauración de larga ejecución** de un clúster virtual pondrán en espera otras operaciones de creación de instancias o de escalado en la misma subred.<br/>**Ejemplo**: si hay una operación de restauración de larga duración y hay una solicitud de creación o escalado en la misma subred, esta solicitud tardará más tiempo en completarse, ya que esperará a que se complete la operación de restauración antes de continuar.
    
- La operación de creación o escalado de instancia que inició el cambio de tamaño del clúster virtual pondrá en espera una operación de **creación o escalado de la instancia posterior**.<br/>**Ejemplo**: si hay varias solicitudes de creación o escalado en la misma subred del mismo clúster virtual y una de ellas inicia el cambio de tamaño de este, todas las solicitudes que se enviaron más de cinco minutos después de la que requería el cambio de tamaño del clúster virtual tardarán más de lo esperado, ya que estas solicitudes tendrán que esperar a que el cambio de tamaño se complete antes de reanudarse.

- **Las operaciones de creación y escalado enviadas en una ventana de 5 minutos** se procesarán por lotes y se ejecutarán en paralelo.<br/>**Ejemplo**: solo se realizará un cambio de tamaño del clúster virtual para todas las operaciones enviadas en la ventana de 5 minutos (medida a partir del momento de la ejecución de la primera solicitud de operación). Si otra solicitud se envía más de 5 minutos después de enviar la primera, tendrá que esperar a que el cambio de tamaño del clúster virtual se complete antes de iniciar la ejecución.

> [!IMPORTANT]
> Las operaciones de administración que se ponen en espera debido a otra operación en curso se reanudarán automáticamente una vez que se cumplan las condiciones para continuar. No se necesita ninguna acción del usuario para reanudar temporalmente las operaciones de administración en pausa.

## <a name="canceling-management-operations"></a>Cancelación de operaciones de administración

En la tabla siguiente se resume la capacidad de cancelar operaciones de administración específicas y las duraciones generales típicas:

Category  |Operación  |Cancelable  |Duración de cancelación estimada  |
|---------|---------|---------|---------|
|Implementación |Creación de instancias |No |  |
|Actualizar |Escalado o reducción vertical del almacenamiento de una instancia (De uso general) |No |  |
|Actualizar |Escalado o reducción vertical del almacenamiento de una instancia (Crítico para la empresa) |Sí |El 90 % de las operaciones finaliza en 5 minutos. |
|Actualizar |Escalado y reducción vertical del proceso de una instancia (núcleos virtuales) (De uso general) |Sí |El 90 % de las operaciones finaliza en 5 minutos. |
|Actualizar |Escalado y reducción vertical del proceso de una instancia (núcleos virtuales) (Crítico para la empresa) |Sí |El 90 % de las operaciones finaliza en 5 minutos. |
|Actualizar |Cambio en el nivel de servicio de una instancia (De uso general a Crítico para la empresa y viceversa) |Sí |El 90 % de las operaciones finaliza en 5 minutos. |
|Eliminar |Eliminación de una instancia |No |  |
|Eliminar |Eliminación de un clúster virtual (como operación iniciada por el usuario) |No |  |

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para cancelar la operación de administración, vaya a la hoja de información general y haga clic en el cuadro de notificación de la operación en curso. En el lado derecho, aparecerá una pantalla con la operación en curso y un botón para cancelar la operación. Después de hacer clic por primera vez, se le pedirá que haga clic de nuevo y confirme que quiere cancelar la operación.

[![Cancelar operación](./media/management-operations-overview/canceling-operation.png)](./media/management-operations-overview/canceling-operation.png#lightbox)

Una vez enviada y procesada una solicitud de cancelación, recibirá una notificación si el envío de la cancelación se realizó correctamente o no.

Si se cancela correctamente la solicitud enviada, la operación de administración se cancelará en un par de minutos, lo que generará un error.

![Resultado de la operación de cancelación](./media/management-operations-overview/canceling-operation-result.png)

Si no se puede cancelar la solicitud o el botón Cancelar no está activo, significa que la operación de administración entró en un estado que no se puede cancelar y que finalizará en un par de minutos. La operación de administración continuará su ejecución hasta que se complete.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Si aún no tiene instalado Azure PowerShell, consulte [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps).

Para cancelar la operación de administración, debe especificar su nombre. Por lo tanto, use primero el comando get para recuperar la lista de operaciones y, a continuación, cancele la operación específica.

```powershell-interactive
$managedInstance = "yourInstanceName"
$resourceGroup = "yourResourceGroupName"

$managementOperations = Get-AzSqlInstanceOperation -ManagedInstanceName $managedInstance  -ResourceGroupName $resourceGroup

foreach ($mo in $managementOperations ) {
    if($mo.State -eq "InProgress" -and $mo.IsCancellable){
        $cancelRequest = Stop-AzSqlInstanceOperation -ResourceGroupName $resourceGroup -ManagedInstanceName $managedInstance -Name $mo.Name
        Get-AzSqlInstanceOperation -ManagedInstanceName $managedInstance  -ResourceGroupName $resourceGroup -Name $mo.Name
    }
}
```

Para obtener una explicación detallada de los comandos, consulte [Get-AzSqlInstanceOperation](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlinstanceoperation) y [Stop-AzSqlInstanceOperation](https://docs.microsoft.com/powershell/module/az.sql/stop-azsqlinstanceoperation).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Si no ha instalado aún la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

Para cancelar la operación de administración, debe especificar su nombre. Por lo tanto, use primero el comando get para recuperar la lista de operaciones y, a continuación, cancele la operación específica.

```azurecli-interactive
az sql mi op list -g yourResourceGroupName --mi yourInstanceName --query "[?state=='InProgress' && isCancellable].{Name: name}" -o tsv |
while read -r operationName; do
    az sql mi op cancel -g yourResourceGroupName --mi yourInstanceName -n $operationName
done
```

Para obtener una explicación detallada de los comandos, consulte [az sql mi op](https://docs.microsoft.com/cli/azure/sql/mi/op?view=azure-cli-latest).

---

## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre cómo crear su primera instancia administrada, vea la [Guía de inicio rápido](instance-create-quickstart.md).
- Para obtener una lista de características y una comparación, consulte [Características comunes de SQL](../database/features-comparison.md).
- Para obtener más información sobre la configuración de redes virtuales, vea [Configuración de una red virtual de Instancia administrada de SQL](connectivity-architecture-overview.md).
- Para ver un inicio rápido en el que se crea una instancia administrada y se restaura una base de datos desde un archivo de copia de seguridad, consulte [Creación de una instancia administrada](instance-create-quickstart.md).
- Para consultar un tutorial sobre el uso de Azure Database Migration Service para la migración, consulte [Migración de Instancia administrada de SQL mediante Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
