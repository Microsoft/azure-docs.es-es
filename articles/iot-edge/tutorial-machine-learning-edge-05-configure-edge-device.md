---
title: 'Tutorial: Configuración de un dispositivo IoT Edge: Machine Learning en Azure IoT Edge'
description: En este tutorial configurará una máquina virtual de Azure que ejecuta Linux como un dispositivo Azure IoT Edge que actúa como puerta de enlace transparente.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 2/5/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: amqp
ms.openlocfilehash: 353ed321ce3b6161b28bf67d852a81f809880603
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "81733006"
---
# <a name="tutorial-configure-an-iot-edge-device"></a>Tutorial: Configure un dispositivo IoT Edge

> [!NOTE]
> Este artículo forma parte de un tutorial sobre el uso de Azure Machine Learning en IoT Edge. Si ha llegado directamente a este artículo, le recomendamos que comience con el [primer el artículo](tutorial-machine-learning-edge-01-intro.md) de la serie para obtener los mejores resultados.

En este artículo, se configurará una máquina virtual de Azure que ejecuta Linux para que sea un dispositivo IoT Edge que actúa como puerta de enlace transparente. La configuración de una puerta de enlace transparente permite que los dispositivos se conecten a Azure IoT Hub a través de la puerta de enlace sin necesidad de saber que dicha puerta existe. Al mismo tiempo, un usuario que interactúa con los dispositivos de Azure IoT Hub no es consciente del dispositivo de puerta de enlace intermedio. En última instancia, se agregará análisis de Edge a nuestro sistema mediante la adición de módulos de IoT Edge a la puerta de enlace transparente.

Los pasos descritos en este artículo los realiza normalmente un desarrollador en la nube.

## <a name="create-certificates"></a>Creación de certificados

Para que un dispositivo funcione como una puerta de enlace, debe poder conectarse de forma segura a los dispositivos de bajada. Azure IoT Edge le permite usar una infraestructura de clave pública (PKI) para configurar conexiones seguras entre los dispositivos. En este caso, vamos a permitir que un dispositivo IoT de bajada se conecte a un dispositivo IoT Edge que actúa como puerta de enlace transparente. Para mantener una seguridad razonable, el dispositivo de bajada debe confirmar la identidad del dispositivo IoT Edge. Para obtener más información acerca de cómo los dispositivos IoT Edge usan certificados, consulte [Información de uso de los certificados de Azure IoT Edge](iot-edge-certs.md).

En esta sección, crearemos los certificados autofirmados con una imagen de Docker que después se compila y ejecuta. Hemos optado por utilizar una imagen de Docker para completar este paso, ya que reduce significativamente el número de pasos necesarios para crear los certificados en el equipo de desarrollo de Windows. Consulte [Creación de certificados de demostración para probar las características del dispositivo IoT Edge](how-to-create-test-certificates.md) para comprender lo que se automatiza con la imagen de Docker.

1. Inicie sesión en la máquina virtual de desarrollo.

2. Cree una nueva carpeta con la ruta de acceso y el nombre `c:\edgeCertificates`.

3. Inicie **Docker para Windows** desde el menú Inicio de Windows si es que aún no se está ejecutando.

4. Abra Visual Studio Code.

5. Seleccione **Archivo** > **Abrir carpeta...** y elija **C:\\origen\\IoTEdgeAndMlSample\\CreateCertificates**.

6. En el panel Explorador, haga clic con el botón derecho en el archivo **dockerfile** y elija **Compilar imagen**.

7. En el cuadro de diálogo, acepte el valor predeterminado para el nombre de la imagen y la etiqueta: **createcertificates: latest**.

    ![Creación de certificados en Visual Studio Code](media/tutorial-machine-learning-edge-05-configure-edge-device/create-certificates.png)

8. Espere a que la compilación finalice.

    > [!NOTE]
    > Es posible que vea una advertencia sobre una clave pública que falta. Puede hacer caso omiso de esta advertencia sin ningún problema. Del mismo modo, verá una advertencia de seguridad que le recomienda comprobar o restablecer los permisos en la imagen, puede hacer caso omiso para esta imagen.

9. En la ventana de terminal de Visual Studio Code, ejecute el contenedor createcertificates.

    ```cmd
    docker run --name createcertificates --rm -v c:\edgeCertificates:/edgeCertificates createcertificates /edgeCertificates
    ```

10. Docker le solicitará el acceso a la unidad **c:\\** . Seleccione **Share it** (Compartir).

11. Escriba sus credenciales cuando se le solicite.

12. Una vez que el contenedor termina de ejecutarse, compruebe los siguientes archivos en **c:\\edgeCertificates**:

    * c:\\edgeCertificates\\certs\\azure-iot-test-only.root.ca.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device-full-chain.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device.cert.pfx
    * c:\\edgeCertificates\\private\\new-edge-device.key.pem

## <a name="upload-certificates-to-azure-key-vault"></a>Carga de certificados a Azure Key Vault

Para almacenar nuestros certificados de forma segura y para que sean accesibles desde varios dispositivos, se cargarán en Azure Key Vault. Como puede ver en la lista anterior, tenemos dos tipos de archivos de certificado: PFX y PEM. El archivo PFX se tratará como certificados de Key Vault para cargarse en Key Vault. Los archivos PEM son texto sin formato y se tratarán como secretos de Key Vault. Usaremos la instancia de Key Vault asociada con el área de trabajo de Azure Machine Learning mediante la ejecución de [Azure Notebooks](tutorial-machine-learning-edge-04-train-model.md#run-azure-notebooks).

1. En [Azure Portal](https://portal.azure.com), vaya al área de trabajo de Azure Machine Learning.

2. En la página de información general del área de trabajo de Azure Machine Learning, busque el nombre de la instancia de **Key Vault**.

    ![Copia del nombre de la instancia de Key Vault](media/tutorial-machine-learning-edge-05-configure-edge-device/find-key-vault-name.png)

3. En el equipo de desarrollo, cargue los certificados a Key Vault. Reemplace **\<subscriptionId\>** y **\<keyvaultname\>** por la información del recurso.

    ```powershell
    c:\source\IoTEdgeAndMlSample\CreateCertificates\upload-keyvaultcerts.ps1 -SubscriptionId <subscriptionId> -KeyVaultName <keyvaultname>
    ```

4. Si se le solicita, inicie sesión en Azure.

5. El script se ejecutará durante unos minutos y, como salida, proporcionará una lista de las nuevas entradas de Key Vault.

    ![Salida del script de Key Vault](media/tutorial-machine-learning-edge-05-configure-edge-device/key-vault-entries-output.png)

## <a name="create-iot-edge-device"></a>Crear dispositivo de IoT Edge

Para conectar un dispositivo Azure IoT Edge a una instancia de IoT Hub, debe crearse primero una identidad para el dispositivo en el concentrador. Tomamos la cadena de conexión de la identidad del dispositivo en la nube y la usamos para configurar el tiempo de ejecución en nuestro dispositivo IoT Edge. Una vez que un dispositivo configurado se conecta a la central, es posible implementar módulos y enviar mensajes. También se puede cambiar la configuración del dispositivo IoT Edge físico modificando la identidad del dispositivo correspondiente en IoT Hub.

Para este tutorial, creamos la nueva identidad del dispositivo con Visual Studio Code. También puede llevar a cabo estos pasos con [Azure Portal](how-to-register-device.md#register-in-the-azure-portal) o la [CLI de Azure](how-to-register-device.md#register-with-the-azure-cli).

1. Abra Visual Studio Code en el equipo de desarrollo.

2. Expanda la sección **Azure IoT Hub** de la vista Explorador de Visual Studio Code.

3. Haga clic en el botón de puntos suspensivos y seleccione **Crear dispositivo IoT Edge**.

4. Asigne un nombre al dispositivo. Para mayor comodidad, usamos el nombre **aaTurbofanEdgeDevice** para que se ordene en la parte superior de la lista de dispositivos.

5. El nuevo dispositivo aparecerá en la lista de dispositivos.

    ![Visualización del dispositivo aaTurbofanEdgeDevice en el explorador de VS Code](media/tutorial-machine-learning-edge-05-configure-edge-device/iot-hub-devices-list.png)

## <a name="deploy-azure-virtual-machine"></a>Implementación de una máquina virtual de Azure

Usamos la imagen [Azure IoT Edge en Ubuntu](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft_iot_edge.iot_edge_vm_ubuntu?tab=Overview) de Azure Marketplace para crear nuestro dispositivo IoT Edge para este tutorial. La imagen Azure IoT Edge en Ubuntu instala el entorno de ejecución de Azure IoT Edge más reciente y sus dependencias durante el inicio. Implementamos la VM con un script de PowerShell, `Create-EdgeVM.ps1`; una plantilla de Resource Manager, `IoTEdgeVMTemplate.json`; y un script de shell, `install packages.sh`.

### <a name="enable-programmatic-deployment"></a>Habilitación de implementaciones mediante programación

Para usar la imagen de Marketplace en una implementación con scripts, es necesario habilitar la implementación mediante programación para la imagen.

1. Inicie sesión en Azure Portal.

1. Seleccione **Todos los servicios**.

1. En la barra de búsqueda, escriba y seleccione **Marketplace**.

1. En la barra de búsqueda de Marketplace, escriba y seleccione **Azure IoT Edge en Ubuntu**.

1. Seleccione el hipervínculo **Introducción** para realizar la implementación mediante programación.

1. Seleccione el botón **Habilitar** y, después, **Guardar**.

    ![Habilitación de implementaciones mediante programación para VM](media/tutorial-machine-learning-edge-05-configure-edge-device/deploy-ubuntu-vm.png)

1. Verá una notificación que indica que la operación se realizó correctamente.

### <a name="create-virtual-machine"></a>Crear máquina virtual

A continuación, ejecute el script para crear la máquina virtual para el dispositivo IoT Edge.

1. Abra una ventana de PowerShell y navegue hasta el directorio **EdgeVM**.

    ```powershell
    cd c:\source\IoTEdgeAndMlSample\EdgeVM
    ```

2. Ejecute el script para crear la máquina virtual.

    ```powershell
    .\Create-EdgeVm.ps1
    ```

3. Cuando se le solicite, proporcione valores para cada parámetro. Para la suscripción, el grupo de recursos y la ubicación, le recomendamos que use los mismos datos que los que usó para todos los recursos a lo largo de este tutorial.

    * **Id. de suscripción de Azure**: lo encontrará en Azure Portal
    * **Nombre del grupo de recursos**: nombre fácil de recordar para agrupar los recursos de este tutorial
    * **Ubicación**: ubicación de Azure donde se creará la máquina virtual. Por ejemplo, westus2 o northeurope. Para obtener más información, vea todas las [ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/).
    * **AdminUsername**: el nombre de la cuenta de administrador que usará para iniciar sesión en la máquina virtual
    * **AdminPassword**: la contraseña que se puede establecer para AdminUsername en la máquina virtual

4. Para que el script pueda configurar la VM, deberá iniciar sesión en Azure con las credenciales asociadas con la suscripción de Azure que está usando.

5. El script confirma la información para crear la máquina virtual. En este caso, seleccione **y** o **ENTRAR** para continuar.

6. El script se ejecuta durante varios minutos mientras realiza los pasos siguientes:

    * Creación del grupo de recursos si todavía no existe
    * Creación de la máquina virtual
    * Adición de excepciones de grupo de seguridad de red para la máquina virtual para los puertos 22 (SSH), 5671 (AMQP), 5672 (AMPQ) y 443 (TLS)
    * Instale la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli-apt?view=azure-cli-latest))

7. El script da como resultado la cadena de conexión de SSH para conectarse a la VM. Copie la cadena de conexión para el paso siguiente.

    ![Copia de la cadena de conexión de SSH para la VM](media/tutorial-machine-learning-edge-05-configure-edge-device/vm-ssh-connection-string.png)

## <a name="connect-to-your-iot-edge-device"></a>Conexión al dispositivo IoT Edge

En las secciones siguientes se configura la máquina virtual de Azure que hemos creado. El primer paso es conectarse a la máquina virtual.

1. Abra un símbolo del sistema y pegue la cadena de conexión de SSH que copió desde la salida del script. Escriba su propia información de nombre de usuario, sufijo y región según los valores proporcionados para el script de PowerShell en la sección anterior.

    ```cmd
    ssh -l <username> iotedge-<suffix>.<region>.cloudapp.azure.com
    ```

2. Cuando se le solicite validar la autenticidad del host, escriba **Sí** y seleccione **ENTRAR**.

3. Cuando se le solicite, proporcione la contraseña.

4. Ubuntu muestra un mensaje de bienvenida y, después, verá una solicitud como `<username>@<machinename>:~$`.

## <a name="download-key-vault-certificates"></a>Descarga de los certificados de Key Vault

Anteriormente en este artículo, cargamos certificados en Key Vault para que estén disponibles para el dispositivo IoT Edge y para nuestro dispositivo hoja. El dispositivo hoja es un dispositivo de nivel inferior que usa el dispositivo IoT Edge como puerta de enlace para comunicarse con IoT Hub.

Vamos a tratar el dispositivo hoja más adelante en el tutorial. En esta sección, descargue los certificados en el dispositivo IoT Edge.

1. En la sesión SSH en la máquina virtual con Linux, inicie sesión en Azure con la CLI de Azure.

    ```azurecli
    az login
    ```

1. Se le solicitará que abra un explorador para <https://microsoft.com/devicelogin> y que proporcione un código único. Puede realizar estos pasos en el equipo local. Cierre la ventana del explorador cuando haya terminado la autenticación.

1. Cuando se autentique correctamente, inicie sesión en la VM de Linux y enumere las suscripciones de Azure.

1. Defina la suscripción de Azure que quiere usar con los comandos de la CLI de Azure.

    ```azurecli
    az account set --subscription <subscriptionId>
    ```

1. Cree un directorio en la VM para los certificados.

    ```bash
    sudo mkdir /edgeMlCertificates
    ```

1. Descargue los certificados que almacenó en Key Vault: new-edge-device-full-chain.cert.pem, new-edge-device.key.pem y azure-iot-test-only.root.ca.cert.pem

    ```azurecli
    key_vault_name="<key vault name>"
    sudo az keyvault secret download --vault-name $key_vault_name --name new-edge-device-full-chain-cert-pem -f /edgeMlCertificates/new-edge-device-full-chain.cert.pem
    sudo az keyvault secret download --vault-name $key_vault_name --name new-edge-device-key-pem -f /edgeMlCertificates/new-edge-device.key.pem
    sudo az keyvault secret download --vault-name $key_vault_name --name azure-iot-test-only-root-ca-cert-pem -f /edgeMlCertificates/azure-iot-test-only.root.ca.cert.pem
    ```

## <a name="update-the-iot-edge-device-configuration"></a>Actualización de la configuración del dispositivo IoT Edge

El entorno de ejecución de IoT Edge usa el archivo `/etc/iotedge/config.yaml` para conservar su configuración. Es necesario actualizar la información siguiente de este archivo:

* **Cadena de conexión del dispositivo:** la cadena de conexión de la identidad del dispositivo en IoT Hub
* **Certificados:** los certificados que se usarán para las conexiones realizadas con los dispositivos de bajada
* **Nombre de host:** la dirección IP pública o el nombre de dominio completo (FQDN) del dispositivo IoT Edge de la VM.

La imagen *Azure IoT Edge en Ubuntu* que se usa para crear la VM de IoT Edge incluye un script de shell que actualiza el archivo config.yaml con la cadena de conexión.

1. En Visual Studio Code, haga clic con el botón derecho en el dispositivo IoT Edge y, después, seleccione **Copy Device Connection String** (Copiar la cadena de conexión del dispositivo).

    ![Copia de la cadena de conexión de Visual Studio Code](media/tutorial-machine-learning-edge-05-configure-edge-device/copy-device-connection-string-command.png)

2. En la sesión de SSH, ejecute el comando para actualizar el archivo config.yaml con la cadena de conexión del dispositivo.

    ```bash
    sudo /etc/iotedge/configedge.sh "<your_iothub_edge_device_connection_string>"
    ```

A continuación, actualizará los certificados y el nombre de host editando directamente el archivo config.yaml.

1. Abra el archivo config.yaml.

    ```bash
    sudo nano /etc/iotedge/config.yaml
    ```

2. Actualice la sección de certificados del archivo config.yaml quitando el símbolo `#` que aparece al principio y configure la ruta para que el archivo tenga un aspecto similar al siguiente ejemplo:

    ```yaml
    certificates:
      device_ca_cert: "/edgeMlCertificates/new-edge-device-full-chain.cert.pem"
      device_ca_pk: "/edgeMlCertificates/new-edge-device.key.pem"
      trusted_ca_certs: "/edgeMlCertificates/azure-iot-test-only.root.ca.cert.pem"
    ```

    Asegúrese de que la línea **certificates:** no tiene ningún espacio en blanco al principio y que a cada uno de los certificados anidados se le aplica una sangría de dos espacios.

    Al hacer clic con el botón derecho en nano, se pegará el contenido del portapapeles en la posición actual del cursor. Para reemplazar la cadena, use las flechas del teclado para desplazarse a la cadena que desea reemplazar, elimine la cadena y luego haga clic con el botón derecho para pegar desde el búfer.

3. En Azure Portal, navegue a la máquina virtual. Copie el nombre DNS (FQDN de la máquina) de la sección **Introducción**.

4. Pegue el FQDN en la sección de nombre de host del archivo config.yml. Asegúrese de que el nombre está escrito en minúsculas.

    ```yaml
    hostname: '<machinename>.<region>.cloudapp.azure.com'
    ```

5. Guarde y cierre el archivo (`Ctrl + X`, `Y`, `Enter`).

6. Reinicie el demonio de iotedge.

    ```bash
    sudo systemctl restart iotedge
    ```

7. Compruebe el estado del demonio de IoT Edge (después del comando, escriba ":q" para salir).

    ```bash
    systemctl status iotedge
    ```

8. Si ve errores (texto en color con el prefijo "\[ERROR\]") en el estado, examine los registros del demonio para obtener información detallada del error.

    ```bash
    journalctl -u iotedge --no-pager --no-full
    ```

## <a name="next-steps"></a>Pasos siguientes

Se acaba de completar la configuración de una VM de Azure como puerta de enlace transparente de Azure IoT Edge. Comenzamos con la generación de certificados de prueba, que se cargaron en Azure Key Vault. A continuación, usamos un script y una plantilla de Resource Manager para implementar la máquina virtual con la imagen "Ubuntu Server 16.04 LTS + Entorno de ejecución de Azure IoT Edge" desde Azure Marketplace. Con la máquina virtual en funcionamiento, nos conectamos mediante SSH, iniciamos sesión en Azure y descargamos certificados de Key Vault. Realizamos varias actualizaciones en la configuración del entorno de ejecución de Azure IoT Edge actualizando el archivo config.yaml.

Para más información, consulte [Uso de un dispositivo IoT Edge como puerta de enlace](iot-edge-as-gateway.md) y [Configuración de un dispositivo IoT Edge para que actúe como puerta de enlace transparente](how-to-create-transparent-gateway.md).

Consulte el artículo siguiente para compilar módulos de IoT Edge.

> [!div class="nextstepaction"]
> [Creación e implementación de módulos personalizados de IoT Edge](tutorial-machine-learning-edge-06-custom-modules.md)
