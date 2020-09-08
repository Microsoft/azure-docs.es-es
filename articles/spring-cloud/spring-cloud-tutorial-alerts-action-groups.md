---
title: 'Tutorial: Supervisión de recursos de Azure Spring Cloud mediante alertas y grupos de acciones | Microsoft Docs'
description: Aprenda a usar las alertas de Spring Cloud.
author: MikeDodaro
ms.author: barbkess
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 12/29/2019
ms.custom: devx-track-java
ms.openlocfilehash: 856bace0c67c5afedd6305158326af46037e7f21
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299234"
---
# <a name="tutorial-how-to-monitor-spring-cloud-resources-using-alerts-and-action-groups"></a>Tutorial: Supervisión de recursos de Spring Cloud mediante alertas y grupos de acciones

Las alertas de Azure Spring Cloud permiten supervisar los recursos en función de condiciones tales como el almacenamiento disponible, la tasa de solicitudes o el uso de datos. Una alerta envía una notificación cuando las condiciones cumplen las especificaciones definidas.

Hay dos pasos para configurar una canalización de alertas: 
1. Configure un grupo de acciones con las acciones que se llevarán a cabo cuando se desencadene una alerta; por ejemplo, correo electrónico, SMS, runbook o webhook. Los grupos de acciones se pueden volver a usar con distintas alertas.
2. Configure las reglas de alerta. Las reglas asocian los patrones de las métricas con los grupos de acciones según el recurso de destino, la métrica, la condición, la agregación de tiempo, etc.

## <a name="prerequisites"></a>Requisitos previos

Además de los requisitos de Azure Spring, los procedimientos de este tutorial funcionan con una instancia implementada de Azure Spring Cloud.  Siga un [inicio rápido](spring-cloud-quickstart.md) para comenzar.

Los procedimientos siguientes inicializan **Grupo de acciones** y **Alertas** a partir de la opción **Alertas** situada en el panel de navegación izquierdo de una instancia de Spring Cloud. (El procedimiento también se puede iniciar desde la página **Información general de Monitor**, en Azure Portal). 

Vaya desde un grupo de recursos a la instancia de Spring Cloud. Seleccione **Alertas** en el panel izquierdo y, a continuación, seleccione **Administrar acciones**:

![Captura de pantalla del grupo de recursos del portal](media/alerts-action-groups/action-1-a.png)

## <a name="set-up-action-group"></a>Configuración de un grupo de acciones

Para comenzar el procedimiento para inicializar un nuevo **Grupo de acciones**, seleccione **+ Agregar grupo de acciones**.

![Captura de pantalla del portal con la opción Agregar grupo de acciones](media/alerts-action-groups/action-1.png)

En la página **Agregar grupo de acciones**:

 1. Especifique los valores de **Nombre del grupo de acciones** y **Nombre corto**.

 1. Especifique los valores de **Suscripción** y **Grupo de recursos**.

 1. Especifique **Nombre de acción**.

 1. Seleccione **Tipo de acción**.  Se abrirá otro panel a la derecha para definir la acción que se realizará en la activación.

 1. Defina la acción con las opciones del panel derecho.  En este caso se usa la notificación por correo electrónico.

 1. Haga clic en **Aceptar** en el panel de acciones de la derecha.

 1. Haga clic en **Aceptar** en el cuadro de diálogo **Agregar grupo de acciones**. 

  ![Captura de pantalla del portal con la definición de la acción](media/alerts-action-groups/action-2.png)

## <a name="set-up-alert"></a>Configuración de una alerta 

En los pasos anteriores se creó un **Grupo de acciones** que usa notificaciones por correo electrónico. También puede usar notificaciones por teléfono, webhooks, Azure Functions, etc.  

Para configurar una **Alerta**, vuelva a la página **Alertas** y haga clic en **Administrar reglas de alerta**.

  ![Captura de pantalla del portal con la definición de la alerta](media/alerts-action-groups/alerts-2.png)

1. Seleccione el valor de **Recurso** para la alerta.

1. Haga clic en **Nueva regla de alertas**.

   ![Captura de pantalla del portal con la nueva regla de alerta](media/alerts-action-groups/alerts-3.png)

1. En la página **Crear regla**, especifique el **recurso**.

1. La opción **Condición** ofrece muchas opciones para supervisar los recursos de **Spring Cloud**.  Haga clic en **Agregar** para abrir el panel **Configurar lógica de señal**.

1. Seleccione una condición. En este ejemplo se usa **System CPU Usage Percentage** (Porcentaje de uso de CPU del sistema).

   ![Captura de pantalla del portal con la nueva regla de alerta 2](media/alerts-action-groups/alerts-3-1.png)

1. Desplácese hacia abajo hasta el panel **Configurar lógica de señal** para establecer el **valor de umbral** que se va a supervisar.

   ![Captura de pantalla del portal con la nueva regla de alerta 3](media/alerts-action-groups/alerts-3-2.png)

1. Haga clic en **Done**(Listo).

Para más información sobre las condiciones disponibles para supervisar, consulte [Métricas en Azure Spring Cloud](spring-cloud-concept-metrics.md#user-metrics-options).

 En **Acciones**, haga clic en **Seleccionar grupo de acciones**. En el panel **Acciones**, seleccione el **Grupo de acciones** definido previamente.

   ![Captura de pantalla del portal con la nueva regla de alerta 4](media/alerts-action-groups/alerts-3-3.png) 

1. En **Detalles de la alerta**, asigne un nombre a la regla de alerta.

1. Establezca el valor de **Gravedad**.

1. Haga clic en **Crear regla de alertas**.

   ![Captura de pantalla del portal con la nueva regla de alerta 5](media/alerts-action-groups/alerts-3-4.png)

Compruebe que la nueva regla de alerta esté habilitada.

   ![Captura de pantalla del portal con la nueva regla de alerta 6](media/alerts-action-groups/alerts-4.png)

También se puede crear una regla mediante la página **Métricas**:

   ![Captura de pantalla del portal con la nueva regla de alerta 7](media/alerts-action-groups/alerts-5.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a configurar alertas y grupos de acciones para una aplicación de Azure Spring Cloud. Para más información sobre los grupos de acciones, consulte:

> [!div class="nextstepaction"]
> [Creación y administración de grupos de acciones en Azure Portal](https://docs.microsoft.com/azure/azure-monitor/platform/action-groups)

> [!div class="nextstepaction"]
> [Comportamiento de las alertas por SMS en los grupos de acciones](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-sms-behavior)
