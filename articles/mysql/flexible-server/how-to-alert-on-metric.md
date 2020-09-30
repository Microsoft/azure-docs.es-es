---
title: Configuración de alertas de métricas de un servidor flexible de Azure Database for MySQL mediante Azure Portal
description: En este artículo se explica cómo configurar y acceder a las alertas de métricas de un servidor flexible de Azure Database for MySQL mediante Azure Portal.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: d4385ccda665e9acd2d2f9fd340e675b8a9dfe6e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932516"
---
# <a name="use-the-azure-portal-to-set-up-alerts-on-metrics-for-azure-database-for-mysql---flexible-server"></a>Uso de Azure Portal para configurar alertas en las métricas de un servidor flexible de Azure Database for MySQL 

> [!IMPORTANT] 
> Actualmente, la opción de implementación Servidor flexible de Azure Database for MySQL se encuentra en versión preliminar pública.

En este artículo se explica cómo configurar alertas de Azure Database for MySQL mediante Azure Portal. Puede recibir una alerta basada en las métricas de supervisión para los servicios de Azure.

La alerta se desencadena cuando el valor de una métrica especificada cruza el umbral que se ha asignado. La alerta se desencadena tanto la primera vez que se cumple la condición como después, cuando dicha condición deja de cumplirse. Como las alertas de métricas tienen un estado, solo envían notificaciones cuando cambia ese estado.

Puede configurar una alerta para realizar las siguientes acciones cuando se desencadene:
* Enviar notificaciones de correo electrónico al administrador y los coadministradores del servicio.
* Enviar un correo electrónico a direcciones de correo electrónico adicionales que especifique.
* Llamar a un webhook

Puede obtener información sobre las reglas de alerta y configurarlas mediante:
* [Azure Portal](../../azure-monitor/platform/alerts-metric.md#create-with-azure-portal)
* [CLI de Azure](../../azure-monitor/platform/alerts-metric.md#with-azure-cli)
* [API de REST de Azure Monitor](https://docs.microsoft.com/rest/api/monitor/metricalerts)

## <a name="create-an-alert-rule-on-a-metric-from-the-azure-portal"></a>Creación de una regla de alerta sobre una métrica desde Azure Portal
1. En [Azure Portal](https://portal.azure.com/), seleccione el servidor flexible de Azure Database for MySQL que desea supervisar.
2. En la sección **Supervisión** de la barra lateral, seleccione **Alertas**.
3. Seleccione **+ Nueva regla de alertas**.
4. Se abre la página **Crear regla**. Rellene la información necesaria:
5. En la sección **Condición**, elija **Seleccionar condición**.
6. Verá una lista de señales compatibles. Seleccione la métrica para la que desea crear una alerta. Por ejemplo, seleccione "Porcentaje de almacenamiento".
7. Verá un gráfico para la métrica de las últimas seis horas. Use la lista desplegable **Período del gráfico** para ver un historial más largo de la métrica.
8. Seleccione el tipo de umbral en el campo **Umbral** (por ejemplo, "Estático" o "Dinámico"), un **operador** (por ejemplo, "Mayor que") y un **tipo de agregación** (por ejemplo, promedio). Esto determinará la lógica que evaluará la regla de alertas de métricas.
    - Si va a usar un umbral **estático**, en **Valor de umbral** especifique un valor (por ejemplo, 85 por ciento). El gráfico de métricas puede ayudar a determinar cuál podría ser un umbral razonable.
    - Si usa un umbral **Dinámico**, siga para definir la **Sensibilidad del umbral**. El gráfico de métricas mostrará los umbrales calculados según los datos recientes. [Más información sobre las opciones de tipo y sensibilidad de la condición de umbrales dinámicos](../../azure-monitor/platform/alerts-dynamic-thresholds.md).
9. Refine la condición, para lo que debe ajustar el intervalo del campo **Granularidad de agregación (período)** , en el que se agrupan los puntos de datos mediante el uso de la función del tipo de agregación (por ejemplo, "30 minutos") y el valor del campo **Frecuencia** (por ejemplo, "Cada 15 minutos").
10. Haga clic en **Done**(Listo).
11. Agregue un grupo de acciones. Un grupo de acciones es una colección de las preferencias de notificación que el propietario de una suscripción de Azure define. En la sección **Grupos de acciones**, elija **Seleccionar grupo de acciones** para seleccionar un grupo de acciones ya existente y asociarlo a la regla de alerta.
12. También puede crear un nuevo grupo de acciones para recibir notificaciones de la alerta. Para más información, consulte [Creación y administración de un grupo de acciones](../../azure-monitor/platform/action-groups.md).
13. Para crear un grupo de acciones, elija **+ Crear grupo de acciones**. Rellene el formulario "Crear grupo de acciones" con los valores de los campos **Suscripción**, **Grupo de recursos**, **Nombre del grupo de acciones** y **Nombre para mostrar**.
14. Configure **notificaciones** del grupo de acciones.
    
    En **Tipo de notificación**, elija "Enviar por correo electrónico al rol de Azure Resource Manager" para seleccionar los propietarios, colaboradores y lectores de la suscripción que recibirán las notificaciones. Elija el **rol de Azure Resource Manager** para enviar el correo electrónico.
    También puede elegir **Email/SMS message/Push/Voice** (Correo electrónico/mensaje SMS/notificación push/mensaje de voz) para enviar notificaciones a destinatarios concretos.

    Especifique un **nombre** para el tipo de notificación y seleccione **Revisar y crear** cuando haya finalizado.

    <!--:::image type="content" source="./media/howto-alert-on-metric/10-action-group-type.png" alt-text="Action group":::-->
    
15. Rellene **Detalles de la regla de alertas** con los valores de los siguientes campos: **Nombre de la regla de alertas**, **Descripción**, **Guardar regla de alerta en el grupo de recursos** y **Gravedad**.

    <!--:::image type="content" source="./media/howto-alert-on-metric/11-name-description-severity.png" alt-text="Action group":::-->

16. Seleccione **Crear regla de alerta** para crear la alerta.
    En cuestión de minutos, se activa la alerta y se desencadena tal como se describió anteriormente.
## <a name="manage-your-alerts"></a>Administración de alertas
Una vez que haya creado una alerta, puede seleccionarla y realizar las acciones siguientes:

* Ver un gráfico que muestre el umbral de las métricas y los valores reales del día anterior en relación con esta alerta.
* **Editar** o **eliminar** la regla de alerta.
* **Deshabilitar** o **habilitar** la alerta, si quiere detener temporalmente o reanudar la recepción de notificaciones.


## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre cómo [establecer alertas en las métricas](../../azure-monitor/platform/alerts-metric.md).
- Obtenga más información sobre las [métricas disponibles en el servidor flexible de Azure Database for MySQL](./concepts-monitoring.md).
- [Descripción del funcionamiento de las alertas de métricas en Azure Monitor](../../azure-monitor/platform/alerts-metric-overview.md)
