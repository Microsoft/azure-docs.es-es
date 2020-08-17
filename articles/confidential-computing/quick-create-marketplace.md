---
title: 'Inicio rápido: Creación de una máquina virtual de computación confidencial de Azure con Marketplace'
description: Para empezar a trabajar con sus implementaciones, aprenda a crear rápidamente una máquina virtual de computación confidencial con Marketplace.
author: JBCook
ms.service: virtual-machines
ms.subservice: workloads
ms.workload: infrastructure
ms.topic: quickstart
ms.date: 04/06/2020
ms.author: JenCook
ms.openlocfilehash: d892d04eddb572ab644693503e454e9f00e0b39c
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281096"
---
# <a name="quickstart-deploy-an-azure-confidential-computing-vm-in-the-marketplace"></a>Inicio rápido: Implementación de una máquina virtual de computación confidencial de Azure en Marketplace

Empiece a trabajar con la computación confidencial de Azure mediante Azure Marketplace para crear una máquina virtual con el respaldo de Intel SGX. A continuación, instalará el kit de desarrollo de software (SDK) de Open Enclave para configurar el entorno de desarrollo. 

Se recomienda este tutorial si desea empezar a implementar rápidamente una máquina virtual de computación confidencial. Las máquinas virtuales se ejecutan en hardware especializado y requieren que se ejecuten las entradas de configuración específicas según lo previsto. La oferta de Marketplace que se describe en este inicio rápido facilita la implementación mediante la restricción de los datos proporcionados por el usuario.

Si está interesado en implementar una máquina virtual de computación confidencial con una configuración más personalizada, siga los [pasos de implementación de máquinas virtuales de computación confidencial de Azure Portal](quick-create-portal.md).

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) antes de empezar.

> [!NOTE]
> Las cuentas de evaluación gratuita no tienen acceso a las máquinas virtuales que se usan en este tutorial. Actualice a una suscripción de pago por uso.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. En la parte superior, escriba **computación confidencial de Azure** en la barra de búsqueda.

1. Seleccione **Azure confidential computing (Virtual Machine)** (Computación confidencial de Azure [máquina virtual]) en la sección de **Marketplace**. 

    ![Seleccione Marketplace](media/quick-create-marketplace/portal-search-marketplace.png)    

1. En la página de aterrizaje de la implementación de la computación confidencial de Azure, seleccione **Crear**.
 

## <a name="configure-your-virtual-machine"></a>Configuración de la máquina virtual

1. En la pestaña **Fundamentos**, seleccione la **suscripción** y el **grupo de recursos**. El grupo de recursos debe estar vacío para implementar una máquina virtual a partir de esta plantilla.

1. Escriba o seleccione los valores siguientes:

   * **Región**: Seleccione la región de Azure adecuada para usted.

        > [!NOTE]
        > Las máquinas virtuales de computación confidencial solo se ejecutan en hardware especializado disponible en regiones específicas. Para ver las regiones más recientes disponibles para las máquinas virtuales de la serie DCsv2, consulte las [regiones disponibles](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines).
    
    * **Elija una imagen**: Seleccione cualquier imagen. Si desea completar este tutorial específico, seleccione Ubuntu 18.04 (Gen 2). Si no es así, se le redirigirá a los pasos adecuados. 

    * En **Nombre de la máquina virtual**, escriba un nombre para la nueva máquina virtual.

    * **Tipo de autenticación**: Seleccione **Clave pública SSH** si va a crear una máquina virtual Linux. 

         > [!NOTE]
         > Para la autenticación, puede una clave pública SSH o una contraseña. La opción de SSH es más segura. Para obtener instrucciones acerca de cómo generar una clave SSH, consulte [Creación y uso de un par de claves SSH pública y privada para máquinas virtuales Linux en Azure](../virtual-machines/linux/mac-create-ssh-keys.md).

    * **Nombre de usuario**: Escriba el nombre del administrador de la máquina virtual.

    * **Clave pública SSH**: Si es aplicable, escriba la clave pública RSA.
    
    * **Contraseña**: Si procede, escriba la contraseña para la autenticación.
 
1. Seleccione el botón **Siguiente: Configuración de la máquina virtual** en la parte inferior de la pantalla.

    > [!IMPORTANT]
    > Espere hasta que la página se actualice. *No debería* aparecer un mensaje que indica que "Las máquinas virtuales de computación confidencial de la serie DCsv2 están disponibles en un número limitado de regiones". Si el mensaje continúa, vuelva a la página anterior y seleccione una región de la serie DCsv2 disponible.

1. Para **cambiar el tamaño**, elija una máquina virtual con funcionalidades de computación confidencial en el selector de tamaño. 

    > [!TIP]
    > Debería ver los tamaños **DC1s_v2**, **DC2s_v2**, **DC4s_V2** y **DC8_v2**. Estos son los únicos tamaños de máquina virtual que admiten actualmente la computación confidencial. [Más información](virtual-machine-solutions.md).

1. En **Tipo de disco de sistema operativo**, seleccione un tipo de disco.

1. En **Red virtual**, cree una nueva o elija una de un recurso existente.

1. Como **Subred**, cree una nueva o elija una de un recurso existente.

1. En **Seleccionar puertos de entrada públicos**, elija **SSH(Linux)/RDP(Windows)** . En este inicio rápido, este paso es necesario para conectarse a la máquina virtual y completar la configuración del SDK de Open Enclave. 

1. Deje la opción **Diagnósticos de arranque** deshabilitada para este inicio rápido. 

1. Seleccione **Revisar + crear**.

1. En el panel **Revisar + crear**, seleccione **Crear**.

> [!NOTE]
> Vaya a la sección siguiente y siga en este tutorial si ha implementado una máquina virtual Linux. Si ha implementado una máquina virtual Windows, [siga estos pasos para conectarse a ella](../virtual-machines/windows/connect-logon.md) y, a continuación, [instale el SDK de Open Enclave en Windows](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Windows.md).


## <a name="connect-to-the-linux-vm"></a>Conexión a la máquina virtual Linux

Si ya utiliza un shell de BASH, conéctese a la máquina virtual de Azure mediante el comando **ssh**. En el siguiente comando, reemplace el nombre de usuario y la dirección IP de la máquina virtual para conectarse a su máquina virtual Linux.

```bash
ssh azureadmin@40.55.55.555
```

Puede buscar la dirección IP pública de la máquina virtual en la sección Información general de la máquina virtual en Azure Portal.

![Dirección IP en Azure Portal](media/quick-create-portal/public-ip-virtual-machine.png)

Si utiliza Windows y no tiene un shell de BASH, instale un cliente de SSH, como PuTTY.

1. [Descargue e instale PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

1. Ejecute PuTTY.

1. En la pantalla de configuración de PuTTY, escriba la dirección IP pública de la máquina virtual.

1. Seleccione **Abrir** y escriba el nombre de usuario y la contraseña en los cuadros.

Para más información acerca de cómo conectarse a máquinas virtuales Linux, consulte [Creación de máquinas virtuales Linux con Azure Portal](../virtual-machines/linux/quick-create-portal.md).

> [!NOTE]
> Si ve una alerta de seguridad de PuTTY que indique que la clave de host del servidor no se almacena en la caché del registro, elija entre las opciones siguientes. Si confía en este host, seleccione **Sí** para agregar la clave a la caché de PuTTY y siga conectándose. Si quiere conectarse solo una vez, sin agregar la clave a la caché, seleccione **No**. Si no confía en este host, seleccione **Cancelar** para abandonar la conexión.

## <a name="install-the-open-enclave-sdk-oe-sdk"></a>Instalación del SDK de Open enclave (SDK de OE) <a id="Install"></a>

Siga las instrucciones detalladas para instalar el [SDK de OE](https://github.com/openenclave/openenclave) en la máquina virtual de la serie DCsv2 con una imagen de Ubuntu 18.04 LTS Gen 2. 

Si la máquina virtual se ejecuta en Ubuntu 16.04 LTS Gen 2, deberá seguir las [instrucciones de instalación de Ubuntu 16.04](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Ubuntu_16.04.md). 

#### <a name="1-configure-the-intel-and-microsoft-apt-repositories"></a>1. Configure los repositorios de Intel y Microsoft APT

```bash
echo 'deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu bionic main' | sudo tee /etc/apt/sources.list.d/intel-sgx.list
wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | sudo apt-key add -

echo "deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-7 main" | sudo tee /etc/apt/sources.list.d/llvm-toolchain-bionic-7.list
wget -qO - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -

echo "deb [arch=amd64] https://packages.microsoft.com/ubuntu/18.04/prod bionic main" | sudo tee /etc/apt/sources.list.d/msprod.list
wget -qO - https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
```

#### <a name="2-install-the-intel-sgx-dcap-driver"></a>2. Instale el controlador Intel SGX DCAP

```bash
sudo apt update
sudo apt -y install dkms
wget https://download.01.org/intel-sgx/sgx-dcap/1.4/linux/distro/ubuntuServer18.04/sgx_linux_x64_driver_1.21.bin -O sgx_linux_x64_driver.bin
chmod +x sgx_linux_x64_driver.bin
sudo ./sgx_linux_x64_driver.bin
```

> [!WARNING]
> Use el controlador Intel SGX DCAP más reciente del [sitio web de Intel SGX](https://01.org/intel-software-guard-extensions/downloads).

#### <a name="3-install-the-intel-and-open-enclave-packages-and-dependencies"></a>3. Instale los paquetes y dependencias de Intel y Open Enclave

```bash
sudo apt -y install clang-7 libssl-dev gdb libsgx-enclave-common libsgx-enclave-common-dev libprotobuf10 libsgx-dcap-ql libsgx-dcap-ql-dev az-dcap-client open-enclave
```

> [!NOTE] 
> En este paso también se instala el paquete [az-dcap-client](https://github.com/microsoft/azure-dcap-client), el cual es necesario para realizar la atestación remota en Azure.

#### <a name="4-verify-the-open-enclave-sdk-install"></a>4. **Compruebe la instalación del SDK de Open Enclave**

Consulte [Uso del SDK de Open Enclave](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/Linux_using_oe_sdk.md) en GitHub para comprobar y usar el SDK instalado.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede eliminar el grupo de recursos, la máquina virtual y todos los recursos relacionados. 

Seleccione el grupo de recursos de la máquina virtual y haga clic en **Eliminar**. Confirme el nombre del grupo de recursos para terminar de eliminar los recursos.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado una máquina virtual de computación confidencial y ha instalado el SDK de Open enclave. Para más información sobre las máquinas virtuales de computación confidencial en Azure, consulte [Soluciones en Virtual Machines](virtual-machine-solutions.md). 

Si ha implementado una máquina virtual Windows, aprenda a compilar aplicaciones con los [ejemplos del SDK de Open Enclave para Windows](https://github.com/openenclave/openenclave/blob/master/samples/README_Windows.md) en GitHub. 

Descubra cómo puede crear aplicaciones de computación confidencial en Linux; para ello, continúe con los ejemplos de Linux del SDK de Open Enclave en GitHub. 

> [!div class="nextstepaction"]
> [Compilación de ejemplos del SDK de Open Enclave en Linux](https://github.com/openenclave/openenclave/blob/master/samples/README_Linux.md)
