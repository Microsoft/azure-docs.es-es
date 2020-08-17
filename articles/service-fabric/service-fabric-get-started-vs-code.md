---
title: Introducción a Azure Service Fabric con VS Code
description: En este artículo se ofrece una introducción a la creación de aplicaciones de Service Fabric mediante Visual Studio Code.
author: peterpogorski
ms.topic: article
ms.date: 06/29/2018
ms.author: pepogors
ms.custom: devx-track-javascript
ms.openlocfilehash: 1480b5e1ddab86761adb1c7caef808de89a3d947
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87428986"
---
# <a name="service-fabric-for-visual-studio-code"></a>Service Fabric para Visual Studio Code

La [extensión Service Fabric Reliable Services para VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-service-fabric-reliable-services) proporciona las herramientas necesarias para crear, compilar y depurar aplicaciones de Service Fabric en los sistemas operativos Windows, Linux y macOS.

En este artículo se proporciona información general sobre los requisitos y la configuración de la extensión, así como el uso de los distintos comandos que la extensión proporciona. 

> [!IMPORTANT]
> Las aplicaciones de Java para Service Fabric se pueden desarrollar en máquinas Windows, pero solo se pueden implementar en clústeres Linux de Azure. En Windows no se admite la depuración de aplicaciones de Java.

## <a name="prerequisites"></a>Prerrequisitos

Los siguientes requisitos previos deben instalarse en todos los entornos.

* [Visual Studio Code](https://code.visualstudio.com/)
* [Node.js](https://nodejs.org/)
* [Git](https://git-scm.com/)
* [SDK de Service Fabric](./service-fabric-get-started.md)
* Generadores Yeoman: instale los generadores adecuados para su aplicación

   ```sh
   npm install -g yo
   npm install -g generator-azuresfjava
   npm install -g generator-azuresfcsharp
   npm install -g generator-azuresfcontainer
   npm install -g generator-azuresfguest
   ```

Los siguientes requisitos previos deben instalarse para programar con Java:

* [SDK de Java](https://aka.ms/azure-jdks) (versión 1.8)
* [Gradle](https://gradle.org/install/)
* [Depurador para la extensión VS Code de Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-debug) Necesario para depurar los servicios de Java. La depuración de los servicios de Java solo se admite en Linux. Para instalar, puede hacer clic en el icono Extensiones en la **Barra de actividades** de VS Code y buscar la extensión, o puede usar VS Code Marketplace.

Los siguientes requisitos previos deben instalarse para programar con .NET Core o C#.

* [.NET Core](https://www.microsoft.com/net/learn/get-started) (versión 2.0.0 o posteriores)
* [Extensión VS Code de C# para Visual Studio Code (con tecnología OmniSharp)](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) Necesario para depurar los servicios de C#. Para instalar, puede hacer clic en el icono Extensiones en la **Barra de actividades** de VS Code y buscar la extensión, o puede usar VS Code Marketplace.

## <a name="setup"></a>Configurar

1. Abra VS Code.
2. Haga clic en el icono Extensiones en la **Barra de actividad** en el lateral izquierdo de VS Code. Busque "Service Fabric". Haga clic en **Instalar** para instalar la extensión de Service Fabric Reliable Services.

## <a name="commands"></a>Comandos:
La extensión de Service Fabric Reliable Services para VS Code ofrece muchos comandos para ayudar a los desarrolladores a crear e implementar proyectos de Service Fabric. Para llamar a los comandos desde la **paleta de comandos**, presione `(Ctrl + Shift + p)`, escriba el nombre de comando en la barra de entrada y seleccione el comando deseado en la lista de mensajes. 

* Service Fabric: Create Application (Crear aplicación) 
* Service Fabric: Publish Application (Publicar aplicación) 
* Service Fabric: Implementación de la aplicación 
* Service Fabric: Remove Application (Quitar aplicación)  
* Service Fabric: Build Application (Compilar aplicación) 
* Service Fabric: Clean Application (Limpiar aplicación) 

### <a name="service-fabric-create-application"></a>Service Fabric: Create Application (Crear aplicación)

El comando **Service Fabric: Create Application** (Crear aplicación) crea una aplicación de Service Fabric en el área de trabajo actual. Dependiendo de qué generadores Yeoman estén instalados en el equipo de desarrollo, puede crear varios tipos de aplicación de Service Fabric, incluidos proyectos de Java, C#, Container y Guest. 

1.  Seleccione el comando **Service Fabric: Comando Create Application** (Crear aplicación).
2.  Seleccione el tipo de la nueva aplicación de Service Fabric. 
3.  Escriba el nombre de la aplicación que desea crear.
3.  Seleccione el tipo de servicio que desea agregar a la aplicación de Service Fabric. 
4.  Siga las indicaciones para asignar un nombre al servicio. 
5.  La nueva aplicación de Service Fabric aparece en el área de trabajo.
6.  Abra la nueva carpeta de aplicación para que se convierta en la carpeta raíz del área de trabajo. Puede continuar la ejecución de los comandos desde aquí.

### <a name="service-fabric-add-service"></a>Service Fabric: Add Service (Agregar servicio)
El comando **Service Fabric: Add Service** (Agregar servicio) agrega un nuevo servicio a una aplicación de Service Fabric existente. La aplicación a la que se agregará el servicio debe ser el directorio raíz del área de trabajo. 

1.  Seleccione el comando **Service Fabric: Add Service** (Agregar servicio).
2.  Seleccione el tipo de la aplicación de Service Fabric actual. 
3.  Seleccione el tipo de servicio que desea agregar a la aplicación de Service Fabric. 
4.  Siga las indicaciones para asignar un nombre al servicio. 
5.  El nuevo servicio aparece en el directorio del proyecto. 

### <a name="service-fabric-publish-application"></a>Service Fabric: Publish Application (Publicar aplicación)
El comando **Service Fabric: Publish Application** (Publicar aplicación) implementa la aplicación de Service Fabric en un clúster remoto. El clúster de destino puede ser un clúster seguro o no seguro. Si no se establecen los parámetros en Cloud.json, la aplicación se implementa en el clúster local.

1.  La primera vez que se compila la aplicación, se genera un archivo Cloud.json en el directorio del proyecto.
2.  Escriba los valores para el clúster al que le gustaría conectarse en el archivo Cloud.json.
3.  Seleccione el comando **Service Fabric: Publish Application** (Publicar aplicación).
4.  Consulte el clúster de destino con Service Fabric Explorer para confirmar que la aplicación se ha instalado. 

### <a name="service-fabric-deploy-application-localhost"></a>Service Fabric: Deploy Application (Implementar aplicación) (Localhost)
El comando **Service Fabric: Deploy Application** (Implementar aplicación) implementa la aplicación de Service Fabric en un clúster local. Asegúrese de que el clúster local se está ejecutando antes de usar el comando. 

1. Seleccione el comando **Service Fabric: Deploy Application** (Implementar aplicación).
2. Consulte el clúster local con Service Fabric Explorer (\//localhost:19080/Explorer) para confirmar que la aplicación se ha instalado. Esto puede tardar algún tiempo, tenga paciencia.
3. También puede usar el comando **Service Fabric: Publish Application** (Publicar aplicación) sin parámetros establecidos en el archivo Cloud.json para implementar en un clúster local.

> [!NOTE]
> No se permite implementar aplicaciones de Java en el clúster local en equipos Windows.

### <a name="service-fabric-remove-application"></a>Service Fabric: Remove Application (Quitar aplicación)
El comando **Service Fabric: Remove Application** (Quitar aplicación) quita una aplicación de Service Fabric del clúster donde se había implementado previamente para usar la extensión de VS Code. 

1.  Seleccione el comando **Service Fabric: Remove Application** (Quitar aplicación).
2.  Consulte el clúster con Service Fabric Explorer para confirmar que la aplicación se ha quitado. Esto puede tardar algún tiempo, tenga paciencia.

### <a name="service-fabric-build-application"></a>Service Fabric: Build Application (Compilar aplicación)
El comando **Service Fabric: Build Application** (Compilar aplicación) puede compilar aplicaciones de Service Fabric para Java o C#. 

1.  Asegúrese de que está en la carpeta raíz de la aplicación antes de ejecutar este comando. El comando identifica el tipo de aplicación (C# o Java) y la compila como corresponda.
2.  Seleccione el comando **Service Fabric: Build Application**.
3.  La salida del proceso de compilación se envía al terminal integrado.

### <a name="service-fabric-clean-application"></a>Service Fabric: Clean Application (Limpiar aplicación)
El comando **Service Fabric: Clean Application** (Limpiar aplicación) elimina todos los archivos .jar y las bibliotecas nativas generadas por la compilación. Solo es válido para aplicaciones de Java. 

1.  Asegúrese de que está en la carpeta raíz de la aplicación antes de ejecutar este comando. 
2.  Seleccione el comando **Service Fabric: Clean Application** (Limpiar aplicación).
3.  La salida del proceso de limpieza se envía al terminal integrado.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre el [desarrollo y la depuración de aplicaciones de C# para Service Fabric con VS Code](./service-fabric-develop-csharp-applications-with-vs-code.md).
* Obtenga más información sobre el [desarrollo y la depuración de aplicaciones de Java para Service Fabric con VS Code](./service-fabric-develop-java-applications-with-vs-code.md).
