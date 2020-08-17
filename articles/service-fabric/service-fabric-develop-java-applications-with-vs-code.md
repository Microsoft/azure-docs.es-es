---
title: Desarrollo de aplicaciones de Java con Visual Studio Code
description: En este artículo se muestra cómo compilar, implementar y depurar aplicaciones de Java para Service Fabric con Visual Studio Code.
author: peterpogorski
ms.topic: article
ms.date: 06/29/2018
ms.custom: devx-track-java
ms.author: pepogors
ms.openlocfilehash: cc65deb924a9f3367c2ea1d7c71544743ccf2697
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327368"
---
# <a name="develop-java-service-fabric-applications-with-visual-studio-code"></a>Desarrollo de aplicaciones de C# para Service Fabric con Visual Studio Code

La [extensión Service Fabric Reliable Services para VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-service-fabric-reliable-services) facilita la tarea de compilar aplicaciones de Java para Service Fabric en sistemas operativos Windows, Linux y macOS.

En este artículo se muestra cómo compilar, implementar y depurar una aplicación de Java para Service Fabric con Visual Studio Code.

> [!IMPORTANT]
> Las aplicaciones de Java para Service Fabric se pueden desarrollar en máquinas Windows, pero solo se pueden implementar en clústeres Linux de Azure. En Windows no se admite la depuración de aplicaciones de Java.

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se da por supuesto que ya ha instalado VS Code, la extensión Service Fabric Reliable Services para VS Code y todas las dependencias necesarias para el entorno de desarrollo. Para obtener más información, consulte la [introducción](./service-fabric-get-started-vs-code.md#prerequisites).

## <a name="download-the-sample"></a>Descarga del ejemplo
En este artículo se usa la aplicación Voting del [repositorio de GitHub de guías de inicio rápido para crear aplicaciones de Java para Service Fabric](https://github.com/Azure-Samples/service-fabric-java-quickstart). 

Para clonar el repositorio en la máquina de desarrollo, ejecute el siguiente comando desde una ventana de terminal (ventana de comandos de Windows):

```sh
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart.git
```

## <a name="open-the-application-in-vs-code"></a>Abrir la aplicación en VS Code

Abra VS Code.  Haga clic en el icono Explorador de la **Barra de actividades** y haga clic en **Abrir carpeta**, o haga clic en **Archivo -> Abrir carpeta**. Vaya al directorio *./service-fabric-java-quickstart/Voting* en la carpeta donde ha clonado el repositorio y haga clic en **Aceptar**. El área de trabajo debe contener los mismos archivos que se muestran en la captura de pantalla siguiente.

![Aplicación Voting de Java en el área de trabajo](./media/service-fabric-develop-java-applications-with-vs-code/java-voting-application.png)

## <a name="build-the-application"></a>Compilar la aplicación

1. Presione Ctrl+Mayús+P para abrir la **paleta de comandos** en VS Code.
2. Busque y seleccione el comando **Service Fabric: Build Application**. La salida de compilación se envía al terminal integrado.

   ![Comando de compilación de la aplicación en VS Code](./media/service-fabric-develop-java-applications-with-vs-code/sf-build-application.png)

## <a name="deploy-the-application-to-the-local-cluster"></a>Implementación de la aplicación en el clúster local
Después de haber compilado la aplicación, se puede implementar en el clúster local. 

> [!IMPORTANT]
> No se permite implementar aplicaciones de Java en el clúster local en equipos Windows.

1. En la **paleta de comandos**, seleccione el comando **Service Fabric: Deploy Application (Localhost)** . La salida del proceso de instalación se envía al terminal integrado.

   ![Comando de implementación de la aplicación en VS Code](./media/service-fabric-develop-java-applications-with-vs-code/sf-deploy-application.png)

4. Una vez completada la implementación, inicie un explorador y abra Service Fabric Explorer: `http://localhost:19080/Explorer`. Debería ver que la aplicación se está ejecutando. Esto puede tardar algún tiempo, tenga paciencia. 

   ![Aplicación Voting en Service Fabric Explorer](./media/service-fabric-develop-java-applications-with-vs-code/sfx-localhost-java.png)

4. Después de haber comprobado que la aplicación se está ejecutando, inicie un explorador y abra esta página: `http://localhost:8080`. Se trata del front-end web de la aplicación. Puede agregar elementos y hacer clic en ellos para votar.

   ![Aplicación Voting en el explorador](./media/service-fabric-develop-java-applications-with-vs-code/voting-sample-in-browser.png)

5. Para quitar la aplicación del clúster, seleccione el comando **Service Fabric: Remove Application**  (Service Fabric: quitar aplicación) de la **paleta de comandos**. La salida del proceso de desinstalación se envía al terminal integrado. Puede usar Service Fabric Explorer para comprobar que la aplicación se ha quitado del clúster local.

## <a name="debug-the-application"></a>Depuración de la aplicación
Al depurar aplicaciones en VS Code, debe ejecutar la aplicación en un clúster local. A continuación, se pueden agregar puntos de interrupción en el código.

> [!IMPORTANT]
> En equipos Windows no se admite la depuración de aplicaciones de Java.

Para preparar VotingDataService y la aplicación Voting para su depuración, complete los pasos siguientes:

1. Actualice el archivo *Voting/VotingApplication/VotingDataServicePkg/Code/entryPoint.sh*.
Marque como comentario la línea 6 (use '#') y agregue el siguiente comando en la parte inferior del archivo:

   ```
   java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -jar VotingDataService.jar
   ```

2. Actualice el archivo *Voting/VotingApplication/ApplicationManifest.xml*. Establezca los atributos **MinReplicaSetSize** y **TargetReplicaSetSize** en "1" en el elemento **StatefulService**:
   
   ```xml
         <StatefulService MinReplicaSetSize="1" ServiceTypeName="VotingDataServiceType" TargetReplicaSetSize="1">
   ```

3. Haga clic en el icono Depurar en la **barra de actividades** para abrir la vista depurador de VS Code. Haga clic en el icono de engranaje en la parte superior de la vista de depurador y seleccione **Java** en el menú desplegable del entorno. Se abre el archivo launch.json. 

   ![Icono Depurar en el área de trabajo de VS Code](./media/service-fabric-develop-java-applications-with-vs-code/debug-icon-workspace.png)

3. En el archivo launch.json, en la opción de configuración llamada **Debug (Attach)** , establezca el valor del puerto en **8001**. Guarde el archivo.

   ![Configuración de depuración para launch.json](./media/service-fabric-develop-java-applications-with-vs-code/launch-json-java.png)

4. Implemente la aplicación en el clúster local mediante el comando **Service Fabric: Deploy Application (Localhost)** (Service Fabric: implementar aplicación [Localhost]). Compruebe que la aplicación se está ejecutando en Service Fabric Explorer. La aplicación ya está lista para depurar.

Para establecer un punto de interrupción, complete los pasos siguientes:

1. En el explorador, abra el archivo */Voting/VotingDataService/src/statefulservice/VotingDataService.java*. Establezca un punto de interrupción en la primera línea de código del bloque `try`, en el método `addItem` (línea 80).
   
   ![Establezca el punto de interrupción en el servicio de datos de Voting](./media/service-fabric-develop-java-applications-with-vs-code/breakpoint-set.png)

   > [!IMPORTANT]
   > Asegúrese de establecer los puntos de interrupción en líneas de código ejecutables. Por ejemplo, el depurador no detectará los puntos de interrupción establecidos en declaraciones de método, instrucciones `try` o instrucciones `catch`.
2. Para empezar a depurar, haga clic en el icono Depurar en la **barra de actividad**, seleccione la opción **Depurar (adjuntar)** en el menú Depurar y haga clic en el botón de ejecución (flecha verde).

   ![Configuración Depurar (adjuntar)](./media/service-fabric-develop-java-applications-with-vs-code/debug-attach-java.png)

3. Vaya a `http://localhost:8080` en un explorador web. Escriba un nuevo elemento en el cuadro de texto y haga clic en **+ Agregar**. Se debe alcanzar el punto de interrupción. Puede usar la barra de herramientas de depuración en la parte superior de VS Code para continuar la ejecución, recorrer paso a paso las líneas por procedimientos, depurar métodos paso a paso por instrucciones o recorrer paso a paso para salir del método actual. 
   
   ![Alcanzar el punto de interrupción](./media/service-fabric-develop-java-applications-with-vs-code/breakpoint-hit.png)
       
4. Para finalizar la sesión de depuración, haga clic en el icono de enchufe de la barra de herramientas de depuración en la parte superior de VS Code.
   
   ![Desconexión del depurador](./media/service-fabric-develop-java-applications-with-vs-code/debug-bar-disconnect.png)
       
5. Cuando la depuración haya terminado, puede usar el comando **Service Fabric: Remove Application** (Service Fabric: quitar aplicación) para quitar la aplicación Voting del clúster local. 

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre el [desarrollo y la depuración de aplicaciones de C# para Service Fabric con VS Code](./service-fabric-develop-csharp-applications-with-vs-code.md).
