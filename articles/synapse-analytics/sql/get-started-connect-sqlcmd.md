---
title: Conexión a Synapse SQL mediante sqlcmd
description: Use la utilidad de línea de comandos sqlcmd para conectarse a SQL a petición (versión preliminar) y a un grupo de SQL y realizar consultas en ellos.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: ''
ms.date: 04/15/2020
ms.author: v-stazar
ms.reviewer: jrasnick
ms.openlocfilehash: 1af42a838463132a241447c11b90bfb489f879ac
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87059466"
---
# <a name="connect-to-synapse-sql-with-sqlcmd"></a>Conexión a Synapse SQL mediante sqlcmd

> [!div class="op_single_selector"]
> * [Azure Data Studio (versión preliminar)](get-started-azure-data-studio.md)
> * [Power BI](get-started-power-bi-professional.md)
> * [Visual Studio](../sql-data-warehouse/sql-data-warehouse-query-visual-studio.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
> * [sqlcmd](../sql/get-started-connect-sqlcmd.md)
> * [SSMS](get-started-ssms.md)

Puede usar la utilidad de línea de comandos [sqlcmd](/sql/tools/sqlcmd-utility?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) para conectarse a SQL a petición (versión preliminar) y a un grupo de SQL, y realizar consultas en Synapse SQL.  

## <a name="1-connect"></a>1. Conectar
Para empezar a trabajar con [sqlcmd](/sql/tools/sqlcmd-utility?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest), abra el símbolo del sistema y escriba **sqlcmd** seguido de la cadena de conexión de la base de datos de Synapse SQL. La cadena de conexión requiere los siguientes parámetros:

* **Server (-S):** servidor con el formato `<`Nombre de servidor`>`.database.windows.net
* **Database (-d):** Nombre de la base de datos
* **Enable Quoted Identifiers (-I):** Los identificadores entre comillas tienen que estar habilitados para poder conectarse a una instancia de Synapse SQL.

Para utilizar la autenticación de SQL Server, debe agregar los parámetros de nombre de usuario y contraseña:

* **User (-U):** usuario del servidor con el formato `<`Usuario`>`
* **Password (-P):** contraseña asociada con el usuario.

La cadena de conexión podría ser similar a la siguiente:

**SQL a petición**

```sql
C:\>sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P Enter_Your_Password_Here -I
```

**Grupo de SQL**

```
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
```

Para usar la autenticación integrada de Azure Active Directory, debe agregar los parámetros de Azure Active Directory:

* **Autenticación de Azure Active Directory (-G):** use Azure Active Directory para la autenticación.

La cadena de conexión podría ser similar a la de los siguientes ejemplos:

**SQL a petición**

```
C:\>sqlcmd -S partyeunrt.database.windows.net -d demo -G -I
```

**Grupo de SQL**

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -G -I
```

> [!NOTE]
> Necesita [habilitar la autenticación de Azure Active Directory](../sql-data-warehouse/sql-data-warehouse-authentication.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) para autenticarse con Active Directory.

## <a name="2-query"></a>2. Consultar

### <a name="use-sql-pool"></a>Uso de un grupo de SQL

Después de la conexión, puede emitir cualquier instrucción [Transact-SQL](/sql/t-sql/language-reference?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) (T-SQL) en la instancia. En este ejemplo, las consultas se envían en modo interactivo:

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
1> SELECT name FROM sys.tables;
2> GO
3> QUIT
```

En el grupo de SQL, los ejemplos siguientes le muestran cómo ejecutar consultas en el modo por lotes mediante la opción -Q o mediante la canalización de su SQL a sqlcmd:

```sql
sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I -Q "SELECT name FROM sys.tables;"
```

```sql
"SELECT name FROM sys.tables;" | sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I > .\tables.out
```

### <a name="use-sql-on-demand"></a>Uso de SQL a petición

Después de la conexión, puede emitir cualquier instrucción [Transact-SQL](/sql/t-sql/language-reference?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) (T-SQL) en la instancia.  En el siguiente ejemplo, las consultas se envían en modo interactivo:

```sql
C:\>sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P Enter_Your_Password_Here -I
1> SELECT COUNT(*) FROM  OPENROWSET(BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet', FORMAT='PARQUET')
2> GO
3> QUIT
```

En el caso de SQL a petición, los ejemplos siguientes muestran cómo ejecutar consultas en el modo por lotes mediante la opción -Q o la canalización de SQL a sqlcmd:

```sql
sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P 'Enter_Your_Password_Here' -I -Q "SELECT COUNT(*) FROM  OPENROWSET(BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet', FORMAT='PARQUET')"
```

```sql
"SELECT COUNT(*) FROM  OPENROWSET(BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet', FORMAT='PARQUET')" | sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P 'Enter_Your_Password_Here' -I > ./tables.out
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las opciones de sqlcmd, consulte la [documentación de sqlcmd](/sql/tools/sqlcmd-utility?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).
