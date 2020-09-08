---
title: Adición de métricas extendidas a máquinas virtuales de Azure
description: Este artículo le ayuda a habilitar y configurar métricas de diagnóstico extendidas en las máquinas virtuales de Azure.
author: bandersmsft
ms.reviewer: vitavor
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: 89084f0631b52631708db68a11595cb24d1b9fee
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690126"
---
# <a name="add-extended-metrics-for-azure-virtual-machines"></a>Adición de métricas extendidas a máquinas virtuales de Azure

Cloudyn usa los datos de métrica de Azure de las máquinas virtuales de Azure para mostrar información detallada acerca de sus recursos. Asimismo, Cloudyn usa los datos de métrica, también denominados contadores de rendimiento, para generar informes. Sin embargo, Cloudyn no recopila automáticamente todos los datos de métrica de Azure de las máquinas virtuales invitadas (para ello, es preciso habilitar la recopilación de métricas). Este artículo le ayuda a habilitar y configurar métricas de diagnóstico adicionales en las máquinas virtuales de Azure.

Tras habilitar la recopilación de métricas puede:

- Saber si las máquinas virtuales están llegando a sus límites de CPU, disco y memoria.
- Detectar tendencias de uso y anomalías.
- Controlar los costos mediante el ajuste de tamaño en función del uso.
- Obtener recomendaciones de Cloudyn para optimizar de forma rentable el tamaño.

Por ejemplo, puede supervisar los porcentajes de CPU y de memoria de las máquinas virtuales de Azure. Las métricas de las VM de Azure corresponden a _Porcentaje de CPU_ y _\Memoria\% de bytes confirmados en uso_.

> [!NOTE]
> La recopilación de datos métricos extendidos solo se admite con la supervisión a nivel de invitado de Azure. Cloudyn no es compatible con el [agente de Log Analytics](../../azure-monitor/platform/agents-overview.md).

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="determine-whether-extended-metrics-are-enabled"></a>Determinar si las métricas extendidas están habilitadas

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. En **Máquinas virtuales**, seleccione una máquina virtual y en **Supervisión**, seleccione **Métricas**. Se muestra una lista de las métricas disponibles.
3. Seleccione algunas métricas y un gráfico mostrará los datos de las mismas.  
    ![Métrica de ejemplo: porcentaje de CPU de host](./media/azure-vm-extended-metrics/metric01.png)

En el ejemplo anterior, hay disponible un conjunto limitado de métricas estándar de los hosts, pero no métricas de memoria. Las métricas de memoria forman parte de las métricas extendidas. En este caso, no están habilitadas las métricas extendidas de la máquina virtual. Para habilitar las métricas extendidas es preciso dar algunos pasos más. La siguiente información le guía por el proceso de su habilitación.

## <a name="enable-extended-metrics-in-the-azure-portal"></a>Habilitación de métricas extendidas en Azure Portal

Las métricas estándar son las métricas del equipo host. La métrica _Porcentaje de CPU_  es un ejemplo. También hay métricas básicas para las máquinas virtuales invitadas y también se denominan métricas extendidas. Entre los ejemplos de métricas se incluyen _\Memoria\% de bytes confirmados en uso_ y _\Memoria\bytes disponibles_.

La habilitación de las métricas extendidas es un proceso sencillo. En cada máquina virtual, habilite la supervisión a nivel de invitado. Cuando se habilita la supervisión a nivel de invitado, el agente de Azure Diagnostics se instala en la máquina virtual. De forma predeterminada, se agrega un conjunto básico de métricas extendidas. El siguiente proceso es el mismo para las máquinas virtuales clásicas y regulares, así como para las máquinas virtuales Windows y Linux.

Tenga en cuenta que tanto la supervisión a nivel de invitado de Azure como la de Linux requieren una cuenta de almacenamiento. Si no elige una cuenta de almacenamiento existente cuando habilite la supervisión de nivel de invitado, se creará una.

### <a name="enable-guest-level-monitoring-on-existing-vms"></a>Habilitar supervisión a nivel de invitado en las máquinas virtuales existentes

1. En **Máquinas virtuales**, vea la lista de las máquinas virtuales y seleccione una de ellas.
2. En **Supervisión**, seleccione **Configuración de diagnóstico**.
3. En la página Configuración de diagnóstico, haga clic en **Habilitar supervisión a nivel de invitado**.  
    ![Habilitación de la supervisión a nivel de invitado en la página de información general](./media/azure-vm-extended-metrics/enable-guest-monitoring.png)
4. Pocos minutos después, el agente de Azure Diagnostics está instalado en la máquina virtual. Se agrega un conjunto básico de métricas. Actualice la página. Los contadores de rendimiento agregados aparecen en la pestaña de información general.
5. En Supervisión, seleccione **Métricas**.
6. En **Espacio de nombres de métrica** del gráfico de métricas, seleccione **Invitado (clásico)** .
7. En la lista Métrica podrá ver todos los contadores de rendimiento disponibles para la máquina virtual invitada.  
    ![lista de métricas extendidas de ejemplo](./media/azure-vm-extended-metrics/extended-metrics.png)

### <a name="enable-guest-level-monitoring-on-new-vms"></a>Habilitar la supervisión a nivel de invitado en las máquinas virtuales nuevas

Cuando cree nuevas máquinas virtuales, en la pestaña Administración, seleccione **Activados** para **Diagnósticos del SO invitado**.

![establecimiento de Diagnósticos del SO invitado en Activado](./media/azure-vm-extended-metrics/new-enable-diag.png)

Para obtener más información acerca de cómo habilitar las métricas extendidas para las máquinas virtuales de Azure, consulte la [Información y uso del agente de Linux de Azure](../../virtual-machines/extensions/agent-linux.md) y la [Información general del agente de máquina virtual de Azure](../../virtual-machines/extensions/agent-windows.md).

## <a name="resource-manager-credentials"></a>Credenciales de Resource Manager

Después de habilitar las métricas extendidas, asegúrese de que Cloudyn tiene acceso a sus [credenciales de Resource Manager](../../cost-management/activate-subs-accounts.md). Dichas credenciales son necesarias para que Cloudyn recopile y muestre los datos de rendimiento de las máquinas virtuales. También se utilizan para crear recomendaciones de optimización de costo. Cloudyn necesita al menos tres días de datos de rendimiento de una instancia para determinar si es candidata a una recomendación de reducción de tamaño.

## <a name="enable-vm-metrics-with-a-script"></a>Habilitar las métricas de una máquina virtual con un script

Las métricas de una máquina virtual se pueden habilitar con scripts de Azure PowerShell. Cuando haya muchas máquinas virtuales en las que desee habilitar las métricas, puede usar un script para automatizar el proceso. En GitHub, en [Azure Enable Diagnostics](https://github.com/Cloudyn/azure-enable-diagnostics), puede encontrar scripts de ejemplo.

## <a name="view-azure-performance-metrics"></a>Visualización de métricas de rendimiento de Azure

Para ver métricas del rendimiento de las instancias de Azure en el portal de Cloudyn, vaya a **Assets** (Recursos) > **Compute** > **Instance Explorer** (Explorador de instancias). En la lista de instancias de máquina virtual, expanda una de ellas y, después, expanda un recurso para ver sus detalles.

![información de ejemplo que se muestra en el Explorador de instancias](./media/azure-vm-extended-metrics/instance-explorer.png)

## <a name="next-steps"></a>Pasos siguientes

- Si no ha habilitado el acceso de la API de Azure Resource Manager en sus cuentas, vaya a la sección [Activate Azure subscriptions and accounts with Azure Cost Management](../../cost-management/activate-subs-accounts.md) (Activación de suscripciones y cuentas de Azure con Azure Cost Management).
