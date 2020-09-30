---
title: Restauración de la base de datos de ejemplo AdventureWorks en SQL Managed Instance
description: Restauración de la base de datos de ejemplo AdventureWorks en SQL Managed Instance
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: bb5cc6fe3b0b6dd1250bbe241086fd4557866d5b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932197"
---
# <a name="restore-the-adventureworks-sample-database-into-sql-managed-instance---azure-arc"></a>Restauración de la base de datos de ejemplo AdventureWorks en SQL Managed Instance: Azure Arc

[AdventureWorks](/sql/samples/adventureworks-install-configure?view=sql-server-ver15&tabs=tsql&preserve-view=true) es una base de datos de ejemplo que contiene una base de datos OLTP que se usa a menudo en los tutoriales y ejemplos. Microsoft la proporciona y mantiene como parte del [repositorio de GitHub de ejemplos de SQL Server](https://github.com/microsoft/sql-server-samples/tree/master/samples/databases).

En este documento se describe un proceso sencillo para restaurar la base de datos de ejemplo AdventureWorks en la instancia de SQL Managed Instance: Azure Arc.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="download-the-adventureworks-backup-file"></a>Descarga del archivo de copia de seguridad de AdventureWorks

Descargue el archivo de copia de seguridad de AdventureWorks (.bak) en el contenedor de SQL Managed Instance. En este ejemplo, se utiliza el comando `kubectl exec` para ejecutar de forma remota un comando en el contenedor de SQL Managed Instance para descargar el archivo .bak en el contenedor. Descargue este archivo desde cualquier ubicación a la que tenga acceso mediante `wget` si tiene otros archivos de copia de seguridad de base de datos que desea extraer en el contenedor de SQL Managed Instance. Una vez que esté dentro del contenedor de SQL Managed Instance, es fácil de restaurar mediante el comando T-SQL estándar `RESTORE DATABASE`.

Ejecute un comando como este para descargar el archivo .bak. Reemplace el valor del nombre del pod y del espacio de nombres antes de ejecutarlo.
> [!NOTE]
>  El contenedor deberá tener conectividad a Internet en el puerto 443 para descargar el archivo de GitHub.

```console
kubectl exec <SQL pod name> -n <namespace name> -c mssql-miaa -- wget https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2019.bak -O /var/opt/mssql/data/AdventureWorks2019.bak
```

Ejemplo

```console
kubectl exec sqltest1-0 -n arc -c mssql-miaa -- wget https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2019.bak -O /var/opt/mssql/data/AdventureWorks2019.bak
```

## <a name="restore-the-adventureworks-database"></a>Restauración de la base de datos AdventureWorks

Del mismo modo, puede ejecutar un comando `kubectl` exec para usar la herramienta de la CLI `sqlcmd` que se incluye en el contenedor de SQL Managed Instance para ejecutar el comando T-SQL para restaurar la base de datos.

Ejecute un comando como este para restaurar la base de datos. Reemplace el valor del nombre del pod, la contraseña y el nombre del espacio de nombres antes de ejecutarlo.

```console
kubectl exec <SQL pod name> -n <namespace name> -c mssql-miaa -- /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P <password> -Q "RESTORE DATABASE AdventureWorks2019 FROM  DISK = N'/var/opt/mssql/data/AdventureWorks2019.bak' WITH MOVE 'AdventureWorks2017' TO '/var/opt/mssql/data/AdventureWorks2019.mdf', MOVE 'AdventureWorks2017_Log' TO '/var/opt/mssql/data/AdventureWorks2019_Log.ldf'"
```
Ejemplo

```console
kubectl exec sqltest1-0 -n arc -- -c mssql-miaa /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P MyPassword! -Q "RESTORE DATABASE AdventureWorks2019 FROM  DISK = N'/var/opt/mssql/data/AdventureWorks2019.bak' WITH MOVE 'AdventureWorks2017' TO '/var/opt/mssql/data/AdventureWorks2019.mdf', MOVE 'AdventureWorks2017_Log' TO '/var/opt/mssql/data/AdventureWorks2019_Log.ldf'"
```
