---
title: Tutorial de los conectores de Video Indexer con Logic Apps y Power Automate
description: En este tutorial se muestra cómo descubrir nuevas experiencias y oportunidades de monetización de los conectores de Video Indexer con Logic Apps y Power Automate.
author: anzaman
manager: johndeu
ms.author: alzam
ms.service: media-services
ms.subservice: video-indexer
ms.topic: tutorial
ms.date: 05/01/2020
ms.openlocfilehash: 5f29e616c0643914ca28921eee481105a5feb0c5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87047092"
---
# <a name="tutorial-use-video-indexer-with-logic-app-and-power-automate"></a>Tutorial: Uso de Video Indexer con Logic Apps y Power Automate

La [API REST de Video Indexer v2](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Delete-Video?) de Azure Media Services admite la comunicación servidor-servidor y cliente-servidor y permite que los usuarios de Video Indexer integren fácilmente información de vídeo y audio con la lógica de la aplicación, de forma que se descubren nuevas experiencias y oportunidades de monetización.

Para facilitar aún más la integración, se admiten los conectores [Logic Apps](https://azure.microsoft.com/services/logic-apps/) y [Power Automate](https://preview.flow.microsoft.com/connectors/shared_videoindexer-v2/video-indexer-v2/) que son compatibles con nuestra API. Puede usar los conectores para configurar flujos de trabajo personalizados con el fin de indexar y extraer información de una gran cantidad de archivos de audio y vídeo, sin necesidad de escribir una sola línea de código. Además, el uso de los conectores para la integración le proporciona una mayor visibilidad sobre el estado del flujo de trabajo y una manera fácil de depurarlo.  

Para ayudarle a empezar a trabajar rápidamente con los conectores de Video Indexer, haremos un tutorial de una aplicación lógica de ejemplo y una solución de Power Automate que puede configurar. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Cargar e indexar el vídeo automáticamente
> * Configurar el flujo de carga de archivos
> * Configurar el flujo de extracción de JSON

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para comenzar, también necesitará una cuenta de Video Indexer junto con el acceso a las API mediante la clave de API. 

Asimismo, deberá tener una cuenta de Azure Storage. Anote la clave de acceso de la cuenta de almacenamiento. Cree dos contenedores, uno para almacenar vídeos y otro para guardar los detalles generados por Video Indexer.  

A continuación, tendrá que abrir dos flujos independientes en Logic Apps o Power Automate (según cuál use).  

## <a name="upload-and-index-your-video-automatically"></a>Carga e indexación del vídeo automáticamente 

Este escenario consta de dos flujos diferentes que funcionan juntos. El primer flujo se desencadena cuando se agrega o modifica un blob en una cuenta de Azure Storage. Este flujo carga el nuevo archivo en Video Indexer con una dirección URL de devolución de llamada para enviar una notificación una vez completada la operación de indexación. El segundo flujo se desencadena en función de la dirección URL de devolución de llamada y guarda de nuevo la información extraída en un archivo JSON en Azure Storage. Esta estrategia de dos flujos se usa para admitir la carga asincrónica y la indexación de archivos más grandes de forma eficaz. 

### <a name="set-up-the-file-upload-flow"></a>Configuración del flujo de carga de archivos 

El primer flujo se desencadena siempre que se agrega un blob en el contenedor de Azure Storage. Una vez que se desencadena, se crea un URI de SAS que puede usar para cargar e indexar el vídeo en Video Indexer. Empiece por crear el siguiente flujo. 

![Flujo de carga de archivos](./media/logic-apps-connector-tutorial/file-upload-flow.png)

Para configurar el primer flujo, debe proporcionar la clave de API de Video Indexer y las credenciales de Azure Storage. 

![Almacenamiento de blobs de Azure](./media/logic-apps-connector-tutorial/azure-blob-storage.png)

![Nombre de la conexión y la clave de API](./media/logic-apps-connector-tutorial/connection-name-api-key.png)

Una vez que pueda conectarse a sus cuentas de Azure Storage y Video Indexer, vaya al desencadenador "Cuando se agrega o modifica un blob" y seleccione el contenedor en el que colocará los archivos de vídeo. 

![Contenedor](./media/logic-apps-connector-tutorial/container.png)

A continuación, vaya a la acción "Crear URI de SAS por ruta de acceso" y seleccione la lista de rutas de acceso de archivo en las opciones de contenido dinámico.  

![URI de SAS por ruta de acceso](./media/logic-apps-connector-tutorial/sas-uri-by-path.jpg)

Rellene [la ubicación y el identificador de la cuenta](./video-indexer-use-apis.md#account-id) para obtener el token de la cuenta de Video Indexer.

![Obtención del token de acceso de la cuenta](./media/logic-apps-connector-tutorial/account-access-token.png)

En "Upload video and index" (Cargar vídeo e indexar), rellene los parámetros necesarios y la dirección URL de vídeo. Seleccione "Add new parameter" (Agregar nuevo parámetro) y elija la dirección URL de devolución de llamada. 

![Carga e indexación](./media/logic-apps-connector-tutorial/upload-and-index.png)

Por ahora, deje en blanco la dirección URL de devolución de llamada. Solo la agregará después de finalizar el segundo flujo en el que se crea la dirección URL de devolución de llamada. 

Puede usar el valor predeterminado para los demás parámetros o configurarlos según sus necesidades. 

Haga clic en "Save" (Guardar). Ahora vamos a configurar el segundo flujo para extraer la información una vez completada la carga y la indexación. 

## <a name="set-up-the-json-extraction-flow"></a>Configuración del flujo de extracción de JSON 

La finalización de la carga y la indexación del primer flujo enviará una solicitud HTTP con la dirección URL de devolución de llamada correcta para desencadenar el segundo flujo. A continuación, recuperará la información generada por Video Indexer. En este ejemplo, se almacena la salida del trabajo de indexación en Azure Storage.  Sin embargo, es decisión suya lo que pueda hacer con la salida.  

Cree el segundo flujo separado del primero. 

![Flujo de extracción de JSON](./media/logic-apps-connector-tutorial/json-extraction-flow.png)

Para configurar este flujo, debe proporcionar de nuevo la clave de API de Video Indexer y las credenciales de Azure Storage. Tendrá que actualizar los mismos parámetros igual que hizo para el primer flujo. 

En el caso del desencadenador, verá un campo de dirección URL HTTP POST. La dirección URL no se generará hasta que se guarde el flujo; sin embargo, la necesitará en el futuro. Luego volveremos a esto. 

Rellene [la ubicación y el identificador de la cuenta](./video-indexer-use-apis.md#account-id) para obtener el token de la cuenta de Video Indexer.  

Vaya a la acción "Get Video Index" (Obtener índice de vídeo) y rellene los parámetros necesarios. Como identificador de vídeo, coloque la siguiente expresión: triggerOutputs()['queries']['id']. 

![información de acción de Video Indexer](./media/logic-apps-connector-tutorial/video-indexer-action-info.jpg)

Esta expresión indica al conector que obtenga el identificador de vídeo de la salida del desencadenador. En este caso, la salida del desencadenador será la salida de "Upload video and index" (Cargar vídeo e indexar) del primer desencadenador. 

Vaya a la acción "Create blob" (Crear blob) y seleccione la ruta de acceso a la carpeta en la que se guardará la información. Establezca el nombre del blob que va a crear. Como contenido del blob, escriba la siguiente expresión: body(‘Get_Video_Index’). 

![Creación de una acción de blob](./media/logic-apps-connector-tutorial/create-blob-action.jpg)

Esta expresión toma la salida de la acción "Get Video Index" (Obtener índice de vídeo) de este flujo. 

Haga clic en "Save flow" (Guardar flujo). 

Una vez guardado el flujo, se crea una dirección URL HTTP POST en el desencadenador. Copie la dirección URL del desencadenador. 

![Guardado de la dirección URL del desencadenador](./media/logic-apps-connector-tutorial/save-url-trigger.png)

Ahora, vuelva al primer flujo y pegue la dirección URL en la acción "Upload video and index" (Cargar vídeo e indexar) en el parámetro de la dirección URL de devolución de llamada. 

Asegúrese de que se guardan ambos flujos. Ya está listo para continuar. 

Pruebe la aplicación lógica que acaba de crear o la solución de Power Automate; para ello, agregue un vídeo al contenedor de blobs de Azure y regrese en unos minutos para ver que la información aparece en la carpeta de destino. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con este tutorial, puede mantener en funcionamiento esta aplicación lógica o solución de Power Automate si es necesario. Sin embargo, si no desea que se le facture por ello, desactive ambos flujos si usa Power Automate. Deshabilite ambos flujos si usa Logic Apps. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial se ha mostrado solo un ejemplo de los conectores de Video Indexer. Puede usar los conectores de Video Indexer en cualquier llamada API proporcionada por Video Indexer. Por ejemplo, cargar y recuperar información, traducir los resultados, obtener widgets insertables e incluso personalizar los modelos. Además, puede optar por desencadenar esas acciones en función de diferentes orígenes, como actualizaciones en repositorios de archivos o correos electrónicos enviados. Después, puede elegir que los resultados se actualicen a nuestra infraestructura o aplicación pertinente o generar un número cualquiera de elementos de acción.  

> [!div class="nextstepaction"]
> [Uso de Video Indexer API](video-indexer-use-apis.md)
