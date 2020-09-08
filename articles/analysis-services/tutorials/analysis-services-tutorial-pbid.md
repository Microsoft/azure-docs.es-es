---
title: 'Tutorial: Conexión de Azure Analysis Services con Power BI Desktop | Microsoft Docs'
author: minewiskan
description: En este tutorial, aprenderá a obtener un nombre de servidor de Analysis Services desde Azure Portal y luego conectarse al servidor mediante Power BI Desktop.
ms.service: azure-analysis-services
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: owend
ms.reviewer: owend
ms.openlocfilehash: 950447aa146f05714b6ec5ea3d6e47c368a2140e
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89229317"
---
# <a name="tutorial-connect-with-power-bi-desktop"></a>Tutorial: conexión con Power BI Desktop

En este tutorial, va a usar Power BI Desktop para conectarse a la base de datos del modelo de ejemplo de adventureworks en el servidor. Las tareas que va a llevar a cabo simulan una conexión de usuario típico al modelo y la creación de un informe básico a partir de los datos del modelo.

> [!div class="checklist"]
> * Obtención del nombre del servidor desde el portal
> * Conexión mediante Power BI Desktop
> * Creación de un informe básico

## <a name="prerequisites"></a>Prerequisites

- [Agregar la base de datos del modelo de ejemplo de adventureworks](../analysis-services-create-sample-model.md) al servidor.
- Tener permisos de [*lectura*](../analysis-services-server-admins.md) para la base de datos del modelo de ejemplo de adventureworks.
- [Instalar la versión más reciente de Power BI Desktop](https://powerbi.microsoft.com/desktop).

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal
En este tutorial, iniciará sesión en el portal solo para obtener el nombre del servidor. Normalmente, los usuarios obtienen el nombre del servidor del administrador del servidor.

Inicie sesión en el [portal](https://portal.azure.com/).

## <a name="get-server-name"></a>Obtención del nombre del servidor
Para poder conectarse al servidor desde Power BI Desktop, primero necesita el nombre de este. Puede obtener el nombre del servidor desde el portal.

En **Azure Portal** > servidor > **Información general** > **Nombre de servidor**, copie el nombre del servidor.
   
   ![Obtención del nombre del servidor en Azure](./media/analysis-services-tutorial-pbid/aas-copy-server-name.png)

## <a name="connect-in-power-bi-desktop"></a>Conexión en Power BI Desktop

1. En Power BI Desktop, haga clic en **Obtener datos** > **Azure** > **Base de datos de Azure Analysis Services**.

   ![Conexión en Obtener datos](./media/analysis-services-tutorial-pbid/aas-pbid-connect-aasserver.png)

2. En **Servidor**, pegue el nombre del servidor y en **Base de datos**, escriba **adventureworks**, después haga clic en **Aceptar**.

   ![Especificación del nombre del servidor y el modelo de base de datos](./media/analysis-services-tutorial-pbid/aas-pbid-connect-aas-servername.png)

3. Cuando se le solicite, escriba las credenciales. La cuenta que especifique tiene que tener al menos permisos de lectura para la base de datos del modelo de ejemplo de adventureworks.

    El modelo adventureworks se abre en Power BI Desktop con un informe en blanco en la vista de informe. La lista **Campos** muestra todos los objetos del modelo no ocultos. El estado de la conexión se muestra en la esquina inferior derecha.

4. En **VISUALIZACIONES**, seleccione **Gráfico de barras agrupadas** y haga clic en **Formato** (icono de rodillo de pintar) y después active **Etiquetas de datos**. 

   ![Visualizaciones](./media/analysis-services-tutorial-pbid/aas-pbid-visualizations-report.png)

5. En la tabla **CAMPOS** > **Venta por Internet**, seleccione las medidas **Total de ventas por Internet** y **Margen**. En la tabla **Categoría de producto**, seleccione **Nombre de categoría de producto**.

   ![Completar el informe](./media/analysis-services-tutorial-pbid/aas-pbid-complete-report.png)

    Tómese unos minutos para explorar el modelo de ejemplo de adventureworks creando diferentes visualizaciones y segmentando datos y métricas. Cuando esté satisfecho con el informe, asegúrese de guardarlo.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no lo necesita, no guarde el informe o elimine el archivo si lo ha guardado.

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha aprendido a utilizar Power BI Desktop para conectarse a un modelo de datos en un servidor y crear un informe básico. Si no está familiarizado con cómo crear un modelo de datos, consulte el [tutorial de modelado de datos tabulares de ventas por Internet de Adventure Works](https://docs.microsoft.com/analysis-services/tutorial-tabular-1400/as-adventure-works-tutorial) en los documentos de SQL Server Analysis Services.