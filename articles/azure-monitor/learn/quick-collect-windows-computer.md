---
title: Recopilación de datos de un equipo Windows híbrido con Azure Monitor
description: En este inicio rápido, aprenderá a implementar el agente de Log Analytics para equipos Windows que se ejecutan fuera de Azure y a habilitar la recopilación de datos con registros de Azure Monitor.
services: azure-monitor
documentationcenter: azure-monitor
author: bwren
manager: carmonm
editor: ''
ms.assetid: ''
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: quickstart
ms.date: 08/22/2019
ms.author: bwren
ms.custom: mvc, seo-javascript-october2019
ms.openlocfilehash: 726caa0c4a7bbd6b8d0c17ffdfade5f9cc3c2a34
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020378"
---
# <a name="collect-data-from-a-windows-computer-in-a-hybrid-environment-with-azure-monitor"></a>Recopilación de datos de un equipo Windows en un entorno híbrido con Azure Monitor

[Azure Monitor](../overview.md) puede recopilar datos directamente de los equipos Windows físicos o virtuales de su entorno en un área de trabajo de Log Analytics para lograr una correlación y un análisis detallados. La instalación del [agente de Log Analytics](../platform/log-analytics-agent.md) permite a Azure Monitor recopilar datos de un centro de datos o de otro entorno en la nube. En esta guía de inicio rápido se muestra cómo configurar y recopilar datos de equipos Windows con unos pasos sencillos. Para información sobre las máquinas virtuales Windows de Azure, consulte [Recopilación de datos acerca de máquinas virtuales de Azure](./quick-collect-azurevm.md).  

Para conocer la configuración admitida, consulte [Sistemas operativos admitidos](../platform/agents-overview.md#supported-operating-systems) y [Configuración del firewall de red](../platform/log-analytics-agent.md#network-requirements).
 
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com). 

## <a name="create-a-workspace"></a>Crear un área de trabajo

1. En Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba **Log Analytics**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Áreas de trabajo de Log Analytics**.

    ![Portal de Azure](media/quick-collect-azurevm/azure-portal-log-analytics-workspaces.png)<br>  

2. Seleccione **Crear** y, después, seleccione opciones para los elementos siguientes:

   * Proporcione el nombre de la nueva **área de trabajo de Log Analytics** como, por ejemplo, *DefaultLAWorkspace*.  
   * Seleccione una **suscripción** a la que vincularlo en la lista desplegable si la opción predeterminada seleccionada no es adecuada.
   * Para **Grupo de recursos**, seleccione un grupo de recursos existente que contenga una o más máquinas virtuales de Azure.  
   * Seleccione la **Ubicación** en que están implementadas las VM.  Para obtener más información, consulte en qué [regiones está disponible Log Analytics](https://azure.microsoft.com/regions/services/).
   * Si va a crear un área de trabajo en una nueva suscripción creada después del 2 de abril de 2018, esta utilizará automáticamente el plan de precios *Por GB* y la opción para seleccionar un plan de tarifas no estará disponible.  Si va a crear un área de trabajo para una suscripción existente creada antes del 2 de abril o para una suscripción asociada a una inscripción de EA existente, seleccione el plan de tarifa que prefiera.  Para obtener más información sobre planes concretos, consulte los [detalles de precios de Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/).
  
        ![Creación de una hoja de recursos de Log Analytics](media/quick-collect-azurevm/create-log-analytics-workspace-azure-portal.png) 

3. Después de proporcionar la información necesaria en el panel **Área de trabajo de Log Analytics**, seleccione **Aceptar**.  

Mientras se comprueba la información y se crea el espacio de trabajo, puede realizar un seguimiento de su progreso en **Notificaciones** en el menú. 


## <a name="get-the-workspace-id-and-key"></a>Obtención de la clave y el identificador del área de trabajo

Antes de instalar el agente de Log Analytics para Windows (también denominado Microsoft Monitoring Agent (MMA)), necesita la clave y el identificador del área de trabajo de Log Analytics. El Asistente para la instalación necesita esta información a fin de configurar el agente de la forma adecuada y asegurarse de que pueda comunicarse con Azure Monitor.  

1. En la esquina superior izquierda de Azure Portal, seleccione **Todos los servicios**. En el cuadro de búsqueda, escriba **Log Analytics**. La lista se filtra en función de lo que escriba. Seleccione **Áreas de trabajo de Log Analytics**.

2. En la lista de áreas de trabajo de Log Analytics, seleccione la que creó anteriormente. (Puede que le haya asignado el nombre **DefaultLAWorkspace**).

3. Seleccione **Configuración avanzada**.

    ![Configuración avanzada de Log Analytics](media/quick-collect-azurevm/log-analytics-advanced-settings-azure-portal.png)
  
4. Seleccione **Orígenes conectados** y **Servidores Windows**.

5. Copie los valores que aparecen a la derecha de **Id. del área de trabajo** y **Clave principal**. Péguelos en el editor que prefiera.

## <a name="install-the-agent-for-windows"></a>Instalación del agente para Windows

En los pasos siguientes se instala y configura el agente para Log Analytics en Azure y Azure Government. Usará el programa de instalación de Microsoft Monitoring Agent para instalar al agente en el equipo.

1. Continuando desde el conjunto anterior de pasos, en la página **Servidores de Windows**, seleccione la versión de **Descargar el agente de Windows** deseada. Seleccione la versión adecuada para la arquitectura de procesador del sistema operativo Windows.

2. Ejecute el programa de instalación para instalar al agente en el equipo.

3. En la página **principal**, seleccione **Siguiente**.

4. En la página **Términos de licencia**, lea la licencia y seleccione **Acepto**.

5. En la página **Carpeta de destino**, cambie o mantenga la carpeta de instalación predeterminada y seleccione **Siguiente**.

6. En la página **Opciones de instalación del agente**, conecte el agente a Azure Log Analytics y, luego, seleccione **Siguiente**.

7. En la página **Azure Log Analytics**, haga lo siguiente:

   1. Pegue el **identificador del área de trabajo** y la **clave del área de trabajo (clave principal)** que copió anteriormente. Si el equipo debe informar a un área de trabajo de Log Analytics en Azure Government, seleccione **Azure Gobierno de EE UU.** en la lista **Azure Cloud**.  
   2. Si el equipo necesita comunicarse a través de un servidor proxy con el servicio de Log Analytics, seleccione **Avanzado** y proporcione la dirección URL y el número de puerto del servidor proxy. Si el servidor proxy requiere autenticación, escriba el nombre de usuario y la contraseña para la autenticación con el servidor proxy y, luego, seleccione **Siguiente**.  

8. Después de agregar los valores de configuración, seleccione **Siguiente**:

    ![Instalación de Microsoft Monitoring Agent](media/quick-collect-windows-computer/log-analytics-mma-setup-laworkspace.png)

9. En la página **Listo para instalar**, revise las opciones elegidas y seleccione **Instalar**.

10. En la página **Configuración completada correctamente**, seleccione **Finalizar**.

Cuando finalice la instalación, Microsoft Monitoring Agent aparece en el Panel de Control. Puede revisar la configuración y verificar que el agente esté conectado al área de trabajo de Log Analytics. Al conectarse, en la pestaña **Azure Log Analytics**, el agente muestra este mensaje: **Microsoft Monitoring Agent se ha conectado correctamente al servicio Microsoft Log Analytics.**<br><br> ![Estado de la conexión de MMA](media/quick-collect-windows-computer/log-analytics-mma-laworkspace-status.png)

## <a name="collect-event-and-performance-data"></a>Recopilación de datos de eventos y rendimiento

Azure Monitor puede recopilar eventos que especifique del registro de eventos de Windows y de contadores de rendimiento para el análisis y la generación de informes a largo plazo. También puede realizar acciones cuando detecta una condición determinada. Siga estos pasos para configurar la colección de eventos desde el Registro de eventos de Windows, así como desde varios contadores de rendimiento comunes, para empezar.  

1. En la esquina inferior izquierda de Azure Portal, seleccione **Más servicios**. En el cuadro de búsqueda, escriba **Log Analytics**. La lista se filtra en función de lo que escriba. Seleccione **Áreas de trabajo de Log Analytics**.

2. Seleccione **Configuración avanzada**.

    ![Configuración avanzada de Log Analytics](media/quick-collect-azurevm/log-analytics-advanced-settings-azure-portal.png)
 
3. Seleccione **Datos** y, a continuación, **Registros de eventos de Windows**.  

4. Para agregar un registro de eventos, escriba el nombre del registro. Escriba **Sistema** y, luego, seleccione el signo más ( **+** ).  

5. En la tabla, seleccione los niveles de gravedad **Error** y **Advertencia**.

6. En la parte superior de la página, seleccione **Guardar**.

7. Seleccione **Windows Performance Counters** (Contadores de rendimiento de Windows) para habilitar la recopilación de contadores de rendimiento en un equipo Windows.

8. La primera vez que se configuran los contadores de rendimiento de Windows para una nueva área de trabajo de Log Analytics, se ofrece la opción de crear rápidamente varios contadores comunes. Cada opción que se muestra tiene al lado una casilla de verificación:

    ![Contadores de rendimiento de Windows](media/quick-collect-windows-computer/windows-perfcounters-default.png).
    
    Seleccione **Agregar los contadores de rendimiento seleccionados**. Los contadores se agregan con un valor preestablecido de un intervalo de ejemplo de recopilación de diez segundos.

9. En la parte superior de la página, seleccione **Guardar**.

## <a name="view-collected-data"></a>Visualización de los datos recopilados

Ahora que ya ha habilitado la recopilación de datos, vamos a ver una búsqueda de registros sencilla para consultar algunos datos del equipo de destino.  

1. En el área de trabajo seleccionada, en el panel izquierdo, seleccione **Registros**.

2. En la página de consulta de registros, escriba `Perf` en el editor de consultas y seleccione **Ejecutar**.
 
    ![Búsqueda de registros de Log Analytics](media/quick-collect-windows-computer/log-analytics-portal-queryexample.png)

    Por ejemplo, la consulta de esta imagen ha devuelto 10 000 registros de rendimiento. Sus resultados serán significativamente inferiores.

    ![Resultado de búsqueda de registros de Log Analytics](media/quick-collect-azurevm/log-analytics-search-perf.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede quitar el agente del equipo y eliminar el área de trabajo de Log Analytics.  

Para quitar al agente, haga lo siguiente:

1. Abra el Panel de control.

2. Abra **Programas y características**.

3. En **Programas y características**, seleccione **Microsoft Monitoring Agent** y, luego, haga clic en **Desinstalar**.

Para eliminar el área de trabajo de Log Analytics que creó anteriormente, selecciónela y, en la página de recursos, seleccione **Eliminar**:

![Eliminación del área de trabajo de Log Analytics](media/quick-collect-azurevm/log-analytics-portal-delete-resource.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha recopilado datos operativos y de rendimiento del equipo Windows, puede comenzar a explorar, analizar e interpretar esos datos fácilmente de forma *gratuita*.  

Para saber cómo ver y analizar los datos, continúe con el tutorial:

> [!div class="nextstepaction"]
> [Ver o analizar datos en Log Analytics](../log-query/get-started-portal.md)

