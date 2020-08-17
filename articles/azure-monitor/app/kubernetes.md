---
title: Uso de Application Insights para supervisar Azure Kubernetes Service (AKS) u otras aplicaciones hospedadas de Kubernetes - Azure Monitor | Microsoft Docs
description: Azure Monitor usa tecnología de malla de servicio, Istio, en el clúster de Kubernetes para proporcionar la supervisión de cualquier aplicación hospedada de Kubernetes. Esto le permite recopilar datos de telemetría de Application Insights pertenecientes a las solicitudes entrantes y salientes relacionadas con los pods que se ejecutan en el clúster.
ms.topic: conceptual
author: tokaplan
ms.author: alkaplan
ms.date: 04/25/2019
ms.openlocfilehash: 3cd43963175594fcdc1c3c67d6b2493ce1ccd313
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87321928"
---
# <a name="zero-instrumentation-application-monitoring-for-kubernetes-hosted-applications-with-istio---deprecated"></a>Supervisión de aplicaciones sin instrumentación para aplicaciones hospedadas de Kubernetes con Istio: EN DESUSO

> [!IMPORTANT]
> Esta función está actualmente en desuso y dejará de admitirse después del 1 de agosto de 2020.
> Actualmente, la supervisión sin código solo se puede habilitar para [Java a través de un agente independiente](./java-in-process-agent.md). Para otros lenguajes, use los SDK a fin de supervisar las aplicaciones en AKS: [ASP.Net Core](./asp-net-core.md), [ASP.Net](./asp-net.md), [Node.js](./nodejs.md), [JavaScript](./javascript.md) y [Python](./opencensus-python.md).

Azure Monitor ahora aprovecha la tecnología de malla de servicio en el clúster de Kubernetes para proporcionar la supervisión de aplicaciones lista para usar para cualquier aplicación hospedada de Kubernetes. Incluye características de Application Insights predeterminadas como el [Mapa de aplicación](./app-map.md) para modelar las dependencias, [Live Metrics Stream](./live-stream.md) para la supervisión en tiempo real, visualizaciones eficaces con el [panel predeterminado](./overview-dashboard.md), el [Explorador de métricas](../platform/metrics-getting-started.md) y [Workbooks](../platform/workbooks-overview.md). Esta característica ayudará a los usuarios a detectar cuellos de botella de rendimiento y zonas activas de error en todas las cargas de trabajo de Kubernetes dentro de un espacio de nombres de Kubernetes seleccionado. Al aprovechar las inversiones existentes en malla de servicio con tecnologías como Istio, Azure Monitor habilita la supervisión de aplicaciones instrumentada automáticamente sin ninguna modificación en el código de la aplicación.

> [!NOTE]
> Se trata de una de las muchas maneras de realizar la supervisión de aplicaciones en Kubernetes. También puede instrumentar cualquier aplicación hospedada en Kubernetes mediante la [SDK de Application Insights](../azure-monitor-app-hub.yml) sin necesidad de una malla de servicio. Para supervisar Kubernetes sin instrumentar la aplicación con una SDK, puede usar el método siguiente.

## <a name="prerequisites"></a>Prerrequisitos

- Un [clúster de Kubernetes](../../aks/concepts-clusters-workloads.md).
- Acceso de la consola al clúster para ejecutar *kubectl*.
- Un [recurso de Application Insights](create-new-resource.md).
- Tener una malla de servicio. Si el clúster no tiene Istio implementado, puede obtener información sobre cómo [instalar y usar Istio en Azure Kubernetes Service](../../aks/servicemesh-istio-install.md).

## <a name="capabilities"></a>Capacidades

Al usar la supervisión de aplicaciones sin instrumentación para las aplicaciones hospedadas de Kubernetes, podrá usar las siguientes características:

- [Mapa de aplicación](./app-map.md)
- [Métricas de Live Stream](./live-stream.md)
- [Paneles](./overview-dashboard.md)
- [Explorador de métricas](../platform/metrics-getting-started.md)
- [Seguimiento distribuido](./distributed-tracing.md)
- [Supervisión de transacciones de un extremo a otro](../learn/tutorial-performance.md#identify-slow-server-operations)

## <a name="installation-steps"></a>Pasos de instalación

Para habilitar la solución, se deberán realizar los pasos siguientes:
- Implementar la aplicación (si aún no está implementada).
- Asegurarse de que la aplicación forma parte de la malla de servicio.
- Observar los datos de telemetría recopilados.

### <a name="configure-your-app-to-work-with-a-service-mesh"></a>Configuración de la aplicación para que funcione con una malla de servicio

Istio admite dos formas de [instrumentar un pod](https://istio.io/docs/setup/kubernetes/additional-setup/sidecar-injection/).
En la mayoría de los casos, lo más fácil es marcar el espacio de nombres de Kubernetes que contiene la aplicación con la etiqueta *istio-injection*:

```console
kubectl label namespace <my-app-namespace> istio-injection=enabled
```

> [!NOTE]
> Puesto que la malla de servicio quita los datos de la conexión, no podemos interceptar el tráfico cifrado. Para el tráfico que no abandona el clúster, use un protocolo sin cifrar (por ejemplo, HTTP). Con el tráfico externo que debe cifrarse, considere la posibilidad de [configurar la terminación TLS](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls) en el controlador de entrada.

Las aplicaciones que se ejecutan fuera de la malla de servicio no se ven afectadas.

### <a name="deploy-your-application"></a>Implementación de aplicación

- Implemente la aplicación en el espacio de nombres *my-app-namespace*. Si la aplicación ya está implementada y ha seguido el método de inserción de sidecar automática que se ha descrito anteriormente, debe volver a crear los pods para asegurarse de que Istio inserta su sidecar; para ello, inicie una actualización gradual o elimine pods individuales y espere a que se vuelvan a crear.
- Asegúrese de que la aplicación cumple con los [requisitos de Istio](https://istio.io/docs/setup/kubernetes/prepare/requirements/).

### <a name="deploy-zero-instrumentation-application-monitoring-for-kubernetes-hosted-apps"></a>Implementación de la supervisión de aplicaciones sin instrumentación para aplicaciones hospedadas de Kubernetes

1. Descargue y extraiga una [versión del *adaptador de Application Insights*](https://github.com/Microsoft/Application-Insights-Istio-Adapter/releases/).
2. Vaya a */src/kubernetes/* dentro de la carpeta de la versión.
3. Editar *application-insights-istio-mixer-adapter-deployment.yaml*
    - Edite el valor de la variable de entorno *ISTIO_MIXER_PLUGIN_AI_INSTRUMENTATIONKEY* para que contenga la clave de instrumentación del recurso de Application Insights en Azure Portal para que contenga los datos de telemetría.
    - Si es necesario, modifique el valor de la variable de entorno *ISTIO_MIXER_PLUGIN_WATCHLIST_NAMESPACES* para que contenga una lista de espacios de nombres separados por comas para los que desea habilitar la supervisión. Déjelo en blanco para supervisar todos los espacios de nombres.
4. Aplique *cada* archivo YAML ubicado en *kubernetes/src/* ejecutando lo siguiente (debe encontrarse dentro de */src/kubernetes/* ):

   ```console
   kubectl apply -f .
   ```

### <a name="verify-deployment"></a>Comprobación de la implementación

- Asegúrese de que el adaptador de Application Insights se ha implementado:

  ```console
  kubectl get pods -n istio-system -l "app=application-insights-istio-mixer-adapter"
  ```
> [!NOTE]
> En algunos casos, es necesario optimizarlo. Para incluir o excluir la recopilación de datos de telemetría de un pod individual, utilice la etiqueta *appinsights/monitoring.enabled* en ese pod. Esto tendrá prioridad sobre toda la configuración basada en el espacio de nombres. Establezca *appinsights/monitoring.enabled* en *true* para incluir el pod y en *false* para excluirlo.

### <a name="view-application-insights-telemetry"></a>Visualización de los datos de telemetría de Application Insights

- Genere una solicitud de ejemplo en la aplicación para confirmar que la supervisión funcione correctamente.
- En un plazo de 3 a 5 minutos, debería empezar a ver los datos de telemetría en Azure Portal. Asegúrese de consultar la sección *Mapa de aplicación* del recurso de Application Insights en el Portal.

## <a name="troubleshooting"></a>Solución de problemas

A continuación se muestra el flujo de solución de problemas que deberá usar si los datos de telemetría no aparecen en Azure Portal como se espera.

1. Asegúrese de que la aplicación esté en carga y esté enviando o recibiendo solicitudes en HTTP sin formato. Puesto que los datos de telemetría se quitan de la conexión, no se admite el tráfico cifrado. Si no hay ninguna solicitud entrante o saliente, tampoco habrá datos de telemetría.
2. Asegúrese de que se ha proporcionado la clave de instrumentación correcta en la variable de entorno *ISTIO_MIXER_PLUGIN_AI_INSTRUMENTATIONKEY* en *application-insights-istio-mixer-adapter-deployment.yaml*. La clave de instrumentación se encuentra en la pestaña *Introducción* del recurso de Application Insights en Azure Portal.
3. Asegúrese de que se ha proporcionado el espacio de nombres de Kubernetes correcto en la variable de entorno *ISTIO_MIXER_PLUGIN_WATCHLIST_NAMESPACES* en *application-insights-istio-mixer-adapter-deployment.yaml*. Déjelo en blanco para supervisar todos los espacios de nombres.
4. Asegúrese de que Istio ha insertado en sidecar los pods de la aplicación. Compruebe que el sidecar de Istio exista en cada pod.

   ```console
   kubectl describe pod -n <my-app-namespace> <my-app-pod-name>
   ```
   Compruebe que en cada pod se ejecute un contenedor denominado *istio-proxy*.

5. Vea los seguimientos del adaptador de Application Insights.

   ```console
   kubectl get pods -n istio-system -l "app=application-insights-istio-mixer-adapter"
   kubectl logs -n istio-system application-insights-istio-mixer-adapter-<fill in from previous command output>
   ```

   El recuento de elementos de datos de telemetría se actualiza cada minuto. Si no aumenta minuto a minuto, Istio no está enviando datos de telemetría al adaptador.
   Busque errores en el registro.
6. Si se ha establecido que el adaptador de *Application Insights para Kubernetes* no recibe datos de telemetría, compruebe los registros Mixer de Istio para descubrir por qué no envía datos al adaptador:

   ```console
   kubectl get pods -n istio-system -l "istio=mixer,app=telemetry"
   kubectl logs -n istio-system istio-telemetry-<fill in from previous command output> -c mixer
   ```
   Busque errores, especialmente relacionados con las comunicaciones con el adaptador *applicationinsightsadapter*.

## <a name="faq"></a>Preguntas más frecuentes

Para obtener la información más reciente sobre el progreso en este proyecto, visite el [GitHub del proyecto del adaptador de Application Insights para Istio Mixer](https://github.com/Microsoft/Application-Insights-Istio-Adapter/blob/master/SETUP.md#faq).

## <a name="uninstall"></a>Desinstalación

Para desinstalar el producto, para *cada* archivo YAML ubicado en *kubernetes/src/* , ejecute:

```console
kubectl delete -f <filename.yaml>
```


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo Azure Monitor y los contenedores colaboran, visite [Introducción a Azure Monitor para contenedores](../insights/container-insights-overview.md)

