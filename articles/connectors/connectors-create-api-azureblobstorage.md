---
title: Conexión con Azure Blob Storage
description: Creación y administración de blobs en cuentas de Azure Storage con Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 02/21/2020
tags: connectors
ms.openlocfilehash: 34608a085c0d60e0ce07e5d198622f80a43f8b38
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284088"
---
# <a name="create-and-manage-blobs-in-azure-blob-storage-by-using-azure-logic-apps"></a>Creación y administración de blobs en Azure Blob Storage con Azure Logic Apps

En este artículo se muestra cómo se puede acceder y administrar los archivos almacenados como blobs en una cuenta de Azure Storage desde dentro de una aplicación de lógica con el conector de Azure Blob Storage. De este modo, puede crear aplicaciones lógicas que automatizan tareas y flujos de trabajo para administrar los archivos. Por ejemplo, puede crear aplicaciones lógicas que creen, obtengan, actualicen y eliminen archivos en su cuenta de almacenamiento.

Imagine que tiene una herramienta que se actualiza en un sitio web de Azure, que actúa como desencadenador para su aplicación lógica. Cuando se produce este evento, la aplicación lógica puede actualizar algunos archivos en el contenedor de Blob Storage, que es una acción de la aplicación lógica.

Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) e [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para obtener información técnica específica del conector, consulte la [referencia sobre el conector de Azure Blob Storage](/connectors/azureblobconnector/).

> [!IMPORTANT]
> Las aplicaciones lógicas no pueden acceder directamente a cuentas de almacenamiento que se encuentren detrás de un firewall si se encuentran en la misma región. Como alternativa, puede tener las aplicaciones lógicas y la cuenta de almacenamiento en diferentes regiones. Para más información sobre cómo habilitar el acceso desde Azure Logic Apps a cuentas de almacenamiento que se encuentran detrás de los firewalls, consulte la sección [Acceso a cuentas de almacenamiento detrás de firewalls](#storage-firewalls) más adelante en este tema.

<a name="blob-storage-limits"></a>

## <a name="limits"></a>límites

* De forma predeterminada, las acciones de Azure Blob Storage pueden leer o escribir archivos de *50 MB o más pequeños*. Para controlar los archivos mayores de 50 MB, pero hasta 1024 MB, las acciones de Azure Blob Storage admiten la [fragmentación de mensajes](../logic-apps/logic-apps-handle-large-messages.md). La acción **Obtener contenido de blog** usa implícitamente la fragmentación.

* Los desencadenadores de Azure Blob Storage no admiten la fragmentación. Cuando se solicita el contenido del archivo, los desencadenadores seleccionan solo los archivos que tienen un tamaño de 50 MB o menos. Para obtener archivos de más de 50 MB, siga este patrón:

  * Use un desencadenador de Azure Blob Storage que devuelva las propiedades de archivo, como **Cuando se agrega o modifica un blob (solo propiedades)** .

  * Siga el desencadenador con la acción **Obtener contenido de blob** de Azure Blob Storage, que lee el archivo completo y utiliza la fragmentación implícitamente.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Una [cuenta de Azure Storage y un contenedor de almacenamiento](../storage/blobs/storage-quickstart-blobs-portal.md)

* La aplicación lógica en la que necesita acceso a la cuenta de Azure Blob Storage. Para iniciar la aplicación lógica con un desencadenador de Azure Blob Storage, necesita una [aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="add-trigger"></a>

## <a name="add-blob-storage-trigger"></a>Agregar un desencadenador de Blob Storage

En Azure Logic Apps, cada aplicación lógica debe comenzar con un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts), que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Logic Apps crea una instancia de aplicación lógica y empieza a ejecutar el flujo de trabajo de la aplicación.

En este ejemplo se muestra cómo se puede iniciar un flujo de trabajo de una aplicación lógica con el desencadenador **Cuando se agrega o modifica un blob (solo propiedades)** cuando las propiedades de un blob se agregan o se actualizan en un contenedor de almacenamiento.

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, cree una aplicación lógica en blanco que abra el diseñador de aplicaciones lógicas. En este ejemplo se usa Azure Portal.

2. En el cuadro de búsqueda, escriba "azure blob" como filtro. En la lista de desencadenadores, seleccione el que desee.

   En este ejemplo se utiliza este desencadenador: **Cuando se agrega o modifica un blob (solo propiedades)**

   ![Seleccionar desencadenador de Azure Blob Storage](./media/connectors-create-api-azureblobstorage/add-azure-blob-storage-trigger.png)

3. Si se le piden los detalles de la conexión, [cree ahora su conexión de Blob Storage](#create-connection). O bien, si la conexión ya existe, especifique la información necesaria para el desencadenador.

   Para este ejemplo, seleccione el contenedor y la carpeta que desea supervisar.

   1. En el cuadro **Contenedor**, seleccione el icono de la carpeta.

   2. En la lista de carpetas, elija el corchete angular derecho ( **>** ) y, después, desplácese hasta que encuentre y seleccione la carpeta que desee.

      ![Seleccionar la carpeta de almacenamiento que se va a usar con el desencadenador](./media/connectors-create-api-azureblobstorage/trigger-select-folder.png)

   3. Seleccione el intervalo y la frecuencia con la que desea que el desencadenador compruebe si hay cambios en la carpeta.

4. Cuando esté listo, elija **Guardar** en la barra de herramientas del diseñador.

5. Ahora, agregue a la aplicación lógica una o varias acciones, en función de las tareas que desea realizar con los resultados del desencadenador.

<a name="add-action"></a>

## <a name="add-blob-storage-action"></a>Agregar un acción a Blob Storage

En Azure Logic Apps, una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) es un paso del flujo de trabajo que sigue a un desencadenador u otra acción. En este ejemplo, la aplicación lógica se inicia con el [desencadenador Periodicidad](../connectors/connectors-native-recurrence.md).

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, abra la aplicación lógica en el diseñador de aplicaciones lógicas. En este ejemplo se usa Azure Portal.

2. En el Diseñador de Logic Apps, en el desencadenador o la acción, elija **Nuevo paso**.

   ![Agregar un paso nuevo al flujo de trabajo de la aplicación lógica](./media/connectors-create-api-azureblobstorage/add-new-step-logic-app-workflow.png) 

   Para agregar una acción entre los pasos existentes, mueva el mouse sobre la flecha de conexión. Elija el signo más ( **+** ) que aparece y seleccione **Agregar una acción**.

3. En el cuadro de búsqueda, escriba "azure blob" como filtro. En la lista de acciones, seleccione la acción que desee.

   Este ejemplo utiliza esta acción: **Obtener contenido de blog**

   ![Seleccionar acción de Azure Blob Storage](./media/connectors-create-api-azureblobstorage/add-azure-blob-storage-action.png)

4. Si se le piden los detalles de la conexión, [cree ahora su conexión de Azure Blob Storage](#create-connection).
O bien, si la conexión ya existe, especifique la información necesaria para la acción.

   En este ejemplo, seleccione el archivo que desee.

   1. En el cuadro **Blob**, seleccione el icono de la carpeta.
  
      ![Seleccionar la carpeta de almacenamiento que se va a usar con la acción](./media/connectors-create-api-azureblobstorage/action-select-folder.png)

   2. Busque el archivo que desee por el número de **identificador** del blob y selecciónelo. Dicho número de **identificador** se puede encontrar en los metadatos del blob que devuelve el desencadenador de almacenamiento del blob descrito anteriormente.

5. Cuando esté listo, elija **Guardar** en la barra de herramientas del diseñador.
Para probar la aplicación lógica, asegúrese de que la carpeta seleccionada contiene un blob.

En este ejemplo solo se obtiene el contenido de un blob. Para ver dicho contenido, agregue otra acción que cree un archivo con el blob mediante otro conector. Por ejemplo, agregue una acción de OneDrive que cree un archivo basándose en el contenido del blob.

<a name="create-connection"></a>

## <a name="connect-to-storage-account"></a>Conectar con la cuenta de almacenamiento

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Cuando se le pida crear la conexión, proporcione esta información:

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Nombre de la conexión** | Sí | <*connection-name*> | El nombre que se va a crear para su conexión |
   | **Storage Account** | Sí | <*storage-account*> | Seleccione la cuenta de almacenamiento en la lista. |
   ||||

   Por ejemplo:

   ![Creación de una conexión de cuenta de Azure Blob Storage](./media/connectors-create-api-azureblobstorage/create-storage-account-connection.png) 

1. Cuando esté listo, seleccione **Crear**.

1. Después de crear la conexión, siga con [Agregar un desencadenador de Blob Storage](#add-trigger) o [Agregar una acción de Blob Storage](#add-action).

## <a name="connector-reference"></a>Referencia de conectores

Si necesita más detalles técnicos sobre este conector, como los desencadenadores, las acciones y los límites que se describen en el archivo de Swagger del conector, vea la [página de referencia del conector](/connectors/azureblobconnector/).

> [!NOTE]
> En el caso de las aplicaciones lógicas de un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión con la etiqueta ISE de este conector usa en su lugar los [límites de mensajes de ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

<a name="storage-firewalls"></a>

## <a name="access-storage-accounts-behind-firewalls"></a>Acceso a cuentas de almacenamiento detrás de firewalls

Puede agregar la seguridad de red a una cuenta de Azure Storage si se restringe el acceso con un [firewall y reglas de firewall](../storage/common/storage-network-security.md). Sin embargo, este programa de instalación crea un desafío para Azure y otros servicios de Microsoft que necesitan acceder a la cuenta de almacenamiento. La comunicación local en el centro de recursos abstrae las direcciones IP internas, por lo que no se pueden configurar reglas de firewall con restricciones de IP. Para más información, vea [Configuración de Firewalls y redes virtuales de Azure Storage](../storage/common/storage-network-security.md).

A continuación se muestran varias opciones acceder a las cuentas de almacenamiento detrás de los firewalls desde Azure Logic Apps mediante el conector de Azure Blob Storage u otras soluciones:

* Conector de blobs de Azure Storage

  * [Acceso a las cuentas de almacenamiento de otras regiones](#access-other-regions)
  * [Acceso a las cuentas de almacenamiento a través de una red virtual de confianza](#access-trusted-virtual-network)

* Otras soluciones

  * [Acceso a las cuentas de almacenamiento como un servicio de confianza con identidades administradas](#access-trusted-service)
  * [Acceso a las cuentas de almacenamiento a través de Azure API Management](#access-api-management)

<a name="access-other-regions"></a>

### <a name="problems-accessing-storage-accounts-in-the-same-region"></a>Problemas de acceso a cuentas de almacenamiento en la misma región

Las aplicaciones lógicas no pueden acceder directamente a cuentas de almacenamiento que se encuentren detrás de un firewall cuando se encuentran en la misma región. Como solución alternativa, coloque las aplicaciones lógicas en una región diferente a la de la cuenta de almacenamiento y proporcione acceso a las [direcciones IP de salida para los conectores administrados de la región](../logic-apps/logic-apps-limits-and-config.md#outbound).

> [!NOTE]
> Esta solución no se aplica al conector de Azure Table Storage y al conector de Azure Queue Storage. En su lugar, para acceder a Table Storage o Queue Storage, use las acciones y el desencadenador HTTP integrados.

<a name="access-trusted-virtual-network"></a>

### <a name="access-storage-accounts-through-a-trusted-virtual-network"></a>Acceso a las cuentas de almacenamiento a través de una red virtual de confianza

Puede colocar la cuenta de almacenamiento en una red virtual de Azure que administra y, a continuación, agregar esa red virtual a la lista de redes virtuales de confianza. Para que la aplicación lógica acceda a la cuenta de almacenamiento a través de una [red virtual de confianza](../virtual-network/virtual-networks-overview.md), debe implementar esa aplicación lógica en un [entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), que se puede conectar a los recursos de una red virtual. Después puede agregar las subredes de ese ISE a la lista de confianza. Los conectores de Azure Storage, como el conector de Blob Storage, pueden acceder directamente al contenedor de almacenamiento. Este programa de instalación es la misma experiencia que al usar los puntos de conexión de servicio de un ISE.

<a name="access-trusted-service"></a>

### <a name="access-storage-accounts-as-a-trusted-service-with-managed-identities"></a>Acceso a las cuentas de almacenamiento como un servicio de confianza con identidades administradas

Para conceder a los servicios de confianza de Microsoft acceso a una cuenta de almacenamiento a través de un firewall, puede configurar una excepción para esos servicios en esa cuenta de almacenamiento. Esta solución permite que los servicios de Azure que admiten las [entidades administradas para la autenticación](../active-directory/managed-identities-azure-resources/overview.md) accedan a las cuentas de almacenamiento detrás de firewalls como servicios de confianza. En concreto, para que una aplicación lógica de una instancia de Azure multiinquilino global acceda a estas cuentas de almacenamiento, primero debe [habilitar la compatibilidad con identidades administradas](../logic-apps/create-managed-service-identity.md) en la aplicación lógica. Luego, use el desencadenador o la acción HTTP en la aplicación lógica y [establezca su tipo de autenticación para usar la identidad administrada de la aplicación lógica](../logic-apps/create-managed-service-identity.md#authenticate-access-with-managed-identity). En este escenario, *solo* puede usar el desencadenador o la acción HTTP.

Para configurar la excepción y la compatibilidad con identidad administrada, siga estos pasos generales:

1. En la cuenta de almacenamiento, en **Configuración**, seleccione **Firewalls y redes virtuales**. En **Permitir acceso desde**, seleccione la opción **Redes seleccionadas** para que aparezca la configuración relacionada.

1. En **Excepciones**, seleccione **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento** y seleccione **Guardar**.

   ![Selección de una excepción que permite los servicios de confianza de Microsoft](./media/connectors-create-api-azureblobstorage/allow-trusted-services-firewall.png)

1. En la configuración de la aplicación lógica, [habilite la compatibilidad con la identidad administrada](../logic-apps/create-managed-service-identity.md).

1. En el flujo de trabajo de la aplicación lógica, agregue y configure la acción o el desencadenador HTTP para acceder a la entidad o la cuenta de almacenamiento.

   > [!IMPORTANT]
   > En el caso de llamadas del desencadenador o la acción HTTP a las cuentas de Azure Storage, asegúrese de que el encabezado de la solicitud incluya la propiedad `x-ms-version` y la versión de API de la operación que quiere ejecutar en la cuenta de almacenamiento. Para más información, consulte [Autenticación del acceso con la identidad administrada](../logic-apps/create-managed-service-identity.md#authenticate-access-with-managed-identity) y [Creación de versiones para los servicios de Azure Storage](/rest/api/storageservices/versioning-for-the-azure-storage-services#specifying-service-versions-in-requests).

1. En esa acción, [seleccione la identidad administrada](../logic-apps/create-managed-service-identity.md#authenticate-access-with-managed-identity) que se va a usar para la autenticación.

<a name="access-api-management"></a>

### <a name="access-storage-accounts-through-azure-api-management"></a>Acceso a las cuentas de almacenamiento a través de Azure API Management

Si usa un nivel dedicado para [API Management](../api-management/api-management-key-concepts.md), puede presentar la API de Azure Storage utilizando API Management y permitiendo las direcciones IP de esta última a través del firewall. Básicamente, agregue la red virtual de Azure que API Management usa a la configuración del firewall de la cuenta de almacenamiento. Después, puede usar la acción API Management o la acción HTTP para llamar a las API de Azure Storage. Aunque si elige esta opción, tendrá que controlar el proceso de autenticación personalmente. Para obtener más información, vea [Arquitectura de integración empresarial sencilla](https://aka.ms/aisarch).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)

