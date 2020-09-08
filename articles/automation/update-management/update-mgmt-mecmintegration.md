---
title: Integración de Update Management de Azure Automation con Microsoft Endpoint Configuration Manager
description: Este artículo se indica cómo configurar Microsoft Endpoint Configuration Manager con Update Management para implementar actualizaciones de software en clientes de administrador.
services: automation
ms.subservice: update-management
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: 09c8ef818428157c7de8c3a87bcce8a7b80e62ea
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88245917"
---
# <a name="integrate-update-management-with-microsoft-endpoint-configuration-manager"></a>Integración de Update Management con Microsoft Endpoint Configuration Manager

Los clientes que han invertido en Microsoft Endpoint Configuration Manager para administrar equipos, servidores y dispositivos móviles también dependen de su solidez y nivel de madurez para administrar actualizaciones de software como parte de su ciclo de Administración de actualizaciones de software (SUM).

Puede crear informes sobre los servidores de Windows administrados y actualizarlos si crea y preconfigura implementaciones de actualizaciones de software en Microsoft Endpoint Configuration Manager, y obtener el estado detallado de las implementaciones de actualizaciones finalizadas mediante [Update Management](update-mgmt-overview.md). Si usa Microsoft Endpoint Configuration Manager para los informes de cumplimiento de las actualizaciones, pero no para administrar las implementaciones de actualizaciones con los servidores de Windows, puede seguir informando a Microsoft Endpoint Configuration Manager mientras se administran las actualizaciones de seguridad con Update Management en Azure Automation.

>[!NOTE]
>Aunque Update Management admite la evaluación y la revisión de las actualizaciones de Windows Server 2008 R2, no admite clientes administrados por Microsoft Endpoint Configuration Manager que ejecuten este sistema operativo.

## <a name="prerequisites"></a>Requisitos previos

* Debe haber agregado [Update Management de Azure Automation](update-mgmt-overview.md) a la cuenta de Automation.
* Los servidores de Windows que actualmente administra el entorno de Microsoft Endpoint Configuration Manager deben informar al área de trabajo de Log Analytics que también tiene Update Management habilitado.
* Esta característica está habilitada en la rama actual de Microsoft Endpoint Configuration Manager, versión 1606 y posteriores. Para integrar el sitio de administración central de Microsoft Endpoint Configuration Manager o un sitio primario independiente con registros de Azure Monitor e importar colecciones, revise [Conexión de Configuration Manager con Azure Monitor](../../azure-monitor/platform/collect-sccm.md).  
* Los agentes de Windows deben estar configurados para comunicarse con un servidor Windows Server Update Services (WSUS) o tener acceso a Microsoft Update si no reciben actualizaciones de seguridad de Microsoft Endpoint Configuration Manager.

La forma de administrar los clientes hospedados en IaaS de Azure con el entorno existente de Microsoft Endpoint Configuration Manager depende, principalmente, de la conexión entre los centros de datos de Azure y la infraestructura. Esta conexión afecta a los cambios de diseño que pueda necesitar realizar en la infraestructura de Microsoft Endpoint Configuration Manager y a los costos derivados de admitir esos cambios. Para comprender qué consideraciones de planeación debe evaluar antes de continuar, revise [Configuration Manager en Azure - Preguntas más frecuentes](/configmgr/core/understand/configuration-manager-on-azure#networking).

## <a name="manage-software-updates-from-microsoft-endpoint-configuration-manager"></a>Administración de actualizaciones de software desde Microsoft Endpoint Configuration Manager

Siga estos pasos si va a continuar con la administración de implementaciones de actualizaciones desde Microsoft Endpoint Configuration Manager. Azure Automation se conecta a Microsoft Endpoint Configuration Manager para aplicar actualizaciones a los equipos cliente conectados al área de trabajo de Log Analytics. El contenido de la actualización está disponible en la caché del equipo cliente como si la implementación se administrara mediante Microsoft Endpoint Configuration Manager.

1. Cree una implementación de actualizaciones de software desde el sitio de nivel superior de la jerarquía de Microsoft Endpoint Configuration Manager mediante el proceso descrito en [Implementación de actualizaciones de software](/configmgr/sum/deploy-use/deploy-software-updates). La única opción que se debe configurar de forma distinta de una implementación estándar es la opción **No instalar actualizaciones de software** para controlar el comportamiento de descarga del paquete de implementación. Este comportamiento se administra en Update Management mediante la creación de una implementación de actualizaciones programada en el paso siguiente.

2. En Azure Automation, seleccione **Update Management**. Siga los pasos descritos en [Creación de una implementación de actualizaciones](update-mgmt-deploy-updates.md#schedule-an-update-deployment) para crear una implementación y seleccione **Grupos importados** en la lista desplegable **Tipo** para seleccionar la colección adecuada de Microsoft Endpoint Configuration Manager. Tenga en cuenta los siguientes puntos importantes:

    a. Si se define una ventana de mantenimiento en la colección de dispositivos de Microsoft Endpoint Configuration Manager, los miembros de la colección respetan sus condiciones, en lugar de la configuración **Duración** definida en la implementación programada.

    b. Los miembros de la colección de destino deben tener una conexión a Internet (ya sea directamente, a través de un servidor proxy o a través de la puerta de enlace de Log Analytics).

Después de completar la implementación de actualizaciones mediante Azure Automation, los equipos de destino que son miembros del grupo de equipos seleccionado instalarán las actualizaciones a la hora programada desde la memoria caché del cliente local. También puede [ver el estado de implementación de actualizaciones](update-mgmt-deploy-updates.md#check-deployment-status) para supervisar los resultados de la implementación.

## <a name="manage-software-updates-from-azure-automation"></a>Administrar actualizaciones de software desde Azure Automation

Con el fin de administrar actualizaciones para máquinas virtuales de Windows Server que son clientes de Microsoft Endpoint Configuration Manager, debe configurar la directiva de cliente para deshabilitar la característica Administración de actualizaciones de software para todos los clientes administrados por Update Management. De forma predeterminada, la configuración de cliente está dirigida a todos los dispositivos de la jerarquía. Para más información sobre esta configuración de directiva y cómo configurarla, consulte [Cómo establecer la configuración de cliente en Configuration Manager](/configmgr/core/clients/deploy/configure-client-settings).

Después de aplicar este cambio de configuración, siga los pasos descritos en [Creación de una implementación de actualizaciones](update-mgmt-deploy-updates.md#schedule-an-update-deployment) para crear una implementación y seleccione **Grupos importados** en la lista desplegable **Tipo** para seleccionar la colección adecuada de Microsoft Endpoint Configuration Manager de Windows.

## <a name="next-steps"></a>Pasos siguientes

Para configurar una programación de integración, vea [Programación de una implementación de actualización](update-mgmt-deploy-updates.md#schedule-an-update-deployment).
