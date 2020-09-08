---
title: Conexión a una base de datos de IBM Informix
description: Automatice las tareas y los flujos de trabajo que administran recursos almacenados en Informix de IBM mediante Azure Logic Apps
services: logic-apps
ms.suite: integration
author: gplarsen
ms.author: daberry
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 01/07/2020
tags: connectors
ms.openlocfilehash: 4995a91783c2302f3bda5cc9409f017248ca29fa
ms.sourcegitcommit: f1b18ade73082f12fa8f62f913255a7d3a7e42d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88761651"
---
# <a name="manage-ibm-informix-database-resources-by-using-azure-logic-apps"></a>Administración de recursos de base de datos Informix de IBM mediante Azure Logic Apps

Con [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y el [conector de Informix](/connectors/informix/), puede crear tareas y flujos de trabajo automatizados que administren recursos en una base de datos Informix de IBM. Este conector incluye un cliente de Microsoft que se comunica con equipos del servidor Informix remotos a través de una red TCP/IP, incluidas bases de datos basadas en la nube como Informix de IBM para Windows, que se ejecuta en la virtualización de Azure, y bases de datos locales al usarse la [puerta de enlace de datos local](../logic-apps/logic-apps-gateway-connection.md). Puede conectarse a estas plataformas y versiones de Informix si se han configurado para admitir conexiones cliente de Arquitectura distribuida de bases de datos relacionales (DRDA):

* IBM Informix 12.1
* IBM Informix 11.7

Este tema le muestra cómo usar el conector en una aplicación lógica para procesar operaciones de base de datos.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* En las bases de datos locales, [descargue e instale la puerta de enlace de datos local](../logic-apps/logic-apps-gateway-install.md) en un equipo local y, a continuación, [cree un recurso de puerta de enlace de datos de Azure en Azure Portal](../logic-apps/logic-apps-gateway-connection.md).

* La aplicación lógica en la que necesita acceso a la base de datos Informix. Este conector ofrece únicamente las acciones, por lo que la aplicación lógica debe iniciarse ya con un desencadenador, por ejemplo, el de [periodicidad](../connectors/connectors-native-recurrence.md). 

## <a name="add-an-informix-action"></a>Adición de una acción de Informix

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en el diseñador de aplicaciones lógicas, si aún no lo ha hecho.

1. En el paso en el que quiera agregar la acción de Informix, seleccione **Nuevo paso**.

   Para agregar una acción entre los pasos existentes, mueva el mouse sobre la flecha de conexión. Seleccione el signo más ( **+** ) que aparece y, luego, seleccione **Agregar una acción**.

1. En el cuadro de búsqueda, escriba `informix` como filtro. En la lista de acciones, seleccione la que desee, por ejemplo:

   ![Selección de la acción de Informix que se va a ejecutar](./media/connectors-create-api-informix/select-informix-connector-action.png)

   El conector proporciona estas acciones, que ejecutan las operaciones de base de datos correspondientes:

   * Obtener tablas: mostrar tablas de base de datos mediante una instrucción `CALL`
   * Obtener filas: leer todas las filas mediante una instrucción `SELECT *`
   * Obtener fila: leer una fila mediante una instrucción `SELECT WHERE`
   * Agregar una fila mediante una instrucción `INSERT`
   * Editar una fila mediante una instrucción `UPDATE`
   * Eliminar una fila mediante una instrucción `DELETE`

1. Si se le pide que proporcione detalles de conexión de su base de datos Informix, siga los [pasos para crear la conexión](#create-connection) y, a continuación, continúe con el siguiente paso.

1. Proporcione la información para la acción seleccionada:

   | Acción | Descripción | Propiedades y descripciones |
   |--------|-------------|-----------------------------|
   | **Obtener tablas** | Muestre tablas de base de datos ejecutando una instrucción CALL de Informix. | None |
   | **Obtener filas** | Recupere todas las filas de la tabla especificada ejecutando una instrucción `SELECT *` de Informix. | **Nombre de la tabla**: nombre de la tabla de Informix que desea <p><p>Para agregar otras propiedades a esta acción, selecciónelas en la lista **Agregar nuevo parámetro**. Para obtener más información, consulte el [tema de referencia del conector](/connectors/informix/). |
   | **Obtener fila** | Recupere una fila de la tabla especificada ejecutando una instrucción `SELECT WHERE` de Informix. | - **Nombre de la tabla**: nombre de la tabla de Informix que desea <br>- **Id. de fila**: Id. exclusivo para la fila, por ejemplo, `9999` |
   | **Insertar fila** | Agregue una fila a la tabla de Informix especificada ejecutando una instrucción `INSERT` de Informix. | - **Nombre de la tabla**: nombre de la tabla de Informix que desea <br>- **elemento**: fila con los valores que se van a agregar |
   | **Actualizar fila** | Cambie una fila en la tabla de Informix especificada ejecutando una instrucción `UPDATE` de Informix. | - **Nombre de la tabla**: nombre de la tabla de Informix que desea <br>- **Id. de fila**: Id. exclusivo para la fila que se va a actualizar, por ejemplo, `9999` <br>- **Fila**: la fila con los valores actualizados, por ejemplo, `102` |
   | **Eliminar fila** | Quite una fila de la tabla de Informix especificada ejecutando una instrucción `DELETE` de Informix. | - **Nombre de la tabla**: nombre de la tabla de Informix que desea <br>- **Id. de fila**: Id. exclusivo para la fila que se va a eliminar, por ejemplo, `9999` |
   ||||

1. Guarde la aplicación lógica. Ahora, [pruebe la aplicación lógica](#test-logic-app) o continúe la compilación de la aplicación lógica.

<a name="create-connection"></a>

## <a name="connect-to-informix"></a>Conexión a Informix

1. Si la aplicación lógica se conecta a una base de datos local, seleccione **Conectarse mediante una puerta de enlace de datos local**.

1. Proporcione la información de esta conexión y, después, seleccione **Crear**.

   | Propiedad | Propiedad JSON | Obligatorio | Valor de ejemplo | Descripción |
   |----------|---------------|----------|---------------|-------------|
   | Nombre de conexión | `name` | Sí | `informix-demo-connection` | Nombre que se va a usar para la conexión a su base de datos Informix |
   | Server | `server` | Sí | - Nube: `informixdemo.cloudapp.net:9089` <br>- Entorno local: `informixdemo:9089` | La dirección TCP/IP o alias en formato IPv4 o IPv6, seguido de dos puntos y un número de puerto TCP/IP |
   | Base de datos | `database` | Sí | `nwind` | Nombre de base de datos relacional de DRDA (RDBNAM) o nombre de base de datos Informix (dbname). Informix acepta una cadena de 128 bytes. |
   | Authentication | `authentication` | Solo en entornos locales | **Basic** o **Windows** (kerberos) | El tipo de autenticación que requiere su base de datos Informix. Esta propiedad aparece solo al seleccionar **Conectarse mediante una puerta de enlace de datos local**. |
   | Nombre de usuario | `username` | No | <*database-user-name*> | Nombre de usuario de la base de datos |
   | Contraseña | `password` | No | <*database-password*> | Contraseña de la base de datos |
   | Puerta de enlace | `gateway` | Solo en entornos locales | - <*Azure-subscription*> <br>- <*Azure-on-premises-data-gateway-resource*> | Suscripción de Azure y nombre de recurso de Azure para la puerta de enlace de datos local que creó en Azure Portal. La propiedad **Puerta de enlace** y las subpropiedades aparecen solo al seleccionar **Conectarse mediante una puerta de enlace de datos local**. |
   ||||||

   Por ejemplo:

   * **Base de datos en la nube**

     ![Información de conexión de base de datos en la nube](./media/connectors-create-api-informix/informix-cloud-connection.png)

   * **Base de datos local**

     ![Información de conexión de base de datos local](./media/connectors-create-api-informix/informix-on-premises-connection.png)

1. Guarde la aplicación lógica.

<a name="test-logic-app"></a>

## <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. En la barra de herramientas del Diseñador de aplicaciones lógicas, elija **Ejecutar**. Una vez que se ejecute la aplicación lógica, podrá ver las salidas de esa ejecución.

1. En el menú de la aplicación lógica, seleccione **Introducción**. En el panel de información general, en **Resumen** > **Historial de ejecuciones**, seleccione la ejecución más reciente.

1. En **Ejecución de aplicación lógica**, seleccione **Detalles de ejecución**.

1. En la lista de acciones, seleccione la acción con las salidas que desea ver, por ejemplo, **Get_tables**.

   Si la acción se realizó correctamente, su propiedad **Estado** se marca como **Correcto**.

1. Para ver las entradas, en **Vínculo de entradas**, seleccione el vínculo de dirección URL. Para ver las salidas, en **Vínculo de salidas**, seleccione el vínculo de dirección URL. A continuación, se incluyen algunas salidas de ejemplo:

   * **Get_tables** muestra una lista de tablas:

     ![Salidas de la acción "Obtener tablas"](./media/connectors-create-api-informix/InformixconnectorGetTablesLogicAppRunOutputs.png)

   * **Get_rows** muestra una lista de filas:

     ![salidas de la acción "Obtener filas"](./media/connectors-create-api-informix/InformixconnectorGetRowsOutputs.png)

   * **Get_row** muestra la fila especificada:

     ![salidas de la acción "Obtener fila"](./media/connectors-create-api-informix/InformixconnectorGetRowOutputs.png)

   * **Insert_row** muestra la nueva fila:

     ![salidas de la acción "Insertar fila"](./media/connectors-create-api-informix/InformixconnectorInsertRowOutputs.png)

   * **Update_row** muestra la fila actualizada:

     ![salidas de la acción "Actualizar fila"](./media/connectors-create-api-informix/InformixconnectorUpdateRowOutputs.png)

   * **Delete_row** muestra la fila eliminada:

     ![salidas de la acción "Eliminar fila"](./media/connectors-create-api-informix/InformixconnectorDeleteRowOutputs.png)

## <a name="connector-specific-details"></a>Detalles específicos del conector

Para obtener información técnica acerca de los desencadenadores, las acciones y los límites, que se detallan en la descripción de Swagger del conector, consulte la [página de referencia del conector](/connectors/informix/).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](apis-list.md)
