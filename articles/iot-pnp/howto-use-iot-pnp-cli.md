---
title: Uso de la extensión de Azure IoT para que la CLI de Azure interactúe con los dispositivos de la versión preliminar de IoT Plug and Play | Microsoft Docs
description: Instale la extensión de Azure IoT para la CLI de Azure y úsela para interactuar con los dispositivos IoT Plug and Play conectados a su centro de IoT.
author: dominicbetts
ms.author: dobett
ms.date: 07/20/2020
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: dadb1f044547acd6e5f0d274143123e89d7dae46
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475488"
---
# <a name="install-and-use-the-azure-iot-extension-for-the-azure-cli"></a>Instalación y uso de la extensión de Azure IoT para la CLI de Azure

[La CLI de Azure](https://docs.microsoft.com/cli/azure?view=azure-cli-latest) es una herramienta de línea de comandos multiplataforma y de código abierto para la administración de recursos de Azure, como IoT Hub. La CLI de Azure está disponible en Windows, Linux y macOS. Permite administrar los recursos de Azure IoT Hub, las instancias de Device Provisioning Service y los centros vinculados sin instalar ninguna extensión.

La extensión de Azure IoT para la CLI de Azure es una herramienta de línea de comandos para interactuar con los dispositivos de la versión preliminar de IoT Plug and Play y probarlos. Puede usar la extensión para lo siguiente:

- Conectar con un dispositivo.
- Ver la telemetría que envía el dispositivo.
- Trabajar con las propiedades del dispositivo.
- Llamar a comandos de dispositivo.

En este artículo aprenderá a:

- Instalar y configurar la extensión de Azure IoT para la CLI de Azure.
- Usar la extensión para interactuar con los dispositivos y probarlos.

## <a name="install-azure-iot-extension-for-the-azure-cli"></a>Instalación de la extensión de Azure IoT para la CLI de Azure

### <a name="step-1---install-the-azure-cli"></a>Paso 1: Instalación de la CLI de Azure

Siga las [instrucciones de instalación](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) para instalar la CLI de Azure en su entorno. Para obtener la mejor experiencia, la CLI de Azure debe ser de la versión 2.9.1 o posterior. Use `az -–version` para asegurarse.

### <a name="step-2---install-iot-extension"></a>Paso 2: Instalación de la extensión de IoT

En el [archivo Léame de la extensión de IoT](https://github.com/Azure/azure-iot-cli-extension) se describen varias maneras de instalarla. La manera más sencilla es ejecutar `az extension add --name azure-iot`. Después de la instalación, puede usar `az extension list` para validar las extensiones instaladas actualmente o `az extension show --name azure-iot` para ver los detalles acerca de la extensión de IoT. En el momento de escribir este documento, el número de versión de la extensión es `0.9.7`.

Para quitar la extensión, puede usar `az extension remove --name azure-iot`.

## <a name="use-azure-iot-extension-for-the-azure-cli"></a>Uso de la extensión de Azure IoT para la CLI de Azure

### <a name="prerequisites"></a>Requisitos previos

Para iniciar sesión en su suscripción a Azure, ejecute el siguiente comando:

```azurecli
az login
```

Para usar la extensión de Azure IoT para la CLI de Azure, necesita:

- Un centro de Azure IoT. Hay muchas maneras de agregar un centro de IoT a la suscripción de Azure, por ejemplo, [crear un centro de IoT mediante la CLI de Azure](../iot-hub/iot-hub-create-using-cli.md). Necesitará la cadena de conexión del centro de IoT para ejecutar los comandos de la extensión de Azure IoT. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

- Un dispositivo registrado en su centro de IoT. Puede usar el siguiente comando de CLI de Azure para registrar un dispositivo: asegúrese de reemplazar los marcadores de posición `{YourIoTHubName}` y `{YourDeviceID}` por sus valores:

    ```azurecli
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id {YourDeviceID}
    ```

### <a name="interact-with-a-device"></a>Interactuación con un dispositivo

Puede usar la extensión para ver e interactuar con los dispositivos de IoT Plug and Play que estén conectados a un centro de IoT. La extensión funciona con el gemelo digital que representa el dispositivo de IoT Plug and Play.

#### <a name="list-devices"></a>Enumerar los dispositivos.

Enumerar todos los dispositivos de una instancia de IoT Hub:

```azurecli
az iot hub device-identity list --hub-name {YourIoTHubName}
```

#### <a name="view-digital-twin"></a>Ver gemelo digital

Ver el gemelo digital de un dispositivo:

```azurecli
az iot pnp twin show -n {iothub_name} -d {device_id}
```

#### <a name="properties"></a>Propiedades

Establecer el valor de una propiedad de lectura y escritura:

```azurecli
az iot pnp twin update --hub-name {iothub_name} --device-id {device_id} --json-patch '{"op":"add", "path":"/thermostat1/targetTemperature", "value": 54}'
```

#### <a name="commands"></a>Comandos:

Invocar un comando:

```azurecli
az iot pnp twin invoke-command --cn getMaxMinReport -n {iothub_name} -d {device_id} --component-path thermostat1
```

#### <a name="digital-twin-events"></a>Eventos gemelos digitales

Supervise todos los eventos gemelos digitales de IoT Plug and Play desde un dispositivo e interfaz específicos que vayan al grupo de consumidores del centro de eventos **$Default**:

```azurecli
az iot hub monitor-events -n {iothub_name} -d {device_id} -i {interface_id}
```

### <a name="manage-models-in-the-model-repository"></a>Administración de modelos en el repositorio de modelos

Puede usar los comandos del repositorio de modelos de la CLI de Azure para administrar los modelos en el repositorio.

#### <a name="create-model-repository"></a>Creación de un repositorio de modelos

Cree un nuevo repositorio de empresa de IoT Plug and Play para su inquilino si es el primer usuario del inquilino:

```azurecli
az iot pnp repo create
```

#### <a name="manage-model-repository-tenant-roles"></a>Administración de los roles de inquilino del repositorio de modelos

Cree una asignación de roles para un usuario o una entidad de servicio en un recurso específico.

Por ejemplo, dé a user@consoso.com el rol de **ModelsCreator** para el inquilino:

```azurecli
az iot pnp role-assignment create --resource-id {tenant_id} --resource-type Tenant --subject-id {user@contoso.com} --subject-type User --role ModelsCreator
```

O dele a user@consoso.com el rol de **ModelAdministrator** para un modelo específico:

```azurecli
az iot pnp role-assignment create --resource-id {model_id} --resource-type Model --subject-id {user@contoso.com} --subject-type User --role ModelAdministrator
```

#### <a name="create-a-model"></a>Crear un modelo

Cree un nuevo modelo en el repositorio de la empresa:

```azurecli
az iot pnp model create --model {model_json or path_to_file}
```

#### <a name="search-a-model"></a>Búsqueda de un modelo

Consiga una lista de los modelos que coinciden con una palabra clave específica:

```azurecli
az iot pnp model list -q {search_keyword}
```

#### <a name="publish-a-model"></a>Publicación de modelos

Publique un modelo de dispositivo ubicado en el repositorio de la empresa en el repositorio público.

Por ejemplo, haga público el modelo con el identificador `dtmi:com:example:ClimateSensor;1`:

```azurecli
az iot pnp model publish --dtmi "dtmi:com:example:ClimateSensor;1"
```

Para publicar un modelo, deben cumplirse los siguientes requisitos:

- El inquilino de la empresa u organización tiene que ser un partner de Microsoft. 
- El usuario o la entidad de servicio tienen que ser miembros del rol **Publisher** del inquilino del repositorio.

## <a name="next-steps"></a>Pasos siguientes

En este artículo de procedimientos, ha aprendido a instalar y usar la extensión de Azure IoT para la CLI de Azure a fin de interactuar con los dispositivos IoT Plug and Play. El siguiente paso sugerido es obtener información sobre cómo usar [Azure IoT Explorer con los dispositivos](./howto-use-iot-explorer.md).
