---
title: Alta disponibilidad de varios SID de ASCS/SCS de SAP con WSFC y disco compartido de Azure | Microsoft Docs
description: Alta disponibilidad de varios SID para una instancia de ASCS/SCS de SAP con WSFC y disco compartido de Azure
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: cbf18abe-41cb-44f7-bdec-966f32c89325
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/12/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a4856b2578a007f72aeeec64588ac7f9c58158de
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88860514"
---
# <a name="sap-ascsscs-instance-multi-sid-high-availability-with-windows-server-failover-clustering-and-azure-shared-disk"></a>Alta disponibilidad de varios SID de una instancia de ASCS/SCS de SAP con clústeres de conmutación por error de Windows Server y disco compartido de Azure

> ![SO Windows][Logo_Windows] Windows
>

En este artículo nos centraremos en cómo pasar de una sola instalación de ASCS/SCS a una configuración de varios SID de SAP mediante la instalación de instancias en clúster de ASCS/SCS de SAP adicionales en un clúster de conmutación por error de Windows Server (WSFC) existente con un disco compartido de Azure. Cuando se complete este proceso, habrá configurado un clúster de varios identificadores de seguridad para SAP.

## <a name="prerequisites-and-limitations"></a>Requisitos previos y limitaciones

Actualmente, puede usar discos SSD Premium de Azure como discos compartidos de Azure para la instancia de ASCS/SCS de SAP. Existen las limitaciones siguientes:

-  No se admite el [disco Ultra de Azure](https://docs.microsoft.com/azure/virtual-machines/windows/disks-types#ultra-disk) como disco compartido de Azure para cargas de trabajo de SAP. Actualmente no es posible colocar máquinas virtuales de Azure, con disco Ultra de Azure, en el conjunto de disponibilidad.
-  El [disco compartido de Azure](https://docs.microsoft.com/azure/virtual-machines/windows/disks-shared) con discos SSD Premium solo se admite con máquinas virtuales en el conjunto de disponibilidad. No se admite en la implementación de Availability Zones. 
-  El valor de disco compartido de Azure [maxShares](https://docs.microsoft.com/azure/virtual-machines/windows/disks-shared-enable?tabs=azure-cli#disk-sizes) determina cuántos nodos de clúster pueden usar el disco compartido. Normalmente, para la instancia de ASCS/SCS de SAP, se configuran dos nodos en el clúster de conmutación por error de Windows, por lo que el valor de `maxShares` debe establecerse en dos.
-  Todas las máquinas virtuales de clúster de ASCS/SCS de SAP deben implementarse en el mismo [grupo con ubicación por proximidad de Azure](https://docs.microsoft.com/azure/virtual-machines/windows/proximity-placement-groups).   
   Aunque puede implementar máquinas virtuales de clúster de Windows en un conjunto de disponibilidad con disco compartido de Azure sin PPG, PPG garantizará una proximidad física cercana de los discos compartidos de Azure y las máquinas virtuales del clúster, con lo que se conseguirá una menor latencia entre las máquinas virtuales y la capa de almacenamiento.    

Para más información sobre las limitaciones del disco compartido de Azure, repase minuciosamente la sección [Limitaciones](https://docs.microsoft.com/azure/virtual-machines/linux/disks-shared#limitations) de la documentación del disco compartido de Azure.  

> [!IMPORTANT]
> Al implementar un clúster de conmutación por error de Windows de ASCS/SCS de SAP con disco compartido de Azure, tenga en cuenta que la implementación funcionará con un solo disco compartido en un clúster de almacenamiento. La instancia de ASCS/SCS de SAP se verá afectada, en el caso de problemas con el clúster de almacenamiento, donde se implementa el disco compartido de Azure.  

> [!IMPORTANT]
> Debe cumplir las condiciones siguientes:
> * Cada SID del sistema de administración de bases de datos (DBMS) tiene que tener su propio clúster de WSFC dedicado.  
> * Los servidores de aplicaciones de SAP que pertenecen a un SID del sistema SAP tienen sus propias máquinas virtuales dedicadas.  
> * No se admite una combinación de Enqueue Replication Server 1 y Enqueue Replication Server 2 en el mismo clúster.  


## <a name="supported-os-versions"></a>Versiones de SO admitidas

Se admiten tanto Windows Server 2016 como Windows Server 2019 (use las imágenes más recientes del centro de datos).

Se recomienda encarecidamente usar **Windows Server 2019 Datacenter**, ya que:
- El servicio de clúster de conmutación por error de Windows 2019 reconoce a Azure.
- Se ha agregado la integración y el reconocimiento del mantenimiento del host de Azure y se ha mejorado la experiencia mediante la supervisión de eventos de programación de Azure.
- Es posible usar el nombre de red distribuida (es la opción predeterminada). Por lo tanto, no es necesario tener una dirección IP dedicada para el nombre de red del clúster. Además, no es necesario configurar esta dirección IP en Azure Internal Load Balancer. 

## <a name="architecture"></a>Arquitectura

Tanto Enqueue Replication Server 1 (ERS1) como Enqueue Replication Server 2 (ERS2) se admiten en la configuración de varios SID.  No se admite una combinación de ERS1 y ERS2 en el mismo clúster. 

1. En el primer ejemplo se muestran dos SID de SAP, ambos con la arquitectura ERS1, donde:

   - El SID1 de SAP está implementado en un disco compartido, con ERS1. La instancia de ERS está instalada en el host local y en la unidad local.
     El SID1 de SAP tiene su propia dirección IP (virtual) (SID1 (A)SCS IP1), que está configurada en Azure Internal Load Balancer.

   - El SID2 de SAP está implementado en un disco compartido, con ERS1. La instancia de ERS está instalada en el host local y en la unidad local.
     El SID2 de SAP tiene su propia dirección IP (virtual) (SID2 (A)SCS IP2), que está configurada también en Azure Internal Load Balancer.

![Instancia de ASCS/SCS de SAP de alta disponibilidad: dos instancias con configuración de ERS1][sap-ha-guide-figure-6007]

2. En el segundo ejemplo se muestran dos SID de SAP, ambos con la arquitectura ERS2, donde: 

   - El SID1 de SAP con ERS2, que también es un clúster y está implementado en la unidad local.  
     El SID1 de SAP tiene su propia dirección IP (virtual) (SID1 (A)SCS IP1), que está configurada en Azure Internal Load Balancer.
     El ERS2 de SAP, que el sistema SID1 de SAP utiliza, tiene su propia dirección IP (virtual) (SID1 ERS2 IP2), que está configurada en Azure Internal Load Balancer.

   - El SID2 de SAP con ERS2, que también es un clúster y está implementado en la unidad local.  
     El SID2 de SAP tiene su propia dirección IP (virtual) (SID2 (A)SCS IP3), que está configurada en Azure Internal Load Balancer.
     El ERS2 de SAP, que el sistema SID2 de SAP utiliza, tiene su propia dirección IP (virtual) (SID2 ERS2 IP4), que está configurada en Azure Internal Load Balancer.

   Tenemos un total de cuatro direcciones IP virtuales:  
   - SID1 (A)SCS IP1
   - SID2 ERS2 IP2
   - SID2 (A)SCS IP3
   - SID2 ERS2 IP4

![Instancia de ASCS/SCS de SAP de alta disponibilidad: dos instancias con configuración de ERS1 y ERS2][sap-ha-guide-figure-6008]

## <a name="infrastructure-preparation"></a>Preparación de la infraestructura

Instalaremos un nuevo SID de SAP, **PR2**, además de la instancia de ASCS/SCS de SAP **en clúster existente**, **PR1**.  

### <a name="host-names-and-ip-addresses"></a>Nombres de host y direcciones IP

| Rol de nombre de host | Nombre de host | Dirección IP estática | Conjunto de disponibilidad | Grupo con ubicación por proximidad |
| --- | --- | --- |---| ---|
| Primer clúster ASCS/SCS de nodo de clúster |pr1-ascs-10 |10.0.0.4 |pr1-ascs-avset |PR1PPG |
| Segundo clúster ASCS/SCS de nodo de clúster |pr1-ascs-11 |10.0.0.5 |pr1-ascs-avset |PR1PPG |
| Nombre de red del clúster | pr1clust |10.0.0.42 (**solo** para el clúster de Win 2016) | N/D | N/D |
| Nombre de red del clúster de ASCS de **SID1** | pr1-ascscl |10.0.0.43 | N/D | N/D |
| Nombre de red del clúster de ERS de **SID1** (**solo** para ERS2) | pr1-erscl |10.0.0.44 | N/D | N/D |
| Nombre de red del clúster de ASCS de **SID2** | pr2-ascscl |10.0.0.45 | N/D | N/D |
| Nombre de red del clúster de ERS de **SID2** (**solo** para ERS2) | pr1-erscl |10.0.0.46 | N/D | N/D |

### <a name="create-azure-internal-load-balancer"></a>Creación de una instancia de Azure Internal Load Balancer

ASCS de SAP, SCS de SAP y el nuevo ERS2 de SAP, usan direcciones IP virtuales y un nombre de host virtual. En Azure, se requiere un [equilibrador de carga](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview) para usar una dirección IP virtual. Se recomienda encarecidamente usar [Standard Load Balancer](https://docs.microsoft.com/azure/load-balancer/quickstart-load-balancer-standard-public-portal). 

Tendrá que agregar la configuración al equilibrador de carga existente para la segunda instancia de ASCS/SCS/ERS de SID de SAP, **PR2**. La configuración del primer SID de SAP **PR1** debe estar ya en vigor.  

**PR2 de (A)SCS [número de instancia 02]**
- Configuración de front-end
    - Dirección IP estática de ASCS/SCS **10.0.0.45**
- Configuración de back-end  
    Ya en vigor: las máquinas virtuales ya se han agregado al grupo de back-end, mientras se configura para el SID de SAP **PR1**
- Puerto de sondeo
    - Puerto 620**nr** [**62002**] Deje la opción predeterminada para Protocolo (TCP), Intervalo (5), Umbral incorrecto (2)
- Reglas de equilibrio de carga.
    - Si usa Standard Load Balancer, seleccione Puertos HA
    - Si usa Basic Load Balancer, cree reglas de equilibrio de carga para los puertos siguientes
        - 32**nr** TCP [**3202**]
        - 36**nr** TCP [**3602**]
        - 39**nr** TCP [**3902**]
        - 81**nr** TCP [**8102**]
        - 5**nr**13 TCP [**50213**]
        - 5**nr**14 TCP [**50214**]
        - 5**nr**16 TCP [**50216**]
        - Asocie con la IP de front-end de ASCS **PR2**, el sondeo de estado y el grupo de back-end existente.  

    - Asegúrese de que el tiempo de expiración de inactividad (minutos) esté establecido en el valor máximo de 30 y que la dirección IP flotante (Direct Server Return) esté habilitada.

**PR2 de ERS2 [número de instancia 12]** 

Como Enqueue Replication Server 2 (ERS2) también es un clúster, la dirección IP virtual de ERS2 también debe estar configurada en Azure ILB además de la IP de ASCS/SCS de SAP anterior. Esta sección solo se aplica si se usa la arquitectura de Enqueue Replication Server 2 para **PR2**.  
- Nueva configuración de front-end
    - Dirección IP de ERS2 de SAP estática **10.0.0.46**

- Configuración de back-end  
  Las máquinas virtuales ya se han agregado al grupo de back-end de ILB.  

- Nuevo puerto de sondeo
    - Puerto 621**nr** [**62112**] Deje la opción predeterminada para Protocolo (TCP), Intervalo (5), Umbral incorrecto (2)

- Nuevas reglas de equilibrio de carga
    - Si usa Standard Load Balancer, seleccione Puertos HA
    - Si usa Basic Load Balancer, cree reglas de equilibrio de carga para los puertos siguientes
        - 32**nr** TCP [**3212**]
        - 33**nr** TCP [**3312**]
        - 5**nr**13 TCP [**51212**]
        - 5**nr**14 TCP [**51212**]
        - 5**nr**16 TCP [**51212**]
        - Asocie con la IP de front-end de ERS2 **PR2**, el sondeo de estado y el grupo de back-end existente.  

    - Asegúrese de que el tiempo de expiración de inactividad (minutos) esté establecido en el valor máximo de 30, por ejemplo, y que la dirección IP flotante (Direct Server Return) esté habilitada.


### <a name="create-and-attach-second-azure-shared-disk"></a>Creación y conexión de un segundo disco compartido de Azure

Ejecute este comando en uno de los nodos del clúster. Tendrá que ajustar los valores para el grupo de recursos, la región de Azure, SAPSID, etc.  

```powershell
    $ResourceGroupName = "MyResourceGroup"
    $location = "MyRegion"
    $SAPSID = "PR2"
    $DiskSizeInGB = 512
    $DiskName = "$($SAPSID)ASCSSharedDisk"
    $NumberOfWindowsClusterNodes = 2
    $diskConfig = New-AzDiskConfig -Location $location -SkuName Premium_LRS  -CreateOption Empty  -DiskSizeGB $DiskSizeInGB -MaxSharesCount $NumberOfWindowsClusterNodes
    
    $dataDisk = New-AzDisk -ResourceGroupName $ResourceGroupName -DiskName $DiskName -Disk $diskConfig
    ##################################
    ## Attach the disk to cluster VMs
    ##################################
    # ASCS Cluster VM1
    $ASCSClusterVM1 = "pr1-ascs-10"
    # ASCS Cluster VM2
    $ASCSClusterVM2 = "pr1-ascs-11"
    # next free LUN number
    $LUNNumber = 1
    # Add the Azure Shared Disk to Cluster Node 1
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM1 
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun $LUNNumber
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
    # Add the Azure Shared Disk to Cluster Node 2
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM2
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun $LUNNumber
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
   ```
### <a name="format-the-shared-disk-with-powershell"></a>Formato del disco compartido con PowerShell
1. Obtenga el número de disco. Ejecute los comandos de PowerShell en uno de los nodos del clúster:

   ```powershell
    Get-Disk | Where-Object PartitionStyle -Eq "RAW"  | Format-Table -AutoSize 
    # Example output
    # Number Friendly Name     Serial Number HealthStatus OperationalStatus Total Size Partition Style
    # ------ -------------     ------------- ------------ ----------------- ---------- ---------------
    # 3      Msft Virtual Disk               Healthy      Online                512 GB RAW            

   ```
2. Formatee el disco. En este ejemplo, el número de disco es 3. 

   ```powershell
    # Format SAP ASCS Disk number '3', with drive letter 'S'
    $SAPSID = "PR2"
    $DiskNumber = 3
    $DriveLetter = "S"
    $DiskLabel = "$SAPSID" + "SAP"
    
    Get-Disk -Number $DiskNumber | Where-Object PartitionStyle -Eq "RAW" | Initialize-Disk -PartitionStyle GPT -PassThru |  New-Partition -DriveLetter $DriveLetter -UseMaximumSize | Format-Volume  -FileSystem ReFS -NewFileSystemLabel $DiskLabel -Force -Verbose
    # Example outout
    # DriveLetter FileSystemLabel FileSystem DriveType HealthStatus OperationalStatus SizeRemaining      Size
    # ----------- --------------- ---------- --------- ------------ ----------------- -------------      ----
    # S           PR2SAP          ReFS       Fixed     Healthy      OK                    504.98 GB 511.81 GB
   ```

3. Compruebe que el disco esté ahora visible como disco de clúster.  
   ```powershell
    # List all disks
    Get-ClusterAvailableDisk -All
    # Example output
    # Cluster    : pr1clust
    # Id         : c469b5ad-d089-4d8f-ae4c-d834cbbde1a2
    # Name       : Cluster Disk 2
    # Number     : 3
    # Size       : 549755813888
    # Partitions : {\\?\GLOBALROOT\Device\Harddisk3\Partition2\}
   ```

4. Registre el disco en el clúster.  
   ```powershell
    # Add the disk to cluster 
    Get-ClusterAvailableDisk -All | Add-ClusterDisk
    # Example output     
    # Name           State  OwnerGroup        ResourceType 
    # ----           -----  ----------        ------------ 
    # Cluster Disk 2 Online Available Storage Physical Disk
   ```

## <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance"></a>Creación de un nombre de host virtual para la instancia de SAP ASCS/SCS en clúster

1. Cree una entrada de DNS para el nombre de host virtual de la instancia de ASCS/SCS de SAP en el Administrador de DNS de Windows.  
   La dirección IP que asigna al nombre de host virtual en DNS debe ser la misma dirección IP que asignó en Azure Load Balancer.  

   ![_Definición de la entrada DNS del nombre virtual del clúster de ASCS/SCS de SAP y la dirección IP][sap-ha-guide-figure-6009]
 
   _Definición de la entrada DNS del nombre virtual del clúster de ASCS/SCS de SAP y la dirección IP_

2. Si usa el nuevo Enqueue Replication Server 2 de SAP, que también es una instancia en clúster, también debe reservar en DNS un nombre de host virtual para ERS2. 
   La dirección IP que asigna al nombre de host virtual para ERS2 en DNS debe ser la misma dirección IP que asignó en Azure Load Balancer.  

   ![_Definición de la entrada DNS del nombre virtual del clúster de ERS2 de SAP y la dirección IP][sap-ha-guide-figure-6010]
 
   _Definición de la entrada DNS del nombre virtual del clúster de ERS2 de SAP y la dirección IP_

3. Para definir la dirección IP asignada al nombre de host virtual, seleccione **Administrador de DNS** > **Dominio**.

   ![Nuevo nombre virtual y dirección IP para la configuración del clúster de ASCS/SCS y ERS2 de SAP][sap-ha-guide-figure-6011]

   _Nuevo nombre virtual y dirección IP para la configuración del clúster de ASCS/SCS y ERS2 de SAP_

## <a name="sap-installation"></a>Instalación de SAP 

### <a name="install-the-sap-first-cluster-node"></a> Instalación del primer nodo de clúster de SAP

Siga el procedimiento de instalación descrito de SAP. Asegúrese de que, en la opción de inicio de la instalación "First Cluster Node" (Primer nodo de clúster), elige "Cluster Shared Disk" (Disco compartido de clúster) como opción de configuración.  
Elija el disco compartido que acaba de crear.  

### <a name="modify-the-sap-profile-of-the-ascsscs-instance"></a> Modificación del perfil SAP de la instancia de ASCS/SCS

Si ejecuta Enqueue Replication Server 1, agregue el parámetro de perfil de SAP `enque/encni/set_so_keepalive`, como se describe a continuación. El parámetro de perfil evita el cierre de las conexiones entre los procesos de trabajo de SAP y el servidor de puesta en cola cuando lleven inactivas demasiado tiempo. El parámetro de SAP no es necesario para ERS2. 

1. Agregue este parámetro de perfil al perfil de la instancia de ASCS/SCS de SAP, si usa ERS1.

   ```
   enque/encni/set_so_keepalive = true
   ```
   
   En el caso de ERS1 y ERS2, asegúrese de que los parámetros del sistema operativo `keepalive` se establecen tal y como se describe en la nota de SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).   

2. Para aplicar los cambios de parámetros de perfil de SAP, reinicie la instancia de ASCS/SCS de SAP.

### <a name="configure-probe-port-on-the-cluster-resource"></a>Configuración del puerto de sondeo en el recurso de clúster

Use la funcionalidad de sondeo del equilibrador de carga interno para que toda la configuración del clúster funcione con Azure Load Balancer. Habitualmente, el equilibrador de carga interno de Azure distribuye la carga de trabajo entrante de manera equitativa entre las máquinas virtuales que participan.

Sin embargo, esto no funcionará en algunas configuraciones de clúster porque solo hay una instancia activa. La otra instancia es pasiva y no puede aceptar nada de la carga de trabajo. Una funcionalidad de sondeo ayuda cuando Azure Internal Load Balancer detecta qué instancia está activa y solo tiene como destino la instancia activa.  

> [!IMPORTANT]
> En esta configuración de ejemplo, **ProbePort** se establece en 620**Nr**. En el caso de la instancia de ASCS de SAP con el número **02** es 620**02**.
> Tendrá que ajustar la configuración para que coincida con los números de instancia de SAP y el SID de SAP.

Para agregar un puerto de sondeo, ejecute este módulo de PowerShell en una de las máquinas virtuales del clúster:

- En el caso de la instancia de ASC/SCS de SAP con el número de instancia **02**. 
   ```powershell
   Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID PR2 -ProbePort 62002
   ```

- Si usa ERS2, con el número de instancia **12**, que está en clúster. No es necesario configurar el puerto de sondeo para ERS1, ya que no está en clúster.  
   ```powershell
   Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID PR2 -ProbePort 62012 -IsSAPERSClusteredInstance $True
   ```

 El código de la función `Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource` tendría el siguiente aspecto:
   ```powershell
    function Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource {
    <#
    .SYNOPSIS 
    Set-AzureLoadBalancerHealthProbePortOnSAPClusterIPResource will set a new Azure Load Balancer Health Probe Port on 'SAP $SAPSID IP' cluster resource.
    
    .DESCRIPTION
    Set-AzureLoadBalancerHealthProbePortOnSAPClusterIPResource will set a new Azure Load Balancer Health Probe Port on 'SAP $SAPSID IP' cluster resource.
    It will also restart SAP Cluster group (default behavior), to activate the changes. 
    
    You need to run it on one of the SAP ASCS/SCS Windows cluster nodes.
    
    Expectation is that SAP group is installed with official SWPM installation tool, which will set default expected naming convention for:
    - SAP Cluster Group:               'SAP $SAPSID'
    - SAP Cluster IP Address Resource: 'SAP $SAPSID IP' 
    
    .PARAMETER SAPSID 
    SAP SID - 3 characters staring with letter.
    
    .PARAMETER ProbePort 
    Azure Load Balancer Health Check Probe Port.
    
    .PARAMETER RestartSAPClusterGroup 
    Optional parameter. Default value is '$True', so SAP cluster group will be restarted to activate the changes.
    
    .PARAMETER IsSAPERSClusteredInstance 
    Optional parameter.Default value is '$False'.
    If set to $True , then handle clsutered new SAP ERS2 instance.
    
    
    .EXAMPLE 
    # Set probe port to 62000, on SAP cluster resource 'SAP AB1 IP', and restart the SAP cluster group 'SAP AB1', to activate the changes.
    Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID AB1 -ProbePort 62000 
    
    .EXAMPLE 
    # Set probe port to 62000, on SAP cluster resource 'SAP AB1 IP'. SAP cluster group 'SAP AB1' IS NOT restarted, therefore changes are NOT active.
    # To activate the changes you need to manualy restart 'SAP AB1' cluster group.
    Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID AB1 -ProbePort 62000 -RestartSAPClusterGroup $False
    
    .EXAMPLE 
    # Set probe port to 62001, on SAP cluster resource 'SAP AB1 ERS IP'. SAP cluster group 'SAP AB1 ERS' IS restarted, to activate the changes.
    Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID AB1 -ProbePort 62000 -IsSAPERSClusteredInstance $True
        
    #> 
    
        [CmdletBinding()]
        param(
            
            [Parameter(Mandatory=$True)]
            [ValidateNotNullOrEmpty()]  
            [ValidateLength(3,3)]      
            [string]$SAPSID,
                  
            [Parameter(Mandatory=$True)]
            [ValidateNotNullOrEmpty()]        
            [int] $ProbePort,
    
            [Parameter(Mandatory=$False)] 
            [bool] $RestartSAPClusterGroup = $True,
    
            [Parameter(Mandatory=$False)] 
            [bool] $IsSAPERSClusteredInstance = $False
        
        )
    
        BEGIN{}
        
        PROCESS{
            try{                                      
                
                if($IsSAPERSClusteredInstance){
                    #Handle clustered SAP ERS Instance
                    $SAPClusterRoleName = "SAP $SAPSID ERS"
                    $SAPIPresourceName = "SAP $SAPSID ERS IP"            
                }else{
                    #Handle clustered SAP ASCS/SCS Instance
                    $SAPClusterRoleName = "SAP $SAPSID"
                    $SAPIPresourceName = "SAP $SAPSID IP"
                }

                $SAPIPResourceClusterParameters =  Get-ClusterResource $SAPIPresourceName | Get-ClusterParameter
                $IPAddress = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Address" }).Value
                $NetworkName = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Network" }).Value
                $SubnetMask = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "SubnetMask" }).Value
                $OverrideAddressMatch = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "OverrideAddressMatch" }).Value
                $EnableDhcp = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "EnableDhcp" }).Value
                $OldProbePort = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "ProbePort" }).Value
    
                $var = Get-ClusterResource | Where-Object {  $_.name -eq $SAPIPresourceName  }
    
                #Write-Host "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" -ForegroundColor Cyan
                Write-Output "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" 
    
                Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter
    
                Write-Output " "
                Write-Output "Current probe port property of the SAP cluster resource '$SAPIPresourceName' is '$OldProbePort'." 
                Write-Output " "
                Write-Output "Setting the new probe port property of the SAP cluster resource '$SAPIPresourceName' to '$ProbePort' ..." 
                Write-Output " "
    
                $var | Set-ClusterParameter -Multiple @{"Address"=$IPAddress;"ProbePort"=$ProbePort;"Subnetmask"=$SubnetMask;"Network"=$NetworkName;"OverrideAddressMatch"=$OverrideAddressMatch;"EnableDhcp"=$EnableDhcp}
    
                Write-Output " "
    
                #$ActivateChanges = Read-Host "Do you want to take restart SAP cluster role '$SAPClusterRoleName', to activate the changes (yes/no)?"
    
                if($RestartSAPClusterGroup){
                    Write-Output ""
                    Write-Output "Activating changes..." 
    
                    Write-Output " "
                    Write-Output "Taking SAP cluster IP resource '$SAPIPresourceName' offline ..."
                    Stop-ClusterResource -Name $SAPIPresourceName
                    sleep 5
    
                    Write-Output "Starting SAP cluster role '$SAPClusterRoleName' ..."
                    Start-ClusterGroup -Name $SAPClusterRoleName
    
                    Write-Output "New ProbePort parameter is active." 
                    Write-Output " "
    
                    Write-Output "New configuration parameters for SAP IP cluster resource '$SAPIPresourceName':" 
                    Write-Output " " 
                    Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter
                }else
                {
                    Write-Output "SAP cluster role '$SAPClusterRoleName' is not restarted, therefore changes are not activated."
                }
            }
            catch{
               Write-Error  $_.Exception.Message
           }
    
        }
    
        END {}
    }
   ```

### <a name="continue-with-the-sap-installation"></a>Continuación de la instalación de SAP

1. Para instalar la instancia de base de datos, siga el proceso que se describe en la guía de instalación de SAP.  
2. Para instalar SAP en el segundo nodo de clúster, siga los pasos que se describen en la guía de instalación de SAP.  
3. Instale la instancia del servidor de aplicaciones principal (PAS) de SAP en la máquina virtual que designó para hospedar el PAS.   
   Siga el proceso descrito en la guía de instalación de SAP. No hay ninguna dependencia en Azure.
4. Instale servidores de aplicaciones adicionales de SAP en las máquinas virtuales, designadas para hospedar instancias de servidores de aplicaciones de SAP.  
   Siga el proceso descrito en la guía de instalación de SAP. No hay ninguna dependencia en Azure. 

## <a name="test-the-sap-ascsscs-instance-failover"></a>Prueba de la conmutación por error de la instancia de ASCS/SCS de SAP
En el caso de las pruebas de conmutación por error descritas, asumimos que ASCS de SAP está activo en el nodo A.  

1. Compruebe que el sistema SAP pueda realizar correctamente la conmutación por error del nodo A al nodo B. En este ejemplo, la prueba se realiza para el SAPSID **PR2**.  
   Asegúrese de que cada SAPSID pueda moverse correctamente al otro nodo del clúster.   
   Elija una de estas opciones para iniciar una conmutación por error del grupo de clústeres \<SID\> de SAP desde el nodo del clúster A al B:
    - Administrador de clústeres de conmutación por error  
    - PowerShell del clúster de conmutación por error

    ```powershell
    $SAPSID = "PR2"     # SAP <SID>
 
    $SAPClusterGroup = "SAP $SAPSID"
    Move-ClusterGroup -Name $SAPClusterGroup

    ```
2. Reinicie el nodo del clúster A dentro del sistema operativo invitado Windows. Al hacerlo, se inicia una conmutación automática por error del grupo de clústeres \<SID\> de SAP del nodo A al B.  
3. Reinicie el nodo del clúster A desde Azure Portal. Al hacerlo, se inicia una conmutación automática por error del grupo de clústeres \<SID\> de SAP del nodo A al B.  
4. Reinicie el nodo del clúster A mediante Azure PowerShell. Al hacerlo, se inicia una conmutación automática por error del grupo de clústeres \<SID\> de SAP del nodo A al B.

## <a name="next-steps"></a>Pasos siguientes

* [Preparación de la infraestructura de Azure para alta disponibilidad de SAP con un clúster de conmutación por error de Windows y un disco compartido para una instancia de ASCS/SCS de SAP][sap-high-availability-infrastructure-wsfc-shared-disk]
* [Instalación de alta disponibilidad para SAP NetWeaver en un clúster de conmutación por error de Windows y un disco compartido para una instancia de ASCS/SCS de SAP][sap-high-availability-installation-wsfc-shared-disk]


[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1869038]:https://launchpad.support.sap.com/#/notes/1869038
[2287140]:https://launchpad.support.sap.com/#/notes/2287140
[2492395]:https://launchpad.support.sap.com/#/notes/2492395

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[networking-limits-azure-resource-manager]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits
[load-balancer-multivip-overview]:../../../load-balancer/load-balancer-multivip-overview.md


[sap-net-weaver-ports]:https://help.sap.com/viewer/ports
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md
[sap-net-weaver-ports-ascs-scs-ports]:sap-high-availability-infrastructure-wsfc-shared-disk.md#fe0bd8b5-2b43-45e3-8295-80bee5415716

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[ha-guide]:sap-high-availability-guide.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-file-share]:sap-high-availability-infrastructure-wsfc-file-share.md

[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk-install-ascs]:sap-high-availability-installation-wsfc-shared-disk.md#31c6bd4f-51df-4057-9fdf-3fcbc619c170
[sap-high-availability-installation-wsfc-shared-disk-modify-ascs-profile]:sap-high-availability-installation-wsfc-shared-disk.md#e4caaab2-e90f-4f2c-bc84-2cd2e12a9556
[sap-high-availability-installation-wsfc-shared-disk-add-probe-port]:sap-high-availability-installation-wsfc-shared-disk.md#10822f4f-32e7-4871-b63a-9b86c76ce761
[sap-high-availability-installation-wsfc-shared-disk-win-firewall-probe-port]:sap-high-availability-installation-wsfc-shared-disk.md#4498c707-86c0-4cde-9c69-058a7ab8c3ac
[sap-high-availability-installation-wsfc-shared-disk-change-ers-service-startup-type]:sap-high-availability-installation-wsfc-shared-disk.md#094bc895-31d4-4471-91cc-1513b64e406a
[sap-high-availability-installation-wsfc-shared-disk-test-ascs-failover-and-sios-repl]:sap-high-availability-installation-wsfc-shared-disk.md#18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9


[sap-high-availability-installation-wsfc-file-share]:sap-high-availability-installation-wsfc-file-share.md
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5c8e5482-841e-45e1-a89d-a05c0907c868

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

[sap-ha-guide-figure-1000]:./media/virtual-machines-shared-sap-high-availability-guide/1000-wsfc-for-sap-ascs-on-azure.png
[sap-ha-guide-figure-1001]:./media/virtual-machines-shared-sap-high-availability-guide/1001-wsfc-on-azure-ilb.png
[sap-ha-guide-figure-1002]:./media/virtual-machines-shared-sap-high-availability-guide/1002-wsfc-sios-on-azure-ilb.png
[sap-ha-guide-figure-2000]:./media/virtual-machines-shared-sap-high-availability-guide/2000-wsfc-sap-as-ha-on-azure.png
[sap-ha-guide-figure-2001]:./media/virtual-machines-shared-sap-high-availability-guide/2001-wsfc-sap-ascs-ha-on-azure.png
[sap-ha-guide-figure-2003]:./media/virtual-machines-shared-sap-high-availability-guide/2003-wsfc-sap-dbms-ha-on-azure.png
[sap-ha-guide-figure-2004]:./media/virtual-machines-shared-sap-high-availability-guide/2004-wsfc-sap-ha-e2e-archit-template1-on-azure.png
[sap-ha-guide-figure-2005]:./media/virtual-machines-shared-sap-high-availability-guide/2005-wsfc-sap-ha-e2e-arch-template2-on-azure.png

[sap-ha-guide-figure-3000]:./media/virtual-machines-shared-sap-high-availability-guide/3000-template-parameters-sap-ha-arm-on-azure.png
[sap-ha-guide-figure-3001]:./media/virtual-machines-shared-sap-high-availability-guide/3001-configuring-dns-servers-for-Azure-vnet.png
[sap-ha-guide-figure-3002]:./media/virtual-machines-shared-sap-high-availability-guide/3002-configuring-static-IP-address-for-network-card-of-each-vm.png
[sap-ha-guide-figure-3003]:./media/virtual-machines-shared-sap-high-availability-guide/3003-setup-static-ip-address-ilb-for-ascs-instance.png
[sap-ha-guide-figure-3004]:./media/virtual-machines-shared-sap-high-availability-guide/3004-default-ascs-scs-ilb-balancing-rules-for-azure-ilb.png
[sap-ha-guide-figure-3005]:./media/virtual-machines-shared-sap-high-availability-guide/3005-changing-ascs-scs-default-ilb-rules-for-azure-ilb.png
[sap-ha-guide-figure-3006]:./media/virtual-machines-shared-sap-high-availability-guide/3006-adding-vm-to-domain.png
[sap-ha-guide-figure-3007]:./media/virtual-machines-shared-sap-high-availability-guide/3007-config-wsfc-1.png
[sap-ha-guide-figure-3008]:./media/virtual-machines-shared-sap-high-availability-guide/3008-config-wsfc-2.png
[sap-ha-guide-figure-3009]:./media/virtual-machines-shared-sap-high-availability-guide/3009-config-wsfc-3.png
[sap-ha-guide-figure-3010]:./media/virtual-machines-shared-sap-high-availability-guide/3010-config-wsfc-4.png
[sap-ha-guide-figure-3011]:./media/virtual-machines-shared-sap-high-availability-guide/3011-config-wsfc-5.png
[sap-ha-guide-figure-3012]:./media/virtual-machines-shared-sap-high-availability-guide/3012-config-wsfc-6.png
[sap-ha-guide-figure-3013]:./media/virtual-machines-shared-sap-high-availability-guide/3013-config-wsfc-7.png
[sap-ha-guide-figure-3014]:./media/virtual-machines-shared-sap-high-availability-guide/3014-config-wsfc-8.png
[sap-ha-guide-figure-3015]:./media/virtual-machines-shared-sap-high-availability-guide/3015-config-wsfc-9.png
[sap-ha-guide-figure-3016]:./media/virtual-machines-shared-sap-high-availability-guide/3016-config-wsfc-10.png
[sap-ha-guide-figure-3017]:./media/virtual-machines-shared-sap-high-availability-guide/3017-config-wsfc-11.png
[sap-ha-guide-figure-3018]:./media/virtual-machines-shared-sap-high-availability-guide/3018-config-wsfc-12.png
[sap-ha-guide-figure-3019]:./media/virtual-machines-shared-sap-high-availability-guide/3019-assign-permissions-on-share-for-cluster-name-object.png
[sap-ha-guide-figure-3020]:./media/virtual-machines-shared-sap-high-availability-guide/3020-change-object-type-include-computer-objects.png
[sap-ha-guide-figure-3021]:./media/virtual-machines-shared-sap-high-availability-guide/3021-check-box-for-computer-objects.png
[sap-ha-guide-figure-3022]:./media/virtual-machines-shared-sap-high-availability-guide/3022-set-security-attributes-for-cluster-name-object-on-file-share-quorum.png
[sap-ha-guide-figure-3023]:./media/virtual-machines-shared-sap-high-availability-guide/3023-call-configure-cluster-quorum-setting-wizard.png
[sap-ha-guide-figure-3024]:./media/virtual-machines-shared-sap-high-availability-guide/3024-selection-screen-different-quorum-configurations.png
[sap-ha-guide-figure-3025]:./media/virtual-machines-shared-sap-high-availability-guide/3025-selection-screen-file-share-witness.png
[sap-ha-guide-figure-3026]:./media/virtual-machines-shared-sap-high-availability-guide/3026-define-file-share-location-for-witness-share.png
[sap-ha-guide-figure-3027]:./media/virtual-machines-shared-sap-high-availability-guide/3027-successful-reconfiguration-cluster-file-share-witness.png
[sap-ha-guide-figure-3028]:./media/virtual-machines-shared-sap-high-availability-guide/3028-install-dot-net-framework-35.png
[sap-ha-guide-figure-3029]:./media/virtual-machines-shared-sap-high-availability-guide/3029-install-dot-net-framework-35-progress.png
[sap-ha-guide-figure-3030]:./media/virtual-machines-shared-sap-high-availability-guide/3030-sios-installer.png
[sap-ha-guide-figure-3031]:./media/virtual-machines-shared-sap-high-availability-guide/3031-first-screen-sios-data-keeper-installation.png
[sap-ha-guide-figure-3032]:./media/virtual-machines-shared-sap-high-availability-guide/3032-data-keeper-informs-service-be-disabled.png
[sap-ha-guide-figure-3033]:./media/virtual-machines-shared-sap-high-availability-guide/3033-user-selection-sios-data-keeper.png
[sap-ha-guide-figure-3034]:./media/virtual-machines-shared-sap-high-availability-guide/3034-domain-user-sios-data-keeper.png
[sap-ha-guide-figure-3035]:./media/virtual-machines-shared-sap-high-availability-guide/3035-provide-sios-data-keeper-license.png
[sap-ha-guide-figure-3036]:./media/virtual-machines-shared-sap-high-availability-guide/3036-data-keeper-management-config-tool.png
[sap-ha-guide-figure-3037]:./media/virtual-machines-shared-sap-high-availability-guide/3037-tcp-ip-address-first-node-data-keeper.png
[sap-ha-guide-figure-3038]:./media/virtual-machines-shared-sap-high-availability-guide/3038-create-replication-sios-job.png
[sap-ha-guide-figure-3039]:./media/virtual-machines-shared-sap-high-availability-guide/3039-define-sios-replication-job-name.png
[sap-ha-guide-figure-3040]:./media/virtual-machines-shared-sap-high-availability-guide/3040-define-sios-source-node.png
[sap-ha-guide-figure-3041]:./media/virtual-machines-shared-sap-high-availability-guide/3041-define-sios-target-node.png
[sap-ha-guide-figure-3042]:./media/virtual-machines-shared-sap-high-availability-guide/3042-define-sios-synchronous-replication.png
[sap-ha-guide-figure-3043]:./media/virtual-machines-shared-sap-high-availability-guide/3043-enable-sios-replicated-volume-as-cluster-volume.png
[sap-ha-guide-figure-3044]:./media/virtual-machines-shared-sap-high-availability-guide/3044-data-keeper-synchronous-mirroring-for-SAP-gui.png
[sap-ha-guide-figure-3045]:./media/virtual-machines-shared-sap-high-availability-guide/3045-replicated-disk-by-data-keeper-in-wsfc.png
[sap-ha-guide-figure-3046]:./media/virtual-machines-shared-sap-high-availability-guide/3046-dns-entry-sap-ascs-virtual-name-ip.png
[sap-ha-guide-figure-3047]:./media/virtual-machines-shared-sap-high-availability-guide/3047-dns-manager.png
[sap-ha-guide-figure-3048]:./media/virtual-machines-shared-sap-high-availability-guide/3048-default-cluster-probe-port.png
[sap-ha-guide-figure-3049]:./media/virtual-machines-shared-sap-high-availability-guide/3049-cluster-probe-port-after.png
[sap-ha-guide-figure-3050]:./media/virtual-machines-shared-sap-high-availability-guide/3050-service-type-ers-delayed-automatic.png
[sap-ha-guide-figure-5000]:./media/virtual-machines-shared-sap-high-availability-guide/5000-wsfc-sap-sid-node-a.png
[sap-ha-guide-figure-5001]:./media/virtual-machines-shared-sap-high-availability-guide/5001-sios-replicating-local-volume.png
[sap-ha-guide-figure-5002]:./media/virtual-machines-shared-sap-high-availability-guide/5002-wsfc-sap-sid-node-b.png
[sap-ha-guide-figure-5003]:./media/virtual-machines-shared-sap-high-availability-guide/5003-sios-replicating-local-volume-b-to-a.png

[sap-ha-guide-figure-6001]:media/virtual-machines-shared-sap-high-availability-guide/6001-sap-multi-sid-ascs-scs-sid1.png
[sap-ha-guide-figure-6002]:media/virtual-machines-shared-sap-high-availability-guide/6002-sap-multi-sid-ascs-scs.png
[sap-ha-guide-figure-6003]:media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png
[sap-ha-guide-figure-6004]:media/virtual-machines-shared-sap-high-availability-guide/6004-sap-multi-sid-dns.png
[sap-ha-guide-figure-6005]:media/virtual-machines-shared-sap-high-availability-guide/6005-sap-multi-sid-azure-portal.png
[sap-ha-guide-figure-6006]:media/virtual-machines-shared-sap-high-availability-guide/6006-sap-multi-sid-sios-replication.png

[sap-ha-guide-figure-6007]:media/virtual-machines-shared-sap-high-availability-guide/6007-sap-multi-sid-ascs-azure-shared-disk-sid1.png
[sap-ha-guide-figure-6008]:media/virtual-machines-shared-sap-high-availability-guide/6008-sap-multi-sid-ascs-azure-shared-disk-sid2.png
[sap-ha-guide-figure-6009]:media/virtual-machines-shared-sap-high-availability-guide/6009-sap-multi-sid-ascs-azure-shared-disk-dns1.png
[sap-ha-guide-figure-6010]:media/virtual-machines-shared-sap-high-availability-guide/6010-sap-multi-sid-ascs-azure-shared-disk-dns2.png
[sap-ha-guide-figure-6011]:media/virtual-machines-shared-sap-high-availability-guide/6011-sap-multi-sid-ascs-azure-shared-disk-dns3.png

[sap-ha-guide-figure-8001]:./media/virtual-machines-shared-sap-high-availability-guide/8001.png
[sap-ha-guide-figure-8002]:./media/virtual-machines-shared-sap-high-availability-guide/8002.png
[sap-ha-guide-figure-8003]:./media/virtual-machines-shared-sap-high-availability-guide/8003.png
[sap-ha-guide-figure-8004]:./media/virtual-machines-shared-sap-high-availability-guide/8004.png
[sap-ha-guide-figure-8005]:./media/virtual-machines-shared-sap-high-availability-guide/8005.png
[sap-ha-guide-figure-8006]:./media/virtual-machines-shared-sap-high-availability-guide/8006.png
[sap-ha-guide-figure-8007]:./media/virtual-machines-shared-sap-high-availability-guide/8007.png
[sap-ha-guide-figure-8008]:./media/virtual-machines-shared-sap-high-availability-guide/8008.png
[sap-ha-guide-figure-8009]:./media/virtual-machines-shared-sap-high-availability-guide/8009.png
[sap-ha-guide-figure-8010]:./media/virtual-machines-shared-sap-high-availability-guide/8010.png
[sap-ha-guide-figure-8011]:./media/virtual-machines-shared-sap-high-availability-guide/8011.png
[sap-ha-guide-figure-8012]:./media/virtual-machines-shared-sap-high-availability-guide/8012.png
[sap-ha-guide-figure-8013]:./media/virtual-machines-shared-sap-high-availability-guide/8013.png
[sap-ha-guide-figure-8014]:./media/virtual-machines-shared-sap-high-availability-guide/8014.png
[sap-ha-guide-figure-8015]:./media/virtual-machines-shared-sap-high-availability-guide/8015.png
[sap-ha-guide-figure-8016]:./media/virtual-machines-shared-sap-high-availability-guide/8016.png
[sap-ha-guide-figure-8017]:./media/virtual-machines-shared-sap-high-availability-guide/8017.png
[sap-ha-guide-figure-8018]:./media/virtual-machines-shared-sap-high-availability-guide/8018.png
[sap-ha-guide-figure-8019]:./media/virtual-machines-shared-sap-high-availability-guide/8019.png
[sap-ha-guide-figure-8020]:./media/virtual-machines-shared-sap-high-availability-guide/8020.png
[sap-ha-guide-figure-8021]:./media/virtual-machines-shared-sap-high-availability-guide/8021.png
[sap-ha-guide-figure-8022]:./media/virtual-machines-shared-sap-high-availability-guide/8022.png
[sap-ha-guide-figure-8023]:./media/virtual-machines-shared-sap-high-availability-guide/8023.png
[sap-ha-guide-figure-8024]:./media/virtual-machines-shared-sap-high-availability-guide/8024.png
[sap-ha-guide-figure-8025]:./media/virtual-machines-shared-sap-high-availability-guide/8025.png


[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md