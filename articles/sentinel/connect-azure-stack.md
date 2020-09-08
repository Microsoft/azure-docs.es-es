---
title: Incorporación de máquinas virtuales de Azure Stack a Azure Sentinel | Microsoft Docs
description: En este artículo se muestra cómo aprovisionar la extensión de máquina virtual de Azure Monitor, Update and Configuration Management en máquinas virtuales de Azure Stack y cómo empezar a supervisarlas con Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/23/2019
ms.author: yelevin
ms.openlocfilehash: 8114a5e6db7b82b846d221471f41dbdf418ddd9d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002001"
---
# <a name="connect-azure-stack-virtual-machines-to-azure-sentinel"></a>Conexión de máquinas virtuales de Azure Stack a Azure Sentinel




Con Azure Sentinel, puede supervisar las máquinas virtuales que se ejecutan en Azure y Azure Stack en un solo lugar. Para incorporar sus máquinas de Azure Stack a Azure Sentinel, primero debe agregar la extensión de máquina virtual a las máquinas virtuales de Azure Stack existentes. 

Después de conectar las máquinas de Azure Stack, elija de una galería de paneles que exponen información basada en los datos. Estos paneles se pueden personalizar fácilmente en función de sus necesidades.



## <a name="add-the-virtual-machine-extension"></a>Adición de la extensión de máquina virtual 

Agregue la extensión de máquina virtual **Azure Monitor, Update and Configuration Management** a las máquinas virtuales que se ejecutan en Azure Stack. 

1. En una nueva pestaña del explorador, inicie sesión en el portal de [Azure Stack](https://docs.microsoft.com/azure-stack/user/azure-stack-use-portal#access-the-portal).
2. Vaya a la página **Máquinas virtuales** y seleccione la máquina virtual que desea proteger con Azure Sentinel. Para obtener información sobre cómo crear una máquina virtual en Azure Stack, vea [Creación de una máquina virtual Windows Server con el portal de Azure Stack](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-quick-windows-portal) o [Creación de una VM de servidor Linux mediante el portal de Azure Stack](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-quick-linux-portal).
3. Seleccione **Extensiones**. Se muestra la lista de extensiones de máquina virtual instaladas en esta máquina virtual.
4. Haga clic en la pestaña **Agregar**. Se abre la hoja del menú **Nuevo recurso** y muestra la lista de extensiones de máquina virtual disponibles. 
5. Seleccione la extensión **Azure Monitor, Update and Configuration Management** y haga clic en **Create** (Crear). Se abre la ventana de configuración **Instalar extensión**.

   ![Configuración de Azure Monitor, Update, and Configuration Management](./media/connect-azure-stack/azure-monitor-extension-fix.png)  

   >[!NOTE]
   > Si no ve la extensión **Azure Monitor, Update and Configuration Management** en Marketplace, póngase en contacto con su operador de Azure Stack para que se la proporcione.

6. En el menú de Azure Sentinel, seleccione **Configuración del área de trabajo** y, después, **Avanzado** y copie el **Id. del área de trabajo** y la **Clave de área de trabajo (clave principal)** . 
1. En la ventana **Instalar extensión** de Azure Stack, péguelos en los campos indicados y haga clic en **Aceptar**.
1. Una vez que se complete la instalación de la extensión, su estado se mostrará como **Aprovisionamiento realizado correctamente**. La máquina virtual puede tardar hasta una hora en aparecer en el portal de Azure Sentinel.

Para más información sobre cómo instalar y configurar el agente para Windows, vea [Conexión de equipos Windows](../azure-monitor/platform/agent-windows.md#install-agent-using-setup-wizard).

Para solucionar problemas en Linux del agente, consulte [Solución de problemas relacionados con el agente Azure Log Analytics para Linux](../azure-monitor/platform/agent-linux-troubleshoot.md).

En el portal de Azure Sentinel de Azure, en **Virtual Machines**, dispone de información general de todas las máquinas virtuales y los equipos, junto con sus estados. 

## <a name="clean-up-resources"></a>Limpieza de recursos
Cuando deje de ser necesaria, la extensión se puede eliminar de la máquina virtual desde el portal de Azure Stack.

Para eliminar la extensión:

1. Abra el **portal de Azure Stack**.
2. Vaya a la página **Máquinas virtuales** y seleccione la máquina virtual de la que desea quitar la extensión.
3. Seleccione **Extensions** (Extensiones) y, después, la extensión **Microsoft.EnterpriseCloud.Monitoring**.
4. Haga clic en **Uninstall** (Desinstalar) y confirme la opción elegida.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
- Transmita datos desde [dispositivos con formatos de error comunes](connect-common-event-format.md) a Azure Sentinel.
