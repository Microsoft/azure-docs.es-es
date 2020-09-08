---
title: Implementación de una solución distribuida geográficamente
description: Aprenda a configurar su base de datos en Azure SQL Database y su aplicación cliente para realizar conmutación por error a una base de datos replicada y probar una conmutación por error.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1, devx-track-azurecli, devx-track-azurepowershell
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, carlrab
ms.date: 03/12/2019
ms.openlocfilehash: bdc03d93654fdb0ae90600f8dad279f0fc8efbe9
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89070843"
---
# <a name="tutorial-implement-a-geo-distributed-database-azure-sql-database"></a>Tutorial: Implementación de una base de datos distribuida geográficamente (Azure SQL Database)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Configure una base de datos en SQL Database y una aplicación cliente para realizar conmutación por error a una región remota y probar un plan de conmutación por error. Aprenderá a:

> [!div class="checklist"]
>
> - Cree un [grupo de conmutación por error](auto-failover-group-overview.md).
> - Ejecutar una aplicación de Java para consultar una base de datos en SQL Database
> - Conmutación por error de prueba

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible con Azure SQL Database, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](https://docs.microsoft.com/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

Para completar el tutorial, asegúrese de que instaló los elementos siguientes:

- [Azure PowerShell](/powershell/azure/)
- Una base de datos única en Azure SQL Database. Para crear uno, use:
  - [Azure Portal](single-database-create-quickstart.md)
  - [La CLI de Azure](az-cli-script-samples-content-guide.md)
  - [PowerShell](powershell-script-content-guide.md)

  > [!NOTE]
  > El tutorial usa la base de datos de ejemplo de *AdventureWorksLT*.

- Java y Maven, consulte [Build an app using SQL Server](https://www.microsoft.com/sql-server/developer-get-started/)(Compilar una aplicación con SQL Server), resalte **Java**, seleccione su entorno y, después, siga los pasos.

> [!IMPORTANT]
> Asegúrese de configurar las reglas de firewall para usar la dirección IP pública del equipo en el que está realizando los pasos de este tutorial. Las reglas de firewall a nivel de base de datos se replicarán automáticamente al servidor secundario.
>
> Para obtener más información, vea [Create a database-level firewall rule](/sql/relational-databases/system-stored-procedures/sp-set-database-firewall-rule-azure-sql-database) (Creación de una regla de firewall a nivel de base de datos) o, para determinar la dirección IP que usa la regla de firewall a nivel de base de datos para el equipo, vea [Create a server-level firewall](firewall-create-server-level-portal-quickstart.md) (Crear un firewall de nivel de servidor).  

## <a name="create-a-failover-group"></a>Creación de un grupo de conmutación por error

Con Azure PowerShell, cree [grupos de conmutación por error](auto-failover-group-overview.md) entre un servidor existente y uno nuevo en otra región. A continuación, agregue la base de datos de ejemplo al grupo de conmutación por error.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

> [!IMPORTANT]
> [!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

Para crear un grupo de conmutación por error, ejecute el siguiente script:

```powershell
$admin = "<adminName>"
$password = "<password>"
$resourceGroup = "<resourceGroupName>"
$location = "<resourceGroupLocation>"
$server = "<serverName>"
$database = "<databaseName>"
$drLocation = "<disasterRecoveryLocation>"
$drServer = "<disasterRecoveryServerName>"
$failoverGroup = "<globallyUniqueFailoverGroupName>"

# create a backup server in the failover region
New-AzSqlServer -ResourceGroupName $resourceGroup -ServerName $drServer `
    -Location $drLocation -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential `
    -ArgumentList $admin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))

# create a failover group between the servers
New-AzSqlDatabaseFailoverGroup –ResourceGroupName $resourceGroup -ServerName $server `
    -PartnerServerName $drServer –FailoverGroupName $failoverGroup –FailoverPolicy Automatic -GracePeriodWithDataLossHours 2

# add the database to the failover group
Get-AzSqlDatabase -ResourceGroupName $resourceGroup -ServerName $server -DatabaseName $database | `
    Add-AzSqlDatabaseToFailoverGroup -ResourceGroupName $resourceGroup -ServerName $server -FailoverGroupName $failoverGroup
```

# <a name="the-azure-cli"></a>[La CLI de Azure](#tab/azure-cli)

> [!IMPORTANT]
> Ejecute `az login` para iniciar sesión en Azure.

```azurecli
$admin = "<adminName>"
$password = "<password>"
$resourceGroup = "<resourceGroupName>"
$location = "<resourceGroupLocation>"
$server = "<serverName>"
$database = "<databaseName>"
$drLocation = "<disasterRecoveryLocation>" # must be different then $location
$drServer = "<disasterRecoveryServerName>"
$failoverGroup = "<globallyUniqueFailoverGroupName>"

# create a backup server in the failover region
az sql server create --admin-password $password --admin-user $admin `
    --name $drServer --resource-group $resourceGroup --location $drLocation

# create a failover group between the servers
az sql failover-group create --name $failoverGroup --partner-server $drServer `
    --resource-group $resourceGroup --server $server --add-db $database `
    --failover-policy Automatic --grace-period 2
```

* * *

Las opciones de configuración de replicación geográfica también pueden cambiarse en Azure Portal mediante la selección de la base de datos y, después, **Configuración** > **Replicación geográfica**.

![Configuración de la replicación geográfica](./media/geo-distributed-application-configure-tutorial/geo-replication.png)

## <a name="run-the-sample-project"></a>Ejecución del proyecto de ejemplo

1. En la consola, cree un proyecto de Maven con el siguiente comando:

   ```bash
   mvn archetype:generate "-DgroupId=com.sqldbsamples" "-DartifactId=SqlDbSample" "-DarchetypeArtifactId=maven-archetype-quickstart" "-Dversion=1.0.0"
   ```

1. Escriba **Y** y presione **ENTRAR**.

1. Cambie los directorios al nuevo proyecto.

   ```bash
   cd SqlDbSample
   ```

1. Con el editor que prefiera, abra el archivo *pom.xml* de la carpeta del proyecto.

1. Agregue la dependencia Microsoft JDBC Driver para SQL Server mediante la adición de la sección `dependency` siguiente. La dependencia se debe pegar en la sección `dependencies` de mayor tamaño.

   ```xml
   <dependency>
     <groupId>com.microsoft.sqlserver</groupId>
     <artifactId>mssql-jdbc</artifactId>
    <version>6.1.0.jre8</version>
   </dependency>
   ```

1. Especifique la versión de Java mediante la adición de la sección `properties` después de la sección `dependencies`:

   ```xml
   <properties>
     <maven.compiler.source>1.8</maven.compiler.source>
     <maven.compiler.target>1.8</maven.compiler.target>
   </properties>
   ```

1. Admita archivos de manifiesto agregando la sección `build` después de la sección `properties`:

   ```xml
   <build>
     <plugins>
       <plugin>
         <groupId>org.apache.maven.plugins</groupId>
         <artifactId>maven-jar-plugin</artifactId>
         <version>3.0.0</version>
         <configuration>
           <archive>
             <manifest>
               <mainClass>com.sqldbsamples.App</mainClass>
             </manifest>
           </archive>
        </configuration>
       </plugin>
     </plugins>
   </build>
   ```

1. Guarde y cierre el archivo *pom.xml*.

1. Abra el archivo *App.java* ubicado en ..\SqlDbSample\src\main\java\com\sqldbsamples y sustituya el contenido por el código siguiente:

   ```java
   package com.sqldbsamples;

   import java.sql.Connection;
   import java.sql.Statement;
   import java.sql.PreparedStatement;
   import java.sql.ResultSet;
   import java.sql.Timestamp;
   import java.sql.DriverManager;
   import java.util.Date;
   import java.util.concurrent.TimeUnit;

   public class App {

      private static final String FAILOVER_GROUP_NAME = "<your failover group name>";  // add failover group name
  
      private static final String DB_NAME = "<your database>";  // add database name
      private static final String USER = "<your admin>";  // add database user
      private static final String PASSWORD = "<your password>";  // add database password

      private static final String READ_WRITE_URL = String.format("jdbc:" +
         "sqlserver://%s.database.windows.net:1433;database=%s;user=%s;password=%s;encrypt=true;" +
         "hostNameInCertificate=*.database.windows.net;loginTimeout=30;", +
         FAILOVER_GROUP_NAME, DB_NAME, USER, PASSWORD);
      private static final String READ_ONLY_URL = String.format("jdbc:" +
         "sqlserver://%s.secondary.database.windows.net:1433;database=%s;user=%s;password=%s;encrypt=true;" +
         "hostNameInCertificate=*.database.windows.net;loginTimeout=30;", +
         FAILOVER_GROUP_NAME, DB_NAME, USER, PASSWORD);

      public static void main(String[] args) {
         System.out.println("#######################################");
         System.out.println("## GEO DISTRIBUTED DATABASE TUTORIAL ##");
         System.out.println("#######################################");
         System.out.println("");

         int highWaterMark = getHighWaterMarkId();

         try {
            for(int i = 1; i < 1000; i++) {
                //  loop will run for about 1 hour
                System.out.print(i + ": insert on primary " +
                   (insertData((highWaterMark + i))?"successful":"failed"));
                TimeUnit.SECONDS.sleep(1);
                System.out.print(", read from secondary " +
                   (selectData((highWaterMark + i))?"successful":"failed") + "\n");
                TimeUnit.SECONDS.sleep(3);
            }
         } catch(Exception e) {
            e.printStackTrace();
      }
   }

   private static boolean insertData(int id) {
      // Insert data into the product table with a unique product name so we can find the product again
      String sql = "INSERT INTO SalesLT.Product " +
         "(Name, ProductNumber, Color, StandardCost, ListPrice, SellStartDate) VALUES (?,?,?,?,?,?);";

      try (Connection connection = DriverManager.getConnection(READ_WRITE_URL);
              PreparedStatement pstmt = connection.prepareStatement(sql)) {
         pstmt.setString(1, "BrandNewProduct" + id);
         pstmt.setInt(2, 200989 + id + 10000);
         pstmt.setString(3, "Blue");
         pstmt.setDouble(4, 75.00);
         pstmt.setDouble(5, 89.99);
         pstmt.setTimestamp(6, new Timestamp(new Date().getTime()));
         return (1 == pstmt.executeUpdate());
      } catch (Exception e) {
         return false;
      }
   }

   private static boolean selectData(int id) {
      // Query the data previously inserted into the primary database from the geo replicated database
      String sql = "SELECT Name, Color, ListPrice FROM SalesLT.Product WHERE Name = ?";

      try (Connection connection = DriverManager.getConnection(READ_ONLY_URL);
              PreparedStatement pstmt = connection.prepareStatement(sql)) {
         pstmt.setString(1, "BrandNewProduct" + id);
         try (ResultSet resultSet = pstmt.executeQuery()) {
            return resultSet.next();
         }
      } catch (Exception e) {
         return false;
      }
   }

   private static int getHighWaterMarkId() {
      // Query the high water mark id stored in the table to be able to make unique inserts
      String sql = "SELECT MAX(ProductId) FROM SalesLT.Product";
      int result = 1;
      try (Connection connection = DriverManager.getConnection(READ_WRITE_URL);
              Statement stmt = connection.createStatement();
              ResultSet resultSet = stmt.executeQuery(sql)) {
         if (resultSet.next()) {
             result =  resultSet.getInt(1);
            }
         } catch (Exception e) {
          e.printStackTrace();
         }
         return result;
      }
   }
   ```

1. Guarde y cierre el archivo*App.java*.

1. En la consola de comandos, ejecute el siguiente comando:

   ```bash
   mvn package
   ```

1. Inicie la aplicación que se ejecutará durante aproximadamente una hora hasta que se detenga manualmente, lo que le proporciona tiempo para ejecutar la prueba de conmutación por error.

   ```bash
   mvn -q -e exec:java "-Dexec.mainClass=com.sqldbsamples.App"
   ```

   ```output
   #######################################
   ## GEO DISTRIBUTED DATABASE TUTORIAL ##
   #######################################

   1. insert on primary successful, read from secondary successful
   2. insert on primary successful, read from secondary successful
   3. insert on primary successful, read from secondary successful
   ...
   ```

## <a name="test-failover"></a>Conmutación por error de prueba

Ejecute los siguientes scripts para simular una conmutación por error y observar los resultados de la aplicación. Observe cómo se producirán errores en algunas inserciones y selecciones durante la migración de la base de datos.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Puede comprobar el rol del servidor de recuperación ante desastres durante la prueba con el siguiente comando:

```powershell
(Get-AzSqlDatabaseFailoverGroup -FailoverGroupName $failoverGroup `
    -ResourceGroupName $resourceGroup -ServerName $drServer).ReplicationRole
```

Para probar una conmutación por error:

1. Inicie la conmutación por error manual del grupo de conmutación por error:

   ```powershell
   Switch-AzSqlDatabaseFailoverGroup -ResourceGroupName $myresourcegroupname `
    -ServerName $drServer -FailoverGroupName $failoverGroup
   ```

1. Revierta el grupo de conmutación por error al servidor principal:

   ```powershell
   Switch-AzSqlDatabaseFailoverGroup -ResourceGroupName $resourceGroup `
    -ServerName $server -FailoverGroupName $failoverGroup
   ```

# <a name="the-azure-cli"></a>[La CLI de Azure](#tab/azure-cli)

Puede comprobar el rol del servidor de recuperación ante desastres durante la prueba con el siguiente comando:

```azurecli
az sql failover-group show --name $failoverGroup --resource-group $resourceGroup --server $drServer
```

Para probar una conmutación por error:

1. Inicie la conmutación por error manual del grupo de conmutación por error:

   ```azurecli
   az sql failover-group set-primary --name $failoverGroup --resource-group $resourceGroup --server $drServer
   ```

1. Revierta el grupo de conmutación por error al servidor principal:

   ```azurecli
   az sql failover-group set-primary --name $failoverGroup --resource-group $resourceGroup --server $server
   ```

* * *

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado una base de datos de Azure SQL Database y una aplicación para realizar la conmutación por error a una región remota y, después, ha probado el plan de conmutación por error. Ha aprendido a:

> [!div class="checklist"]
>
> - Crear un grupo de conmutación por error de replicación geográfica
> - Ejecutar una aplicación de Java para consultar una base de datos en SQL Database
> - Conmutación por error de prueba

Avance al siguiente tutorial sobre cómo agregar una instancia de Azure SQL Managed Instance a un grupo de conmutación por error:

> [!div class="nextstepaction"]
> [Adición de una instancia de SQL Managed Instance a un grupo de conmutación por error](../managed-instance/failover-group-add-instance-tutorial.md)
