---
title: 'Tutorial: Implementación del clúster de vSphere en Azure'
description: Aprenda a implementar un clúster de vSphere en Azure mediante la solución de VMWare en Azure (AVS)
ms.topic: tutorial
ms.date: 07/15/2020
ms.openlocfilehash: 4f3b33ea401c62124ae5f8a4c881d86d2f19b40c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079424"
---
# <a name="tutorial-deploy-an-avs-private-cloud-in-azure"></a>Tutorial: Implementación de una nube privada de AVS en Azure

La solución de VMware en Azure (AVS) permite implementar un clúster de vSphere en Azure. La implementación inicial mínima es de tres hosts. Se pueden agregar más hosts de uno en uno, hasta un máximo de 16 hosts por clúster. 

Dado que AVS no permite administrar su nube privada con su instancia local de vCenter inicialmente, se necesita una configuración adicional y conexión a una instancia de vCenter local, una red virtual y más. Estos procedimientos y los requisitos previos relacionados se tratan en este tutorial.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una nube privada de AVS
> * Comprobar la nube privada implementada

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Permisos y derechos administrativos adecuados para crear una nube privada.
- Asegúrese de que tiene las opciones de red correctamente configuradas como se describe en [Tutorial: Lista de comprobación de red](tutorial-network-checklist.md).

## <a name="register-the-resource-provider"></a>Registrar el proveedor de recursos

Para usar AVS, primero debe registrar el proveedor de recursos con su suscripción.

```
azurecli-interactive
az provider register -n Microsoft.AVS --subscription <your subscription ID>
```

Para ver otras formas de registrar el proveedor de recursos, consulte [Tipos y proveedores de recursos de Azure](../azure-resource-manager/management/resource-providers-and-types.md).


## <a name="create-a-private-cloud"></a>Creación de una nube privada

Puede crear una nube privada de AVS mediante [Azure Portal](#azure-portal) o con la [CLI de Azure](#azure-cli).

### <a name="azure-portal"></a>Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione **Crear un nuevo recurso**. En el cuadro de texto **Buscar en el Marketplace**, escriba `Azure VMware Solution` y seleccione **Azure VMware Solution** en la lista. En la ventana **Azure VMware Solution**, seleccione **Crear**.

1. En la pestaña **Básica**, escriba valores en los campos. En la tabla siguiente se enumeran las propiedades de los campos.

   | Campo   | Value  |
   | ---| --- |
   | **Suscripción** | La suscripción que va a usar para la implementación.|
   | **Grupos de recursos** | El grupo de recursos para los recursos de nube privada. |
   | **Ubicación** | Seleccione una ubicación, como **Este de EE. UU.**|
   | **Nombre del recurso** | El nombre de la nube privada de AVS. |
   | **SKU** | Seleccione el valor de SKU siguiente: AV36 |
   | **Hosts** | Número de hosts que se van a agregar al clúster de nube privada. El valor predeterminado es 3, que se puede aumentar o reducir después de la implementación.  |
   | **Contraseña del administrador de vCenter** | Escriba una contraseña de administrador de la nube. |
   | **Contraseña de administrador de NSX-T** | Escriba una contraseña de administrador de NSX-T. |
   | **Bloque de direcciones** | Escriba un bloque de direcciones IP para la red CIDR para la nube privada; por ejemplo, 10.175.0.0/22. |

   :::image type="content" source="./media/tutorial-create-private-cloud/create-private-cloud.png" alt-text="Creación de una nube privada" border="true":::

1. Cuando termine, seleccione **Revisar y crear**. En la siguiente pantalla, compruebe la información especificada. Si toda la información es correcta, seleccione **Crear**.

   > [!NOTE]
   > Este paso tarda aproximadamente dos horas. 

1. Comprobación de que la implementación se ha realizado correctamente. Navegue hasta el grupo de recursos que creó y seleccione la nube privada.  Una vez que se haya completado la implementación, se mostrará el estado **Correcto**. 

   :::image type="content" source="./media/tutorial-create-private-cloud/validate-deployment.png" alt-text="Validación de la nube privada implementada" border="true":::

### <a name="azure-cli"></a>Azure CLI

En lugar de usar Azure Portal para crear una nube privada de AVS, puede usar la CLI de Azure con Azure Cloud Shell. Este un shell interactivo gratuito con herramientas comunes de Azure preinstaladas y configuradas para usar con la cuenta. 

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
| **-n** (nombre de nube privada)     | El nombre de la nube privada de AVS.        |
| **--location**     | Ubicación que se usó para la nube privada.         |
| **--cluster-size**     | El tamaño del clúster. El valor mínimo es 3.         |
| **--network-block**     | Bloque de red de ñas direcciones IP de CIDR que se usará para la nube privada. El bloque de direcciones no debe superponerse a los bloques de direcciones que se hayan usado en otras redes virtuales ubicadas en las redes locales y de la suscripción.        |
| **--sku** | Valor de la SKU: AV36 |

```
azurecli-interactive
az vmware private-cloud create -g myResourceGroup -n myPrivateCloudName --location eastus --cluster-size 3 --network-block xx.xx.xx.xx/22 --sku AV36
```

## <a name="delete-a-private-cloud-azure-portal"></a>Eliminación de una nube privada (Azure Portal)

Si tiene una nube privada de AVS que ya no necesita, puede eliminarla. Al eliminar una nube privada, se eliminan todos los clústeres, junto con todos sus componentes.

Para ello, vaya a su nube privada en Azure Portal y seleccione **Eliminar**. En la página de confirmación, confirme el nombre de la nube privada y seleccione **Sí**.

> [!CAUTION]
> La eliminación de la nube privada es una operación irreversible. Una vez eliminada la nube privada, no se pueden recuperar los datos, ya que se finalizan todas las cargas de trabajo en ejecución, se eliminan los componentes y se destruyen todos los datos y valores de configuración de la nube privada, incluidas las direcciones IP públicas. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear una nube privada de AVS
> * Comprobar la nube privada implementada

Continúe con el siguiente tutorial para aprender a crear una red virtual para su uso con la nube privada como parte de la configuración de la administración local de los clústeres de nube privada.

> [!div class="nextstepaction"]
> [Creación de una red virtual](tutorial-configure-networking.md)
