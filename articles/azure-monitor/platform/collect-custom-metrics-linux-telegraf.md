---
title: Recopilación de métricas personalizadas para una máquina virtual Linux con el agente de InfluxData Telegraf
description: En este artículo, se proporcionan instrucciones sobre cómo implementar el agente de InfluxData Telegraf en una máquina virtual Linux en Azure y configurar el agente para que publique las métricas en Azure Monitor.
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ancav
ms.subservice: metrics
ms.openlocfilehash: b80f27e490dd3b1890eab7740fb4650ba4280abb
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88207801"
---
# <a name="collect-custom-metrics-for-a-linux-vm-with-the-influxdata-telegraf-agent"></a>Recopilación de métricas personalizadas para una máquina virtual Linux con el agente de InfluxData Telegraf

Mediante Azure Monitor, puede recopilar métricas personalizadas a través de los datos de telemetría de la aplicación, un agente que se ejecuta en los recursos de Azure o, incluso, fuera del sistema de supervisión. A continuación, puede enviarlas directamente a Azure Monitor. En este artículo, se proporcionan instrucciones sobre cómo implementar al agente de [InfluxData](https://www.influxdata.com/) Telegraf en una máquina virtual Linux en Azure y configurar el agente para que publique las métricas en Azure Monitor. 

## <a name="influxdata-telegraf-agent"></a>Agente de InfluxData Telegraf 

[Telegraf](https://docs.influxdata.com/telegraf/) es un agente controlado por complemento que habilita la recopilación de métricas más de 150 orígenes diferentes. Según las cargas de trabajo que se ejecuten en la máquina virtual, puede configurar al agente para aprovechar los complementos de entrada especializados para recopilar métricas. Algunos ejemplos son MySQL, NGINX y Apache. Mediante el uso de complementos de salida, el agente, a continuación, puede escribir en los destinos que elija. El agente de Telegraf se integra directamente con la API REST de métricas personalizadas de Azure Monitor. Admite un complemento de salida de Azure Monitor. Mediante este complemento, el agente puede recopilar métricas específicas de la carga de trabajo de la máquina virtual Linux y enviarlas como métricas personalizadas a Azure Monitor. 

 ![Información general del agente de Telegraf](./media/collect-custom-metrics-linux-telegraf/telegraf-agent-overview.png)

> [!NOTE]  
> Las métricas personalizadas no se admiten en todas las regiones. Las regiones admitidas se enumeran [aquí](./metrics-custom-overview.md#supported-regions).

## <a name="send-custom-metrics"></a>Envío de métricas personalizadas 

En este tutorial, implementamos una máquina virtual Linux que ejecuta el sistema operativo Ubuntu 16.04 LTS. El agente de Telegraf es compatible con la mayoría de los sistemas operativos Linux. En el [portal de descarga de InfluxData](https://portal.influxdata.com/downloads) están disponibles los paquetes Debian y RPM, además de los archivos binarios de Linux sin empaquetar. Consulte esta [guía de instalación de Telegraf](https://docs.influxdata.com/telegraf/v1.8/introduction/installation/) para conocer instrucciones y opciones de instalación adicionales. 

Inicie sesión en [Azure Portal](https://portal.azure.com).

> [!NOTE]  
> Si quiere migrar reglas de alertas clásicas y usar una máquina virtual de Linux existente, asegúrese de que la máquina virtual tenga establecida una identidad asignada por el sistema en **Activado**.

Cree una nueva máquina virtual Linux: 

1. Seleccione la opción **Crear un recurso** en el panel de navegación izquierdo. 
1. Busque **Máquina virtual**.  
1. Seleccione **Ubuntu 16.04 LTS** y seleccione **Crear**. 
1. Especifique un nombre de máquina virtual como **MyTelegrafVM**.  
1. Deje el tipo de disco como **SSD**. A continuación, proporcione un **Nombre de usuario**, por ejemplo, **usuarioAzure**. 
1. En **Tipo de autenticación**, seleccione **Contraseña**. A continuación, escriba una contraseña que utilizará más adelante para conectarse mediante SSH a la máquina virtual. 
1. Elija **Crear nuevo grupo de recursos**. A continuación, indique un nombre, como **miGrupoDeRecursos**. Elija su **Ubicación**. Después, seleccione **Aceptar**. 

    ![Creación de una máquina virtual de Ubuntu](./media/collect-custom-metrics-linux-telegraf/create-vm.png)

1. Seleccione un tamaño para la máquina virtual. Puede filtrar por **Tipo de proceso** o **Tipo de disco**, por ejemplo. 

    ![Información general del tamaño de máquina virtual en el agente de Telegraf](./media/collect-custom-metrics-linux-telegraf/vm-size.png)

1. En la página **Configuración**, en **Red** > **Grupo de seguridad de red** > **Seleccionar puertos de entrada públicos**, seleccione **HTTP** y **SSH (22)** . Deje el resto de valores predeterminados y seleccione **Aceptar**. 

1. En la página de resumen, seleccione **Crear** para iniciar la implementación de la máquina virtual. 

1. La máquina virtual se ancla al panel de Azure Portal. Una vez completada la implementación, se abrirá automáticamente el resumen de la máquina virtual. 

1. En el panel de la máquina virtual, vaya a la pestaña **Identidad**. Asegúrese de que la máquina virtual tiene una identidad asignada por el sistema establecida en **Activada**. 
 
    ![Vista previa de la identidad de la máquina virtual de Telegraf](./media/collect-custom-metrics-linux-telegraf/connect-to-VM.png)
 
## <a name="connect-to-the-vm"></a>Conexión a la máquina virtual 

Cree una conexión SSH con la máquina virtual. Seleccione el botón **Conectar** en la página de información general de la máquina virtual. 

![Página de información general de la máquina virtual de Telegraf](./media/collect-custom-metrics-linux-telegraf/connect-VM-button2.png)

En la página **Connect to virtual machine** (Conexión a una máquina virtual), mantenga las opciones predeterminadas para conectarse por nombre DNS a través del puerto 22. En **Iniciar sesión con la cuenta local de VM** se muestra un comando de conexión. Seleccione el botón para copiar el comando. En el ejemplo siguiente se muestra el aspecto que tiene el comando de conexión SSH: 

```cmd
ssh azureuser@XXXX.XX.XXX 
```

Pegue el comando de conexión SSH en un shell, como Azure Cloud Shell o Bash en Ubuntu o Windows, o bien use un cliente de SSH de su elección para crear la conexión. 

## <a name="install-and-configure-telegraf"></a>Instalación y configuración de Telegraf 

Para instalar el paquete de Telegraf para Debian en la máquina virtual, ejecute los siguientes comandos desde la sesión SSH: 

```cmd
# download the package to the VM 
wget https://dl.influxdata.com/telegraf/releases/telegraf_1.8.0~rc1-1_amd64.deb 
# install the package 
sudo dpkg -i telegraf_1.8.0~rc1-1_amd64.deb
```
El archivo de configuración de Telegraf define las operaciones de Telegraf. De forma predeterminado, se instala un archivo de configuración de ejemplo en la ruta de acceso **/etc/telegraf/telegraf.conf**. El archivo de configuración de ejemplo enumera todos los posibles complementos de entrada y salida. Sin embargo, vamos a crear un archivo de configuración personalizado y haremos que el agente lo use; para ello, hay que ejecutar los siguientes comandos: 

```cmd
# generate the new Telegraf config file in the current directory 
telegraf --input-filter cpu:mem --output-filter azure_monitor config > azm-telegraf.conf 

# replace the example config with the new generated config 
sudo cp azm-telegraf.conf /etc/telegraf/telegraf.conf 
```

> [!NOTE]  
> El código anterior solo permite dos complementos de entrada: **cpu** y **mem**. Puede agregar más complementos de entrada, en función de la carga de trabajo que se ejecute en la máquina. Algunos ejemplos son Docker, MySQL y NGINX. Para obtener una lista completa de complementos de entrada, consulte la sección **Configuración adicional**. 

Por último, para que el agente se inicie con la nueva configuración, hacemos que el agente se detenga y se inicie mediante los siguientes comandos: 

```cmd
# stop the telegraf agent on the VM 
sudo systemctl stop telegraf 
# start the telegraf agent on the VM to ensure it picks up the latest configuration 
sudo systemctl start telegraf 
```
Ahora, el agente recopilará las métricas de cada uno de los complementos de entrada especificados y los emitirá a Azure Monitor. 

## <a name="plot-your-telegraf-metrics-in-the-azure-portal"></a>Trazado de las métricas de Telegraf en Azure Portal 

1. Abra [Azure Portal](https://portal.azure.com). 

1. Vaya a la pestaña **Supervisar**. Seleccione **Métricas**.  

     ![Monitor: Métricas (versión preliminar)](./media/collect-custom-metrics-linux-telegraf/metrics.png)

1. Seleccione la máquina virtual en el selector de recursos.

     ![Gráfico de métricas](./media/collect-custom-metrics-linux-telegraf/metric-chart.png)

1. Seleccione el espacio de nombres **Telegraf/CPU** y la métrica **usage_system**. Puede elegir filtrar por las dimensiones de esta métrica, o dividir según dichas dimensiones.  

     ![Seleccione el espacio de nombres y la métrica](./media/collect-custom-metrics-linux-telegraf/VM-resource-selector.png)

## <a name="additional-configuration"></a>Configuración adicional 

En el tutorial anterior, se proporciona información sobre cómo configurar el agente de Telegraf para recopilar métricas de algunos complementos básicos de entrada. El agente de Telegraf es compatible con más de 150 complementos de entrada, con algunas opciones de configuración adicionales de compatibilidad. InfluxData ha publicado una [lista de complementos compatibles](https://docs.influxdata.com/telegraf/v1.15/plugins/inputs/) e instrucciones en inglés sobre [cómo configurarlas](https://docs.influxdata.com/telegraf/v1.15/administration/configuration/).  

Además, en este tutorial usó el agente de Telegraf para emitir métricas acerca de la máquina virtual en la que se implementó el agente. El agente de Telegraf también se puede usar como un recopilador y centro de reenvío de las métricas para otros recursos. Para obtener más información sobre cómo configurar el agente para que emita métricas para otros recursos de Azure, consulte [Azure Monitor Custom Metric Output for Telegraf](https://github.com/influxdata/telegraf/blob/fb704500386214655e2adb53b6eb6b15f7a6c694/plugins/outputs/azure_monitor/README.md) (Salida de métricas personalizadas de Azure Monitor para Telegraf).  

## <a name="clean-up-resources"></a>Limpieza de recursos 

Cuando ya no los necesite, puede eliminar el grupo de recursos, la máquina virtual y todos los recursos relacionados. Para ello, seleccione el grupo de recursos de la máquina virtual y seleccione **Eliminar**. Después, confirme el nombre del grupo de recursos que hay que eliminar. 

## <a name="next-steps"></a>Pasos siguientes
- Más información acerca de las [métricas personalizadas](metrics-custom-overview.md).
