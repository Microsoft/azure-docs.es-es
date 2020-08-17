---
title: Registros DNS del centro de datos de Azure utilizados por Azure Automation | Microsoft Docs
description: En este artículo se proporcionan los registros DNS requeridos por las características de Azure Automation al restringir la comunicación a una región específica de Azure que hospeda esa cuenta de Automation.
services: automation
ms.subservice: process-automation
ms.date: 07/23/2020
ms.topic: conceptual
ms.openlocfilehash: 17d0857a8979cfcc632ab8951fb255f97229a665
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87117186"
---
# <a name="dns-records-for-azure-regions-used-by-azure-automation"></a>Registros DNS para regiones de Azure utilizados por Azure Automation

El servicio de [Azure Automation](../automation-intro.md) utiliza varios registros DNS para que las características se conecten al servicio. Si tiene una cuenta de Automation definida para una región específica, puede restringir la comunicación con ese centro de datos regional. Es posible que necesite conocer estos registros para permitir que las siguientes características de Automation funcionen cuando se hospedan detrás de un firewall:

* Hybrid Runbook Worker
* State Configuration
* webhooks

>[!NOTE]
>El registro de Hybrid Runbook Worker de Linux producirá un error con los nuevos registros a menos que sea de la versión 1.6.10.2 o posterior. Debe actualizar a una versión más reciente del [agente de Log Analytics para Linux](../../azure-monitor/platform/agent-linux.md) para que la máquina reciba una versión actualizada del rol de trabajo y use estos nuevos registros. Los equipos existentes seguirán funcionando sin problemas.  

## <a name="dns-records-per-region"></a>Registros DNS por región

En la tabla siguiente se proporciona el registro de DNS para cada región.

>[!NOTE]
>Aunque la lista de los registros DNS de Automation que se proporcionan aquí se ha retirado, siguen siendo funcionales para darle tiempo de migrar a los nuevos registros que aparecen en [Compatibilidad con el vínculo privado](#support-for-private-link) y evitar errores con los procesos de automatización.

| **Región** | **Registro de DNS** |
| --- | --- |
| Centro de Australia |ac-jobruntimedata-prod-su1.azure-automation.net</br>ac-agentservice-prod-1.azure-automation.net |
| Este de Australia |ae-jobruntimedata-prod-su1.azure-automation.net</br>ae-agentservice-prod-1.azure-automation.net |
| Sudeste de Australia |ase-jobruntimedata-prod-su1.azure-automation.net</br>ase-agentservice-prod-1.azure-automation.net |
| Centro de Canadá |cc-jobruntimedata-prod-su1.azure-automation.net</br>cc-agentservice-prod-1.azure-automation.net |
| Centro de la India |cid-jobruntimedata-prod-su1.azure-automation.net</br>cid-agentservice-prod-1.azure-automation.net |
| Este de EE. UU. 2 |eus2-jobruntimedata-prod-su1.azure-automation.net</br>eus2-agentservice-prod-1.azure-automation.net |
| Japón Oriental |jpe-jobruntimedata-prod-su1.azure-automation.net</br>jpe-agentservice-prod-1.azure-automation.net |
| Norte de Europa |ne-jobruntimedata-prod-su1.azure-automation.net</br>ne-agentservice-prod-1.azure-automation.net |
| Centro-sur de EE. UU. |scus-jobruntimedata-prod-su1.azure-automation.net</br>scus-agentservice-prod-1.azure-automation.net |
| Sudeste de Asia |sea-jobruntimedata-prod-su1.azure-automation.net</br>sea-agentservice-prod-1.azure-automation.net|
| Sur de Reino Unido | uks-jobruntimedata-prod-su1.azure-automation.net</br>uks-agentservice-prod-1.azure-automation.net |
| US Gov - Virginia | usge-jobruntimedata-prod-su1.azure-automation.us<br>usge-agentservice-prod-1.azure-automation.us |
| Centro-Oeste de EE. UU. | wcus-jobruntimedata-prod-su1.azure-automation.net</br>wcus-agentservice-prod-1.azure-automation.net |
| Oeste de Europa |we-jobruntimedata-prod-su1.azure-automation.net</br>we-agentservice-prod-1.azure-automation.net |
| Oeste de EE. UU. 2 |wus2-jobruntimedata-prod-su1.azure-automation.net</br>wus2-agentservice-prod-1.azure-automation.net |

### <a name="support-for-private-link"></a>Compatibilidad con el vínculo privado

Para admitir el [vínculo privado](../../private-link/private-link-overview.md) en Azure Automation, se han actualizado los registros DNS de cada centro de datos compatible. En lugar de direcciones URL específicas de la región, las direcciones URL son específicas de la cuenta de Automation.

| **Región** | **Registro de DNS** |
| --- | --- |
| Centro-Oeste de EE. UU. |`https://<accountId>.webhook.wcus.azure-automation.net`<br>`https://<accountId>.agentsvc.wcus.azure-automation.net`<br>`https://<accountId>.jrds.wcus.azure-automation.net` |
| Oeste de EE. UU. |`https://<accountId>.webhook.wus.azure-automation.net`<br>`https://<accountId>.agentsvc.wus.azure-automation.net`<br>`https://<accountId>.jrds.wus.azure-automation.net` |
| Oeste de EE. UU. 2 |`https://<accountId>.webhook.wus2.azure-automation.net`<br>`https://<accountId>.agentsvc.wus2.azure-automation.net`<br>`https://<accountId>.jrds.wus2.azure-automation.net` |
| Centro de EE. UU. |`https://<accountId>.webhook.cus.azure-automation.net`<br>`https://<accountId>.agentsvc.cus.azure-automation.net`<br>`https://<accountId>.jrds.cus.azure-automation.net` |
| Centro-sur de EE. UU. |`https://<accountId>.webhook.scus.azure-automation.net`<br>`https://<accountId>.agentsvc.scus.azure-automation.net`<br>`https://<accountId>.jrds.scus.azure-automation.net` |
| Centro-Norte de EE. UU |`https://<accountId>.webhook.ncus.azure-automation.net`<br>`https://<accountId>.agentsvc.ncus.azure-automation.net`<br>`https://<accountId>.jrds.ncus.azure-automation.net` |
| Este de EE. UU. |`https://<accountId>.webhook.eus.azure-automation.net`<br>`https://<accountId>.agentsvc.eus.azure-automation.net`<br>`https://<accountId>.jrds.eus.azure-automation.net` |
| Este de EE. UU. 2 |`https://<accountId>.webhook.eus2.azure-automation.net`<br>`https://<accountId>.agentsvc.eus2.azure-automation.net`<br>`https://<accountId>.jrds.eus2.azure-automation.net` |
| Centro de Canadá |`https://<accountId>.webhook.cc.azure-automation.net`<br>`https://<accountId>.agentsvc.cc.azure-automation.net`<br>`https://<accountId>.jrds.cc.azure-automation.net` |
| Oeste de Europa |`https://<accountId>.webhook.we.azure-automation.net`<br>`https://<accountId>.agentsvc.we.azure-automation.net`<br>`https://<accountId>.jrds.we.azure-automation.net` |
| Norte de Europa |`https://<accountId>.webhook.ne.azure-automation.net`<br>`https://<accountId>.agentsvc.ne.azure-automation.net`<br>`https://<accountId>.jrds.ne.azure-automation.net` |
| Sudeste de Asia |`https://<accountId>.webhook.sea.azure-automation.net`<br>`https://<accountId>.agentsvc.sea.azure-automation.net`<br>`https://<accountId>.jrds.sea.azure-automation.net` |
| Este de Asia |`https://<accountId>.webhook.ea.azure-automation.net`<br>`https://<accountId>.agentsvc.ea.azure-automation.net`<br>`https://<accountId>.jrds.ea.azure-automation.net` |
| Centro de la India |`https://<accountId>.webhook.cid.azure-automation.net`<br>`https://<accountId>.agentsvc.cid.azure-automation.net`<br>`https://<accountId>.jrds.cid.azure-automation.net` |
| Japón Oriental |`https://<accountId>.webhook.jpe.azure-automation.net`<br>`https://<accountId>.agentsvc.jpe.azure-automation.net`<br>`https://<accountId>.jrds.jpe.azure-automation.net` |
| Centro de Corea del Sur |`https://<accountId>.webhook.kc.azure-automation.net`<br>`https://<accountId>.agentsvc.kc.azure-automation.net`<br>`https://<accountId>.jrds.kc.azure-automation.net` |
| Sudeste de Australia |`https://<accountId>.webhook.ase.azure-automation.net`<br>`https://<accountId>.agentsvc.ase.azure-automation.net`<br>`https://<accountId>.jrds.ase.azure-automation.net` |
| Este de Australia |`https://<accountId>.webhook.ae.azure-automation.net`<br>`https://<accountId>.agentsvc.ae.azure-automation.net`<br>`https://<accountId>.jrds.ae.azure-automation.net` |
| Centro de Australia |`https://<accountId>.webhook.ac.azure-automation.net`<br>`https://<accountId>.agentsvc.ac.azure-automation.net`<br>`https://<accountId>.jrds.ac.azure-automation.net` |
| Sur de Reino Unido |`https://<accountId>.webhook.uks.azure-automation.net`<br>`https://<accountId>.agentsvc.uks.azure-automation.net`<br>`https://<accountId>.jrds.uks.azure-automation.net` |
| Centro de Francia |`https://<accountId>.webhook.fc.azure-automation.net`<br>`https://<accountId>.agentsvc.fc.azure-automation.net`<br>`https://<accountId>.jrds.fc.azure-automation.net` |
| Norte de Sudáfrica |`https://<accountId>.webhook.san.azure-automation.net`<br>`https://<accountId>.agentsvc.san.azure-automation.net`<br>`https://<accountId>.jrds.san.azure-automation.net` |
| Sur de Brasil |`https://<accountId>.webhook.brs.azure-automation.net`<br>`https://<accountId>.agentsvc.brs.azure-automation.net`<br>`https://<accountId>.jrds.brs.azure-automation.net` |
| Norte de China |`https://<accountId>.webhook.bjb.azure-automation.cn`<br>`https://<accountId>.agentsvc.bjb.azure-automation.cn`<br>`https://<accountId>.jrds.bjb.azure-automation.cn` |
| Norte de China 2 |`https://<accountId>.webhook.bjs2.azure-automation.cn`<br>`https://<accountId>.agentsvc.bjs2.azure-automation.cn`<br>`https://<accountId>.jrds.bjs2.azure-automation.cn` |
| Este de China 2 |`https://<accountId>.webhook.sha2.azure-automation.cn`<br>`https://<accountId>.agentsvc.sha2.azure-automation.cn`<br>`https://<accountId>.jrds.sha2.azure-automation.cn` |
| US Gov - Virginia |`https://<accountId>.webhook.usge.azure-automation.us`<br>`https://<accountId>.agentsvc.usge.azure-automation.us`<br>`https://<accountId>.jrds.usge.azure-automation.us` |
| US Gov Texas |`https://<accountId>.webhook.ussc.azure-automation.us`<br>`https://<accountId>.agentsvc.ussc.azure-automation.us`<br>`https://<accountId>.jrds.ussc.azure-automation.us` |
| US Gov: Arizona |`https://<accountId>.webhook.phx.azure-automation.us`<br>`https://<accountId>.agentsvc.phx.azure-automation.us`<br>`https://<accountId>.jrds.phx.azure-automation.us` |

Reemplace `<accountId>` en el registro DNS por el GUID que representa el id. de la cuenta de Automation de la **dirección URL** del valor. Puede obtener el id. necesario de la sección **Claves** en **Configuración de la cuenta** de Azure Portal.

![Página de clave principal de la cuenta de Automation](./media/automation-region-dns-records/automation-account-keys.png)

Copie el valor después de *accounts/* desde el campo **URL**: `https://<GUID>.agentsvc.<region>.azure-automation.net/accounts/<GUID>`

Se recomienda utilizar las direcciones mostradas al definir [excepciones](../automation-runbook-execution.md#exceptions). Para obtener una lista de direcciones IP de regiones en lugar de nombres de regiones, descargue el archivo JSON desde el Centro de descarga de Microsoft para los siguientes entornos de nube:

* [Rangos de direcciones IP y etiquetas de servicio de Azure: Azure público](https://www.microsoft.com/download/details.aspx?id=56519)
* [Rangos de direcciones IP y etiquetas de servicio de Azure: Azure Government](https://www.microsoft.com/download/details.aspx?id=57063)
* [Rangos de direcciones IP y etiquetas de servicio de Azure: Azure Alemania](https://www.microsoft.com/download/details.aspx?id=57064)
* [Rangos de direcciones IP y etiquetas de servicio de Azure: Azure China 21Vianet](https://www.microsoft.com/download/details.aspx?id=57062)

El archivo de direcciones IP muestra los intervalos de direcciones IP que se usan en los centros de datos de Microsoft Azure. Incluye intervalos de proceso, SQL y almacenamiento, y refleja los intervalos implementados actualmente y los próximos cambios en los intervalos IP. Los nuevos intervalos que aparecen en el archivo no se utilizan en los centros de datos durante al menos una semana.

Es conveniente descargar el nuevo archivo de direcciones IP cada semana. A continuación, actualice el sitio para identificar correctamente los servicios que se ejecutan en Azure.

> [!NOTE]
> Si utiliza Azure ExpressRoute, recuerde que el archivo de direcciones IP se usa para actualizar la publicidad del Protocolo de puerta de enlace de borde (BGP) del espacio de Azure la primera semana de cada mes.

## <a name="next-steps"></a>Pasos siguientes

* Para aprender a solucionar problemas con las instancias de Hybrid Runbook Worker, consulte [Solución de incidencias de Hybrid Runbook Worker](../troubleshoot/hybrid-runbook-worker.md#general).

* Para obtener información sobre cómo solucionar problemas relacionados con la configuración de estado, consulte [Solución de problemas de State Configuration](../troubleshoot/desired-state-configuration.md).