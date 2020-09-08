---
title: Envío de configuraciones a App Configuration con Azure Pipelines
description: Aprenda a usar Azure Pipelines para enviar pares clave-valor a un almacén de App Configuration
services: azure-app-configuration
author: lisaguthrie
ms.service: azure-app-configuration
ms.topic: how-to
ms.date: 07/27/2020
ms.author: lcozzens
ms.openlocfilehash: c1142ef7b37dee916118964778f6c1db2a65d591
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88719592"
---
# <a name="push-settings-to-app-configuration-with-azure-pipelines"></a>Envío de configuraciones a App Configuration con Azure Pipelines

La tarea [Azure App Configuration Push](https://marketplace.visualstudio.com/items?itemName=AzureAppConfiguration.azure-app-configuration-task-push) envía pares clave-valor de un archivo de configuración en el almacén de App Configuration. Esta tarea habilita la funcionalidad completa dentro de la canalización, ya que ahora es posible extraer la configuración del almacén de App Configuration, así como enviar la configuración al mismo destino.

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
- Recurso de App Configuration: cree uno de forma gratuita en [Azure Portal](https://portal.azure.com).
- Proyecto de Azure DevOps: [cree uno gratis](https://go.microsoft.com/fwlink/?LinkId=2014881).
- Tarea Azure App Configuration Push: descárguela de manera gratuita de [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=AzureAppConfiguration.azure-app-configuration-task-push#:~:text=Navigate%20to%20the%20Tasks%20tab,the%20Azure%20App%20Configuration%20instance.).

## <a name="create-a-service-connection"></a>Creación de una conexión de servicio

Una conexión de servicio permite acceder a los recursos en la suscripción de Azure desde el proyecto de Azure DevOps.

1. En Azure DevOps, vaya al proyecto que contiene la canalización de destino y abra la **Configuración del proyecto** en la parte inferior izquierda.
1. En **Canalizaciones**, seleccione **Conexiones de servicio** y seleccione **Nueva conexión de servicio** en la parte superior derecha.
1. Seleccione **Azure Resource Manager**.
1. Seleccione **Entidad de servicio (automática)** .
1. Rellene la suscripción y el recurso. Asigne un nombre a la conexión del servicio.

Ahora que ha creado la conexión de servicio, busque el nombre de la entidad de servicio asignada a ella. En el paso siguiente, agregará una nueva asignación de roles a esta entidad de servicio.

1. Vaya a **Configuración del proyecto** > **Conexiones de servicio**.
1. Seleccione la conexión de servicio que creó en la sección anterior.
1. Seleccione **Administrar entidad de servicio**.
1. Tome nota del **nombre para mostrar** que se muestra.

## <a name="add-role-assignment"></a>Agregar asignación de roles

Asigne las asignaciones de roles de App Configuration adecuadas a las credenciales que se van a usar en la tarea para que esta pueda acceder al almacén de App Configuration.

1. Navegue al almacén de App Configuration de destino. 
1. En el lado izquierdo, seleccione **Control de acceso (IAM)** .
1. En la parte superior, seleccione **+ Agregar** y elija **Agregar asignación de roles**.
1. En **Rol**, seleccione **Propietario de los datos de App Configuration**. Este rol permite que la tarea lea y escriba en el almacén de App Configuration. 
1. Seleccione la entidad de servicio asociada con la conexión de servicio que creó en la sección anterior.
  
## <a name="use-in-builds"></a>Uso en compilaciones

En esta sección se explicará cómo usar la tarea Azure App Configuration Push en una canalización de compilación de Azure DevOps.

1. Vaya a la página de canalización de compilación al hacer clic en **Canalizaciones** > **Canalizaciones**. La documentación para las canalizaciones de compilación se puede encontrar [aquí](https://docs.microsoft.com/azure/devops/pipelines/create-first-pipeline?view=azure-devops&tabs=tfs-2018-2).
      - Si va a crear una nueva canalización de compilación, seleccione **Mostrar ayudante** en el lado derecho de la canalización y busque la tarea **Azure App Configuration Push**.
      - Si usa una canalización de compilación existente, vaya a la pestaña **Tareas** al editar la canalización y busque la tarea **Azure App Configuration Push**.
2. Configure los parámetros necesarios para que la tarea envíe los pares clave-valor del archivo de configuración al almacén de App Configuration. El parámetro **Ruta de archivo de configuración** comienza en la raíz del repositorio de archivos.
3. Guarde y ponga en cola una compilación. El registro de compilación mostrará los errores que se produjeron durante la ejecución de la tarea.

## <a name="use-in-releases"></a>Uso en versiones

En esta sección se explicará cómo usar la tarea Azure App Configuration Push en las canalizaciones de versión de Azure DevOps.

1. Vaya a la página de canalización de versión; para ello, seleccione **Canalizaciones** > **Versiones**. La documentación para las canalizaciones de versión se puede encontrar [aquí](https://docs.microsoft.com/azure/devops/pipelines/release?view=azure-devops).
1. Elija una canalización de versión existente. Si no tiene una, seleccione **+ Nueva** para crear una nueva.
1. Seleccione el botón **Editar** en la esquina superior derecha para editar la canalización de versión.
1. Elija la **Fase** para agregar la tarea. Puede encontrar más información sobre las fases [aquí](https://docs.microsoft.com/azure/devops/pipelines/release/environments?view=azure-devops).
1. Seleccione **+** para ese trabajo y, a continuación, agregue la tarea **Azure App Configuration Push** en la pestaña **Implementar**.
1. Configure los parámetros necesarios dentro de la tarea para enviar los pares clave-valor del archivo de configuración al almacén de App Configuration. Las explicaciones de los parámetros están disponibles en la sección **Parámetros** siguiente y en la información sobre herramientas junto a cada parámetro.
1. Guarde y ponga en cola una versión. El registro de versión mostrará los errores que se hayan encontrado durante la ejecución de la tarea.

## <a name="parameters"></a>Parámetros

La tarea App Configuration Push usa los siguientes parámetros:

- **Suscripción de Azure**: una lista desplegable que contiene las conexiones de servicio de Azure disponibles. Para actualizar la lista de conexiones de servicio de Azure disponibles, presione el botón **Refresh Azure subscription** (Actualizar suscripción a Azure) a la derecha del cuadro de texto.
- **Nombre de App Configuration**: una lista desplegable que carga los almacenes de configuración disponibles para la suscripción seleccionada. Para actualizar la lista de almacenes de configuración disponibles, presione el botón **Refresh App Configuration Name** (Actualizar nombre de App Configuration) a la derecha del cuadro de texto.
- **Ruta del archivo de configuración**: la ruta de acceso a su archivo de configuración. Puede examinar el artefacto de compilación para seleccionar un archivo de configuración. (Botón `...` a la derecha del cuadro de texto).
- **Separador**: el separador que se usa para aplanar los archivos .yml y .json.
- **Profundidad**: la profundidad con la que se aplanarán los archivos .yml y .json.
- **Prefijo**: una cadena que se anexa al principio de cada clave enviada al almacén de App Configuration.
- **Etiqueta**: una cadena que se agrega a cada par clave-valor como etiqueta en el almacén de App Configuration.
- **Tipo de contenido**: una cadena que se agrega a cada par clave-valor como tipo de contenido en el almacén de App Configuration.
- **Etiquetas**: un objeto JSON en el formato `{"tag1":"val1", "tag2":"val2"}`, que define las etiquetas que se agregaron a cada par clave-valor enviado al almacén de App Configuration.
- **Elimine todos los demás pares clave-valor en el almacén con el prefijo y la etiqueta especificados**: El valor predeterminado es **Sin activar**.
  - **Activado**: quita todos los pares clave-valor en el almacén de App Configuration que coinciden con el prefijo y la etiqueta especificados antes de enviar nuevos pares clave-valor del archivo de configuración.
  - **Sin activar**: envía todos los pares clave-valor del archivo de configuración al almacén de App Configuration y deja intacto el resto del contenido del almacén.

Después de rellenar los parámetros necesarios, ejecute la canalización. Todos los pares clave-valor del archivo de configuración especificado se cargarán en App Configuration.

## <a name="troubleshooting"></a>Solución de problemas

Si se produce un error inesperado, los registros de depuración se pueden habilitar; para ello, establezca la variable de canalización `system.debug` en `true`.

## <a name="faq"></a>Preguntas más frecuentes

**¿Cómo puedo cargar varios archivos de configuración?**

Cree varias instancias de la tarea Azure App Configuration Push dentro de la misma canalización para enviar varios archivos de configuración al almacén de App Configuration.

**¿Por qué recibo un error 409 al intentar enviar pares clave-valor a mi almacén de configuración?**

Se producirá un mensaje de error de conflicto 409 si la tarea intenta quitar o sobrescribir un par clave-valor que está bloqueado en el almacén de App Configuration.
