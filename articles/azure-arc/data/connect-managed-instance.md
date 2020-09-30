---
title: Conexión a SQL Managed Instance habilitada para Azure Arc
description: Conexión a SQL Managed Instance habilitada para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 3277dc4d9c4485b117bfcfd1d6e130e7370cd8c2
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932244"
---
# <a name="connect-to-azure-arc-enabled-sql-managed-instance"></a>Conexión a SQL Managed Instance habilitada para Azure Arc

En este artículo se explica cómo puede conectarse a la instancia de SQL Managed Instance habilitada para Azure Arc. 

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="view-azure-arc-enabled-sql-managed-instances"></a>Visualización de instancias de SQL Managed Instance habilitada para Azure Arc

Para ver la instancia de SQL Managed Instance habilitada para Azure Arc y los puntos de conexión externos, use el siguiente comando:

```console
azdata arc sql mi list
```

La salida debe ser similar a la que se muestra a continuación:

```console
Name    Replicas    ExternalEndpoint    State
------  ----------  ----------------  -------
sqldemo 1/1         10.240.0.4:32023  Ready
```

Si usa AKS, kubeadm, OpenShift, etc., puede copiar el número de puerto y la dirección IP externa desde aquí y conectarse a él mediante su herramienta favorita para conectarse a una instancia de SQL Server o Azure SQL, como Azure Data Studio o SQL Server Management Studio.  Sin embargo, si usa la máquina virtual de inicio rápido, consulte a continuación para obtener información especial sobre cómo conectarse a esa máquina virtual desde fuera de Azure. 

> [!NOTE]
> Las directivas corporativas pueden bloquear el acceso a la dirección IP y el puerto, especialmente si se ha creado en la nube pública.

## <a name="connect"></a>Conectar 

Conexión con Azure Data Studio, SQL Server Management Studio o SQLCMD

Abra Azure Data Studio y conéctese a la instancia con la dirección IP y el número de puerto del punto de conexión externo anteriores. Si usa una máquina virtual de Azure, necesitará la dirección IP _pública_, que se puede identificar según se describe en [Nota especial sobre las implementaciones de máquinas virtuales de Azure](#special-note-about-azure-virtual-machine-deployments).

Por ejemplo:

- Servidor:  52.229.9.30,30913
- Nombre de usuario: sa
- Contraseña: contraseña de SQL que especificó en el momento del aprovisionamiento

> [!NOTE]
> Puede usar Azure Data Studio para [ver los paneles de SQL Managed Instance](azure-data-studio-dashboards.md#view-the-sql-managed-instance-dashboards).

Para conectarse con SQLCMD en Linux o Windows, puede usar un comando como este. Escriba la contraseña de SQL cuando se le solicite.

```bash
sqlcmd -S 52.229.9.30,30913 -U sa
```

## <a name="special-note-about-azure-virtual-machine-deployments"></a>Nota especial sobre las implementaciones de máquinas virtuales de Azure

Si usa una máquina virtual de Azure, la dirección IP del punto de conexión no mostrará la dirección IP pública. Para buscar la dirección IP externa, use el siguiente comando:

```console
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

Después, puede combinar la dirección IP pública con el puerto para establecer la conexión.

Es posible que también tenga que exponer el puerto de la instancia de SQL mediante el grupo de seguridad de red (NSG). Para permitir el tráfico en el NSG, tendrá que agregar una regla mediante el comando siguiente.

Para establecer una regla, deberá conocer el nombre del NSG, el cual puede encontrar mediante el comando siguiente:

```console
az network nsg list -g azurearcvm-rg --query "[].{NSGName:name}" -o table
```

Una vez que tenga el nombre del NSG, puede agregar una regla de firewall mediante el comando siguiente. En estos valores de ejemplo se crea una regla de NSG para el puerto 30913 y se permite la conexión desde **cualquier** dirección IP de origen.  No es un procedimiento recomendado de seguridad.  Puede conseguir un bloqueo mejor si especifica un valor para -source-address-prefixes específico de la dirección IP del cliente o un intervalo de direcciones IP que abarque las direcciones IP del equipo o la organización.

Reemplace el valor del parámetro `--destination-port-ranges` siguiente por el número de puerto que recibió del comando `azdata sql instance list` anterior.

```console
az network nsg rule create -n db_port --destination-port-ranges 30913 --source-address-prefixes '*' --nsg-name azurearcvmNSG --priority 500 -g azurearcvm-rg --access Allow --description 'Allow port through for db access' --destination-address-prefixes '*' --direction Inbound --protocol Tcp --source-port-ranges '*'
```

## <a name="next-steps"></a>Pasos siguientes

- [Visualización de los panales de SQL Managed Instance](azure-data-studio-dashboards.md#view-the-sql-managed-instance-dashboards)
- [Visualización de la instancia de SQL Managed Instance en Azure Portal](view-arc-data-services-inventory-in-azure-portal.md)
