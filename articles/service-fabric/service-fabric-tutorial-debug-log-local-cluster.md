---
title: Depuración de una aplicación Java en un clúster local de Service Fabric
description: En este tutorial, aprenderá a depurar y obtener registros de una aplicación de Java en Service Fabric que se ejecuta en un clúster local.
author: suhuruli
ms.topic: tutorial
ms.date: 02/26/2018
ms.author: suhuruli
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 040eaeb3ef77dfd68feb2aebac71d945ef669d6d
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372440"
---
# <a name="tutorial-debug-a-java-application-deployed-on-a-local-service-fabric-cluster"></a>Tutorial: Depuración de una aplicación de Java implementada en un clúster local de Service Fabric

Este tutorial es la segunda parte de una serie. Aprenderá a asociar un depurador remoto mediante Eclipse en la aplicación de Service Fabric. También aprenderá a redirigir los registros desde las aplicaciones en ejecución hasta una ubicación práctica para el desarrollador.

En esta serie de tutoriales, se aprende a:
> [!div class="checklist"]
> * [Compilar una aplicación Java de Reliable Services en Service Fabric](service-fabric-tutorial-create-java-app.md)
> * Implementar y depurar la aplicación en un clúster local
> * [Implementar la aplicación en un clúster de Azure](service-fabric-tutorial-java-deploy-azure.md)
> * [Configurar la supervisión y el diagnóstico para la aplicación](service-fabric-tutorial-java-elk.md)
> * [Configure CI/CD](service-fabric-tutorial-java-jenkins.md)


En la segunda parte de la serie, se aprende a:
> [!div class="checklist"]
> * Depurar la aplicación de Java mediante Eclipse
> * Redirigir los registros a una ubicación configurable


## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar este tutorial:

* Configure el entorno de desarrollo para [Mac](service-fabric-get-started-mac.md) o [Linux](service-fabric-get-started-linux.md). Siga las instrucciones para instalar el complemento de Eclipse, Gradle, el SDK de Service Fabric y la CLI de Service Fabric (sfctl).

## <a name="download-the-voting-sample-application"></a>Descarga de la aplicación de ejemplo de votación

Si no compiló la aplicación de ejemplo de votación en la [primera parte de esta serie de tutoriales](service-fabric-tutorial-create-java-app.md), puede descargarla. En una ventana Comandos, ejecute el comando siguiente para clonar el repositorio de la aplicación de ejemplo en la máquina local.

```bash
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart
```

[Compile e implemente](service-fabric-tutorial-create-java-app.md#deploy-application-to-local-cluster) la aplicación en el clúster de desarrollo local.

## <a name="debug-java-application-using-eclipse"></a>Depurar la aplicación de Java mediante Eclipse

1. Abra el entorno de desarrollo integrado (IDE) de Eclipse en su máquina y haga clic en **File (Archivo) -> Import (Importar)....**

2. En la ventana emergente, seleccione la opción **General -> Existing Projects into Workspace (Proyectos existentes en área de trabajo)** y presione Next (Siguiente).

3. En la ventana Import Projects (Importar proyectos), elija la opción **Select root directory** (Seleccionar directorio raíz) y seleccione el directorio **Voting**. Si ha seguido la parte uno de la serie de tutoriales, el directorio **Voting** se encuentra en el directorio **Eclipse-workspace**.

4. Actualice el archivo entryPoint.sh del servicio que desea depurar para que comience el proceso de Java con parámetros de depuración remota. En este tutorial, se usa el front-end sin estado: *Voting/VotingApplication/VotingWebPkg/Code/entryPoint.sh*. En este ejemplo, el puerto 8001 está establecido para depuración.

    ```bash
    java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -jar VotingWeb.jar
    ```

5. Actualice el manifiesto de aplicación y establezca en 1 el recuento de instancias o el recuento de réplicas del servicio que se está depurando. Esta configuración evita conflictos en el puerto que se usa para depuración. Por ejemplo, para servicios sin estado, establezca ``InstanceCount="1"`` y, para los servicios con estado, establezca los tamaños del conjunto de réplicas mínimo y de destino en 1 de la manera siguiente: ``TargetReplicaSetSize="1" MinReplicaSetSize="1"``.

6. En el IDE de Eclipse, seleccione **Run (Ejecutar) -> Debug Configurations (Configuraciones de depuración) -> Remote Java Application (Aplicación de Java remota)** , presione el botón **New** (Nuevo), establezca las propiedades de la manera siguiente y haga clic en **Apply** (Aplicar).

    ```
    Name: Voting
    Project: Voting
    Connection Type: Standard
    Host: localhost
    Port: 8001
    ```

7. Coloque un punto de interrupción en la línea 109 del archivo *Voting/VotingWeb/src/statelessservice/HttpCommunicationListener.java*.

8. En el explorador de paquetes, haga clic con el botón derecho en el proyecto **Voting** y haga clic en **Service Fabric -> Publish Application (Publicar aplicación)...**

9. En la ventana **Publish Application** (Publicar aplicación), seleccione **Local.json** en la lista desplegable y haga clic en **Publish** (Publicar).

10. En el IDE de Eclipse IDE, seleccione **Run (Ejecutar) -> Debug Configurations (Configuraciones de depuración) -> Remote Java Application (Aplicación de Java remota)** , haga clic en la configuración de **Voting** que ha creado y haga clic en **Debug** (Depurar).

11. Vaya a su explorador web y acceda a **localhost: 8080**. Alcanzará automáticamente el punto de interrupción y Eclipse escribirá la **perspectiva de depuración**.

Ahora puede aplicar estos mismos pasos para depurar cualquier aplicación de Service Fabric en Eclipse.

## <a name="redirect-application-logs-to-custom-location"></a>Redirección de los registros de aplicaciones a la ubicación personalizada

En los pasos siguientes se describe cómo redirigir los registros de aplicaciones desde la ubicación predeterminada */var/log/syslog* hasta una ubicación personalizada.

1. Actualmente, las aplicaciones que se ejecutan en clústeres de Linux de Service Fabric solo admiten la selección de un único archivo de registro. Para configurar una aplicación de tal modo que los registros siempre se dirijan a */tmp/mysfapp0.0.log*, cree un archivo llamado logging.properties en la siguiente ubicación *Voting/VotingApplication/VotingWebPkg/Code/logging.properties* y agregue el siguiente contenido.

    ```
    handlers = java.util.logging.FileHandler

    java.util.logging.FileHandler.level = ALL
    java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter

    # This value specifies your custom location.
    # You will have to ensure this path has read and write access by the process running the SF Application
    java.util.logging.FileHandler.pattern = /tmp/mysfapp0.0.log
    ```

2. Agregue el siguiente parámetro de *Voting/VotingApplication/VotingWebPkg/Code/entryPoint.sh* al comando de ejecución de Java:

    ```bash
    -Djava.util.logging.config.file=logging.properties
    ```

    En el ejemplo siguiente, se muestra una ejecución de ejemplo con el depurador adjunto, similar a la ejecución de la sección anterior.

    ```bash
    java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -Djava.util.logging.config.file=logging.properties -jar VotingWeb.jar
    ```

En esta fase, ha aprendido a depurar los registros de aplicaciones, y a acceder a ellos, durante el desarrollo de aplicaciones de Java en Service Fabric.

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha aprendido a:

> [!div class="checklist"]
> * Depurar la aplicación de Java mediante Eclipse
> * Redirigir los registros a una ubicación configurable

Avance hasta el siguiente tutorial:
> [!div class="nextstepaction"]
> [Implementación de aplicaciones en Azure](service-fabric-tutorial-java-deploy-azure.md)