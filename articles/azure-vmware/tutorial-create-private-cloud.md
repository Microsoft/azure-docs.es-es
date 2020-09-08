---
title: 'Tutorial: Implementación del clúster de vSphere en Azure'
description: Aprenda a implementar un clúster de vSphere en Azure mediante Azure VMware Solution
ms.topic: tutorial
ms.date: 09/07/2020
ms.openlocfilehash: 69a29a459ba283bb34169112ac2fa174ac6a14af
ms.sourcegitcommit: 8791f69d44150767807d215cafc4076f3ed43f9f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/08/2020
ms.locfileid: "89512388"
---
# <a name="tutorial-deploy-an-azure-vmware-solution-private-cloud-in-azure"></a>Tutorial: Implementación de una nube privada de Azure VMware Solution en Azure

Azure VMware Solution permite implementar un clúster de vSphere en Azure. La implementación inicial mínima es de tres hosts. Se pueden agregar más hosts de uno en uno, hasta un máximo de 16 hosts por clúster. 

Dado que Azure VMware Solution no permite administrar la nube privada con la instancia local de vCenter inicial, se necesitan una configuración adicional y una conexión a una instancia local de vCenter, una red virtual y otros elementos. Estos procedimientos y los requisitos previos relacionados se tratan en este tutorial.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una nube privada de Azure VMware Solution
> * Comprobar la nube privada implementada

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Permisos y derechos administrativos adecuados para crear una nube privada.
- Asegúrese de que tiene las opciones de red correctamente configuradas como se describe en [Tutorial: Lista de comprobación de red](tutorial-network-checklist.md).

## <a name="register-the-resource-provider"></a>Registrar el proveedor de recursos

[!INCLUDE [register-resource-provider-steps](includes/register-resource-provider-steps.md)]


## <a name="create-a-private-cloud"></a>Creación de una nube privada

Puede crear una nube privada de Azure VMware Solution mediante [Azure Portal](#azure-portal) o con la [CLI de Azure](#azure-cli).

### <a name="azure-portal"></a>Azure Portal

[!INCLUDE [create-avs-private-cloud-azure-portal](includes/create-avs-private-cloud-azure-portal-steps.md)]

### <a name="azure-cli"></a>Azure CLI

En lugar de usar Azure Portal para crear una nube privada de Azure VMware Solution, puede usar la CLI de Azure con Azure Cloud Shell. Este un shell interactivo gratuito con herramientas comunes de Azure preinstaladas y configuradas para usar con la cuenta. 

#### <a name="open-azure-cloud-shell"></a>Apertura de Azure Cloud Shell

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/bash](https://shell.azure.com/bash) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione **Entrar** para ejecutarlos.

#### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```
azurecli-interactive
az group create --name myResourceGroup --location eastus
```

#### <a name="create-a-private-cloud"></a>Creación de una nube privada

Proporcione un nombre de grupo de recursos, un nombre para la nube privada, una ubicación y el tamaño del clúster.

| Propiedad  | Descripción  |
| --------- | ------------ |
| **-g** (nombre del grupo de recursos)     | Nombre del grupo de recursos para los recursos de nube privada.        |
| **-n** (nombre de nube privada)     | Nombre de la nube privada de Azure VMware Solution.        |
| **--location**     | Ubicación que se usó para la nube privada.         |
| **--cluster-size**     | El tamaño del clúster. El valor mínimo es 3.         |
| **--network-block**     | Bloque de red de ñas direcciones IP de CIDR que se usará para la nube privada. El bloque de direcciones no debe superponerse a los bloques de direcciones que se hayan usado en otras redes virtuales ubicadas en las redes locales y de la suscripción.        |
| **--sku** | Valor de la SKU: AV36 |

```
azurecli-interactive
az vmware private-cloud create -g myResourceGroup -n myPrivateCloudName --location eastus --cluster-size 3 --network-block xx.xx.xx.xx/22 --sku AV36
```

## <a name="delete-a-private-cloud-azure-portal"></a>Eliminación de una nube privada (Azure Portal)

Si tiene una nube privada de Azure VMware Solution que ya no necesita, puede eliminarla. Al eliminar una nube privada, se eliminan todos los clústeres, junto con todos sus componentes.

Para ello, vaya a su nube privada en Azure Portal y seleccione **Eliminar**. En la página de confirmación, confirme el nombre de la nube privada y seleccione **Sí**.

> [!CAUTION]
> La eliminación de la nube privada es una operación irreversible. Una vez eliminada la nube privada, no se pueden recuperar los datos, ya que se finalizan todas las cargas de trabajo en ejecución, se eliminan los componentes y se destruyen todos los datos y valores de configuración de la nube privada, incluidas las direcciones IP públicas. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Creación de una nube privada de Azure VMware Solution
> * Comprobar la nube privada implementada

Continúe con el siguiente tutorial para aprender a crear una red virtual para su uso con la nube privada como parte de la configuración de la administración local de los clústeres de nube privada.

> [!div class="nextstepaction"]
> [Creación de una red virtual](tutorial-configure-networking.md)
