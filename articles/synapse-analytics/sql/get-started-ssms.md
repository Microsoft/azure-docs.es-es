---
title: Conexión a Synapse SQL con SQL Server Management Studio (SSMS)
description: Use SQL Server Management Studio (SSMS) para conectarse a Synapse SQL en Azure Synapse Analytics y realizar consultas.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: ''
ms.date: 04/15/2020
ms.author: v-stazar
ms.reviewer: jrasnick
ms.openlocfilehash: dc467eebbd7346777765af7143d13c76627ab648
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077722"
---
# <a name="connect-to-synapse-sql-with-sql-server-management-studio-ssms"></a>Conexión a Synapse SQL con SQL Server Management Studio (SSMS)
> [!div class="op_single_selector"]
> * [Azure Data Studio](get-started-azure-data-studio.md)
> * [Power BI](get-started-power-bi-professional.md)
> * [Visual Studio](../sql-data-warehouse/sql-data-warehouse-query-visual-studio.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
> * [sqlcmd](../sql/get-started-connect-sqlcmd.md)
> * [SSMS](get-started-ssms.md)
> 
> 

Puede usar [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) para conectarse y realizar consultas a Synapse SQL en Azure Synapse Analytics con los recursos de SQL a petición (versión preliminar) o del grupo de SQL. 

### <a name="supported-tools-for-sql-on-demand-preview"></a>Herramientas compatibles con SQL a petición (versión preliminar)

[Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio) es totalmente compatible a partir de la versión 1.18.0. SSMS es parcialmente compatible a partir de la versión 18.5, que solo se puede usar para establecer conexión y realizar consultas.

> [!NOTE]
> Si el inicio de sesión de AAD tiene abierta la conexión durante más de una hora en el momento de la ejecución de la consulta, se producirá un error en todas las consultas que usen AAD, lo que incluye la realización de consultas en el almacenamiento mediante el paso a través de AAD y las instrucciones que interactúan con AAD (como CREATE EXTERNAL PROVIDER). Esto afecta a todas las herramientas que mantienen la conexión abierta, como en el editor de consultas de SSMS y ADS. Las herramientas que abren una conexión nueva para ejecutar cada consulta no se ven afectadas, como Synapse Studio.
> Para mitigar este problema, puede reiniciar SSMS, o bien conectarse y desconectarse en ADS. .
## <a name="prerequisites"></a>Prerrequisitos

Asegúrese de que cumple los siguientes requisitos previos antes de empezar:  

* [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms). 
* Para el grupo de SQL, necesita un almacén de datos existente. Para crear uno, vea [Creación de un grupo de SQL](../quickstart-create-sql-pool-portal.md). En el caso de SQL a petición, ya se ha aprovisionado uno en el área de trabajo en el momento de la creación. 
* El nombre del servidor SQL completo. Para encontrarlo, consulte [Conexión a Synapse SQL](connect-overview.md).

## <a name="connect"></a>Conectar

### <a name="sql-pool"></a>Grupo de SQL

Para conectarse a Synapse SQL mediante el grupo de SQL, siga estos pasos: 

1. Abra SQL Server Management Studio (SSMS). 
1. En el cuadro de diálogo **Conectar a servidor**, rellene los campos y, a continuación, seleccione **Conectar**: 
  
    ![Conectar con el servidor](../sql-data-warehouse/media/sql-data-warehouse-query-ssms/connect-object-explorer1.png)
   
   * **Nombre del servidor**: Escriba el **nombre del servidor** definido anteriormente.
   * **Autenticación**:  Elija un tipo de autenticación como, por ejemplo, **Autenticación de SQL Server** o **Autenticación integrada de Active Directory**.
   * **Nombre de usuario** y **contraseña**: Escriba su nombre de usuario y la contraseña si la autenticación de SQL Server se seleccionó anteriormente.

1. Expanda el servidor de Azure SQL Server en **Explorador de objetos**. Puede ver las bases de datos asociadas al servidor, como la base de datos AdventureWorksDW de ejemplo. Puede expandir la base de datos para ver las tablas:
   
    ![Explorar AdventureWorksDW](../sql-data-warehouse/media/sql-data-warehouse-query-ssms/explore-tables.png)


### <a name="sql-on-demand-preview"></a>SQL a petición (versión preliminar)

Para conectarse a Synapse SQL mediante SQL a petición, siga estos pasos: 

1. Abra SQL Server Management Studio (SSMS).
1. En el cuadro de diálogo **Conectar a servidor**, rellene los campos y, a continuación, seleccione **Conectar**: 
   
    ![Conectar con el servidor](./media/get-started-ssms/connect-object-explorer1.png)
   
   * **Nombre del servidor**: Escriba el **nombre del servidor** definido anteriormente.
   * **Autenticación**: Elija un tipo de autenticación como, por ejemplo, **Autenticación de SQL Server** o **Autenticación integrada de Active Directory**:
   * **Nombre de usuario** y **contraseña**: Escriba su nombre de usuario y la contraseña si la autenticación de SQL Server se seleccionó anteriormente.
   * Haga clic en **Conectar**.

4. Para explorar, expanda su Azure SQL Server. Puede ver las bases de datos asociadas al servidor. Expanda *demo* para ver el contenido de la base de datos de ejemplo.
   
    ![Explorar AdventureWorksDW](./media/get-started-ssms/explore-tables.png)


## <a name="run-a-sample-query"></a>Ejecución de una consulta de ejemplo

### <a name="sql-pool"></a>Grupo de SQL

Ahora que se ha establecido una conexión de base de datos, puede consultar los datos.

1. Haga clic con el botón derecho en la base de datos en el Explorador de objetos de SQL Server.
2. Seleccione **Nueva consulta**. Se abrirá una nueva ventana de consulta.
   
    ![Nueva consulta](../sql-data-warehouse/media/sql-data-warehouse-query-ssms/new-query.png)
3. Copie esta consulta T-SQL en la ventana de consulta:
   
    ```sql
    SELECT COUNT(*) FROM dbo.FactInternetSales;
    ```
4. Ejecuta la consulta. Para hacerlo, haga clic en `Execute` o use la combinación de teclas `F5`.
   
    ![Ejecutar consulta](../sql-data-warehouse/media/sql-data-warehouse-query-ssms/execute-query.png)
5. Consulte los resultados de la consulta. En este ejemplo, la tabla FactInternetSales tiene 60398 filas.
   
    ![Resultados de la consulta](../sql-data-warehouse/media/sql-data-warehouse-query-ssms/results.png)

### <a name="sql-on-demand"></a>SQL a petición

Ahora que se ha establecido una conexión de base de datos, puede consultar los datos.

1. Haga clic con el botón derecho en la base de datos en el Explorador de objetos de SQL Server.
2. Seleccione **Nueva consulta**. Se abrirá una nueva ventana de consulta.
   
    ![Nueva consulta](./media/get-started-ssms/new-query.png)
3. Copie la siguiente consulta T-SQL en la ventana de consulta:
   
    ```sql
    SELECT COUNT(*) FROM demo.dbo.usPopulationView
    ```
4. Ejecuta la consulta. Para hacerlo, haga clic en `Execute` o use la combinación de teclas `F5`.
   
    ![Ejecutar consulta](./media/get-started-ssms/execute-query.png)
5. Consulte los resultados de la consulta. En este ejemplo, la vista usPopulationView tiene 3664512 filas.
   
    ![Resultados de la consulta](./media/get-started-ssms/results.png)

## <a name="next-steps"></a>Pasos siguientes
Ahora que puede conectarse y realizar consultas, intente realizar la [Visualización de datos con Power BI](get-started-power-bi-professional.md).

Para configurar el entorno para la autenticación de Azure Active Directory, consulte [Autenticación en Synapse SQL](../sql-data-warehouse/sql-data-warehouse-authentication.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

