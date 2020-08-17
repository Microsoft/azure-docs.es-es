---
title: 'Tutorial: Implementación de aplicaciones de Node.js con tecnología de Azure Cosmos DB con Azure DevOps Starter'
description: Azure DevOps Starter facilita la introducción a Azure. Con DevOps Starter, puede implementar la aplicación de Node.js con tecnología de Azure Cosmos DB en Windows Web App en tan solo unos pocos pasos rápidos.
ms.author: mlearned
ms.manager: gwallace
ms.prod: devops
ms.technology: devops-cicd
ms.topic: tutorial
ms.date: 03/24/2020
author: mlearned
ms.custom: devx-track-javascript
ms.openlocfilehash: 47e175a47d3548962f63e283c80abe8333e9ecc4
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421627"
---
# <a name="deploy-nodejs-apps-powered-by-azure-cosmos-db-with-devops-starter"></a>Implementación de aplicaciones de Node.js con tecnología de Azure Cosmos DB con DevOps Starter

Azure DevOps Starter ofrece una experiencia simplificada, con la que puede crear una canalización de integración continua (CI) y de implementación continua (CD) en Azure. Para hacerlo, use el código y el repositorio Git existentes, o bien seleccione una aplicación de ejemplo.

DevOps Starter también:

* Crea automáticamente recursos de Azure, como Azure Cosmos DB, Azure Application Insights, Azure App Service y planes de App Service.

* Crea y configura una canalización de versión de CI/CD en Azure DevOps.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Usar DevOps Starter para la implementación de aplicaciones de Node.js con tecnología de Azure Cosmos DB
> * Configuración de Azure DevOps y una suscripción de Azure
> * Examen de Azure Cosmos DB
> * Examen de la canalización de CI
> * Examen de la canalización de CD
> * Confirmar los cambios en Git e implementarlos automáticamente en Azure
> * Limpiar los recursos

## <a name="prerequisites"></a>Prerrequisitos

Necesita una suscripción de Azure, que puede obtener de forma gratuita mediante [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).

## <a name="use-devops-starter-to-deploy-nodejs-app"></a>Uso de DevOps Starter para implementar la aplicación Node.js

DevOps Starter crea una canalización de CI/CD en Azure Pipelines. Puede crear una organización de Azure DevOps nueva o usar una existente. DevOps Starter también crea recursos de Azure (como Azure Cosmos DB, Application Insights, App Service y planes de App Service) en la suscripción de Azure que prefiera.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En el cuadro de búsqueda, escriba **DevOps Starter** y, a continuación, seleccione. Haga clic en **Agregar** para crear un recurso.

    ![Panel de DevOps Starter](_img/azure-devops-starter-aks/search-devops-starter.png)

1. Seleccione **Node. js** como entorno de ejecución y, después, seleccione **Siguiente**. En **Elegir un marco de trabajo de la aplicación**, seleccione **Express.js**.

1. Habilite la sección **Agregar una base de datos** para **Cosmos DB** y, después, seleccione **Siguiente**.

    ![Adición de una base de datos](_img/azure-devops-project-cosmos-db/add-database.png)

    Azure DevOps Starter admite varios marcos de trabajo de las aplicaciones, como **Express.js**, **Aplicación de ejemplo Node.js** y **Sail.js**. En este tutorial, usaremos **Express.js**.

1. Seleccione un servicio de Azure donde quiera implementar la aplicación y, después, haga clic en **Siguiente**. Entre sus opciones, se incluyen App Service en Windows, Azure Kubernetes Service y Azure Web App for Containers. En este tutorial, usaremos **App Service en Windows**.

## <a name="configure-azure-devops-and-azure-subscription"></a>Configuración de Azure DevOps y una suscripción de Azure

1. Escriba el nombre del proyecto de Azure DevOps.

1. Cree una organización de Azure DevOps nueva o seleccione una existente.

1. Seleccione su suscripción a Azure.

1. Para ver otras opciones de configuración de Azure o identificar la ubicación y el plan de tarifa, seleccione **Configuración adicional**. En este panel, se muestran varias opciones para configurar el plan de tarifa y la ubicación de los servicios de Azure.

1. Salga del área de configuración de Azure y seleccione **Listo**.

1. El proceso se completará en unos minutos. Se configurará una aplicación de ejemplo de Node.js en un repositorio Git de la organización de Azure DevOps. Después, se crearán recursos de Azure Cosmos DB, App Service, plan de App Service y Application Insights, así como la canalización de CI/CD. En el paso siguiente, la aplicación se implementará en Azure.

   Cuando se completen todos estos procesos, se mostrará el panel de Azure DevOps Starter en Azure Portal. También puede ir al panel de DevOps Starter directamente desde la opción **Todos los recursos** de Azure Portal.

   Este panel permite obtener información sobre el repositorio de código de Azure DevOps, la canalización de CI/CD y la base de datos de Azure Cosmos DB. En la canalización de Azure DevOps se pueden configurar otras opciones de CI/CD. En la parte derecha del panel, seleccione **Azure Cosmos DB** para ver estas opciones.

## <a name="examine-azure-cosmos-db"></a>Examen de Azure Cosmos DB

DevOps Starter configura automáticamente una instancia de Azure Cosmos DB que puede explorar y personalizar. Para familiarizarse con Azure Cosmos DB, siga este procedimiento:

1. Vaya al panel de inicio de DevOps Starter.

    ![Panel de DevOps Projects](_img/azure-devops-project-cosmos-db/devops-starter-dashboard.png)

1. En la parte derecha, seleccione Azure Cosmos DB. Se abrirá un panel de Azure Cosmos DB. En esta vista, puede realizar varias acciones, como supervisar operaciones y buscar registros.

    ![Panel de Azure Cosmos DB](_img/azure-devops-project-cosmos-db/cosmos-db.png)

## <a name="examine-the-ci-pipeline"></a>Examen de la canalización de CI

DevOps Starter configura automáticamente una canalización de CI/CD de Azure completa en su organización de Azure DevOps. Puede explorar y personalizar la canalización. Para familiarizarse con ella, siga estos pasos:

1. Vaya al panel de inicio de DevOps Starter.

1. Seleccione el hipervínculo debajo de **Compilar**. Una pestaña del explorador muestra la canalización de compilación del nuevo proyecto.

    ![Panel Compilar](_img/azure-devops-project-cosmos-db/build.png)

1. Seleccione **Editar**. En este panel puede examinar las distintas tareas de la canalización de compilación. La compilación ejecuta varias tareas, como capturar códigos fuente del repositorio de Git, compilar la aplicación, ejecutar pruebas unitarias y publicar salidas que se usan para implementaciones.

1. Seleccione **Desencadenadores**. DevOps Starter crea automáticamente un desencadenador de integración continua y cada confirmación al repositorio inicia una compilación. De manera opcional, también puede incluir o excluir ramas del proceso de CI.

1. Seleccione **Retención**. En función del escenario, puede especificar directivas para conservar o quitar un determinado número de compilaciones.

1. En la parte superior de la canalización de compilación, seleccione el nombre de esta.

1. Cambie el nombre de la canalización de compilación por otro más descriptivo y seleccione **Guardar** en el menú desplegable **Guardar y poner en cola**.

1. En el nombre de la canalización de compilación, seleccione **Historial**. Este panel muestra un registro de auditoría de los cambios recientes de la compilación. Azure DevOps realiza un seguimiento de los cambios realizados en la canalización de compilación y permite comparar las versiones.

## <a name="examine-the-cd-release-pipeline"></a>Examen de la canalización de versión de la implementación continua

DevOps Starter crea y configura automáticamente los pasos necesarios para implementar desde la organización de Azure DevOps en la suscripción de Azure. Dichos pasos incluyen la configuración de una conexión de servicio de Azure para realizar la autenticación de Azure DevOps en la suscripción de Azure. La automatización también crea una canalización de versión, lo que proporciona la implementación continua en Azure. Para más información acerca de la canalización de versión, siga estos pasos:

1. Vaya a **Canalizaciones** y seleccione **Versiones**.

1. Seleccione **Editar**.

1. En **Artefactos**, seleccione **Colocar**. La canalización de compilación que ha examinado en los pasos anteriores genera la salida que se usa para el artefacto.

1. A la derecha del icono **Colocar**, seleccione **Desencadenador de implementación continua**. Esta canalización de versión ha habilitado un desencadenador de implementación continua, que ejecuta una implementación cada vez que hay disponible un nuevo artefacto de compilación. Puede deshabilitar el desencadenador para que las implementaciones se ejecuten de forma manual.

1. A la derecha, seleccione la sección **Ver versiones** para mostrar un historial de las versiones.

1. Al seleccionar la versión, se mostrará la canalización. Seleccione un entorno para comprobar el resumen de la publicación, las confirmaciones o los elementos de trabajo asociados.

1. Seleccione **Confirmaciones**. Esta vista muestra las confirmaciones de código que están asociadas a esta implementación. Compare las versiones para ver las diferencias de confirmación entre las implementaciones.

1. Seleccione **Ver registros**. Los registros contienen información útil sobre el proceso de implementación. Se pueden ver tanto durante las implementaciones como después de ellas.

## <a name="commit-code-changes-and-execute-the-cicd-pipeline"></a>Confirmación de cambios de código y ejecución de la canalización de CI/CD

> [!NOTE]
> El siguiente procedimiento prueba la canalización de CI/CD. Para ello, realiza un cambio de texto simple.

Ya está preparado para colaborar con un equipo en su aplicación mediante un proceso de CI/CD que implemente el trabajo más reciente en su instancia de App Service. Cada cambio que se realiza en el repositorio de Git inicia una compilación en Azure DevOps y una canalización de CD ejecuta una implementación en Azure. Siga el procedimiento descrito en esta sección o utilice otra técnica para confirmar los cambios en el repositorio. Por ejemplo, puede clonar el repositorio de Git en su herramienta favorita o IDE, y luego insertar los cambios en este repositorio.

1. En el menú de Azure DevOps, seleccione **Repositorios** y, después, **Archivos**. Después, vaya al repositorio.

1. En el repositorio, ya se incluye código basado en el lenguaje de la aplicación que haya seleccionado en el proceso de creación. Abra el archivo **Application/views/index.pug**.

1. Seleccione **Editar** y, después, realice un cambio en la **línea número 15**. Por ejemplo, puede cambiarla a “Mi primera implementación en Azure App Service con tecnología de Azure Cosmos DB”.

1. En la esquina superior derecha, seleccione **Confirmar** y, después, vuelva a seleccionar **Confirmar** para insertar los cambios.

     Después de unos instantes, se iniciará una compilación en Azure DevOps y se ejecutará una versión para implementar los cambios. Supervise el estado de la compilación en el panel de DevOps Starter o en el explorador con la organización de Azure DevOps.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos relacionados que haya creado, elimínelos. Use la funcionalidad **Eliminación** del panel de DevOps Starter.

## <a name="next-steps"></a>Pasos siguientes

Puede modificar estas canalizaciones de compilación y de versión para satisfacer las necesidades de su equipo. También puede usar este patrón de CI/CD como plantilla para las demás canalizaciones. En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Usar DevOps Starter para la implementación de aplicaciones de Node.js con tecnología de Azure Cosmos DB
> * Configuración de Azure DevOps y una suscripción de Azure 
> * Examen de Azure Cosmos DB
> * Examen de la canalización de CI
> * Examen de la canalización de CD
> * Confirmación de los cambios en Git y su implementación automática en Azure
> * Limpieza de recursos

Para obtener más información y conocer los pasos siguientes, vea [Definición de la canalización de implementación continua (CD) de varias fases](https://docs.microsoft.com/azure/devops/pipelines/release/define-multistage-release-process?view=azure-devops&viewFallbackFrom=vsts).


