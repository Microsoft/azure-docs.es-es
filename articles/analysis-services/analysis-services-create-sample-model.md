---
title: 'Tutorial: Adición de un modelo de ejemplo; Azure Analysis Services | Microsoft Docs'
description: En este tutorial, aprenderá a agregar un modelo de ejemplo en Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 5b84c2e50ae7f5038cf9cb8a96c0c51d6537c929
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89230913"
---
# <a name="tutorial-add-a-sample-model-from-the-portal"></a>Tutorial: Adición de un modelo de ejemplo desde el portal

En este tutorial, agregará una base de datos modelo tabular de ejemplo, Adventure Works, al servidor. El modelo de ejemplo es una versión completa del modelo de datos de Adventure Works Internet Sales (1200). Un modelo de ejemplo es útil para probar la administración de modelos, la conexión con las herramientas y las aplicaciones cliente, y para consultar datos de modelos. En el tutorial se usa [Azure Portal](https://portal.azure.com) y [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) para: 

> [!div class="checklist"]
> * Agregar un modelo de datos tabulares de ejemplo completo a un servidor 
> * Conectarse al modelo con SSMS

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

- Un servidor de Azure Analysis Services. Para más información, consulte [Creación de un servidor en el portal](analysis-services-create-server.md).
- Permisos de administrador del servidor
- [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)


## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en el [portal](https://portal.azure.com/).

## <a name="add-a-sample-model"></a>Incorporación de un modelo de ejemplo

1. En **Información general** del servidor, haga clic en **Nuevo modelo**.

    ![Creación de un modelo de ejemplo](./media/analysis-services-create-sample-model/aas-create-sample-new-model.png)

2. En **Nuevo modelo** > **Choose a datasource** (Elegir un origen de datos), compruebe que la opción **Datos de ejemplo** está seleccionada y, a continuación, haga clic en **Agregar**.

    ![Selección de un nuevo modelo](./media/analysis-services-create-sample-model/aas-create-sample-data.png)

3. En **Información general**, compruebe que se ha agregado el modelo de ejemplo `adventureworks`.

    ![Seleccionar datos de ejemplo](./media/analysis-services-create-sample-model/aas-create-sample-verify.png)


## <a name="clean-up-resources"></a>Limpieza de recursos

El modelo de ejemplo usa los recursos de memoria caché. Si no va a usar el modelo de ejemplo para realizar pruebas, debe quitarlo del servidor.

Estos pasos describen cómo eliminar un modelo de un servidor mediante el uso de SSMS.

1. En SSMS > **Explorador de objetos**, haga clic en **Conectar** > **Analysis Services**.

2. En **Conectar con el servidor**, pegue el nombre del servidor y, a continuación, en **Autenticación**, elija **Active Directory - Universal compatible con MFA**, escriba el nombre de usuario y, a continuación, haga clic en **Conectar**.

    ![Iniciar sesión](./media/analysis-services-create-sample-model/aas-create-sample-cleanup-signin.png)

3. En **Explorador de objetos**, haga clic con el botón derecho en la base de datos de ejemplo `adventureworks` y, después, haga clic en **Eliminar**.

    ![Eliminar la base de datos de ejemplo](./media/analysis-services-create-sample-model/aas-create-sample-cleanup-delete.png)

## <a name="next-steps"></a>Pasos siguientes 

En este tutorial, aprendió a agregar un modelo de ejemplo básico al servidor. Ahora que tiene una base de datos modelo, puede conectarse a ella desde SQL Server Management Studio y agregar roles de usuario. Para más información, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Tutorial: Configuración de los roles de administrador y usuario del servidor](tutorials/analysis-services-tutorial-roles.md)


