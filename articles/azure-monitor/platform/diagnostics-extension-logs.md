---
title: Uso de Blob Storage para IIS y Table Storage para eventos en Azure Monitor | Microsoft Docs
description: Azure Monitor puede leer los registros de los servicios de Azure que escriben los diagnósticos en Table Storage o los registros de IIS escritos en Blob Storage.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/14/2020
ms.openlocfilehash: 095fd0b534c0dffaf80d2464fb9734f295335b84
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87317185"
---
# <a name="collect-data-from-azure-diagnostics-extension-to-azure-monitor-logs"></a>Recopilación de datos de Azure Diagnostics Extension en los registros de Azure Monitor
Azure Diagnostics Extension es un [agente de Azure Monitor](agents-overview.md) que recopila datos de supervisión del sistema operativo invitado de los recursos de proceso de Azure, incluidas las máquinas virtuales. En este artículo, se explica cómo se recopilan los datos obtenidos por la extensión de diagnóstico de Azure Storage en los registros de Azure Monitor.

> [!NOTE]
> Por lo general, el agente de Log Analytics de Azure Monitor es el mecanismo preferido para recopilar datos del sistema operativo invitado en registros de Azure Monitor. Consulte [Introducción a los agentes de Azure Monitor](agents-overview.md) para ver una comparativa detallada de los agentes.

## <a name="supported-data-types"></a>Tipos de datos admitidos
Azure Diagnostics Extension almacena los datos en una cuenta de Azure Storage. Para que los registros de Azure Monitor recopilen estos datos, deben estar en las siguientes ubicaciones:

| Tipo de registro | Tipo de recurso | Location |
| --- | --- | --- |
| Registros IIS |Virtual Machines <br> Roles web <br> Roles de trabajo |wad-iis-logfiles (Blob Storage) |
| syslog |Virtual Machines |LinuxsyslogVer2v0 (Table Storage) |
| Eventos operativos de Service Fabric |Nodos de Service Fabric |WADServiceFabricSystemEventTable |
| Service Fabric Reliable Actor Events |Nodos de Service Fabric |WADServiceFabricReliableActorEventTable |
| Service Fabric Reliable Service Events |Nodos de Service Fabric |WADServiceFabricReliableServiceEventTable |
| Registros de eventos de Windows |Nodos de Service Fabric <br> Virtual Machines <br> Roles web <br> Roles de trabajo |WADWindowsEventLogsTable (Table Storage) |
| Registros de ETW de Windows |Nodos de Service Fabric <br> Virtual Machines <br> Roles web <br> Roles de trabajo |WADETWEventTable (Table Storage) |

## <a name="data-types-not-supported"></a>Tipos de datos no admitidos

- Datos de rendimiento del sistema operativo invitado
- Registros de IIS procedentes de sitios web de Azure


## <a name="enable-azure-diagnostics-extension"></a>Habilitación de Azure Diagnostics Extension
Consulte el artículo [Instalación y configuración de Windows Azure Diagnostics Extension (WAD)](diagnostics-extension-windows-install.md) y [Uso de la extensión Diagnostics de Linux para supervisar métricas y registros](../../virtual-machines/extensions/diagnostics-linux.md) si necesita más información sobre la instalación y configuración de Diagnostics Extension. De este modo, podrá especificar la cuenta de almacenamiento y configurar la colección de datos que desea reenviar a los registros de Azure Monitor.


## <a name="collect-logs-from-azure-storage"></a>Recopilación de registros de Azure Storage
Utilice el procedimiento siguiente para habilitar la recopilación de datos de Diagnostics Extension desde una cuenta de Azure Storage:

1. En Azure Portal, vaya a las **Áreas de trabajo de Log Analytics** y seleccione su área de trabajo.
1. Haga clic en **Registros de las cuentas de almacenamiento** en la sección **Orígenes de datos del área de trabajo** del menú.
2. Haga clic en **Agregar**.
3. En **Cuenta de almacenamiento**, seleccione la cuenta de almacenamiento que contenga los datos que desea recopilar.
4. En **Tipos de datos**, seleccione el tipo de datos que desea recopilar.
5. El valor de Origen se rellenará automáticamente en función del tipo de datos elegido.
6. Haga clic en **Aceptar** para guardar la configuración.
7. Repita la operación con los demás tipos de datos.

En aproximadamente 30 minutos podrá ver los datos de la cuenta de almacenamiento en el área de trabajo de Log Analytics. Solo verá los datos que se escriban en el almacenamiento de una vez aplicada la configuración. El área de trabajo no lee los datos preexistentes de la cuenta de almacenamiento.

> [!NOTE]
> El portal no valida si el origen existe en la cuenta de almacenamiento ni si se escriben nuevos datos.



## <a name="next-steps"></a>Pasos siguientes

* [Recopilación de registros y métricas de los servicios de Azure](./resource-logs.md#send-to-log-analytics-workspace) para los servicios compatibles de Azure.
* [Incorporación de soluciones de Log Analytics desde la galería de soluciones](../insights/solutions.md) para más información sobre los datos.
* [Búsquedas de registros en Log Analytics](../log-query/log-query-overview.md) para analizar los datos.

