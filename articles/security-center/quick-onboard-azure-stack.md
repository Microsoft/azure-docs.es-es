---
title: Incorporación de máquinas virtuales de Azure Stack a Azure Security Center
description: En este tutorial de inicio rápido se muestra cómo aprovisionar la extensión de máquina virtual de Azure Monitor, Update and Configuration Management en una máquina virtual de Azure Stack.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 8982348a-0624-40c7-8a1e-642a523c7f6b
ms.service: security-center
ms.devlang: na
ms.topic: quickstart
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/02/2019
ms.author: memildin
ms.openlocfilehash: 7469d60095026c8a016fc1ecaa1a4f92361800f3
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019386"
---
# <a name="quickstart-onboard-your-azure-stack-virtual-machines-to-security-center"></a>Inicio rápido: Incorporación de máquinas virtuales de Azure Stack a Security Center
Después de incorporar la suscripción de Azure, puede habilitar Security Center para proteger las máquinas virtuales que se ejecutan en Azure Stack mediante la incorporación de la extensión de máquina virtual de **Azure Monitor, Update and Configuration Management** desde el Marketplace de Azure Stack.

En este tutorial de inicio rápido se muestra cómo agregar la extensión de máquina virtual **Azure Monitor, Update and Configuration Management** en una máquina virtual (se admiten tanto Linux como Windows) que se ejecuta en Azure Stack.

## <a name="prerequisites"></a>Prerrequisitos
Para empezar a trabajar con el Centro de seguridad, debe disponer de una suscripción a Microsoft Azure. Si no tiene una suscripción, puede registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).

Debe tener una suscripción de Azure en el nivel de servicio Estándar de Security Center para iniciar este tutorial de inicio rápido. Vea [Incorporación de su suscripción de Azure al nivel Estándar de Security Center](security-center-get-started.md) para obtener instrucciones de actualización. Dicho nivel de Security Center se puede probar de forma gratuita durante 30 días. Para más información, consulte la [página de precios](https://azure.microsoft.com/pricing/details/security-center/).

## <a name="select-your-workspace-in-azure-security-center"></a>Selección del área de trabajo en Azure Security Center

1. Inicie sesión en el [Portal de Azure](https://azure.microsoft.com/features/azure-portal/).
2. En el menú de **Microsoft Azure**, seleccione **Security Center**. Se abre **Security Center: Información general**. 

   ![Introducción a Security Center][2]

3. En el menú principal de Security Center, seleccione **Introducción**.
4. Seleccione la pestaña **Introducción**.

   ![Introducción][3]

5. Haga clic en **Configurar** en **Agregar nuevos equipos que no son de Azure**. Aparecerá una lista de las áreas de trabajo de Log Analytics. La lista incluye, si procede, el área de trabajo predeterminada que Security Center crea automáticamente si el aprovisionamiento automático está habilitado. Seleccione esta área de trabajo, u otra a la que desee que la máquina virtual de Azure Stack notifique los datos de seguridad.

    ![Agregar un equipo que no es de Azure](./media/quick-onboard-windows-computer/non-azure.png)

   Se abre la hoja **Agente directo** con un vínculo para descargar el agente y las claves del identificador del área de trabajo que se usa para configurar el agente.

   >[!NOTE]
   > NO es preciso descargar al agente manualmente. El agente se instalará como una extensión de máquina virtual en los pasos siguientes.

6. A la derecha de **Id. del área de trabajo**, seleccione el icono Copiar y pegue el identificador en el Bloc de notas.

7. A la derecha de **Clave principal**, seleccione el icono Copiar y pegue la clave en el Bloc de notas.

## <a name="add-the-virtual-machine-extension-to-your-existing-azure-stack-virtual-machines"></a>Agregue la extensión de máquina virtual a las máquinas virtuales existentes de Azure Stack
Ahora debe agregar la extensión de máquina virtual **Azure Monitor, Update and Configuration Management** a las máquinas virtuales que se ejecutan en Azure Stack.

1. En una nueva pestaña del explorador, inicie sesión en el portal de **Azure Stack**.
2. Vaya a la página **Máquinas virtuales** y seleccione la máquina virtual que desea proteger con Security Center. Para obtener información acerca de cómo crear una máquina virtual en Azure Stack, consulte [este tutorial de inicio rápido de máquinas virtuales Windows](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-quick-windows-portal) o [este tutorial de inicio rápido de máquinas virtuales Linux](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-quick-linux-portal).
3. Seleccione **Extensiones**. Se muestra la lista de extensiones de máquina virtual instaladas en esta máquina virtual.
4. Haga clic en la pestaña **Agregar**. Se abre la hoja del menú **Nuevo recurso** y muestra la lista de extensiones de máquina virtual disponibles. 
5. Seleccione la extensión **Azure Monitor, Update and Configuration Management** y haga clic en **Create** (Crear). Se abre la hoja de configuración **Instalar extensión**.

>[!NOTE]
> Si no ve la extensión **Azure Monitor, Update and Configuration Management** en Marketplace, póngase en contacto con su operador de Azure Stack para que se la proporcione.

6. En la hoja de configuración **Instalar extensión**, pegue el **identificador del área de trabajo** y la **clave del área de trabajo (clave principal)** que copió en el Bloc de notas en el procedimiento anterior.
7. Cuando haya terminado de especificar los valores de configuración necesarios, haga clic en **Aceptar**.
8. Una vez que se complete la instalación de la extensión, su estado se mostrará como **Aprovisionamiento realizado correctamente**. La máquina virtual puede tardar hasta una hora en aparecer en el portal de Security Center.

Para más información acerca de cómo instalar y configurar el agente para Windows, consulte [Conexión de equipos Windows](../azure-monitor/platform/agent-windows.md#install-agent-using-setup-wizard).

Para solucionar problemas en Linux del agente, consulte [Solución de problemas relacionados con el agente Azure Log Analytics para Linux](../azure-monitor/platform/agent-linux-troubleshoot.md).

Ahora puede supervisar las máquinas virtuales de Azure y los equipos que no son de Azure en un único lugar. En el portal de Security Center de Azure, en **Compute**, dispone de información general de todas las máquinas virtuales y los equipos, junto con sus recomendaciones. Security Center también muestra todas las detecciones de estos equipos en las alertas de seguridad.

  ![Hoja Proceso][6]

Hay dos tipos de iconos representados en la hoja **Compute**:

![icon1](./media/quick-onboard-windows-computer/security-center-monitoring-icon1.png) Equipos que no son de Azure 

![icon2](./media/quick-onboard-windows-computer/security-center-monitoring-icon2.png) Máquina virtual de Azure (las máquinas virtuales de Azure Stack se mostrarán en este grupo)

## <a name="clean-up-resources"></a>Limpieza de recursos
Cuando deje de ser necesaria, la extensión se puede eliminar de la máquina virtual desde el portal de Azure Stack.

Para eliminar la extensión:

1. Abra el **portal de Azure Stack**.
2. Vaya a la página **Máquinas virtuales** y seleccione la máquina virtual de la que desea quitar la extensión.
3. Seleccione **Extensions** (Extensiones) y, después, la extensión **Microsoft.EnterpriseCloud.Monitoring**.
4. Haga clic en **Uninstall** (Desinstalar) y confirme la opción elegida, para lo que debe hacer clic en **Yes** (Sí).

## <a name="next-steps"></a>Pasos siguientes
En este tutorial de inicio rápido ha aprovisionado la extensión Azure Monitor, Update and Configuration Management en una máquina virtual que se ejecuta en Azure Stack. Para más información acerca del uso de Security Center, siga con el tutorial para configurar una directiva de seguridad y evaluar la seguridad de los recursos.

> [!div class="nextstepaction"]
> [Tutorial: Definición y evaluación de las directivas de seguridad](tutorial-security-policy.md)

<!--Image references-->
[2]: ./media/quick-onboard-windows-computer/overview.png
[3]: ./media/quick-onboard-windows-computer/get-started.png
[4]: ./media/quick-onboard-windows-computer/add-computer.png
[5]: ./media/quick-onboard-windows-computer/log-analytics-mma-setup-laworkspace.png
[6]: ./media/quick-onboard-windows-computer/compute.png
