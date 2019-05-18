---
title: Use Application Insights para supervisar Azure Kubernetes Service (AKS) o en otra Kubernetes hospedado - aplicaciones de Azure Monitor | Microsoft Docs
description: Monitor de Azure usa tecnología de malla de servicio, Istio, en el clúster de Kubernetes para proporcionar supervisión de aplicaciones para cualquier aplicación hospedado de Kubernetes. Esto le permite recopilar datos de telemetría de Application Insights que pertenecen a las solicitudes entrantes y salientes a y desde los pods que se ejecutan en el clúster.
services: application-insights
author: tokaplan
manager: carmonm
ms.service: application-insights
ms.topic: conceptual
ms.date: 04/25/2019
ms.author: alkaplan
ms.openlocfilehash: c94d589875195207ec6f71c35ad077cac281fda5
ms.sourcegitcommit: 9e8dfa1169a55c3c8af93a6c5f4e0dace4de48b2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65555833"
---
# <a name="zero-instrumentation-application-monitoring-for-kubernetes-hosted-applications"></a>Cero supervisión de aplicaciones de instrumentación para Kubernetes las aplicaciones hospedadas

> [!IMPORTANT]
> Esta funcionalidad está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Monitor ahora aprovecha tech de malla de servicio en el clúster de Kubernetes para proporcionar fuera de la supervisión de aplicaciones de la casilla para cualquier aplicación hospedado de Kubernetes. No tiene valor predeterminado, como las características de Application Insight [mapa de aplicación](../../azure-monitor/app/app-map.md) para modelar las dependencias, [Live Stream de métricas](../../azure-monitor/app/live-stream.md) para la supervisión en tiempo real, visualizaciones eficaces con los [predeterminada panel](../../azure-monitor/app/overview-dashboard.md), [Explorador de métricas](../../azure-monitor/platform/metrics-getting-started.md), y [libros](../../azure-monitor/app/usage-workbooks.md). Esta característica le ayudará a los usuarios detectar los cuellos de botella y zonas activas de un error en todas sus cargas de trabajo de Kubernetes dentro de un espacio de nombres de Kubernetes seleccionado. Al aprovechar sus inversiones existentes de malla de servicio con tecnologías como Istio, Azure Monitor habilita la supervisión de aplicación instrumentada por automáticamente sin ninguna modificación en el código de la aplicación.

> [!NOTE]
> Esta es una de las muchas maneras de realizar la supervisión de aplicaciones en Kubernetes. También puede instrumentar cualquier aplicación hospedada en Kubernetes mediante el uso de la [SDK de Application Insights](../../azure-monitor/azure-monitor-app-hub.md) sin necesidad de una malla de servicio. Para supervisar Kubernetes sin instrumentar la aplicación con un SDK que puede usar el método.

## <a name="prerequisites"></a>Requisitos previos

- Un [clúster de Kubernetes](https://docs.microsoft.com/azure/aks/concepts-clusters-workloads).
- Acceso al clúster para ejecutar la consola *kubectl*.
- Un [recurso de Application Insights](create-new-resource.md)
- Tener una malla de servicio. Si el clúster no tiene Istio implementada, obtenga información sobre cómo [instalar y usar Istio en Azure Kubernetes Service](https://docs.microsoft.com/azure/aks/istio-install).

## <a name="capabilities"></a>Capacidades

Al usar cero aplicación instrumentación de supervisión de Kubernetes hospedadas aplicaciones, podrá usar:

- [Mapa de aplicación](../../azure-monitor/app/app-map.md)
- [Live Metrics Stream](../../azure-monitor/app/live-stream.md)
- [Paneles](../../azure-monitor/app/overview-dashboard.md)
- [Explorador de métricas](../../azure-monitor/platform/metrics-getting-started.md)
- [Seguimiento distribuido](../../azure-monitor/app/distributed-tracing.md)
- [Supervisión de transacciones to-end](../../azure-monitor/learn/tutorial-performance.md#identify-slow-server-operations)

## <a name="installation-steps"></a>Pasos de instalación

Para habilitar la solución, se deberá realizar los pasos siguientes:
- Implementar la aplicación (si aún no implementado).
- Asegúrese de que la aplicación forma parte de la malla de servicio.
- Observe la telemetría recopilada.

### <a name="configure-your-app-to-work-with-a-service-mesh"></a>Configuración de la aplicación para que funcione con una malla de servicio

Istio admite dos formas de [instrumentar un pod](https://istio.io/docs/setup/kubernetes/additional-setup/sidecar-injection/).
En la mayoría de los casos, es más fácil marcar el espacio de nombres de Kubernetes que contiene la aplicación con el *istio inyección* etiqueta:

```console
kubectl label namespace <my-app-namespace> istio-injection=enabled
```

> [!NOTE]
> Puesto que los datos del servicio malla ascensores desactivado la conexión, no podemos interceptamos el tráfico cifrado. Para el tráfico que no abandonen el clúster, use un protocolo sin cifrar (por ejemplo, HTTP). Para el tráfico externo que se debe cifrar, considere la posibilidad de [configurar terminación SSL](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls) en el controlador de entrada.

Aplicaciones que se ejecutan fuera de la malla de servicio no se ven afectadas.

### <a name="deploy-your-application"></a>Implementación de aplicación

- Implementar la aplicación en *my-app-namespace* espacio de nombres. Si la aplicación ya está implementada y ha seguido el método de inyección de sidecar automática que se ha descrito anteriormente, debe volver a crear los pods para asegurarse de que istio inserta su sidecar; ya sea inician una actualización gradual o eliminar pods individuales y esperar a que se vuelven a crear.
- Asegúrese de que la aplicación cumple con [Istio requisitos](https://istio.io/docs/setup/kubernetes/prepare/requirements/).

### <a name="deploy-zero-instrumentation-application-monitoring-for-kubernetes-hosted-apps"></a>Implementar cero aplicación instrumentación de supervisión de Kubernetes hospedadas aplicaciones

1. Descargue y extraiga una [ *adaptador Application Insights* versión](https://github.com/Microsoft/Application-Insights-Istio-Adapter/releases/).
2. Vaya a */src/kubernetes/* dentro de la carpeta de versión.
3. Editar *application-insights-istio-mixer-adapter-deployment.yaml*
    - Edite el valor de *ISTIO_MIXER_PLUGIN_AI_INSTRUMENTATIONKEY* variable de entorno para que contenga la clave de instrumentación del recurso de Application Insights en Azure portal para contener los datos de telemetría.
    - Si es necesario, modifique el valor de *ISTIO_MIXER_PLUGIN_WATCHLIST_NAMESPACES* variable de entorno para que contenga una lista separada por comas de los espacios de nombres para el que desea habilitar la supervisión. Déjelo en blanco para supervisar todos los espacios de nombres.
4. Aplicar *cada* archivo YAML se encuentra en *kubernetes/src/* ejecutando lo siguiente (todavía debe encontrarse dentro de */src/kubernetes/*):

   ```console
   kubectl apply -f .
   ```

### <a name="verify-deployment"></a>Comprobar la implementación

- Asegúrese de que se ha implementado el adaptador de Application Insights:

  ```console
  kubectl get pods -n istio-system -l "app=application-insights-istio-mixer-adapter"
  ```
> [!NOTE]
> En algunos casos, es necesaria ajustar una optimización. Para incluir o excluir la telemetría para un pod individual desde que se recopilan, utilice *appinsights/monitoring.enabled* etiqueta en ese pod. Esto tendrá prioridad sobre toda la configuración basada en el espacio de nombres. Establecer *appinsights/monitoring.enabled* a *true* para incluir el pod y a *false* para excluirlo.

### <a name="view-application-insights-telemetry"></a>Ver la telemetría de Application Insights

- Generar una solicitud de ejemplo en la aplicación para confirmar que la supervisión funcione correctamente.
- Dentro de 3 a 5 minutos, debería empezar a ver datos de telemetría aparecen en el portal de Azure. Asegúrese de consultar la *mapa de aplicación* sección del recurso de Application Insights en el Portal.

## <a name="troubleshooting"></a>solución de problemas

A continuación se espera que el flujo que se usará al telemetría no aparece en el portal de Azure como solución de problemas.

1. Asegúrese de la aplicación está bajo carga y es enviar o recibir las solicitudes de HTTP sin formato. Puesto que los datos de telemetría se elevación fuera de la conexión, no se admite el tráfico cifrado. Si no hay ninguna solicitud entrante o saliente, no habrá ninguna telemetría bien.
2. Asegúrese de que se proporciona la clave de instrumentación correcta en el *ISTIO_MIXER_PLUGIN_AI_INSTRUMENTATIONKEY* variable de entorno en *application-insights-istio-mixer-adapter-deployment.yaml*. La clave de instrumentación se encuentra en la *Introducción* pestaña del recurso de Application Insights en el portal de Azure.
3. Asegúrese de que se proporciona el espacio de nombres correcto de Kubernetes en la *ISTIO_MIXER_PLUGIN_WATCHLIST_NAMESPACES* variable de entorno en *application-insights-istio-mixer-adapter-deployment.yaml*. Déjelo en blanco para supervisar todos los espacios de nombres.
4. Asegúrese de que los pods de la aplicación se han insertado sidecar por Istio. Comprobar la existencia de sidecar de Istio en cada pod.

   ```console
   kubectl describe pod -n <my-app-namespace> <my-app-pod-name>
   ```
   Compruebe que hay un contenedor denominado *istio proxy* que se ejecutan en el pod.

5. Ver los seguimientos del adaptador de Application Insights.

   ```console
   kubectl get pods -n istio-system -l "app=application-insights-istio-mixer-adapter"
   kubectl logs -n istio-system application-insights-istio-mixer-adapter-<fill in from previous command output>
   ```

   El recuento de elementos de telemetría recibida se actualiza cada minuto. Si lo no aumente minuto a minuto - no hay datos de telemetría que envían al adaptador las Istio.
   Buscar los errores en el registro.
6. Si se ha establecido que *Application Insight para Kubernetes* adaptador no se introduce telemetría, compruebe los registros de Mixer de Istio para averiguar por qué no se envían datos al adaptador:

   ```console
   kubectl get pods -n istio-system -l "istio=mixer,app=telemetry"
   kubectl logs -n istio-system istio-telemetry-<fill in from previous command output> -c mixer
   ```
   Busque los errores, especialmente relacionados con las comunicaciones con *applicationinsightsadapter* adaptador.

## <a name="faq"></a>Preguntas más frecuentes

Para obtener la información más reciente para ver el progreso en este proyecto, visite la [el adaptador de Application Insights para GitHub del proyecto de Mixer Istio](https://github.com/Microsoft/Application-Insights-Istio-Adapter/blob/master/SETUP.md#faq).

## <a name="uninstall"></a>Desinstalar

Para desinstalar el producto, para *cada* archivo YAML se encuentra en *kubernetes/src/* ejecutar:

```console
kubectl delete -f <filename.yaml>
```


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo Azure Monitor y contenedores colaboran visita [Azure Monitor para la introducción a los contenedores](../../azure-monitor/insights/container-insights-overview.md)