---
title: Conexión de equipos mediante la puerta de enlace de Log Analytics | Microsoft Docs
description: Conecte los dispositivos y los equipos supervisados por Operations Manager con la puerta de enlace de Log Analytics para enviar datos al servicio de Log Analytics y Azure Automation cuando no tengan acceso a Internet.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 12/24/2019
ms.openlocfilehash: 94c668e7ffaff81fed9c2e511bc38239069fa43e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87305217"
---
# <a name="connect-computers-without-internet-access-by-using-the-log-analytics-gateway-in-azure-monitor"></a>Conexión de equipos sin acceso a Internet mediante la puerta de enlace de Log Analytics en Azure Monitor

>[!NOTE]
>La terminología cambia a medida que se realiza la transición de Microsoft Operations Management Suite (OMS) a Microsoft Azure Monitor. Este artículo hace referencia a la puerta de enlace de OMS como la puerta de enlace de Azure Log Analytics. 
>

Este artículo describe cómo configurar la comunicación con Azure Automation y Azure Monitor mediante la puerta de enlace de Log Analytics cuando los equipos que están directamente conectados o están supervisados por Operations Manager no tienen acceso a Internet. 

La puerta de enlace de Log Analytics es un proxy de reenvío de HTTP que admite la tunelización HTTP mediante el comando HTTP CONNECT. Esta puerta de enlace envía datos a Azure Automation y a un área de trabajo de Log Analytics en Azure Monitor en nombre de los equipos que no pueden conectarse a Internet. 

La puerta de enlace de Log Analytics admite lo siguiente:

* Informes de las mismas áreas de trabajo de Log Analytics configuradas en cada agente que hay detrás y que están configuradas con Hybrid Runbook Workers de Azure Automation.  
* Equipos Windows en los que Microsoft Monitoring Agent está directamente conectado a un área de trabajo de Log Analytics en Azure Monitor.
* Equipos Linux en los que el agente de Log Analytics está directamente conectado a un área de trabajo de Azure Monitor.  
* System Center Operations Manager 2012 SP1 con UR7, Operations Manager 2012 R2 con UR3 o un grupo de administración en Operations Manager 2016 o posterior integrado con Log Analytics.  

Algunas directivas de seguridad de TI no permiten la conexión a Internet para los equipos de red. Estos equipos no conectados podrían ser dispositivos de punto de venta (POS) o los servidores que admiten servicios de TI, por ejemplo. Para conectar estos dispositivos a Azure Automation o un área de trabajo de Log Analytics para poder administrarlos y supervisarlos, configúrelos para que se comuniquen directamente con la puerta de enlace de Log Analytics. La puerta de enlace de Log Analytics puede recibir información de configuración y reenviar los datos en su nombre. Si los equipos están configurados con el agente de Log Analytics para conectarse directamente a un área de trabajo de Log Analytics, los equipos se comunicarán en su lugar con la puerta de enlace de Log Analytics.  

La puerta de enlace de Log Analytics transfiere datos desde los agentes al servicio directamente. No analiza ninguno de los datos en tránsito ni almacena datos en la memoria caché cuando pierde la conectividad con el servicio. Cuando la puerta de enlace no puede comunicarse con el servicio, el agente continúa ejecutándose y pone en cola los datos recopilados en el disco del equipo supervisado. Cuando se restaura la conexión, el agente envía los datos en caché recopilados a Azure Monitor.

Cuando un grupo de administración de Operations Manager se integra en Log Analytics, se pueden configurar los servidores de administración para que se conecten a la puerta de enlace de Log Analytics para recibir información de configuración y enviar los datos recopilados según la solución que haya habilitado.  Los agentes de Operations Manager envían algunos datos al servidor de administración. Por ejemplo, los agentes pueden enviar alertas de Operations Manager, datos de evaluación de configuración, datos del espacio de instancia y datos de capacidad. Otros datos de gran volumen, como registros, datos de rendimiento y eventos de seguridad de Internet Information Services (IIS), se envían directamente a la puerta de enlace de Log Analytics. 

Si ha implementado uno o más servidores de puerta de enlace de Operations Manager para supervisar sistemas no confiables en una red perimetral o una red aislada, dichos servidores no se podrán comunicar con la puerta de enlace de Log Analytics.  Los servidores de puerta de enlace de Operations Manager solo pueden informar a un servidor de administración.  Cuando se configura un grupo de administración de Operations Manager para comunicarse con la puerta de enlace de Log Analytics, la información de configuración de proxy se distribuye automáticamente a todos los equipos administrados por agente y configurados para recopilar datos de registro para Azure Monitor, incluso si la configuración está vacía.

Para proporcionar alta disponibilidad a grupos directamente conectados o grupos de Operations Management que se comunican con un área de trabajo de Log Analytics mediante la puerta de enlace, use el equilibrio de carga de red (NLB) para redirigir y distribuir el tráfico entre varios servidores de puerta de enlace. De esa forma, si un servidor de puerta de enlace deja de funcionar, el tráfico se redirige a otro nodo disponible.  

El equipo que ejecuta la puerta de enlace de Log Analytics requiere el agente de Windows de Log Analytics para identificar los puntos de conexión de servicio que necesita la puerta de enlace para establecer la conexión. El agente también necesita dirigir la puerta de enlace para informar a las mismas áreas de trabajo que los agentes o grupo de administración de Operations Manager detrás de la puerta de enlace con la que se ha configurado. Esta configuración permite que la puerta de enlace y el agente se comuniquen con su área de trabajo asignada.

Una puerta de enlace puede ser de hosts múltiples en hasta cuatro áreas de trabajo. Este es el número total de áreas de trabajo que admite un agente de Windows.  

Cada agente debe tener conectividad de red a la puerta de enlace para que los agentes puedan transferir automáticamente datos a la puerta de enlace y recibirlos de esta. Evite instalar la puerta de enlace en un controlador de dominio. Los equipos Linux que están detrás de un servidor de puerta de enlace no pueden usar el método de [instalación del script contenedor](agent-linux.md#install-the-agent-using-wrapper-script) para instalar el agente de Log Analytics para Linux. El agente debe descargarse manualmente, copiarse en el equipo e instalarse manualmente porque la puerta de enlace solo admite la comunicación con los servicios de Azure que se han mencionado anteriormente.

El siguiente diagrama muestra el flujo de datos desde los agentes directos, a través de la puerta de enlace, a Azure Automation y Log Analytics. La configuración de proxy del agente debe coincidir con el puerto con el que se ha configurado la puerta de enlace de Log Analytics.  

![Diagrama de comunicación de agente directo con los servicios](./media/gateway/oms-omsgateway-agentdirectconnect.png)

El siguiente diagrama muestra el flujo de datos de un grupo de administración de Operations Manager a Log Analytics.   

![Diagrama de comunicación de Operations Manager con Log Analytics](./media/gateway/log-analytics-agent-opsmgrconnect.png)

## <a name="set-up-your-system"></a>Configuración del sistema

Los equipos designados para ejecutar la puerta de enlace de Log Analytics deben tener la siguiente configuración:

* Windows 10, Windows 8.1 o Windows 7
* Windows Server 2019, Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 o Windows Server 2008
* Microsoft .NET Framework 4.5
* Procesador de 4 núcleos y 8 GB de memoria como mínimo 
* El [agente de Windows para Log Analytics](agent-windows.md) que está configurado para informar a la misma área de trabajo que los agentes que se comunican a través de la puerta de enlace.

### <a name="language-availability"></a>Disponibilidad de idiomas

La puerta de enlace de Log Analytics está disponible en los idiomas siguientes:

- Chino (simplificado)
- Chino (tradicional)
- Checo
- Neerlandés
- Inglés
- Francés
- Alemán
- Húngaro
- Italiano
- Japonés
- Coreano
- Polaco
- Portugués (Brasil)
- Portugués (Portugal)
- Ruso
- Español (internacional)

### <a name="supported-encryption-protocols"></a>Protocolos de cifrado admitidos

La puerta de enlace de Log Analytics solo admite Seguridad de la capa de transporte (TLS) 1.0, 1.1 y 1.2.  No se admite Capa de sockets seguros (SSL).  Para garantizar la seguridad de datos en tránsito en Log Analytics, configure la puerta de enlace para que use como mínimo TLS 1.2. Las versiones anteriores de TLS o SSL son vulnerables. Aunque permiten actualmente compatibilidad con versiones anteriores, evite su uso.  

Para información adicional, revise [Sending data securely using TLS 1.2](./data-security.md#sending-data-securely-using-tls-12) (Envío de datos de forma segura mediante TLS 1.2). 

### <a name="supported-number-of-agent-connections"></a>Número admitido de conexiones del agente

En la tabla siguiente se muestra aproximadamente cuántos agentes pueden comunicarse con un servidor de puerta de enlace. La compatibilidad se basa en los agentes que cargan aproximadamente 200 KB de datos cada 6 segundos. El volumen de datos por agente de prueba es de unos 2,7 GB por día.

|Puerta de enlace |Agentes compatibles (aproximado)|  
|--------|----------------------------------|  
|CPU: Procesador Intel Xeon E5-2660 v3 \@, 2,6 GHz, doble núcleo<br> Memoria: 4 GB<br> Ancho de banda de la red: 1 Gbps| 600|  
|CPU: Procesador Intel Xeon E5-2660 v3 \@, 2,6 GHz, cuatro núcleos<br> Memoria: 8 GB<br> Ancho de banda de la red: 1 Gbps| 1000|  

## <a name="download-the-log-analytics-gateway"></a>Descargar la puerta de enlace de Log Analytics

Obtenga la versión más reciente del archivo de instalación de puerta de enlace de Log Analytics en el Centro de descarga de Microsoft ([vínculo de descarga](https://go.microsoft.com/fwlink/?linkid=837444)) o Azure Portal.

Para obtener la puerta de enlace de Log Analytics desde Azure Portal, siga estos pasos:

1. Vaya a la lista de servicios y, luego, seleccione **Log Analytics**. 
1. Seleccione un área de trabajo.
1. En su hoja de área de trabajo, en **General**, seleccione **Inicio rápido**. 
1. En **Elija un origen de datos para conectarse al área de trabajo**, seleccione **Equipos**.
1. En la hoja **Agente directo**, seleccione **Descargar puerta de enlace de Log Analytics**.
 
   ![Captura de pantalla de los pasos para descargar la puerta de enlace de Log Analytics](./media/gateway/download-gateway.png)

or 

1. En la hoja de área de trabajo en **Configuración**, seleccione **Configuración avanzada**.
1. Vaya a **Orígenes conectados** > **Servidores de Windows** y seleccione **Descargar puerta de enlace de Log Analytics**.

## <a name="install-log-analytics-gateway-using-setup-wizard"></a>Instalación de la puerta de enlace de Log Analytics mediante el asistente para la instalación

Para instalar una puerta de enlace mediante el asistente para la instalación, siga estos pasos. 

1. En la carpeta de destino, haga doble clic en **Log Analytics gateway.msi**.
1. En la página **principal**, seleccione **Siguiente**.

   ![Captura de pantalla de la página de bienvenida en el asistente para la instalación de la puerta de enlace](./media/gateway/gateway-wizard01.png)

1. En la página del **Contrato de licencia**, seleccione **Acepto los términos del contrato de licencia** para aceptar los Términos de licencia del software de Microsoft y, a continuación, seleccione **Siguiente**.
1. En la página de **dirección del proxy y puerto**:

   a. Escriba el número de puerto TCP que se va a usar para la puerta de enlace. El programa de instalación utiliza este número de puerto para configurar una regla de entrada en el firewall de Windows.  El valor predeterminado es 8080.
      El intervalo válido del número de puerto es de 1 a 65535. Si el valor especificado no se encuentra en este intervalo, aparece un mensaje de error.

   b. Si el servidor en que está instalada la puerta de enlace necesita comunicarse a través de un proxy, escriba la dirección del proxy al que debe conectarse la puerta de enlace. Por ejemplo, escriba `http://myorgname.corp.contoso.com:80`:  Si deja este campo en blanco, la puerta de enlace intentará conectarse directamente a Internet.  Si el servidor proxy requiere autenticación, especifique el nombre de usuario y la contraseña.

   c. Seleccione **Next** (Siguiente).

   ![Captura de pantalla de la configuración del proxy de la puerta de enlace](./media/gateway/gateway-wizard02.png)

1. Si Microsoft Update no está habilitado, aparecerá la página de Microsoft Update, donde puede elegir habilitarlo. Elija la opción que desee y seleccione **Siguiente**. De lo contrario, continúe con el paso siguiente.
1. En la página de la **carpeta de destino**, deje la carpeta predeterminada C:\Archivos de programa\OMS Gateway o escriba la ubicación en la que desea instalar la puerta de enlace. Luego, seleccione **Siguiente**.
1. En la página **Preparado para instalar**, seleccione **Instalar**. Si un Control de cuentas de usuario solicita permiso para realizar la instalación, seleccione **Sí**.
1. Una vez finalizada la instalación, seleccione **Finalizar**. Para comprobar que el servicio se está ejecutando, abra el complemento services.msc y compruebe que la **puerta de enlace de OMS** aparece en la lista de servicios y que su estado sea **En ejecución**.

   ![Captura de pantalla de los servicios locales, que muestra que la puerta de enlace de OMS se está ejecutando](./media/gateway/gateway-service.png)

## <a name="install-the-log-analytics-gateway-using-the-command-line"></a>Instalación de la puerta de enlace de Log Analytics mediante la línea de comandos

El archivo descargado para la puerta de enlace es un paquete de Windows Installer que admite la instalación silenciosa desde la línea de comandos u otro método automatizado. Si no está familiarizado con las opciones de línea de comandos estándar de Windows Installer, consulte [Opciones de línea de comandos](/windows/desktop/msi/command-line-options).
 
En la tabla siguiente se resaltan los parámetros admitidos por el programa de instalación.

|Parámetros| Notas|
|----------|------| 
|PORTNUMBER | Número de puerto TCP para que la puerta de enlace realice la escucha. |
|PROXY | Dirección IP del servidor proxy. |
|INSTALLDIR | Ruta de acceso completa para especificar el directorio de instalación de archivos de software de puerta de enlace. |
|USERNAME | Id. de usuario para autenticarse con el servidor proxy |
|PASSWORD | Contraseña del Id. de usuario para autenticarse con el proxy |
|LicenseAccepted | Especifique un valor de **1** para comprobar que acepta el contrato de licencia. |
|HASAUTH | Especifique un valor de **1** cuando se especifiquen los parámetros USERNAME/PASSWORD. |
|HASPROXY | Especifique un valor de **1** cuando establezca la dirección IP para el parámetro **PROXY**. |

Para realizar una instalación silenciosa de la puerta de enlace y configurarla con un número de puerto y una dirección de proxy específicos, escriba lo siguiente:

```dos
Msiexec.exe /I "oms gateway.msi" /qn PORTNUMBER=8080 PROXY="10.80.2.200" HASPROXY=1 LicenseAccepted=1 
```

Con la opción de línea de comandos/qn se oculta el programa de instalación, con /qb se muestra el programa de instalación durante la instalación silenciosa.  

Si tiene que proporcionar credenciales para autenticarse con el proxy, escriba lo siguiente:

```dos
Msiexec.exe /I "oms gateway.msi" /qn PORTNUMBER=8080 PROXY="10.80.2.200" HASPROXY=1 HASAUTH=1 USERNAME="<username>" PASSWORD="<password>" LicenseAccepted=1 
```

Después de la instalación, puede confirmar que se ha aceptado la configuración (excluido el nombre de usuario y contraseña) mediante los siguientes cmdlets de PowerShell:

- **Get-OMSGatewayConfig**: devuelve el puerto TCP para el que se ha configurado la escucha de la puerta de enlace.
- **Get-OMSGatewayRelayProxy**: devuelve la dirección IP del servidor proxy configurado con el que comunicarse.

## <a name="configure-network-load-balancing"></a>Configuración de equilibrio de carga de red

Puede configurar la puerta de enlace para lograr alta disponibilidad con equilibrio de carga de red (NLB) mediante [Equilibrio de carga de red de (NLB)](/windows-server/networking/technologies/network-load-balancing) de Microsoft, [Azure Load Balancer](../../load-balancer/load-balancer-overview.md) o equilibradores de carga basados en hardware. El equilibrador de carga administra el tráfico al redirigir las conexiones solicitadas desde los agentes de Log Analytics o los servidores de administración de Operations Manager mediante sus nodos. Si un servidor de puerta de enlace deja de funcionar, el tráfico se redirige a otros nodos.

### <a name="microsoft-network-load-balancing"></a>Equilibrio de carga de red de Microsoft

Para más información sobre cómo diseñar e implementar un clúster de equilibrio de carga de red de Windows Server 2016, consulte [Equilibrio de carga de red](/windows-server/networking/technologies/network-load-balancing). Los pasos siguientes describen cómo configurar un clúster de equilibrio de carga de red de Microsoft.  

1. Inicie sesión en el servidor Windows que sea miembro del clúster NLB con una cuenta administrativa.  
2. Abra el Administrador de equilibrio de carga de red en el Administrador del servidor, haga clic en **Herramientas** y, luego, en **Administrador de equilibrio de carga de red**.
3. Para conectar un servidor de puerta de enlace de Log Analytics con Microsoft Monitoring Agent instalado, haga clic con el botón derecho en la dirección IP del clúster y, después, seleccione **Agregar host al clúster**. 

    ![Administrador de equilibrio de carga de red: Agregar host al clúster](./media/gateway/nlb02.png)
 
4. Escriba la dirección IP del servidor de la puerta de enlace al que desea conectarse. 

    ![Administrador de equilibrio de carga de red: Agregar host al clúster: Conectar](./media/gateway/nlb03.png) 

### <a name="azure-load-balancer"></a>Azure Load Balancer

Para obtener información sobre cómo diseñar e implementar Azure Load Balancer, consulte [¿Qué es Azure Load Balancer?](../../load-balancer/load-balancer-overview.md) Para implementar un equilibrador de carga básico, siga los pasos descritos en esta [guía de inicio rápido](../../load-balancer/quickstart-load-balancer-standard-public-portal.md), excepto los pasos descritos en la sección **Creación de servidores back-end**.   

> [!NOTE]
> La configuración de Azure Load Balancer mediante la **SKU básica** requiere que las máquinas virtuales de Azure pertenezcan a un conjunto de disponibilidad. Para obtener más información sobre los conjuntos de disponibilidad, consulte [Administración de la disponibilidad de las máquinas virtuales Windows en Azure](../../virtual-machines/windows/manage-availability.md). Para agregar máquinas virtuales existentes a un conjunto de disponibilidad, consulte [Configuración del conjunto de disponibilidad de Azure Resource Manager](https://gallery.technet.microsoft.com/Set-Azure-Resource-Manager-f7509ec4).
> 

Después de crear el equilibrador de carga, debe crearse un grupo back-end, que distribuye el tráfico a uno o más servidores de puerta de enlace. Siga los pasos descritos en la sección del artículo de la guía de inicio rápido [Creación de recursos para el equilibrador de carga](../../load-balancer/quickstart-load-balancer-standard-public-portal.md).  

>[!NOTE]
>Cuando configure el sondeo de estado, debe configurarse para usar el puerto TCP del servidor de puerta de enlace. El sondeo de estado agrega o quita de forma dinámica los servidores de puerta de enlace de la rotación de Load Balancer en base a su respuesta a las comprobaciones de estado. 
>

## <a name="configure-the-log-analytics-agent-and-operations-manager-management-group"></a>Configuración del agente de Log Analytics y el grupo de administración de Operations Manager

En esta sección, se indicarán los pasos para configurar los agentes de Log Analytics conectados directamente, un grupo de administración de Operations Manager o Hybrid Runbook Workers de Azure Automation con la puerta de enlace de Log Analytics para comunicarse con Azure Automation o Log Analytics.  

### <a name="configure-a-standalone-log-analytics-agent"></a>Configuración del agente de Log Analytics independiente

Cuando configure el agente de Log Analytics, cambie el valor de servidor de proxy por la dirección IP del servidor de puerta de enlace de Log Analytics y el número de puerto. Si ha implementado varios servidores de puerta de enlace detrás de un equilibrador de carga, la configuración de proxy del agente de Log Analytics es la dirección IP virtual del equilibrador de carga.  

>[!NOTE]
>Para instalar el agente de Log Analytics en la puerta de enlace y los equipos Windows que se conectan directamente a Log Analytics, consulte [Conexión de equipos Windows al servicio Log Analytics de Azure](agent-windows.md). Para los equipos Linux, consulte [Conexión de equipos Linux a Azure Monitor](agent-linux.md). 
>

Después de instalar al agente en el servidor de puerta de enlace, configúrelo para que informe al área de trabajo o a los agentes de las áreas de trabajo que se comunican con la puerta de enlace. Si el agente de Windows de Log Analytics no está instalado en la puerta de enlace, se escribe el evento 300 en el registro de eventos de la puerta de enlace de OMS con un mensaje que indica que el agente debe instalarse. Si el agente está instalado, pero no está configurado para informar a la misma área de trabajo que los agentes que se comunican a través de ella, se escribirá el evento 105 en el mismo registro de eventos indicando que el agente de la puerta de enlace debe configurarse para llevar a cabo dicha comunicación.

Después de completar la configuración, reinicie el servicio **Puerta de enlace de OMS** para aplicar los cambios. En caso contrario, la puerta de enlace rechazará los intentos de los agentes por comunicarse con Log Analytics e informará sobre el evento 105 en el registro de eventos de puerta de enlace de OMS. Esto se aplicará también al agregar o quitar un área de trabajo de la configuración de agente en el servidor de puerta de enlace.

Para obtener información relacionada con Hybrid Runbook Worker de Automation, consulte [Automatización de recursos en los centros de datos o nube con Hybrid Runbook Worker](../../automation/automation-hybrid-runbook-worker.md).

### <a name="configure-operations-manager-where-all-agents-use-the-same-proxy-server"></a>Configuración de Operations Manager: todos los agentes usan el mismo servidor proxy

La configuración de proxy de Operations Manager se aplica automáticamente a todos los agentes que informan a Operations Manager, incluso si la configuración está vacía.  

Si desea utilizar la puerta de enlace de OMS para que sea compatible con Operations Manager, debe tener:

* Microsoft Monitoring Agent (versión 8.0.10900.0 o posterior) instalado en el servidor de la puerta de enlace de OMS y configurado con las mismas áreas de trabajo de Log Analytics que configuró para que el grupo de administración enviara informes.
* Conectividad de Internet. De forma alternativa, la puerta de enlace de OMS debe estar conectada a un servidor proxy conectado a Internet.

> [!NOTE]
> Si no especifica ningún valor para la puerta de enlace, se insertan valores en blanco para todos los agentes.
>

Si el grupo de administración de Operations Manager se registra con un área de trabajo de Log Analytics por primera vez, la opción para especificar la configuración de proxy para el grupo de administración no estará disponible en la consola del operador. Esta opción solo está disponible si el grupo de administración se ha registrado con el servicio.  

Para configurar la integración, actualice la configuración de proxy del sistema con Netsh en el sistema en el que se esté ejecutando la consola del operador y en todos los servidores de administración del grupo de administración. Siga estos pasos:

1. Abra un símbolo del sistema con privilegios elevados:

   a. Seleccione **Inicio** y especifique **cmd**.  

   b. Haga clic con el botón derecho en el **símbolo del sistema** y seleccione **Ejecutar como administrador**.  

1. Escriba el comando siguiente:

   `netsh winhttp set proxy <proxy>:<port>`

Después de completar la integración con Log Analytics, quite el cambio ejecutando `netsh winhttp reset proxy`. A continuación, en la consola de operación, use la opción **Configurar el servidor proxy** para especificar el servidor de puerta de enlace de Log Analytics. 

1. En la consola de Operations Manager, en **Operations Management Suite**, seleccione **Conexión** y, a continuación, **Configurar servidor proxy**.

   ![Captura de pantalla de Operations Manager, que muestra la selección de la opción Configurar servidor proxy](./media/gateway/scom01.png)

1. Seleccione **Use un servidor proxy para obtener acceso al servicio Operations Management Suite** y, después, escriba la dirección IP del servidor de puerta de enlace de Log Analytics o la dirección IP virtual del equilibrador de carga. Tenga cuidado al comenzar con el prefijo `http://`.

   ![Captura de pantalla de Operations Manager, que muestra la dirección de servidor proxy](./media/gateway/scom02.png)

1. Seleccione **Finalizar**. El grupo de administración de Operations Manager está ahora configurado para comunicarse mediante el servidor de puerta de enlace con el servicio Log Analytics.

### <a name="configure-operations-manager-where-specific-agents-use-a-proxy-server"></a>Configuración de Operations Manager: determinados agentes usan un servidor proxy

En entornos grandes o complejos, puede que solo quiera tener determinados servidores (o grupos) que usen el servidor de puerta de enlace de Log Analytics.  En estos servidores, no se puede actualizar el agente Operations Manager directamente, ya que el valor global para el grupo de administración sobrescribe este valor.  En su lugar, invalide la regla utilizada para insertar estos valores.  

> [!NOTE] 
> Use esta técnica de configuración si desea permitir varios servidores de puerta de enlace de Log Analytics en su entorno.  Por ejemplo, puede exigir que se especifiquen determinados servidores de puerta de enlace de Log Analytics en cada región.
>

Para configurar servidores o grupos específicos que utilicen el servidor de puerta de enlace de Log Analytics: 

1. Abra la consola de Operations Manager y seleccione el área de trabajo **Creación**.  
1. En el área de trabajo Creación, seleccione **Reglas**. 
1. Seleccione el botón **Ámbito** en barra de herramientas de Operations Manager. Si este botón no está disponible, asegúrese de que ha seleccionado un objeto, y no una carpeta, en el panel **Supervisión**. El cuadro de diálogo **Objetos de módulo de administración de ámbito** muestra una lista de clases, grupos u objetos de destino comunes. 
1. Escriba **Servicio de mantenimiento** en el campo **Buscar** y selecciónelo en la lista. Seleccione **Aceptar**.  
1. Busque **Regla de configuración de proxy de Advisor**. 
1. En la barra de herramientas de Operations Manager, seleccione **Invalidaciones** y luego vaya a **Invalidar la regla\Para un objeto de clase específico: Servicio de mantenimiento** y seleccione un objeto de la lista.  También puede crear un grupo personalizado que incluya el objeto de servicio de mantenimiento de los servidores a los que desea aplicar esta invalidación. Después, aplique la invalidación a ese grupo personalizado.
1. En el cuadro de diálogo **Propiedades de invalidación**, agregue una marca de verificación en la columna **Invalidar** junto al parámetro **WebProxyAddress**.  En el campo **Valor de invalidación**, escriba la dirección URL del servidor de puerta de enlace de Log Analytics. Tenga cuidado al comenzar con el prefijo `http://`.  

    >[!NOTE]
    > No es necesario habilitar la regla. Esta ya se administra automáticamente con una invalidación en el módulo de administración de reemplazo de referencia segura de Microsoft System Center Advisor dirigido al grupo de servidores de supervisión de Microsoft System Center Advisor.
    > 

1. Seleccione un módulo de administración de la lista **Seleccionar módulo de administración de destino** o cree un nuevo módulo de administración no sellado seleccionando **Nuevo**. 
1. Cuando termine, seleccione **Aceptar**. 

### <a name="configure-for-automation-hybrid-runbook-workers"></a>Configuración de Hybrid Runbook Workers de Automation

Si tiene Hybrid Runbook Workers de Azure Automation en su entorno, siga estos pasos para configurar la puerta de enlace de OMS de modo que sea compatible con los trabajos.

Consulte la sección sobre [configuración de la red](../../automation/automation-hybrid-runbook-worker.md#network-planning) de la documentación de Automation para buscar la dirección URL de cada región.

Si el equipo se registra automáticamente como Hybrid Runbook Worker, por ejemplo, si la solución de administración de actualizaciones está habilitada para una o más máquinas virtuales, siga estos pasos:

1. Agregue las direcciones URL del servicio de datos en tiempo de ejecución del trabajo a la lista de hosts permitidos de puerta de enlace de Log Analytics. Por ejemplo: `Add-OMSGatewayAllowedHost we-jobruntimedata-prod-su1.azure-automation.net`
1. Reinicie el servicio de puerta de enlace de Log Analytics mediante el siguiente cmdlet de PowerShell: `Restart-Service OMSGatewayService`

Si el equipo está incorporado en Azure Automation mediante el cmdlet de registro de Hybrid Runbook Worker, siga estos pasos:

1. Agregue la dirección URL de registro del servicio Agente a la lista de hosts permitidos de la puerta de enlace de Log Analytics. Por ejemplo: `Add-OMSGatewayAllowedHost ncus-agentservice-prod-1.azure-automation.net`
1. Agregue las direcciones URL del servicio de datos en tiempo de ejecución del trabajo a la lista de hosts permitidos de puerta de enlace de Log Analytics. Por ejemplo: `Add-OMSGatewayAllowedHost we-jobruntimedata-prod-su1.azure-automation.net`
1. Reinicie el servicio de puerta de enlace de Log Analytics.
    `Restart-Service OMSGatewayService`

## <a name="useful-powershell-cmdlets"></a>Cmdlets de PowerShell útiles

Puede usar cmdlets para completar las tareas necesarias para actualizar las opciones de configuración de la puerta de enlace de Log Analytics. Para poder usar cmdlets, asegúrese de lo siguiente:

1. Instale la puerta de enlace de Log Analytics (Microsoft Windows Installer).
1. Abra una ventana de la consola de PowerShell.
1. Para importar el módulo, escriba este comando: `Import-Module OMSGateway`.
1. Si no se ha producido ningún error en el paso anterior, el módulo se ha importado correctamente y se pueden usar los cmdlets. Escriba `Get-Module OMSGateway`.
1. Después de usar los cmdlets para realizar cambios, reinicie el servicio de puerta de enlace de OMS.

Un error en el paso 3 implica que el módulo no se ha importado. Este error podría producirse si PowerShell no puede encontrar el módulo. Puede encontrar el módulo en la ruta de instalación de puerta de enlace de OMS: *C:\Archivos de programa\Microsoft OMS Gateway\PowerShell\OmsGateway*.

| **Cmdlet** | **Parámetros** | **Descripción** | **Ejemplo** |
| --- | --- | --- | --- |  
| `Get-OMSGatewayConfig` |Clave |Obtiene la configuración del servicio |`Get-OMSGatewayConfig` |  
| `Set-OMSGatewayConfig` |Clave (se requiere) <br> Value |Cambia la configuración del servicio |`Set-OMSGatewayConfig -Name ListenPort -Value 8080` |  
| `Get-OMSGatewayRelayProxy` | |Obtiene la dirección del proxy de retransmisión (ascendente) |`Get-OMSGatewayRelayProxy` |  
| `Set-OMSGatewayRelayProxy` |Dirección<br> Nombre de usuario<br> Contraseña (cadena segura) |Establece la dirección (y credencial) del proxy de retransmisión (ascendente) |1. Establezca un proxy de retransmisión y la credencial:<br> `Set-OMSGatewayRelayProxy`<br>`-Address http://www.myproxy.com:8080`<br>`-Username user1 -Password 123` <br><br> 2. Establezca un proxy de retransmisión que no necesite autenticación: `Set-OMSGatewayRelayProxy`<br> `-Address http://www.myproxy.com:8080` <br><br> 3. Desactive la configuración del proxy de retransmisión:<br> `Set-OMSGatewayRelayProxy` <br> `-Address ""` |  
| `Get-OMSGatewayAllowedHost` | |Obtiene el host permitido actualmente (solo el host permitido configurado localmente, no hosts permitidos que se descargan automáticamente) |`Get-OMSGatewayAllowedHost` | 
| `Add-OMSGatewayAllowedHost` |Host (obligatorio) |Agrega el host a la lista de permitidos |`Add-OMSGatewayAllowedHost -Host www.test.com` |  
| `Remove-OMSGatewayAllowedHost` |Host (obligatorio) |Quita el host de la lista de permitidos |`Remove-OMSGatewayAllowedHost`<br> `-Host www.test.com` |  
| `Add-OMSGatewayAllowedClientCertificate` |Asunto (obligatorio) |Agrega el asunto del certificado de cliente a la lista de permitidos |`Add-OMSGatewayAllowed`<br>`ClientCertificate` <br> `-Subject mycert` |  
| `Remove-OMSGatewayAllowedClientCertificate` |Asunto (obligatorio) |Quita el asunto del certificado de cliente de la lista de permitidos |`Remove-OMSGatewayAllowed` <br> `ClientCertificate` <br> `-Subject mycert` |  
| `Get-OMSGatewayAllowedClientCertificate` | |Obtiene los asuntos de certificado de cliente permitidos actualmente (solo los asuntos permitidos configurados localmente, no los asuntos permitidos que se descargan automáticamente) |`Get-`<br>`OMSGatewayAllowed`<br>`ClientCertificate` |  

## <a name="troubleshooting"></a>Solución de problemas

Para recopilar eventos registrados por la puerta de enlace, debe tener instalado el agente de Log Analytics.

![Captura de pantalla de la lista de Visor de eventos en el registro de puerta de enlace de Log Analytics](./media/gateway/event-viewer.png)

### <a name="log-analytics-gateway-event-ids-and-descriptions"></a>Identificadores de evento y descripciones de la puerta de enlace de Log Analytics

En la tabla siguiente, se muestran los identificadores de evento y las descripciones de los eventos de registro de la puerta de enlace de Log Analytics.

| **ID** | **Descripción** |
| --- | --- |
| 400 |Cualquier error de la aplicación que no tiene un identificador específico. |
| 401 |Configuración errónea. Por ejemplo, listenPort = "text", en lugar de un entero. |
| 402 |Excepción al analizar los mensajes de protocolo de enlace TLS. |
| 403 |Error de red. Por ejemplo, no se puede conectar al servidor de destino. |
| 100 |Información general. |
| 101 |El servicio se ha iniciado. |
| 102 |El servicio se ha detenido. |
| 103 |Se ha recibido un comando HTTP CONNECT del cliente. |
| 104 |No es un comando HTTP CONNECT. |
| 105 |El servidor de destino no está en la lista de permitidos o el puerto de destino no es seguro (443). <br> <br> Asegúrese de que el agente MMA del servidor de puerta de enlace de OMS y los agentes que se comunican con la puerta de enlace de OMS están conectados a la misma área de trabajo de Log Analytics. |
| 105 |ERROR TcpConnection – Invalid Client certificate: CN=Gateway. <br><br> Asegúrese de que está usando la versión de puerta de enlace de OMS 1.0.395.0 o superior. Asegúrese también de que el agente MMA del servidor de puerta de enlace de OMS y los agentes que se comunican con la puerta de enlace de OMS están conectados a la misma área de trabajo de Log Analytics. |
| 106 |Versión del protocolo TLS/SSL no admitida.<br><br> La puerta de enlace de Log Analytics admite TLS 1.0, 1.1 y 1.2. No admite SSL.|
| 107 |Se ha comprobado la sesión de TLS. |

### <a name="performance-counters-to-collect"></a>Contadores de rendimiento que se recopilan

En la tabla siguiente, se muestran los contadores de rendimiento disponibles para la puerta de enlace de Log Analytics. Utilice el Monitor de rendimiento para agregar contadores.

| **Nombre** | **Descripción** |
| --- | --- |
| Puerta de enlace de Log Analytics/conexión de cliente activa |Número de conexiones de red (TCP) de cliente activas |
| Puerta de enlace de Log Analytics/número de errores |Número de errores |
| Puerta de enlace de Log Analytics/cliente conectado |Número máximo de clientes conectados |
| Puerta de enlace de Log Analytics/número de rechazos |Número de rechazos debido a un error de validación de TLS |

![Captura de pantalla de la interfaz de puerta de enlace de Log Analytics, que muestra los contadores de rendimiento](./media/gateway/counters.png)

## <a name="assistance"></a>Asistencia

Después de iniciar sesión en Azure Portal, puede obtener ayuda con la puerta de enlace de Log Analytics o con cualquier otro servicio o característica de Azure.
Para solicitar asistencia, seleccione el icono de signo de interrogación en la esquina superior derecha del portal y seleccione **Nueva solicitud de soporte técnico**. A continuación, complete el formulario de solicitud de soporte técnico nuevo.

![Captura de pantalla de una nueva solicitud de soporte técnico](./media/gateway/support.png)

## <a name="next-steps"></a>Pasos siguientes

[Agregue orígenes de datos](./agent-data-sources.md) para recopilar datos de los orígenes conectados y almacenar los datos en el área de trabajo de Log Analytics.

