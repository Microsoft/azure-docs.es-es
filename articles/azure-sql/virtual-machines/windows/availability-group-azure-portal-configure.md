---
title: Configuración de un grupo de disponibilidad (Azure Portal)
description: Use Azure Portal para crear el clúster de conmutación por error de Windows, la escucha de grupo de disponibilidad y el equilibrador de carga interno en una VM con SQL Server en Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/20/2020
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: 46e2563b0d1c26c984616b523a367c8b2cff7aaa
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89026125"
---
# <a name="configure-an-availability-group-for-sql-server-on-azure-vm-azure-portal---preview"></a>Configuración de un grupo de disponibilidad para SQL Server en máquinas virtuales de Azure (Azure Portal: versión preliminar)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se explica cómo usar [Azure Portal](https://portal.azure.com) para configurar un grupo de disponibilidad para SQL Server en máquinas virtuales de Azure. 

Use Azure Portal para crear un nuevo clúster o incorporar un clúster existente y, luego, cree el grupo de disponibilidad, la escucha y el equilibrador de carga interno. 

   > [!NOTE]
   > Esta característica se encuentra actualmente en versión preliminar y en implementación, de modo que si la región deseada no está disponible, vuelva a comprobarlo pronto. 


## <a name="prerequisites"></a>Requisitos previos

Para configurar un grupo de disponibilidad Always On mediante Azure Portal, debe cumplir los siguientes requisitos previos: 

- Una [suscripción de Azure](https://azure.microsoft.com/free/).
- Un grupo de recursos con un controlador de dominio. 
- Una o varias [máquinas virtuales de Azure con SQL Server 2016 (o posterior) Enterprise Edition](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-server-provision) unidas a dominio en el *mismo* conjunto de disponibilidad o en zonas de disponibilidad *distintas* que se hayan [registrado en el proveedor de recursos de máquina virtual de SQL en modo de administración completa](sql-vm-resource-provider-register.md) y que usen la misma cuenta de dominio para el servicio SQL Server en cada máquina virtual.
- Dos direcciones IP disponibles (no utilizadas por ninguna entidad). Una es para el equilibrador de carga interno. La otra es para el cliente de escucha de grupo de disponibilidad dentro de la misma subred que el grupo de disponibilidad. Si se usa un equilibrador de carga existente, solo se necesita una dirección IP disponible para la escucha de grupo de disponibilidad. 

## <a name="permissions"></a>Permisos

Se necesitan los siguientes permisos de cuenta para configurar el grupo de disponibilidad mediante Azure Portal: 

- Una cuenta de usuario de dominio existente que tenga el permiso **Crear objeto de equipo** en el dominio. Por ejemplo, una cuenta de administrador de dominio normalmente tiene permisos suficientes (como account@domain.com). _Esta cuenta también debe formar parte del grupo de administradores local en cada máquina virtual para crear el clúster._
- La cuenta de usuario del dominio que controla SQL Server. Debe ser la misma cuenta para cada VM con SQL Server que se pretenda agregar al grupo de disponibilidad. 

## <a name="configure-cluster"></a>Configuración del inicio de sesión del clúster

Configure el clúster mediante Azure Portal. Puede crear un nuevo clúster, o bien, si ya tiene uno existente, puede incorporarlo al proveedor de recursos de VM con SQL para poder administrar el portal.


### <a name="create-a-new-cluster"></a>Creación de un clúster

Si ya tiene un clúster, omita esta sección y pase a [Incorporación de un clúster existente](#onboard-existing-cluster) en su lugar. 

Si aún no tiene un clúster, créelo mediante Azure Portal con estos pasos:

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com). 
1. Vaya al recurso [Máquinas virtuales SQL](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines). 
1. Seleccione **Alta disponibilidad** en **Configuración**. 
1. Seleccione **+ New Windows Server failover cluster** (Nuevo clúster de conmutación por error de Windows Server) para abrir la página **Configure Windows Failover cluster** (Configurar clúster de conmutación por error de Windows).  

   :::image type="content" source="media/availability-group-az-portal-configure/create-new-cluster.png" alt-text="Creación de un nuevo clúster mediante la selección de + Nuevo clúster en el portal":::

1. Asigne un nombre al clúster y proporcione una cuenta de almacenamiento para usarla como testigo en la nube. Use una cuenta de almacenamiento existente o seleccione **Crear nueva** para crear una nueva. El nombre de la cuenta de almacenamiento debe tener entre 3 y 24 caracteres, y solo puede contener números y letras minúsculas.

   :::image type="content" source="media/availability-group-az-portal-configure/configure-new-cluster-1.png" alt-text="Asignación de nombre, cuenta de almacenamiento y credenciales al clúster":::

1. Expanda **Windows Server Failover Cluster credentials** (Credenciales del clúster de conmutación por error de Windows Server) para proporcionar las [credenciales](https://docs.microsoft.com/rest/api/sqlvm/sqlvirtualmachinegroups/createorupdate#wsfcdomainprofile) para la cuenta de servicio de SQL Server, así como las cuentas de operador de clúster y de arranque si son diferentes a la cuenta usada para el servicio de SQL Server. 

   :::image type="content" source="media/availability-group-az-portal-configure/configure-new-cluster-2.png" alt-text="Asignación de credenciales para la cuenta de servicio de SQL, la cuenta de operador de clúster y la cuenta de arranque del clúster":::

1. Seleccione las VM con SQL Server que quiere agregar al clúster. Observe si se requiere un reinicio y proceda con precaución. Solo son visibles las máquinas virtuales registradas en el proveedor de recursos de VM con SQL en modo de administración completa y que se encuentran en la misma ubicación, el mismo dominio y la misma red virtual que la VM con SQL Server principal. 
1. Seleccione **Aplicar** para crear el clúster. 

Puede comprobar el estado de la implementación en el **registro de actividad**, que es accesible desde el icono de campana de la barra de navegación superior. 

### <a name="onboard-existing-cluster"></a>Incorporación de un clúster existente

Si ya tiene un clúster configurado en el entorno de VM con SQL Server, puede incorporarlo desde Azure Portal.

Para hacerlo, siga estos pasos:

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com). 
1. Vaya al recurso [Máquinas virtuales SQL](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines). 
1. Seleccione **Alta disponibilidad** en **Configuración**. 
1. Seleccione **Onboard existing Windows Server Failover Cluster** (Incorporar clúster de conmutación por error existente de Windows Server) para abrir la página **Onboard Windows Server Failover Cluster** (Incorporar clúster de conmutación por error de Windows Server). 

   :::image type="content" source="media/availability-group-az-portal-configure/onboard-existing-cluster.png" alt-text="Incorporación de un clúster existente desde la página Alta disponibilidad del recurso de máquinas virtuales de SQL":::

1. Revise la configuración del clúster. 
1. Seleccione **Aplicar** para incorporar el clúster y luego **Sí** en el símbolo del sistema para continuar.


## <a name="create-availability-group"></a>Crear grupo de disponibilidad

Una vez creado o incorporado el clúster, cree el grupo de disponibilidad mediante Azure Portal. Para hacerlo, siga estos pasos:

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com). 
1. Vaya al recurso [Máquinas virtuales SQL](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines). 
1. Seleccione **Alta disponibilidad** en **Configuración**. 
1. Seleccione **+ New Always On availability group** (Nuevo grupo de disponibilidad Always On) para abrir la página **Crear grupo de disponibilidad**.

   :::image type="content" source="media/availability-group-az-portal-configure/create-new-availability-group.png" alt-text="Selección de Nuevo grupo de disponibilidad Always On para abrir la página Crear grupo de disponibilidad.":::

1. Escriba un nombre para el grupo de disponibilidad. 
1. Seleccione **Configure listener** (Configurar escucha) para abrir la página **Configure availability group listener** (Configurar escucha de grupo de disponibilidad). 

   :::image type="content" source="media/availability-group-az-portal-configure/create-availability-group.png" alt-text="Asignación de un nombre para el grupo de disponibilidad y configuración de una escucha":::

1. Rellene los valores y use un equilibrador de carga existente o bien seleccione **Crear nuevo** para crear un nuevo equilibrador de carga.  Seleccione **Aplicar** para guardar la configuración y crear la escucha y el equilibrador de carga. 

   :::image type="content" source="media/availability-group-az-portal-configure/configure-new-listener.png" alt-text="Especificación de los valores del formulario para crear la nueva escucha y el equilibrador de carga":::

1. Seleccione **+ Select replica** (Seleccionar réplica) para abrir la página **Configure availability group replicas** (Configurar réplicas de grupo de disponibilidad).
1. Seleccione las máquinas virtuales que quiere agregar al grupo de disponibilidad y luego la configuración de grupo de disponibilidad que mejor se adapte a sus necesidades empresariales. Seleccione **Aplicar** para guardar la configuración. 

   :::image type="content" source="media/availability-group-az-portal-configure/add-replicas.png" alt-text="Selección de las máquinas virtuales que se van a agregar al grupo de disponibilidad y configuración de las opciones adecuadas para la empresa":::

1. Compruebe la configuración del grupo de disponibilidad y seleccione **Aplicar** para crearlo. 

Puede comprobar el estado de la implementación en el **registro de actividad**, que es accesible desde el icono de campana de la barra de navegación superior. 

  > [!NOTE]
  > **Synchronization health** (Estado de sincronización), en la página **Alta disponibilidad** de Azure Portal, muestra **Incorrecto** hasta que se agregan bases de datos al grupo de disponibilidad. 


## <a name="add-database-to-availability-group"></a>Incorporación de una base de datos al grupo de disponibilidad

Agregue las bases de datos al grupo de disponibilidad una vez finalizada la implementación. En los pasos siguientes se usa SQL Server Management Studio (SSMS), pero también se puede usar [Transact-SQL o PowerShell](/sql/database-engine/availability-groups/windows/availability-group-add-a-database). 

Para agregar bases de datos al grupo de disponibilidad mediante SQL Server Management Studio, siga estos pasos:

1. Conéctese a una de las VM con SQL Server mediante el método que prefiera, como conexión a Escritorio remoto (RDP). 
1. Abra SQL Server Management Studio (SSMS).
1. Conéctese a la instancia de SQL Server. 
1. Expanda **Alta disponibilidad de Always On** en el **Explorador de objetos**.
1. Expanda **Grupos de disponibilidad**, haga clic con el botón derecho en el grupo de disponibilidad y seleccione **Agregar base de datos...**

   :::image type="content" source="media/availability-group-az-portal-configure/add-database.png" alt-text="Clic con el botón derecho en el grupo de disponibilidad en el Explorador de objetos y selección de Agregar base de datos":::

1. Siga los mensajes para seleccionar las bases de datos que quiere agregar al grupo de disponibilidad. 
1. Seleccione **Aceptar** para guardar la configuración y agregar la base de datos al grupo de disponibilidad. 
1. Una vez agregada la base de datos, actualice el **Explorador de objetos** para confirmar el estado de la base de datos como `synchronized`. 

Una vez agregadas las bases de datos, puede comprobar el estado del grupo de disponibilidad en Azure Portal: 

:::image type="content" source="media/availability-group-az-portal-configure/healthy-availability-group.png" alt-text="Comprobación del estado del grupo de disponibilidad en la página Alta disponibilidad de Azure Portal después de sincronizar las bases de datos":::

## <a name="add-more-vms"></a>Incorporación de más máquinas virtuales

Para agregar más VM con SQL Server al clúster, siga estos pasos: 

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com). 
1. Vaya al recurso [Máquinas virtuales SQL](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines). 
1. Seleccione **Alta disponibilidad** en **Configuración**. 
1. Seleccione **Configure Windows Server Failover Cluster** (Configurar clúster de conmutación por error de Windows Server) para abrir la página **Configure Windows Server Failover Cluster** (Configurar clúster de conmutación por error de Windows Server). 

   :::image type="content" source="media/availability-group-az-portal-configure/configure-existing-cluster.png" alt-text="Selección de Configurar clúster de conmutación por error de Windows Server para agregar máquinas virtuales al clúster.":::

1. Expanda **Windows Server Failover Cluster credentials** (Credenciales del clúster de conmutación por error de Windows Server) y escriba las cuentas usadas para el servicio de SQL Server, el operador de clúster y el arranque del clúster. 
1. Seleccione las VM con SQL Server que quiere agregar al clúster. 
1. Seleccione **Aplicar**. 

Puede comprobar el estado de la implementación en el **registro de actividad**, que es accesible desde el icono de campana de la barra de navegación superior. 


## <a name="modify-availability-group"></a>Modificación de un grupo de disponibilidad 


Es posible **Add more replicas** (Agregar más réplicas) al grupo de disponibilidad, **Configure the Listener** (Configurar la escucha) o **Delete the Listener** (Eliminar la escucha) en la página **Alta disponibilidad** de Azure Portal si se seleccionan los puntos suspensivos (...) junto al grupo de disponibilidad: 

:::image type="content" source="media/availability-group-az-portal-configure/configure-listener.png" alt-text="Selección de los puntos suspensivos junto al grupo de disponibilidad y de Agregar réplica para agregar más réplicas al grupo de disponibilidad.":::

## <a name="remove-cluster"></a>Eliminación del clúster

Quite todas las VM con SQL Server del clúster para destruirlo y luego quite los metadatos del clúster del proveedor de recursos de VM con SQL. Para ello, puede usar la versión más reciente de la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) o PowerShell. 

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

En primer lugar, quite todas las VM con SQL Server del clúster. Esto quita físicamente los nodos del clúster y destruye este:  

```azurecli-interactive
# Remove the VM from the cluster metadata
# example: az sql vm remove-from-group --name SQLVM2 --resource-group SQLVM-RG

az sql vm remove-from-group --name <VM1 name>  --resource-group <resource group name>
az sql vm remove-from-group --name <VM2 name>  --resource-group <resource group name>
```

Si estas son las únicas máquinas virtuales del clúster, este se destruye. Si hay otras máquinas virtuales en el clúster aparte de las VM con SQL Server que se han quitado, las otras no se quitan y el clúster no se destruye. 

Luego, quite los metadatos del clúster del proveedor de recursos de VM con SQL: 

```azurecli-interactive
# Remove the cluster from the SQL VM RP metadata
# example: az sql vm group delete --name Cluster --resource-group SQLVM-RG

az sql vm group delete --name <cluster name> Cluster --resource-group <resource group name>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

En primer lugar, quite todas las VM con SQL Server del clúster. Esto quita físicamente los nodos del clúster y destruye este: 

```powershell-interactive
# Remove the SQL VM from the cluster
# example: $sqlvm = Get-AzSqlVM -Name SQLVM3 -ResourceGroupName SQLVM-RG
#  $sqlvm. SqlVirtualMachineGroup = ""
#  Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm

$sqlvm = Get-AzSqlVM -Name <VM Name> -ResourceGroupName <Resource Group Name>
   $sqlvm. SqlVirtualMachineGroup = ""
   
   Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm
```

Si estas son las únicas máquinas virtuales del clúster, este se destruye. Si hay otras máquinas virtuales en el clúster aparte de las VM con SQL Server que se han quitado, las otras no se quitan y el clúster no se destruye. 


Luego, quite los metadatos del clúster del proveedor de recursos de VM con SQL: 

```powershell-interactive
# Remove the cluster metadata
# example: Remove-AzSqlVMGroup -ResourceGroupName "SQLVM-RG" -Name "Cluster"

Remove-AzSqlVMGroup -ResourceGroupName "<resource group name>" -Name "<cluster name> "
```

---

## <a name="troubleshooting"></a>Solución de problemas

Si tiene problemas, puede ver el historial de implementación y revisar los errores comunes y sus soluciones. 

### <a name="check-deployment-history"></a>Visualización del historial de implementación

Los cambios en el clúster y el grupo de disponibilidad por medio del portal se realizan mediante implementaciones. El historial de implementación puede proporcionar más detalles en caso de problemas a la hora de crear, o incorporar el clúster, o de crear el grupo de disponibilidad.

Para ver los registros de la implementación y consultar el historial de implementación, siga estos pasos:

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
1. Vaya al grupo de recursos.
1. Seleccione **Implementaciones** en **Configuración**.
1. Seleccione la implementación que le interese para obtener más información sobre ella. 


   :::image type="content" source="media/availability-group-az-portal-configure/failed-deployment.png" alt-text="Selección de la implementación que interesa para obtener más información." :::

### <a name="common-errors"></a>Errores comunes

Revise los siguientes errores comunes y sus soluciones. 

#### <a name="the-account-which-is-used-to-start-up-sql-service-is-not-a-domain-account"></a>La cuenta que se usa para iniciar el servicio SQL no es una cuenta de dominio

Esto indica que el proveedor de recursos no ha podido acceder al servicio de SQL Server con las credenciales proporcionadas. Algunas soluciones comunes:
- Asegúrese de que el controlador de dominio se está ejecutando.
- Compruebe que las credenciales proporcionadas en el portal coinciden con las del servicio de SQL Server. 


## <a name="next-steps"></a>Pasos siguientes


Para obtener más información sobre grupos de disponibilidad, vea:

- [Información general de los grupos de disponibilidad](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server)
* [Administración de un grupo de disponibilidad](/sql/database-engine/availability-groups/windows/administration-of-an-availability-group-sql-server)   
* [Herramientas para supervisar grupos de disponibilidad Always On &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/monitoring-of-availability-groups-sql-server)
* [Instrucciones Transact-SQL para grupos de disponibilidad](/sql/database-engine/availability-groups/windows/transact-sql-statements-for-always-on-availability-groups)   
* [Cmdlets de PowerShell para grupos de disponibilidad](/sql/database-engine/availability-groups/windows/overview-of-powershell-cmdlets-for-always-on-availability-groups-sql-server)  

Para obtener más información sobre VM con SQL Server, vea: 

* [Introducción a las máquinas virtuales con SQL Server](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Notas de la versión de las máquinas virtuales con SQL Server](../../database/doc-changes-updates-release-notes.md)
* [Preguntas más frecuentes sobre las máquinas virtuales con SQL Server](frequently-asked-questions-faq.md)
