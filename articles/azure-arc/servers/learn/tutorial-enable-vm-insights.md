---
title: 'Tutorial: Supervisión de una máquina híbrida con Azure Monitor para VM'
description: Aprenda a recopilar y analizar los datos de una máquina híbrida en Azure Monitor.
ms.topic: tutorial
ms.date: 08/12/2020
ms.openlocfilehash: 76df7d403fdce6fc3ac77c0b24849aedffb57ce0
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213161"
---
# <a name="tutorial-monitor-a-hybrid-machine-with-azure-monitor-for-vms"></a>Tutorial: Supervisión de una máquina híbrida con Azure Monitor para VM

[Azure Monitor](../overview.md) puede recopilar datos directamente de las máquinas virtuales híbridas en un área de trabajo de Log Analytics para lograr una correlación y un análisis detallados. Normalmente, esto implicaría la instalación del [agente de Log Analytics](../../../azure-monitor/platform/agents-overview.md#log-analytics-agent) en la máquina mediante un script, un método manual o automatizado, según los estándares de administración de la configuración. Los servidores habilitados para Arc (versión preliminar) han incorporado recientemente compatibilidad para instalar las [extensiones de máquinas virtuales](../manage-vm-extensions.md) del agente de Log Analytics y de Dependency Agent para Windows y Linux, lo que permite a Azure Monitor recopilar datos de máquinas virtuales que no son de Azure.

En este tutorial se muestra cómo configurar y recopilar datos de máquinas virtuales Linux o Windows mediante la habilitación de Azure Monitor para VM siguiendo un conjunto simplificado de pasos, lo que optimiza la experiencia y exige un tiempo menor.  

## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

* La funcionalidad de la extensión de VM solo está disponible en la lista de [regiones admitidas](../overview.md#supported-regions).

* Consulte [Sistemas operativos compatibles](../../../azure-monitor/insights/vminsights-enable-overview.md#supported-operating-systems) para asegurarse de que el sistema operativo de las máquinas virtuales que está habilitando sea compatible con Azure Monitor para VM.

* Revise los requisitos de firewall para el agente de Log Analytics, que se proporcionan en el artículo sobre [información general del agente de Log Analytics](../../../azure-monitor/platform/log-analytics-agent.md#network-requirements). La extensión Dependency Agent para Service Map de Azure Monitor para VM no transmite ningún dato y no requiere ningún cambio en firewalls o puertos.

## <a name="sign-in-to-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="enable-azure-monitor-for-vms"></a>Habilitar Azure Monitor para VM

1. Inicie el servicio Azure Arc en Azure Portal. Para ello, haga clic en **Todos los servicios** y, a continuación, busque y seleccione **Máquinas: Azure Arc**.

    :::image type="content" source="./media/quick-enable-hybrid-vm/search-machines.png" alt-text="Búsqueda de servidores habilitados para Arc en todos los servicios" border="false":::

1. En la página **Máquinas: Azure Arc**, seleccione la máquina conectada que creó en el artículo de [inicio rápido](quick-enable-hybrid-vm.md).

1. En el panel izquierdo, en la sección **Supervisión**, seleccione **Insights** y, luego, **Habilitar**.

    :::image type="content" source="./media/tutorial-enable-vm-insights/insights-option.png" alt-text="Selección de la opción Insights en el menú izquierdo" border="false":::

1. En la página de Azure Monitor **Incorporación a Insights**, se le pedirá que cree un área de trabajo. En este tutorial, no se recomienda que seleccione un área de trabajo de Log Analytics existente si tiene una. Seleccione el valor predeterminado que es un área de trabajo con un nombre único en la misma región que la máquina conectada registrada. Esta área de trabajo se crea y se configura automáticamente.

    :::image type="content" source="./media/tutorial-enable-vm-insights/enable-vm-insights.png" alt-text="Página Habilitar Azure Monitor para VM" border="false":::

1. Recibirá mensajes de estado mientras se lleva a cabo la configuración. Este proceso tarda unos minutos mientras se instalan las extensiones en la máquina conectada.

    :::image type="content" source="./media/tutorial-enable-vm-insights/onboard-vminsights-vm-portal-status.png" alt-text="Mensaje de estado de progreso de Habilitar Azure Monitor para VM" border="false":::

    Cuando haya finalizado, recibirá un mensaje que indica que la máquina se ha incorporado correctamente y que la información se ha implementado sin problemas.

## <a name="view-data-collected"></a>Ver datos recopilados

Una vez completada la implementación y la configuración, seleccione **Insights**y, a continuación, seleccione la pestaña **Rendimiento**. En la pestaña Rendimiento, se muestra un grupo seleccionado de contadores de rendimiento recopilados desde el sistema operativo invitado de la máquina virtual. Desplácese hacia abajo para ver más contadores, y mueva el mouse sobre un gráfico para ver el promedio y los percentiles que se han tomado desde el momento en que se instaló la extensión de máquina virtual de Log Analytics en la máquina.

:::image type="content" source="./media/tutorial-enable-vm-insights/insights-performance-charts.png" alt-text="Gráficos de rendimiento de Azure Monitor para VM para la máquina seleccionada" border="false":::

Seleccione **Asignar** para abrir la característica de asignaciones que muestra los procesos que se ejecutan en la máquina virtual y sus dependencias. Seleccione **Propiedades** para abrir el panel de propiedades si aún no está abierto.

:::image type="content" source="./media/tutorial-enable-vm-insights/insights-map.png" alt-text="Asignación de Azure Monitor para VM para la máquina seleccionada" border="false":::

Expanda los procesos de la máquina virtual. Seleccione uno de los procesos para ver sus detalles y resaltar sus dependencias.

Vuelva a seleccionar la máquina virtual y, luego, seleccione **Registrar eventos**. Verá una lista de tablas que se almacenan en el área de trabajo de Log Analytics de la máquina virtual. Esta lista será diferente en función de si usa una máquina virtual Windows o Linux. Seleccione la tabla **Evento**. La tabla **Evento** incluye todos los eventos del registro de eventos de Windows. Se abre Log Analytics con una consulta simple para recuperar las entradas del registro de eventos recopiladas.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de Azure Monitor, consulte el siguiente artículo:

> [!div class="nextstepaction"]
> [Introducción a Azure Monitor](../../../azure-monitor/overview.md)