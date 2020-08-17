---
title: 'Control de seguridad de Azure: Registro y supervisión'
description: Registro y supervisión del control de seguridad de Azure
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 82114164d70eae71678e70ff2bdb7ea44a54d4cd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076302"
---
# <a name="security-control-logging-and-monitoring"></a>Control de seguridad: Registro y supervisión

El registro y la supervisión de seguridad se centra en las actividades relacionadas con la habilitación, adquisición y almacenamiento de registros de auditoría de los servicios de Azure.

## <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 2.1 | 6.1 | Microsoft |

Microsoft mantiene los orígenes de hora de los recursos de Azure; sin embargo, tiene la opción de administrar la configuración de la sincronización de hora de los recursos de proceso.

- [Configuración de la sincronización de hora de los recursos de proceso de Windows de Azure](https://docs.microsoft.com/azure/virtual-machines/windows/time-sync)

- [Configuración de la sincronización de hora de los recursos de proceso de Linux de Azure](https://docs.microsoft.com/azure/virtual-machines/linux/time-sync)

## <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 2.2 | 6.5, 6.6 | Customer |

Recopile registros mediante Azure Monitor para agregar datos de seguridad generados por dispositivos de punto de conexión, recursos de red y otros sistemas de seguridad. En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y utilice cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros. 

- [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

- [Recopilación de registros y métricas de plataforma con Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings)

- [Recopilación de registros de host internos de máquina virtual de Azure con Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/learn/quick-collect-azurevm)

- [Introducción a Azure Monitor e integración con herramienta SIEM de terceros](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

## <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 2.3 | 6.2, 6.3 | Customer |

Habilite Configuración de diagnóstico en los recursos de Azure para poder acceder a los registros de auditoría, seguridad y diagnóstico. Los registros de actividad, que están disponibles automáticamente, incluyen el origen del evento, la fecha, el usuario, la marca de tiempo, las direcciones de origen y de destino, y otros elementos útiles.

- [Recopilación de registros y métricas de plataforma con Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings)

- [Descripción del registro y de los distintos tipos de registro de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/platform-logs-overview)

## <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 2.4 | 6.2, 6.3 | Customer |

Si el recurso de proceso es propiedad de Microsoft, este será responsable de su supervisión. Si el recurso de proceso es propiedad de su organización, la responsabilidad de su supervisión recaerá sobre usted. Puede usar Azure Security Center para supervisar el sistema operativo. Entre los datos que recopila Security Center del sistema operativo se incluyen: tipo y versión del sistema operativo, sistema operativo (registros de eventos de Windows), procesos en ejecución, nombre de la máquina, direcciones IP y usuario conectado. El agente de Log Analytics también recopila los archivos de volcado de memoria.

- [Recopilación de registros de host internos de máquina virtual de Azure con Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/learn/quick-collect-azurevm)

- [Descripción de la recopilación de datos de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-enable-data-collection)

## <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 2.5 | 6.4 | Customer |

En Azure Monitor, establezca el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

- [Cambio del período de retención de datos en Log Analytics](https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period)

- [Configuración de la directiva de retención para los registros de la cuenta de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-monitor-storage-account#configure-logging)

## <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 2.6 | 6.7 | Customer |

Analice y supervise los registros en busca de comportamientos anómalos y revise los resultados con regularidad. Use el área de trabajo de Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos del registro.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros. 

- [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

- [Descripción del área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal)

- [Procedimiento para realizar consultas personalizadas en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries)

## <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 2.7 | 6,8 | Customer |

Use Azure Security Center con el área de trabajo de Log Analytics para la supervisión y alertas sobre actividades anómalas que se encuentran en los registros y eventos de seguridad.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel.

- [Incoporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

- [Administración de alertas de seguridad en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts)

- [Alertas sobre datos de registro de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response)

## <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 2.8 | 8.6 | Customer |

Habilite la recopilación de eventos antimalware para Azure Virtual Machines y Cloud Services.

- [Configuración de Microsoft Antimalware para Virtual Machines](/powershell/module/servicemanagement/azure.service/set-azurevmmicrosoftantimalwareextension?view=azuresmps-4.0.0)

- [Configuración de Microsoft Antimalware para Cloud Services](/powershell/module/servicemanagement/azure.service/set-azureserviceantimalwareextension?view=azuresmps-4.0.0)

- [Descripción de Microsoft Antimalware](https://docs.microsoft.com/azure/security/fundamentals/antimalware)

## <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 2.9 | 8,7 | Customer |

Implemente una solución de terceros de Azure Marketplace para la solución de registro DNS según las necesidades de su organización.  

## <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 2.10 | 8.8 | Customer |

Use Microsoft Monitoring Agent en todas las máquinas virtuales Windows de Azure compatibles para registrar el evento de creación de procesos y el campo CommandLine.   En el caso de las máquinas virtuales Linux de Azure compatibles, puede configurar manualmente el registro de la consola en cada nodo y usar Syslog para almacenar los datos.  Además, use el área de trabajo de Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos registrados de las máquinas virtuales de Azure. 

- [Recopilación de datos en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-enable-data-collection#data-collection-tier)

- [Procedimiento para realizar consultas personalizadas en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries)

- [Orígenes de datos de Syslog en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/data-sources-syslog)


## <a name="next-steps"></a>Pasos siguientes

- Consulte el control de seguridad siguiente: [Identidad y control de acceso](security-control-identity-access-control.md)