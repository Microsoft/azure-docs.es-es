---
title: Creación de una Instancia administrada de Azure SQL en Azure Arc
description: Creación de una Instancia administrada de Azure SQL en Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 2a931b1a3c3f88af1abec4fd1810aae09c849c48
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932628"
---
# <a name="create-an-azure-sql-managed-instance-on-azure-arc"></a>Creación de una Instancia administrada de Azure SQL en Azure Arc

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="login-to-the-azure-arc-data-controller"></a>Inicio de sesión en el controlador de datos de Azure Arc

Antes de poder crear una instancia, inicie la sesión en el controlador de datos de Azure Arc si aún no lo ha hecho.

```console
azdata login
```

Después, se le pedirá el nombre de usuario, la contraseña y el espacio de nombres del sistema.  

```console
Username: arcadmin
Password:
Namespace: arc
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="create-an-azure-sql-managed-instance"></a>Creación de una instancia administrada de Azure SQL

Para ver las opciones de creación disponibles para una instancia administrada de SQL, use el siguiente comando:
```console
azdata arc sql mi create --help
```

Para crear una instancia administrada de SQL, use el siguiente comando:

```console
azdata arc sql mi create -n <instanceName> --storage-class-data <storage class> --storage-class-logs <storage class>
```

Ejemplo:

```console
azdata arc sql mi create -n sqldemo --storage-class-data managed-premium --storage-class-logs managed-premium
```
> [!NOTE]
>  Los nombres deben tener menos de 13 caracteres, y ajustarse a las [convenciones de nomenclatura de DNS](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names)
>
>  Al especificar la asignación de memoria y la asignación de núcleo virtual, use esta fórmula para asegurarse de que la creación se realice correctamente; por cada núcleo virtual necesita al menos 4 GB de RAM de capacidad disponible en el nodo Kubernetes, donde se ejecutará el pod de instancia de SQL Managed Instance.
>
>  Al crear una instancia de SQL, no use mayúsculas en el nombre si está aprovisionando en Azure
>
>  Para enumerar las clases de almacenamiento disponibles en el clúster de Kubernetes, ejecute `kubectl get storageclass` 


> [!NOTE]
> Si quiere automatizar la creación de instancias de SQL y evitar el aviso interactivo para la contraseña del administrador, puede establecer las variables de entorno `AZDATA_USERNAME` y `AZDATA_PASSWORD` en el nombre de usuario y la contraseña que quiera antes de ejecutar el comando `azdata arc sql mi create`.
> 
>  Si creó el controlador de datos con AZDATA_USERNAME y AZDATA_PASSWORD en la misma sesión de terminal, los valores de AZDATA_USERNAME y AZDATA_PASSWORD también se usarán para crear la instancia de SQL Managed Instance.

> [!NOTE]
> La creación de Azure SQL Managed Instance no registrará los recursos en Azure. Los pasos para registrar el recurso se encuentran en los siguientes artículos: 
> - [Visualización de registros y métricas mediante Kibana y Grafana](monitor-grafana-kibana.md)
> - [Carga de datos de facturación en Azure y visualización en Azure Portal](view-billing-data-in-azure.md) 


## <a name="view-instance-on-azure-arc"></a>Visualización de la instancia en Azure Arc

Para ver la instancia, use el siguiente comando:

```console
azdata arc sql mi list
```

La salida debe ser similar a la que se muestra a continuación:

```console
Name    Replicas    ServerEndpoint    State
------  ----------  ----------------  -------
sqldemo 1/1         10.240.0.4:32023  Ready
```

Si usa AKS o `kubeadm` u OpenShift, etc., puede copiar el número de puerto y la dirección IP externa desde aquí y conectarse a él mediante su herramienta favorita para conectarse a una instancia de SQL Server o Azure SQL, como Azure Data Studio o SQL Server Management Studio. Sin embargo, si está usando la máquina virtual de inicio rápido, consulte el artículo [Conexión a SQL Managed Instance habilitada para Azure Arc](connect-managed-instance.md) para obtener instrucciones especiales.


## <a name="next-steps"></a>Pasos siguientes
- [Conexión a SQL Managed Instance habilitada para Azure Arc](connect-managed-instance.md)
- [Registro de su instancia con Azure y carga de métricas y registros sobre la instancia](upload-metrics-and-logs-to-azure-monitor.md)
- [Implementación de Azure SQL Managed Instance con Azure Data Studio](create-sql-managed-instance-azure-data-studio.md)
