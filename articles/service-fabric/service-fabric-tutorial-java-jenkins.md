---
title: Configuración de Jenkins para una aplicación Java de Service Fabric en Azure
description: En este tutorial, aprenderá a configurar la integración continua mediante Jenkins para implementar una aplicación de Java en Service Fabric.
author: suhuruli
ms.topic: tutorial
ms.date: 08/27/2018
ms.author: suhuruli
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 42efc13d37ccfdaf60896f338b1a38384ef06568
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320721"
---
# <a name="tutorial-configure-a-jenkins-environment-to-enable-cicd-for-a-java-application-on-service-fabric"></a>Tutorial: Configuración de un entorno Jenkins para habilitar CI/CD para una aplicación Java en Service Fabric

Este tutorial es la parte quinta de una serie. En ella se muestra cómo usar Jenkins para implementar actualizaciones en su aplicación. En este tutorial, el complemento Jenkins de Service Fabric se usa en combinación con el repositorio de GitHub que hospeda la aplicación Voting para implementarla en un clúster.

En la parte número cinco de la serie, aprenderá a:
> [!div class="checklist"]
> * Implementar el contenedor de Jenkins de Service Fabric en su máquina
> * Configurar el entorno de Jenkins para la implementación en Service Fabric
> * Actualizar la aplicación

En esta serie de tutoriales, se aprende a:
> [!div class="checklist"]
> * [Compilar una aplicación Java de Reliable Services en Service Fabric](service-fabric-tutorial-create-java-app.md)
> * [Implementar y depurar la aplicación en un clúster local](service-fabric-tutorial-debug-log-local-cluster.md)
> * [Implementar la aplicación en un clúster de Azure](service-fabric-tutorial-java-deploy-azure.md)
> * [Configurar la supervisión y el diagnóstico para la aplicación](service-fabric-tutorial-java-elk.md)
> * Configuración de CI/CD

## <a name="prerequisites"></a>Requisitos previos

* Instalar Git en el equipo local desde la [página de descargas de Git](https://git-scm.com/downloads). Para más información sobre Git, leer la [documentación correspondiente](https://git-scm.com/docs).
* Tener conocimientos prácticos de [Jenkins](https://jenkins.io/).
* Crear una cuenta de [GitHub](https://github.com/) y saber cómo usar GitHub.
* Instalar [Docker](https://www.docker.com/community-edition) en el equipo.

## <a name="pull-and-deploy-service-fabric-jenkins-container-image"></a>Extracción e implementación de la imagen de contenedor de Jenkins en Service Fabric

Jenkins se puede configurar dentro o fuera de un clúster de Service Fabric. En las siguientes instrucciones se muestra cómo configurarlo fuera de un clúster mediante una imagen de Docker proporcionada. Sin embargo, también se puede usar un entorno de compilación de Jenkins preconfigurado. La siguiente imagen de contenedor viene instalada con el complemento de Service Fabric y está lista para usarla inmediatamente con Service Fabric.

1. Extraiga la imagen del contenedor de Jenkins de Service Fabric: ``docker pull rapatchi/jenkins:v10``. Esta imagen lleva preinstalado el complemento Jenkins de Service Fabric.

1. Ejecute la imagen de contenedor con la ubicación en la que están almacenados los certificados de Azure en el equipo local montado.

    ```bash
    docker run -itd -p 8080:8080 -v /service-fabric-java-quickstart/AzureCluster rapatchi/jenkins:v10
    ```

1. Obtenga el identificador de la instancia de la imagen de contenedor. Puede enumerar todos los contenedores de Docker con el comando ``docker ps –a``

1. Recupere la contraseña de la instancia de Jenkins mediante el comando siguiente:

    ```sh
    docker exec [first-four-digits-of-container-ID] cat /var/jenkins_home/secrets/initialAdminPassword
    ```

    Si el identificador del contenedor es 2d24a73b5964, use 2d24.
    * Esta contraseña es necesaria para iniciar sesión en el panel de Jenkins desde el portal, que es ``http://<HOST-IP>:8080``
    * Después de iniciar sesión por primera vez, puede crear su propia cuenta de usuario o usar la cuenta de administrador.

1. Para configurar GitHub para que funcione con Jenkins, siga los pasos que se indican en [Generating a new SSH key and adding it to the SSH agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) (Generación de una clave SSH nueva y su adición al agente de SSH). Puesto que los comandos se ejecutan desde el contenedor de Docker, siga las instrucciones correspondientes al entorno Linux.
   * Use las instrucciones proporcionadas por GitHub para generar la clave SSH. A continuación, agregue la clave SSH a la cuenta de GitHub que hospeda el repositorio.
   * Ejecute los comandos que se mencionan en el vínculo anterior en el shell de Docker para Jenkins (no en el host).
   * Para iniciar sesión en el shell de Jenkins desde un host, use los siguientes comandos:

     ```sh
     docker exec -t -i [first-four-digits-of-container-ID] /bin/bash
     ```

     Asegúrese de que el clúster o el equipo donde se hospeda la imagen del contenedor de Jenkins o tienen una dirección IP pública. De esta forma, la instancia de Jenkins puede recibir notificaciones de GitHub.

## <a name="create-and-configure-a-jenkins-job"></a>Creación y configuración de trabajos de Jenkins

1. En primer lugar, si no tiene un repositorio que pueda usar para hospedar el proyecto Voting en GitHub, cree uno. El repositorio se denomina **dev_test** en el resto de este tutorial.

1. Cree un **elemento nuevo** en el panel de Jenkins en ``http://<HOST-IP>:8080``.

1. Escriba un nombre para dicho elemento (por ejemplo, **MyJob**). Seleccione **free-style project** (proyecto de estilo libre) y haga clic en **OK** (Aceptar).

1. Vaya a la página del trabajo y haga clic en **Configure** (Configurar).

   a. En la sección general, seleccione la casilla **GitHub project** (Proyecto de GitHub) y especifique la dirección URL del proyecto de GitHub. Dicha URL hospeda la aplicación de Java para Service Fabric que desea integrar con el flujo de integración continua e implementación continua (CI/CD) de Jenkins (por ejemplo, ``https://github.com/testaccount/dev_test``).

   b. En la sección **Source Code Management** (Administración del código fuente), seleccione **Git**. Especifique la dirección URL del repositorio que hospeda la aplicación de Java para Service Fabric que desea integrar con el flujo de CI/CD de Jenkins (por ejemplo, *`https://github.com/testaccount/dev_test.git`* ). También puede especificar la rama que se va a compilar (por ejemplo, **/master**).

1. Configure su instancia de *GitHub* (la que hospeda el repositorio) para que pueda comunicarse con Jenkins. Siga estos pasos:

   a. Vaya a la página del repositorio de GitHub. Vaya a **Settings** >  (Configuración) **Integrations and Services** (Integraciones y servicios).

   b. Seleccione **Add Service** (Agregar servicio), escriba **Jenkins** y seleccione el **complemento Jenkins-GitHub**.

   c. Escriba la dirección URL del webhook de Jenkins (de forma predeterminada, será ``http://<PublicIPorFQDN>:8081/github-webhook/``). Haga clic en **Add/Update Service** (Agregar o actualizar servicio).

   d. Se enviará un evento de prueba a la instancia de Jenkins. Debería ver una marca de verificación verde junto al webhook en GitHub y el proyecto se compila.

   ![Configuración de Jenkins para Service Fabric](./media/service-fabric-tutorial-java-jenkins/jenkinsconfiguration.png)

1. En la sección **Build Triggers** (Compilar desencadenadores), seleccione la opción de compilación que desea. Para este ejemplo, desea desencadenar una compilación cada vez que se produzca alguna inserción en el repositorio. Por tanto, selecciona **GitHub hook trigger for GITScm polling** (Desencadenador de enlace de GitHub para el sondeo de GITScm).

1. En la **sección de compilación**, en la lista desplegable **Add build step** (Agregar paso de compilación), seleccione la opción **Invoke Gradle Script** (Invocar script de Gradle). En el widget que aparece, abra el menú de opciones avanzadas y especifique la ruta de acceso a **Root build script** (Script de compilación raíz) para la aplicación. Este toma el archivo build.gradle de la ruta de acceso especificada y funciona en consecuencia.

    ![Acción de compilación de Jenkins en Service Fabric](./media/service-fabric-tutorial-java-jenkins/jenkinsbuildscreenshot.png)

1. En la lista desplegable **Post-Build Actions** (Acciones posteriores a la compilación), seleccione **Deploy Service Fabric Project** (Implementar proyecto de Service Fabric). Aquí es preciso proporcionar los detalles del clúster donde se implementará la aplicación de Service Fabric compilada por Jenkins. La ruta de acceso al certificado es donde el volumen se ha montado (/ tmp/myCerts).

    También puede proporcionar detalles adicionales usados para implementar la aplicación. Consulte la siguiente captura de pantalla para obtener un ejemplo de los detalles de la aplicación:

    ![Acción de compilación de Jenkins en Service Fabric](./media/service-fabric-tutorial-java-jenkins/sfjenkins.png)

    > [!NOTE]
    > Este clúster puede ser el mismo que el que hospeda la aplicación del contenedor de Jenkins, en caso de que se vaya a usar Service Fabric para implementar la imagen del contenedor de Jenkins.
    >

1. Haga clic en **Save**(Guardar).

## <a name="update-your-existing-application"></a>Actualización de la aplicación existente

1. Actualice el título de HTML en el archivo *VotingApplication/VotingWebPkg/Code/wwwroot/index.html* con **Service Fabric Voting Sample V2**.

    ```html
    <div ng-app="VotingApp" ng-controller="VotingAppController" ng-init="refresh()">
        <div class="container-fluid">
            <div class="row">
                <div class="col-xs-8 col-xs-offset-2 text-center">
                    <h2>Service Fabric Voting Sample V2</h2>
                </div>
            </div>
        </div>
    </div>
    ```

1. Actualice la versión de **ApplicationTypeVersion** y **ServiceManifestVersion** a **2.0.0** en el archivo *Voting/VotingApplication/ApplicationManifest.xml*.

    ```xml
    <?xml version="1.0" encoding="utf-8" standalone="no"?>
    <ApplicationManifest xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingApplicationType" ApplicationTypeVersion="2.0.0">
      <Description>Voting Application</Description>
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="2.0.0"/>
      </ServiceManifestImport>
      <ServiceManifestImport>
            <ServiceManifestRef ServiceManifestName="VotingDataServicePkg" ServiceManifestVersion="1.0.0"/>
        </ServiceManifestImport>
        <DefaultServices>
          <Service Name="VotingWeb">
             <StatelessService InstanceCount="1" ServiceTypeName="VotingWebType">
                <SingletonPartition/>
             </StatelessService>
          </Service>
       <Service Name="VotingDataService">
                <StatefulService MinReplicaSetSize="3" ServiceTypeName="VotingDataServiceType" TargetReplicaSetSize="3">
                    <UniformInt64Partition HighKey="9223372036854775807" LowKey="-9223372036854775808" PartitionCount="1"/>
                </StatefulService>
            </Service>
        </DefaultServices>
    </ApplicationManifest>
    ```

1. Actualice el campo de **versión** de **ServiceManifest** y el campo de **versión** de la etiqueta **CodePackage** en el archivo *Voting/VotingApplication/VotingWebPkg/ServiceManifest.xml* a **2.0.0**.

    ```xml
    <CodePackage Name="Code" Version="2.0.0">
    <EntryPoint>
        <ExeHost>
        <Program>entryPoint.sh</Program>
        </ExeHost>
    </EntryPoint>
    </CodePackage>
    ```

1. Para inicializar un trabajo Jenkins que realice una actualización de la aplicación, inserte los nuevos cambios en el repositorio de GitHub.

1. En Service Fabric Explorer, haga clic en la lista desplegable **Applications** (Aplicaciones). Para ver el estado de la actualización, haga clic en la pestaña **Upgrades in Progress** (Actualizaciones en curso).

    ![Actualización en curso](./media/service-fabric-tutorial-create-java-app/upgradejava.png)

1. Si accede a **http://\<Host-IP>:8080**, verá que la aplicación Voting funciona ahora completamente.

    ![Aplicación Voting local](./media/service-fabric-tutorial-java-jenkins/votingv2.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Implementar el contenedor de Jenkins de Service Fabric en su máquina
> * Configurar el entorno de Jenkins para la implementación en Service Fabric
> * Actualizar la aplicación

* Consulte otros [ejemplos de Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)
