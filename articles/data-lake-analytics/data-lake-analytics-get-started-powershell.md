---
title: 'Creación y consulta de Azure Data Lake Analytics: PowerShell'
description: Use Azure PowerShell para crear una cuenta de Azure Data Lake Analytics y enviar un trabajo de U-SQL.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: 8a4e901e-9656-4a60-90d0-d78ff2f00656
ms.topic: conceptual
ms.date: 05/04/2017
ms.openlocfilehash: ccee8cef9cfcfc57ac19284b9c42e5c8371aa746
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132558"
---
# <a name="get-started-with-azure-data-lake-analytics-using-azure-powershell"></a>Introducción a Azure Data Lake Analytics mediante Azure PowerShell

[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

Aprenda a usar Azure PowerShell para crear cuentas de Azure Data Lake Analytics y luego enviar y ejecutar trabajos de U-SQL. Para obtener más información acerca de Análisis de Data Lake, consulte [Información general sobre Análisis de Azure Data Lake](data-lake-analytics-overview.md).

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Antes de empezar este tutorial, debe tener la siguiente información:

* **Una cuenta de Análisis de Azure Data Lake**. Consulte [Introducción a Data Lake Analytics](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-get-started-portal).
* **Una estación de trabajo con Azure PowerShell**. Consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

En este tutorial se asume que sabe usar Azure PowerShell. En concreto, debe saber iniciar sesión en Azure. Si necesita ayuda, consulte [Get started with Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps) (Introducción a Azure PowerShell).

Para iniciar sesión con un nombre de suscripción:

```powershell
Connect-AzAccount -SubscriptionName "ContosoSubscription"
```

En lugar del nombre de la suscripción, también puede utilizar un identificador de suscripción para iniciar sesión:

```powershell
Connect-AzAccount -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

Si se realiza correctamente, la salida de este comando es similar a la siguiente:

```text
Environment           : AzureCloud
Account               : joe@contoso.com
TenantId              : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
SubscriptionId        : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
SubscriptionName      : ContosoSubscription
CurrentStorageAccount :
```

## <a name="preparing-for-the-tutorial"></a>Preparativos para el tutorial

Los fragmentos de código de PowerShell de este tutorial usan estas variables para almacenar esta información:

```powershell
$rg = "<ResourceGroupName>"
$adls = "<DataLakeStoreAccountName>"
$adla = "<DataLakeAnalyticsAccountName>"
$location = "East US 2"
```

## <a name="get-information-about-a-data-lake-analytics-account"></a>Información acerca de una cuenta de Data Lake Analytics

```powershell
Get-AdlAnalyticsAccount -ResourceGroupName $rg -Name $adla  
```

## <a name="submit-a-u-sql-job"></a>Envío de un trabajo de U-SQL

Cree una variable de PowerShell para almacenar el script U-SQL.

```powershell
$script = @"
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();

"@
```

Envíe el texto del script con el cmdlet `Submit-AdlJob` y el parámetro `-Script`.

```powershell
$job = Submit-AdlJob -Account $adla -Name "My Job" -Script $script
```

Como alternativa, puede enviar un archivo de script mediante el parámetro `-ScriptPath`:

```powershell
$filename = "d:\test.usql"
$script | out-File $filename
$job = Submit-AdlJob -Account $adla -Name "My Job" -ScriptPath $filename
```

Obtenga el estado de un trabajo con `Get-AdlJob`. 

```powershell
$job = Get-AdlJob -Account $adla -JobId $job.JobId
```

En lugar de llamar a Get-AdlJob una y otra vez hasta que finalice un trabajo, use el cmdlet `Wait-AdlJob`.

```powershell
Wait-AdlJob -Account $adla -JobId $job.JobId
```

Descargue el archivo de salida mediante `Export-AdlStoreItem`.

```powershell
Export-AdlStoreItem -Account $adls -Path "/data.csv" -Destination "C:\data.csv"
```

## <a name="see-also"></a>Consulte también

* Para ver el mismo tutorial con otras herramientas, haga clic en los selectores de pestañas en la parte superior de la página.
* Para obtener más información sobre U-SQL, consulte [Introducción al lenguaje U-SQL de Análisis de Azure Data Lake](data-lake-analytics-u-sql-get-started.md).
* Para conocer las tareas de administración, consulte [Administración de Azure Data Lake Analytics mediante el Azure Portal](data-lake-analytics-manage-use-portal.md).
