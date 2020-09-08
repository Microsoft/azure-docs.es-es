---
title: Consulta de Apache Hive con ODBC Driver y PowerShell en Azure HDInsight
description: Utilice Microsoft Hive ODBC Driver y PowerShell para consultar los clústeres de Apache Hive en Azure HDInsight.
keywords: hive,hive odbc,powershell
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.date: 06/27/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 211b23db56b6deebb519b9eaac4164ea83edabac
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078578"
---
# <a name="tutorial-query-apache-hive-with-odbc-and-powershell"></a>Tutorial: Consulta de Apache Hive con ODBC y PowerShell

Los controladores ODBC de Microsoft proporcionan una manera flexible de interactuar con distintos tipos de orígenes de datos, como Apache Hive. Puede escribir código en lenguajes de scripting como PowerShell que utilizan los controladores ODBC para abrir una conexión con el clúster de Hive, pasar una consulta de su elección y mostrar los resultados.

En este tutorial, realizará las tareas siguientes:

> [!div class="checklist"]
> * Descargar e instalar Microsoft Hive ODBC Driver
> * Crear un origen de datos ODBC de Apache Hive vinculado al clúster
> * Consultar la información de ejemplo desde el clúster mediante PowerShell

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar este tutorial, debe contar con lo siguiente:

* Un clúster de Interactive Query en HDInsight. Para crear uno, vea [Introducción a HDInsight de Azure](../hdinsight-hadoop-provision-linux-clusters.md). Seleccione **Interactive Query** como el tipo de clúster.

## <a name="install-microsoft-hive-odbc-driver"></a>Instalación de Microsoft Hive ODBC Driver

Descargue e instale [Microsoft Hive ODBC Driver](https://www.microsoft.com/download/details.aspx?id=40886).

## <a name="create-apache-hive-odbc-data-source"></a>Creación de un origen de datos de Apache Hive ODBC

En los siguientes pasos se explica cómo crear un origen de datos ODBC de Apache Hive.

1. En Windows, vaya a **Inicio** > **Herramientas administrativas de Windows** > **Orígenes de datos ODBC (32 bits)/(64 bits)** .  Se abre la ventana **Administrador de orígenes de datos ODBC**.

    ![Administrador de orígenes de datos OBDC](./media/apache-hive-query-odbc-driver-powershell/hive-odbc-driver-dsn-setup.png "Configuración de un DSN mediante el Administrador de orígenes de datos ODBC")

1. Desde la pestaña **DSN del usuario**, seleccione **Agregar** para abrir la ventana **Crear nuevo origen de datos**.

1. Seleccione **Microsoft Hive ODBC Driver** y, luego, seleccione **Finalizar** para abrir la ventana **Microsoft Hive ODBC Driver DSN Setup** (Configuración de DSN de Microsoft Hive ODBC Driver).

1. Escriba o seleccione los valores siguientes:

   | Propiedad | Descripción |
   | --- | --- |
   |  Data Source Name |Asigne un nombre al origen de datos |
   |  Host(s) |Escriba `CLUSTERNAME.azurehdinsight.net`. Por ejemplo: `myHDICluster.azurehdinsight.net` |
   |  Port |Use **443**.|
   |  Base de datos |Use el **valor predeterminado**. |
   |  Mechanism |Seleccione **Servicio HDInsight de Microsoft Azure**. |
   |  Nombre de usuario |Escriba el nombre de usuario HTTP del clúster de HDInsight. El nombre de usuario predeterminado es **admin**. |
   |  Contraseña |Escriba la contraseña del usuario del clúster de HDInsight. Seleccione la casilla **Save Password (Encrypted)** [Guardar contraseña (cifrada)].|

1. Opcional: Seleccione **Opciones avanzadas**.  

   | Parámetro | Descripción |
   | --- | --- |
   |  Use Native Query |Cuando esta opción está seleccionada, el controlador ODBC NO trata de convertir TSQL en HiveQL. Utilice esta opción solo si está 100 % seguro de que está enviando instrucciones HiveQL puras. Al conectarse a SQL Server o a Azure SQL Database, debe dejar esta opción desactivada. |
   |  Rows fetched per block |Al capturar un gran volumen de registros, es posible que sea necesario ajustar este parámetro para garantizar un rendimiento óptimo. |
   |  Default string column length, Binary column length, Decimal column scale |La longitud y precisión del tipo de datos pueden afectar a la forma en que se devuelven los datos. Pueden dar lugar a que se devuelva información incorrecta debido a la pérdida de precisión o al truncamiento. |

    ![Opciones de configuración avanzada de DSN](./media/apache-hive-query-odbc-driver-powershell/odbc-data-source-advanced-options.png "Opciones de configuración avanzada de DSN")

1. Seleccione **Probar** para probar el origen de datos. Cuando el origen de datos esté configurado correctamente, el resultado de la prueba mostrará **SUCCESS** (Correcto).  

1. Seleccione **Aceptar** para cerrar la ventana Probar.  

1. Seleccione **Aceptar** para cerrar la ventana **Microsoft Hive ODBC Driver DSN Setup**.  

1. Seleccione **Aceptar** para cerrar la ventana **Administrador de orígenes de datos ODBC**.  

## <a name="query-data-with-powershell"></a>Consulta de datos con PowerShell

El siguiente script de PowerShell es una función que utiliza ODBC para consultar un clúster de Hive.

```powershell
function Get-ODBC-Data {

   param(
   [string]$query=$(throw 'query is required.'),
   [string]$dsn,  
   [PSCredential] $cred = (Get-Credential)  
   )

   $conn = New-Object System.Data.Odbc.OdbcConnection
   $uname = $cred.UserName

   $pswd = (New-Object System.Net.NetworkCredential -ArgumentList "", $cred.Password).Password
   $conn.ConnectionString = "DSN=$dsn;Uid=$uname;Pwd=$pswd;"
   $conn.open()
   $cmd = New-object System.Data.Odbc.OdbcCommand($query,$conn)

   $ds = New-Object system.Data.DataSet

   (New-Object system.Data.odbc.odbcDataAdapter($cmd)).fill($ds) #| out-null
   $conn.close()
   $ds.Tables
}
```

El siguiente fragmento de código utiliza la función anterior para ejecutar una consulta en el clúster de Interactive Query que creó al principio del tutorial. Reemplace `DATASOURCENAME` por el **nombre del origen de datos** que especificó en la pantalla de **configuración de DNS de Microsoft Hive ODBC Driver**. Cuando se le soliciten las credenciales, escriba el nombre de usuario y la contraseña que especificó en **Nombre de usuario de inicio de sesión del clúster**y **Contraseña de inicio de sesión del clúster** cuando creó el clúster.

```powershell

$dsn = "DATASOURCENAME"

$query = "select count(distinct clientid) AS total_clients from hivesampletable"

Get-ODBC-Data -query $query -dsn $dsn
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine el grupo de recursos, el clúster de HDInsight y la cuenta de almacenamiento. Para ello, seleccione el grupo de recursos donde se creó el clúster y haga clic en **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a usar Microsoft Hive ODBC Driver y PowerShell para recuperar datos desde el clúster de Interactive Query de Azure HDInsight.

> [!div class="nextstepaction"]
> [Conexión de Excel a Apache Hive mediante ODBC](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md)
