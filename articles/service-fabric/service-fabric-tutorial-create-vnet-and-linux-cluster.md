---
title: Creación de un clúster de Service Fabric de Linux en Azure
description: Obtenga información sobre cómo implementar un clúster de Service Fabric con Linux en una instancia existente de Azure Virtual Network mediante la CLI de Azure.
ms.topic: conceptual
ms.date: 02/14/2019
ms.custom: mvc
ms.openlocfilehash: c4b71328ce59284f8870407c9492d24afe9acd8a
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88586927"
---
# <a name="deploy-a-linux-service-fabric-cluster-into-an-azure-virtual-network"></a>Implementación de un clúster de Service Fabric de Linux en una red virtual de Azure

En este artículo, aprenderá a implementar un clúster de Service Fabric con Linux en una [red virtual de Azure (VNET)](../virtual-network/virtual-networks-overview.md) mediante la CLI de Azure y una plantilla. Cuando haya terminado, tendrá un clúster que se ejecuta en la nube en el que puede implementar aplicaciones. Para crear un clúster con Windows mediante PowerShell, consulte la información sobre la [creación de un clúster con Windows seguro en Azure](service-fabric-tutorial-create-vnet-and-windows-cluster.md).

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar:

* Si no tiene ninguna suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Instale la [CLI de Service Fabric](service-fabric-cli.md).
* Instale la [CLI de Azure](/cli/azure/install-azure-cli).
* Para aprender los conceptos básicos de clústeres, lea [Información general de los clústeres de Azure](service-fabric-azure-clusters-overview.md).
* [Planeación y preparación](service-fabric-cluster-azure-deployment-preparation.md) para la implementación de un clúster de producción.

Los siguientes procedimientos crean un clúster de Service Fabric de siete nodos. Para calcular el costo producido por la ejecución de un clúster de Service Fabric en Azure, use la [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/).

## <a name="download-and-explore-the-template"></a>Descarga y exploración de la plantilla

Descargue los siguientes archivos de plantilla de Resource Manager:

Para Ubuntu 16.04 LTS:

* [AzureDeploy.json][template]
* [AzureDeploy.Parameters.json][parameters]

Para Ubuntu 18.04 LTS:

* [AzureDeploy.json][template2]
* [AzureDeploy.Parameters.json][parameters2]

En el caso de Ubuntu 18.04 LTS, la diferencia entre las dos plantillas es 
* el establecimiento del atributo **vmImageSku** en "18.04-LTS"
* el establecimiento de **typeHandlerVersion** de cada nodo en 1.1
* Microsoft.ServiceFabric/clusters resource's
   - el establecimiento de **apiVersion** en "2019-03-01" o versiones posteriores
   - el establecimiento de la propiedad **vmImage** en "Ubuntu18_04"

Esta plantilla implementa un clúster seguro de siete máquinas virtuales y tres tipos de nodo en una red virtual.  Se pueden encontrar otras plantillas de ejemplo en [GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates). El archivo [AzureDeploy.json][template] permite implementar varios recursos, como los siguientes.

### <a name="service-fabric-cluster"></a>Clúster de Service Fabric

En el recurso **Microsoft.ServiceFabric/clusters**, se ha implementado un clúster de Linux con las siguientes características:

* tres tipos de nodo
* cinco nodos en el tipo de nodo principal (configurable en los parámetros de la plantilla), cada nodo está presente en cada uno de los otros tipos de nodo
* Sistema operativo: (Ubuntu 16.04 LTS / Ubuntu 18.04 LTS) (configurable en los parámetros de la plantilla)
* protección con certificado (configurable en los parámetros de la plantilla)
* se habilita el [servicio DNS](service-fabric-dnsservice.md)
* [Nivel de durabilidad](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster): Bronze (configurable en los parámetros de plantilla)
* [Nivel de confiabilidad](service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster): Silver (configurable en los parámetros de plantilla)
* punto de conexión de la conexión de cliente: 19000 (configurable en los parámetros de la plantilla)
* punto de conexión de la puerta de enlace HTTP: 19080 (configurable en los parámetros de la plantilla)

### <a name="azure-load-balancer"></a>Azure Load Balancer

En el recurso **Microsoft.Network/loadBalancers** se han configurado un equilibrador de carga y sondeos y reglas para los siguientes puertos:

* punto de conexión de la conexión de cliente: 19000
* punto de conexión de la puerta de enlace HTTP: 19080
* puerto de la aplicación: 80
* puerto de la aplicación: 443

### <a name="virtual-network-and-subnet"></a>Red virtual y subred

Los nombres de la red virtual y la subred se declaran en los parámetros de la plantilla.  Los espacios de direcciones de la red virtual y de la subred se declaran también en los parámetros de plantilla y se configuran en el recurso **Microsoft.Network/virtualNetworks**:

* espacio de direcciones de red virtual: 10.0.0.0/16
* espacio de direcciones de subred de Service Fabric: 10.0.2.0/24

Si se necesitan otros puertos de aplicación, deberá ajustar el recurso Microsoft.Network/loadBalancers para permitir el tráfico de entrada.

## <a name="set-template-parameters"></a>Configuración de los parámetros de plantilla

El archivo **AzureDeploy.Parameters** permite declarar muchos valores que se usan para implementar el clúster y los recursos asociados. Estos son algunos de los parámetros que debe modificar para su implementación:

|Parámetro|Valor de ejemplo|Notas|
|---|---||
|adminUserName|vmadmin| Nombre de usuario del administrador de las máquinas virtuales del clúster. |
|adminPassword|Password#1234| Contraseña del administrador de las máquinas virtuales del clúster.|
|clusterName|mysfcluster123| Nombre del clúster. |
|ubicación|southcentralus| Ubicación del clúster. |
|certificateThumbprint|| <p>El valor debe estar vacío si se va a crear un certificado autofirmado o a proporcionar un archivo de certificados.</p><p>Para usar un certificado existente cargado previamente en un almacén de claves, rellene el valor de huella digital SHA1 del certificado. Por ejemplo, "6190390162C988701DB5676EB81083EA608DCCF3". </p>|
|certificateUrlValue|| <p>El valor debe estar vacío si se va a crear un certificado autofirmado o a proporcionar un archivo de certificados.</p><p>Para usar un certificado existente cargado previamente en un almacén de claves, especifique la dirección URL del certificado. Por ejemplo, "https:\//mykeyvault.vault.azure.net:443/secrets/mycertificate/02bea722c9ef4009a76c5052bcbf8346".</p>|
|sourceVaultValue||<p>El valor debe estar vacío si se va a crear un certificado autofirmado o a proporcionar un archivo de certificados.</p><p>Para usar un certificado existente cargado previamente en un almacén de claves, especifique el valor del almacén de claves de origen. Por ejemplo, "/subscriptions/333cc2c84-12fa-5778-bd71-c71c07bf873f/resourceGroups/MyTestRG/providers/Microsoft.KeyVault/vaults/MYKEYVAULT".</p>|

<a id="createvaultandcert" name="createvaultandcert_anchor"></a>

## <a name="deploy-the-virtual-network-and-cluster"></a>Implementación de la red virtual y el clúster

A continuación, configure la topología de red e implemente el clúster de Service Fabric. La plantilla **AzureDeploy.json** de Resource Manager crea una red virtual (VNET) y una subred para Service Fabric. La plantilla permite también implementar un clúster con seguridad mediante certificados habilitada.  Para los clústeres de producción, use un certificado de una entidad de certificación (CA) como certificado del clúster. Un certificado autofirmado se puede usar para proteger los clústeres de prueba.

La plantilla de este artículo implementa un clúster que usa la huella digital del certificado para identificar el certificado del clúster.  No hay dos certificados que puedan tener la misma huella digital, lo que dificulta la administración de certificados. Si cambia un clúster implementado para que use nombres comunes del certificado en vez de las huellas digitales del mismo, será mucho más fácil administrar los certificados.  Para más información sobre cómo actualizar el clúster para que use nombres comunes del certificado para la administración de certificados, consulte cómo [cambiar el clúster a administración de nombre común del certificado](service-fabric-cluster-change-cert-thumbprint-to-cn.md).

### <a name="create-a-cluster-using-an-existing-certificate"></a>Creación de un clúster mediante un certificado existente

El script siguiente usa el comando [az sf cluster create](/cli/azure/sf/cluster?view=azure-cli-latest) y una plantilla para implementar un clúster nuevo protegido con un certificado existente. El comando también crea un nuevo almacén de claves de Azure y carga el certificado.

```azurecli
ResourceGroupName="sflinuxclustergroup"
Location="southcentralus"
Password="q6D7nN%6ck@6"
VaultName="linuxclusterkeyvault"
VaultGroupName="linuxclusterkeyvaultgroup"
CertPath="C:\MyCertificates\MyCertificate.pem"

# sign in to your Azure account and select your subscription
az login
az account set --subscription <guid>

# Create a new resource group for your deployment and give it a name and a location.
az group create --name $ResourceGroupName --location $Location

# Create the Service Fabric cluster.
az sf cluster create --resource-group $ResourceGroupName --location $Location \
   --certificate-password $Password --certificate-file $CertPath \
   --vault-name $VaultName --vault-resource-group $ResourceGroupName  \
   --template-file AzureDeploy.json --parameter-file AzureDeploy.Parameters.json
```

### <a name="create-a-cluster-using-a-new-self-signed-certificate"></a>Creación de un clúster mediante un nuevo certificado autofirmado

El script siguiente usa el comando [az sf cluster create](/cli/azure/sf/cluster?view=azure-cli-latest) y una plantilla para implementar un clúster nuevo en Azure. El comando también crea un almacén de claves en Azure, agrega un nuevo certificado autofirmado al almacén de claves y carga el certificado en el archivo de certificado de forma local.

```azurecli
ResourceGroupName="sflinuxclustergroup"
ClusterName="sflinuxcluster"
Location="southcentralus"
Password="q6D7nN%6ck@6"
VaultName="linuxclusterkeyvault"
VaultGroupName="linuxclusterkeyvaultgroup"
CertPath="C:\MyCertificates"

az sf cluster create --resource-group $ResourceGroupName --location $Location \
   --cluster-name $ClusterName --template-file C:\temp\cluster\AzureDeploy.json \
   --parameter-file C:\temp\cluster\AzureDeploy.Parameters.json --certificate-password $Password \
   --certificate-output-folder $CertPath --certificate-subject-name $ClusterName.$Location.cloudapp.azure.com \
   --vault-name $VaultName --vault-resource-group $ResourceGroupName
```

## <a name="connect-to-the-secure-cluster"></a>Conexión al clúster seguro

Conéctese al clúster mediante el comando `sfctl cluster select` de la CLI de Service Fabric con su clave.  Nota: Use la opción **--no-verify** solo para obtener un certificado autofirmado.

```console
sfctl cluster select --endpoint https://aztestcluster.southcentralus.cloudapp.azure.com:19080 \
--pem ./aztestcluster201709151446.pem --no-verify
```

Compruebe que está conectado y que el estado del clúster es correcto mediante el comando `sfctl cluster health`.

```console
sfctl cluster health
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a pasar inmediatamente al siguiente artículo, puede [eliminar el clúster](./service-fabric-tutorial-delete-cluster.md) para evitar incurrir en cargos.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [escalar un clúster](service-fabric-tutorial-scale-cluster.md).

La plantilla de este artículo implementa un clúster que usa la huella digital del certificado para identificar el certificado del clúster.  No hay dos certificados que puedan tener la misma huella digital, lo que dificulta la administración de certificados. Si cambia un clúster implementado para que use nombres comunes del certificado en vez de las huellas digitales del mismo, será mucho más fácil administrar los certificados.  Para más información sobre cómo actualizar el clúster para que use nombres comunes del certificado para la administración de certificados, consulte cómo [cambiar el clúster a administración de nombre común del certificado](service-fabric-cluster-change-cert-thumbprint-to-cn.md).

[template]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-3-NodeTypes-Secure/AzureDeploy.json
[parameters]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-3-NodeTypes-Secure/AzureDeploy.Parameters.json
[template2]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-1804-3-NodeTypes-Secure/AzureDeploy.json
[parameters2]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-1804-3-NodeTypes-Secure/AzureDeploy.Parameters.json
