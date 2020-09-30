---
title: Integración del control de código fuente
description: Experiencia de DevOps de bases de datos de clase empresarial para el grupo de SQL con integración nativa de control de código fuente mediante Azure Repos (Git y GitHub).
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql-dw
ms.date: 08/23/2019
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 84abfea39cb7311e7cd60346d936c08c28c334d4
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89441422"
---
# <a name="source-control-integration-for-sql-pool"></a>Integración del control de código fuente para el grupo de SQL

En este tutorial se describe cómo integrar el proyecto de base de datos de SQL Server Data Tools (SSDT) con control de código fuente.  La integración del control de código fuente es el primer paso en la creación de una canalización de integración e implementación continuas con el recurso de grupo de SQL en Azure Synapse Analytics.

## <a name="before-you-begin"></a>Antes de empezar

- Regístrese como [organización de Azure DevOps](https://azure.microsoft.com/services/devops/)
- Realice el tutorial de [creación y conexión](create-data-warehouse-portal.md)
- [Instale Visual Studio 2019](https://visualstudio.microsoft.com/vs/older-downloads/)

## <a name="set-up-and-connect-to-azure-devops"></a>Configuración y conexión con Azure DevOps

1. En su organización de Azure DevOps, cree un proyecto que hospede su proyecto de base de datos de SSDT mediante un repositorio de Azure Repos.

   ![Crear proyecto](./media/sql-data-warehouse-source-control-integration/1-create-project-azure-devops.png "Crear proyecto")

2. Abra Visual Studio y conéctese a su organización y proyecto de Azure DevOps desde el paso un y seleccione **Administrar conexión**.

   ![Administrar conexiones](./media/sql-data-warehouse-source-control-integration/2-manage-connections.png "Administración de conexiones")

3. Conéctese al proyecto seleccionando **Administrar conexiones** y, después, **Conectarse a un proyecto**.
 ![Connect1](./media/sql-data-warehouse-source-control-integration/3-connect-project.png "Conectar")


4. Busque el proyecto que creó en el paso uno y seleccione **Conectar**.
![Connect2](./media/sql-data-warehouse-source-control-integration/3.5-connect.png "Conectar")


3. Clone el repositorio de Azure DevOps del proyecto en la máquina local.

   ![Clonación de repositorio](./media/sql-data-warehouse-source-control-integration/4-clone-repo.png "Clonación de repositorio")

Para más información sobre cómo conectar proyectos mediante Visual Studio, consulte [Conexión a proyectos en Team Explorer](https://docs.microsoft.com/visualstudio/ide/connect-team-project?view=vs-2019). Para obtener una guía para clonar un repositorio mediante Visual Studio, consulte el artículo sobre la [clonación de un repositorio de Git existente](https://docs.microsoft.com/azure/devops/repos/git/clone?view=azure-devops&tabs=visual-studio). 

## <a name="create-and-connect-your-project"></a>Creación y conexión del proyecto

1. En Visual Studio, cree un proyecto de base de datos de SQL Server con un directorio y un repositorio de Git local en el **repositorio local clonado**.

   ![Crear proyecto](./media/sql-data-warehouse-source-control-integration/5-create-new-project.png "Creación de un proyecto")  

2. Haga clic con el botón derecho en el proyecto de SQL vacío e importe el almacenamiento de datos en el proyecto de base de datos.

   ![Importar proyecto](./media/sql-data-warehouse-source-control-integration/6-import-new-project.png "Importar proyecto")  

3. En Team Explorer, en Visual Studio, confirme los cambios en el repositorio local de Git.

   ![Confirmar](./media/sql-data-warehouse-source-control-integration/6.5-commit-push-changes.png "Commit")  

4. Ahora que ha confirmado los cambios localmente en el repositorio clonado, sincronice e inserte los cambios en el repositorio de Azure Repos en el proyecto de Azure DevOps.

   ![Sincronizar e insertar (almacenamiento provisional)](./media/sql-data-warehouse-source-control-integration/7-commit-push-changes.png "Sincronizar e insertar (almacenamiento provisional)")

   ![Sincronizar e insertar](./media/sql-data-warehouse-source-control-integration/7.5-commit-push-changes.png "Sincronizar e insertar")  

## <a name="validation"></a>Validación

1. Compruebe que los cambios se han insertado en Azure Repos mediante la actualización de una columna de la tabla de su proyecto de base de datos desde Visual Studio SQL Server Data Tools (SSDT).

   ![Validar columna de actualización](./media/sql-data-warehouse-source-control-integration/8-validation-update-column.png "Validar columna de actualización")

2. Confirme e inserte el cambio del repositorio local en Azure Repos.

   ![Insertar cambios](./media/sql-data-warehouse-source-control-integration/9-push-column-change.png "Inserción de los cambios")

3. Compruebe que el cambio se ha insertado en el repositorio de Azure Repos.

   ![Verify](./media/sql-data-warehouse-source-control-integration/10-verify-column-change-pushed.png "Comprobar los cambios")

4. (**Opcional**) Use Comparación de esquemas y actualice los cambios en el almacenamiento de datos de destino mediante SSDT para asegurarse de que las definiciones de los objetos del repositorio de Azure Repos y del repositorio local reflejan el almacenamiento de datos.

## <a name="next-steps"></a>Pasos siguientes

- [Desarrollo para el grupo de SQL](sql-data-warehouse-overview-develop.md)
