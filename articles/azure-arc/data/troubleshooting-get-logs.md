---
title: Obtención de registros para solucionar problemas del controlador de datos habilitado para Azure Arc
description: Obtenga registros de servicio para solucionar problemas del controlador de datos habilitado para Azure Arc.
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 71c84b35c001be7fafdc2df53014050ae21dec63
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932228"
---
# <a name="get-azure-arc-enabled-data-services-logs"></a>Obtención de registros de los servicios de datos habilitados para Azure Arc

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Requisitos previos

Para recuperar los registros de los servicios de datos habilitados para Azure Arc, necesitará la herramienta CLI de datos de Azure. [Instrucciones de instalación](./install-client-tools.md)

Debe poder iniciar sesión en el servicio de controlador de servicios de datos habilitados para Azure Arc como administrador.

## <a name="get-azure-arc-enabled-data-services-logs"></a>Obtención de registros de los servicios de datos habilitados para Azure Arc

Puede obtener los registros de los servicios de datos habilitados para Azure Arc de todos los pods o de pods específicos para solucionar problemas.  Para ello, puede usar herramientas estándar de Kubernetes, como el comando `kubectl logs`. En este artículo, utilizará la herramienta CLI de datos de Azure, que facilita la obtención de todos los registros a la vez.

En primer lugar, asegúrese de que ha iniciado sesión en el controlador de datos.

```console
azdata login
```

A continuación, ejecute el comando siguiente para hacer un volcado de los registros:
```console
azdata arc dc debug copy-logs --namespace <namespace name> --exclude-dumps --skip-compress

#Example:
#azdata arc dc debug copy-logs --namespace arc --exclude-dumps --skip-compress
```

Los archivos de registro se crearán de manera predeterminada en el directorio de trabajo actual, en un subdirectorio denominado "logs".  Puede utilizar el parámetro `--target-folder` para que la salida de los archivos de registro llegue a otro directorio.

Puede comprimir los archivos si omite el parámetro `--skip-compress`.

Puede desencadenar e incluir volcados de memoria si omite el parámetro `--exclude-dumps`, pero no se recomienda hacerlo, a menos que el servicio de soporte técnico de Microsoft haya solicitado esos volcados.  El volcado de memoria requiere que el valor `allowDumps` del controlador de datos esté establecido en `true` cuando se cree el controlador de datos.

Si lo desea, puede filtrar para recopilar registros solo para un pod (`--pod`) o un contenedor (`--container`) específicos por nombre.

También puede optar por filtrar para recopilar los registros de un recurso personalizado específico pasando el parámetro `--resource-kind` y `--resource-name`.  El valor del parámetro `resource-kind` debe ser uno de los nombres de definición de recursos personalizados que puede recuperar el comando `kubectl get customresourcedefinition`.

```console
azdata arc dc debug copy-logs --target-folder <desired folder> --exclude-dumps --skip-compress -resource-kind <custom resource definition name> --resource-name <resource name> --namespace <namespace name>

#Example
#azdata arc dc debug copy-logs --target-folder C:\temp\logs --exclude-dumps --skip-compress --resource-kind postgresql-12 --resource-name pg1 --namespace arc
```

Ejemplo de jerarquía de carpetas.  Tenga en cuenta que la jerarquía de carpetas está organizada por nombre de pod, después por contenedor y después por jerarquía de directorios dentro del contenedor.

```console
<export directory>
├───debuglogs-arc-20200827-180403
│   ├───bootstrapper-vl8j2
│   │   └───bootstrapper
│   │       ├───apt
│   │       └───fsck
│   ├───control-j2dm5
│   │   ├───controller
│   │   │   └───controller
│   │   │       ├───2020-08-27
│   │   │       └───2020-08-28
│   │   └───fluentbit
│   │       ├───agent
│   │       ├───fluentbit
│   │       └───supervisor
│   │           └───log
│   ├───controldb-0
│   │   ├───fluentbit
│   │   │   ├───agent
│   │   │   ├───fluentbit
│   │   │   └───supervisor
│   │   │       └───log
│   │   └───mssql-server
│   │       ├───agent
│   │       ├───mssql
│   │       ├───mssql-server
│   │       └───supervisor
│   │           └───log
│   ├───controlwd-ln6j8
│   │   └───controlwatchdog
│   │       └───controlwatchdog
│   ├───logsdb-0
│   │   └───elasticsearch
│   │       ├───agent
│   │       ├───elasticsearch
│   │       ├───provisioner
│   │       └───supervisor
│   │           └───log
│   ├───logsui-7gg2d
│   │   └───kibana
│   │       ├───agent
│   │       ├───apt
│   │       ├───fsck
│   │       ├───kibana
│   │       └───supervisor
│   │           └───log
│   ├───metricsdb-0
│   │   └───influxdb
│   │       ├───agent
│   │       ├───influxdb
│   │       └───supervisor
│   │           └───log
│   ├───metricsdc-2f62t
│   │   └───telegraf
│   │       ├───agent
│   │       ├───apt
│   │       ├───fsck
│   │       ├───supervisor
│   │       │   └───log
│   │       └───telegraf
│   ├───metricsdc-jznd2
│   │   └───telegraf
│   │       ├───agent
│   │       ├───apt
│   │       ├───fsck
│   │       ├───supervisor
│   │       │   └───log
│   │       └───telegraf
│   ├───metricsdc-n5vnx
│   │   └───telegraf
│   │       ├───agent
│   │       ├───apt
│   │       ├───fsck
│   │       ├───supervisor
│   │       │   └───log
│   │       └───telegraf
│   ├───metricsui-h748h
│   │   └───grafana
│   │       ├───agent
│   │       ├───grafana
│   │       └───supervisor
│   │           └───log
│   └───mgmtproxy-r5zxs
│       ├───fluentbit
│       │   ├───agent
│       │   ├───fluentbit
│       │   └───supervisor
│       │       └───log
│       └───service-proxy
│           ├───agent
│           ├───nginx
│           └───supervisor
│               └───log
└───debuglogs-kube-system-20200827-180431
    ├───coredns-8bbb65c89-kklt7
    │   └───coredns
    ├───coredns-8bbb65c89-z2vvr
    │   └───coredns
    ├───coredns-autoscaler-5585bf8c9f-g52nt
    │   └───autoscaler
    ├───kube-proxy-5c9s2
    │   └───kube-proxy
    ├───kube-proxy-h6x56
    │   └───kube-proxy
    ├───kube-proxy-nd2b7
    │   └───kube-proxy
    ├───metrics-server-5f54b8994-vpm5r
    │   └───metrics-server
    └───tunnelfront-db87f4cd8-5xwxv
        ├───tunnel-front
        │   ├───apt
        │   └───journal
        └───tunnel-probe
            ├───apt
            ├───journal
            └───openvpn
```