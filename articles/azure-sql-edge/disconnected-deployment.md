---
title: 'Implementación de Azure SQL Edge con Docker: Azure SQL Edge'
description: Más información sobre la implementación de Azure SQL Edge con Docker
keywords: SQL Edge, contenedor, Docker
services: sql-edge
ms.service: sql-edge
ms.topic: quickstart
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: ff14f8a9f236701889aea95911f2a1e381eabf83
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945242"
---
# <a name="deploy-azure-sql-edge-with-docker"></a>Implementación de Azure SQL Edge con Docker

En este inicio rápido, usará Docker para extraer y ejecutar la imagen de contenedor de Azure SQL Edge. A continuación, tendrá que conectar con **sqlcmd** para crear la primera base de datos y ejecutar consultas.

Esta imagen se compone de Azure SQL Edge basado en Ubuntu 18.04. Se puede usar junto al motor de Docker 1.8 o versiones posteriores en Linux, o bien en Docker en Mac y Windows.

## <a name="prerequisites"></a>Prerequisites

- Motor de Docker 1.8 o versiones posteriores en cualquier distribución de Linux admitida o Docker para Mac y Windows. Para obtener más información, consulte [Instalar Docker](https://docs.docker.com/engine/installation/). Dado que las imágenes de Azure SQL Edge se basan en Ubuntu 18.04, se recomienda usar un host de Docker de Ubuntu 18.04.
- Controlador de almacenamiento **overlay2** de Docker. Este es el valor predeterminado para la mayoría de los usuarios. Si no está usando este proveedor de almacenamiento y necesita cambiar, vea las instrucciones y advertencias de la [documentación de Docker para configurar overlay2](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#configure-docker-with-the-overlay-or-overlay2-storage-driver).
- Un mínimo de 10 GB de espacio en disco.
- Un mínimo de 1 GB de RAM.
- [Requisitos de hardware de Azure SQL Edge](https://docs.microsoft.com/azure/azure-sql-edge/features#hardware-support).


## <a name="pull-and-run-the-container-image"></a>Extraer y ejecutar la imagen de contenedor

Antes de iniciar los pasos siguientes, asegúrese de que ha seleccionado el shell preferido (Bash, PowerShell o cmd) en la parte superior de este artículo.

1. Extraiga la imagen de contenedor de Azure SQL Edge desde Microsoft Container Registry.

    - Extracción de la imagen de contenedor de Azure SQL Edge
        ```bash
        sudo docker pull mcr.microsoft.com/azure-sql-edge:latest 
        ```

> [!NOTE]
> En los comandos de Bash de este artículo se usa `sudo`. En macOS y Windows, es posible que sudo no sea necesario. En Linux, si no quiere usar sudo para ejecutar Docker, puede configurar un grupo de Docker y agregarle usuarios. Para obtener más información, vea [Pasos posteriores a la instalación para Linux](https://docs.docker.com/engine/install/linux-postinstall/).

El comando anterior extrae las imágenes de contenedor de Azure SQL Edge más recientes. Para ver todas las imágenes disponibles, consulte la [página de Docker Hub azure-sql-egde](https://hub.docker.com/_/microsoft-azure-sql-edge).

2. Para ejecutar la imagen de contenedor con Docker, puede usar el siguiente comando desde un shell de Bash (Linux y macOS), o bien una línea de comandos de PowerShell con privilegios elevados.
    
    - Inicio de una instancia de Azure SQL Edge que se ejecuta como la edición Developer
        ```bash
        sudo docker run --cap-add SYS_PTRACE -e 'ACCEPT_EULA=1' -e 'MSSQL_SA_PASSWORD=yourStrong(!)Password' -p 1433:1433 --name azuresqledge -d mcr.microsoft.com/azure-sql-edge
        ```

    - Inicio de una instancia de Azure SQL Edge que se ejecuta como la edición Premium
        ```bash
        sudo docker run --cap-add SYS_PTRACE -e 'ACCEPT_EULA=1' -e 'MSSQL_SA_PASSWORD=yourStrong(!)Password' -e 'MSSQL_PID=Premium' -p 1433:1433 --name azuresqledge -d mcr.microsoft.com/azure-sql-edge
        ```
    > [!NOTE]
    > Si usa PowerShell en Windows para ejecutar estos comandos, use comillas dobles en lugar de comillas simples.


    > [!NOTE]
    > La contraseña debe seguir la directiva de contraseñas predeterminada del Motor de base de datos de Microsoft SQL. En caso contrario, el contenedor no podrá instalar el motor SQL y dejará de funcionar. De forma predeterminada, la contraseña debe tener al menos 8 caracteres y contener caracteres de tres de los cuatro conjuntos siguientes: Letras mayúsculas, letras minúsculas, dígitos de Base 10 y símbolos. Puede examinar el registro de errores ejecutando el comando [docker logs](https://docs.docker.com/engine/reference/commandline/logs/).
    
    En la tabla siguiente, se proporciona una descripción de los parámetros del ejemplo de `docker run` anterior:

    | Parámetro | Descripción |
    |-----|-----|
    | **-e "ACCEPT_EULA=Y"** |  Establezca la variable **ACCEPT_EULA** en cualquier valor para confirmar que acepta el [Contrato de licencia de usuario final](https://go.microsoft.com/fwlink/?linkid=2139274). Configuración requerida para la imagen de Azure SQL Edge. |
    | **-e "MSSQL_SA_PASSWORD=yourStrong(!)Password"** | Especifique su contraseña segura con al menos 8 caracteres y que cumpla los [requisitos de contraseña de Azure SQL Edge](https://docs.microsoft.com/sql/relational-databases/security/password-policy). Configuración requerida para la imagen de Azure SQL Edge. |
    | **-p 1433:1433** | Asigne un puerto TCP en el entorno de host (el primer valor) a un puerto TCP en el contenedor (el segundo valor). En este ejemplo, Azure SQL Edge realiza escuchas en TCP 1433 en el contenedor y se expone al puerto 1433 del host. |
    | **--name azuresqledge** | Especifique un nombre personalizado para el contenedor en lugar de uno generado aleatoriamente. Si ejecuta más de un contenedor, no podrá usar el mismo nombre. |
    | **-d** | Ejecute el contenedor en segundo plano (demonio). |

    Para obtener una lista completa de todas las variables de entorno de Azure SQL Edge, consulte [Configuración de Azure SQL Edge con variables de entorno](configure.md#configure-by-using-environment-variables). También puede usar un archivo [mssql.conf file](configure.md#configure-by-using-an-mssqlconf-file) para configurar contenedores de Azure SQL Edge.

3. Para ver los contenedores de Docker, use el comando `docker ps`.
   
   ```bash
    sudo docker ps -a
   ```

4. Si la columna **ESTADO** muestra el estado **Activo**, esto indica que Azure SQL Edge se está ejecutando en el contenedor y que realiza escuchas en el puerto especificado en la columna **PUERTOS**. Si la columna **ESTADO** de su contenedor de Azure SQL Edge muestra **Cerrado**, consulte la sección Solución de problemas de la documentación de Azure SQL Edge.

    El parámetro `-h` (nombre del host) también es útil, pero no se usa en este tutorial para simplificar el proceso. Esto cambia el nombre interno del contenedor a un valor personalizado. Este es el nombre que verá que se devuelve en la siguiente consulta de Transact-SQL:

    ```sql
    SELECT @@SERVERNAME,
        SERVERPROPERTY('ComputerNamePhysicalNetBIOS'),
        SERVERPROPERTY('MachineName'),
        SERVERPROPERTY('ServerName')
    ```

    Establecer `-h` y `--name` en el mismo valor es una buena manera de identificar fácilmente el contenedor de destino.

5. Como paso final, cambie la contraseña de administrador del sistema porque `SA_PASSWORD` se ve en la salida de `ps -eax` y se almacena en la variable de entorno con el mismo nombre. Vea los pasos siguientes.

## <a name="change-the-sa-password"></a>Cambiar la contraseña de SA

La cuenta de **SA** es un administrador del sistema de la instancia de Azure SQL Edge que se crea durante la instalación. Después de crear el contenedor de Azure SQL Edge, la variable de entorno `MSSQL_SA_PASSWORD` especificada se reconoce mediante la ejecución de `echo $SA_PASSWORD` en el contenedor. Por motivos de seguridad, cambie la contraseña de administrador del sistema.

1. Elija una contraseña segura que se usará para el usuario SA.

2. Use `docker exec` para ejecutar **sqlcmd** a fin de cambiar la contraseña mediante Transact-SQL. En el ejemplo siguiente, reemplace la contraseña anterior, `<YourStrong!Passw0rd>`, y la nueva contraseña, `<YourNewStrong!Passw0rd>`, por sus propios valores de contraseña.

   ```bash
   sudo docker exec -it azuresqledge /opt/mssql-tools/bin/sqlcmd \
      -S localhost -U SA -P "<YourStrong@Passw0rd>" \
      -Q 'ALTER LOGIN SA WITH PASSWORD="<YourNewStrong@Passw0rd>"'
   ```

## <a name="connect-to-azure-sql-edge"></a>Conexión a Azure SQL Edge

En los pasos siguientes, se usa la herramienta de línea de comandos de Azure SQL Edge, **sqlcmd**, dentro del contenedor para conectarse a Azure SQL Edge.

> [!NOTE]
> La herramienta sqlcmd no está disponible en la versión ARM64 de los contenedores de SQL Edge.

1. Use el comando `docker exec -it` para iniciar un shell de Bash interactivo dentro de su contenedor en ejecución. En el ejemplo siguiente, el parámetro `--name` especifica el nombre de `azuresqledge` al crear el contenedor.

   ```bash
   sudo docker exec -it azuresqledge "bash"
   ```

2. Una vez dentro del contenedor, conecte localmente con sqlcmd. Sqlcmd no está en la ruta de acceso de forma predeterminada, por lo que deberá especificar la ruta de acceso completa.

   ```bash
   /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "<YourNewStrong@Passw0rd>"
   ```

   > [!TIP]
   > Puede omitir la contraseña en la línea de comandos para que se le solicite escribirla.

3. Si se realiza correctamente, debe ver un símbolo de sistema de **sqlcmd**: `1>`.

## <a name="create-and-query-data"></a>Creación y consulta de datos

Las secciones siguientes le guían en el uso de **sqlcmd** y Transact-SQL para crear una base de datos, agregar datos y ejecutar una consulta simple.

### <a name="create-a-new-database"></a>Creación de una base de datos

En los pasos siguientes se crea una base de datos denominada `TestDB`.

1. En el símbolo del sistema de **sqlcmd**, pegue el comando Transact-SQL siguiente para crear una base de datos de prueba:

   ```sql
   CREATE DATABASE TestDB
   Go
   ```

2. En la línea siguiente, escriba una consulta para devolver el nombre de todas las bases de datos del servidor:

   ```sql
   SELECT Name from sys.Databases
   Go
   ```

### <a name="insert-data"></a>Insertar datos

Luego cree una tabla, `Inventory`, e inserte dos filas nuevas.

1. En el símbolo del sistema de **sqlcmd**, cambie el contexto a la nueva base de datos `TestDB`:

   ```sql
   USE TestDB
   ```

2. Cree una tabla llamada `Inventory`:

   ```sql
   CREATE TABLE Inventory (id INT, name NVARCHAR(50), quantity INT)
   ```

3. Inserte datos en la nueva tabla:

   ```sql
   INSERT INTO Inventory VALUES (1, 'banana', 150); INSERT INTO Inventory VALUES (2, 'orange', 154);
   ```

4. Escriba `GO` para ejecutar los comandos anteriores:

   ```sql
   GO
   ```

### <a name="select-data"></a>Selección de datos

Ahora ejecute una consulta para devolver datos desde la tabla `Inventory`.

1. En el símbolo del sistema **sqlcmd**, escriba una consulta que devuelva filas desde la tabla `Inventory` donde la cantidad sea mayor que 152:

   ```sql
   SELECT * FROM Inventory WHERE quantity > 152;
   ```

2. Ejecute el comando:

   ```sql
   GO
   ```

### <a name="exit-the-sqlcmd-command-prompt"></a>Salida del símbolo del sistema de sqlcmd

1. Para finalizar la sesión de **sqlcmd**, escriba `QUIT`:

   ```sql
   QUIT
   ```

2. Para salir de la línea de comandos interactiva del contenedor, escriba `exit`. El contenedor continuará ejecutándose después de salir del shell de Bash interactivo.

## <a name="connect-from-outside-the-container"></a> Conectarse desde fuera del contenedor

También puede conectarse a la instancia de Azure SQL Edge en la máquina de Docker desde cualquier herramienta externa de Linux, Windows o Mac OS que admita conexiones de SQL. Para más información sobre cómo conectarse a un contenedor de SQL Edge desde fuera, vea [Conexión a Azure SQL Edge y consulta (versión preliminar)](connect.md).

## <a name="remove-your-container"></a>Quitar el contenedor

Si quiere quitar el contenedor de Azure SQL Edge que se usa en este tutorial, ejecute los comandos siguientes:

```bash
sudo docker stop azuresqledge
sudo docker rm azuresqledge
```

> [!WARNING]
> Al detener y quitar un contenedor, se eliminarán permanentemente los datos de Azure SQL Edge que contenga. Si tiene que conservar sus datos, [cree un archivo de copia de seguridad y cópielo en una ubicación externa al contenedor](backup-restore.md), o bien use una [técnica de persistencia de datos de contenedor](configure.md#persist-your-data).

## <a name="next-steps"></a>Pasos siguientes

- [Aprendizaje automático e inteligencia artificial con ONNX en SQL Edge](onnx-overview.md).
- [Creación de una solución de IoT de un extremo a otro con SQL Edge mediante IoT Edge](tutorial-deploy-azure-resources.md).
- [Streaming de datos en Azure SQL Edge](stream-data.md)
