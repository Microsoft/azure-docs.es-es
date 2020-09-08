---
title: 'Tutorial: Integración continua e implementación continua (CI/CD) en máquinas virtuales de Azure mediante Azure Pipelines'
description: En este tutorial, aprenderá a configurar la integración continua y la implementación continua de una aplicación Node.js en máquinas virtuales de Azure mediante la canalización de Azure basada en YAML.
author: ushan
tags: azure-devops-pipelines
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.tgt_pltfrm: azure-pipelines
ms.workload: infrastructure
ms.date: 1/3/2020
ms.author: ushan
ms.custom: devops, devx-track-javascript
ms.openlocfilehash: c83a67f7d524a062485f2c68e0adb7fdd2855a84
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89462180"
---
# <a name="tutorial-deploy-your-app-to-linux-virtual-machines-in-azure-using-azure-devops-services-and-azure-pipelines"></a>Tutorial: Implementación de una aplicación en máquinas virtuales Linux de Azure con Azure DevOps Services y Azure Pipelines

La integración continua y la implementación continua constituyen una canalización mediante la que se puede compilar, publicar e implementar el código tras cada confirmación de este. Este documento contiene los pasos asociados con la configuración de una canalización de CI/CD para realizar implementaciones en varias máquinas mediante Azure Pipelines.

Azure Pipelines ofrece un conjunto completo de herramientas de automatización de CI/CD para implementaciones en máquinas virtuales, tanto en el entorno local como en cualquier nube.

En este tutorial, configurará una canalización de CI/CD basada en YAML para implementar una aplicación en un [entorno](/azure/devops/pipelines/process/environments?view=azure-devops) de Azure Pipelines con máquinas virtuales Linux como recursos, cada una de los cuales actúa como servidor web para ejecutar la aplicación.

Aprenderá a:

> [!div class="checklist"]
> * Obtener una aplicación de ejemplo.
> * Crear una canalización de CI de Azure Pipelines basada en YAML para compilar la aplicación de ejemplo.
> * Crear un entorno de Azure Pipelines para las máquinas virtuales de Azure.
> * Crear una canalización de CD de Azure Pipelines.
> * Ejecutar las implementaciones desencadenadas por CI y de manera manual.

## <a name="before-you-begin"></a>Antes de empezar

* Inicie sesión en su organización de Azure DevOps Services ( **https://dev.azure.com/** ). 
  Puede obtener una [organización de Azure DevOps Services gratis](https://go.microsoft.com/fwlink/?LinkId=307137&clcid=0x409&wt.mc_id=o~msft~vscom~home-vsts-hero~27308&campaign=o~msft~vscom~home-vsts-hero~27308).

  > [!NOTE]
  > Para más información, consulte el artículo de [Conexión a Azure DevOps Services](/azure/devops/organizations/projects/connect-to-projects?view=vsts).

*  Necesita una máquina virtual de Linux para un destino de implementación.  Para obtener más información, consulte [Creación y administración de máquinas virtuales Linux con la CLI de Azure](./tutorial-manage-vm.md).

*  Abra el puerto de entrada 80 para la máquina virtual. Para obtener más información, consulte [Creación de grupos de seguridad de red con Azure Portal](../../virtual-network/tutorial-filter-network-traffic.md).

## <a name="get-your-sample-app-code"></a>Obtención del código de la aplicación de ejemplo

Si ya tiene una aplicación en GitHub que quiere implementar, puede intentar crear una canalización para ese código.

Sin embargo, si es un nuevo usuario, es posible que sea mejor comenzar con nuestro código de ejemplo. En ese caso, bifurque este repositorio en GitHub:

#### <a name="java"></a>[Java](#tab/java)

```
https://github.com/spring-projects/spring-petclinic
```

> [!NOTE]
> Petclinic es una aplicación [Java Spring Boot](https://spring.io/guides/gs/spring-boot) compilada con [Maven](https://spring.io/guides/gs/maven/).

#### <a name="javascript"></a>[JavaScript](#tab/java-script)

```
https://github.com/azure-devops/fabrikam-node
```

> [!NOTE]
> Esta aplicación Node.js se ha compilado con [Yeoman](https://yeoman.io/learning/index.html). Utiliza Express, bower y Grunt. Y tiene algunos paquetes de npm dependientes.
> El ejemplo también contiene un script que configura Nginx e implementa la aplicación. Se ejecuta en las máquinas virtuales. En concreto, el script:
> 1. Instala Node, Nginx y PM2.
> 2. Configura Nginx y PM2.
> 3. Inicia la aplicación Node.

* * * 

## <a name="prerequisites-for-the-linux-vm"></a>Requisitos previos para la máquina virtual Linux

Las aplicaciones de ejemplo mencionadas anteriormente se han probado en Ubuntu 16.04, y se recomienda usar la misma versión de la máquina virtual Linux en este inicio rápido.
Siga los pasos adicionales que se describen a continuación en función de la pila en tiempo de ejecución usada para la aplicación.

#### <a name="java"></a>[Java](#tab/java)

- Para implementar aplicaciones basadas en Java Spring Boot y Spring Cloud, cree una máquina virtual Linux en Azure mediante [esta](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004) plantilla, que proporciona un entorno de ejecución basado en OpenJDK totalmente compatible.
- Para implementar servlets de Java en el servidor de Tomcat, cree una máquina virtual Linux con Java 8 mediante [esta](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004) plantilla de Azure y [configure Tomcat 9.x como servicio](https://tomcat.apache.org/tomcat-9.0-doc/setup.html).
- Para implementar una aplicación basada en Java EE, use una plantilla de Azure para crear una [máquina virtual Linux + Java + WebSphere 9. x](https://azuremarketplace.microsoft.com/marketplace/apps/midvision.websphere-application-server-nde-90), una [máquina virtual Linux + Java + WebLogic 12. x](https://azuremarketplace.microsoft.com/marketplace/apps/oracle.20191009-arm-oraclelinux-wls-admin) o una [máquina virtual Linux + Java](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004) + WildFly/JBoss 14. 


#### <a name="javascript"></a>[JavaScript](#tab/java-script)

Para instalar una aplicación JavaScript o Node.js, necesitará una máquina virtual Linux con el servidor web Nginx para implementar la aplicación.
Si aún no tiene una máquina virtual Linux con Ngnix, cree una ahora en Azure siguiendo los pasos descritos en [este ejemplo](./quick-create-cli.md).

* * * 

## <a name="create-an-azure-pipelines-environment-with-azure-virtual-machines"></a>Creación de un entorno de Azure Pipelines con máquinas virtuales de Azure

Las máquinas virtuales se pueden agregar como recursos en [entornos](/azure/devops/pipelines/process/environments) y se pueden destinar a implementaciones de varias máquinas. Las vistas del historial de implementación dentro del entorno proporcionan rastreabilidad de la máquina virtual a la canalización y, luego, a la confirmación.

Puede crear un entorno en el centro "**Environments**" (Entornos) de la sección "**Pipelines**" (Canalizaciones).
1.  Inicie sesión en su organización de Azure DevOps y vaya a su proyecto.
2.  En el proyecto, vaya a la página **Pipelines** (Canalizaciones). A continuación, elija **Environments** (Entornos) y haga clic en **Create Environment** (Crear entorno). Especifique un valor de **Name** (Nombre) (obligatorio) para el entorno y un valor de **Description** (Descripción).
3.  Elija **Virtual Machines** como **recurso** que se agregará al entorno y haga clic en **Next** (Siguiente).
4.  Elija el sistema operativo (Windows/Linux) y **copie el script de registro de PS**. 
5.  Ahora, desde un símbolo del sistema de administrador de PowerShell, ejecute el script copiado en cada una de las máquinas virtuales de destino que se van a registrar en este entorno.
    > [!NOTE]
    > - Previamente se ha insertado en el script el token de acceso personal del usuario que ha iniciado sesión, que expira el mismo día, lo que hace que el script copiado quede inutilizable a partir de entonces.
    > - Si la máquina virtual ya tiene un agente en ejecución, proporcione un nombre único para "agente" para registrarlo en el entorno.
6.  Una vez registrada la máquina virtual, comenzará a aparecer como un recurso de entorno en la pestaña "recursos" del entorno.

    ![VMcreation](media/tutorial-deploy-vms-azure-pipelines/vm-creation.png)

7.  Para agregar más máquinas virtuales, puede ver y copiar el script de nuevo; para ello, haga clic en "Add resource" (Agregar recurso) y elija "Virtual Machines" como recurso. Este script seguirá siendo el mismo para todas las máquinas virtuales que se vayan a agregar a este entorno. 
8.  Cada máquina interactúa con Azure Pipelines para coordinar la implementación de la aplicación.

    ![VMresource_view](media/tutorial-deploy-vms-azure-pipelines/vm-resourceview.png)

9. Puede agregar etiquetas a la máquina virtual como parte del script de registro de PS interactivo; o, también puede agregarlas o quitarlas de la vista de recursos si hace clic en los tres puntos al final de cada recurso de máquina virtual de la vista de recursos.

   Las etiquetas que asigne permiten limitar la implementación a máquinas virtuales específicas cuando el entorno se usa en un trabajo de implementación. Las etiquetas están limitadas a 256 caracteres, pero no hay límite para el número de etiquetas que se pueden usar.

   ![VMtags](media/tutorial-deploy-vms-azure-pipelines/vm-tags.png)

* * * 

## <a name="define-your-ci-build-pipeline"></a>Definición de la canalización de compilación de CI

Necesitará una canalización de compilación de integración continua (CI) que publique su aplicación web, así como un script de implementación que se puede ejecutar localmente en el servidor Ubuntu. Configure una canalización de compilación de CI basada en el tiempo de ejecución que quiera usar. 

1. Inicie sesión en su organización de Azure DevOps y vaya a su proyecto.

1. En el proyecto, vaya a la página **Pipelines** (Canalizaciones). A continuación, elija la acción para crear una canalización.

1. Siga los pasos del asistente y seleccione primero **GitHub** como ubicación del código fuente.

1. Puede que se le redirija a GitHub para iniciar sesión. Si es así, escriba sus credenciales de GitHub.

1. Cuando aparezca la lista de repositorios, seleccione el repositorio de la aplicación de ejemplo que quiera.

1. Azure Pipelines analizará el repositorio y recomendará una plantilla de canalización adecuada.

#### <a name="java"></a>[Java](#tab/java)

Seleccione la plantilla **starter** y copie el fragmento de código de YAML siguiente que compila el proyecto de Java y ejecuta las pruebas con Apache Maven:

```YAML
- job: Build
  displayName: Build Maven Project
  steps:
  - task: Maven@3
    displayName: 'Maven Package'
    inputs:
      mavenPomFile: 'pom.xml'
  - task: CopyFiles@2
    displayName: 'Copy Files to artifact staging directory'
    inputs:
      SourceFolder: '$(System.DefaultWorkingDirectory)'
      Contents: '**/target/*.?(war|jar)'
      TargetFolder: $(Build.ArtifactStagingDirectory)
  - upload: $(Build.ArtifactStagingDirectory)
    artifact: drop
```

Para más información, siga los pasos mencionados en [Compilación de la aplicación Java con Maven](/azure/devops/pipelines/ecosystems/java).

#### <a name="javascript"></a>[JavaScript](#tab/java-script)

Seleccione la plantilla **starter** y copie el siguiente fragmento de código de YAML que compila un proyecto de Node.js general con npm.

```YAML
- stage: Build
  displayName: Build stage
  jobs:  
  - job: Build
    displayName: Build
    - task: NodeTool@0
      inputs:
        versionSpec: '10.x'
      displayName: 'Install Node.js'
    - script: |
        npm install
        npm run build --if-present
        npm run test --if-present
      displayName: 'npm install, build and test'
    - task: ArchiveFiles@2
      displayName: 'Archive files'
      inputs:
        rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
        includeRootFolder: false
        archiveType: zip
        archiveFile: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
        replaceExistingArchive: true
    - upload: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
      artifact: drop
```

Para más información, siga los pasos descritos en [Compilación de la aplicación Node.js con Gulp](/azure/devops/pipelines/ecosystems/javascript).

- Eche un vistazo a la canalización para ver lo que hace. Asegúrese de que todas las entradas predeterminadas sean adecuadas para el código.

- Seleccione **Save and run** (Guardar y ejecutar) y, luego, seleccione **Commit directly to the master branch** (Confirmar directamente en la rama maestra) y, luego, elija de nuevo **Save and run** (Guardar y ejecutar).

- Se inicia una nueva ejecución. Espere a que finalice.

* * * 

## <a name="define-cd-steps-to-deploy-to-the-linux-vm"></a>Definición de los pasos de CD para realizar la implementación en la máquina virtual Linux

1. Edite la canalización anterior e incluya un [trabajo de implementación](/azure/devops/pipelines/process/deployment-jobs); para ello, haga referencia al entorno y a los recursos de la máquina virtual que ha utilizado anteriormente mediante la sintaxis YAML siguiente:

   ```YAML
   jobs:  
   - deployment: VMDeploy
     displayName: web
     environment:
       name:  <environment name>
       resourceType: VirtualMachine
       tags: web1
     strategy:
   ```
2. Puede seleccionar conjuntos específicos de máquinas virtuales del entorno para que reciban la implementación mediante la especificación de las **etiquetas** que haya definido para cada una.
[Aquí](/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema#deployment-job) está el esquema de YAML completo para el trabajo de implementación.

3. Puede especificar `runOnce` o `rolling` como estrategia de implementación. 

   `runOnce` es la estrategia de implementación más sencilla en la que todos los enlaces del ciclo de vida, es decir, `preDeploy` `deploy`, `routeTraffic` y `postRouteTraffic`, se ejecutan una vez. Luego, se ejecuta `on:` `success` o `on:` `failure`.

   A continuación se muestra el fragmento de código YAML de ejemplo para `runOnce`:
   ```YAML
   jobs:
   - deployment: VMDeploy
     displayName: web
     pool:
       vmImage: 'Ubuntu-16.04'
     environment:
       name: <environment name>
       resourceType: VirtualMachine
     strategy:
         runOnce:
           deploy:
             steps:
             - script: echo my first deployment
   ```

4. A continuación se muestra un fragmento de código YAML de ejemplo que puede usar para definir una estrategia gradual para las actualizaciones de máquinas virtuales de hasta 5 destinos en cada iteración. `maxParallel` determinará el número de destinos que se pueden implementar en paralelo. La selección tiene en cuenta el número absoluto o el porcentaje de destinos que deben permanecer disponibles en cualquier momento excluyendo los destinos en los que se está realizando la implementación. También se usa para determinar las condiciones de acierto y error durante la implementación.

   ```YAML
   jobs: 
   - deployment: VMDeploy
     displayName: web
     environment:
       name: <environment name>
       resourceType: VirtualMachine
     strategy:
         rolling:
           maxParallel: 5  #for percentages, mention as x%
           preDeploy:
             steps:
             - download: current
               artifact: drop
             - script: echo initialize, cleanup, backup, install certs
           deploy:
             steps:
             - task: Bash@3
               inputs:
                 targetType: 'inline'
                 script: |
                   # Modify deployment script based on the app type
                   echo "Starting deployment script run"
                   sudo java -jar '$(Pipeline.Workspace)/drop/**/target/*.jar'
           routeTraffic:
             steps:
             - script: echo routing traffic
           postRouteTraffic:
             steps:
             - script: echo health check post-route traffic
           on:
             failure:
               steps:
               - script: echo Restore from backup! This is on failure
             success:
               steps:
               - script: echo Notify! This is on success
   ```

   Con cada ejecución de este trabajo, el historial de implementación se registra en el entorno `<environment name>` en el que se hayan creado y registrado las máquinas virtuales.

## <a name="run-your-pipeline-and-get-traceability-views-in-environment"></a>Ejecución de la canalización y obtención de vistas de rastreabilidad en el entorno
La vista de implementaciones del entorno proporciona una completa rastreabilidad de las confirmaciones y los elementos de trabajo, y un historial de implementación entre canalizaciones por entorno o recurso.

![VMDeployments_view](media/tutorial-deploy-vms-azure-pipelines/vm-deployments.png)
  
![VMjobs_view](media/tutorial-deploy-vms-azure-pipelines/vm-jobsview.png)

## <a name="next-steps"></a>Pasos siguientes
- Puede continuar con la [personalización de la canalización](/azure/devops/pipelines/customize-pipeline) que acaba de crear.
- Para saber qué más puede hacer en las canalizaciones de YAML, consulte [Referencia de esquema de YAML](/azure/devops/pipelines/yaml-schema).
- Para aprender a implementar una pila LAMP (Linux, Apache, MySQL y PHP), pase al siguiente tutorial.

> [!div class="nextstepaction"]
> [Implementación de una pila de LAMP](tutorial-lamp-stack.md)
