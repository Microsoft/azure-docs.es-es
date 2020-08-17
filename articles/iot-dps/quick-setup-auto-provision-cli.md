---
title: Configuración de Azure IoT Hub Device Provisioning Service mediante la CLI de Azure
description: 'Inicio rápido: configuración de Azure IoT Hub Device Provisioning Service (DPS) mediante la CLI de Azure'
author: wesmc7777
ms.author: wesmc
ms.date: 11/08/2019
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 313dfe930dec5233f96b89fdebf978a41fdcdf80
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497671"
---
# <a name="quickstart-set-up-the-iot-hub-device-provisioning-service-with-azure-cli"></a>Inicio rápido: Configuración del servicio Azure IoT Hub Device Provisioning con la CLI de Azure

La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. En este inicio rápido se detalla cómo usar la CLI de Azure para crear un centro de IoT y una instancia de IoT Hub Device Provisioning Service y para vincular los dos servicios entre sí. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

> [!IMPORTANT]
> Tanto el centro de IoT como el servicio de aprovisionamiento que se crean en este inicio rápido se podrán detectar públicamente como puntos de conexión de DNS. Asegúrese de evitar cualquier información confidencial si decide cambiar que se usan para estos recursos.
>

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]


## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group#az-group-create). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

En el ejemplo siguiente, se crea un grupo de recursos llamado *my-sample-resource-group* en la ubicación *westus*.

```azurecli-interactive 
az group create --name my-sample-resource-group --location westus
```

> [!TIP]
> El ejemplo crea el grupo de recursos en la ubicación de oeste de EE. UU. Puede ejecutar el comando `az account list-locations -o table` para ver una lista de las ubicaciones disponibles.
>
>

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

Para crear un centro de IoT, use el comando [az iot hub create](/cli/azure/iot/hub#az-iot-hub-create).

En el ejemplo siguiente, se crea un centro de IoT llamado *my-sample-hub* en la ubicación *westus*. El nombre del centro de IoT debe ser único globalmente en Azure, por lo que es posible que desee agregar un prefijo o sufijo único al nombre del ejemplo, o bien elegir un nombre completamente nuevo. Asegúrese de que el nombre sigue las convenciones de nomenclatura de un centro de IoT: debe tener entre 3 y 50 caracteres, y solo puede contener caracteres alfanuméricos en mayúsculas y minúsculas o guiones ('-'). 

```azurecli-interactive 
az iot hub create --name my-sample-hub --resource-group my-sample-resource-group --location westus
```

## <a name="create-a-device-provisioning-service"></a>Creación de una instancia de Device Provisioning Service

Cree una instancia de Device Provisioning Service con el comando [az iot dps create](/cli/azure/iot/dps#az-iot-dps-create). 

En el ejemplo siguiente, se crea un servicio de aprovisionamiento llamado *my-sample-dps* en la ubicación *westus*. También tendrá que elegir un nombre único global para su propio servicio de aprovisionamiento. Asegúrese de que sigue las convenciones de nomenclatura de IoT Hub Device Provisioning Service: debe tener entre 3 y 64 caracteres, y solo puede contener caracteres alfanuméricos en mayúsculas y minúsculas o guiones ('-').

```azurecli-interactive 
az iot dps create --name my-sample-dps --resource-group my-sample-resource-group --location westus
```

> [!TIP]
> El ejemplo crea el servicio de aprovisionamiento en la ubicación de oeste de EE. UU. Para ver una lista de las ubicaciones disponibles, ejecute el comando `az provider show --namespace Microsoft.Devices --query "resourceTypes[?resourceType=='ProvisioningServices'].locations | [0]" --out table` o vaya a la página [Estado de Azure](https://azure.microsoft.com/status/) página y busque "Servicio Device Provisioning". En los comandos, las ubicaciones se pueden especificar en formato de una palabra o de varias; por ejemplo, westus, West US, WEST US, etc. El valor no distingue mayúsculas de minúsculas. Si utiliza el formato de varias palabras para especificar la ubicación, escriba el valor entre comillas; por ejemplo, `--location "West US"`.
>

## <a name="get-the-connection-string-for-the-iot-hub"></a>Obtención de la cadena de conexión del centro de IoT

La cadena de conexión de un centro de IoT se necesita para vincularlo el servicio Device Provisioning. Utilice el comando [az iot hub show-connection-string](/cli/azure/iot/hub#az-iot-hub-show-connection-string) para obtener la cadena de conexión y use su resultado para establecer una variable que utilizará al vincular los dos recursos. 

En el siguiente ejemplo se establece la variable *hubConnectionString* en el valor de la cadena de conexión de la clave principal de la directiva *iothubowner* del centro (el parámetro `--policy-name` se puede usar para especificar otra directiva). Cambie *my-sample-hub* por el nombre del IoT Hub único que eligió antes. El comando usa las opciones [query](/cli/azure/query-azure-cli) y [output](/cli/azure/format-output-azure-cli#tsv-output-format) de la CLI de Azure para extraer la cadena de conexión de la salida del comando.

```azurecli-interactive 
hubConnectionString=$(az iot hub show-connection-string --name my-sample-hub --key primary --query connectionString -o tsv)
```

Puede usar el comando `echo` para ver la cadena de conexión.

```azurecli-interactive 
echo $hubConnectionString
```

> [!NOTE]
> Estos dos comandos son válidos para un host que funcione en Bash. Si usa un shell CMD o Windows local, o un host de PowerShell, tendrá que modificar los comandos para utilizar la sintaxis correcta de dicho entorno.
>

## <a name="link-the-iot-hub-and-the-provisioning-service"></a>Vinculación del centro de IoT y el servicio de aprovisionamiento

El centro de IoT y el servicio de aprovisionamiento se vinculan con el comando [az iot dps linked-hub create](/cli/azure/iot/dps/linked-hub#az-iot-dps-linked-hub-create). 

En el ejemplo siguiente se vinculan un centro de IoT llamado *my-sample-hub* de la ubicación *westus* y una instancia de Device Provisioning Service llamada *my-sample-dps*. Cambie estos nombres por los nombres únicos de IoT Hub y Device Provisioning Service que eligió antes. El comando utiliza la cadena de conexión para el centro de IoT que se almacenó en la variable *hubConnectionString* en el paso anterior.

```azurecli-interactive 
az iot dps linked-hub create --dps-name my-sample-dps --resource-group my-sample-resource-group --connection-string $hubConnectionString --location westus
```

El comando puede tardar varios minutos en completarse.

## <a name="verify-the-provisioning-service"></a>Comprobación del servicio de aprovisionamiento

Para obtener los detalles del servicio de aprovisionamiento, use el comando [az iot dps show](/cli/azure/iot/dps#az-iot-dps-show).

El ejemplo siguiente obtiene los detalles de un servicio de aprovisionamiento llamado *my-sample-dps*. Cambie este nombre por su propio nombre de Device Provisioning Service.

```azurecli-interactive
az iot dps show --name my-sample-dps
```
El centro de IoT vinculado se muestra en la colección *properties.iotHubs*.

![Comprobación del servicio de aprovisionamiento](./media/quick-setup-auto-provision-cli/verify-provisioning-service.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido de esta colección se basan en los valores de esta. Si tiene previsto seguir trabajando con las siguientes guías de inicio rápido o tutoriales, no elimine los recursos creados en esta guía de inicio rápido. Si no planea continuar, puede utilizar los siguientes comandos para eliminar el servicio de aprovisionamiento, el centro de IoT o el grupo de recursos y todos sus recursos. Reemplace los nombres de los recursos escritos a continuación por los nombres de sus propios recursos.

Para eliminar el servicio de aprovisionamiento, ejecute el comando [az iot dps delete](/cli/azure/iot/dps#az-iot-dps-delete):

```azurecli-interactive
az iot dps delete --name my-sample-dps --resource-group my-sample-resource-group
```
Para eliminar el centro de IoT, ejecute el comando [az group delete](/cli/azure/iot/hub#az-iot-hub-delete):

```azurecli-interactive
az iot hub delete --name my-sample-hub --resource-group my-sample-resource-group
```

Para eliminar un grupo de recursos y todos sus recursos, use el comando [az group delete](/cli/azure/group#az-group-delete):

```azurecli-interactive
az group delete --name my-sample-resource-group
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado un centro de IoT y una instancia de Device Provisioning Service, y ha vinculado los dos recursos. Para aprender a usar esta configuración para aprovisionar un dispositivo simulado, vaya al inicio rápido en el que se explica cómo crear un dispositivo simulado.

> [!div class="nextstepaction"]
> [Inicio rápido para crear un dispositivo simulado](./quick-create-simulated-device.md)
