---
title: Creación de un área de trabajo de Log Analytics en Azure Portal | Microsoft Docs
description: Aprenda cómo crear un área de trabajo de Log Analytics para habilitar soluciones de administración y recopilación de datos en sus entornos tanto locales como en la nube en Azure Portal.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/26/2020
ms.openlocfilehash: f86cc5e3edc659a233066fcf2544ed4a5019d0f9
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324291"
---
# <a name="create-a-log-analytics-workspace-in-the-azure-portal"></a>Creación de un área de trabajo de Log Analytics en Azure Portal
Use el menú **Áreas de trabajo de Log Analytics** para crear un área de trabajo de Log Analytics con Azure Portal. Un área de trabajo de Log Analytics es un entorno único de datos de registro de Azure Monitor. Cada área de trabajo tiene su propio repositorio de datos y configuración, y las soluciones y orígenes de datos están configurados para almacenar sus datos en una determinada área de trabajo. Necesitará un área de trabajo de Log Analytics si tiene intención de recopilar datos de los orígenes siguientes:

* Recursos de Azure de la suscripción
* Equipos locales supervisados por System Center Operations Manager
* Colecciones de dispositivos de Configuration Manager 
* Datos de diagnóstico o de registro de Azure Storage

Para otros orígenes, como las máquinas virtuales de Azure y la máquinas virtuales Windows o Linux del entorno, consulte los temas siguientes:

*  [Recopilación de datos de máquinas virtuales de Azure](./quick-collect-azurevm.md) 
*  [Recopilación de datos de un equipo Linux híbrido](./quick-collect-linux-computer.md)
*  [Recopilación de datos de un equipo Windows híbrido](quick-collect-windows-computer.md)

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-azure-portal"></a>Inicio de sesión en Azure Portal
Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com). 

## <a name="create-a-workspace"></a>Crear un área de trabajo
1. En Azure Portal, haga clic en **Todos los servicios**. En la lista de recursos, escriba **Log Analytics**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Áreas de trabajo de Log Analytics**.

    ![Azure Portal](media/quick-create-workspace/azure-portal-01.png)
  
2. Haga clic en **Agregar** y, después, seleccione las opciones de los siguientes elementos:

   * Proporcione el nombre de la nueva **área de trabajo de Log Analytics** como, por ejemplo, *DefaultLAWorkspace*. Este nombre debe ser único globalmente en todas las suscripciones de Azure Monitor.
   * Seleccione una **suscripción** a la que vincularlo en la lista desplegable si la opción predeterminada seleccionada no es adecuada.
   * Como **Grupo de recursos** puede usar un grupo de recursos existente que ya esté configurado, o crear uno nuevo.  
   * Seleccione una **ubicación** disponible.  Para obtener información, consulte [las regiones en las que Log Analytics está disponible](https://azure.microsoft.com/regions/services/) y busque Azure Monitor desde el campo **Buscar un producto**.  
   * Si va a crear un área de trabajo en una nueva suscripción creada después del 2 de abril de 2018, esta utilizará automáticamente el plan de precios *Por GB* y la opción para seleccionar un plan de tarifas no estará disponible.  Si va a crear un área de trabajo para una suscripción existente creada antes del 2 de abril o para una suscripción asociada a una inscripción de Contrato Enterprise (EA) existente, seleccione el plan de tarifa que prefiera.  Para más información sobre planes concretos, consulte los [detalles de precios de Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/).

        ![Creación de una hoja de recursos de Log Analytics](media/quick-create-workspace/create-loganalytics-workspace-02.png)  

3. Después de proporcionar la información necesaria en el panel **Área de trabajo de Log Analytics**, haga clic en **Aceptar**.  

Mientras se comprueba la información y se crea el espacio de trabajo, puede realizar un seguimiento de su progreso en **Notificaciones** en el menú. 

## <a name="troubleshooting"></a>Solución de problemas
Al crear un área de trabajo que se eliminó en los últimos 14 días y en [estado de eliminación temporal](../platform/delete-workspace.md#soft-delete-behavior), la operación podría tener un resultado diferente en función de la configuración del área de trabajo:
1. Si proporciona el mismo nombre de área de trabajo, grupo de recursos, suscripción y región que en el área de trabajo eliminado, se recuperará el área de trabajo, incluidos los datos, la configuración y los agentes conectados.
2. Si usa el mismo nombre de área de trabajo, pero un grupo de recursos, una suscripción o una región diferentes, obtendrá un error *El nombre del área de trabajo ya esta en uso. Pruebe otro*. Para invalidar la eliminación temporal y eliminar inmediatamente el área de trabajo y crear una con el mismo nombre, siga estos pasos para recuperar el área de trabajo primero y realizar una eliminación permanente:
   - [Recuperar](../platform/delete-workspace.md#recover-workspace) el área de trabajo
   - [Eliminación permanente del área de trabajo](../platform/delete-workspace.md#permanent-workspace-delete)
   - Crear una nueva área de trabajo con el mismo nombre

## <a name="next-steps"></a>Pasos siguientes
Ahora que tiene un área de trabajo disponible, puede configurar la recopilación de datos de telemetría de supervisión, realizar búsquedas en el registro para analizar dichos datos y agregar una solución de administración que proporcione datos adicionales e información de los análisis. 

* Para habilitar la recopilación de datos de recursos de Azure con Azure Diagnostics o Azure Storage, consulte [Recopilación de registros y métricas de Azure para servicios de Log Analytics](../platform/resource-logs.md#send-to-log-analytics-workspace).  
* [Agregue System Center Operations Manager como origen de datos](../platform/om-agents.md) para recopilar datos de agentes que informan a su grupo de administración de Operations Manager y almacenarlos en el repositorio del área de trabajo de Log Analytics. 
* Conecte [Configuration Manager](../platform/collect-sccm.md) para importar equipos que son miembros de colecciones en la jerarquía.  
* Revise las [soluciones de supervisión](../insights/solutions.md) disponibles y cómo agregar o quitar una solución del área de trabajo.

