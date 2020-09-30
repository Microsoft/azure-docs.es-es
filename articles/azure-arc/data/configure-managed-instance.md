---
title: Configuración de una instancia de SQL Managed Instance habilitada para Azure Arc
description: Configuración de una instancia de SQL Managed Instance habilitada para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: f687923f1185d848c2c4f2f40d949c71a7743d65
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932553"
---
# <a name="configure-azure-arc-enabled-sql-managed-instance"></a>Configuración de una instancia de SQL Managed Instance habilitada para Azure Arc

En este artículo se explica cómo configurar la instancia de SQL Managed Instance habilitada para Azure Arc.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="configure-resources-for-azure-arc-enabled-sql-managed-instance"></a>Configuración de los recursos de SQL Managed Instance habilitada para Azure Arc

### <a name="configure-using-azdata"></a>Configuración mediante azdata

Puede editar la configuración de las instancias de SQL Managed Instance habilitada para Azure Arc con la CLI `azdata`. Ejecute el siguiente comando para ver las opciones de configuración. 

```
azdata arc sql mi edit --help
```

En el ejemplo siguiente se establecen las solicitudes y límites de CPU, núcleos y memoria.

```
azdata arc sql mi edit --cores-limit 4 --cores-request 2 --memory-limit 4Gi --memory-request 2Gi -n <NAME_OF_SQL_MI>
```

Para ver los cambios realizados en la instancia de SQL Managed Instance, puede usar los comandos siguientes para ver el archivo de configuración YAML:

```
azdata arc sql mi show -n <NAME_OF_SQL_MI>
```

## <a name="configure-server-options"></a>Configuración de las opciones del servidor

Puede configurar las opciones de configuración del servidor para la instancia de SQL Managed Instance habilitada para Azure Arc después de la creación. En este artículo se describe cómo configurar opciones como habilitar o deshabilitar el agente mssql o habilitar marcas de seguimiento específicas para escenarios de solución de problemas.

Haga lo siguiente para cambiar alguna de esas opciones:

1. Cree un archivo `mssql-custom.conf` personalizado que incluya la configuración de destino. En el ejemplo siguiente se habilita el Agente SQL y se habilita la marca de seguimiento 1204.:

   ```
   [sqlagent]
   enabled=true
   
   [traceflag]
   traceflag0 = 1204
   ```

1. Copie el archivo `mssql-custom.conf` en `/var/opt/mssql` en el contenedor `mssql-miaa` del pod `master-0`. Reemplace `<namespaceName>` por el nombre del clúster de macrodatos.

   ```bash
   kubectl cp mssql-custom.conf master-0:/var/opt/mssql/mssql-custom.conf -c mssql-server -n <namespaceName>
   ```

1. Reinicie la instancia de SQL Server.  Reemplace `<namespaceName>` por el nombre del clúster de macrodatos.

   ```bash
   kubectl exec -it master-0  -c mssql-server -n <namespaceName> -- /bin/bash
   supervisorctl restart mssql-server
   exit
   ```


**Restricciones conocidas**
- Los procedimientos anteriores requieren permisos de administrador de clústeres de Kubernetes.
- Esto está sujeto a cambios durante la versión preliminar.
