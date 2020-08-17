---
title: Uso de Python para consultar una base de datos
description: En este tema se muestra cómo usar Python para crear un programa que se conecta a una base de datos de Azure SQL Database y realiza consultas mediante instrucciones Transact-SQL.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: seo-python-october2019, sqldbrb=2, devx-track-python
ms.devlang: python
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 05/29/2020
ms.openlocfilehash: 5257e8730830c6b51808d5d014bc3bdec6036bfe
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87843987"
---
# <a name="quickstart-use-python-to-query-a-database-in-azure-sql-database-or-azure-sql-managed-instance"></a>Inicio rápido: Uso de Python para consultar una base de datos de Azure SQL Database o Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

En este inicio rápido se utiliza Python para conectarse a Azure SQL Database o Instancia administrada de Azure SQL y se usan instrucciones T-SQL para consultar los datos.

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido necesita instalar:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

  | Acción | SQL Database | Instancia administrada de SQL | SQL Server en máquina virtual de Azure |
  |:--- |:--- |:---|:---|
  | Crear| [Portal](single-database-create-quickstart.md) | [Portal](../managed-instance/instance-create-quickstart.md) | [Portal](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  || [CLI](scripts/create-and-configure-database-cli.md) | [CLI](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44) |
  || [PowerShell](scripts/create-and-configure-database-powershell.md) | [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | [PowerShell](../virtual-machines/windows/sql-vm-create-powershell-quickstart.md)
  | Configuración | [Regla de firewall de IP en el nivel de servidor](firewall-create-server-level-portal-quickstart.md)| [Conectividad desde una máquina virtual](../managed-instance/connect-vm-instance-configure.md)|
  |||[Conectividad desde un entorno local](../managed-instance/point-to-site-p2s-configure.md) | [Conexión a una instancia de SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  |Cargar datos|Adventure Works cargado por inicio rápido|[Restauración de World Wide Importers](../managed-instance/restore-sample-database-quickstart.md) | [Restauración de World Wide Importers](../managed-instance/restore-sample-database-quickstart.md) |
  |||Restauración o importación de Adventure Works desde un archivo [BACPAC](database-import.md) desde [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)| Restauración o importación de Adventure Works desde un archivo [BACPAC](database-import.md) desde [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)|
  |||

- [Python](https://python.org/downloads) 3 y software relacionado

  # <a name="macos"></a>[macOS](#tab/macos)

  Para instalar Homebrew y Python, el controlador ODBC y SQLCMD, y el controlador de Python para SQL Server, use los pasos **1.2**, **1.3** y **2.1** en [Creación de aplicaciones de Python con SQL Server en macOS](https://www.microsoft.com/sql-server/developer-get-started/python/mac/).

  Para más información, consulte [Microsoft ODBC Driver en macOS](/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server).

  # <a name="ubuntu"></a>[Ubuntu](#tab/ubuntu)

  Para instalar Python y otros paquetes necesarios, use `sudo apt-get install python python-pip gcc g++ build-essential`.

  Para instalar el controlador ODBC, SQLCMD y el controlador de Python para SQL Server, consulte el artículo para [configurar un entorno para el desarrollo de pyodbc Python](/sql/connect/python/pyodbc/step-1-configure-development-environment-for-pyodbc-python-development#linux).

  Para más información, consulte [Microsoft ODBC Driver en Linux](/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server).

  # <a name="windows"></a>[Windows](#tab/windows)

  Para instalar Python, el controlador ODBC, SQLCMD y el controlador de Python para SQL Server, consulte el artículo para [configurar un entorno para el desarrollo de pyodbc Python](/sql/connect/python/pyodbc/step-1-configure-development-environment-for-pyodbc-python-development#windows).

  Para más información, consulte [Microsoft ODBC Driver](/sql/connect/odbc/microsoft-odbc-driver-for-sql-server).

---

> [!IMPORTANT]
> Los scripts de este artículo se escriben para utilizar la base de datos **Adventure Works**.

> [!NOTE]
> También puede elegir una instancia administrada de Azure SQL.
>
> Para crearlo y configurarlo, use [Azure Portal](../managed-instance/instance-create-quickstart.md), [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md)o la [CLI](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44) y, después, configure una conectividad [local](../managed-instance/point-to-site-p2s-configure.md) o de [máquina virtual](../managed-instance/connect-vm-instance-configure.md).
>
> Para cargar los datos, consulte la [restauración con BACPAC](database-import.md) con el archivo [Adventure Works](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works) o consulte la [restauración de la base de datos de Wide World Importers](../managed-instance/restore-sample-database-quickstart.md).

Para explorar más Python y la base de datos de Azure SQL Database, consulte [Bibliotecas Azure SQL Database para Python](/python/api/overview/azure/sql), el [repositorio de pyodbc](https://github.com/mkleehammer/pyodbc/wiki/) y un [ejemplo de pyodbc](https://github.com/mkleehammer/pyodbc/wiki/Getting-started).

## <a name="get-server-connection-information"></a>Obtención de información de conexión del servidor

Obtenga la información de conexión necesaria para conectarse a la base de datos de Azure SQL Database. En los procedimientos siguientes, necesitará el nombre completo del servidor o nombre de host, el nombre de la base de datos y la información de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Vaya a las páginas **Bases de datos SQL** o **Instancias administradas de SQL**.

3. En la página **Información general**, revise el nombre completo del servidor junto a **Nombre del servidor** para la base de datos de Azure SQL Database o el nombre completo (o la dirección IP) del servidor junto a **Host** para una instancia administrada de Azure SQL o SQL Server en la máquina virtual de Azure. Para copiar el nombre del servidor o nombre de host, mantenga el cursor sobre él y seleccione el icono **Copiar**.

> [!NOTE]
> Para obtener información de la conexión de SQL Server en una máquina virtual de Azure, consulte [Conexión a una instancia de SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md#connect-to-sql-server).

## <a name="create-code-to-query-your-database"></a>Creación de código para consultar la base de datos 

1. En el editor de texto, cree un archivo llamado *sqltest.py*.  
   
1. Agregue el código siguiente: Sustituya \<server>, \<database>, \<username> y \<password> por sus propios valores.
   
   >[!IMPORTANT]
   >El código de este ejemplo usa los datos de AdventureWorksLT de ejemplo, que puede elegir como origen al crear la base de datos. Si la base de datos tiene datos distintos, use las tablas de su propia base de datos en la consulta SELECT. 
   
   ```python
   import pyodbc
   server = '<server>.database.windows.net'
   database = '<database>'
   username = '<username>'
   password = '<password>'   
   driver= '{ODBC Driver 17 for SQL Server}'
   
   with pyodbc.connect('DRIVER='+driver+';SERVER='+server+';PORT=1433;DATABASE='+database+';UID='+username+';PWD='+ password) as conn:
       with conn.cursor() as cursor:
           cursor.execute("SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName FROM [SalesLT].[ProductCategory] pc JOIN [SalesLT].[Product] p ON pc.productcategoryid = p.productcategoryid")
           row = cursor.fetchone()
           while row:
               print (str(row[0]) + " " + str(row[1]))
               row = cursor.fetchone()
   ```
   

## <a name="run-the-code"></a>Ejecución del código

1. En el símbolo del sistema, ejecute el siguiente comando:

   ```cmd
   python sqltest.py
   ```

1. Compruebe que se devuelven las 20 primeras filas de categoría o productos y, después, cierre la ventana de comandos.

## <a name="next-steps"></a>Pasos siguientes

- [Diseño de la primera base de datos en Azure SQL Database](design-first-database-tutorial.md)
- [Controladores de Microsoft para Python para SQL Server](https://docs.microsoft.com/sql/connect/python/python-driver-for-sql-server/)
- [Centro para desarrolladores de Python](https://azure.microsoft.com/develop/python/?v=17.23h)

