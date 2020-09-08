---
title: Conexión de una máquina híbrida con servidores habilitados para Azure Arc (versión preliminar)
description: Aprenda a conectar una máquina híbrida con servidores habilitados para Azure Arc y a registrarla en ellos (versión preliminar).
ms.topic: quickstart
ms.date: 08/12/2020
ms.openlocfilehash: eacf75871b1f7cc7fc3b703d8859338578e43456
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213611"
---
# <a name="quickstart-connect-hybrid-machine-with-azure-arc-enabled-servers-preview"></a>Inicio rápido: Conexión de una máquina híbrida con servidores habilitados para Azure Arc (versión preliminar)

Los [servidores habilitados para Azure Arc](../overview.md) (versión preliminar) permiten administrar y controlar las máquinas Windows y Linux hospedadas en entornos locales, perimetrales y multinube. En este inicio rápido, implementará y configurará el agente de Connected Machine de una máquina Windows o Linux hospedada fuera de Azure para que lo administren servidores habilitados para Arc (versión preliminar).

## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

* La implementación del agente de máquina híbrida conectada para servidores habilitados para Arc (versión preliminar) requiere tener permisos de administrador en la máquina para instalar y configurar el agente. En Linux, esto se realiza mediante la cuenta raíz, mientras que en Windows con una cuenta que pertenezca al grupo de administradores locales.

* Antes de empezar, asegúrese de revisar los [requisitos previos](../agent-overview.md#prerequisites) del agente y compruebe que:

    * La máquina de destino utiliza un [sistema operativo](../agent-overview.md#supported-operating-systems) compatible.

    * A su cuenta se le ha concedido una asignación a los [roles de Azure requeridos](../agent-overview.md#required-permissions).

    * Si la máquina se conecta mediante un firewall o un servidor proxy para comunicarse a través de Internet, asegúrese de que las direcciones URL de la [lista](../agent-overview.md#networking-configuration) no están bloqueadas.

    * Los servidores habilitados para Azure Arc (versión preliminar) admiten solo las regiones que se especifican [aquí](../overview.md#supported-regions).

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

## <a name="register-azure-resource-providers"></a>Registro de proveedores de recursos de Azure

Los servidores habilitados para Azure Arc (versión preliminar) dependen de los siguientes proveedores de recursos de Azure de la suscripción para poder usar este servicio:

* Microsoft.HybridCompute
* Microsoft.GuestConfiguration

Regístrelos con los comandos siguientes:

```azurecli-interactive
az account set --subscription "{Your Subscription Name}"
az provider register --namespace 'Microsoft.HybridCompute'
az provider register --namespace 'Microsoft.GuestConfiguration'
```

## <a name="generate-installation-script"></a>Generación de un script de instalación

El script para tanto automatizar la descarga e instalación, como para establecer la conexión con Azure Arc, está disponible en Azure Portal. Para completar el proceso, haga lo siguiente:

1. Inicie el servicio Azure Arc en Azure Portal. Para ello, haga clic en **Todos los servicios** y, a continuación, busque y seleccione **Máquinas: Azure Arc**.

    :::image type="content" source="./media/quick-enable-hybrid-vm/search-machines.png" alt-text="Búsqueda de servidores habilitados para Arc en todos los servicios" border="false":::

1. En la página **Máquinas: Azure Arc**, seleccione **Agregar**, en la parte superior izquierda, o la opción **Crear máquina: Azure Arc** en la parte inferior del panel central.

1. En la página **Seleccionar un método**, seleccione el icono **Agregar máquinas mediante un script interactivo** y, a continuación, seleccione **Generar script**.

1. En la página **Generar script**, seleccione la suscripción y el grupo de recursos en el que desea que se administre la máquina en Azure. Seleccione la ubicación de Azure en la que se almacenarán los metadatos de la máquina.

1. En la página **Generar script**, en la lista desplegable **Sistema operativo**, seleccione el sistema operativo en el que se ejecutará el script.

1. Si la máquina se comunica mediante un servidor proxy para conectarse a Internet, seleccione **Siguiente: Servidor proxy**.

1. En la pestaña **Servidor proxy**, especifique la dirección IP del servidor proxy o el nombre y el número de puerto que usará la máquina para comunicarse con el servidor proxy. Escriba el valor con el formato `http://<proxyURL>:<proxyport>`.

1. Seleccione **Revisar y generar**.

1. En la pestaña **Revisar y generar**, revise la información del resumen y, a continuación, seleccione **Descargar**. Si todavía necesita realizar cambios, seleccione **Anterior**.

## <a name="install-the-agent-using-the-script"></a>Instalación del agente mediante el script

### <a name="windows-agent"></a>Agente de Windows

1. Inicie sesión en el servidor.

1. Abra un símbolo del sistema de PowerShell de 64 bits con privilegios elevados.

1. Cambie a la carpeta o recurso compartido en el que copió el script y ejecútelo en el servidor mediante el script `./OnboardingScript.ps1`.

### <a name="linux-agent"></a>Agente Linux

1. Para instalar el agente Linux en la máquina de destino que pueda comunicarse directamente con Azure, ejecute el siguiente comando:

    ```bash
    bash ~/Install_linux_azcmagent.sh
    ```

    * Si la máquina de destino se comunica a través de un servidor proxy, ejecute el siguiente comando:

        ```bash
        bash ~/Install_linux_azcmagent.sh --proxy "{proxy-url}:{proxy-port}"
        ```

## <a name="verify-the-connection-with-azure-arc"></a>Comprobación de la conexión con Azure Arc

Después de instalar el agente y configurarlo para que se conecte a los servidores habilitados para Azure Arc (versión preliminar), vaya a Azure Portal para comprobar que el servidor se ha conectado correctamente. Vea la máquina en [Azure Portal](https://aka.ms/hybridmachineportal).

:::image type="content" source="./media/quick-enable-hybrid-vm/enabled-machine.png" alt-text="Conexión de una máquina realizada correctamente" border="false":::

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha habilitado un máquina híbrida de Linux o Windows y la ha conectado correctamente al servicio, está listo para habilitar Azure Policy para conocer el cumplimiento en Azure.

Para aprender a identificar una máquina virtual de los servidores habilitados para Azure Arc (versión preliminar) que no tiene instalado el agente de Log Analytics, continúe con el tutorial:

> [!div class="nextstepaction"]
> [Creación de una asignación de directiva para identificar recursos no compatibles](tutorial-assign-policy-portal.md)
