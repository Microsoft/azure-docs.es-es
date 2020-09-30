---
title: Implementación de Azure SQL Edge mediante Azure Portal
description: Obtenga información sobre cómo implementar Azure SQL Edge mediante Azure Portal
keywords: implementación de SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: ffd967797a4e586387a0385169672220727f78a7
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90886768"
---
# <a name="deploy-azure-sql-edge"></a>Implementación de Azure SQL Edge 

Azure SQL Edge es un motor de base de datos relacional optimizado para implementaciones de IoT y Azure IoT Edge. Proporciona funcionalidades para crear una capa de procesamiento y almacenamiento de datos de alto rendimiento para las aplicaciones y soluciones de IoT. Esta guía de inicio rápido le muestra cómo comenzar a crear un módulo de Azure SQL Edge a través de Azure IoT Edge mediante Azure Portal.

## <a name="before-you-begin"></a>Antes de empezar

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/).
* Inicie sesión en [Azure Portal](https://portal.azure.com/).
* Cree un [Azure IoT Hub](../iot-hub/iot-hub-create-through-portal.md).
* Registre un [dispositivo IoT Edge desde Azure Portal](../iot-edge/how-to-register-device-portal.md).
* Prepare el dispositivo IoT Edge para [implementar un módulo de IoT Edge desde Azure Portal](../iot-edge/how-to-deploy-modules-portal.md).

> [!NOTE]   
> Para implementar una máquina virtual Linux de Azure como un dispositivo IoT Edge, consulte esta [guía de inicio rápido](../iot-edge/quickstart-linux.md).

## <a name="deploy-sql-edge-module-from-azure-marketplace"></a>Implementación del módulo de SQL Edge desde Azure Marketplace

Azure Marketplace es un mercado de aplicaciones y servicios en línea en el que puede examinar una amplia gama de aplicaciones y soluciones empresariales que están certificadas y optimizadas para ejecutarse en Azure, incluyendo los [módulos de IoT Edge](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules). Azure SQL Edge se puede implementar en un dispositivo perimetral a través de Marketplace.

1. Busque el módulo de Azure SQL Edge en Azure Marketplace.<br><br>

   ![SQL Edge en Marketplace](media/deploy-portal/find-offer-marketplace.png)

2. Elija el plan de software que mejor se adapte a sus requisitos y haga clic en **Crear**. <br><br>

   ![Elegir el plan de software correcto](media/deploy-portal/pick-correct-plan.png)

3. En la página de dispositivos de destino para el módulo de IoT Edge, especifique los detalles siguientes y, a continuación, haga clic en **Crear**

   |**Campo**  |**Descripción**  |
   |---------|---------|
   |Subscription  |  La suscripción de Azure en la que se creó el IoT Hub |
   |IoT Hub   |  Nombre del IoT Hub en el que está registrado el dispositivo IoT Edge y, a continuación, seleccione la opción "Implementar en un dispositivo"|
   |Nombre de dispositivo IoT Edge  |  Nombre del dispositivo IoT Edge en el que se implementará SQL Edge |

4. En la página **Establecer módulos en el dispositivo**, haga clic en el módulo de Azure SQL Edge en **Módulos de IoT Edge**. El nombre del módulo predeterminado se establece en *AzureSQLEdge*. 

5. En la sección *configuración del módulo de*  de la hoja **Actualizar módulo IoT Edge**, especifique los valores que desee para *Nombre del módulo IoT Edge*, *Directiva de reinicio* y *Estado deseado*. 

   > [!IMPORTANT]    
   > No cambie ni actualice la configuración de **URI de la imagen** del módulo.

6. En la sección *Variables de entorno* de la hoja **Actualizar módulo IoT Edge**, especifique los valores que desee para las variables de entorno. Para obtener una lista completa de las variables de entorno de Azure SQL Edge, consulte [Configuración mediante variables de entorno](configure.md#configure-by-using-environment-variables). Se han definido las siguientes variables de entorno predeterminadas para el módulo. 

   |**Parámetro**  |**Descripción**|
   |---------|---------|
   | Nombre | Nombre del módulo. |
   | MSSQL_SA_PASSWORD  | Cambie el valor predeterminado para especificar una contraseña segura para la cuenta de administrador de SQL Edge. |
   | MSSQL_LCID   | Cambie el valor predeterminado para establecer el identificador de idioma deseado que se va a usar para SQL Edge. Por ejemplo, 1036 es francés. |
   | MSSQL_COLLATION | Cambie el valor predeterminado para establecer la intercalación predeterminada para SQL Edge. Esta configuración invalida la asignación predeterminada de identificador de idioma (LCID) a la intercalación. |

   > [!IMPORTANT]    
   > No cambie ni actualice la variable de entorno **ACCEPT_EULA** para el módulo.

7. En la sección *Opciones de creación del contenedor* de la hoja **Actualizar módulo IoT Edge**, actualice las siguientes opciones según sea necesario. 
   - **Puerto de host:**  Asigne el puerto de host especificado al puerto 1433 (puerto SQL predeterminado) en el contenedor.
   - **Enlaces** y **Montajes:** Si necesita implementar más de un módulo de SQL Edge, asegúrese de actualizar la opción de montajes a fin de crear un nuevo par de origen y destino para el volumen persistente. Para obtener más información sobre los montajes y el volumen, consulte el artículo sobre cómo [usar volúmenes](https://docs.docker.com/storage/volumes/) en la documentación de Docker. 

   ```json
   {
    "HostConfig": {
        "CapAdd": [
            "SYS_PTRACE"
        ],
        "Binds": [
            "sqlvolume:/sqlvolume"
        ],
        "PortBindings": {
            "1433/tcp": [
                {
                    "HostPort": "1433"
                }
            ]
        },
        "Mounts": [
            {
                "Type": "volume",
                "Source": "sqlvolume",
                "Target": "/var/opt/mssql"
            }
        ]
    },
    "Env": [
        "MSSQL_AGENT_ENABLED=TRUE",
        "ClientTransportType=AMQP_TCP_Only",
        "PlanId=asde-developer-on-iot-edge"
    ]
   }
   ```
   > [!IMPORTANT]    
   > No cambie la variable de entorno `PlanId` definida en la opción de crear configuración. Si se cambia este valor, el contenedor de Azure SQL Edge no se iniciará. 
   
8. En el panel **Actualizar módulo IoT Edge**, haga clic en **Actualizar**.
9. En la página **Establecer módulos en el dispositivo**, haga clic en **Siguiente: Rutas >** si necesita definir rutas para su implementación. De lo contrario, haga clic en **Revisar + crear**. Para obtener más información sobre la configuración de rutas, vea [Implementación de módulos y establecimiento de rutas en IoT Edge](../iot-edge/module-composition.md).
11. En el panel **Actualizar módulo IoT Edge**, haga clic en **Crear**.

## <a name="connect-to-azure-sql-edge"></a>Conexión a Azure SQL Edge

En los pasos siguientes, se usa la herramienta de línea de comandos de Azure SQL Edge, **sqlcmd**, dentro del contenedor para conectarse a Azure SQL Edge.

> [!NOTE]      
> Las herramientas de línea de comandos (sqlcmd) de SQL no están disponibles en la versión ARM64 de los contenedores perimetrales de Azure SQL Edge.

1. Use el comando `docker exec -it` para iniciar un shell de Bash interactivo dentro de su contenedor en ejecución. En el ejemplo siguiente, `azuresqledge` es el nombre especificado por el parámetro `Name` del módulo de IoT Edge.

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

En las secciones siguientes se le guía por el uso de **sqlcmd** y Transact-SQL para crear una base de datos, agregar datos y ejecutar una consulta.

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

Puede conectarse a la instancia de Azure SQL Edge y ejecutar en ella consultas SQL desde cualquier herramienta externa de Linux, Windows o macOS compatible con las conexiones de SQL. Para más información sobre cómo conectarse a un contenedor de SQL Edge desde fuera, vea [Conexión a Azure SQL Edge y consulta (versión preliminar)](https://docs.microsoft.com/azure/azure-sql-edge/connect).

Con esta guía de inicio rápido, ha implementado un módulo de SQL Edge en un dispositivo IoT Edge. 

## <a name="next-steps"></a>Pasos siguientes

- [Aprendizaje automático e inteligencia artificial con ONNX en SQL Edge](onnx-overview.md)
- [Creación de una solución de IoT de un extremo a otro con SQL Edge mediante IoT Edge](tutorial-deploy-azure-resources.md)
- [Streaming de datos en Azure SQL Edge](stream-data.md)
- [Solución de errores de implementación](troubleshoot.md)
