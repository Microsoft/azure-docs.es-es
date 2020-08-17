---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 02/10/2020
ms.author: erhopf
ms.openlocfilehash: 517b092f933bedae0f254461d9bfee4d4640db8b
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87374894"
---
Para crear un proyecto de Visual Studio C++ para el desarrollo de escritorio, debe configurar las opciones de desarrollo de Visual Studio, crear el proyecto, seleccionar la arquitectura de destino e instalar el SDK de voz.

### <a name="set-up-visual-studio-development-options"></a>Configuración de las opciones de desarrollo de Visual Studio

Para empezar, asegúrese de que Visual Studio está configurado correctamente para el desarrollo de escritorio de C++:

1. Abra Visual Studio 2019 para mostrar la ventana **Inicio**.

   ![Ventana Inicio: Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/vs-start-window.png)

1. Seleccione **Continuar sin código** para ir al IDE de Visual Studio.

1. En la barra de menús de Visual Studio, seleccione **Herramientas** > **Get Tools and Features** (Obtener herramientas y características) para abrir el Instalador de Visual Studio y ver el cuadro de diálogo **Modificar**.

   ![Pestaña Cargas de trabajo, cuadro de diálogo Modificar, Instalador de Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-cpp-workload.png)

1. En la pestaña **Cargas de trabajo**, en **Windows**, busque la carga de trabajo del **desarrollo de escritorio con C++** . Si la casilla que hay junto a esa carga de trabajo no está seleccionada, selecciónela.

1. En la pestaña **Componentes individuales**, busque la casilla **Administrador de paquetes NuGet**. Si la casilla no está seleccionada todavía, selecciónela.

1. Seleccione el botón de la esquina con la etiqueta **Cerrar** o **Modificar**. (El nombre del botón varía en función de si seleccionó alguna característica para la instalación). Si selecciona **Modificar**, la instalación comienza, lo que puede tardar unos minutos.

1. Cierre el Instalador de Visual Studio.

### <a name="create-the-project-and-select-the-target-architecture"></a>Cree el proyecto y seleccione la arquitectura de destino.

Después, cree el proyecto.

1. En la barra de menús de Visual Studio, elija **Archivo** > **Nuevo** > **Proyecto** para mostrar la ventana **Crear un nuevo proyecto**.

   ![Creación de un proyecto en C++: Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/qs-cpp-windows-01-new-console-app.png)

1. Busque y seleccione **Aplicación de consola**. Asegúrese de seleccionar la versión de C++ de este tipo de proyecto (en lugar de C# o de Visual Basic).

1. Seleccione **Siguiente** para abrir la pantalla **Configurar el nuevo proyecto**.

   ![Configuración de un nuevo proyecto en C++: Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-cpp-configure-your-new-project.png)

1. En **Nombre del proyecto**, escriba `helloworld`.

1. En **Ubicación**, vaya la carpeta en la que desea guardar el proyecto y selecciónela o créela.

Ahora seleccione la arquitectura de la plataforma de destino. En la barra de herramientas de Visual Studio, busque el cuadro desplegable **Plataformas de solución**. (Si no lo ve, elija **Ver** > **Barra de herramientas** > **Estándar** para mostrar la barra de herramientas que contiene **Plataformas de solución**). Si está ejecutando Windows de 64 bits, elija **x64** en el cuadro desplegable. Windows de 64 bits puede ejecutar también aplicaciones de 32 bits, por lo que puede elegir **x86**, si lo prefiere.

### <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

Por último, instale el [paquete NuGet del SDK de voz](https://aka.ms/csspeech/nuget) y haga referencia al SDK de voz en el proyecto:

1. En el **Explorador de soluciones**, haga clic con el botón derecho en la solución y elija **Manage NuGet Packages for Solution** (Administrar paquetes NuGet para la solución) para ir a la ventana **NuGet: solución**.

1. Haga clic en **Examinar**.

   ![NuGet: pestaña solución, Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/qs-cpp-windows-03-manage-nuget-packages.png)

1. En **Origen del paquete**, elija **nuget.org**.

1. En el cuadro de búsqueda **Buscar**, escriba `Microsoft.CognitiveServices.Speech` y, a continuación, elija el paquete cuando aparezca en los resultados de la búsqueda.

   ![Instalación del paquete C++ Microsoft.CognitiveServices.Speech: Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/qs-cpp-windows-04-nuget-install-1.0.0.png)

1. En el panel de estado del paquete situado junto a los resultados de la búsqueda, seleccione el proyecto **HelloWorld**.

1. Seleccione **Instalar**.

1. En el cuadro de diálogo **Vista previa de los cambios**, seleccione **Aceptar**.

1. En el cuadro de diálogo **Aceptación de licencia**, vea la licencia y, a continuación, seleccione **Acepto**. La instalación del paquete comienza y, cuando se completa, el panel **Salida** muestra un mensaje similar al siguiente: `Successfully installed 'Microsoft.CognitiveServices.Speech 1.13.0' to helloworld`.
