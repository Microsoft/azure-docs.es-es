---
title: Incorporación de un dispositivo Azure IoT Edge a Azure IoT Central | Microsoft Docs
description: Como operador, agregue un dispositivo Azure IoT Edge a la aplicación Azure IoT Central.
author: rangv
ms.author: rangv
ms.date: 05/29/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.custom:
- mvc
- device-developer
- iot-edge
ms.openlocfilehash: cc612d5dc21594da855799ed965effeb4b547d00
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90018669"
---
# <a name="tutorial-add-an-azure-iot-edge-device-to-your-azure-iot-central-application"></a>Tutorial: Incorporación de un dispositivo Azure IoT Edge a la aplicación Azure IoT Central

*Este artículo se aplica a operadores, generadores de soluciones y desarrolladores de dispositivos.*

En este tutorial se explica cómo configurar y agregar un dispositivo Azure IoT Edge a la aplicación de Azure IoT Central. En el tutorial se usa una máquina virtual Linux (VM) habilitada para IoT Edge para simular un dispositivo IoT Edge. El dispositivo IoT Edge usa un módulo que genera datos de telemetría ambiental simulados. Puede ver los datos de telemetría en un panel en la aplicación de IoT Central.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una plantilla de dispositivo para un dispositivo IoT Edge
> * Crear un dispositivo IoT Edge en IoT Central
> * Implementar un dispositivo IoT Edge simulado en una máquina virtual Linux

## <a name="prerequisites"></a>Requisitos previos

Complete el inicio rápido [Creación de una aplicación de Azure IoT Central](./quick-deploy-iot-central.md) para crear una aplicación de IoT Central mediante la plantilla **Aplicación personalizada > Aplicación personalizada**.

Para completar los pasos de este tutorial, deberá tener una suscripción de Azure activa.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Descargue el archivo de manifiesto de IoT Edge desde GitHub. Haga clic con el botón derecho en el siguiente vínculo y seleccione **Guardar vínculo como**: [EnvironmentalSensorManifest.json](https://raw.githubusercontent.com/Azure-Samples/iot-central-docs-samples/master/iotedge/EnvironmentalSensorManifest.json)

## <a name="create-device-template"></a>Creación de la plantilla de dispositivo

En esta sección, creará una plantilla de dispositivo de IoT Central para un dispositivo IoT Edge. Para empezar a trabajar, importe un manifiesto de IoT Edge y, a continuación, modifique la plantilla para agregar las definiciones de telemetría y las vistas:

### <a name="import-manifest-to-create-template"></a>Importación de un manifiesto para crear una plantilla

Para crear una plantilla de dispositivo a partir de un manifiesto de IoT Edge:

1. En la aplicación de IoT Central, vaya a **Plantillas de dispositivo** y seleccione **+ Nueva**.

1. En la página **Seleccionar tipo de plantilla**, seleccione el icono **Azure IoT Edge**. Después, seleccione **Next: Customize** (Personalizar)

1. En la página **Carga de un manifiesto de implementación de Azure IoT Edge**, escriba *Dispositivo perimetral de sensor ambiental* como nombre de la plantilla de dispositivo. A continuación, seleccione **Examinar** para cargar el archivo **EnvironmentalSensorManifest.json** que descargó anteriormente. Después, seleccione **Next: Review** (Siguiente: revisar).

1. En la página **Revisar**, seleccione **Crear**.

1. Seleccione la interfaz **Administrar** en el módulo **SimulatedTemperatureSensor** para ver las dos propiedades definidas en el manifiesto:

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/imported-manifest.png" alt-text="Plantilla de dispositivo creada a partir de un manifiesto de IoT Edge":::

### <a name="add-telemetry-to-manifest"></a>Adición de telemetría al manifiesto

Un manifiesto de IoT Edge no define la telemetría que envía un módulo. Debe agregar las definiciones de telemetría a la plantilla de dispositivo en IoT Central. El módulo **SimulatedTemperatureSensor** envía mensajes de telemetría que tienen un aspecto similar al del siguiente código JSON:

```json
{
  "machine": {
    "temperature": 75.0,
    "pressure": 40.2
  },
  "ambient": {
    "temperature": 23.0,
    "humidity": 30.0
  },
  "timeCreated": ""
}
```

Para agregar las definiciones de telemetría a la plantilla de dispositivo:

1. Seleccione la interfaz **Administrar** en la plantilla **Environmental Sensor Edge Device**.

1. Seleccione **+ Agregar funcionalidad**. Escriba *machine* (máquina) como **Nombre para mostrar** y asegúrese de que **Tipo de funcionalidad** es **Telemetría**.

1. Seleccione **Objeto** como tipo de esquema y, a continuación, seleccione **Definir**. En la página de definición de objeto, agregue *temperature* (temperatura) y *pressure* (presión) como atributos de tipo **Doble** y, a continuación, seleccione **Aplicar**.

1. Seleccione **+ Agregar funcionalidad**. Escriba *ambient* (ambiental) como **Nombre para mostrar** y asegúrese de que **Tipo de funcionalidad** es **Telemetría**.

1. Seleccione **Objeto** como tipo de esquema y, a continuación, seleccione **Definir**. En la página de definición de objeto, agregue *temperature* (temperatura) y *humidity* (humedad) como atributos de tipo **Doble** y, a continuación, seleccione **Aplicar**.

1. Seleccione **+ Agregar funcionalidad**. Escriba *timeCreated* (hora de creación) como **Nombre para mostrar** y asegúrese de que **Tipo de funcionalidad** es **Telemetría**.

1. Seleccione **DateTime** (Fecha y hora) como el tipo de esquema.

1. Seleccione **Guardar** para actualizar la plantilla.

La interfaz **Administrar** ahora incluye los tipos de telemetría **machine**, **ambient** y **timeCreated**:

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/manage-interface.png" alt-text="Interfaz con los tipos de telemetría machine y ambient":::

### <a name="add-views-to-template"></a>Adición de vistas a la plantilla

La plantilla de dispositivo todavía no tiene una vista que permita al operador ver la telemetría del dispositivo IoT Edge. Para agregar una vista a la plantilla de dispositivo:

1. Seleccione **Vistas** en la plantilla **Environmental Sensor Edge Device**.

1. En la página **Seleccionar para agregar una nueva vista**, seleccione el icono **Visualizar el dispositivo**.

1. Cambie el nombre de la vista a *View IoT Edge device telemetry* (Ver telemetría del dispositivo IoT Edge).

1. Seleccione los tipos de telemetría **ambient** y **machine**. A continuación, seleccione **Agregar icono**.

1. Seleccione **Guardar** para guardar la vista **View IoT Edge device telemetry**.

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/template-telemetry-view.png" alt-text="Plantilla de dispositivo con vista de telemetría":::

### <a name="publish-the-template"></a>Publicación de la plantilla

Antes de poder agregar un dispositivo que use la plantilla **Environmental Sensor Edge Device**, debe publicar la plantilla.

Vaya a la plantilla **Environmental Sensor Edge Device** y seleccione **Publicar**. En el panel **Publicar esta plantilla de dispositivo en la aplicación**, seleccione **Publicar** para publicar la plantilla:

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/publish-template.png" alt-text="Publicación de la plantilla de dispositivo":::

## <a name="add-iot-edge-device"></a>Adición de un dispositivo IoT Edge

Ahora que ha publicado la plantilla **Environmental Sensor Edge Device**, puede agregar un dispositivo a la aplicación de IoT Central:

1. En la aplicación de IoT Central, vaya a la página **Dispositivos** y seleccione **Environmental Sensor Edge Device** en la lista de plantillas disponibles.

1. Seleccione **+ Nuevo** para agregar un nuevo dispositivo desde la plantilla. En la página **Crear nuevo dispositivo**, seleccione **Crear**.

Ahora tiene un nuevo dispositivo con el estado de **Registrado**:

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/new-device.png" alt-text="Nuevo dispositivo registrado":::

### <a name="get-the-device-credentials"></a>Obtención de las credenciales del dispositivo

Cuando implemente el dispositivo IoT Edge más adelante en este tutorial, necesitará las credenciales que permiten al dispositivo conectarse a la aplicación de IoT Central. Para obtener las credenciales del dispositivo:

1. En la página **Dispositivo**, seleccione el dispositivo que ha creado.

1. Seleccione **Conectar**.

1. En la página **Conexión del dispositivo**, tome nota de los valores **Ámbito del identificador**, **Identificador de dispositivo** y **Clave principal**. Utilizará estos valores más adelante.

1. Seleccione **Cerrar**.

Ya ha terminado de configurar la aplicación de IoT Central para habilitar la conexión de un dispositivo IoT Edge.

## <a name="deploy-an-iot-edge-device"></a>Implementación de un dispositivo IoT Edge

En este tutorial se usa una máquina virtual Linux habilitada para Azure IoT Edge, creada en Azure para simular un dispositivo IoT Edge. Para crear la VM habilitada para IoT Edge en su suscripción de Azure, haga clic en:

[![Botón Implementar en Azure de iotedge-vm-deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fazure%2Fiotedge-vm-deploy%2Fmaster%2FedgeDeploy.json)

En la página **Implementación personalizada**:

1. Seleccione su suscripción a Azure.

1. Seleccione **Crear nuevo** para crear un nuevo grupo de recursos denominado *central-edge-rg*.

1. Elija una región cercana.

1. Agregue un **prefijo de etiqueta DNS** único, como *contoso-central-edge*.

1. Elija un nombre de usuario administrador para la máquina virtual.

1. Escriba *temp* como la cadena de conexión. Más adelante, configurará el dispositivo para la conexión mediante DPS.

1. Acepte los valores predeterminados de tamaño de VM, versión de Ubuntu y ubicación.

1. Seleccione **contraseña** como tipo de autenticación.

1. Escriba una contraseña para la VM.

1. Después, seleccione **Revisar y crear**.

1. Revise sus opciones y luego seleccione **Crear**:

    :::image type="content" source="media/tutorial-add-edge-as-leaf-device/vm-deployment.png" alt-text="Crear una VM de IoT Edge":::

La implementación tarda un par de minutos en completarse. Una vez finalizada la implementación, vaya al grupo de recursos **central-edge-rg** en Azure Portal.

### <a name="configure-the-iot-edge-vm"></a>Configuración de la VM de IoT Edge

Para configurar IoT Edge en la VM para que use DPS para registrarse y conectarse a la aplicación de IoT Central:

1. En el grupo de recursos **contoso-edge-rg**, seleccione la instancia de máquina virtual.

1. En la sección **Soporte técnico y solución de problemas**, seleccione **Serial Console**. Si se le pide que configure los diagnósticos de arranque, siga las instrucciones del portal.

1. Presione **Entrar** para ver el símbolo del sistema `login:`. Escriba su nombre de usuario y contraseña para iniciar sesión.

1. Ejecute el siguiente comando para comprobar la versión del entorno de ejecución de IoT Edge. En el momento de escribir este documento, la versión es la 1.0.9.1:

    ```bash
    sudo iotedge --version
    ```

1. Use el editor `nano` para abrir el archivo config.yaml de IoT Edge:

    ```bash
    sudo nano /etc/iotedge/config.yaml
    ```

1. Desplácese hacia abajo hasta que vea `# Manual provisioning configuration`. Convierta en comentario las tres líneas siguientes, tal como se muestra en el siguiente fragmento de código:

    ```yaml
    # Manual provisioning configuration
    #provisioning:
    #  source: "manual"
    #  device_connection_string: "temp"
    ```

1. Desplácese hacia abajo hasta que vea `# DPS symmetric key provisioning configuration`. Quite las marcas de comentario de las siguientes ocho líneas, como se muestra en el siguiente fragmento de código:

    ```yaml
    # DPS symmetric key provisioning configuration
    provisioning:
      source: "dps"
      global_endpoint: "https://global.azure-devices-provisioning.net"
      scope_id: "{scope_id}"
      attestation:
        method: "symmetric_key"
        registration_id: "{registration_id}"
        symmetric_key: "{symmetric_key}"
    ```

    > [!TIP]
    > Asegúrese de que no quede un espacio delante de `provisioning:`

1. Reemplace `{scope_id}` por el valor de **Ámbito del identificador** que anotó anteriormente.

1. Reemplace `{registration_id}` por el valor de **Identificador de dispositivo** que anotó anteriormente.

1. Reemplace `{symmetric_key}` por el valor de **Clave principal** que anotó anteriormente.

1. Guarde los cambios (**Ctrl-O**) y salga (**Ctrl-X**) del editor `nano`.

1. Ejecute el siguiente comando para reiniciar el demonio de IoT Edge:

    ```bash
    sudo systemctl restart iotedge
    ```

1. Ejecute el siguiente comando para comprobar el estado de los módulos de IoT Edge:

    ```bash
    iotedge list
    ```

    La siguiente salida de ejemplo muestra los módulos en ejecución:

    ```bash
    NAME                        STATUS           DESCRIPTION      CONFIG
    SimulatedTemperatureSensor  running          Up 20 seconds    mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0
    edgeAgent                   running          Up 27 seconds    mcr.microsoft.com/azureiotedge-agent:1.0
    edgeHub                     running          Up 22 seconds    mcr.microsoft.com/azureiotedge-hub:1.0
    ```

    > [!TIP]
    > Es posible que tenga que esperar a que todos los módulos empiecen a ejecutarse.

## <a name="view-the-telemetry"></a>Visualización de la telemetría

El dispositivo IoT Edge simulado ahora está en ejecución en la máquina virtual. En la aplicación de IoT Central, el estado del dispositivo ahora es de **Aprovisionado** en la página **Dispositivos**:

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/provisioned-device.png" alt-text="Dispositivo IoT Edge aprovisionado":::

Puede ver los datos de telemetría del dispositivo en la página **Ver datos de telemetría del dispositivo IoT Edge**:

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/device-telemetry-view.png" alt-text="Telemetría del dispositivo":::

En la página **Módulos** se muestra el estado de los módulos IoT Edge en el dispositivo:

:::image type="content" source="media/tutorial-add-edge-as-leaf-device/edge-module-status.png" alt-text="Estado de los módulos del dispositivo":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Si tiene previsto seguir trabajando con la VM IoT Edge, puede mantener y reutilizar los recursos que usó en este tutorial. De lo contrario, puede eliminar los recursos que creó para este tutorial a fin de evitar cargos adicionales:

* Para eliminar la VM IoT Edge y sus recursos asociados, elimine el grupo de recursos **contoso-edge-rg** en Azure Portal.
* Para eliminar la aplicación IoT Central, vaya a la página **Su aplicación** en la sección **Administración** de la aplicación y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

Como desarrollador de dispositivos, ahora que ha aprendido a trabajar y a administrar dispositivos IoT Edge en IoT Central, este es el siguiente paso sugerido:

> [!div class="nextstepaction"]
> [Desarrollar módulos IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)

Como desarrollador de soluciones u operador, ahora que ha aprendido a trabajar y a administrar dispositivos IoT Edge en IoT Central, este es el siguiente paso sugerido:

> [!div class="nextstepaction"]
> [Usar grupos de dispositivos para analizar la telemetría del dispositivo](./tutorial-use-device-groups.md)
