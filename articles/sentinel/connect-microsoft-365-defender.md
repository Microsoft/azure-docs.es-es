---
title: Conexión de datos sin procesar de Microsoft 365 Defender con Azure Sentinel | Microsoft Docs
description: Aprenda a ingerir datos de eventos sin procesar de Microsoft 365 Defender en Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/13/2019
ms.author: yelevin
ms.openlocfilehash: e9e892ae08c65ee5c5008bbdc1b419d8dec30183
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932101"
---
# <a name="connect-data-from-microsoft-365-defender-to-azure-sentinel"></a>Conexión de datos de Microsoft 365 Defender con Azure Sentinel

> [!IMPORTANT]
>
> **Microsoft 365 Defender** se conocía antes como **Microsoft Threat Protection** o **MTP**.
>
> **Microsoft Defender para punto de conexión** se conocía antes como **Protección contra amenazas avanzada de Microsoft Defender** o **MDATP**.
>
> Es posible que vea que los nombres antiguos todavía están en uso durante un tiempo.

## <a name="background"></a>Fondo

El nuevo conector [Microsoft 365 Defender](https://docs.microsoft.com/microsoft-365/security/mtp/microsoft-threat-protection) le permite transmitir registros de **búsqueda avanzada**, un tipo de datos de eventos sin procesar, desde Microsoft 365 Defender hasta Azure Sentinel. 

Con la integración de [Microsoft Defender para punto de conexión (MDATP)](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection) en el paraguas de seguridad de Microsoft 365 Defender, ahora puede recopilar los eventos de [búsqueda avanzada](https://aka.ms/mdatpAH) de Microsoft Defender para punto de conexión mediante el conector Microsoft 365 Defender y transmitirlos directamente a nuevas tablas creadas específicamente en el área de trabajo de Azure Sentinel. Estas tablas se basan en el mismo esquema que se usa en el portal de Microsoft 365 Defender, lo que le ofrece acceso pleno al conjunto completo de registros de búsqueda avanzada y le permite hacer lo siguiente:

- Copiar fácilmente las consultas de búsqueda avanzada de ATP de Microsoft Defender existentes en Azure Sentinel.

- Usar los registros de eventos sin procesar para proporcionar información detallada adicional sobre las alertas, la búsqueda y la investigación, y correlacionar los eventos con datos de orígenes de datos adicionales en Azure Sentinel.

- Almacenar los registros con mayor periodo de retención, por encima de la retención predeterminada de 30 días de Microsoft Defender para punto de conexión o de Microsoft 365 Defender. Para ello, configure la retención del área de trabajo o configure la retención por tabla en Log Analytics.

> [!IMPORTANT]
>
> El conector Microsoft 365 Defender está actualmente en versión preliminar pública. Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

- Debe tener una licencia válida de Microsoft Defender para punto de conexión, como se describe en [Configuración de la implementación de Microsoft Defender para punto de conexión](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/licensing). 

- El usuario debe tener asignado el rol de administrador global en el inquilino (en Azure Active Directory).

## <a name="connect-to-microsoft-365-defender"></a>Conexión a Microsoft 365 Defender

Si Microsoft Defender para punto de conexión está implementado e ingiriendo datos, los registros de eventos se pueden transmitir a Azure Sentinel fácilmente.

1. En Azure Sentinel, seleccione **Data connectors** (Conectores de datos), elija **Microsoft 365 Defender (Preview)** (Microsoft 365 Defender [versión preliminar]) en la galería y seleccione **Open connector page** (Abrir página del conector).

1. Los siguientes tipos de eventos se pueden recopilar de sus tablas de búsqueda avanzada correspondientes. Marque las casillas de los tipos de evento que quiere recopilar:

    | Tipo de eventos | Nombre de la tabla |
    |-|-|
    | Información de la máquina (incluida la información del sistema operativo) | DeviceInfo |
    | Propiedades de red de las máquinas | DeviceNetworkInfo |
    | Creación de procesos y eventos relacionados | DeviceProcessEvents |
    | Conexión de red y eventos relacionados | DeviceNetworkEvents |
    | Creación y modificación de archivos, y otros eventos del sistema de archivos | DeviceFileEvents |
    | Creación y modificación de entradas del Registro | DeviceRegistryEvents |
    | Inicios de sesión y otros eventos de autenticación | DeviceLogonEvents |
    | Eventos de carga de DLL | DeviceImageLoadEvents |
    | Tipos de eventos adicionales | DeviceEvents |
    |

1. Haga clic en **Aplicar cambios**. 

1. Para consultar las tablas de búsqueda avanzada en Log Analytics, escriba el nombre de la tabla de la lista anterior en la ventana de consulta.

## <a name="verify-data-ingestion"></a>Comprobación de la ingesta de datos

El grafo de datos de la página del conector indica que se están ingiriendo datos. Observará que se muestra una sola línea con el volumen de eventos agregados en todas las tablas habilitadas. Cuando haya habilitado el conector, puede usar la siguiente consulta KQL para generar un grafo de volumen de eventos similar para una sola tabla (cambie la tabla *DeviceEvents* por la tabla necesaria de su elección):

```kusto
let Now = now();
(range TimeGenerated from ago(14d) to Now-1d step 1d
| extend Count = 0
| union isfuzzy=true (
    DeviceEvents
    | summarize Count = count() by bin_at(TimeGenerated, 1d, Now)
)
| summarize Count=max(Count) by bin_at(TimeGenerated, 1d, Now)
| sort by TimeGenerated
| project Value = iff(isnull(Count), 0, Count), Time = TimeGenerated, Legend = "Events")
| render timechart
```

En la pestaña **Pasos siguientes**, encontrará algunas consultas de ejemplo que se han incluido. Puede ejecutarlas en el acto o modificarlas y guardarlas.

## <a name="next-steps"></a>Pasos siguientes
En este documento, aprendió a obtener datos de eventos sin procesar de Microsoft Defender para punto de conexión en Azure Sentinel mediante el conector Microsoft 365 Defender. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats.md).
