---
title: 'Inicio rápido: Recopilación de datos de un equipo Linux híbrido con Azure Monitor'
description: En este inicio rápido, aprenderá a implementar el agente de Log Analytics para equipos Linux que se ejecutan fuera de Azure y a habilitar la recopilación de datos con registros de Azure Monitor.
services: azure-monitor
documentationcenter: azure-monitor
author: bwren
manager: carmonm
editor: ''
ms.assetid: ''
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: quickstart
ms.date: 12/24/2019
ms.author: bwren
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019
ms.openlocfilehash: b856381e8d085c6839effd1de6d9867b214d24ee
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020395"
---
# <a name="quickstart-collect-data-from-a-linux-computer-in-a-hybrid-environment-with-azure-monitor"></a>Inicio rápido: Recopilación de datos de un equipo Linux en un entorno híbrido con Azure Monitor

[Azure Monitor](../overview.md) puede recopilar datos directamente de los equipos Linux físicos o virtuales del entorno en un área de trabajo de Log Analytics para una correlación y un análisis detallados. La instalación del [agente de Log Analytics](../platform/log-analytics-agent.md) permite a Azure Monitor recopilar datos de un centro de datos o de otro entorno en la nube. En esta guía de inicio rápido se muestra cómo configurar y recopilar datos del servidor Linux con unos pasos sencillos. Para más información sobre las máquinas virtuales Linux de Azure, consulte [Recopilación de datos acerca de máquinas virtuales de Azure](./quick-collect-azurevm.md).  

Para conocer la configuración admitida, consulte [Sistemas operativos admitidos](../platform/agents-overview.md#supported-operating-systems) y [Configuración del firewall de red](../platform/log-analytics-agent.md#network-requirements).
 
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com). 

## <a name="create-a-workspace"></a>Crear un área de trabajo

1. En Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba **Log Analytics**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Áreas de trabajo de Log Analytics**.

    ![Búsqueda de un área de trabajo de Log Analytics en Azure Portal](media/quick-collect-azurevm/azure-portal-log-analytics-workspaces.png)<br>  

2. Seleccione **Crear** y, después, seleccione opciones para los elementos siguientes:

   * Proporcione el nombre de la nueva **área de trabajo de Log Analytics** como, por ejemplo, *DefaultLAWorkspace*.  
   * Seleccione una **suscripción** a la que vincularlo en la lista desplegable si la opción predeterminada seleccionada no es adecuada.
   * Para **Grupo de recursos**, seleccione un grupo de recursos existente que contenga una o más máquinas virtuales de Azure.  
   * Seleccione la **Ubicación** en que están implementadas las VM.  Para obtener más información, consulte en qué [regiones está disponible Log Analytics](https://azure.microsoft.com/regions/services/).
   * Si va a crear un área de trabajo en una nueva suscripción creada después del 2 de abril de 2018, esta utilizará automáticamente el plan de precios *Por GB* y la opción para seleccionar un plan de tarifas no estará disponible.  Si va a crear un área de trabajo para una suscripción existente creada antes del 2 de abril o para una suscripción asociada a una inscripción de EA existente, seleccione el plan de tarifa que prefiera.  Para obtener más información sobre planes concretos, consulte los [detalles de precios de Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/).
  
        ![Creación de un área de trabajo de Log Analytics en Azure Portal](media/quick-collect-azurevm/create-log-analytics-workspace-azure-portal.png) 

3. Después de proporcionar la información necesaria en el panel **Área de trabajo de Log Analytics**, seleccione **Aceptar**.  

Mientras se comprueba la información y se crea el espacio de trabajo, puede realizar un seguimiento de su progreso en **Notificaciones** en el menú. 

## <a name="obtain-workspace-id-and-key"></a>Obtención de la clave y el identificador de área de trabajo

Antes de instalar el agente Log Analytics para Linux, necesita la clave y el identificador de área de trabajo para el área de trabajo de Log Analytics. El script contenedor del agente necesita esta información para configurar el agente de la forma adecuada y asegurarse de que puede comunicarse correctamente con Azure Monitor.

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]  

1. En la esquina superior izquierda de Azure Portal, seleccione **Todos los servicios**. En el cuadro de búsqueda, escriba **Log Analytics**. La lista se filtra en función de lo que escriba. Seleccione **Áreas de trabajo de Log Analytics**.

2. En la lista de áreas de trabajo de Log Analytics, seleccione la que creó anteriormente. (Puede que le haya asignado el nombre **DefaultLAWorkspace**).

3. Seleccione **Configuración avanzada**.

    ![Menú Opciones avanzadas de Log Analytics en Azure Portal](media/quick-collect-azurevm/log-analytics-advanced-settings-azure-portal.png) 
 
4. Seleccione **Orígenes conectados** y **Servidores Linux**.

5. Encontrará los valores a la derecha de **Id. del área de trabajo** y **Clave principal**. Copie y pegue ambos valores en el editor que prefiera.

## <a name="install-the-agent-for-linux"></a>Instalación del agente para Linux

Los pasos siguientes configuran la instalación del agente de Log Analytics en Azure y en la nube de Azure Government.  

>[!NOTE]
>No se puede configurar el agente de Log Analytics para Linux para informar a varias áreas de trabajo de Log Analytics.  

Si el equipo Linux necesita comunicarse mediante un servidor proxy con Log Analytics, la configuración del proxy puede especificarse en la línea de comandos mediante la inclusión de `-p [protocol://][user:password@]proxyhost[:port]`.  La propiedad *proxyhost* acepta un nombre de dominio completo o la dirección IP del servidor proxy. 

Por ejemplo: `https://user01:password@proxy01.contoso.com:30443`

1. Para configurar el equipo Linux de modo que se conecte a un área de trabajo de Log Analytics, ejecute el comando siguiente con el identificador del área de trabajo y la clave principal que ha copiado anteriormente. El siguiente comando descarga el agente, valida su suma de comprobación y lo instala. 
    
    ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY>
    ```

    El siguiente comando incluye el parámetro de proxy `-p` y la sintaxis de ejemplo necesaria cuando el servidor proxy requiere autenticación:

   ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -p [protocol://][user:password@]proxyhost[:port] -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY>
    ```

2. Para configurar el equipo Linux de modo que se conecte con un área de trabajo de Log Analytics en la nube de Azure Government, ejecute el comando siguiente con el identificador del área de trabajo y la clave principal que ha copiado anteriormente. El siguiente comando descarga el agente, valida su suma de comprobación y lo instala. 

    ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY> -d opinsights.azure.us
    ``` 

    El siguiente comando incluye el parámetro de proxy `-p` y la sintaxis de ejemplo necesaria cuando el servidor proxy requiere autenticación:

   ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -p [protocol://][user:password@]proxyhost[:port] -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY> -d opinsights.azure.us
    ```

3. Ejecute el comando siguiente para reiniciar el agente: 

    ```
    sudo /opt/microsoft/omsagent/bin/service_control restart [<workspace id>]
    ``` 

## <a name="collect-event-and-performance-data"></a>Recopilación de datos de eventos y rendimiento

Azure Monitor puede recopilar eventos de Linux Syslog y de los contadores de rendimiento que especifique para el análisis y la generación de informes a más largo plazo. También puede realizar acciones cuando detecta una condición determinada. Siga estos pasos para configurar la recopilación de eventos desde Syslog de Linux, así como desde varios contadores de rendimiento comunes, para empezar.  

1. En Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba Log Analytics. La lista se filtra en función de lo que escriba. Seleccione **Áreas de trabajo de Log Analytics** y, en la lista, seleccione el área de trabajo que busca y elija **Configuración avanzada** del área de trabajo de **Log Analytics**.

2. Seleccione **Datos** y, después, seleccione **Syslog**.  

3. Para agregar una instancia de syslog, escriba el nombre del registro. Escriba **Syslog** y, luego, seleccione el signo más **+** .  

4. En la tabla, desactive los niveles de gravedad **Información**, **Aviso** y **Depurar**. 

5. Seleccione **Guardar** en la parte superior de la página para guardar la configuración.

6. Seleccione **Linux Performance Data** (Datos de rendimiento de Linux) para habilitar la recopilación de contadores de rendimiento en un equipo Linux. 

7. La primera vez que se configuran los contadores de rendimiento Linux para un área de trabajo de Log Analytics nueva, se ofrece la opción de crear rápidamente varios contadores comunes. Se muestran todos con una casilla junto a cada uno.

    ![Contadores de rendimiento de Linux predeterminados seleccionados en Azure Monitor](media/quick-collect-azurevm/linux-perfcounters-azure-monitor.png)

    Seleccione **Aplicar la configuración siguiente a mis máquinas** y, luego, **Agregar los contadores de rendimiento seleccionados**. Se agregan con el valor preestablecido de un intervalo de ejemplo de recopilación de diez segundos.  

8. Seleccione **Guardar** en la parte superior de la página para guardar la configuración.

## <a name="view-data-collected"></a>Ver datos recopilados

Ahora que ya ha habilitado la recopilación de datos, vamos a ver un sencillo ejemplo de búsqueda de registros para consultar algunos datos del equipo de destino.  

1. En el área de trabajo seleccionada, en el panel izquierdo, seleccione **Registros**.

2. En la página de consulta de registros, escriba `Perf` en el editor de consultas y seleccione **Ejecutar**.
 
    ![Búsqueda de registros de Log Analytics](media/quick-collect-windows-computer/log-analytics-portal-queryexample.png)

    Por ejemplo, la consulta de la imagen siguiente ha devuelto 10 000 registros de rendimiento. Sus resultados serán significativamente inferiores.

    ![Resultado de búsqueda de registros de Log Analytics](media/quick-collect-linux-computer/log-analytics-search-perf.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede quitar el agente del equipo Linux y eliminar el área de trabajo de Log Analytics.  

Para quitar el agente, ejecute el siguiente comando en el equipo Linux. El argumento *--purge* quita completamente el agente y su configuración.

   `wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh --purge`

Para eliminar el área de trabajo, seleccione el área de trabajo de Log Analytics que ha creado anteriormente y, en la página de recursos, seleccione **Eliminar**.

![Eliminación de recursos de Log Analytics](media/quick-collect-azurevm/log-analytics-portal-delete-resource.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya recopila datos sobre el funcionamiento y el rendimiento del equipo Linux local, le resultará muy fácil empezar a explorar y analizar los datos que se recopilan, además de tomar las medidas correspondientes a partir de estos. Todo ello, *de forma gratuita*.  

Para obtener información sobre cómo ver y analizar los datos, continúe con el tutorial.

> [!div class="nextstepaction"]
> [Ver o analizar datos en Log Analytics](../log-query/get-started-portal.md)

