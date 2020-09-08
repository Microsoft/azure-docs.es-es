---
title: 'Tutorial: Configuración de un dispositivo para el servicio Azure IoT Hub Device Provisioning'
description: En este tutorial se indica cómo puede configurar el dispositivo para el aprovisionamiento mediante IoT Hub Device Provisioning Service (DPS) durante el proceso de fabricación del dispositivo.
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 6ff732888e416fcd51216070b3b30ed37b79e92c
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "84687131"
---
# <a name="tutorial-set-up-a-device-to-provision-using-the-azure-iot-hub-device-provisioning-service"></a>Tutorial: Configuración del dispositivo para el aprovisionamiento mediante el servicio Azure IoT Hub Device Provisioning

En el tutorial anterior, aprendió a configurar el servicio Azure IoT Hub Device Provisioning para aprovisionar automáticamente los dispositivos en su instancia de IoT Hub. Este tutorial muestra cómo configurar el dispositivo durante el proceso de fabricación, lo que permite aprovisionarlo automáticamente con IoT Hub. El dispositivo se aprovisiona en función de su [mecanismo de atestación](concepts-device.md#attestation-mechanism), tras el primer arranque y su conexión al servicio de aprovisionamiento. En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Compilar un SDK de cliente de los servicios Device Provisioning específico de la plataforma
> * Extracción de los artefactos de seguridad
> * Crear el software de registro del dispositivo

En este tutorial se presupone que ya ha creado la instancia de Device Provisioning Service y un centro de IoT, para lo que debe seguir las instrucciones del tutorial [Configuración de los recursos de la nube](tutorial-set-up-cloud.md).

En este tutorial se usan los [SDK y las bibliotecas de Azure IoT para el repositorio de C](https://github.com/Azure/azure-iot-sdk-c), que contiene el SDK de cliente del servicio Device Provisioning. El SDK proporciona actualmente compatibilidad con TPM y X.509 para los dispositivos que se ejecutan en implementaciones de Windows o Ubuntu. Este tutorial se basa en el uso de un cliente de desarrollo de Windows, que también da por supuesto un conocimiento básico de Visual Studio. 

Si no está familiarizado con el proceso de aprovisionamiento automático, no olvide revisar los [conceptos sobre aprovisionamiento automático](concepts-auto-provisioning.md) antes de continuar. 


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Los siguientes requisitos previos corresponden a un entorno de desarrollo de Windows. En el caso de Linux o macOS, consulte la sección correspondiente en [Preparación del entorno de desarrollo](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) en la documentación del SDK.

* [Visual Studio](https://visualstudio.microsoft.com/vs/) 2019 con la carga de trabajo ["Desarrollo para el escritorio con C++"](https://docs.microsoft.com/cpp/?view=vs-2019#pivot=workloads) habilitada. También se admiten Visual Studio 2015 y Visual Studio 2017.

* Tener instalada la versión más reciente de [Git](https://git-scm.com/download/).

## <a name="build-a-platform-specific-version-of-the-sdk"></a>Compilación de una versión específica de la plataforma del SDK

El SDK de cliente del servicio Device Provisioning le ayuda a implementar el software de registro del dispositivo. Pero para poder usarlo, es preciso compilar una versión del SDK específico de su plataforma de cliente de desarrollo y mecanismo de plataforma. En este tutorial, se compila un SDK que usa Visual Studio en una plataforma de desarrollo de Windows para un tipo de atestación compatible:

1. Descargue el [sistema de compilación CMake](https://cmake.org/download/).

    **Antes** de comenzar la instalación de `CMake`, es importante que los requisitos previos de Visual Studio (Visual Studio y la carga de trabajo de desarrollo de escritorio con C++) estén instalados en la máquina. Una vez que los requisitos previos están en su lugar, y se ha comprobado la descarga, instale el sistema de compilación de CMake.

2. Busque el nombre de etiqueta de la [versión más reciente](https://github.com/Azure/azure-iot-sdk-c/releases/latest) del SDK.

3. Abra un símbolo del sistema o el shell de Bash de Git. Ejecute los siguientes comandos para clonar la versión más reciente del repositorio de GitHub del [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c). Use la etiqueta que encontró en el paso anterior como valor del parámetro `-b`:

    ```cmd/sh
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Esta operación puede tardar varios minutos en completarse.

4. Cree un subdirectorio `cmake` en el directorio raíz del repositorio de Git y vaya a esa carpeta. Ejecute los siguientes comandos desde el directorio `azure-iot-sdk-c`:

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

5. Compile el SDK para su plataforma de desarrollo según los mecanismos de atestación que vaya a usar. Utilice uno de los siguientes comandos (observe también los dos caracteres de punto finales de cada comando). Al finalizar, CMake crea el subdirectorio `/cmake` con contenido específico para su dispositivo:
 
    - Para dispositivos que usan el simulador de TPM para la atestación:

        ```cmd/sh
        cmake -Duse_prov_client:BOOL=ON -Duse_tpm_simulator:BOOL=ON ..
        ```

    - Para cualquier otro dispositivo (TPM/HSM/X.509 físico o un certificado X.509 simulado):

        ```cmd/sh
        cmake -Duse_prov_client:BOOL=ON ..
        ```


Ya está listo para usar el SDK para compilar el código de registro del dispositivo. 
 
<a id="extractsecurity"></a> 

## <a name="extract-the-security-artifacts"></a>Extracción de los artefactos de seguridad 

El siguiente paso es extraer los artefactos de seguridad del mecanismo de atestación que usa el dispositivo. 

### <a name="physical-devices"></a>Dispositivos físicos 

Dependiendo de si ha compilado el SDK para usar la atestación para un dispositivo TPM/HSM físico o mediante certificados X.509, la recopilación de los artefactos de seguridad es como sigue:

- En el caso de un dispositivo con TPM, es preciso que determine la **clave de aprobación** asociada a este con la ayuda del fabricante del chip TPM. Puede derivar un único **identificador del registro** para el dispositivo TPM aplicando un algoritmo hash a la clave de aprobación.  

- Para un dispositivo X.509, deberá obtener los certificados emitidos para los dispositivos. El servicio de aprovisionamiento expone dos tipos de entradas de inscripción que controlan el acceso de los dispositivos mediante el mecanismo de atestación de X.509. Los certificados necesarios dependen de los tipos de inscripción que vaya a usar.

    - Inscripciones individuales: inscripción de un dispositivo individual concreto. Este tipo de entrada de inscripción requiere [certificados de entidad final o de "hoja"](concepts-security.md#end-entity-leaf-certificate).
    
    - Grupos de inscripción: este tipo de entrada de inscripción requiere certificados intermedios o de raíz. Para más información, consulte [Control del acceso de dispositivo al servicio de aprovisionamiento con certificados X.509](concepts-security.md#controlling-device-access-to-the-provisioning-service-with-x509-certificates).

### <a name="simulated-devices"></a>Dispositivos simulados

Dependiendo de si ha compilado el SDK para usar la atestación para un dispositivo mediante certificados TPM o X.509, la recopilación de los artefactos de seguridad es como sigue:

- Para un dispositivo con TPM simulado:

   1. Abra un símbolo del sistema en Windows, vaya al subdirectorio `azure-iot-sdk-c` y ejecute el simulador de TPM. Atiende en un socket en los puertos 2321 y 2322. No cierre esta ventana de comandos; este simulador debe estar en ejecución hasta el final de la siguiente guía de inicio rápido. 

      En el directorio `azure-iot-sdk-c`, ejecute el siguiente comando para iniciar el simulador:

      ```cmd/sh
      .\provisioning_client\deps\utpm\tools\tpm_simulator\Simulator.exe
      ```

      > [!NOTE]
      > Si usa el símbolo del sistema de Git Bash para este paso, deberá cambiar las barras diagonales inversas por barras diagonales, por ejemplo: `./provisioning_client/deps/utpm/tools/tpm_simulator/Simulator.exe`.

   1. Con Visual Studio, abra la solución generada en la carpeta *cmake* denominada `azure_iot_sdks.sln` y compílela mediante el comando "Compilar solución" del menú "Compilar".

   1. En el panel *Explorador de soluciones* de Visual Studio, vaya a la carpeta **Provision\_Tools**. Haga clic con el botón derecho en el proyecto **tpm_device_provision** y seleccione **Establecer como proyecto de inicio**. 

   1. Ejecute la solución con cualquiera de los comandos de "Inicio" del menú "Depurar". En la ventana de salida se muestra el **_identificador de registro_** y la **_clave de aprobación_** del simulador de TPM, que necesarios para la inscripción y el registro del dispositivo. Copie estos valores para usarlos más adelante. Puede cerrar esta ventana (con el identificador de registro y la clave de aprobación), pero deje que se ejecute la ventana del simulador de TPM que inició en el paso 1.

- Para un dispositivo con X.509 simulado:

  1. Con Visual Studio, abra la solución generada en la carpeta *cmake* denominada `azure_iot_sdks.sln` y compílela mediante el comando "Compilar solución" del menú "Compilar".

  1. En el panel *Explorador de soluciones* de Visual Studio, vaya a la carpeta **Provision\_Tools**. Haga clic con el botón derecho en el proyecto **dice\_device\_enrollment** y seleccione **Establecer como proyecto de inicio**. 
  
  1. Ejecute la solución con cualquiera de los comandos de "Inicio" del menú "Depurar". En la ventana de salida, escriba **i** para la inscripción individual cuando se le solicite. La ventana de salida muestra un certificado X.509 generado localmente para el dispositivo simulado. Copie en el portapapeles la salida desde *---BEGIN CERTIFICATE---* hasta *-----END CERTIFICATE-----* y asegúrese de incluir ambas líneas. Solo necesita el primer certificado de la ventana de salida.
 
  1. Cree un archivo llamado **_X509testcert.pem_** , ábralo en el editor de texto que desee y copie el contenido del portapapeles en este archivo. Guarde el archivo tal como lo usará más adelante para la inscripción de dispositivo. Cuando se ejecuta el software de registro, utiliza el mismo certificado durante el aprovisionamiento automático.    

Estos artefactos de seguridad se requieren durante la inscripción del dispositivo al servicio Device Provisioning. El servicio de aprovisionamiento espera a que el dispositivo arranque y se conecte con él en cualquier momento posterior en el tiempo. La primera vez que arranca el dispositivo, la lógica del SDK de cliente interactúa con el chip (o simulador) para extraer los artefactos de seguridad del dispositivo y comprueba el registro con el servicio Device Provisioning. 

## <a name="create-the-device-registration-software"></a>Crear el software de registro del dispositivo

El último paso es escribir una aplicación de registro que use el SDK de cliente del servicio Device Provisioning para registrar el dispositivo con el servicio IoT Hub. 

> [!NOTE]
> Para este paso supondremos que se usa un dispositivo simulado, lo que se realiza mediante la ejecución de una aplicación de registro de ejemplo de SDK desde la estación de trabajo. Sin embargo, se aplican los mismos conceptos si se compila una aplicación de registro para su implementación en un dispositivo físico. 

1. En Azure Portal, seleccione la hoja **Información general** del servicio Device Provisioning y copie el valor de **_Ámbito de id_** . El servicio genera el *ámbito del identificador* y este garantiza la exclusividad. Es inmutable y se utiliza para identificar de forma exclusiva los identificadores de registro.

    ![Extracción de información del punto de conexión del servicio Device Provisioning desde la hoja del portal](./media/tutorial-set-up-device/extract-dps-endpoints.png) 

1. En el *Explorador de soluciones* de Visual Studio de su máquina, vaya a la carpeta **Provision\_Samples**. Seleccione el proyecto de ejemplo denominado **prov\_dev\_client\_sample** y abra el archivo de origen **prov\_dev\_client\_sample.c**.

1. Asigne el valor de _Ámbito de id._ que obtuvo en el paso 1 a la variable `id_scope` (quitando los corchetes izquierdo /`[` y derecho /`]`): 

    ```c
    static const char* global_prov_uri = "global.azure-devices-provisioning.net";
    static const char* id_scope = "[ID Scope]";
    ```

    Como referencia, la variable `global_prov_uri`, que permite que la API de registro de cliente de IoT Hub `IoTHubClient_LL_CreateFromDeviceAuth` se conecte con la instancia del servicio Device Provisioning designada.

1. En la función **main()** del mismo archivo, agregue o quite el comentario de la variable `hsm_type` que coincida con el mecanismo de atestación que use el software de registro de su dispositivo (TPM o X.509): 

    ```c
    hsm_type = SECURE_DEVICE_TYPE_TPM;
    //hsm_type = SECURE_DEVICE_TYPE_X509;
    ```

1. Guarde los cambios y vuelva a compilar el ejemplo **prov\_dev\_client\_sample**, para lo que debe seleccionar "Compilar solución" en el menú "Compilar". 

1. Haga clic en el proyecto **prov\_dev\_client\_sample** de la carpeta **Provision\_Samples** y seleccione **Establecer como proyecto de inicio**. NO ejecute aún la aplicación de ejemplo.

> [!IMPORTANT]
> No ejecute o inicie el dispositivo aún. Es preciso que finalice el proceso, para lo que debe inscribir el dispositivo en el servicio antes de iniciarlo. La sección Pasos siguientes le guiará al siguiente artículo.

### <a name="sdk-apis-used-during-registration-for-reference-only"></a>API de SDK usadas en el registro (solo para referencia)

Como referencia, el SDK proporciona las siguientes API para que su aplicación las use en el registro. Estas API ayudan a su dispositivo a conectarse y registrarse en el servicio Device Provisioning al arrancar. A cambio, el dispositivo recibe la información que necesita para establecer una conexión con una instancia de IoT Hub:

```C
// Creates a Provisioning Client for communications with the Device Provisioning Client Service.  
PROV_DEVICE_LL_HANDLE Prov_Device_LL_Create(const char* uri, const char* scope_id, PROV_DEVICE_TRANSPORT_PROVIDER_FUNCTION protocol)

// Disposes of resources allocated by the provisioning Client.
void Prov_Device_LL_Destroy(PROV_DEVICE_LL_HANDLE handle)

// Asynchronous call initiates the registration of a device.
PROV_DEVICE_RESULT Prov_Device_LL_Register_Device(PROV_DEVICE_LL_HANDLE handle, PROV_DEVICE_CLIENT_REGISTER_DEVICE_CALLBACK register_callback, void* user_context, PROV_DEVICE_CLIENT_REGISTER_STATUS_CALLBACK reg_status_cb, void* status_user_ctext)

// Api to be called by user when work (registering device) can be done
void Prov_Device_LL_DoWork(PROV_DEVICE_LL_HANDLE handle)

// API sets a runtime option identified by parameter optionName to a value pointed to by value
PROV_DEVICE_RESULT Prov_Device_LL_SetOption(PROV_DEVICE_LL_HANDLE handle, const char* optionName, const void* value)
```

También puede darse cuenta de que necesita refinar la aplicación de registro de cliente del servicio Device Provisioning, para lo que debe usar al principio un dispositivo simulado, y una configuración del servicio de prueba. Una vez que la aplicación está funcionando en el entorno de prueba, puede compilarla para el dispositivo concreto y copiar el archivo ejecutable en la imagen del dispositivo. 

## <a name="clean-up-resources"></a>Limpieza de recursos

En este momento, los servicios Device Provisioning e IoT Hub se ejecutan en el portal. Si desea abandonar la configuración del aprovisionamiento del dispositivo o retrasar la finalización de esta serie de tutoriales, es aconsejable apagarlos para no incurrir en costos innecesarios.

1. Desde el menú de la izquierda en Azure Portal, haga clic en **Todos los recursos** y seleccione el servicio Device Provisioning. En la parte superior de la hoja **Todos los recursos**, haga clic en **Eliminar**.  
1. Desde el menú de la izquierda en Azure Portal, haga clic en **Todos los recursos** y seleccione su centro de IoT. En la parte superior de la hoja **Todos los recursos**, haga clic en **Eliminar**.  

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Compilar un SDK de cliente del servicio Device Provisioning específico de la plataforma
> * Extracción de los artefactos de seguridad
> * Crear el software de registro del dispositivo

Avance hasta el siguiente tutorial para aprender a aprovisionar el dispositivo en su instancia de IoT Hub inscribiéndolo en el servicio Azure IoT Hub Device Provisioning para su aprovisionamiento automático.

> [!div class="nextstepaction"]
> [Aprovisionar el dispositivo en la instancia de IoT Hub](tutorial-provision-device-to-hub.md)
