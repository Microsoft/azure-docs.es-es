---
title: 'Mediciones de usuario reales con Visual Studio Mobile Center: Azure Traffic Manager'
description: Configuración de la aplicación móvil desarrollada mediante Visual Studio Mobile Center para enviar medidas de usuarios reales a Traffic Manager
services: traffic-manager
documentationcenter: traffic-manager
author: rohinkoul
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/16/2018
ms.author: rohink
ms.custom: devx-track-javascript
ms.openlocfilehash: 6c16e8541c703aa574e4ddae8ed01571e027b261
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87431957"
---
# <a name="how-to-send-real-user-measurements-to-traffic-manager-with-visual-studio-mobile-center"></a>Envío de medidas de usuarios reales a Traffic Manager con Visual Studio Mobile Center

Puede configurar su aplicación móvil desarrollada mediante Visual Studio Mobile Center para enviar medidas de usuarios reales a Traffic Manager siguiendo estos pasos:

>[!NOTE]
> Actualmente, el envío de medidas de usuarios reales a Traffic Manager solo se admite en Android.

Para configurar Real User Measurements, debe obtener una clave e instrumentar la aplicación con el paquete RUM.

## <a name="step-1-obtain-a-key"></a>Paso 1: Obtención de una clave
    
El servicio identifica las medidas que se realizan y envían a Traffic Manager desde la aplicación cliente mediante una cadena única, llamada clave de Real User Measurements (RUM). Puede obtener una clave RUM mediante Azure Portal, una API de REST o a través de las interfaces de PowerShell o de la CLI.

Para obtener la clave RUM mediante Azure Portal, use el procedimiento siguiente:
1. En un explorador, inicie sesión en Azure Portal. Si aún no tiene una cuenta, puede registrarse para disfrutar de una evaluación gratuita de un mes.
2. En la barra de búsqueda del portal, busque el nombre del perfil de Traffic Manager que desea modificar y haga clic en él en los resultados que se muestran.
3. En la página de perfil de Traffic Manager, haga clic en **Real User Measurements** en **Configuración**.
4. Haga clic en **Generar clave** para crear una nueva clave RUM.
        
   ![Generación de una clave de Real User Measurements](./media/traffic-manager-create-rum-visual-studio/generate-rum-key.png)

   **Figura 1: Generación de la clave de Real User Measurements**

5. La página muestra la clave RUM que se genera y un fragmento de código de JavaScript que debe insertarse en la página HTML.
 
   ![Código JavaScript para la clave de Real User Measurements](./media/traffic-manager-create-rum-visual-studio/rum-key.png)

   **Figura 2: Clave de Real User Measurements y JavaScript de medida**
 
6. Haga clic en el botón **Copiar** para copiar la clave RUM. 

## <a name="step-2-instrument-your-app-with-the-rum-package-of-mobile-center-sdk"></a>Paso 2: Instrumentación de la aplicación con el paquete RUM del SDK de Mobile Center

Si no está familiarizado con Visual Studio Mobile Center, visite su [sitio web](https://mobile.azure.com). Para obtener instrucciones detalladas sobre la integración con el SDK, consulte [Getting Started with the Android SDK](https://docs.microsoft.com/mobile-center/sdk/getting-started/Android) (Introducción al SDK de Android).

Para usar Real User Measurements, realice el procedimiento siguiente:

1.  Agregue el SDK al proyecto.

    Durante la versión preliminar del SDK de RUM de ATM, debe hacer referencia explícitamente al repositorio de paquetes.

    En su archivo **app/build.gradle**, agregue las siguientes líneas:

    ```groovy
    repositories {
        maven {
            url "https://dl.bintray.com/mobile-center/mobile-center-snapshot"
        }
    }
    ```
    En su archivo **app/build.gradle**, agregue las siguientes líneas:

    ```groovy
    dependencies {
     
        def mobileCenterSdkVersion = '0.12.1-16+3fe5b08'
        compile "com.microsoft.azure.mobile:mobile-center-rum:${mobileCenterSdkVersion}"
    }
    ```

2. Inicie el SDK.

    Abra la clase de actividad principal de la aplicación y agregue las siguientes instrucciones Import:

    ```java
    import com.microsoft.azure.mobile.MobileCenter;
    import com.microsoft.azure.mobile.rum.RealUserMeasurements;
    ```

    Busque la devolución de llamada `onCreate` en el mismo archivo y agregue el código siguiente:

    ```java
    RealUserMeasurements.setRumKey("<Your RUM Key>");
    MobileCenter.start(getApplication(), "<Your Mobile Center AppSecret>", RealUserMeasurements.class);
    ```

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre [Real User Measurements](traffic-manager-rum-overview.md).
- Aprenda [cómo funciona el Administrador de tráfico](traffic-manager-overview.md)
- Obtenga más información sobre [Mobile Center](https://docs.microsoft.com/mobile-center/).
- [Suscríbase](https://mobile.azure.com) a Mobile Center.
- Aprenda más sobre los [métodos de enrutamiento de tráfico](traffic-manager-routing-methods.md) que admite el Administrador de tráfico.
- Aprenda a [crear un perfil del Administrador de tráfico](traffic-manager-create-profile.md)

