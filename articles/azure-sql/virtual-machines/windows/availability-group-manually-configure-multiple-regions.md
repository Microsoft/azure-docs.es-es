---
title: Configuración de un grupo de disponibilidad AlwaysOn de SQL Server en distintas regiones
description: En este artículo se explica cómo configurar un grupo de disponibilidad AlwaysOn de SQL Server en Azure Virtual Machines con una réplica en una región distinta.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.assetid: 388c464e-a16e-4c9d-a0d5-bb7cf5974689
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/02/2017
ms.author: mathoma
ms.custom: seo-lt-2019
ms.openlocfilehash: e33a5e32848dfd0a6cf252d6876616fc89edd1d7
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284275"
---
# <a name="configure-a-sql-server-always-on-availability-group-across-different-azure-regions"></a>Configuración de un grupo de disponibilidad AlwaysOn de SQL Server en distintas regiones de Azure

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se explica cómo configurar un grupo de disponibilidad de SQL Server AlwaysOn en máquinas virtuales de Azure en una ubicación remota de Azure. Use esta configuración para admitir la recuperación ante desastres.

Este artículo se aplica a Azure Virtual Machines en el modo de Resource Manager.

La siguiente imagen muestra una implementación común de un grupo de disponibilidad en Azure Virtual Machines:

   ![Grupo de disponibilidad](./media/availability-group-manually-configure-multiple-regions/00-availability-group-basic.png)

En esta implementación, todas las máquinas virtuales se encuentran en una única región de Azure. Las réplicas del grupo de disponibilidad pueden tener confirmación sincrónica con conmutación por error automática en SQL-1 y SQL-2. Para crear esta arquitectura, consulte [Plantilla o tutorial de grupo de disponibilidad](availability-group-overview.md).

Esta arquitectura es vulnerable al tiempo de inactividad si la región de Azure deja de estar accesible. Para superar esta vulnerabilidad, agregue una réplica en una región distinta de Azure. El diagrama siguiente muestra el aspecto que tendría la nueva arquitectura:

   ![Recuperación ante desastres de grupo de disponibilidad](./media/availability-group-manually-configure-multiple-regions/00-availability-group-basic-dr.png)

El diagrama anterior muestra una nueva máquina virtual denominada SQL-3. SQL-3 está en una región distinta de Azure. SQL-3 se agrega al clúster de conmutación por error de Windows Server. SQL-3 puede hospedar una réplica del grupo de disponibilidad. Por último, observe que la región de Azure para SQL-3 tiene un nuevo equilibrador de carga de Azure.

>[!NOTE]
> Se requiere un conjunto de disponibilidad de Azure cuando haya más de una máquina virtual en la misma región. Si solo hay una máquina virtual en la región, el conjunto de disponibilidad no es necesario. Solo puede colocar una máquina virtual en un conjunto de disponibilidad en el momento de la creación. Si la máquina virtual ya está en un conjunto de disponibilidad, puede agregar una máquina virtual para una réplica adicional más adelante.

En esta arquitectura, la réplica en la región remota se suele configurar con el modo de disponibilidad de confirmación asincrónica y el modo de conmutación por error manual.

Una vez que las réplicas del grupo de disponibilidad están en Azure Virtual Machines en diferentes regiones de Azure, cada región necesita:

* Una puerta de enlace de red virtual
* Una conexión de una puerta de enlace de red virtual

El diagrama siguiente muestra cómo se comunican las redes entre centros de datos.

   ![Grupo de disponibilidad](./media/availability-group-manually-configure-multiple-regions/01-vpngateway-example.png)

>[!IMPORTANT]
>Esta arquitectura incurre en cargos por datos de salida para datos replicados entre regiones de Azure. Consulte [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/).  

## <a name="create-remote-replica"></a>Creación de réplica remota

Para crear una réplica en un centro de datos remoto, siga estos pasos:

1. [Cree una red virtual en la nueva región](../../../virtual-network/manage-virtual-network.md#create-a-virtual-network).

1. [Configure una conexión de red virtual a red virtual mediante Azure Portal](../../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md).

   >[!NOTE]
   >En algunos casos, tendrá que usar PowerShell para crear la conexión de red virtual a red virtual. Por ejemplo, si utiliza diferentes cuentas de Azure, no puede configurar la conexión en el portal. En este caso, consulte [Configuración de una conexión de red virtual a red virtual mediante Azure Portal](../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md).

1. [Cree un controlador de dominio en la nueva región](../../../active-directory/active-directory-new-forest-virtual-machine.md).

   Este controlador de dominio proporciona autenticación si el controlador de dominio del sitio principal no está disponible.

1. [Cree una máquina virtual de SQL Server en la nueva región](create-sql-vm-portal.md).

1. [Cree un equilibrador de carga de Azure en la red en la nueva región](availability-group-manually-configure-tutorial.md#configure-internal-load-balancer).

   Este equilibrador de carga debe:

   - Estar en la misma red y subred que la nueva máquina virtual.
   - Tener una dirección IP estática para el agente de escucha del grupo de disponibilidad.
   - Incluir un grupo de back-end que conste solo de las máquinas virtuales de la misma región que el equilibrador de carga.
   - Usar un sondeo de puerto TCP específico para la dirección IP.
   - Tener una regla de equilibrio de carga específica de SQL Server en la misma región.  
   - Ser una instancia de Standard Load Balancer si las máquinas virtuales del grupo de back-end no forman parte de un único conjunto de disponibilidad o conjunto de escalado de máquinas virtuales. Para más información, consulte [Introducción a Azure Load Balancer estándar](https://docs.microsoft.com/azure/load-balancer/load-balancer-standard-overview).

1. [Agregue la característica Clústeres de conmutación por error al nuevo servidor SQL Server](availability-group-manually-configure-prerequisites-tutorial.md#add-failover-clustering-features-to-both-sql-server-vms).

1. [Una el nuevo servidor SQL Server al dominio](availability-group-manually-configure-prerequisites-tutorial.md#joinDomain).

1. [Establezca la nueva cuenta de servicio de SQL Server para que utilice una cuenta de dominio](availability-group-manually-configure-prerequisites-tutorial.md#setServiceAccount).

1. [Agregue el nuevo servidor SQL Server al clúster de conmutación por error de Windows Server](availability-group-manually-configure-tutorial.md#addNode).

1. Agregue un recurso de dirección IP al clúster.

   Puede crear el recurso de dirección IP en el Administrador de clústeres de conmutación por error. Seleccione el nombre del clúster y, después, haga clic en él con el botón derecho en **Recursos principales de clúster** y seleccione **Propiedades**: 

   ![Propiedades de clúster](./media/availability-group-manually-configure-multiple-regions/cluster-name-properties.png)

   En el cuadro de diálogo **Propiedades**, seleccione **Agregar** en **Dirección IP** y, después, agregue la dirección IP del nombre de clúster de la región de red remota. Seleccione **Aceptar** en el cuadro de diálogo **Dirección IP** y, después, seleccione de nuevo **Aceptar** en el cuadro de diálogo **Propiedades de clúster** para guardar la nueva dirección IP. 

   ![Adición de una dirección IP al clúster](./media/availability-group-manually-configure-multiple-regions/add-cluster-ip-address.png)


1. Agregue la dirección IP como una dependencia al nombre del clúster principal.

   Abra las propiedades del clúster una vez más y seleccione la pestaña **Dependencias**. Configure una dependencia OR para las dos direcciones IP: 

   ![Propiedades de clúster](./media/availability-group-manually-configure-multiple-regions/cluster-ip-dependencies.png)

1. Agregue un recurso de dirección IP al rol del grupo de disponibilidad en el clúster. 

   Haga clic con el botón derecho en el rol de grupo de disponibilidad en Administrador de clústeres de conmutación por error, elija **Agregar recurso**, **Más recursos** y seleccione **Dirección IP**.

   ![Crear dirección IP](./media/availability-group-manually-configure-multiple-regions/20-add-ip-resource.png)

   Configure esta dirección IP de la siguiente manera:

   - Use la red del centro de datos remoto.
   - Asigne la dirección IP del nuevo equilibrador de carga de Azure. 

1. Agregue el recurso de dirección IP como una dependencia para el clúster (nombre de red) de punto de acceso cliente del agente de escucha.

   La pantalla siguiente muestra un recurso de clúster de dirección IP correctamente configurado:

   ![Grupo de disponibilidad](./media/availability-group-manually-configure-multiple-regions/50-configure-dependency-multiple-ip.png)

   >[!IMPORTANT]
   >El grupo de recursos de clúster incluye ambas direcciones IP. Las dos direcciones IP son dependencias para el punto de acceso cliente del agente de escucha. Use el operador **OR** en la configuración de la dependencia de clúster.

1. [Establecer los parámetros de clúster en PowerShell](availability-group-manually-configure-tutorial.md#setparam).

   Ejecute el script de PowerShell con el nombre de red del clúster, la dirección IP y el puerto de sondeo que configuró en el equilibrador de carga en la nueva región.

   ```powershell
   $ClusterNetworkName = "<MyClusterNetworkName>" # The cluster name for the network in the new region (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name).
   $IPResourceName = "<IPResourceName>" # The cluster name for the new IP Address resource.
   $ILBIP = "<n.n.n.n>" # The IP Address of the Internal Load Balancer (ILB) in the new region. This is the static IP address for the load balancer you configured in the Azure portal.
   [int]$ProbePort = <nnnnn> # The probe port you set on the ILB.

   Import-Module FailoverClusters

   Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"=$ProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
   ```

1. En el nuevo servidor SQL Server, en el Administrador de configuración de SQL Server, [habilite grupos de disponibilidad AlwaysOn](/sql/database-engine/availability-groups/windows/enable-and-disable-always-on-availability-groups-sql-server).

1. [Abra los puertos de firewall en el nuevo servidor SQL](availability-group-manually-configure-prerequisites-tutorial.md#endpoint-firewall).

   Los números de puerto que debe abrir dependerán del entorno. Abra puertos para el punto de conexión de creación de reflejo y el sondeo de estado del equilibrador de carga de Azure.


1. [Agregue una réplica al grupo de disponibilidad en el nuevo servidor SQL Server](/sql/database-engine/availability-groups/windows/use-the-add-replica-to-availability-group-wizard-sql-server-management-studio).

   Para una réplica en una región remota de Azure, establézcala en replicación asincrónica con conmutación por error manual.  

## <a name="set-connection-for-multiple-subnets"></a>Configuración de conexión para varias subredes

La réplica en el centro de datos remoto forma parte del grupo de disponibilidad pero está en una subred diferente. Si esta réplica se convierte en la réplica principal, pueden producirse tiempos de espera de conexión de la aplicación. Este comportamiento es el mismo que el de un grupo de disponibilidad local en una implementación con varias subredes. Para permitir conexiones de aplicaciones de cliente, actualice la conexión de cliente o configure el almacenamiento en caché de la resolución de nombres en el recurso de nombres de red del clúster.

Es preferible actualizar las cadenas de conexión de cliente para establecer `MultiSubnetFailover=Yes`. Consulte [Conectarse a MultiSubnetFailover](https://msdn.microsoft.com/library/gg471494#Anchor_0).

Si no puede modificar las cadenas de conexión, puede configurar el almacenamiento en caché de la resolución de nombres. Consulte [Error de tiempo de espera y no se puede conectar a un agente de escucha de SQL Server 2012 AlwaysOn disponibilidad grupo en un entorno de varias subredes](https://support.microsoft.com/help/2792139/time-out-error-and-you-cannot-connect-to-a-sql-server-2012-alwayson-av).

## <a name="fail-over-to-remote-region"></a>Conmutación por error a una región remota

Para probar la conectividad del agente de escucha con la región remota, puede conmutar por error a la réplica de la región remota. Aunque la réplica sea asincrónica, la conmutación por error es vulnerable a posibles pérdidas de datos. Para conmutar por error sin pérdida de datos, cambie el modo de disponibilidad a sincrónico y establezca el modo de conmutación por error en automático. Siga estos pasos:

1. En el **Explorador de objetos**, conéctese a la instancia de SQL Server que hospeda la réplica principal.
1. En **Grupos de disponibilidad AlwaysOn**, **Grupos de disponibilidad**, haga clic con el botón derecho en el grupo de disponibilidad y seleccione **Propiedades**.
1. En la página **General**, en **Réplicas de disponibilidad**, establezca la réplica secundaria en el sitio de recuperación ante desastres para que utilice el modo de disponibilidad **Confirmación sincrónica** y el modo de conmutación por error **Automático**.
1. Si tiene una réplica secundaria en el mismo sitio que la réplica principal para alta disponibilidad, establezca esta réplica en **Confirmación asincrónica** y **Manual**.
1. Seleccione Aceptar.
1. En el **Explorador de objetos**, haga clic con el botón derecho en el grupo de disponibilidad y luego seleccione **Mostrar panel**.
1. En el panel, compruebe que la réplica en el sitio de recuperación ante desastres esté sincronizada.
1. En el **Explorador de objetos**, haga clic con el botón derecho en el grupo de disponibilidad y luego seleccione **Conmutación por error...** SQL Server Management Studios abre un asistente para conmutar por error a SQL Server.  
1. Seleccione **Siguiente** y la instancia de SQL Server en el sitio de recuperación ante desastres. Seleccione **Siguiente** de nuevo.
1. Conéctese a la instancia de SQL Server en el sitio de recuperación ante desastres y seleccione **Siguiente**.
1. En la página **Resumen**, revise la configuración y seleccione **Finalizar**.

Después de probar la conectividad, mueva la réplica principal de nuevo a su centro de datos principal y vuelva a establecer el modo de disponibilidad en su configuración de funcionamiento normal. En la tabla siguiente se muestra la configuración de funcionamiento normal de la arquitectura descrita en este documento:

| Location | Instancia del servidor | Role | Modo de disponibilidad | Modo de conmutación por error
| ----- | ----- | ----- | ----- | -----
| Centro de datos principal | SQL-1 | Principal | Sincrónica | Automático
| Centro de datos principal | SQL-2 | Secundario | Sincrónica | Automático
| Centro de datos secundario o remoto | SQL-3 | Secundario | Asincrónica | Manual


### <a name="more-information-about-planned-and-forced-manual-failover"></a>Más información acerca de la conmutación por error manual planeada y forzada

Para obtener más información, vea los temas siguientes:

- [Realizar una conmutación por error manual planeada de un grupo de disponibilidad (SQL Server)](https://msdn.microsoft.com/library/hh231018.aspx)
- [Realizar una conmutación por error manual forzada de un grupo de disponibilidad (SQL Server)](https://msdn.microsoft.com/library/ff877957.aspx)

## <a name="next-steps"></a>Pasos siguientes

* [Grupos de disponibilidad AlwaysOn (SQL Server)](https://msdn.microsoft.com/library/hh510230.aspx)
* [Azure Virtual Machines](https://docs.microsoft.com/azure/virtual-machines/windows/)
* [Equilibradores de carga de Azure](availability-group-manually-configure-tutorial.md#configure-internal-load-balancer)
* [Conjuntos de disponibilidad de Azure](../../../virtual-machines/linux/manage-availability.md)
