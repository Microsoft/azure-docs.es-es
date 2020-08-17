---
title: Conexión de SQL a petición a Power BI Desktop y creación de informes
description: En este tutorial, aprenderá a conectar SQL a petición de Azure Synapse Analytics a Power BI Desktop y a crear informes de demostración basados en una vista.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: sql
ms.date: 05/20/2020
ms.author: v-stazar
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: 325a2015e4107a20dfaec22e904cf3cc6ce3085d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089182"
---
# <a name="tutorial-use-sql-on-demand-with-power-bi-desktop--create-a-report"></a>Tutorial: Uso de SQL a petición con Power BI Desktop y creación de informes

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Crear una base de datos de demostración
> - Crear una vista usada para el informe
> - Conectar Power BI Desktop a SQL a petición
> - Crear un informe basado en el conjunto de datos

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, debe cumplir los siguientes requisitos previos:

- [Power BI Desktop](https://powerbi.microsoft.com/downloads/): necesario para visualizar los datos y crear un informe.
- [Área de trabajo de Azure Synapse](https://docs.microsoft.com/azure/synapse-analytics/quickstart-synapse-studio): necesaria para crear la base de datos, el origen de datos externo y la vista.

Opcional:

- Una herramienta de consulta SQL, como [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio) o [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).

Valores de estos parámetros:

| Parámetro                                 | Descripción                                                   |
| ----------------------------------------- | ------------------------------------------------------------- |
| Dirección de un punto de conexión del servicio de SQL a petición.    | Se usa como nombre de servidor.                                   |
| Región de punto de conexión del servicio de SQL a petición.     | Se utiliza para determinar el almacenamiento usado en los ejemplos. |
| Nombre de usuario y contraseña para el acceso al punto de conexión. | Se usa para acceder al punto de conexión.                               |
| Base de datos que se va a usar para crear vistas.     | La base de datos utilizada como punto de partida en los ejemplos.       |

## <a name="1---create-database"></a>1\. Creación de la base de datos

Para el entorno de demostración, cree su propia base de datos de demostración. Use esta base de datos para ver los metadatos, no para almacenar los datos reales.

Cree la base de datos de demostración (y quite una base de datos existente si es necesario); para ello, ejecute el siguiente script de Transact-SQL (T-SQL):

```sql
-- Drop database if it exists
DROP DATABASE IF EXISTS Demo
GO

-- Create new database
CREATE DATABASE [Demo];
GO
```

## <a name="2---create-data-source"></a>2\. Creación de un origen de datos

Se necesita un origen de datos para que el servicio SQL On-Demand acceda a los archivos del almacenamiento. Cree el origen de datos para una cuenta de almacenamiento que se encuentre en la misma región que el punto de conexión. Aunque SQL a petición puede acceder a las cuentas de almacenamiento de distintas regiones, tener el almacenamiento y el punto de conexión en la misma región proporciona un mejor rendimiento.

Para crear el origen de datos, ejecute el siguiente script de Transact-SQL (T-SQL):

```sql
-- There is no credential in data surce. We are using public storage account which doesn't need a secret.
CREATE EXTERNAL DATA SOURCE AzureOpenData
WITH ( LOCATION = 'https://azureopendatastorage.blob.core.windows.net/')
```

## <a name="3---prepare-view"></a>3\. Preparación de la vista

Cree la vista en función de los datos de demostración externos para Power BI que se van a consumir; para ello, ejecute el siguiente script de Transact-SQL (T-SQL):

Cree la vista `usPopulationView` dentro de la base de datos `Demo` con la siguiente consulta:

```sql
DROP VIEW IF EXISTS usPopulationView;
GO

CREATE VIEW usPopulationView AS
SELECT
    *
FROM
    OPENROWSET(
        BULK 'censusdatacontainer/release/us_population_county/year=20*/*.parquet',
        DATA_SOURCE = 'AzureOpenData',
        FORMAT='PARQUET'
    ) AS uspv;
```

Los datos de demostración contienen los siguientes conjuntos de datos:

Población estadounidense por sexo y raza para cada distrito de EE. UU. procedente del censo decenal del 2000 y 2010 en formato Parquet.

| Ruta de acceso a la carpeta                                                  | Descripción                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| /release/                                                    | Carpeta principal para los datos de la cuenta de almacenamiento de demostración               |
| /release/us_population_county/                               | Archivos de datos de población estadounidense en formato Parquet, dividida por año mediante el esquema de partición Hive/Hadoop. |

## <a name="4---create-power-bi-report"></a>4\. Creación de un informe de Power BI

Cree el informe para Power BI Desktop mediante los pasos siguientes:

1. Abra la aplicación Power BI Desktop y seleccione **Obtener datos**.

   ![Abra la aplicación Power BI Desktop y seleccione Obtener datos.](./media/tutorial-connect-power-bi-desktop/step-0-open-powerbi.png)

2. Seleccione **Azure** > **Azure SQL Database**. 

   ![Seleccione el origen de datos.](./media/tutorial-connect-power-bi-desktop/step-1-select-data-source.png)

3. Escriba el nombre del servidor en el que se encuentra la base de datos en el campo **Servidor** y, a continuación, escriba `Demo` en el nombre de la base de datos. Seleccione la opción **Importar** y, a continuación, seleccione **Aceptar**. 

   ![Seleccione la base de datos en el punto de conexión.](./media/tutorial-connect-power-bi-desktop/step-2-db.png)

4. Seleccione el método de autenticación que desee:

    - Ejemplo de AAD 
  
        ![Haga clic en Iniciar sesión.](./media/tutorial-connect-power-bi-desktop/step-2.1-select-aad-auth.png)

    - Ejemplo de inicio de sesión de SQL: escriba su nombre de usuario y contraseña.

        ![Use el inicio de sesión de SQL.](./media/tutorial-connect-power-bi-desktop/step-2.2-select-sql-auth.png)


5. Seleccione la vista `usPopulationView` y, a continuación, seleccione **Cargar**. 

   ![Seleccione una vista en la base de datos seleccionada.](./media/tutorial-connect-power-bi-desktop/step-3-select-view.png)

6. Espere a que se complete la operación y, a continuación, aparecerá un elemento emergente que indica que `There are pending changes in your queries that haven't been applied` (hay cambios pendientes en las consultas que no se han aplicado). Seleccione **Aplicar cambios**. 

   ![Haga clic en Aplicar cambios.](./media/tutorial-connect-power-bi-desktop/step-4-apply-changes.png)

7. Espere a que el cuadro de diálogo **Aplicar los cambios de la consulta** desaparezca, lo que puede tardar unos minutos. 

   ![Espere a que finalice una consulta.](./media/tutorial-connect-power-bi-desktop/step-5-wait-for-query-to-finish.png)

8. Una vez completada la carga, seleccione las columnas siguientes en este orden para crear el informe:
   - countyName
   - rellenado
   - stateName

   ![Seleccione las columnas de interés para generar un informe de mapa.](./media/tutorial-connect-power-bi-desktop/step-6-select-columns-of-interest.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado de usar este informe, elimine los recursos con los pasos siguientes:

1. Eliminación de la credencial de la cuenta de almacenamiento

   ```sql
   DROP EXTENAL DATA SOURCE AzureOpenData
   ```

2. Eliminación de la vista

   ```sql
   DROP VIEW usPopulationView;
   ```

3. Eliminación de la base de datos

   ```sql
   DROP DATABASE Demo;
   ```

## <a name="next-steps"></a>Pasos siguientes

Consulte [Archivos de almacenamiento de consulta](develop-storage-files-overview.md) para aprender a consultar los archivos de almacenamiento mediante Synapse SQL.
