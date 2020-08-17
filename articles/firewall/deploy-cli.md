---
title: Implementación y configuración de Azure Firewall mediante la CLI de Azure
description: En este artículo, aprenderá a implementar y configurar Azure Firewall mediante la CLI de Azure.
services: firewall
author: vhorne
ms.service: firewall
ms.date: 08/29/2019
ms.author: victorh
ms.topic: how-to
ms.openlocfilehash: 3087b01e849aaa4d1f3c2b6b4060cf202927f55f
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2020
ms.locfileid: "85602642"
---
# <a name="deploy-and-configure-azure-firewall-using-azure-cli"></a>Implementación y configuración de Azure Firewall mediante la CLI de Azure

El control del acceso de red saliente es una parte importante de un plan de seguridad de red de ámbito general. Por ejemplo, es posible que desee limitar el acceso a sitios web. O bien, que desee limitar las direcciones IP de salida y los puertos a los que se puede acceder.

Una manera de controlar el acceso de red saliente desde una subred de Azure es con Azure Firewall. Con Azure Firewall, puede configurar:

* Reglas de aplicación que definen los nombres de dominio completos (FQDN) a los que se puede acceder desde una subred. El nombre de dominio completo también puede [incluir instancias de SQL](sql-fqdn-filtering.md).
* Reglas de red que definen la dirección de origen, el protocolo, el puerto de destino y la dirección de destino.

El tráfico está sujeto a las reglas de firewall configuradas cuando enruta el tráfico al firewall como puerta de enlace predeterminada de la subred.

En este artículo, creará una red virtual única simplificada con tres subredes para facilitar la implementación. Para las implementaciones de producción, se recomienda un [modelo de concentrador y radio](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke). El firewall está en su propia red virtual. Los servidores de las cargas de trabajo están en redes virtuales emparejadas en la misma región con una o varias subredes.

* **AzureFirewallSubnet**: el firewall está en esta subred.
* **Workload-SN**: el servidor de carga de trabajo está en esta subred. El tráfico de red de esta subred va a través del firewall.
* **Jump-SN**: el servidor de "salto" está en esta subred. El servidor de salto tiene una dirección IP pública a la que puede conectarse mediante Escritorio remoto. Desde allí se puede conectar posteriormente al servidor de carga de trabajo (mediante otro Escritorio remoto).

![Infraestructura de red del tutorial](media/tutorial-firewall-rules-portal/Tutorial_network.png)

En este artículo aprenderá a:

> [!div class="checklist"]
> * Configurar un entorno de red de prueba
> * Implementar un firewall
> * Crear una ruta predeterminada
> * Configurar una regla de aplicación para permitir el acceso a www.google.com
> * Configuración de una regla de red para permitir el acceso a los servidores DNS externos
> * Probar el firewall

Si lo prefiere, puede completar este procedimiento desde [Azure Portal](tutorial-firewall-deploy-portal.md) o [Azure PowerShell](deploy-ps.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Prerrequisitos

### <a name="azure-cli"></a>Azure CLI

Si decide instalar y usar la CLI localmente, ejecute la CLI de Azure versión 2.0.4 o posterior. Para averiguar la versión, ejecute el comando **az --version**. Para más información sobre la instalación o actualización, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

Instalación de la extensión de Azure Firewall:

```azurecli-interactive
az extension add -n azure-firewall
```


## <a name="set-up-the-network"></a>Configuración de la red

En primer lugar, cree un grupo de recursos para que contenga los recursos necesarios para implementar el firewall. A continuación, cree una red virtual, subredes y pruebe los servidores.

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

El grupo de recursos contiene todos los recursos necesarios para la implementación.

```azurecli-interactive
az group create --name Test-FW-RG --location eastus
```

### <a name="create-a-vnet"></a>Creación de una red virtual

Esta red virtual tiene tres subredes.

> [!NOTE]
> El tamaño de la subred AzureFirewallSubnet es /26. Para más información sobre el tamaño de la subred, consulte [Preguntas más frecuentes sobre Azure Firewall](firewall-faq.md#why-does-azure-firewall-need-a-26-subnet-size).

```azurecli-interactive
az network vnet create \
  --name Test-FW-VN \
  --resource-group Test-FW-RG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name AzureFirewallSubnet \
  --subnet-prefix 10.0.1.0/26
az network vnet subnet create \
  --name Workload-SN \
  --resource-group Test-FW-RG \
  --vnet-name Test-FW-VN   \
  --address-prefix 10.0.2.0/24
az network vnet subnet create \
  --name Jump-SN \
  --resource-group Test-FW-RG \
  --vnet-name Test-FW-VN   \
  --address-prefix 10.0.3.0/24
```

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Ahora cree las máquinas virtuales de salto y de cargas de trabajo, y colóquelas en las subredes adecuadas.
Cuando se le solicite, escriba la contraseña de la máquina virtual.

Cree la máquina virtual Srv-Jump.

```azurecli-interactive
az vm create \
    --resource-group Test-FW-RG \
    --name Srv-Jump \
    --location eastus \
    --image win2016datacenter \
    --vnet-name Test-FW-VN \
    --subnet Jump-SN \
    --admin-username azureadmin
az vm open-port --port 3389 --resource-group Test-FW-RG --name Srv-Jump
```



Cree una tarjeta de interfaz de red para Srv-Work con las direcciones IP del servidor DNS específico y ninguna dirección IP pública con la que realizar pruebas.

```azurecli-interactive
az network nic create \
    -g Test-FW-RG \
    -n Srv-Work-NIC \
   --vnet-name Test-FW-VN \
   --subnet Workload-SN \
   --public-ip-address "" \
   --dns-servers 209.244.0.3 209.244.0.4
```

Ahora cree la máquina virtual de cargas de trabajo.
Cuando se le solicite, escriba la contraseña de la máquina virtual.

```azurecli-interactive
az vm create \
    --resource-group Test-FW-RG \
    --name Srv-Work \
    --location eastus \
    --image win2016datacenter \
    --nics Srv-Work-NIC \
    --admin-username azureadmin
```

## <a name="deploy-the-firewall"></a>Implementación del firewall

Ahora, implemente el firewall en la red virtual.

```azurecli-interactive
az network firewall create \
    --name Test-FW01 \
    --resource-group Test-FW-RG \
    --location eastus
az network public-ip create \
    --name fw-pip \
    --resource-group Test-FW-RG \
    --location eastus \
    --allocation-method static \
    --sku standard
az network firewall ip-config create \
    --firewall-name Test-FW01 \
    --name FW-config \
    --public-ip-address fw-pip \
    --resource-group Test-FW-RG \
    --vnet-name Test-FW-VN
az network firewall update \
    --name Test-FW01 \
    --resource-group Test-FW-RG 
az network public-ip show \
    --name fw-pip \
    --resource-group Test-FW-RG
fwprivaddr="$(az network firewall ip-config list -g Test-FW-RG -f Test-FW01 --query "[?name=='FW-config'].privateIpAddress" --output tsv)"
```

Anote la dirección IP privada. Se usará más adelante al crear la ruta predeterminada.

## <a name="create-a-default-route"></a>Crear una ruta predeterminada

Creación de una tabla con la propagación de la ruta BGP deshabilitada

```azurecli-interactive
az network route-table create \
    --name Firewall-rt-table \
    --resource-group Test-FW-RG \
    --location eastus \
    --disable-bgp-route-propagation true
```

Cree la ruta.

```azurecli-interactive
az network route-table route create \
  --resource-group Test-FW-RG \
  --name DG-Route \
  --route-table-name Firewall-rt-table \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address $fwprivaddr
```

Asociación de la tabla de rutas a la subred

```azurecli-interactive
az network vnet subnet update \
    -n Workload-SN \
    -g Test-FW-RG \
    --vnet-name Test-FW-VN \
    --address-prefixes 10.0.2.0/24 \
    --route-table Firewall-rt-table
```

## <a name="configure-an-application-rule"></a>Configuración de una regla de aplicación

La regla de aplicación permite el acceso saliente a www.google.com.

```azurecli-interactive
az network firewall application-rule create \
   --collection-name App-Coll01 \
   --firewall-name Test-FW01 \
   --name Allow-Google \
   --protocols Http=80 Https=443 \
   --resource-group Test-FW-RG \
   --target-fqdns www.google.com \
   --source-addresses 10.0.2.0/24 \
   --priority 200 \
   --action Allow
```

Azure Firewall incluye una colección de reglas integradas para FQDN de infraestructura que están permitidos de forma predeterminada. Estos FQDN son específicos para la plataforma y no se pueden usar para otros fines. Para más información, consulte [Nombres de dominio completos de infraestructura](infrastructure-fqdns.md).

## <a name="configure-a-network-rule"></a>Configurar una regla de red

La regla de red permite el acceso saliente a dos direcciones IP en el puerto 53 (DNS).

```azurecli-interactive
az network firewall network-rule create \
   --collection-name Net-Coll01 \
   --destination-addresses 209.244.0.3 209.244.0.4 \
   --destination-ports 53 \
   --firewall-name Test-FW01 \
   --name Allow-DNS \
   --protocols UDP \
   --resource-group Test-FW-RG \
   --priority 200 \
   --source-addresses 10.0.2.0/24 \
   --action Allow
```

## <a name="test-the-firewall"></a>Probar el firewall

Ahora, pruebe el firewall para confirmar que funciona según lo previsto.

1. Anote la dirección IP privada de la máquina virtual **Srv-Work**:

   ```azurecli-interactive
   az vm list-ip-addresses \
   -g Test-FW-RG \
   -n Srv-Work
   ```

1. Conecte un escritorio remoto a la máquina virtual **Srv-Jump** e inicie sesión. Desde ahí, abra una conexión del escritorio remoto a la dirección IP privada de **Srv-Work** e inicie sesión.

3. En **SRV-Work**, abra una ventana de PowerShell y ejecute los comandos siguientes:

   ```
   nslookup www.google.com
   nslookup www.microsoft.com
   ```

   Ambos comandos deben devolver respuestas en las que se muestre que sus consultas de DNS están pasando por el firewall.

1. Ejecute los comandos siguientes:

   ```
   Invoke-WebRequest -Uri https://www.google.com
   Invoke-WebRequest -Uri https://www.google.com

   Invoke-WebRequest -Uri https://www.microsoft.com
   Invoke-WebRequest -Uri https://www.microsoft.com
   ```

   Las solicitudes de `www.google.com` deberían realizarse correctamente, mientras que las solicitudes de `www.microsoft.com` deberían producir un error. Esto demostraría que las reglas de firewall están funcionando según lo previsto.

Con ello, ha comprobado que las reglas de firewall funcionan:

* Puede resolver nombres DNS con el servidor DNS externo configurado.
* Puede navegar al FQDN permitido pero no a ningún otro.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede conservar los recursos relacionados con el firewall para el siguiente tutorial o, si ya no los necesita, eliminar el grupo de recursos **Test-FW-RG** para eliminarlos todos:

```azurecli-interactive
az group delete \
  -n Test-FW-RG
```

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Supervisión de los registros de Azure Firewall](./tutorial-diagnostics.md)
