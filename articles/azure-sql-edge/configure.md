---
title: Configuración de Azure SQL Edge (versión preliminar)
description: Información sobre la configuración de Azure SQL Edge (versión preliminar).
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 07/28/2020
ms.openlocfilehash: 722d33e76b6009a44811dfcb8a3238b042ec6918
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816888"
---
# <a name="configure-azure-sql-edge-preview"></a>Configuración de Azure SQL Edge (versión preliminar)

Azure SQL Edge admite la configuración a través de una de las dos opciones siguientes:

- Variables de entorno
- Un archivo mssql.conf colocado en la carpeta /var/opt/mssql

> [!NOTE]
> La configuración de variables de entorno invalida la configuración especificada en el archivo mssql.conf.

## <a name="configure-by-using-environment-variables"></a>Configuración con variables de entorno

Azure SQL Edge expone varias variables de entorno diferentes que se pueden usar para configurar el contenedor de SQL Edge. Estas variables de entorno son un subconjunto de las disponibles para SQL Server en Linux. Para más información acerca de las variables de entorno de SQL Server en Linux, consulte [Variables de entorno](/sql/linux/sql-server-linux-configure-environment-variables/).

La siguiente variable de entorno de SQL Server en Linux no se admite en Azure SQL Edge. Si se define, esta variable de entorno se omitirá durante la inicialización del contenedor.

| Variable de entorno | Descripción |
|-----|-----|
| **MSSQL_ENABLE_HADR** | Habilite el grupo de disponibilidad. Por ejemplo, **1** significa que está habilitado y **0**, deshabilitado. |

> [!IMPORTANT]
> La variable de entorno **MSSQL_PID** para SQL Edge solo acepta **Premium** y **Desarrollador** como valores válidos. Azure SQL Edge no admite la inicialización mediante una clave de producto.

> [!NOTE]
> Descargue los [términos de licencia del software de Microsoft](https://go.microsoft.com/fwlink/?linkid=2128283) para Azure SQL Edge.

### <a name="specify-the-environment-variables"></a>Especificación de las variables de entorno

Especifique las variables de entorno de SQL Edge al implementar el servicio mediante [Azure Portal](deploy-portal.md). Puede agregarlas en la sección **Variables de entorno** de la implementación del módulo o como parte de las **Opciones de creación del contenedor**.

Agregue valores en **Variables de entorno**.

![Establecimiento mediante la lista de variables de entorno](media/configure/set-environment-variables.png)

Agregue valores en **Opciones de creación del contenedor**.

![Establecimiento mediante las opciones de creación del contenedor](media/configure/set-environment-variables-using-create-options.png)

## <a name="configure-by-using-an-mssqlconf-file"></a>Configuración mediante un archivo mssql.conf

Azure SQL Edge no incluye la [utilidad de configuración mssql-conf](/sql/linux/sql-server-linux-configure-mssql-conf/), como sí hace SQL Server en Linux. Debe configurar manualmente el archivo mssql.conf y colocarlo en la unidad de almacenamiento persistente asignada a la carpeta /var/opt/mssql/ en el módulo SQL Edge. Al implementar SQL Edge desde Azure Marketplace, esta asignación se especifica como la opción **Mounts** en las **Opciones de creación del contenedor**.

```json
    {
        "Mounts": [
          {
            "Type": "volume",
            "Source": "sqlvolume",
            "Target": "/var/opt/mssql"
          }
        ]
      }
    }
```

Las siguientes opciones de mssql.conf no son aplicables a SQL Edge:

|Opción|Descripción|
|:---|:---|
|**Comentarios del cliente** | Elija si SQL Server envía comentarios a Microsoft. |
|**Perfil de correo electrónico de base de datos** | Establezca el perfil de correo electrónico de base de datos predeterminado para SQL Server en Linux. |
|**Alta disponibilidad** | Habilite los grupos de disponibilidad. |
|**Microsoft DTC (Coordinador de transacciones distribuidas)** | Configure y solucione problemas de MSDTC en Linux. No se admiten opciones de configuración relacionadas con transacciones distribuidas adicionales para SQL Edge. Para más información sobre estas opciones de configuración adicionales, consulte [Configuración de MSDTC](https://docs.microsoft.com/sql/linux/sql-server-linux-configure-mssql-conf#msdtc). |
|**CLUF de ML Services** | Acepte los CLUF de R y Python de los paquetes de Azure Machine Learning. Solo se aplica a SQL Server 2019.|
|**outboundnetworkaccess** |Habilite el acceso de red saliente para las extensiones de R, Python y Java de [Machine Learning Services](/sql/linux/sql-server-linux-setup-machine-learning/).|

El siguiente archivo mssql.conf de ejemplo funciona con SQL Edge. Para más información sobre el formato del archivo mssql.conf, consulte [Formato de mssql.conf](https://docs.microsoft.com/sql/linux/sql-server-linux-configure-mssql-conf#mssql-conf-format).

```ini
[EULA]
accepteula = Y

[coredump]
captureminiandfull = true
coredumptype = full

[filelocation]
defaultbackupdir = /var/opt/mssql/backup/
defaultdatadir = /var/opt/mssql/data/
defaultdumpdir = /var/opt/mssql/data/
defaultlogdir = /var/opt/mssql/log/

[language]
lcid = 1033

[memory]
memorylimitmb = 6144

[sqlagent]
errorlogfile = /var/opt/mssql/log/sqlagentlog.log
errorlogginglevel = 7

[traceflag]
traceflag0 = 3604
traceflag1 = 3605
traceflag2 = 1204
```

## <a name="run-azure-sql-edge-as-non-root-user"></a>Ejecución de Azure SQL Edge como usuario no raíz

A partir de Azure SQL Edge CTP 2.2, los contenedores de SQL Edge se pueden ejecutar con un grupo o usuario no raíz. Cuando se implementa mediante Azure Marketplace, a menos que se especifique un usuario o grupo diferente, los contenedores de SQL Edge se inician como el usuario MSSQL (no raíz). Para especificar un usuario no raíz diferente durante la implementación, agregue el par clave-valor `*"User": "<name|uid>[:<group|gid>]"*` en las opciones de creación del contenedor. En el ejemplo siguiente, SQL Edge está configurado para iniciarse como el usuario `*IoTAdmin*`.

```json
{
    ..
    ..
    ..
    "User": "IoTAdmin",
    "Env": [
        "MSSQL_AGENT_ENABLED=TRUE",
        "ClientTransportType=AMQP_TCP_Only",
        "MSSQL_PID=Premium"
    ]
}
```

Para permitir que el usuario no raíz acceda a los archivos de base de datos que se encuentran en volúmenes montados, asegúrese de que el usuario o grupo en el que se ejecuta el contenedor tenga permisos de lectura y escritura en el almacenamiento de archivos persistente. En el ejemplo siguiente, se establece el usuario no raíz con user_id 10001 como propietario de los archivos. 

```bash
chown -R 10001:0 <database file dir>
```

### <a name="upgrading-from-earlier-ctp-releases"></a>Actualización a partir de versiones anteriores de CTP

Las CTP anteriores de Azure SQL Edge se configuraron para ejecutarse como usuarios raíz. Las siguientes opciones están disponibles al actualizar desde CTP anteriores.

- Continuar usando el usuario raíz: para seguir usando el usuario raíz, agregue el par clave-valor `*"User": "0:0"*` en las opciones de creación del contenedor.
- Usar el usuario predeterminado de MSSQL: para usar el usuario de MSSQL predeterminado, siga los pasos que se indican a continuación.
  - Agregue un usuario llamado MSSQL en el host de Docker. En el ejemplo siguiente, se agrega un usuario MSSQL con el id. 10001. Este usuario también se agrega al grupo raíz.
    ```bash
    sudo useradd -M -s /bin/bash -u 10001 -g 0 mssql
    ```
  - Cambiar el permiso en el directorio o el volumen de montaje donde reside el archivo de base de datos. 
    ```bash
    sudo chgrp -R 0 /var/lib/docker/volumes/kafka_sqldata/
    sudo chmod -R g=u /var/lib/docker/volumes/kafka_sqldata/
    ```
- Usar una cuenta de usuario no raíz diferente: para usar una cuenta de usuario no raíz diferente, puede hacer lo siguiente:
  - Actualice las opciones de creación del contenedor para especificar que se agregue un par clave-valor `*"User": "user_name | user_id*` en las opciones de creación del contenedor. Reemplace user_name o user_id por un valor de user_name o user_id real del host de Docker. 
  - Cambie los permisos del directorio o el volumen de montaje.

## <a name="persist-your-data"></a> Conservación de los datos

Los cambios de configuración de Azure SQL Edge y los archivos de base de datos se conservan en el contenedor aunque se reinicie el contenedor con `docker stop` y `docker start`. Pero si quita el contenedor con `docker rm`, se elimina todo el contenedor, incluidos Azure SQL Edge y las bases de datos. En la siguiente sección se explica cómo usar **volúmenes de datos** para conservar los archivos de base de datos incluso si se eliminan los contenedores asociados.

> [!IMPORTANT]
> Por Azure SQL Edge, es fundamental que comprenda la persistencia de los datos en Docker. Además de la explicación de esta sección, consulte la documentación de Docker sobre [cómo administrar datos en contenedores de Docker](https://docs.docker.com/engine/tutorials/dockervolumes/).

### <a name="mount-a-host-directory-as-data-volume"></a>Montaje de un directorio host como volumen de datos

La primera opción consiste en montar un directorio en el host como un volumen de datos en el contenedor. Para ello, use el comando `docker run` con la marca `-v <host directory>:/var/opt/mssql`. Esto permite restaurar los datos entre las ejecuciones del contenedor.

```bash
docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>' -p 1433:1433 -v <host directory>/data:/var/opt/mssql/data -v <host directory>/log:/var/opt/mssql/log -v <host directory>/secrets:/var/opt/mssql/secrets -d mcr.microsoft.com/azure-sql-edge-developer
```

```PowerShell
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>" -p 1433:1433 -v <host directory>/data:/var/opt/mssql/data -v <host directory>/log:/var/opt/mssql/log -v <host directory>/secrets:/var/opt/mssql/secrets -d mcr.microsoft.com/azure-sql-edge-developer
```

Esta técnica también le permite compartir y ver los archivos en el host fuera de Docker.

> [!IMPORTANT]
> La asignación de volúmenes de host para **Docker en Windows** no admite actualmente la asignación del directorio completo de `/var/opt/mssql`. Pero puede asignar un subdirectorio, como `/var/opt/mssql/data`, a su equipo host.

> [!IMPORTANT]
> En este momento, no se admite la asignación de volumen de host para **Docker en Mac** con la imagen de Azure SQL Edge. En su lugar, use contenedores de volúmenes de datos. Esta restricción es específica del directorio `/var/opt/mssql`. La lectura desde un directorio montado funciona correctamente. Por ejemplo, puede montar un directorio host mediante -v en Mac y restaurar una copia de seguridad desde un archivo .bak que resida en el host.

### <a name="use-data-volume-containers"></a>Uso de contenedores de volúmenes de datos

La segunda opción consiste en usar un contenedor de volúmenes de datos. Puede crear un contenedor de volúmenes de datos especificando un nombre de volumen en lugar de un directorio host con el parámetro `-v`. En el ejemplo siguiente se crea un volumen de datos compartidos denominado **sqlvolume**.

```bash
docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>' -p 1433:1433 -v sqlvolume:/var/opt/mssql -d mcr.microsoft.com/azure-sql-edge-developer
```

```PowerShell
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=<YourStrong!Passw0rd>" -p 1433:1433 -v sqlvolume:/var/opt/mssql -d mcr.microsoft.com/azure-sql-edge-developer
```

> [!NOTE]
> Esta técnica para crear de forma implícita un volumen de datos en el comando de ejecución no funciona con versiones anteriores de Docker. En ese caso, use los pasos explícitos que se describen en la documentación de Docker, [Creación y montaje de un contenedor de volúmenes de datos](https://docs.docker.com/engine/tutorials/dockervolumes/#creating-and-mounting-a-data-volume-container).

Incluso si detiene y quita este contenedor, el volumen de datos se conserva. Puede verlo con el comando `docker volume ls`.

```bash
docker volume ls
```

Si después crea otro contenedor con el mismo nombre de volumen, el nuevo contenedor usará los mismos datos de Azure SQL Edge contenidos en el volumen.

Para quitar un contenedor de volúmenes de datos, use el comando `docker volume rm`.

> [!WARNING]
> Si elimina el contenedor de volúmenes de datos, todos los datos de Azure SQL Edge del contenedor se eliminarán *de forma permanente*.


## <a name="next-steps"></a>Pasos siguientes

- [Conexión a Azure SQL Edge](connect.md)
- [Creación de una solución de IoT global con SQL Edge](tutorial-deploy-azure-resources.md)
