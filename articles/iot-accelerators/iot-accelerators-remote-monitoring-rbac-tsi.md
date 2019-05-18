---
title: Los datos de supervisión remotos obtener acceso a controlar - Azure | Microsoft Docs
description: En este artículo se proporciona información sobre cómo puede configurar controles de acceso para el explorador de telemetría de Time Series Insights en el acelerador de soluciones de supervisión remota
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 08/06/2018
ms.topic: conceptual
ms.openlocfilehash: 9d5d572c3e32e3645e65ba8d6fc28b567b3c1e9a
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827171"
---
# <a name="configure-access-controls-for-the-time-series-insights-telemetry-explorer"></a>Configurar controles de acceso para el explorador de telemetría de Time Series Insights

En este artículo se proporciona información sobre cómo configurar controles de acceso para el explorador de Time Series Insights en el acelerador de soluciones de supervisión remota. Para permitir que los usuarios del acelerador de soluciones accedan al explorador de Time Series Insights, deberá conceder acceso a los datos a cada usuario.

Las directivas de acceso a datos conceden permisos para emitir consultas de datos, manipular datos de referencia en el entorno y compartir consultas guardadas y perspectivas asociadas con el entorno.

## <a name="grant-data-access"></a>Concesión de acceso a datos

Siga estos pasos para conceder acceso a datos a una entidad de seguridad de usuario:

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).

2. Busque el entorno de Time Series Insights. Escriba **Time Series** en el cuadro de **búsqueda**. Seleccione **Entorno de Time Series** en los resultados de búsqueda. 

3. Seleccione el entorno de Time Series Insights de la lista.

4. Seleccione **Data Access Policies** (Directivas de acceso a datos) y, luego, **+ Agregar**.
    ![Administración del entorno del origen de Time Series Insights: entorno](media/iot-accelerators-remote-monitoring-rbac-tsi/getstarted-grant-data-access1.png)

5. Seleccione **Seleccionar usuario**.  Busque el nombre de usuario o la dirección de correo para encontrar al usuario que quiere agregar. Haga clic en **Seleccionar** para confirmar la selección. 

    ![Administración del origen de Time Series Insights: agregar](media/iot-accelerators-remote-monitoring-rbac-tsi/getstarted-grant-data-access2.png)

6. Seleccione **Seleccionar rol**. Elija el rol de acceso adecuado para el usuario:
   - Seleccione **Colaborador** si desea permitir al usuario cambiar los datos de referencia y compartir las consultas almacenadas y las perspectivas con otros usuarios del entorno. 
   - En caso contrario, seleccione **Lector** para permitir al usuario consultar los datos del entorno y guardar consultas personales (no compartidas) en el entorno.

     Seleccione **Aceptar** para confirmar la elección del rol.

     ![Administración del origen de Time Series Insights: seleccionar usuario](media/iot-accelerators-remote-monitoring-rbac-tsi/getstarted-grant-data-access3.png)

7. Seleccione **Aceptar** en la página **Select User Role** (Seleccionar rol de usuario).

    ![Administración del origen de Time Series Insights: seleccionar rol](media/iot-accelerators-remote-monitoring-rbac-tsi/getstarted-grant-data-access4.png)

8. En la página **Data Access Policies** (Directivas de acceso a datos) se muestran los usuarios y los roles de cada uno.

    ![Administración del origen de Time Series Insights: resultados](media/iot-accelerators-remote-monitoring-rbac-tsi/getstarted-grant-data-access5.png)

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido cómo se conceden los controles de acceso para el explorador de Time Series Insights en el acelerador de soluciones de supervisión remota.

Para información más conceptual sobre el acelerador de la solución de supervisión remota, vea [Arquitectura de supervisión remota](iot-accelerators-remote-monitoring-sample-walkthrough.md).

Para más información sobre cómo personalizar la solución de supervisión remota, consulte [Personalización y nueva implementación de un microservicio](iot-accelerators-microservices-example.md).
<!-- Next tutorials in the sequence -->