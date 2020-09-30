---
title: Obtención de puntos de conexión y cadenas de conexión de formularios para el grupo de servidores de hiperescala de PostgreSQL habilitado para Arc
titleSuffix: Azure Arc enabled data services
description: Obtención de puntos de conexión y cadenas de conexión de formularios para el grupo de servidores de hiperescala de PostgreSQL habilitado para Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 93b0547ba25f6534593a0a016ebfa5cbe4d2be2e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932608"
---
# <a name="get-connection-endpoints-and-form-connection-strings-for-your-arc-enabled-postgresql-hyperscale-server-group"></a>Obtención de puntos de conexión y cadenas de conexión de formularios para el grupo de servidores de hiperescala de PostgreSQL habilitado para Arc

En este artículo se explica cómo se pueden recuperar los puntos de conexión del grupo de servidores y cómo se forman las cadenas de conexión que se utilizarán con las aplicaciones o herramientas.


[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="get-connection-end-points"></a>Obtención de puntos de conexión:

### <a name="from-cli-with-azdata"></a>Desde la CLI con azdata
#### <a name="1-connect-to-your-arc-data-controller"></a>1. Conéctese al controlador de datos de Arc:
- Si ya tiene una sesión abierta en el host del controlador de datos de Arc: Ejecute el siguiente comando:
```console
azdata login
```

- Si no tiene una sesión abierta en el host del controlador de datos de Arc: ejecute el siguiente comando 
```console
azdata login --endpoint https://<external IP address of host/data controller>:30080
```

#### <a name="2-show-the-connection-endpoints"></a>2. Muestre los puntos de conexión
Ejecute el siguiente comando:
```console
azdata arc postgres endpoint list -n <server group name>
```
Devuelve un resultado como el siguiente:
```console
[
  {
    "Description": "PostgreSQL Instance",
    "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
  },
  {
    "Description": "Log Search Dashboard",
    "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
  },
  {
    "Description": "Metrics Dashboard",
    "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
  }
]
```
Use estos puntos de conexión para:
- Formar las cadenas de conexión y conectarse con las aplicaciones o herramientas de cliente
- Acceder a los paneles Grafana y Kibana desde el explorador

Por ejemplo, puede usar el punto final denominado _Instancia de PostgreSQL_ para conectarse con psql al grupo de servidores. Por ejemplo:
```console
psql postgresql://postgres:MyPassworkd@12.345.123.456:1234
psql (10.14 (Ubuntu 10.14-0ubuntu0.18.04.1), server 12.4 (Ubuntu 12.4-1.pgdg16.04+1))
WARNING: psql major version 10, server major version 12.
         Some psql features might not work.
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=#
```
> [!NOTE]
>
> - La contraseña de usuario _postgresql_ indicada en el punto de conexión denominado "_instancia de PostgreSQL_ es la contraseña que eligió al crear el grupo de servidores.
> - Acerca de azdata: la concesión asociada a la conexión dura unas 10 horas. Después, debe volver a conectarse. Si la concesión ha expirado, recibirá el siguiente mensaje de error al intentar ejecutar un comando con azdata (distinto del inicio de sesión de azdata): _ERROR: (401)_ 
> _Motivo: Encabezados de respuesta_
> _HTTP no autorizados: HTTPHeaderDict({'Date': 'Sun, 06 Sep 2020 16:58:38 GMT', 'Content-Length': '0', 'WWW-Authenticate': '_ 
> _Basic realm="Se requieren_ las credenciales para el inicio de sesión", Bearer error="invalid_token", error_description="El token ha caducado"'})_ Cuando ocurre esto, debe volver a conectarse con azdata, tal como se ha explicado anteriormente.

## <a name="from-cli-with-kubectl"></a>Desde la CLI con kubectl
- Si el grupo de servidores es de la versión 12 de Postgres (valor predeterminado), el siguiente comando:
```console
kubectl get postgresql-12/<server group name>
```
- Si el grupo de servidores es de la versión 11 de Postgres, el siguiente comando:
```console
kubectl get postgresql-11/<server group name>
```

Estos comandos producirán una salida como la siguiente. Puede usar esa información para formar las cadenas de conexión:
```console
NAME         STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
postgres01   Ready   3/3          123.456.789.4:31066      5d20h
``` 


## <a name="form-connection-strings"></a>Cadenas de conexión de formulario:
Use la siguiente tabla de plantillas de cadenas de conexiones para el grupo de servidores. Después, puede copiarlas, pegarlas y personalizarlas según sea necesario:

### <a name="adonet"></a>ADO.NET

```ado.net
Server=192.168.1.121;Database=postgres;Port=24276;User Id=postgres;Password={your_password_here};Ssl Mode=Require;`
```

### <a name="c-libpq"></a>C++ (libpq)

```cpp
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
```

### <a name="jdbc"></a>JDBC

```jdbc
jdbc:postgresql://192.168.1.121:24276/postgres?user=postgres&password={your_password_here}&sslmode=require
```

### <a name="nodejs"></a>Node.js

```node.js
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
```

### <a name="php"></a>PHP

```php
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
```

### <a name="psql"></a>psql

```psql
psql "host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require"
```

### <a name="python"></a>Python

```python
dbname='postgres' user='postgres' host='192.168.1.121' password='{your_password_here}' port='24276' sslmode='true'
```

### <a name="ruby"></a>Ruby

```ruby
host=192.168.1.121; dbname=postgres user=postgres password={your_password_here} port=24276 sslmode=require
```

### <a name="web-app"></a>Aplicación web

```webapp
Database=postgres; Data Source=192.168.1.121; User Id=postgres; Password={your_password_here}
```

## <a name="next-steps"></a>Pasos siguientes
- Obtenga información sobre el [escalado horizontal (adición de nodos de trabajo)](scale-out-postgresql-hyperscale-server-group.md) del grupo de servidores
- Obtenga información sobre [cómo escalar o reducir verticalmente (aumentar o reducir la memoria o los núcleos virtuales)](scale-up-down-postgresql-hyperscale-server-group-using-cli.md) el grupo de servidores


