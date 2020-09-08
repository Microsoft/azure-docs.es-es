---
title: Obtención de imágenes con drones
description: En este artículo se describe cómo obtener imágenes con drones de los asociados.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: 04f0c3c63d00ea49bf43f00f256266599a73d6c0
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88508811"
---
# <a name="get-drone-imagery-from-drone-partners"></a>Obtención de imágenes con drones de asociados

En este artículo se describe cómo puede incorporar los datos del ortomosaico de las imágenes con drones de los asociados al centro de datos de Azure FarmBeats. Un ortomosaico es una imagen o ilustración aérea que se corrige geométricamente y se ensambla a partir de los datos recopilados por los drones.

Actualmente se admiten los siguientes asociados de imágenes.

  ![Asociados de imágenes de dron de FarmBeats](./media/get-drone-imagery-from-drone-partner/drone-partner-1.png)

La integración de datos de imágenes de dron con Azure FarmBeats le ayuda a obtener datos del ortomosaico de los vuelos de dron que realiza en la granja de servidores en el centro de datos. Una vez que los datos estén disponibles, puede verlos en el acelerador de FarmBeats. Los datos se pueden usar para la fusión de datos y la creación de modelos de inteligencia artificial y de aprendizaje automático.

## <a name="before-you-begin"></a>Antes de empezar

  - Asegúrese de que ha instalado Azure FarmBeats. Para obtener información sobre cómo instalar FarmBeats, consulte [Instalación de Azure FarmBeats](install-azure-farmbeats.md).
  - Asegúrese de que tiene la granja para la que desea las imágenes de dron definida en el sistema de FarmBeats.

## <a name="enable-drone-imagery-integration-with-farmbeats"></a>Habilitación de la integración de imágenes de dron con FarmBeats

Para habilitar la integración con FarmBeats, proporcione la siguiente información al proveedor de datos:
 - Punto de conexión de API
 - Id. de inquilino
 - Id. de cliente
 - Secreto del cliente

Siga estos pasos.

1. Descargue este [script](https://aka.ms/farmbeatspartnerscript) y extráigalo en la unidad local. Hay dos archivos en el archivo zip.
2. Inicie sesión en [Azure Portal](https://portal.azure.com/) y abra Azure Cloud Shell. Esta opción está disponible en la barra de herramientas de la esquina superior derecha del portal.

    ![Abra Azure Cloud Shell en la barra superior derecha del portal.](./media/get-drone-imagery-from-drone-partner/navigation-bar-1.png)

3. Asegúrese de que el entorno esté establecido en **PowerShell**.

    ![Configuración de PowerShell](./media/get-drone-imagery-from-drone-partner/power-shell-new-1.png)

4. Cargue los dos archivos que descargó en el paso 1 en la instancia de Cloud Shell.

    ![Carga de archivos](./media/get-drone-imagery-from-drone-partner/power-shell-two-1.png)

5. Vaya al directorio donde se cargaron los archivos. De forma predeterminada, el archivo se carga en el directorio particular bajo el nombre de usuario.
6. Ejecute el siguiente script:

    ```azurepowershell-interactive

    ./generateCredentials.ps1

    ```

7. Siga las instrucciones en pantalla para capturar los valores del punto de conexión de API, el identificador de inquilino, el identificador de cliente, el secreto de cliente y la cadena de conexión de EventHub.

    Después de escribir las credenciales necesarias en el sistema de software de drones del asociado, puede importar todas las granjas del sistema FarmBeats. Después, puede usar los detalles de la granja para planear la ruta de vuelo y recopilar imágenes con drones.

    Una vez que el software de los proveedores de los drones haya procesado las imágenes sin procesar, el sistema del software de drones carga el ortomosaico unido y otras imágenes procesadas en el centro de datos.

## <a name="view-drone-imagery"></a>Visualización de imágenes con drones

Cuando los datos se hayan enviado al centro de datos de FarmBeats, podrá consultar el almacén de escenas mediante las API del centro de datos de FarmBeats.

Como alternativa, puede ver la imagen de dron más reciente en la página **Farm Details** (Detalles de la granja). Para ver la imagen, siga los pasos.

1. Seleccione la granja en la que se han cargado las imágenes. Se mostrará la página de detalles de **Farm** (Granja).
2. Desplácese hacia abajo hasta la sección **Precision Maps** (Mapas de precisión).
3. Vea la imagen en la sección **Drone Imagery** (Imágenes de dron).

    ![Visualización de imágenes con drones](./media/get-drone-imagery-from-drone-partner/drone-imagery-1.png)

## <a name="download-drone-imagery"></a>Descarga de imágenes de dron

Al seleccionar la sección de imágenes de dron, se abre un elemento emergente que muestra una imagen de alta resolución del ortomosaico del dron.

![Ortomosaico de alta resolución](./media/get-drone-imagery-from-drone-partner/download-drone-imagery-1.png)

## <a name="view-all-drone-maps"></a>Visualización de todos los mapas de dron

Los archivos y las imágenes cargados por el proveedor de drones aparecen en la sección **Maps** (Mapas). Seleccione la sección **Maps** (Mapas), filtre por **Farm** (Granja) y elija los archivos que desee ver y descargar.

  ![Sección Maps (Mapas)](./media/get-drone-imagery-from-drone-partner/view-drone-maps-1.png)

## <a name="next-steps"></a>Pasos siguientes

Aprenda a usar las [API](rest-api-in-azure-farmbeats.md) del centro de datos de FarmBeats para obtener las imágenes de dron.
