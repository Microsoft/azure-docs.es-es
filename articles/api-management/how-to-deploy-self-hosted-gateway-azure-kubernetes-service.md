---
title: Implementación de una puerta de enlace autohospedada en Azure Kubernetes Service | Microsoft Docs
description: Aprenda a implementar un componente de puerta de enlace autohospedada de Azure API Management en Azure Kubernetes Service
services: api-management
documentationcenter: ''
author: miaojiang
manager: gwallace
editor: ''
ms.service: api-management
ms.topic: article
ms.date: 04/26/2020
ms.author: apimpm
ms.openlocfilehash: 02962e9c5be2c4b73d121a53a7b595c573ad6cd0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87015228"
---
# <a name="deploy-to-azure-kubernetes-service"></a>Implementación en Azure Kubernetes Service

En este artículo se detallan los pasos para implementar un componente de puerta de enlace autohospedada de Azure API Management en [Azure Kubernetes Service](https://azure.microsoft.com/services/kubernetes-service/). Para implementar una puerta de enlace autohospedada en un clúster de Kubernetes, consulte este [documento](how-to-deploy-self-hosted-gateway-kubernetes.md).

## <a name="prerequisites"></a>Prerrequisitos

- [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)
- [Creación de un clúster de Azure Kubernetes](../aks/kubernetes-walkthrough-portal.md)
- [Aprovisione un recurso de puerta de enlace en la instancia de API Management](api-management-howto-provision-self-hosted-gateway.md).

## <a name="deploy-the-self-hosted-gateway-to-aks"></a>Implementación de la puerta de enlace autohospedada en AKS

1. Seleccione **Puertas de enlace** en **Deployment and infrastructure** (Implementación e infraestructura).
2. Seleccione el recurso de puerta de enlace autohospedada que desea implementar.
3. Seleccione **Implementación**.
4. Tenga en cuenta que en el cuadro de texto **Token** se generará automáticamente un nuevo token con los valores predeterminados de **Expiración** y **Clave secreta**. Ajuste uno o ambos si lo desea y seleccione **Generar** para crear un nuevo token.
5. Asegúrese de que **Kubernetes** esté seleccionado en **Scripts de implementación**.
6. Seleccione el vínculo al archivo **<gateway-name>.yml** junto a **Implementación** para descargar el archivo.
7. Ajuste las asignaciones de puerto y el nombre del contenedor en el archivo yml según sea necesario.
8. En función de su escenario, puede que tenga que cambiar el [tipo de servicio](../aks/concepts-network.md#services). El valor predeterminado es `NodePort`.
9. Seleccione el icono **copiar** situado en el extremo derecho del cuadro de texto **Implementar** para guardar el comando `kubectl` en el portapapeles.
10. Pegue el comando en la ventana de terminal (o comando). Tenga en cuenta que el comando espera que el archivo de entorno descargado esté presente en el directorio actual.
```console
    kubectl apply -f <gateway-name>.yaml
```
11. Ejecute el comando. El comando indica al clúster de AKS que ejecute el contenedor mediante la imagen de la puerta de enlace autohospedada descargada de Microsoft Container Registry y que configure el contenedor para exponer los puertos HTTP (8080) y HTTPS (443).
12. Ejecute el siguiente comando para comprobar que el pod de puerta de enlace se está ejecutando. Tenga en cuenta que el nombre del pod será diferente.
```console
kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
contoso-apim-gateway-59f5fb94c-s9stz   1/1       Running   0          1m
```
13. Ejecute el siguiente comando para comprobar que el servicio de puerta de enlace se está ejecutando. Tenga en cuenta el nombre del servicio y las direcciones IP serán diferentes.
```console
kubectl get services
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
contosogateway   NodePort    10.110.230.87   <none>        80:32504/TCP,443:30043/TCP   1m
```
14. Vuelva a Azure Portal y confirme que el nodo de puerta de enlace que acaba de implementar notifique que su estado sea correcto.

> [!TIP]
> Use el comando <code>kubectl logs <gateway-pod-name></code> para ver una instantánea del registro de la puerta de enlace autohospedada.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre la puerta de enlace autohospedada, consulte [Introducción a la puerta de enlace autohospedada de Azure API Management](self-hosted-gateway-overview.md)
* Más información acerca de [Azure Kubernetes Service](../aks/intro-kubernetes.md)
* Información sobre cómo [configurar y conservar los registros en la nube](how-to-configure-cloud-metrics-logs.md)
* * Información sobre cómo [configurar y conservar los registros localmente](how-to-configure-local-metrics-logs.md)
