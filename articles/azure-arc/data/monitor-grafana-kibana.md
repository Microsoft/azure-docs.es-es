---
title: Visualización de registros y métricas mediante Kibana y Grafana
description: Visualización de registros y métricas mediante Kibana y Grafana
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: d876862d8f41ab8df646bef051629fd45c4d4601
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932327"
---
# <a name="view-logs-and-metrics-using-kibana-and-grafana"></a>Visualización de registros y métricas mediante Kibana y Grafana

Los paneles web Kibana y Grafana se proporcionan para ofrecer información y claridad a los espacios de nombres de Kubernetes que usan los servicios de datos habilitados para Azure Arc.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="retrieve-the-ip-address-of-your-cluster"></a>Recuperación de la dirección IP del clúster

Para acceder a los paneles, tendrá que recuperar la dirección IP del clúster. El método para recuperar la dirección IP correcta varía en función de cómo se haya elegido implementar Kubernetes. Recorra las opciones siguientes para encontrar la adecuada.

### <a name="azure-virtual-machine"></a>Máquina virtual de Azure

Use el comando siguiente para recuperar la IP pública:

```console
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

### <a name="kubeadm-cluster"></a>Clúster de kubeadm

Use el comando siguiente para recuperar la dirección IP del clúster:

```console
kubectl cluster-info
```


### <a name="aks-or-other-load-balanced-cluster"></a>AKS u otro clúster con carga equilibrada

Para supervisar el entorno en AKS u otro clúster con carga equilibrada, debe obtener la dirección IP del servicio de proxy de administración. Use este comando para recuperar la dirección **IP externa**:

```console
kubectl get svc mgmtproxy-svc-external -n <namespace>

#Example:
#kubectl get svc mgmtproxy-svc-external -n arc
NAME                     TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)           AGE
mgmtproxy-svc-external   LoadBalancer   10.0.186.28   52.152.148.25   30777:30849/TCP   19h
```

## <a name="additional-firewall-configuration"></a>Configuración adicional del firewall

Es posible que tenga que abrir puertos en el firewall para acceder a los puntos de conexión Kibana y Grafana.

A continuación, se muestra un ejemplo del procedimiento para una máquina virtual de Azure. Tendrá que hacer esto si se ha implementado Kubernetes mediante el script.

En los pasos siguientes, se resalta cómo crear una regla de NSG para los puntos de conexión Kibana y Grafana:

### <a name="find-the-name-of-the-nsg"></a>Buscar el nombre del NSG

```console
az network nsg list -g azurearcvm-rg --query "[].{NSGName:name}" -o table
```

### <a name="add-the-nsg-rule"></a>Agregar la regla de NSG

Una vez que tenga el nombre del NSG, se puede agregar una regla usando el comando siguiente:

```console
az network nsg rule create -n ports_30777 --nsg-name azurearcvmNSG --priority 600 -g azurearcvm-rg --access Allow --description 'Allow Kibana and Grafana ports' --destination-address-prefixes '*' --destination-port-ranges 30777 --direction Inbound --protocol Tcp --source-address-prefixes '*' --source-port-ranges '*'
```

## <a name="monitor-azure-sql-managed-instances-on-azure-arc"></a>Supervisión de instancias administradas de Azure SQL en Azure Arc

Ahora que tiene la dirección IP y expone los puertos, debe ser capaz de acceder a Grafana y Kibana.

> [!NOTE]
>  Cuando se le pida que escriba un nombre de usuario y una contraseña, escriba el nombre de usuario y la contraseña que proporcionó en el momento en que creó el controlador de datos de Azure Arc.

> [!NOTE]
>  Se le mostrará una advertencia de certificado porque los certificados usados en la versión preliminar son certificados autofirmados.

Use el patrón siguiente de dirección URL para acceder a los paneles de registro y supervisión de la instancia administrada de Azure SQL:

```html
https://<external-ip-from-above>:30777/grafana
https://<external-ip-from-above>:30777/kibana
```

Los paneles pertinentes son los siguientes:

* "Métricas de la instancia administrada de Azure SQL"
* "Métricas del nodo de host"
* "Métricas de los pods de host"

## <a name="monitor-azure-database-for-postgresql-hyperscale---azure-arc"></a>Supervisión de Hiperescala de Azure Database for PostgreSQL: Azure Arc

Use el patrón siguiente de dirección URL para acceder a los paneles de registro y supervisión de Hiperescala de PostgreSQL:

```html
https://<external-ip-from-above>:30777/grafana
https://<external-ip-from-above>:30777/kibana
```

Los paneles pertinentes son los siguientes:

* "Métricas de Postgres"
* "Métricas de la tabla Postgres"
* "Métricas del nodo de host"
* "Métricas de los pods de host"

## <a name="next-steps"></a>Pasos siguientes
- Intente [cargar métricas y registros en Azure Monitor](upload-metrics-and-logs-to-azure-monitor.md)
- Obtenga información acerca de Grafana:
   - [Introducción](https://grafana.com/docs/grafana/latest/getting-started/getting-started)
   - [Aspectos básicos de Grafana](https://grafana.com/tutorials/grafana-fundamentals/#1)
   - [Tutoriales de Grafana](https://grafana.com/tutorials/grafana-fundamentals/#1)
- Obtenga información acerca de Kibana:
   - [Introducción](https://www.elastic.co/webinars/getting-started-kibana?baymax=default&elektra=docs&storm=top-video)
   - [Guía de Kibana](https://www.elastic.co/guide/en/kibana/current/index.html)
   - [Introducción al panel de exploraciones en profundidad con visualizaciones de datos en Kibana](https://www.elastic.co/webinars/dashboard-drilldowns-with-data-visualizations-in-kibana/)
   - [Cómo crear paneles de Kibana](https://www.elastic.co/webinars/how-to-build-kibana-dashboards/)

