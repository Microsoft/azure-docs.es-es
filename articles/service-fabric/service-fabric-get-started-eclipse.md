---
title: Complemento de Azure Service Fabric para Eclipse
description: Obtenga información sobre cómo empezar a usar Azure Service Fabric en Java mediante Eclipse y el complemento que Service Fabric proporciona.
author: rapatchi
ms.topic: conceptual
ms.date: 04/06/2018
ms.author: rapatchi
ms.custom: devx-track-java
ms.openlocfilehash: 004fb022847432a9739e79f7063b80636f8dcf2b
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87374242"
---
# <a name="service-fabric-plug-in-for-eclipse-java-application-development"></a>Complemento de Service Fabric para el desarrollo de aplicaciones Java de Eclipse
Eclipse es uno de los entornos de desarrollo integrado (IDE) por los desarrolladores de Java. En este artículo se describe cómo configurar un entorno de desarrollo de Eclipse para que funcione con Azure Service Fabric. Aprenderá a instalar el complemento de Service Fabric, crear una aplicación de Service Fabric e implementarla en un clúster de Service Fabric local o remoto en Eclipse. 

> [!NOTE]
> El complemento de Eclipse no se admite actualmente en Windows. 

> [!IMPORTANT]
> Asegúrese de que JDK 8 esté instalado en el sistema y seleccionado en Eclipse.

## <a name="install-or-update-the-service-fabric-plug-in-in-eclipse"></a>Instalación o actualización del complemento de Service Fabric en Eclipse
En Eclipse se puede instalar un complemento Service Fabric. El complemento puede ayudar a simplificar el proceso de compilación e implementación de servicios de Java.

> [!IMPORTANT]
> El complemento de Service Fabric requiere Eclipse Neon o una versión posterior. Consulte las instrucciones que siguen a esta nota acerca de cómo comprobar la versión de Eclipse. Si tiene una versión anterior de Eclipse instalado, puede descargar las versiones más recientes de el [sitio web de Eclipse](https://www.eclipse.org). No se recomienda instalar (sobrescribir) sobre una instalación existente de Eclipse. Puede eliminarla antes de ejecutar el programa de instalación o instalar la versión más reciente en un directorio diferente. 
> 
> En Ubuntu, se recomienda instalar directamente desde el sitio de Eclipse, en lugar de usar un instalador de paquetes (`apt` o `apt-get`). Si lo hace así, se asegura de que tendrá la versión más reciente de Eclipse. 

Instale Neon Eclipse o una versión más reciente desde el [sitio de Eclipse](https://www.eclipse.org).  Instale también la versión 2.2.1 o posterior de Buildship (el complemento de Service Fabric no es compatible con versiones anteriores de Buildship):
-   Para comprobar las versiones de los componentes instalados, en Eclipse, vaya a **Help** >  (Ayuda) **About Eclipse** >  (Acerca de Eclipse) **Installation Details** (Detalles de instalación).
-   Para actualizar Buildship, consulte [Eclipse Buildship: Complementos de Eclipse para Gradle][buildship-update].
-   Para buscar actualizaciones de Eclipse e instalarlas, vaya a **Help** > **Check for Updates** (Ayuda > Buscar actualizaciones).

Para instalar el complemento de Service Fabric, en Eclipse, vaya a **Help** >  (Ayuda) **Install New Software** (Instalar software nuevo).
1. En el cuadro **Trabajar con**, escriba \//dl.microsoft.com/eclipse.
2. Haga clic en **Agregar**.

   ![Complemento de Service Fabric para Eclipse][sf-eclipse-plugin-install]
3. Seleccione el complemento Service Fabric y, después, haga clic en **Next** (Siguiente).
4. Complete los pasos de la instalación y acepte los términos de licencia del software de Microsoft.
  
Si el complemento de Service Fabric ya está instalado, instale la versión más reciente. 
1. Para comprobar si hay actualizaciones disponibles, vaya a **Help** >  (Ayuda) **About Eclipse** >  (Acerca de Eclipse) **Installation Details** (Detalles de instalación). 
2. En la lista de complementos instalados, seleccione Service Fabric y, después, haga clic en **Update** (Actualizar). Se instalarán las actualizaciones disponibles.
3. Cuando actualiza el complemento de Service Fabric, también actualiza el proyecto de Gradle.  Haga clic con el botón derecho en **build.gradle** y, luego, seleccione **Refresh** (Actualizar).

> [!NOTE]
> Si el proceso de instalación o actualización del complemento Service Fabric se realiza lentamente, es posible que se deba a la configuración de Eclipse. Eclipse recopila metadatos de todos los cambios para actualizar los sitios que están registrados en la instancia de Eclipse. Para acelerar el proceso de búsqueda e instalación de una actualización del complemento Service Fabric, vaya a **Available Software Sites** (Sitios de software disponibles). Desactive las casillas de todos los sitios, excepto la del que apunta a la ubicación del complemento Service Fabric (https:\//dl.microsoft.com/eclipse/azure/servicefabric).

> [!NOTE]
>Si Eclipse no funciona según lo previsto en el equipo Mac (o necesita que ejecute como superusuario), vaya a la carpeta **ECLIPSE_INSTALLATION_PATH** y navegue a la subcarpeta **Eclipse.app/Contents/MacOS**. Inicie Eclipse ejecutando `./eclipse`.


## <a name="create-a-service-fabric-application-in-eclipse"></a>Creación de una aplicación de Service Fabric en Eclipse

1.  En Eclipse, vaya a **File** > **New** > **Other** (Archivo > Nuevo > Otro). Seleccione **Service Fabric Project** (Proyecto de Service Fabric) y haga clic en **Next** (Siguiente).

    ![Página 1 del nuevo proyecto de Service Fabric][create-application/p1]

2.  Escriba el nombre del proyecto y haga clic en **Next** (Siguiente).

    ![Página 2 del nuevo proyecto de Service Fabric][create-application/p2]

3.  En la lista de plantillas, seleccione **Service Template** (Plantilla de servicio). Seleccione el tipo de plantilla de servicio (Actor, Stateless, Container o Guest Binary) y, después, haga clic en **Next** (Siguiente).

    ![Página 3 del nuevo proyecto de Service Fabric][create-application/p3]

4.  Escriba el nombre y los detalles del servicio y, después, haga clic en **Finish** (Finalizar).

    ![Página 4 del nuevo proyecto de Service Fabric][create-application/p4]

5. Al crear el primer proyecto de Service Fabric, en el cuadro de diálogo **Open Associated Perspective** (Abrir perspectiva asociada), haga clic en **Yes** (Sí).

    ![Página 5 del nuevo proyecto de Service Fabric][create-application/p5]

6.  Así es el nuevo proyecto:

    ![Página 6 del nuevo proyecto de Service Fabric][create-application/p6]

## <a name="build-a-service-fabric-application-in-eclipse"></a>Creación de una aplicación de Service Fabric en Eclipse

1.  Haga clic con el botón derecho en la nueva aplicación de Service Fabric y, después, seleccione **Service Fabric**.

    ![Menú contextual de Service Fabric][publish/RightClick]

2. En el menú contextual, seleccione una de las siguientes opciones:
    -   Para compilar la aplicación sin realizar una limpieza, haga clic en **Build Application** (Compilar aplicación).
    -   Para realizar una compilación limpia de la aplicación, haga clic en **Rebuild Application** (Recompilar aplicación).
    -   Para limpiar la aplicación de los artefactos compilados, haga clic en **Clean Application** (Limpiar aplicación).
     
## <a name="deploy-a-service-fabric-application-to-the-local-cluster-with-eclipse"></a>Implementación de una aplicación de Service Fabric en el clúster local con Eclipse

Después de haber creado la aplicación de Service Fabric, siga estos pasos para implementarla en el clúster local.

1. Si no ha iniciado el clúster local, siga las instrucciones de [Instalación de un clúster local](./service-fabric-get-started-linux.md#set-up-a-local-cluster) para iniciar el clúster local y asegúrese de que se está ejecutando.
2. Haga clic con el botón derecho en la aplicación de Service Fabric y, después, seleccione **Service Fabric**.

    ![Menú contextual de Service Fabric][publish/RightClick]

3.  En el menú contextual, haga clic en **Implementar aplicación**.
4.  Puede seguir el progreso de la operación de implementación en la ventana Consola.
5.  Para comprobar que la aplicación se ejecuta, abra Service Fabric Explorer en el clúster local en una ventana del explorador `http://localhost:19080/Explorer`. Expanda el nodo **Aplicaciones** y asegúrese de que la aplicación se está ejecutando. 

Para obtener información sobre cómo depurar la aplicación en Eclipse mediante el clúster local, consulte [Depuración de la aplicación de Service Fabric para Java con Eclipse](./service-fabric-debugging-your-application-java.md).

También puede implementar la aplicación en el clúster local mediante el comando **Publicar aplicación**:

1. Haga clic con el botón derecho en la aplicación de Service Fabric y, después, seleccione **Service Fabric**.
2. En el menú contextual, haga clic en **Publicar aplicación...** .
3. En la ventana **Publicar aplicación**, elija **PublishProfiles/Local.json** como el perfil de destino y haga clic en **Publicar**.

    ![Cuadro de diálogo local Publicar](./media/service-fabric-get-started-eclipse/localjson.png)

    De forma predeterminada, el perfil de publicación de Local.json está configurado para publicar en el clúster local. Para más información sobre los parámetros de conexión y de punto de conexión presentes en los perfiles de publicación, vea la sección siguiente.

## <a name="publish-your-service-fabric-application-to-azure-with-eclipse"></a>Publicación de la aplicación de Service Fabric en Azure con Eclipse

Para publicar la aplicación en la nube, siga estos pasos:

1. Para publicar la aplicación en un clúster seguro en la nube, necesita un certificado X.509 que se usan para comunicarse con el clúster. En entornos de desarrollo y pruebas, el certificado que se usa a menudo es el certificado del clúster. En entornos de producción, el certificado debe ser un certificado de cliente que es distinto al certificado de clúster. Necesita tanto el certificado como la clave privada. El archivo de certificado (y la clave) deben tener el formato PEM. Puede crear un archivo PEM que contenga el certificado y la clave privada a partir de un archivo PFX con el siguiente comando de openssl:

    ```bash
    openssl pkcs12 -in your-cert-file.pfx -out your-cert-file.pem -nodes -passin pass:your-pfx-password
    ```

   Si el archivo PFX no está protegido con contraseña, utilice `--passin pass:` para el último parámetro.

2. Abra el archivo **Cloud.json** bajo el directorio **PublishProfiles**. Deberá configurar las credenciales de seguridad y el punto de conexión del clúster correctamente para el clúster.

   - El campo `ConnectionIPOrURL` contiene la dirección IP o la dirección URL del clúster. Tenga en cuenta que el valor no contiene el esquema de dirección URL (`https://`).
   - De forma predeterminada el campo `ConnectionPort` debe ser `19080`, a menos que explícitamente haya cambiado este puerto para el clúster.
   - El campo `ClientKey` debe apuntar a un archivo .pem o .key con el formato PEM en el equipo local que contiene la clave privada para el certificado de cliente o de clúster.
   - El campo `ClientCert` debe apuntar a un archivo .pem o .crt con el formato PEM en el equipo local que contiene los datos de certificado para el certificado. de cliente o de clúster. 

     ```bash
     {
         "ClusterConnectionParameters":
         {
            "ConnectionIPOrURL": "lnxxug0tlqm5.westus.cloudapp.azure.com",
            "ConnectionPort": "19080",
            "ClientKey": "[path_to_your_pem_file_on_local_machine]",
            "ClientCert": "[path_to_your_pem_file_on_local_machine]"
         }
     }
     ```

2. Haga clic con el botón derecho en la aplicación de Service Fabric y, después, seleccione **Service Fabric**.
3. En el menú contextual, haga clic en **Publicar aplicación...** .
3. En la ventana **Publicar aplicación**, elija **PublishProfiles/Cloud.json** como el perfil de destino y haga clic en **Publicar**.

    ![Cuadro de diálogo Publicar en la nube](./media/service-fabric-get-started-eclipse/cloudjson.png)

4. Puede seguir el progreso de la operación de publicación implementación en la ventana Consola.
5. Para comprobar que la aplicación se está ejecutando, abra Service Fabric Explorer en el clúster de Azure en una ventana del explorador. Para el ejemplo anterior, esto sería: `https://lnxxug0tlqm5.westus.cloudapp.azure.com:19080/Explorer`. Expanda el nodo **Aplicaciones** y asegúrese de que la aplicación se está ejecutando. 


En clústeres seguros de Linux, si la aplicación contiene servicios de Reliable Services, también deberá configurar un certificado que los servicios puedan utilizar para llamar a las API de runtime de Service Fabric. Para obtener más información, consulte [Configure a Reliable Services app to run on Linux clusters](./service-fabric-configure-certificates-linux.md#configure-a-reliable-services-app-to-run-on-linux-clusters) (Configurar una aplicación de Reliable Services para ejecutarla en clústeres Linux).

Para realizar un recorrido rápido para saber cómo implementar una aplicación de Reliable Services para Service Fabric escrita en Java en un clúster de Linux seguro, consulte [Inicio rápido: Implementación de una aplicación de Reliable Services](./service-fabric-quickstart-java-reliable-services.md).

## <a name="deploy-a-service-fabric-application-by-using-eclipse-run-configurations"></a>Implementación de una aplicación de Service Fabric mediante las configuraciones de ejecución de Eclipse

Una forma alternativa de implementar una aplicación de Service Fabric es mediante las configuraciones de ejecución de Eclipse.

1. En Eclipse, vaya a **Run** > **Run Configurations** (Ejecutar > Ejecutar configuraciones).
2. En **Gradle Project** (Proyecto Gradle), seleccione la configuración de ejecución **ServiceFabricDeployer**.
3. En el panel derecho, en la pestaña **Arguments**  (Argumentos), asegúrese de que los parámetros **ip**, **port**, **clientCert** y **clientKey** están configurados correctamente para su implementación. De forma predeterminada, los parámetros se establecen para implementarse en el clúster local como en la captura de pantalla siguiente. Para publicar la aplicación en Azure puede modificar los parámetros para que contenga los detalles del punto de conexión y las credenciales de seguridad para el clúster de Azure. Para más información, consulte la sección anterior, [Publicación de la aplicación de Service Fabric en Azure con Eclipse](#publish-your-service-fabric-application-to-azure-with-eclipse).

    ![Cuadro de diálogo Run Configurations (Ejecutar configuraciones)](./media/service-fabric-get-started-eclipse/run-config-local.png)

5. Asegúrese de que **Working Directory** (Directorio de trabajo) apunta a la aplicación que desea implementar. Para cambiar la aplicación, haga clic en el botón **Workspace...** (Espacio de trabajo) y seleccione la aplicación que desee.
6. Haga clic en **Apply** (Aplicar) y luego en **Run** (Ejecutar).

La aplicación se compila e implementa en un momento. El estado de implementación se puede implementar en Service Fabric Explorer.  

## <a name="add-a-service-fabric-service-to-your-service-fabric-application"></a>Incorporación de un servicio de Service Fabric a una aplicación de Service Fabric

Para agregar un servicio de Service Fabric a una aplicación de Service Fabric existente, siga estos pasos:

1.  Haga clic con el botón derecho en el proyecto al que desea agregar un servicio y, después, haga clic en **Service Fabric**.

    ![Página 1 de Agregar servicio de Service Fabric][add-service/p1]

2.  Haga clic en **Add Service Fabric Service** (Agregar servicio de Service Fabric) y complete el conjunto de pasos necesarios para agregar un servicio al proyecto.
3.  Seleccione la plantilla de servicio que desea agregar al proyecto y haga clic en **Next** (Siguiente).

    ![Página 2 de Agregar servicio de Service Fabric][add-service/p2]

4.  Escriba el nombre del servicio (y otros detalles, en caso de que sea necesario) y, después, haga clic en el botón **Add Service** (Agregar servicio).  

    ![Página 3 de Agregar servicio de Service Fabric][add-service/p3]

5.  Después de agregar el servicio, la estructura general del proyecto tiene un aspecto similar al siguiente proyecto:

    ![Página 4 de Agregar servicio de Service Fabric][add-service/p4]

## <a name="edit-manifest-versions-of-your-service-fabric-java-application"></a>Edición de las versiones de manifiesto de su aplicación Java de Service Fabric

Para editar las versiones de manifiesto, haga clic con el botón derecho en el proyecto, vaya a **Service Fabric** y seleccione **Editar versiones de manifiesto...**  en el menú desplegable. En el asistente, puede actualizar las versiones del manifiesto de aplicación, el manifiesto de servicio y las versiones de los paquetes de **Code**, **Config** y **Data**.

Si activa la opción **Actualizar automáticamente las versiones de aplicación y servicio** y, a continuación, actualice una versión; las versiones de manifiesto se actualizarán automáticamente. Para ver un ejemplo, primero active la casilla y, después, actualice la versión de **Code** de 0.0.0 a 0.0.1 y haga clic en **Finalizar**; la versión del manifiesto de servicio y la versión del manifiesto de aplicación se actualizarán automáticamente a 0.0.1.

## <a name="upgrade-your-service-fabric-java-application"></a>Actualización de la aplicación Java de Service Fabric

Para un escenario de actualización, supongamos que creó el proyecto **App1** mediante el complemento Service Fabric de Eclipse. Para implementarlo, usó el complemento para crear una aplicación llamada **fabric:/App1Application**. El tipo de aplicación es **App1ApplicationType** y su versión es 1.0. Ahora desea actualizar la aplicación sin interrumpir la disponibilidad.

En primer lugar, realice en la aplicación los cambios que desee y vuelva a compilar el servicio modificado. Actualice el archivo de manifiesto del servicio modificado (ServiceManifest.xml) con las versiones actualizadas para el servicio (y Code, Config o Data, lo que corresponda). Modifique también el manifiesto de la aplicación (ApplicationManifest.xml) con el número de la versión actualizada de la aplicación y el servicio modificado.  

Para actualizar la aplicación mediante Eclipse, puede crear un perfil de configuración de ejecución duplicado. A continuación, utilícelo para actualizar la aplicación según sea necesario.

1.  Vaya a **Run** > **Run Configurations** (Ejecución > Configuraciones de ejecución). En el panel izquierdo, haga clic en la flecha pequeña situada a la izquierda de **Gradle Project** (Proyecto Gradle).
2.  Haga clic con el botón derecho en **ServiceFabricDeployer** y seleccione **Duplicate** (Duplicar). Escriba un nuevo nombre para esta configuración, por ejemplo, **ServiceFabricUpgrader**.
3.  En el panel derecho, en la pestaña **Arguments** (Argumentos), cambie **-Pconfig='deploy'** a **-Pconfig='upgrade'** y haga clic en **Apply** (Aplicar).

Este proceso crea y guarda un perfil de configuración de ejecución que se puede usar en cualquier momento para actualizar la aplicación. También obtiene la versión del tipo de aplicación más reciente del archivo de manifiesto de la aplicación.

La aplicación tarda unos minutos en actualizarse. La actualización de la aplicación se puede supervisar en Service Fabric Explorer.

## <a name="migrating-old-service-fabric-java-applications-to-be-used-with-maven"></a>Migración de aplicaciones de Java de Service Fabric anteriores para su uso con Maven
Recientemente se han transferido las bibliotecas de Java de Service Fabric desde el SDK de Java de Service Fabric al repositorio de Maven. Aunque las nuevas aplicaciones que genere con Eclipse generarán proyectos actualizados (que podrán trabajar con Maven), puede actualizar las aplicaciones existentes sin estado o de actor de Service Fabric, que utilizaban anteriormente el SDK de Java de Service Fabric, para que usen las dependencias de Java de Service Fabric de Maven. Siga los pasos mencionados [aquí](service-fabric-migrate-old-javaapp-to-use-maven.md) para asegurarse de que las aplicaciones anteriores funcionan con Maven.

## <a name="next-steps"></a>Pasos siguientes

- Para unos pasos rápidos sobre la creación de una aplicación de Reliable Services para Java y la implementación de dicha aplicación localmente y en Azure, consulte [Inicio rápido: Implementación de una aplicación de Reliable Services](./service-fabric-quickstart-java-reliable-services.md).
- Para información sobre cómo depurar una aplicación en el clúster local, consulte [Depuración de la aplicación de Service Fabric para Java con Eclipse](./service-fabric-debugging-your-application-java.md).
- Para aprender a supervisar y diagnosticar aplicaciones de Service Fabric, vea [Supervisión y diagnóstico de servicios en una configuración de desarrollo de máquina local](./service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux.md).

<!-- Images -->

[sf-eclipse-plugin-install]: ./media/service-fabric-get-started-eclipse/service-fabric-eclipse-plugin.png

[create-application/p1]:./media/service-fabric-get-started-eclipse/create-application/p1.png
[create-application/p2]:./media/service-fabric-get-started-eclipse/create-application/p2.png
[create-application/p3]:./media/service-fabric-get-started-eclipse/create-application/p3.png
[create-application/p4]:./media/service-fabric-get-started-eclipse/create-application/p4.png
[create-application/p5]:./media/service-fabric-get-started-eclipse/create-application/p5.png
[create-application/p6]:./media/service-fabric-get-started-eclipse/create-application/p6.png

[publish/Publish]: ./media/service-fabric-get-started-eclipse/publish/Publish.png
[publish/RightClick]: ./media/service-fabric-get-started-eclipse/publish/RightClick.png

[add-service/p1]: ./media/service-fabric-get-started-eclipse/add-service/p1.png
[add-service/p2]: ./media/service-fabric-get-started-eclipse/add-service/p2.png
[add-service/p3]: ./media/service-fabric-get-started-eclipse/add-service/p3.png
[add-service/p4]: ./media/service-fabric-get-started-eclipse/add-service/p4.png

<!-- Links -->
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship
