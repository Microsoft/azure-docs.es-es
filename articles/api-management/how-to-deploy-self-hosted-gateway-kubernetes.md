---
title: Implementación de una puerta de enlace autohospedada en Kubernetes | Microsoft Docs
description: Aprenda a implementar un componente de puerta de enlace autohospedada de Azure API Management en Kubernetes
services: api-management
author: vladvino
manager: gwallace
ms.service: api-management
ms.workload: mobile
ms.topic: article
ms.author: apimpm
ms.date: 04/23/2020
ms.openlocfilehash: abcda4ea4b14f058325318661daa574494268780
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87056375"
---
# <a name="deploy-a-self-hosted-gateway-to-kubernetes"></a>Implementación de una puerta de enlace autohospedada en Kubernetes

En este artículo se describen los pasos para implementar un componente de puerta de enlace autohospedada de Azure API Management en un clúster de Kubernetes.

## <a name="prerequisites"></a>Prerrequisitos

- Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
- Cree un clúster de Kubernetes.
   > [!TIP]
   > Los [clústeres de un solo nodo](https://kubernetes.io/docs/setup/#learning-environment) funcionan bien con fines de desarrollo y evaluación. Use clústeres de varios nodos [certificados de Kubernetes](https://kubernetes.io/partners/#conformance) en un entorno local o en la nube para cargas de trabajo de producción.
- [Aprovisione un recurso de puerta de enlace autohospedada en la instancia de API Management](api-management-howto-provision-self-hosted-gateway.md).

## <a name="deploy-to-kubernetes"></a>Implementación en Kubernetes

1. Seleccione **Puertas de enlace** en **Deployment and infrastructure** (Implementación e infraestructura).
2. Seleccione el recurso de puerta de enlace autohospedada que quiere implementar.
3. Seleccione **Implementación**.
4. Se ha generado automáticamente un token de acceso en el cuadro de texto **Token** según los valores predeterminados de **Expiración** y **Clave secreta**. Si es necesario, elija valores en uno o ambos controles para generar un nuevo token.
5. Seleccione la pestaña **Kubernetes** en **Scripts de implementación**.
6. Seleccione el vínculo del archivo **\<gateway-name\>.yml** y descargue el archivo YAML.
7. Seleccione el icono **copiar** situado en la esquina inferior derecha del cuadro de texto **Implementar** para guardar los comandos `kubectl` en el Portapapeles.
8. Pegue los comandos en la ventana de terminal o comando. El primer comando crea un secreto de Kubernetes que contiene el token de acceso generado en el paso 4. El segundo comando aplica el archivo de configuración descargado en el paso 6 al clúster de Kubernetes y espera que el archivo esté en el directorio actual.
9. Ejecute los comandos para crear los objetos de Kubernetes necesarios en el [espacio de nombres predeterminado](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) e inicie los pods de la puerta de enlace autohospedada desde la [imagen de contenedor](https://aka.ms/apim/sputnik/dhub) descargada de Microsoft Azure Container Registry.
10. Ejecute el siguiente comando para comprobar si la implementación se realizó correctamente. Tenga en cuenta que la creación de todos los objetos y la inicialización de los pods pueden tardar un poco.
    ```console
    kubectl get deployments
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    <gateway-name>   1/1     1            1           18s
    ```
11. Ejecute el siguiente comando para comprobar si el servicio se creó correctamente. Tenga en cuenta que las direcciones IP y los puertos del servicio serán diferentes.
    ```console
    kubectl get services
    NAME             TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
    <gateway-name>   LoadBalancer   10.99.236.168   <pending>     80:31620/TCP,443:30456/TCP   9m1s
    ```
12. Vuelva a Azure Portal y seleccione **Información general**.
13. Confirme que **Estado** muestra una marca de verificación verde seguida de un recuento de nodos que coincide con el número de réplicas especificado en el archivo YAML. Este estado significa que los pods de puerta de enlace autohospedados implementados se comunican correctamente con el servicio de API Management y tienen un "latido" normal.

    ![Estado de la puerta de enlace](media/how-to-deploy-self-hosted-gateway-kubernetes/status.png)

> [!TIP]
> Ejecute el comando <code>kubectl logs deployment/<gateway-name></code> para ver los registros de un pod seleccionado aleatoriamente si hay más de uno.
> Ejecute <code>kubectl logs -h</code> para un conjunto completo de opciones de comando; por ejemplo, para ver los registros de un pod o contenedor específico.

## <a name="production-deployment-considerations"></a>Consideraciones de implementación en producción

### <a name="access-token"></a>Access token
Sin un token de acceso válido, una puerta de enlace autohospedada no puede tener acceso a los datos de configuración del punto de conexión del servicio API Management asociado ni descargarlos. El token de acceso puede ser válido durante 30 días como máximo. Se debe regenerar y el clúster se debe configurar con un token nuevo, ya sea manualmente o mediante automatización antes de que expire. 

Al automatizar la actualización de tokens, use esta [operación de la API de administración](/rest/api/apimanagement/2019-12-01/gateway/generatetoken) para generar un nuevo token. Para obtener información sobre la administración de secretos de Kubernetes, consulte el [sitio web de Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret).

### <a name="namespace"></a>Espacio de nombres
Los espacios de nombres de [Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) ayudan a dividir un solo clúster entre varios equipos, proyectos o aplicaciones. Los espacios de nombres proporcionan un ámbito para los recursos y los nombres. Pueden asociarse con una cuota de recursos y directivas de control de acceso.

Azure Portal proporciona comandos para crear recursos de la puerta de enlace autohospedada en el espacio de nombres **predeterminado**. Este espacio de nombres se crea automáticamente, existe en todos los clústeres y no se puede eliminar.
Considere la posibilidad de [crear e implementar](https://kubernetesbyexample.com/ns/) una puerta de enlace autohospedada en un espacio de nombres independiente en producción.

### <a name="number-of-replicas"></a>Número de réplicas
El número mínimo de réplicas adecuado en producción es dos.

De forma predeterminada, la puerta de enlace autohospedada se implementa con una [estrategia](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy) de implementación **RollingUpdate**. Revise los valores predeterminados y considere la posibilidad de establecer de forma explícita los campos [maxUnavailable](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable)y [maxSurge](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge), especialmente si usa un número elevado de réplicas.

### <a name="container-resources"></a>Recursos de contenedor
De forma predeterminada, el archivo YAML proporcionado en Azure Portal no especifica solicitudes de recursos de contenedor.

No es posible predecir de forma confiable y recomendar la cantidad de recursos de memoria y CPU por contenedor y el número de réplicas necesarias para admitir una carga de trabajo específica. Hay muchos factores en juego; por ejemplo:

- Hardware específico en el que se ejecuta el clúster
- Presencia y tipo de virtualización
- Número y tasa de conexiones de cliente simultáneas
- Velocidad de solicitudes
- Tipo y número de directivas configuradas
- Tamaño de carga útil y si las cargas útiles se almacenan en búfer o se transmiten en secuencias
- Latencia del servicio back-end

Se recomienda establecer las solicitudes de recursos en dos núcleos y 2 GiB como punto de partida. Realice una prueba de carga y escale o reduzca vertical u horizontalmente en función de los resultados.

### <a name="container-image-tag"></a>Etiqueta de la imagen de contenedor
El archivo YAML proporcionado en Azure Portal usa la etiqueta **más reciente**. Esta etiqueta siempre hace referencia a la versión más reciente de la imagen de contenedor de la puerta de enlace autohospedada.

Considere la posibilidad de usar una etiqueta de versión específica en producción para evitar una actualización involuntaria a una versión más reciente.

Puede [descargar una lista completa de etiquetas disponibles](https://mcr.microsoft.com/v2/azure-api-management/gateway/tags/list).

### <a name="dns-policy"></a>Directiva de DNS
La resolución de nombres DNS desempeña un rol fundamental en la capacidad de una puerta de enlace autohospedada para conectarse a las dependencias en Azure y enviar llamadas API a los servicios de back-end.

El archivo YAML proporcionado en Azure Portal aplica la directiva [ClusterFirst](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pod-s-dns-policy) predeterminada. Esta directiva hace que las solicitudes de resolución de nombres que el DNS del clúster no resuelve se reenvíen al servidor DNS ascendente heredado del nodo.

Para obtener información acerca de la resolución de nombres en Kubernetes, consulte el [sitio web de Kubernetes](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service). Considere la posibilidad de personalizar la [directiva de DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pod-s-dns-policy) o la [configuración de DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pod-s-dns-config) según corresponda para su configuración.

### <a name="custom-domain-names-and-ssl-certificates"></a>Nombres de dominio personalizados y certificados SSL

Si usa nombres de dominio personalizados para los puntos de conexión de API Management, especialmente si usa un nombre de dominio personalizado para el punto de conexión de administración, es posible que tenga que actualizar el valor de `config.service.endpoint` en el archivo **\<gateway-name\>.yaml** para reemplazar el nombre de dominio predeterminado por el nombre de dominio personalizado. Asegúrese de que se puede acceder al punto de conexión de administración desde el pod de la puerta de enlace autohospedada del clúster de Kubernetes.

En este escenario, si el certificado SSL que usa el punto de conexión de administración no está firmado por una entidad de certificación conocida, debe asegurarse de que el pod de la puerta de enlace autohospedada confíe en el certificado de CA.

### <a name="configuration-backup"></a>Copia de seguridad de configuración
Para obtener información sobre el comportamiento de la puerta de enlace autohospedada en caso de interrupción temporal de la conectividad de Azure, consulte [Información general de la puerta de enlace autohospedada](self-hosted-gateway-overview.md#connectivity-to-azure).

Configure un volumen de almacenamiento local para el contenedor de la puerta de enlace autohospedada, de modo que pueda conservar una copia de seguridad de la última configuración descargada. Si la conectividad está inactiva, el volumen de almacenamiento puede usar la copia de seguridad al reiniciar. La ruta de montaje del volumen debe ser <code>/apim/config</code>. Vea un ejemplo en [GitHub](https://github.com/Azure/api-management-self-hosted-gateway/blob/master/examples/self-hosted-gateway-with-configuration-backup.yaml).
Para obtener información acerca del almacenamiento en Kubernetes, consulte el [sitio web de Kubernetes](https://kubernetes.io/docs/concepts/storage/volumes/).

### <a name="local-logs-and-metrics"></a>Registros y métricas locales
La puerta de enlace autohospedada envía telemetría a [Azure Monitor](api-management-howto-use-azure-monitor.md) y [Azure Application Insights](api-management-howto-app-insights.md) según los valores de configuración del servicio API Management asociado.
Cuando la [conectividad a Azure](self-hosted-gateway-overview.md#connectivity-to-azure) se pierde temporalmente, se interrumpe el flujo de telemetría a Azure y se pierden los datos mientras dure la interrupción.
Considere la posibilidad de [configurar la supervisión local](how-to-configure-local-metrics-logs.md) para garantizar la capacidad de observar el tráfico de la API y evitar la pérdida de telemetría durante las interrupciones de la conectividad de Azure.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre la puerta de enlace autohospedada, consulte [Introducción a la puerta de enlace autohospedada](self-hosted-gateway-overview.md).
