---
title: 'Inicio rápido: Creación de un asistente de voz mediante comandos personalizados'
titleSuffix: Azure Cognitive Services
description: En este inicio rápido, creará y probará una aplicación básica de Comandos personalizados mediante Speech Studio.
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: sausin
ms.openlocfilehash: 05b47a786fe845460177b66b5bd54cdb140c246e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289422"
---
# <a name="create-a-voice-assistant-using-custom-commands"></a>Creación de un asistente de voz mediante comandos personalizados

En este inicio rápido, creará y probará una aplicación básica de Comandos personalizados mediante Speech Studio. También podrá acceder a esta aplicación desde una aplicación cliente de Windows. **Comandos personalizados** facilita la creación de aplicaciones de comandos de voz enriquecidas y optimizadas para las experiencias de interacción que priorizan la voz. Proporcionan una experiencia de creación unificada, un modelo de hospedaje automático y una complejidad relativamente inferior, lo que ayuda a centrarse en la creación de la mejor solución para sus escenarios de comandos de voz.

## <a name="region-availability"></a>Disponibilidad regional
En este momento, los Comandos personalizados admiten suscripciones de Voz creadas en estas regiones:
* Oeste de EE. UU.
* Oeste de EE. UU. 2
* Este de EE. UU.
* Este de EE. UU. 2
* Norte de Europa
* Oeste de Europa
* Centro-Oeste de EE. UU.
* Centro de la India
* Este de Asia
* Sudeste de Asia

## <a name="prerequisites"></a>Requisitos previos

> [!div class="checklist"]
> * <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices" target="_blank">Cree un recurso de Voz de Azure en una región que admita Comandos personalizados.<span class="docon docon-navigate-external x-hidden-focus"></span></a> Consulte la sección **Disponibilidad regional** anterior para ver una lista de las regiones compatibles.
> * Descargue el archivo JSON de ejemplo [Smart Room Lite](https://aka.ms/speech/cc-quickstart).
> * Descargue la versión más reciente del [cliente de asistente de voz de Windows](https://aka.ms/speech/va-samples-wvac).

## <a name="go-to-the-speech-studio-for-custom-commands"></a>Navegación a Speech Studio para los comandos personalizados

1. En un explorador web, vaya a [Speech Studio](https://speech.microsoft.com/).
1. Introduzca sus credenciales para iniciar sesión en el portal.

   La vista predeterminada es la lista de suscripciones de Voz.
   > [!NOTE]
   > Si no ve la página para seleccionar una suscripción, puede navegar a ella al seleccionar "Recursos de voz" en el menú de configuración de la barra superior.

1. Seleccione su suscripción a voz y, a continuación, seleccione **Ir a Studio**.
1. Seleccione **Comandos personalizados**.

   La vista predeterminada es una lista de las aplicaciones de comandos personalizados de la suscripción seleccionada.

## <a name="import-an-existing-application-as-a-new-custom-commands-project"></a>Importación de una aplicación existente como un nuevo proyecto de Comandos personalizados

1. Seleccione **Nuevo proyecto** para crear un proyecto.

1. En el cuadro **Nombre**, escriba el nombre del proyecto `Smart-Room-Lite` (u otro de su elección).
1. En la lista **Idioma**, seleccione **Inglés (Estados Unidos)** .
1. Seleccione **Examinar archivos** y, en la ventana Examinar, seleccione el archivo **SmartRoomLite.json**.

    > [!div class="mx-imgBorder"]
    > ![Creación de un proyecto](media/custom-commands/import-project.png)

1.  En la lista **Recurso de creación de LUIS**, seleccione un recurso de creación. Si no hay ninguno válido, créelo. Para ello, seleccione **Create new LUIS authoring resource** (crear nuevo recurso de creación LUIS).

    > [!div class="mx-imgBorder"]
    > ![Creación de un recurso](media/custom-commands/create-new-luis-resource.png)
    
    
    1. En el cuadro **Nombre de recurso**, escriba el nombre del recurso.
    1. En la lista **Grupo de recursos**, seleccione uno.
    1. En la lista desplegable **Ubicación**, seleccione una.
    1. En la lista **Plan de tarifa**, seleccione un nivel.
    
    
    > [!NOTE]
    > Para crear grupos de recursos, escriba el nombre del grupo de recursos deseado en el campo "Grupo de recursos". El grupo de recursos se creará cuando se seleccione **Crear**.


1. A continuación, seleccione **Crear** para crear el proyecto.
1. Después de crear el proyecto, selecciónelo.
Ahora se debe mostrar una visión general de la nueva aplicación de Comandos personalizados.

## <a name="try-out-some-voice-commands"></a>Prueba de algunos comandos de voz
1. Seleccione **Train** (Entrenar) en la parte superior del panel derecho.
1. Una vez completado el entrenamiento, seleccione **Probar** y pruebe las siguientes expresiones:
    - Turn on the tv (enciende el televisor)
    - Establecer la temperatura en 80 grados
    - Apagarlo
    - El televisor
    - Establecer una alarma a las 5 p.m.

## <a name="integrate-custom-commands-application-in-an-assistant"></a>Integración de la aplicación de Comandos personalizados en un asistente
Para acceder a esta aplicación desde fuera de Speech Studio, debe publicarla. Para publicar una aplicación, tendrá que configurar el recurso LUIS de predicción.  

### <a name="update-prediction-luis-resource"></a>Actualización del recurso LUIS de predicción


1. Seleccione **Configuración** en el panel izquierdo y **LUIS resources** (recursos de LUIS) en el panel central.
1. Seleccione un recurso de predicción o cree uno seleccionando **Crear nuevo recurso**.
1. Seleccione **Guardar**.
    
    > [!div class="mx-imgBorder"]
    > ![Establecimiento de recursos de LUIS](media/custom-commands/set-luis-resources.png)

> [!NOTE]
> Ya que el recurso de creación solo admite 1000 solicitudes de punto de conexión de predicción al mes, deberá establecer obligatoriamente un recurso de predicción LUIS antes de publicar la aplicación de Comandos personalizados.

### <a name="publish-the-application"></a>Publicación de la aplicación

Seleccione **Publicar** en la parte superior del panel derecho. Una vez completada la publicación, se abrirá una nueva ventana. Anote el valor del **Id. de la aplicación** y la **Clave de recurso de voz**. Necesitará estos dos valores para acceder a la aplicación desde fuera de Speech Studio.

También puede obtener estos valores al seleccionar la sección **Configuración** > **General**.

### <a name="access-application-from-client"></a>Acceso a la aplicación desde el cliente

En este artículo, se usará el cliente del asistente de voz de Windows que descargó como parte de los requisitos previos. Descomprima la carpeta.
1. Inicie **VoiceAssistantClient.exe**.
1. Cree un nuevo perfil de publicación y especifique el valor de **Perfil de conexión**. En la sección **General Settings** (Configuración general), escriba los valores de **Subscription Key** (Clave de suscripción), que es el mismo que el valor de **Speech resource key** (Clave del recurso de Voz) que guardó al publicar la aplicación; **Subscription key region** (Región de clave de suscripción) y **Custom commands app ID** (Id. de la aplicación de comandos personalizados).
    > [!div class="mx-imgBorder"]
    > ![Creación de perfil de WVAC](media/custom-commands/create-profile.png)
1. Seleccione **Save and Apply Profile** (guardar y aplicar perfil).
1. Ahora pruebe las siguientes entradas mediante voz o texto.
    > [!div class="mx-imgBorder"]
    > ![Creación de perfil de WVAC](media/custom-commands/conversation.png)


> [!TIP]
> Puede hacer clic en las entradas del **Registro de actividad** para inspeccionar las respuestas sin procesar envía el servicio de Comandos personalizados.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, se usó una aplicación existente. A continuación, en las [secciones de procedimientos](how-to-custom-commands-create-application-with-simple-commands.md), aprenderá a diseñar, desarrollar, depurar, probar e integrar una aplicación de Comandos personalizados desde cero.
