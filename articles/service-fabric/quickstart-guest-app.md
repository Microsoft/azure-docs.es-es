---
title: Implementar rápidamente una aplicación existente en un clúster
description: Use un clúster de Azure Service Fabric para hospedar una aplicación existente de Node.js con Visual Studio.
ms.topic: conceptual
ms.date: 12/06/2017
ms.custom: devx-track-javascript
ms.openlocfilehash: 9b39e14b18a784274f1bac12072f4fa39de2bb87
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87430734"
---
# <a name="host-a-nodejs-application-on-azure-service-fabric"></a>Hospedaje de una aplicación de Node.js en Azure Service Fabric

Esta guía de inicio rápido le ayudará a implementar una aplicación existente (Node.js en este ejemplo) en un clúster de Service Fabric que se ejecuta en Azure.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, asegúrese de haber [configurado el entorno de desarrollo](service-fabric-get-started.md). Esto también incluye la instalación del SDK de Service Fabric y Visual Studio 2019 o 2015.

También es preciso que tenga una aplicación de Node.js existente para la implementación. Este inicio rápido usa un sitio web en Node.js simple que se puede descargar [aquí][download-sample]. Extraiga este archivo en su carpeta `<path-to-project>\ApplicationPackageRoot\<package-name>\Code\` después de crear el proyecto en el paso siguiente.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita][create-account].

## <a name="create-the-service"></a>Creación del servicio

Inicie Visual Studio como **administrador**.

Creación de un proyecto con `CTRL`+`SHIFT`+`N`

En el cuadro de diálogo **Nuevo proyecto**, elija **Nube > Aplicación de Service Fabric**.

Asigne el nombre **MyGuestApp** a la aplicación y presione **Aceptar**.

>[!IMPORTANT]
>Node.js puede superar fácilmente el límite de 260 caracteres en las rutas de acceso que tiene Windows. Use una ruta de acceso corta para el propio proyecto como **c:\code\svc1**. Si lo desea, puede seguir **[estas instrucciones](https://stackoverflow.com/a/41687101/1664231)** para habilitar las rutas de acceso de archivo largas en Windows 10.
   
![Cuadro de diálogo de proyecto nuevo en Visual Studio][new-project]

Puede crear cualquier tipo de servicio de Service Fabric en el cuadro de diálogo siguiente. Para esta guía de inicio rápido, elija **Archivo ejecutable invitado**.

Asigne al servicio el nombre **MyGuestService** y elija los siguientes valores en las opciones de la derecha:

| Configuración                   | Value |
| ------------------------- | ------ |
| Carpeta del paquete de código       | _&lt;la carpeta con la aplicación de Node.js&gt;_ |
| Comportamiento del paquete de código     | Copie el contenido de la carpeta al proyecto |
| Programa                   | node.exe |
| Argumentos                 | server.js |
| Carpeta de trabajo            | CodePackage |

Presione **Aceptar**.

![Cuadro de diálogo de servicio nuevo en Visual Studio.][new-service]

Visual Studio crea el proyecto de la aplicación y el proyecto de servicio de actor y los muestra en el Explorador de soluciones.

El proyecto de la aplicación (**MyGuestApp**) no contiene código directamente. En su lugar, hace referencia a un conjunto de proyectos de servicio. Además, contiene otros tres tipos de contenido:

* **Perfiles de publicación**  
Preferencias en cuanto a las herramientas para los diferentes entornos.

* **Scripts**  
Script de PowerShell para implementar o actualizar la aplicación.

* **Definición de la aplicación**  
Incluye el manifiesto de aplicación en *ApplicationPackageRoot*. Los archivos de aplicación asociados se encuentran en *ApplicationParameters*, donde se define la aplicación y se la puede configurar específicamente para un entorno determinado.
    
Para obtener información general del contenido del proyecto de servicio, consulte [Introducción a Reliable Services de Microsoft Azure Service Fabric](service-fabric-reliable-services-quick-start.md).

## <a name="set-up-networking"></a>Configuración de las redes

La aplicación de Node.js de ejemplo que estamos implementando utiliza el puerto **80** y necesitamos indicar a Service Fabric que el puerto está expuesto.

Abra el archivo **ServiceManifest.xml** en el proyecto. En la parte inferior del manifiesto, hay un conjunto `<Resources> \ <Endpoints>` con una entrada ya definida. Modifique dicha entrada para agregar `Port`, `Protocol` y `Type`. 

```xml
  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="MyGuestAppServiceTypeEndpoint" Port="80" Protocol="http" Type="Input" />
    </Endpoints>
  </Resources>
```

## <a name="deploy-to-azure"></a>Implementar en Azure

Si presiona **F5** y ejecuta el proyecto, este se implementa en el clúster local. Sin embargo, vamos a implementarlo en Azure.

Haga clic con el botón derecho en el proyecto y elija **Publicar...** , lo que abre un cuadro de diálogo para publicar en Azure.

![Cuadro de diálogo Publicar en Azure para un servicio de Service Fabric][publish]

Seleccione el perfil de destino **PublishProfiles\Cloud.xml**.

Si no lo ha hecho aún, elija la cuenta de Azure en la que va a realizar la implementación. Si aún no tiene ninguno, [regístrese para obtenerlo][create-account].

En **Punto de conexión**, seleccione el clúster de Service Fabric en que se va a realizar la implementación. Si no tiene ninguno, seleccione **&lt;Crear nuevo clúster... &gt;** y se abrirá una ventana del explorador en Azure Portal. Para más información, consulte [Creación de un clúster en Azure Portal](service-fabric-cluster-creation-via-portal.md#create-cluster-in-the-azure-portal). 

Al crear el clúster de Service Fabric, asegúrese de establecer la opción **Puntos de conexión personalizados** en **80**.

![Configuración del tipo de nodo de Service Fabric con el punto de conexión personalizado][custom-endpoint]

La creación de un clúster de Service Fabric nuevo tarda un tiempo en completarse. Una vez que se haya creado, vuelva al cuadro de diálogo Publicar y seleccione **&lt;Actualizar&gt;** . El nuevo clúster se enumera en el cuadro de lista desplegable; selecciónelo.

Pulse **Publicar** y espere a que finalice la implementación.

Esta operación puede tardar unos minutos. Una vez que se completa, la aplicación puede tardar unos minutos en estar totalmente disponible.

## <a name="test-the-website"></a>Prueba del sitio web

Una vez publicado su servicio, pruébelo en un explorador web. 

En primer lugar, abra Azure Portal y busque el servicio Service Fabric.

Compruebe la hoja de información general de la dirección del servicio. Use el nombre de dominio de la propiedad _Punto de conexión del cliente_. Por ejemplo, `http://mysvcfab1.westus2.cloudapp.azure.com`.

![Hoja de información general de Service Fabric en Azure Portal][overview]

Navegue hasta esta dirección, donde verá la respuesta de `HELLO WORLD`.

## <a name="delete-the-cluster"></a>Eliminación del clúster

No olvide eliminar todos los recursos que haya creado para este inicio rápido, ya que se le cobran.

## <a name="next-steps"></a>Pasos siguientes
Más información sobre [ejecutables de invitado](service-fabric-guest-executables-introduction.md).

<!-- Image References -->

[new-project]: ./media/quickstart-guest-app/new-project.png
[new-service]: ./media/quickstart-guest-app/template.png
[solution-exp]: ./media/quickstart-guest-app/solution-explorer.png
[publish]: ./media/quickstart-guest-app/publish.png
[overview]: ./media/quickstart-guest-app/overview.png
[custom-endpoint]: ./media/quickstart-guest-app/custom-endpoint.png

[download-sample]: https://github.com/MicrosoftDocs/azure-cloud-services-files/raw/temp/service-fabric-node-website.zip
[create-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
