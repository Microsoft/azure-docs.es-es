---
title: Conexión a SQL Server, Azure SQL Database o Azure SQL Managed Instance
description: Automatización de tareas en bases de datos SQL locales o en la nube mediante Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, jonfan, logicappspm
ms.topic: conceptual
ms.date: 06/06/2020
tags: connectors
ms.openlocfilehash: e500f678d2066d24de12a04f28ccbdb3f76eda3a
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288186"
---
# <a name="automate-workflows-for-a-sql-database-by-using-azure-logic-apps"></a>Automatización de flujos de trabajo en una base de datos SQL mediante Azure Logic Apps

En este artículo se muestra cómo puede acceder a los datos de la base de datos SQL desde una aplicación lógica con el conector de SQL Server. De este modo, puede automatizar las tareas, los procesos o los flujos de trabajo que administran los datos y los recursos de SQL mediante la creación de aplicaciones lógicas. El conector de SQL Server funciona en [SQL Server](/sql/sql-server/sql-server-technical-documentation), así como [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md) y la [instancia administrada de Azure SQL](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md).

Puede crear aplicaciones lógicas que se ejecutan cuando las desencadenan eventos en la base de datos SQL o en otros sistemas,como Dynamics CRM Online. Las aplicaciones lógicas también pueden obtener, insertar o eliminar datos y, además, pueden ejecutar consultas SQL o procedimientos almacenados. Por ejemplo, puede compilar una aplicación lógica que comprueba automáticamente si hay registros nuevos en Dynamics CRM Online, agrega elementos a la base de datos SQL para cualquier registro nuevo y envía alertas de correo electrónicos sobre los elementos agregados.

Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) e [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para información técnica específica del conector, así como sobre las limitaciones y los problemas conocidos, consulte la [página de referencia sobre el conector de SQL Server](/connectors/sql/).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* Una [base de datos SQL Server](/sql/relational-databases/databases/create-a-database), [Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md) o [Azure SQL Managed Instance](../azure-sql/managed-instance/instance-create-quickstart.md).

  Las tablas deben tener datos para que la aplicación lógica pueda devolver los resultados cuando se llame a las operaciones. Si usa Azure SQL Database, puede usar las bases de datos de ejemplo, que están incluidas.

* El nombre del servidor SQL, el nombre de la base de datos, el nombre de usuario y la contraseña. Necesita estas credenciales para poder autorizar que la lógica acceda al servidor SQL.

  * Para SQL Server en el entorno local, puede encontrar estos detalles en la cadena de conexión:

    `Server={your-server-address};Database={your-database-name};User Id={your-user-name};Password={your-password};`

  * Para Azure SQL Database, puede encontrar estos detalles en la cadena de conexión.
  
    Por ejemplo, para buscar esta cadena en Azure Portal, abra la base de datos. En el menú de base de datos, seleccione **Cadenas de conexión** o **Propiedades**:

    `Server=tcp:{your-server-name}.database.windows.net,1433;Initial Catalog={your-database-name};Persist Security Info=False;User ID={your-user-name};Password={your-password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;`

<a name="multi-tenant-or-ise"></a>

* En función de si las aplicaciones lógicas se van a ejecutar en Azure global, multiinquilino o en un [entorno de servicios de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), estos son requisitos adicionales para conectarse a SQL Server en el entorno local:

  * En el caso de las aplicaciones lógicas en Azure global y multiinquilino que se conectan a SQL Server en el entorno local, debe tener la [puerta de enlace de datos local](../logic-apps/logic-apps-gateway-install.md) instalada en un equipo local y un [recurso de puerta de enlace de datos que ya se haya creado en Azure](../logic-apps/logic-apps-gateway-connection.md).

  * En el caso de las aplicaciones lógicas en un ISE que se conecten a SQL Server en el entorno local y usen la autenticación de Windows, el conector de SQL Server con versión ISE no es compatible con la autenticación de Windows. Por lo tanto, sigue siendo necesario usar la puerta de enlace de datos y el conector de SQL Server que no es ISE. Para el resto de tipos de autenticación, no es necesario usar la puerta de enlace de datos y puede usar el conector de versión ISE.

* La aplicación lógica en la que necesita acceso a la base de datos SQL. Para iniciar la aplicación lógica con un desencadenador de SQL, necesita una [aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="create-connection"></a>

## <a name="connect-to-your-database"></a>Conectarse a la base de datos

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

Ahora, continúe con estos pasos:

* [Conexión a Azure SQL Database o Azure SQL Managed Instance basados en la nube](#connect-azure-sql-db)
* [Conexión a un servidor SQL Server local](#connect-sql-server)

<a name="connect-azure-sql-db"></a>

### <a name="connect-to-azure-sql-database-or-managed-instance"></a>Conexión a Azure SQL Database o Azure SQL Managed Instance

La primera vez que agregue un [desencadenador SQL](#add-sql-trigger) o una [acción de SQL](#add-sql-action) y no haya creado previamente una conexión a la base de datos, se le pedirá que complete estos pasos:

1. En **Tipo de autenticación**, seleccione la autenticación necesaria y habilitada en la base de datos en Azure SQL Database o en Azure SQL Managed Instance:

   | Authentication | Descripción |
   |----------------|-------------|
   | [**Azure AD integrado**](../azure-sql/database/authentication-aad-overview.md) | - Admite el conector de SQL Server tanto si es ISE como si no. <p><p>- Requiere una identidad válida en Azure Active Directory (Azure AD) que tenga acceso a la base de datos. <p>Para más información, consulte los temas siguientes: <p>- [Información general sobre la seguridad de Azure SQL: autenticación](../azure-sql/database/security-overview.md#authentication) <br>- [Autorización del acceso de bases de datos a Azure SQL: autenticación y autorización](../azure-sql/database/logins-create-manage.md#authentication-and-authorization) <br>- [Azure SQL: Autenticación integrada de Azure AD](../azure-sql/database/authentication-aad-overview.md) |
   | [**Autenticación de SQL Server**](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-sql-server-authentication) | - Admite el conector de SQL Server tanto si es ISE como si no. <p><p>- Requiere un nombre de usuario y una contraseña segura válidos que se creen y almacenen en la base de datos. <p>Para más información, consulte los temas siguientes: <p>- [Información general sobre la seguridad de Azure SQL: autenticación](../azure-sql/database/security-overview.md#authentication) <br>- [Autorización del acceso de bases de datos a Azure SQL: autenticación y autorización](../azure-sql/database/logins-create-manage.md#authentication-and-authorization) |
   |||

   Este ejemplo continúa con **Azure AD integrado**:

   ![Selección del tipo de autenticación](./media/connectors-create-api-sqlazure/select-azure-ad-authentication.png)

1. Después de seleccionar **Azure AD integrado**, seleccione **Iniciar sesión**. En función de si usa Azure SQL Database o Azure SQL Managed Instance, seleccione las credenciales de usuario para la autenticación.

1. Seleccione estos valores para la base de datos:

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre del servidor** | Sí | La dirección de SQL Server; por ejemplo, `Fabrikam-Azure-SQL.database.windows.net` |
   | **Nombre de la base de datos** | Sí | El nombre de la instancia de la base de datos SQL, por ejemplo `Fabrikam-Azure-SQL-DB` |
   | **Nombre de la tabla** | Sí | La tabla que desee usar, por ejemplo, `SalesLT.Customer` |
   ||||

   > [!TIP]
   > Puede encontrar esta información en la cadena de conexión de la base de datos. Por ejemplo, en Azure Portal, busque y abra la base de datos. En el menú de base de datos, seleccione **Cadenas de conexión** o **Propiedades** donde puede encontrar esta cadena:
   >
   > `Server=tcp:{your-server-address}.database.windows.net,1433;Initial Catalog={your-database-name};Persist Security Info=False;User ID={your-user-name};Password={your-password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;`

   En este ejemplo se muestra la apariencia que podrían tener estos valores:

   ![Creación de una conexión a una base de datos SQL](./media/connectors-create-api-sqlazure/azure-sql-database-create-connection.png)

1. Ahora, continúe con los pasos que todavía no haya completado en [Adición de un desencadenador de SQL](#add-sql-trigger) o [Incorporación de una acción de SQL](#add-sql-action).

<a name="connect-sql-server"></a>

### <a name="connect-to-on-premises-sql-server"></a>Conexión a un servidor SQL Server local

La primera vez que agregue un [desencadenador SQL](#add-sql-trigger) o una [acción de SQL](#add-sql-action) y no haya creado previamente una conexión a la base de datos, se le pedirá que complete estos pasos:

1. En el caso de las conexiones a la instancia de SQL Server en el entorno que requieran la puerta de enlace de datos local, asegúrese de haber [completado estos requisitos previos](#multi-tenant-or-ise).

   De lo contrario, su recurso de puerta de enlace de datos no aparecerá en la lista **Puerta de enlace de conexión** al crear la conexión.

1. En **Tipo de autenticación**, seleccione la autenticación necesaria y habilitada en el servidor SQL Server:

   | Authentication | Descripción |
   |----------------|-------------|
   | [**Autenticación de Windows**](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-windows-authentication) | - Solo admite el conector de SQL Server que no es ISE, lo que requiere un recurso de puerta de enlace de datos creado previamente en Azure para la conexión, con independencia de si se usa Azure multiinquilino o un ISE. <p><p>- Requiere un nombre de usuario y una contraseña de Windows válidos para confirmar su identidad con su cuenta de Windows. <p>Para más información, consulte [Autenticación de Windows](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-windows-authentication) |
   | [**Autenticación de SQL Server**](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-sql-server-authentication) | - Admite el conector de SQL Server tanto si es ISE como si no. <p><p>- Requiere un nombre de usuario y una contraseña segura válidos que se crean y se almacenan en el servidor SQL Server. <p>Para más información, consulte [Autenticación de SQL Server](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-sql-server-authentication). |
   |||

   Este ejemplo continúa con la **Autenticación de Windows**:

   ![Selección del tipo de autenticación](./media/connectors-create-api-sqlazure/select-windows-authentication.png)

1. Seleccione o proporcione los siguientes valores para la base de datos SQL:

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre del servidor SQL Server** | Sí | La dirección de SQL Server; por ejemplo, `Fabrikam-Azure-SQL.database.windows.net` |
   | **Nombre de la base de datos SQL** | Sí | El nombre de la base de datos de SQL Server, por ejemplo, `Fabrikam-Azure-SQL-DB` |
   | **Nombre de usuario** | Sí | Su nombre de usuario para el servidor SQL Server y la base de datos |
   | **Contraseña** | Sí | La contraseña para el servidor SQL Server y la base de datos |
   | **Suscripción** |  Sí, para la autenticación de Windows | La suscripción a Azure para el recurso de puerta de enlace de datos que creó anteriormente en Azure |
   | **Puerta de enlace de conexión** | Sí, para la autenticación de Windows | El nombre del recurso de puerta de enlace de datos que creó anteriormente en Azure <p><p>**Sugerencia**: Si la puerta de enlace no aparece en la lista, compruebe que la [configuró](../logic-apps/logic-apps-gateway-connection.md) correctamente. |
   |||

   > [!TIP]
   > Puede encontrar esta información en la cadena de conexión de la base de datos:
   > 
   > * `Server={your-server-address}`
   > * `Database={your-database-name}`
   > * `User ID={your-user-name}`
   > * `Password={your-password}`

   En este ejemplo se muestra la apariencia que podrían tener estos valores:

   ![Creación de una conexión a SQL Server finalizada](./media/connectors-create-api-sqlazure/sql-server-create-connection-complete.png)

1. Cuando esté listo, seleccione **Crear**.

1. Ahora, continúe con los pasos que todavía no haya completado en [Adición de un desencadenador de SQL](#add-sql-trigger) o [Incorporación de una acción de SQL](#add-sql-action).

<a name="add-sql-trigger"></a>

## <a name="add-a-sql-trigger"></a>Adición de un desencadenador de SQL

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, cree una aplicación lógica en blanco que abra el Diseñador de aplicación lógica. Este ejemplo continúa con Azure Portal.

1. En el cuadro de búsqueda del diseñador, escriba `sql server`. En la lista de desencadenadores, seleccione el desencadenador de SQL que desee. En este ejemplo se usa el desencadenador **Cuando se crea un elemento**.

   ![Selección del desencadenador "Cuando se crea un elemento"](./media/connectors-create-api-sqlazure/select-sql-server-trigger.png)

1. Si se va a conectar a la base de datos SQL por primera vez, se le pedirá que [cree la conexión a la base de datos de SQL ahora](#create-connection). Después de crear esta conexión, puede continuar con el siguiente paso.

1. En el desencadenador, especifique el intervalo y la frecuencia con que el desencadenador comprueba la tabla.

1. Para agregar el resto de propiedades disponibles para este desencadenador, abra la lista **Agregar nuevo parámetro**.

   Este desencadenador solo devuelve una fila de la tabla seleccionada. Para realizar otras tareas, continúe agregando una [acción de conector SQL](#add-sql-action) u [otra acción](../connectors/apis-list.md) que lleve a cabo la siguiente tarea que desee en el flujo de trabajo de la aplicación lógica.
   
   Por ejemplo, para ver los datos de esta fila, puede agregar otras acciones que creen un archivo que incluya los campos de la fila devuelta y, luego, que luego envíen alertas por correo electrónico. Para información sobre otras acciones disponibles para este conector, consulte la [página de referencia del conector](/connectors/sql/).

1. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

   Aunque este paso habilita y publica de forma automática la aplicación lógica en Azure, la única acción que la aplicación lógica necesita actualmente es comprobar la base de datos según el intervalo y la frecuencia especificados.

<a name="add-sql-action"></a>

## <a name="add-a-sql-action"></a>Incorporación de una acción de SQL

En este ejemplo, la aplicación lógica empieza con el [desencadenador de periodicidad](../connectors/connectors-native-recurrence.md) y llama a una acción que obtiene una fila de una base de datos SQL.

1. En [Azure Portal](https://portal.azure.com) o Visual Studio, abra la aplicación lógica en el Diseñador de aplicación lógica. Este ejemplo continúa en Azure Portal.

1. En el desencadenador o la acción donde quiera agregar la acción de SQL, seleccione **Nuevo paso**.

   ![Incorporación de una acción a la aplicación lógica](./media/connectors-create-api-sqlazure/select-new-step-logic-app.png)

   O bien, para agregar una acción entre los pasos existentes, mueva el mouse sobre la flecha de conexión. Seleccione el signo más ( **+** ) que aparece y, luego, seleccione **Agregar una acción**.

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `sql server`. En la lista de acciones, seleccione la acción SQL que quiera. En este ejemplo se usa la acción **Obtener fila**, que obtiene un único registro.

   ![Selección de la acción "Obtener fila" de SQL](./media/connectors-create-api-sqlazure/select-sql-get-row-action.png)

1. Si se va a conectar a la base de datos SQL por primera vez, se le pedirá que [cree la conexión a la base de datos de SQL ahora](#create-connection). Después de crear esta conexión, puede continuar con el siguiente paso.

1. Seleccione **Nombre de tabla**, que es `SalesLT.Customer` en este ejemplo. Escriba el **identificador de fila** para el registro que desee.

   ![Selección del nombre de la tabla y especificación del identificador de fila](./media/connectors-create-api-sqlazure/specify-table-row-id.png)

   Esta acción solo devuelve una fila de la tabla seleccionada, nada más. Por tanto, para ver los datos de esta fila, puede agregar otras acciones que creen un archivo que incluya los campos de la fila devuelta y almacenar ese archivo en una cuenta de almacenamiento en la nube. Para información sobre otras acciones disponibles para este conector, consulte la [página de referencia del conector](/connectors/sql/).

1. Cuando esté listo, seleccione **Guardar** en la barra de herramientas del diseñador.

   Este paso habilita y publica de manera automática la aplicación lógica en vivo en Azure.

## <a name="handle-bulk-data"></a>Controlar datos masivos

En ocasiones, tendrá que trabajar con conjuntos de resultados tan grandes que el conector no devuelva todos los resultados al mismo tiempo, o necesitará un mayor control sobre el tamaño y la estructura de los conjuntos de resultados. A continuación se indican algunas maneras de controlar conjuntos de resultados tan grandes:

* Para ayudarle a administrar los resultados como conjuntos más pequeños, active la *paginación*. Para obtener más información, consulte el tema sobre la [obtención de datos masivos, registros y elementos mediante la paginación](../logic-apps/logic-apps-exceed-default-page-size-with-pagination.md).

* Cree un procedimiento almacenado que organice los resultados como desee.

  Al obtener o insertar varias filas, la aplicación lógica puede iterar a través de estas filas mediante un [*bucle Until*](../logic-apps/logic-apps-control-flow-loops.md#until-loop) dentro de estos [límites](../logic-apps/logic-apps-limits-and-config.md). No obstante, si su aplicación lógica tiene que trabajar con conjuntos de registros tan grandes, por ejemplo, con miles o millones de filas, y quiere minimizar los costos de las llamadas a la base de datos.

  Para organizar los resultados de la manera deseada, puede crear un [*procedimiento almacenado*](/sql/relational-databases/stored-procedures/stored-procedures-database-engine) que se ejecute en la instancia de SQL y use la instrucción **SELECT - ORDER BY**. Esta solución le permite tener mayor control sobre el tamaño y la estructura de los resultados. La aplicación lógica llama al procedimiento almacenado mediante la acción **Ejecutar procedimiento almacenado** del conector de SQL Server.

  Para más información sobre las soluciones, consulte estos artículos:

  * [SQL Pagination for bulk data transfer with Logic Apps](https://social.technet.microsoft.com/wiki/contents/articles/40060.sql-pagination-for-bulk-data-transfer-with-logic-apps.aspx) (Paginación de SQL para la transferencia de datos masiva con Logic Apps)

  * [Cláusula SELECT - ORDER BY](/sql/t-sql/queries/select-order-by-clause-transact-sql)

### <a name="handle-dynamic-bulk-data"></a>Manipulación de datos dinámicos masivos

Cuando llama a un procedimiento almacenado con el conector de SQL Server, a veces, la salida devuelta es dinámica. En este ejemplo, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. Para ver el formato de salida, realice una serie de pruebas. Copie y guarde la salida del ejemplo.

1. En el diseñador, en la acción para llamar al procedimiento almacenado, seleccione **Nuevo paso**.

1. En **Elegir una acción**, busque la acción [**Analizar JSON**](../logic-apps/logic-apps-perform-data-operations.md#parse-json-action) y selecciónela.

1. En la acción **Análisis del archivo JSON**, seleccione **Usar una carga de ejemplo para generar el esquema**.

1. En la ventana **Enter or paste a sample JSON payload** (Especificar o pegar una carga JSON de ejemplo), proporcione una salida de ejemplo y seleccione **Listo**.

   > [!NOTE]
   > Si recibe un error que dice que Logic Apps no puede generar un esquema, compruebe que la sintaxis de la salida de ejemplo tiene el formato correcto. Si sigue sin poder generar el esquema, escríbalo manualmente en el cuadro **Esquema**.

1. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

1. Si desea hacer referencia a las propiedades de contenido de JSON, haga clic dentro de los cuadros de edición en los que desee hacer referencia a esas propiedades para que aparezca la lista de contenido dinámico. En la lista, en el encabezado [**Analizar JSON**](../logic-apps/logic-apps-perform-data-operations.md#parse-json-action), seleccione los tokens de datos para las propiedades de contenido JSON que desee.

## <a name="connector-specific-details"></a>Detalles específicos del conector

Para obtener información técnica acerca de los desencadenadores, las acciones y los límites de este conector, consulte la [página de referencia del conector](/connectors/sql/), que se genera a partir de la descripción de Swagger.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre otros [conectores para Azure Logic Apps](../connectors/apis-list.md).

