---
title: archivo de inclusión
description: archivo de inclusión
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 02/05/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 5f919a04b47aa6fdef9500f3d7e6bef4ddaa239e
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65515495"
---
1. Inicie sesión en la [consola Firebase](https://firebase.google.com/console/). Si aún no tiene uno, cree un nuevo proyecto de Firebase.
2. Después de crear el proyecto, seleccione **Add Firebase to your Android app** (Añade Firebase a tu aplicación de Android). 

    ![Agregar Firebase a una aplicación de Android](./media/notification-hubs-enable-firebase-cloud-messaging/notification-hubs-add-firebase-to-android-app.png)
3. En la página **Agregar Firebase a una aplicación de Android**, realice los pasos siguientes: 
    1. En **Nombre del paquete de Android**, copie el valor de su **applicationId** en el archivo **build.gradle** de la aplicación. En este ejemplo, es `com.fabrikam.fcmtutorial1app`. 

        ![Especifique el nombre del paquete.](./media/notification-hubs-enable-firebase-cloud-messaging/specify-package-name-fcm-settings.png)
    2. Seleccione **Registrar aplicación**. 
4. Seleccione **Descargar google-services.json**, guarde el archivo en la carpeta **app** del proyecto y, a continuación, seleccione **Siguiente**. 

    ![Descargue el archivo google-services.json.](./media/notification-hubs-enable-firebase-cloud-messaging/download-google-service-button.png)
5. Haga que los siguientes **cambios de configuración** en el proyecto en Android Studio. 
    1.  En el archivo**project-level build.gradle** (&lt;project&gt;/build.gradle), agregue la siguiente instrucción a la sección de **dependencias**. 

        ```
        classpath 'com.google.gms:google-services:4.0.1'
        ```
    2. En el archivo**app-level build.gradle** (&lt;project&gt;/&lt;app-module&gt;/build.gradle), agregue la siguiente instrucción a la sección de **dependencias**. 

        ```
        implementation 'com.google.firebase:firebase-core:16.0.1'
        ```

    3. Agregue la siguiente línea al final del archivo **app-level build.gradle** después de la sección de dependencias. 

        ```
        apply plugin: 'com.google.gms.google-services'
        ```        
    4. Seleccione **Sinc Now*** (Sincronizar ahora) en la barra de herramientas. 
 
        ![cambios de configuración de build.gradle](./media/notification-hubs-enable-firebase-cloud-messaging/build-gradle-configurations.png)
6. Seleccione **Siguiente** en la página. 
7. Seleccione **Omitir este paso** en la página. 

    ![Omita el último paso.](./media/notification-hubs-enable-firebase-cloud-messaging/skip-this-step.png)1. 
8. En la consola Firebase, seleccione el icono de la rueda dentada del proyecto. Luego, seleccione **Project Settings** (Configuración del proyecto).

    ![Seleccionar Project Settings (Configuración del proyecto)](./media/notification-hubs-enable-firebase-cloud-messaging/notification-hubs-firebase-console-project-settings.png)
4. Si aún no ha descargado el archivo **google-services.json** en la carpeta **app** del proyecto de Android Studio, puede hacerlo en esta página. 
5. Cambie a la pestaña **Cloud Messaging** de la parte superior. 
6. Copie y guarde la **clave del servidor** para su uso posterior. Este valor se utilizará para configurar el centro de notificaciones.
