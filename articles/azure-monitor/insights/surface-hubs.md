---
title: Supervisión de Surface Hub con Azure Monitor | Microsoft Docs
description: Use la solución Surface Hub para realizar un seguimiento del estado de sus dispositivos con esta solución y comprender cómo se están utilizando.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 01/16/2018
ms.openlocfilehash: 7e3b0f92770b48ef5163846e67940efe80fb669a
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90085355"
---
# <a name="monitor-surface-hubs-with-azure-monitor-to-track-their-health"></a>Supervisión de Surface Hub con Azure Monitor para realizar un seguimiento de su estado

![Símbolo de Surface Hub](./media/surface-hubs/surface-hub-symbol.png)

En este artículo se describe cómo se puede usar la solución Surface Hub en Azure Monitor para supervisar los dispositivos Microsoft Surface Hub. La solución le ayuda a realizar un seguimiento del estado de sus dispositivos con esta solución y comprender cómo se están utilizando.

Cada dispositivo Surface Hub tiene el agente de supervisión de Microsoft instalado. A través del agente se pueden enviar datos desde Surface Hub a un área de trabajo de Log Analytics en Azure Monitor. Los archivos de registro se leen desde los dispositivos Surface Hub y se envían a Azure Monitor. Los problemas (por ejemplo, que los servidores estén sin conexión, el calendario no se sincronice o la cuenta del dispositivo no pueda iniciar sesión en Skype) se muestran en Log Analytics en el panel de Azure Monitor. Mediante el uso de los datos en el panel, puede identificar los dispositivos que no se están ejecutando o que tienen otros problemas, así como, posiblemente aplicar correcciones para los problemas detectados.

## <a name="install-and-configure-the-solution"></a>Instalación y configuración de la solución
Utilice la siguiente información para instalar y configurar la solución. Con el fin de administrar los dispositivos Surface Hub en Azure Monitor, necesitará lo siguiente:

* Un nivel de [suscripción a Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/) que admitirá el número de dispositivos que desea supervisar. Los precios de Log Analytics varían en función de cuántos dispositivos inscritos haya y de la cantidad de datos que se procesen. Deberá tener esto en cuenta al planear la implementación de Surface Hub.

A continuación, agregará un área de trabajo de Log Analytics existente o creará una nueva. Se pueden encontrar instrucciones detalladas sobre el uso de cualquiera de los métodos en [Creación de un área de trabajo de Log Analytics en Azure Portal](../learn/quick-create-workspace.md). Una vez que se configure el área de trabajo de Log Analytics, hay dos maneras de inscribir los dispositivos Surface Hub:

* Automáticamente mediante Intune
* Manualmente a través de la aplicación **Configuración** del dispositivo Surface Hub.

## <a name="set-up-monitoring"></a>Configuración de la supervisión
Puede supervisar el estado y la actividad de su dispositivo Surface Hub mediante Azure Monitor. Puede inscribir el dispositivo Surface Hub en Surface Hub mediante Intune o de manera local con **Configuración** de Surface Hub.

## <a name="connect-surface-hubs-to-azure-monitor-through-intune"></a>Conexión de Surface Hub a Azure Monitor mediante Intune
Necesitará el identificador de área de trabajo y la clave de área de trabajo del área de trabajo de Log Analytics que va a administrar sus dispositivos Surface Hub. Puede obtenerlos en la configuración del área de trabajo en Azure Portal.

Intune es un producto de Microsoft que permite administrar de forma centralizada la configuración del área de trabajo de Log Analytics que se aplica a uno o varios de los dispositivos. Siga estos pasos para configurar los dispositivos mediante Intune:

1. Inicie sesión en Intune.
2. Vaya a **Configuración** > **Orígenes conectados**.
3. Cree o edite una directiva basada en la plantilla de Surface Hub.
4. Vaya a la sección Azure Operational Insights de la directiva y agregue el *identificador de área de trabajo* y la *clave de área de trabajo* de Log Analytics a la directiva.
5. Guarde la directiva.
6. Asocia la directiva con el grupo de dispositivos adecuado.

   ![Directiva de Intune](./media/surface-hubs/intune.png)

Después, Intune sincroniza la configuración de Log Analytics con los dispositivos en el grupo de destino, con lo que se inscriben en el área de trabajo de Log Analytics.

## <a name="connect-surface-hubs-to-azure-monitor-using-the-settings-app"></a>Conexión de los dispositivos Surface Hub a Azure Monitor a través de la aplicación Configuración
Necesitará el identificador de área de trabajo y la clave de área de trabajo del área de trabajo de Log Analytics que va a administrar sus dispositivos Surface Hub. Puede obtenerlos en la configuración del área de trabajo de Log Analytics en Azure Portal.

Si no usa Intune para administrar su entorno, puede inscribir dispositivos manualmente con **Configuración** en cada dispositivo Surface Hub:

1. En Surface Hub, abra **Configuración**.
2. Escriba las credenciales de administrador de dispositivos cuando se le solicite.
3. Haga clic en **Este dispositivo** y, en **Supervisión**, haga clic en **Configuración de Log Analytics**.
4. Seleccione **Habilitar supervisión**.
5. En el cuadro de diálogo de configuración de Log Analytics, escriba el **identificador de área de trabajo** y la **clave de área de trabajo** de Log Analytics.  
   ![Captura de pantalla que muestra la configuración del conjunto de aplicaciones de Microsoft Operations Manager con la opción Habilitar supervisión seleccionada y los cuadros de texto Id. de área de trabajo y Clave de área de trabajo.](./media/surface-hubs/settings.png)
6. Haga clic en **Aceptar** para completar la configuración.

Aparecerá una confirmación que indica si la configuración se aplicó correctamente en el dispositivo. Si es así, aparecerá un mensaje que indica que el agente se ha conectado correctamente a Azure Monitor. El dispositivo comienza a enviar datos a Azure Monitor, donde puede verlos y usarlos.

## <a name="monitor-surface-hubs"></a>Supervisión de dispositivos Surface Hub
La supervisión de los dispositivos Surface Hub con Azure Monitor es muy similar a la de todos los demás dispositivos inscritos.

[!INCLUDE [azure-monitor-solutions-overview-page](../../../includes/azure-monitor-solutions-overview-page.md)]

Cuando hace clic en el icono de Surface Hub, aparece el estado del dispositivo.

   ![Panel de Surface Hub](./media/surface-hubs/surface-hub-dashboard.png)

Puede crear [alertas](../platform/alerts-overview.md) en función de las búsquedas de registros existentes o personalizadas. Con los datos que recopile Azure Monitor de los dispositivos Surface Hub, puede buscar problemas y generar alertas sobre las condiciones que defina para sus dispositivos.

## <a name="next-steps"></a>Pasos siguientes
* Use las [consultas de registros de Azure Monitor](../log-query/log-query-overview.md) para ver datos detallados de Surface Hub.
* Cree [alertas](../platform/alerts-overview.md) para recibir una notificación cuando se produzcan problemas con Surface Hub.
