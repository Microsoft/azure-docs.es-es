---
title: Conexión a Azure Analysis Services con Power BI | Microsoft Docs
description: Aprenda a conectarse a un servidor de Azure Analysis Services mediante Power BI. Una vez conectados, los usuarios pueden explorar los datos del modelo.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 03/30/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: f79ff9af1272ca908c7a5ba35fbaa1e8bb87fc50
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86506719"
---
# <a name="connect-with-power-bi"></a>Conexión con Power BI

Una vez que se ha creado un servidor en Azure y se ha implementado un modelo tabular, los usuarios de su organización pueden conectarse y comenzar a explorar los datos. 

> [!TIP]
> Asegúrese de usar la versión más reciente de [Power BI Desktop](https://powerbi.microsoft.com/desktop/).
> 
> 
  
## <a name="connect-in-power-bi-desktop"></a>Conexión en Power BI Desktop

1. En Power BI Desktop, haga clic en **Obtener datos** > **Azure** > **Base de datos de Azure Analysis Services**.

2. En **Servidor**, escriba el nombre del servidor. Asegúrese de incluir la dirección URL completa; por ejemplo, asazure://westcentralus.asazure.windows.net/advworks.

3. En **Base de datos**, si conoce el nombre de la base de datos de modelo tabular o la perspectiva a la que desea conectarse, péguelo aquí. En caso contrario, puede dejar este campo en blanco y seleccionar una base de datos o una perspectiva más adelante.

4. Seleccione una opción de conexión y presione **Conectar**. 

    Se admiten las opciones **Conectar en directo** e **Importar**. Sin embargo, se recomienda usar conexiones en directo porque el modo de importación tiene algunas limitaciones. En particular, durante la importación se puede afectar el rendimiento del servidor. Además, si el modelo se actualizará en el servicio Power BI, la configuración **Allow access from Power BI** (Permitir el acceso desde Power BI) solo se aplica cuando se elige **Conectar en directo**.

5. Escriba sus credenciales de inicio de sesión, si se le solicitan. 

   > [!NOTE]
   > No se admiten las cuentas de código de acceso de un solo uso (OTP). 

6. En **Navegador**, expanda el servidor, seleccione el modelo o la perspectiva a la que desea conectarse y haga clic en **Conectar**. Haga clic en un modelo o una perspectiva para mostrar todos los objetos de esa vista.

    El modelo se abre en Power BI Desktop con un informe en blanco en la vista de informe. La lista de campos muestra todos los objetos del modelo no ocultos. El estado de la conexión se muestra en la esquina inferior derecha.

## <a name="connect-in-power-bi-service"></a>Conexión en Power BI (servicio)

1. Cree un archivo de Power BI Desktop que tenga una conexión en directo con el modelo del servidor.
2. En [Power BI](https://powerbi.microsoft.com), haga clic en **Obtener datos** > **Archivos**, busque el archivo .pbix y selecciónelo.

## <a name="see-also"></a>Consulte también
[Conexión a Azure Analysis Services](analysis-services-connect.md)   
[Bibliotecas de cliente](https://docs.microsoft.com/analysis-services/client-libraries?view=azure-analysis-services-current)

