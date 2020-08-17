---
title: Acceso y administración de archivos en Microsoft OneDrive
description: Cargue y administre archivos en OneDrive mediante la creación de flujos de trabajo automatizados en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 10/18/2016
tags: connectors
ms.openlocfilehash: 9fe4988b5499943f4b11ec5d640209ceb68e84ec
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87040235"
---
# <a name="access-and-manage-files-in-onedrive-connector-by-using-azure-logic-apps"></a>Acceda a archivos y adminístrelos en el conector de OneDrive mediante Azure Logic Apps

Con [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y el [conector de OneDrive](/connectors/onedriveconnector/), puede crear tareas automatizadas y flujos de trabajo para administrar los archivos, como cargar, obtener, eliminar archivos, etc. Con OneDrive, puede realizar estas tareas:

* Crear un flujo de trabajo almacenando archivos en OneDrive o actualizar las archivos que ya tenga en OneDrive. 
* Usar desencadenadores para iniciar el flujo de trabajo cuando se crea o se actualiza un archivo en OneDrive.
* Usar acciones para crear o eliminar un archivo, entre otras muchas cosas. Por ejemplo, cuando se reciba un nuevo correo electrónico de Office 365 con datos adjuntos (desencadenador), cree un nuevo archivo en OneDrive (acción).

En este artículo se muestra cómo usar un conector de OneDrive en una aplicación lógica, y además se enumeran los desencadenadores y las acciones.

Para más información sobre Logic Apps, consulte [¿Qué son las aplicaciones lógicas?](../logic-apps/logic-apps-overview.md) y [Creación de una aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="connect-to-onedrive"></a>Conexión a OneDrive

Antes de que la aplicación lógica pueda acceder a cualquier servicio, cree primero una *conexión* a este. Una conexión proporciona conectividad entre una aplicación lógica y otro servicio. Por ejemplo, para conectarse a OneDrive, primero necesita una *conexión* de OneDrive. Para crear una conexión, escriba las credenciales que utiliza normalmente para acceder al servicio al que desea conectarse. Por lo tanto, con OneDrive, escriba las credenciales de la cuenta de OneDrive para crear la conexión.

### <a name="create-the-connection"></a>Creación de la conexión

[!INCLUDE [Steps to create a connection to OneDrive](../../includes/connectors-create-api-onedrive.md)]

## <a name="use-a-trigger"></a>Uso de un desencadenador

Un desencadenador es un evento que se puede utilizar para iniciar el flujo de trabajo definido en una aplicación lógica. Los desencadenadores "sondean" el servicio en el intervalo y la frecuencia que desee. [Más información sobre los desencadenadores](../logic-apps/logic-apps-overview.md#logic-app-concepts).

1. En el Diseñador de aplicación lógica, escriba `onedrive` para obtener una lista de los desencadenadores:  

   ![Un cuadro de diálogo titulado "Mostrar las API administradas por Microsoft" tiene un cuadro que contiene "onedrive". A continuación se muestra una lista de los cuatro desencadenadores. El primero de ellos es "OneDrive: Cuando se crea un archivo". El segundo, "OneDrive: Cuando se modifica un archivo", se ha seleccionado.](./media/connectors-create-api-onedrive/onedrive-1.png)

2. Seleccione **Cuando se modifica un archivo**. Si ya existe una conexión, seleccione el botón Mostrar Selector para seleccionar una carpeta.

   ![El cuadro de diálogo titulado "Cuando se modifica un archivo" tiene el cuadro denominado "CARPETA" con un botón de búsqueda asociado.](./media/connectors-create-api-onedrive/sample-folder.png)

   Si se le solicita que inicie sesión, escriba los datos de inicio de sesión para crear la conexión. En la sección [Creación de la conexión](connectors-create-api-onedrive.md#create-the-connection) de este artículo se enumeran los pasos.

   En este ejemplo, la aplicación lógica se ejecuta cuando un archivo de la carpeta que elija se actualiza. Para ver los resultados de este desencadenador, agregue otra acción que envíe un correo electrónico. Por ejemplo, agregue la acción *Enviar un correo electrónico* de Office 365 Outlook que le envía un correo electrónico cuando se actualiza un archivo.

3. Seleccione el botón **Editar** y defina los valores de **Frecuencia** e **Intervalo**. Por ejemplo, si desea que el desencadenador sondee cada 15 minutos, establezca el valor de **Frecuencia** en **Minuto** y el de **Intervalo** en **15**. 

   ![El cuadro de diálogo titulado "Cuando se modifica un archivo" muestra cinco cuadros con la etiqueta: "CARPETA", "FRECUENCIA", "INTERVALO", "ZONA HORARIA" y "HORA DE INICIO". Hay listas desplegables para los campos "FRECUENCIA" y "ZONA HORARIA".](./media/connectors-create-api-onedrive/trigger-properties.png)

4. **Guarde** los cambios (esquina superior izquierda de la barra de herramientas). La aplicación lógica se guarda y se puede habilitar automáticamente.

## <a name="use-an-action"></a>Uso de una acción

Una acción es una operación que se lleva a cabo mediante el flujo de trabajo definido en una aplicación lógica. [Más información acerca de las acciones](../logic-apps/logic-apps-overview.md#logic-app-concepts).

1. Seleccione el signo más. Aparecen varias opciones: **Agregar una acción**, **Agregar una condición** o una de las opciones de **Más**.

   ![Captura de pantalla se muestran cuatro botones: + Nuevo paso", "Agregar una acción", "Agregar una condición" y "...Más".](./media/connectors-create-api-onedrive/add-action.png)

2. Elija **Add an action**(Agregar una acción).

3. En el cuadro de búsqueda, escriba `onedrive` para obtener una lista de todas las acciones disponibles.

   ![Un cuadro de diálogo titulado "Mostrar las API administradas por Microsoft" tiene un cuadro que contiene "onedrive". A continuación se muestra una lista de ocho acciones. El primero es "OneDrive: Crear archivo" y está seleccionado.](./media/connectors-create-api-onedrive/onedrive-actions.png) 

4. En nuestro ejemplo, elija **OneDrive - Crear archivo**. Si ya existe una conexión, seleccione la **Ruta de la carpeta** en la que se incluirá el archivo, escriba el **Nombre de archivo** y elija el **Contenido de archivo** que desee:  

   ![El cuadro de diálogo "Crear archivo" muestra tres cuadros con la etiqueta "RUTA DE ACCESO DE LA CARPETA", "NOMBRE DE ARCHIVO" y "CONTENIDO DE LA CARPETA". Hay un botón de búsqueda de directorios junto al cuadro "RUTA DE ACCESO DE LA CARPETA".](./media/connectors-create-api-onedrive/sample-action.png)

   Si se le solicita la información de conexión, escriba los detalles para [crear la conexión como se describe](#create-the-connection) en este tema.

   En este ejemplo, creará un nuevo archivo en una carpeta de OneDrive. Puede utilizar la salida de otro desencadenador para crear el archivo de OneDrive. Por ejemplo, agregue el desencadenador *Cuando llega un nuevo correo electrónico* de Office 365 Outlook. A continuación, agregue la acción *Crear archivo* de OneDrive, que usa los campos Attachments y Content-Type de una instrucción ForEach para crear el nuevo archivo en OneDrive.

   ![Un cuadro de diálogo titulado "Para cada uno" tiene un cuadro con la etiqueta "SELECCIONAR UNA SALIDA DE LOS PASOS ANTERIORES" que contiene "Datos adjuntos". Hay un cuadro de diálogo "Crear archivo" que cubre el resto del cuadro "Para cada uno", con cuadros con la etiqueta "RUTA DE ACCESO DE LA CARPETA", "NOMBRE DE ARCHIVO" y "CONTENIDO DEL ARCHIVO". ](./media/connectors-create-api-onedrive/foreach-action.png)

5. **Guarde** los cambios (esquina superior izquierda de la barra de herramientas). La aplicación lógica se guarda y se puede habilitar automáticamente.

## <a name="connector-specific-details"></a>Detalles específicos del conector

Vea los desencadenadores y las acciones definidos en Swagger y vea también todos los límites en los [detalles del conector](/connectors/onedriveconnector/).

## <a name="next-steps"></a>Pasos siguientes

* [Conectores para Azure Logic Apps](apis-list.md)