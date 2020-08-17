---
title: Creación de una configuración de alta disponibilidad de SAP con varios SID en Azure
description: Guía de alta disponibilidad para la configuración de varios SID de SAP NetWeaver en máquinas virtuales Windows
services: virtual-machines-windows, virtual-network, storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 0b89b4f8-6d6c-45d7-8d20-fe93430217ca
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 9cf62be1ad9d95ff20351966bf21af0cb3d9f0a3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079883"
---
# <a name="create-a-high-availability-sap-netweaver-multi-sid-configuration"></a>Creación de una configuración de alta disponibilidad con varios SID de SAP NetWeaver

[load-balancer-multivip-overview]:../../../load-balancer/load-balancer-multivip-overview.md
[sap-ha-guide]:sap-high-availability-guide.md 
[sap-ha-guide-figure-6001]:./media/virtual-machines-shared-sap-high-availability-guide/6001-sap-multi-sid-ascs-scs-sid1.png
[sap-ha-guide-figure-6002]:./media/virtual-machines-shared-sap-high-availability-guide/6002-sap-multi-sid-ascs-scs.png
[sap-ha-guide-figure-6003]:./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png
[sap-ha-guide-figure-6004]:./media/virtual-machines-shared-sap-high-availability-guide/6004-sap-multi-sid-dns.png
[sap-ha-guide-figure-6005]:./media/virtual-machines-shared-sap-high-availability-guide/6005-sap-multi-sid-azure-portal.png
[sap-ha-guide-figure-6006]:./media/virtual-machines-shared-sap-high-availability-guide/6006-sap-multi-sid-sios-replication.png
[networking-limits-azure-resource-manager]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits
[sap-ha-guide-9.1.1]:sap-high-availability-guide.md#a97ad604-9094-44fe-a364-f89cb39bf097 
[sap-ha-guide-8.8]:sap-high-availability-guide.md#f19bd997-154d-4583-a46e-7f5a69d0153c
[sap-ha-guide-8.12.3.3]:sap-high-availability-guide.md#d9c1fc8e-8710-4dff-bec2-1f535db7b006 
[sap-ha-guide-9]:sap-high-availability-guide.md#a06f0b49-8a7a-42bf-8b0d-c12026c5746b
[sap-ha-guide-9.1]:sap-high-availability-guide.md#31c6bd4f-51df-4057-9fdf-3fcbc619c170 
[sap-ha-guide-9.1.2]:sap-high-availability-guide.md#eb5af918-b42f-4803-bb50-eff41f84b0b0 
[sap-ha-guide-9.1.3]:sap-high-availability-guide.md#e4caaab2-e90f-4f2c-bc84-2cd2e12a9556 
[sap-ha-guide-9.1.4]:sap-high-availability-guide.md#10822f4f-32e7-4871-b63a-9b86c76ce761 
[sap-ha-guide-9.4]:sap-high-availability-guide.md#094bc895-31d4-4471-91cc-1513b64e406a 
[sap-ha-guide-9.5]:sap-high-availability-guide.md#2477e58f-c5a7-4a5d-9ae3-7b91022cafb5 
[sap-ha-guide-9.6]:sap-high-availability-guide.md#0ba4a6c1-cc37-4bcf-a8dc-025de4263772 
[sap-ha-guide-10]:sap-high-availability-guide.md#18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9

En septiembre de 2016, Microsoft publicó una característica con la que puede administrar varias direcciones IP virtuales mediante un equilibrador de carga interno de Azure. Esta funcionalidad ya existe en el equilibrador de carga externo de Azure.

Si tiene una implementación de SAP, puede usar un equilibrador de carga interno para crear una configuración de clúster de Windows para ASCS/SCS de SAP, tal y como se describe en la [guía de alta disponibilidad de SAP NetWeaver en máquinas virtuales Windows][sap-ha-guide].

En este artículo nos centraremos en cómo pasar de una sola instalación ASCS/SCS de este tipo a una configuración de varios SID de SAP mediante la instalación de instancias en clúster ASCS/SCS de SAP adicionales en un clúster de servidor de conmutación por error de Windows (WSFC) existente. Cuando se complete este proceso, habrá configurado un clúster de varios SID de SAP.

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos
Ya ha configurado un clúster de WSFC que se usa para una instancia de ASCS/SCS de SAP, tal como se describe en la [guía de alta disponibilidad SAP NetWeaver en máquinas virtuales Windows][sap-ha-guide] y se muestra en este diagrama.

![Instancia de ASCS/SCS de SAP de alta disponibilidad][sap-ha-guide-figure-6001]

## <a name="target-architecture"></a>Arquitectura de destino

El objetivo es poder instalar varias instancias en clúster SAP ABAP ASCS o SAP Java SCS en el mismo clúster de WSFC:

![Varias instancias en clúster ASCS/SCS de SAP en Azure][sap-ha-guide-figure-6002]

> [!NOTE]
>Hay un límite en el número de direcciones IP de front-end privadas por cada equilibrador de carga interno de Azure.
>
>El número máximo de instancias ASCS/SCS de SAP en un clúster de WSFC es igual al número máximo de IP de front-end privadas por equilibrador de carga interno de Azure.
>

Para más información sobre los límites del equilibrador de carga, consulte "Dirección IP de front-end pública por Load Balancer" en [Límites de redes: Azure Resource Manager][networking-limits-azure-resource-manager].

La visión global con la perspectiva completa con dos sistemas SAP de alta disponibilidad sería esta:

![Configuración de varios SID de alta disponibilidad de SAP con dos sistemas SAP][sap-ha-guide-figure-6003]

> [!IMPORTANT]
> Debe cumplir las condiciones siguientes:
> - Las instancias ASCS/SCS de SAP deben compartir el mismo clúster de WSFC.
> - Cada SID de DBMS tiene su propio clúster de WSFC dedicado.
> - Los servidores de aplicaciones de SAP que pertenecen a un SID del sistema SAP tienen sus propias máquinas virtuales dedicadas.


## <a name="prepare-the-infrastructure"></a>Preparación de la infraestructura
Para preparar la infraestructura, puede instalar una instancia de ASCS/SCS de SAP adicionales con los siguientes parámetros:

| Nombre de parámetro | Value |
| --- | --- |
| SID de ASCS/SCS de SAP |pr1-lb-ascs |
| Equilibrador de carga interno de DBMS de SAP | PR5 |
| Nombre de host virtual de SAP | pr5-sap-cl |
| Dirección IP del host virtual de ASCS/SCS de SAP (dirección IP adicional de Azure Load Balancer) | 10.0.0.50 |
| Número de instancia de ASCS/SCS de SAP | 50 |
| Puerto de sondeo de ILB para instancia adicional de ASCS/SCS de SAP | 62350 |

> [!NOTE]
> Para las instancias en clúster ASCS/SCS de SAP, cada dirección IP requiere un puerto de sondeo específico. Por ejemplo, si una dirección IP en un equilibrador de carga interno de Azure usa el puerto de sondeo 62300, no puede usarlo ninguna otra dirección IP de ese equilibrador de carga.
>
>Para nuestro objetivo, dado que el puerto de sondeo 62300 está reservado, estamos usando el puerto de sondeo 62350.

Instalará la instancia adicional ASCS/SCS de SAP en el clúster de WSFC existente con dos nodos:

| Rol de máquina virtual | Nombre de host de máquina virtual | Dirección IP estática |
| --- | --- | --- |
| Primer nodo del clúster de la instancia de ASCS/SCS |pr1-ascs-0 |10.0.0.10 |
| Segundo nodo del clúster de la instancia de ASCS/SCS |pr1-ascs-1 |10.0.0.9 |

### <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance-on-the-dns-server"></a>Creación de un nombre de host virtual para la instancia de ASCS/SCS de SAP en clúster en el servidor DNS

Puede crear una entrada DNS para el nombre de host virtual de la instancia de ASCS/SCS con los siguientes parámetros:

| Nuevo nombre de host virtual de ASCS/SCS de SAP | Dirección IP asociada |
| --- | --- |
|pr5-sap-cl |10.0.0.50 |

El nuevo nombre de host y la dirección IP se muestran en el Administrador de DNS, como se muestra en la captura de pantalla siguiente:

![El Administrador de DNS destaca la entrada DNS para el nombre virtual del clúster de ASCS/SCS de SAP y la dirección TCP/IP.][sap-ha-guide-figure-6004]

El procedimiento de creación de una entrada DNS también se describe en [Guía de alta disponibilidad de SAP NetWeaver en máquinas virtuales Windows][sap-ha-guide-9.1.1].

> [!NOTE]
> La dirección IP que asigna al nombre de host virtual de la instancia de ASCS/SCS adicional debe ser la misma que la nueva dirección IP que asignó a Azure Load Balancer para SAP.
>
>En nuestro escenario, la dirección IP es 10.0.0.50.

### <a name="add-an-ip-address-to-an-existing-azure-internal-load-balancer-by-using-powershell"></a>Adición de una dirección IP a un equilibrador de carga interno de Azure existente con PowerShell

Para crear más de una instancia de ASCS/SCS de SAP en el mismo clúster de WSFC, utilice PowerShell para agregar una dirección IP adicional a un equilibrador de carga interno de Azure existente. Cada dirección IP requiere sus propias reglas de equilibrio de carga, puerto de sondeo, grupo de IP de front-end y grupo de back-end.

El script siguiente agrega una nueva dirección IP a un equilibrador de carga existente. Actualice las variables de PowerShell de su entorno. El script creará todas las reglas de equilibrio de carga necesarias para todos los puertos de ASCS/SCS de SAP.

```powershell

# Select-AzSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>
Clear-Host
$ResourceGroupName = "SAP-MULTI-SID-Landscape"      # Existing resource group name
$VNetName = "pr2-vnet"                        # Existing virtual network name
$SubnetName = "Subnet"                        # Existing subnet name
$ILBName = "pr2-lb-ascs"                      # Existing ILB name                      
$ILBIP = "10.0.0.50"                          # New IP address
$VMNames = "pr2-ascs-0","pr2-ascs-1"          # Existing cluster virtual machine names
$SAPInstanceNumber = 50                       # SAP ASCS/SCS instance number: must be a unique value for each cluster
[int]$ProbePort = "623$SAPInstanceNumber"     # Probe port: must be a unique value for each IP and load balancer

$ILB = Get-AzLoadBalancer -Name $ILBName -ResourceGroupName $ResourceGroupName

$count = $ILB.FrontendIpConfigurations.Count + 1
$FrontEndConfigurationName ="lbFrontendASCS$count"
$LBProbeName = "lbProbeASCS$count"

# Get the Azure VNet and subnet
$VNet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName
$Subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName

# Add second front-end and probe configuration
Write-Host "Adding new front end IP Pool '$FrontEndConfigurationName' ..." -ForegroundColor Green
$ILB | Add-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.Id
$ILB | Add-AzLoadBalancerProbeConfig -Name $LBProbeName  -Protocol Tcp -Port $Probeport -ProbeCount 2 -IntervalInSeconds 10  | Set-AzLoadBalancer

# Get new updated configuration
$ILB = Get-AzLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
# Get new updated LP FrontendIP COnfig
$FEConfig = Get-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
$HealthProbe  = Get-AzLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

# Add new back-end configuration into existing ILB
$BackEndConfigurationName  = "backendPoolASCS$count"
Write-Host "Adding new backend Pool '$BackEndConfigurationName' ..." -ForegroundColor Green
$BEConfig = Add-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB | Set-AzLoadBalancer

# Get new updated config
$ILB = Get-AzLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName

# Assign VM NICs to backend pool
$BEPool = Get-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
foreach($VMName in $VMNames){
        $VM = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName
        $NICName = ($VM.NetworkInterfaceIDs[0].Split('/') | select -last 1)        
        $NIC = Get-AzNetworkInterface -name $NICName -ResourceGroupName $ResourceGroupName                
        $NIC.IpConfigurations[0].LoadBalancerBackendAddressPools += $BEPool
        Write-Host "Assigning network card '$NICName' of the '$VMName' VM to the backend pool '$BackEndConfigurationName' ..." -ForegroundColor Green
        Set-AzNetworkInterface -NetworkInterface $NIC
        #start-AzVM -ResourceGroupName $ResourceGroupName -Name $VM.Name
}


# Create load-balancing rules
$Ports = "445","32$SAPInstanceNumber","33$SAPInstanceNumber","36$SAPInstanceNumber","39$SAPInstanceNumber","5985","81$SAPInstanceNumber","5$SAPInstanceNumber`13","5$SAPInstanceNumber`14","5$SAPInstanceNumber`16"
$ILB = Get-AzLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
$FEConfig = get-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
$BEConfig = Get-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
$HealthProbe  = Get-AzLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

Write-Host "Creating load balancing rules for the ports: '$Ports' ... " -ForegroundColor Green

foreach ($Port in $Ports) {

        $LBConfigrulename = "lbrule$Port" + "_$count"
        Write-Host "Creating load balancing rule '$LBConfigrulename' for the port '$Port' ..." -ForegroundColor Green

        $ILB | Add-AzLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig  -BackendAddressPool $BEConfig -Probe $HealthProbe -Protocol tcp -FrontendPort  $Port -BackendPort $Port -IdleTimeoutInMinutes 30 -LoadDistribution Default -EnableFloatingIP   
}

$ILB | Set-AzLoadBalancer

Write-Host "Successfully added new IP '$ILBIP' to the internal load balancer '$ILBName'!" -ForegroundColor Green

```
Cuando se haya ejecutado el script, los resultados se muestran en Azure Portal, como se muestra en la captura de pantalla siguiente:

![Nuevo grupo de IP de front-end en Azure Portal][sap-ha-guide-figure-6005]

### <a name="add-disks-to-cluster-machines-and-configure-the-sios-cluster-share-disk"></a>Adición de máquinas de clúster y configuración de discos compartidos de clúster de SIOS

Debe agregar un nuevo disco compartido de clúster para una nueva instancia adicional ASCS/SCS de SAP. Para Windows Server 2012 R2, el disco compartido de clúster de WSFC que se usa actualmente es la solución de software SIOS DataKeeper.

Haga lo siguiente:
1. Agregue discos adicionales o del mismo tamaño (que deba seccionar) con el mismo tamaño a cada uno de los nodos del clúster y formatéelos.
2. Configure la replicación de almacenamiento con SIOS DataKeeper.

Este procedimiento da por supuesto que ya ha instalado SIOS DataKeeper en los equipos del clúster WSFC. Si se ha instalado, ahora debe configurar la replicación entre las máquinas. El procedimiento se describe detalladamente en la [Guía de alta disponibilidad de SAP NetWeaver en máquinas virtuales Windows][sap-ha-guide-8.12.3.3].  

![El reflejo sincrónico de DataKeeper para el disco compartido de ASCS/SCS de SAP][sap-ha-guide-figure-6006]

### <a name="deploy-vms-for-sap-application-servers-and-dbms-cluster"></a>Implementación de máquinas virtuales para servidores de aplicaciones SAP y clústeres de DBMS

Con el fin de finalizar la preparación de la infraestructura para el segundo sistema SAP, necesita lo siguiente:

1. Implementar máquinas virtuales dedicadas para servidores de aplicaciones de SAP y colocarlas en el propio grupo de disponibilidad dedicado
2. Implementar máquinas virtuales dedicadas para el clúster de DBMS y colocarlas en el propio grupo de disponibilidad dedicado


## <a name="install-the-second-sap-sid2-netweaver-system"></a>Instalar el segundo sistema SAP SID2 NetWeaver

El procedimiento completo de instalar un segundo sistema SID2 de SAP se describe detalladamente en la [Guía de alta disponibilidad de SAP NetWeaver en máquinas virtuales Windows][sap-ha-guide-9].

El procedimiento general es el siguiente:

1. [Instale el primer nodo de clúster de SAP][sap-ha-guide-9.1.2].  
 En este paso, instalará SAP con una instancia de ASCS/SCS de alta disponibilidad en el **nodo 1 del clúster de WSFC existente**.

2. [Modifique el perfil SAP de la instancia de ASCS/SCS][sap-ha-guide-9.1.3].

3. [Configure un puerto de sondeo][sap-ha-guide-9.1.4].  
 En este paso, va a configurar un puerto de sondeo SAP-SID2-IP del recurso de clúster de SAP mediante PowerShell. Ejecute esta configuración en uno de los nodos del clúster ASCS/SCS de SAP.

4. [Instalación de la instancia de base de datos][sap-ha-guide-9.2].  
 En este paso, va a instalar DBMS en un clúster WSFC específico.

5. [Instalación del segundo nodo del clúster][sap-ha-guide-9.3].  
 En este paso, instalará SAP con una instancia de ASCS/SCS de alta disponibilidad en el nodo 2 del clúster de WSFC existente.

6. Abra los puertos de Firewall de Windows de la instancia ASCS/SCS de SAP y el puerto de sondeo.  
 En ambos nodos del clúster usados para la instancia ASCS/SCS de SAP, abra todos los puertos de Firewall de Windows usados por los puertos ASCS/SCS de SAP. Estos puertos se enumeran en la [Guía de alta disponibilidad de SAP NetWeaver en máquinas virtuales Windows][sap-ha-guide-8.8].  
 Además, abra el puerto de sondeo del equilibrador de carga interno de Azure, que en nuestro caso es 62350.

7. [Cambie el tipo de inicio del servicio de Windows para la instancia de ERS de SAP][sap-ha-guide-9.4].

8. [Instale el servidor de aplicaciones principal de SAP][sap-ha-guide-9.5] en la nueva máquina virtual dedicada.

9. [Instale el servidor de aplicaciones adicional de SAP][sap-ha-guide-9.6] en la nueva máquina virtual dedicada.

10. [Pruebe la conmutación por error de la instancia de ASCS/SCS de SAP y de la replicación de SIOS][sap-ha-guide-10].

## <a name="next-steps"></a>Pasos siguientes

- [Límites de redes: Azure Resource Manager][networking-limits-azure-resource-manager]
- [Varias IP virtuales para Azure Load Balancer][load-balancer-multivip-overview]
- [Guía de alta disponibilidad para SAP NetWeaver en máquinas virtuales Windows][sap-ha-guide]
