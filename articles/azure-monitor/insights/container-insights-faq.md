---
title: Preguntas más frecuentes sobre Azure Monitor para contenedores | Microsoft Docs
description: Azure Monitor para contenedores es una solución que supervisa el estado de los clústeres de AKS y Container Instances en Azure. En este artículo se responden preguntas comunes.
services: azure-monitor
author: mgoedtel
manager: carmonm
editor: tysonn
ms.service: azure-monitor
ms.topic: article
ms.workload: infrastructure-services
ms.date: 04/17/2019
ms.author: magoedte
ms.openlocfilehash: afa332b40884a79b5114b3b8093cd27108c39984
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780005"
---
# <a name="azure-monitor-for-containers-frequently-asked-questions"></a>Preguntas más frecuentes sobre Azure Monitor para contenedores

En este artículo de preguntas frecuentes de Microsoft, se presenta una lista con las preguntas frecuentes sobre Azure Monitor para contenedores. Si tiene alguna otra pregunta sobre esta solución, vaya al [foro de discusión](https://feedback.azure.com/forums/34192--general-feedback) y publíquela. Si una pregunta es frecuente, se agrega a este artículo para que se pueda encontrar de forma rápida y sencilla.

## <a name="why-dont-i-see-data-in-my-log-analytics-workspace"></a>¿Por qué no veo datos en Mi área de trabajo de Log Analytics?

Si no puede ver ningún dato en el área de trabajo de Log Analytics en un diario de tiempo determinado, podría haber alcanzado el límite de 500 MB de forma predeterminada o el límite diario especificado para controlar la cantidad de datos para recopilar diariamente. Cuando se alcanza el límite para el día, la recopilación de datos se detiene y se reanuda sólo en el día siguiente. Para revisar el uso de datos y actualizar a un plan de tarifa diferente en función de los patrones de uso previsto, consulte [uso de datos y el costo de registro](../platform/manage-cost-storage.md). 

## <a name="what-are-the-container-states-specified-in-the-containerinventory-table"></a>¿Cuáles son los Estados de contenedor especificados en la tabla ContainerInventory?

La tabla ContainerInventory contiene información sobre los contenedores detenidos y en ejecución. La tabla se rellena con un flujo de trabajo dentro del agente que consulta el docker de todos los contenedores (en ejecución y detenidos) y reenvía dichos datos al área de trabajo de Log Analytics.
 
## <a name="how-do-i-resolve-missing-subscription-registration-error"></a>¿Cómo resolver **registro de la suscripción que faltan** error?

Si recibe el error **registro de la suscripción que faltan para Microsoft.OperationsManagement**, puede resolverlo mediante el registro del proveedor de recursos **Microsoft.OperationsManagement** en el suscripción donde se define el área de trabajo. La documentación sobre cómo hacer esto se puede encontrar [aquí](../../azure-resource-manager/resource-manager-register-provider-errors.md).

## <a name="is-there-support-for-rbac-enabled-aks-clusters"></a>¿Hay compatibilidad con RBAC ha habilitado los clústeres de AKS?

La solución de supervisión de contenedores no es compatible con RBAC, pero es compatible con Azure Monitor para contenedores. La página de detalles de la solución no puede mostrar la información correcta en las hojas que muestran datos para estos clústeres.

## <a name="how-do-i-enable-log-collection-for-containers-in-the-kube-system-namespace-through-helm"></a>¿Cómo habilitar la recopilación de registros para contenedores en el espacio de nombres kube-system mediante Helm?

La recopilación de registros de contenedores en el espacio de nombres kube-system está deshabilitada de forma predeterminada. La recopilación de registros puede habilitarse mediante la configuración de una variable de entorno en omsagent. Para obtener más información, consulte el [Azure Monitor para contenedores](https://github.com/helm/charts/tree/master/incubator/azuremonitor-containers) página de GitHub. 

## <a name="how-do-i-update-the-omsagent-to-the-latest-released-version"></a>¿Cómo se puede actualizar el complemento omsagent a la última versión?

Para obtener información sobre cómo actualizar el agente, vea la información sobre la [administración del agente](container-insights-manage-agent.md).

## <a name="how-do-i-enable-multi-line-logging"></a>¿Cómo se puede habilitar el registro de varias líneas?

Actualmente, Azure Monitor para contenedores no admite el registro de varias líneas, pero hay soluciones alternativas disponibles. Puede configurar todos los servicios para que escriban en formato JSON y, a continuación, Docker/Moby los escribirán como una sola línea.

Por ejemplo, puede ajustar el registro como un objeto JSON como se muestra en el ejemplo siguiente para una aplicación de Node.js de ejemplo:

```
console.log(json.stringify({ 
      "Hello": "This example has multiple lines:",
      "Docker/Moby": "will not break this into multiple lines",
      "and you will receive":"all of them in log analytics",
      "as one": "log entry"
      }));
```

Estos datos tendrá un aspecto similar al ejemplo siguiente en Azure Monitor para los registros al realizar una consulta para él:

```
LogEntry : ({“Hello": "This example has multiple lines:","Docker/Moby": "will not break this into multiple lines", "and you will receive":"all of them in log analytics", "as one": "log entry"}

```

Para obtener una visión detallada del problema, visite el siguiente [vínculo de GitHub](https://github.com/moby/moby/issues/22920).

## <a name="how-do-i-resolve-azure-ad-errors-when-i-enable-live-logs"></a>¿Cómo resolver errores de Azure AD cuando se habilita los registros en vivo? 

Puede ver el siguiente error: **La respuesta a la dirección url especificada en la solicitud no coincide con las direcciones URL de respuesta configuradas para la aplicación: ' < Id. de aplicación\>'**. La solución para solucionar el problema puede encontrarse en el artículo [cómo ver los tiempo de real de registros de contenedor con Azure Monitor para contenedores](container-insights-live-logs.md#configure-aks-with-azure-active-directory). 

## <a name="why-cant-i-upgrade-cluster-after-onboarding"></a>¿Por qué no puedo actualizar el clúster después de la incorporación?

Si después de habilitar Azure Monitor para los contenedores para un clúster de AKS, eliminar el área de trabajo de Log Analytics el clúster estaba enviando sus datos, al intentar actualizar el clúster se producirá un error. Para solucionar este problema, tendrá que deshabilitar la supervisión y, a continuación, volver a habilitarla que hacen referencia a un área de trabajo válido diferente en su suscripción. Al intentar volver a realizar la actualización del clúster, debe procesar y se completa correctamente.  

## <a name="which-ports-and-domains-do-i-need-to-openwhitelist-for-the-agent"></a>¿Qué puertos y dominios necesito para abrir/lista de permitidos para el agente?
- *.ods.opinsights.azure.com   443
- *.oms.opinsights.azure.com   443
- *.blob.core.windows.net      443
- dc.services.visualstudio.com 443

## <a name="next-steps"></a>Pasos siguientes

Para comenzar a supervisar el clúster de AKS, revise [Cómo incorporar Azure Monitor para contenedores](container-insights-onboard.md) para conocer los requisitos y los métodos disponibles para habilitar la supervisión. 
