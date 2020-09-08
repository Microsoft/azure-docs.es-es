---
title: Uso de Visual Studio con .NET y C# para realizar consultas
description: Use Visual Studio para crear una aplicación en C# que se conecte a una base de datos de Azure SQL Database o Instancia administrada de Azure SQL y ejecuta consultas.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: devx-track-csharp, sqldbrb=2
ms.devlang: dotnet
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 08/10/2020
ms.openlocfilehash: 8fe541432366d3c2ac1dc1470fea66d328f79780
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213040"
---
# <a name="quickstart-use-net-and-c-in-visual-studio-to-connect-to-and-query-a-database-in-azure-sql-database-or-azure-sql-managed-instance"></a>Inicio rápido: Uso de .NET y C# en Visual Studio para conectarse a una base de datos de Azure SQL Database o Instancia administrada de Azure SQL y consultarla
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

En este inicio rápido se muestra cómo usar [.NET Framework](https://www.microsoft.com/net/) y el código de C# en Visual Studio para consultar una base de datos de Azure SQL Database con instrucciones Transact-SQL.

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido necesita instalar:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- Una base de datos de Azure SQL Database. Puede utilizar uno de estos inicios rápidos para crear y configurar una base de datos en Azure SQL Database:

  | Acción | SQL Database | Instancia administrada de SQL | SQL Server en máquina virtual de Azure |
  |:--- |:--- |:---|:---|
  | Crear| [Portal](single-database-create-quickstart.md) | [Portal](../managed-instance/instance-create-quickstart.md) | [Portal](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  || [CLI](scripts/create-and-configure-database-cli.md) | [CLI](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44) |
  || [PowerShell](scripts/create-and-configure-database-powershell.md) | [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | [PowerShell](../virtual-machines/windows/sql-vm-create-powershell-quickstart.md)
  | Configuración | [Regla de firewall de IP en el nivel de servidor](firewall-create-server-level-portal-quickstart.md)| [Conectividad desde una máquina virtual](../managed-instance/connect-vm-instance-configure.md)|
  |||[Conectividad desde un entorno local](../managed-instance/point-to-site-p2s-configure.md) | [Conectarse a SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  |Cargar datos|Adventure Works cargado por inicio rápido|[Restauración de World Wide Importers](../managed-instance/restore-sample-database-quickstart.md) | [Restauración de World Wide Importers](../managed-instance/restore-sample-database-quickstart.md) |
  |||Restauración o importación de Adventure Works desde un archivo [BACPAC](database-import.md) desde [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)| Restauración o importación de Adventure Works desde un archivo [BACPAC](database-import.md) desde [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)|
  |||

  > [!IMPORTANT]
  > Los scripts de este artículo se escriben para utilizar la base de datos Adventure Works. Con una instancia administrada de SQL, debe importar la base de datos Adventure Works en una base de datos de instancia, o bien modificar los scripts de este artículo para utilizar la base de datos Wide World Importers.

- [Visual Studio 2019](https://www.visualstudio.com/downloads/) Community, Professional o Enterprise.

## <a name="get-server-connection-information"></a>Obtención de información de conexión del servidor

Obtenga la información de conexión que necesita para conectarse a la base de datos. En los procedimientos siguientes, necesitará el nombre completo del servidor o nombre de host, el nombre de la base de datos y la información de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Vaya a las páginas **Bases de datos SQL** o **Instancias administradas de SQL**.

3. En la página **Información general**, revise el nombre completo del servidor junto a **Nombre del servidor** para una base de datos de Azure SQL Database o el nombre completo (o la dirección IP) del servidor junto a **Host** para una instancia administrada de Azure SQL o SQL Server en la máquina virtual de Azure. Para copiar el nombre del servidor o nombre de host, mantenga el cursor sobre él y seleccione el icono **Copiar**.

> [!NOTE]
> Para obtener información de la conexión de SQL Server en una máquina virtual de Azure, consulte [Conexión a una instancia de SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md#connect-to-sql-server).

## <a name="create-code-to-query-the-database-in-azure-sql-database"></a>Creación de código para consultar la base de datos en Azure SQL Database

1. En Visual Studio, cree un nuevo proyecto. 
   
1. En el cuadro de diálogo **Nuevo proyecto**, seleccione **Visual C#** y **Aplicación de consola (.NET Framework)** .
   
1. Escriba *sqltest* en el nombre del proyecto y, luego, seleccione **Aceptar**. El proyecto nuevo se crea. 
   
1. Seleccione **Proyecto** > **Administrar paquetes NuGet**. 
   
1. En **Administrador de paquetes NuGet**, seleccione la pestaña **Examinar** y busque y seleccione **Microsoft.Data.SqlClient**.
   
1. En la página **Microsoft.Data.SqlClient**, seleccione **Instalar**. 
   - Si se le solicita, seleccione **Aceptar** para seguir con la instalación. 
   - Si aparece la ventana **Aceptación de la licencia**, seleccione **Acepto**.
   
1. Cuando se complete la instalación, puede cerrar el **Administrador de paquetes NuGet**. 
   
1. En el editor de código, reemplace el contenido de **Program.cs** por el código siguiente. Reemplace los valores de `<server>`, `<username>`, `<password>` y `<database>`.
   
   >[!IMPORTANT]
   >El código de este ejemplo usa los datos de AdventureWorksLT de ejemplo, que puede elegir como origen al crear la base de datos. Si la base de datos tiene datos distintos, use las tablas de su propia base de datos en la consulta SELECT. 
   
   ```csharp
   using System;
   using Microsoft.Data.SqlClient;
   using System.Text;
   
   namespace sqltest
   {
       class Program
       {
           static void Main(string[] args)
           {
               try 
               { 
                   SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder();
                   builder.DataSource = "<server>.database.windows.net"; 
                   builder.UserID = "<username>";            
                   builder.Password = "<password>";     
                   builder.InitialCatalog = "<database>";
   
                   using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                   {
                       Console.WriteLine("\nQuery data example:");
                       Console.WriteLine("=========================================\n");
                       
                       StringBuilder sb = new StringBuilder();
                       sb.Append("SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName ");
                       sb.Append("FROM [SalesLT].[ProductCategory] pc ");
                       sb.Append("JOIN [SalesLT].[Product] p ");
                       sb.Append("ON pc.productcategoryid = p.productcategoryid;");
                       String sql = sb.ToString();
   
                       using (SqlCommand command = new SqlCommand(sql, connection))
                       {
                           connection.Open();
                           using (SqlDataReader reader = command.ExecuteReader())
                           {
                               while (reader.Read())
                               {
                                   Console.WriteLine("{0} {1}", reader.GetString(0), reader.GetString(1));
                               }
                           }
                       }                    
                   }
               }
               catch (SqlException e)
               {
                   Console.WriteLine(e.ToString());
               }
               Console.ReadLine();
           }
       }
   }
   ```

## <a name="run-the-code"></a>Ejecución del código

1. Para ejecutar la aplicación, seleccione **Depurar** > **Iniciar depuración**, o bien seleccione **Iniciar** en la barra de herramientas o presione **F5**.
1. Compruebe que se devuelven las 20 primeras filas de categoría o productos de la base de datos y cierre la ventana de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [conectarse a una base de datos de Azure SQL Database y a consultarla mediante .NET Core](connect-query-dotnet-core.md) en Windows, Linux y macOS.  
- Para más información, consulte [Introducción a .NET Core en Windows, Linux y macOS mediante la línea de comandos](/dotnet/core/tutorials/using-with-xplat-cli).
- Aprenda a [diseñar su primera base de datos de Azure SQL Database mediante SSMS](design-first-database-tutorial.md) o a [diseñar su primera base de datos de Azure SQL Database mediante .NET](design-first-database-csharp-tutorial.md).
- Para más información acerca de. NET, consulte la [Documentación de .NET](https://docs.microsoft.com/dotnet/).
- Ejemplo de lógica de reintento: [Conexión resistente a Azure SQL con ADO.NET][step-4-connect-resiliently-to-sql-with-ado-net-a78n].


<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-ado-net-a78n]: https://docs.microsoft.com/sql/connect/ado-net/step-4-connect-resiliently-sql-ado-net

