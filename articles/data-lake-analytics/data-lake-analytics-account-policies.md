---
title: Administración de directivas de cuentas de Azure Data Lake Analytics
description: Aprenda a usar directivas de cuentas para controlar el uso de una cuenta de Data Lake Analytics, como las unidades de análisis y los trabajos máximos.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: 0a6102d1-7554-4df2-b487-4dae9a7287b6
ms.topic: how-to
ms.date: 04/30/2018
ms.openlocfilehash: cf64424d1d422e599585b76fc068c940f9311b05
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87127713"
---
# <a name="manage-azure-data-lake-analytics-using-account-policies"></a>Administración de Azure Data Lake Analytics con directivas de cuentas

Con las directivas de cuentas, puede controlar cómo se usan los recursos de una cuenta de Azure Data Lake Analytics. Estas directivas le permiten controlar el costo de usar Azure Data Lake Analytics. Por ejemplo, con estas directivas puede evitar subidas de costo inesperadas al limitar cuántas unidades de análisis (AU) puede usar a la vez la cuenta.## Directivas de nivel de cuenta

Estas directivas se aplican a todos los trabajos de una cuenta de Data Lake Analytics.### Número máximo de unidades de análisis en una cuenta de Data Lake Analytics Una directiva controla el número total de unidades de análisis (AU) que puede usar la cuenta de Data Lake Analytics. De forma predeterminada, el valor se establece en 250. Por ejemplo, si este valor está establecido en 250 unidades, puede tener un trabajo en ejecución con 250 unidades asignadas o 10 trabajos en ejecución con 25 unidades cada uno. Los trabajos adicionales que se envían se ponen en cola hasta que finalizan los trabajos en ejecución. Cuando terminan los trabajos en ejecución, se liberan unidades de análisis para la ejecución de los trabajos en cola.

Para cambiar el número de unidades de análisis de la cuenta de Data Lake Analytics:

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.
2. Haga clic en **Límites y directivas**.
3. En **Número de AU máximo**, mueva el control deslizante para seleccionar un valor o escríbalo en el cuadro de texto. 
4. Haga clic en **Save**(Guardar).

   > [!NOTE]
   > Si necesita más unidades de análisis que el valor predeterminado (250), haga clic en **Ayuda y soporte técnico** en el portal para enviar una solicitud de soporte técnico. Es posible aumentar el número de unidades de análisis disponibles en la cuenta de Data Lake Analytics.

### <a name="maximum-number-of-jobs-that-can-run-simultaneously"></a>Número máximo de trabajos que se pueden ejecutar simultáneamente
Esta directiva limita el número de trabajos que se pueden ejecutar simultáneamente. De forma predeterminada, este valor se establece en 20. Si Data Lake Analytics tiene unidades de análisis disponibles, los trabajos nuevos se programan para ejecutarse de inmediato hasta que el número total de trabajos en ejecución alcanza el valor de esta directiva. Cuando se alcanza el número máximo de trabajos que se pueden ejecutar al mismo tiempo, los trabajos posteriores se ponen en cola en orden de prioridad hasta que finalizan uno o varios de los trabajos que están en ejecución (según las unidades de análisis disponibles).

Para cambiar el número de trabajos que se pueden ejecutar simultáneamente:

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.
2. Haga clic en **Límites y directivas**.
3. En **Número máximo de trabajos en ejecución**, mueva el control deslizante para seleccionar un valor o escríbalo en el cuadro de texto. 
4. Haga clic en **Save**(Guardar).

   > [!NOTE]
   > Si necesita ejecutar más que el número predeterminado de trabajos (20), en el portal, haga clic en **Ayuda y soporte técnico** para enviar una solicitud de soporte técnico. Es posible aumentar el número de trabajos que se pueden ejecutar al mismo tiempo en la cuenta de Data Lake Analytics.

### <a name="how-long-to-keep-job-metadata-and-resources"></a>Periodo de conservación de metadatos de trabajo y recursos 
Cuando los usuarios ejecutan trabajos de U-SQL, el servicio Data Lake Analytics conserva todos los archivos relacionados. Estos archivos incluyen el script de U-SQL, los archivos DLL a los que se hace referencia en el script de U-SQL, los recursos compilados y las estadísticas. Los archivos están en la carpeta /system/ de la cuenta predeterminada de Azure Data Lake Storage. Esta directiva controla durante cuánto tiempo se almacenan estos recursos antes de su eliminación automática (el valor predeterminado es 30 días). Puede usar estos archivos para la depuración y para la optimización de rendimiento de los trabajos que se vayan a volver a ejecutar en el futuro.

Para cambiar el periodo de conservación de metadatos de trabajo y recursos:

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.
2. Haga clic en **Límites y directivas**.
3. En **Días durante los que se conservarán las consultas de trabajo**, mueva el control deslizante para seleccionar un valor o escríbalo en el cuadro de texto.  
4. Haga clic en **Save**(Guardar).

## <a name="job-level-policies"></a>Directivas de nivel de trabajo

Las directivas de nivel de trabajo le permiten controlar las unidades de análisis máximas y la prioridad máxima que los usuarios individuales (o los miembros de grupos de seguridad concretos) pueden establecer en los trabajos que envían. Esta directiva permite controlar los costos en que incurren los usuarios. También permite controlar el efecto que los trabajos programados podrían tener en los trabajos de producción de alta prioridad que se ejecutan en la misma cuenta de Data Lake Analytics.

Data Lake Analytics tiene dos directivas que se pueden establecer en el nivel de trabajo:* **Límite de AU por trabajo**: los usuarios solo pueden enviar trabajos que tengan como máximo este número de unidades de análisis. De forma predeterminada, este límite es el mismo que el límite máximo de unidades de análisis de la cuenta.
* **Prioridad**: los usuarios solo pueden enviar trabajos con una prioridad inferior o igual a este valor. Un número más alto indica una prioridad más baja. De forma predeterminada, este límite se establece en 1, que es la prioridad más alta posible.

Hay una directiva predeterminada establecida en cada cuenta. La directiva predeterminada se aplica a todos los usuarios de la cuenta. Puede crear directivas adicionales para usuarios y grupos concretos. 

> [!NOTE]
> Las directivas de nivel de cuenta y las directivas de nivel de trabajo se aplican de forma simultánea.

### <a name="add-a-policy-for-a-specific-user-or-group"></a>Agregar una directiva para un usuario o grupo concreto

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.
2. Haga clic en **Límites y directivas**.
3. En **Límites del envío de trabajos**, haga clic en el botón **Agregar directiva**. Luego, seleccione o escriba los siguientes valores:
    1. **Compute Policy Name** (Nombre de la directiva de cálculo): escriba un nombre de directiva que le recuerde su propósito.
    2. **Select User or Group** (Seleccionar usuario o grupo): seleccione el usuario o el grupo a los que se aplica esta directiva.
    3. **Set the Job AU Limit** (Establecer el límite de AU del trabajo): establezca el límite de unidades de análisis que se aplica al usuario o al grupo seleccionados.
    4. **Set the Priority Limit** (Establecer el límite de prioridad): establezca el límite de prioridad que se aplica al usuario o al grupo seleccionados.

4. Haga clic en **Aceptar**.

5. La nueva directiva aparece en la tabla de directivas **Predeterminado** de **Límites del envío de trabajos**. ### Eliminación o edición de una directiva existente

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.
2. Haga clic en **Límites y directivas**.
3. En **Límites del envío de trabajos**, busque la directiva que quiere editar.
4.  Para ver las opciones **Eliminar** y **Editar**, en la columna situada más a la derecha de la tabla, haga clic en `...`.## Recursos adicionales para directivas de trabajos
* [Entrada de blog de información general sobre directivas](https://blogs.msdn.microsoft.com/azuredatalake/2017/06/08/managing-your-azure-data-lake-analytics-compute-resources-overview/)
* [Entrada de blog sobre directivas de nivel de cuenta](https://blogs.msdn.microsoft.com/azuredatalake/2017/06/08/managing-your-azure-data-lake-analytics-compute-resources-account-level-policy/)
* [Entrada de blog sobre directivas de nivel de trabajo](https://blogs.msdn.microsoft.com/azuredatalake/2017/06/08/managing-your-azure-data-lake-analytics-compute-resources-job-level-policy/)## Pasos siguientes

* [Información general de Azure Data Lake Analytics](data-lake-analytics-overview.md)
* [Introducción al uso de Azure Portal por parte de Azure Data Lake Analytics](data-lake-analytics-get-started-portal.md)
* [Manage Azure Data Lake Analytics by using Azure PowerShell](data-lake-analytics-manage-use-powershell.md) (Administración de Azure Data Lake Analytics mediante Azure PowerShell)