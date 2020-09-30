---
title: Conexión a Azure SQL Edge y consulta
description: Aprenda a conectarse a Azure SQL Edge y enviarle consultas.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 07/25/2020
ms.openlocfilehash: 4548d4956b4cd01886fb1be9a530cc1627f76b2c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90888239"
---
# <a name="connect-and-query-azure-sql-edge"></a>Conexión a Azure SQL Edge y consulta

En Azure SQL Edge, después de implementar un contenedor, puede conectarse al motor de base de datos desde cualquiera de las siguientes ubicaciones:

- En el contenedor.
- Desde otro contenedor de Docker que se ejecute en el mismo host.
- Desde el equipo host.
- Desde cualquier otro equipo cliente de la red.

## <a name="tools-to-connect-to-azure-sql-edge"></a>Herramientas para conectarse a Azure SQL Edge

Puede conectarse a una instancia de Azure SQL Edge desde cualquiera de las siguientes herramientas comunes:

* [sqlcmd](https://docs.microsoft.com/sql/linux/sql-server-linux-setup-tools): la imagen de contenedor de Azure SQL Edge ya incluye las herramientas de cliente de sqlcmd. Si adjunta un contenedor en ejecución con un shell de Bash interactivo, puede ejecutar las herramientas de forma local. Las herramientas de cliente SQL NO están disponibles en la plataforma ARM64, ya que no se incluyen en la versión ARM64 de los contenedores de SQL Edge. 
* [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/sql-server-management-studio-ssms)
* [Azure Data Studio](https://docs.microsoft.com/sql/azure-data-studio/download-azure-data-studio)
* [Visual Studio Code](https://docs.microsoft.com/sql/visual-studio-code/sql-server-develop-use-vscode)

Para conectarse a un motor de base de datos de Azure SQL Edge desde un equipo de red, necesitará lo siguiente:

- **Dirección IP o nombre de red del equipo host**: Este es el equipo host en el que se ejecuta el contenedor de Azure SQL Edge.
- **Asignación de puertos de host del contenedor de Azure SQL Edge**: Esta es la asignación de un puerto de contenedor de Docker a un puerto en el host. Dentro del contenedor, Azure SQL Edge siempre se asigna al puerto 1433. Si quiere, puede cambiarlo. Para cambiar el número de puerto, actualice las **Opciones de creación de contenedores** para el módulo de Azure SQL Edge en Azure IoT Edge. En el siguiente ejemplo, el puerto 1433 del contenedor está asignado al puerto 1600 en el host.

    ```JSON
    {
        "PortBindings": {
          "1433/tcp": [
            {
              "HostPort": "1600"
            }
          ]
        }
    }
    ```

- **Contraseña de administrador del sistema para la instancia de Azure SQL Edge**: Este es el valor especificado para la variable de entorno `SA_PASSWORD` durante la implementación de Azure SQL Edge.

## <a name="connect-to-the-database-engine-from-within-the-container"></a>Conexión al motor de base de datos dentro del contenedor

Las [herramientas de línea de comandos de SQL Server](https://docs.microsoft.com/sql/linux/sql-server-linux-setup-tools) están incluidas en la imagen de contenedor de Azure SQL Edge. Si se asocia el contenedor con un símbolo del sistema interactivo, puede ejecutar las herramientas de forma local. Las herramientas de cliente SQL NO están disponibles en la plataforma ARM64, ya que no se incluyen en la versión ARM64 de los contenedores de SQL Edge. 

1. Use el comando `docker exec -it` para iniciar un shell de Bash interactivo dentro de su contenedor en ejecución. En el ejemplo siguiente, `e69e056c702d` es el identificador del contenedor.

    ```bash
    docker exec -it <Azure SQL Edge container ID or name> /bin/bash
    ```

    > [!TIP]
    > No siempre tiene que especificar el id. completo del contenedor. Solo tiene que especificar suficientes caracteres para identificarlo de forma única. Por lo tanto, en este ejemplo, podría ser suficiente usar `e6` o `e69` en lugar del id. completo.

2. Cuando esté dentro del contenedor, conéctese localmente con sqlcmd. Sqlcmd no está en la ruta de acceso de forma predeterminada, por lo que deberá especificar la ruta de acceso completa.

    ```bash
    /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P '<YourPassword>'
    ```

3. Cuando haya terminado con sqlcmd, escriba `exit`.

4. Cuando haya terminado con el símbolo del sistema interactivo, escriba `exit`. El contenedor continuará ejecutándose después de salir del shell de Bash interactivo.

## <a name="connect-to-azure-sql-edge-from-another-container-on-the-same-host"></a>Conexión a Azure SQL Edge desde otro contenedor en el mismo host

Ya que dos contenedores que se ejecutan en el mismo host se encuentran en la misma red de Docker, se puede acceder fácilmente a ellos mediante el nombre del contenedor y la dirección del puerto para el servicio. Por ejemplo, si se va a conectar a la instancia de Azure SQL Edge desde otro módulo de Python (contenedor) en el mismo host, puede usar una cadena de conexión similar a la siguiente. (En este ejemplo se supone que Azure SQL Edge está configurado para escuchar en el puerto predeterminado).

```python

import pyodbc
server = 'MySQLEdgeContainer' # Replace this with the actual name of your SQL Edge Docker container
username = 'sa' # SQL Server username
password = 'MyStrongestP@ssword' # Replace this with the actual SA password from your deployment
database = 'MyEdgeDatabase' # Replace this with the actual database name from your deployment. If you do not have a database created, you can use Master database.
db_connection_string = "Driver={ODBC Driver 17 for SQL Server};Server=" + server + ";Database=" + database + ";UID=" + username + ";PWD=" + password + ";"
conn = pyodbc.connect(db_connection_string, autocommit=True)

```

## <a name="connect-to-azure-sql-edge-from-another-network-machine"></a>Conexión a Azure SQL Edge desde otro equipo de red

Es posible que quiera conectarse a la instancia de Azure SQL Edge desde otro equipo de la red. Para ello, use la dirección IP del host de Docker y el puerto de host al que está asignado el contenedor perimetral de Azure SQL Edge. Por ejemplo, si la dirección IP del host de Docker es *xxx.xxx.xxx.xxx* y el contenedor de Azure SQL Edge está asignado al puerto de host *1600*, la dirección del servidor para la instancia de Azure SQL Edge sería *xxx.xxx.xxx.xxx,1600*. El script de Python actualizado es:

```python

import pyodbc
server = 'xxx.xxx.xxx.xxx,1600' # Replace this with the actual name of your SQL Edge Docker container
username = 'sa' # SQL Server username
password = 'MyStrongestP@ssword' # Replace this with the actual SA password from your deployment
database = 'MyEdgeDatabase' # Replace this with the actual database name from your deployment. If you do not have a database created, you can use Master database.
db_connection_string = "Driver={ODBC Driver 17 for SQL Server};Server=" + server + ";Database=" + database + ";UID=" + username + ";PWD=" + password + ";"
conn = pyodbc.connect(db_connection_string, autocommit=True)

```

Para conectarse a una instancia de Azure SQL Edge con SQL Server Management Studio en ejecución en un equipo Windows, consulte [SQL Server Management Studio](https://docs.microsoft.com/sql/linux/sql-server-linux-manage-ssms).

Para conectarse a una instancia de Azure SQL Edge con Visual Studio Code en un equipo Windows, Mac o Linux, consulte [Visual Studio Code](https://docs.microsoft.com/sql/visual-studio-code/sql-server-develop-use-vscode).

Para conectarse a una instancia de Azure SQL Edge con Azure Data Studio en un equipo Windows, Mac o Linux, consulte [Azure Data Studio](https://docs.microsoft.com/sql/azure-data-studio/quickstart-sql-server).

## <a name="next-steps"></a>Pasos siguientes

[Conexión y consultas](https://docs.microsoft.com/sql/linux/sql-server-linux-configure-docker#connect-and-query)

[Instalación de las herramientas de SQL Server en Linux](https://docs.microsoft.com/sql/linux/sql-server-linux-setup-tools)
