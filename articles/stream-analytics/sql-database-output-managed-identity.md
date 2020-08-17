---
title: 'Uso de identidades administradas para acceder a Azure SQL Database: Azure Stream Analytics'
description: En este artículo se describe cómo usar identidades administradas para autenticar su trabajo de Azure Stream Analytics en la salida de Azure SQL Database.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 05/08/2020
ms.openlocfilehash: fff44617a5621d4edd84bee8f9b1e6b6e6924cd8
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87305931"
---
# <a name="use-managed-identities-to-access-azure-sql-database-from-an-azure-stream-analytics-job-preview"></a>Uso de identidades administradas para acceder a Azure SQL Database desde un trabajo de Azure Stream Analytics (versión preliminar)

Azure Stream Analytics admite la [autenticación de identidades administradas](../active-directory/managed-identities-azure-resources/overview.md) para los receptores de salida de Azure SQL Database. Las identidades administradas eliminan las limitaciones de los métodos de autenticación basada en el usuario, como la necesidad de volver a realizar la autenticación debido a los cambios de contraseña o la expiración de tokens de usuario que se produce cada 90 días. Cuando se elimina la necesidad de autenticarse manualmente, las implementaciones de Stream Analytics se pueden automatizar completamente.

Una identidad administrada es una aplicación administrada registrada en Azure Active Directory que representa un trabajo de Stream Analytics determinado. La aplicación administrada se usa para la autenticación en un recurso de destino. En este artículo se muestra cómo habilitar la identidad administrada para las salidas de Azure SQL Database de un trabajo de Stream Analytics a través de Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

Para usar esta característica se requiere lo siguiente:

- Un trabajo de Azure Stream Analytics.

- Un recurso de Azure SQL Database.

## <a name="create-a-managed-identity"></a>Creación de una entidad administrada

En primer lugar, debe crear una identidad administrada para el trabajo de Azure Stream Analytics.

1. En [Azure Portal](https://portal.azure.com), abra el trabajo de Azure Stream Analytics.

1. En el menú de navegación izquierdo, seleccione **Identidad administrada** en **Configurar**. A continuación, active la casilla situada junto a **Usar la identidad administrada asignada por el sistema** y seleccione **Guardar**.

   ![Selección de la identidad administrada asignada por el sistema](./media/sql-db-output-managed-identity/system-assigned-managed-identity.png)


   Se crea una entidad de servicio para la identidad del trabajo de Stream Analytics en Azure Active Directory. El ciclo de vida de la identidad recién creada lo administrará Azure. Cuando se elimina el trabajo de Stream Analytics, Azure elimina automáticamente la identidad asociada (es decir, la entidad de servicio). 

1. Cuando se guarda la configuración, el id. de objeto (OID) de la entidad de servicio aparece como id. de entidad de seguridad, tal como se muestra a continuación: 

   ![Id. de objeto mostrado como id. de entidad de seguridad](./media/sql-db-output-managed-identity/principal-id.png)

   La entidad de servicio se llama igual que el trabajo de Stream Analytics. Por ejemplo, si es el nombre del trabajo es *MyASAJob*, el nombre de la entidad de servicio también será *MyASAJob*.

## <a name="select-an-active-directory-admin"></a>Selección de un administrador de Active Directory

Después de crear una identidad administrada, seleccione un administrador de Active Directory.

1. Navegue hasta el recurso de Azure SQL Database y seleccione la instancia de SQL Server en la que se encuentra la base de datos. Puede encontrar el nombre de SQL Server junto a *Nombre del servidor* en la página de información general de recursos. 

1. Seleccione **Administrador de Active Directory** en **Configuración**. A continuación, seleccione **Establecer administrador**. 

   ![Página Administrador de Active Directory](./media/sql-db-output-managed-identity/active-directory-admin-page.png)
 
1. En la página Administrador de Active Directory, busque un usuario o un grupo para que sea administrador de SQL Server y haga clic en **Seleccionar**.

   ![Adición de un administrador de Active Directory](./media/sql-db-output-managed-identity/add-admin.png)

   En la hoja Administrador de Active Directory se muestran todos los miembros y grupos de Active Directory. Los usuarios o grupos que aparecen atenuados no se pueden seleccionar porque no se admiten como administradores de Azure AD. Vea la lista de administradores admitidos en la sección  **Características y limitaciones de Azure AD** de  [Uso de la autenticación de Azure Active Directory con SQL Database o Azure Synapse](../sql-database/sql-database-aad-authentication.md#azure-ad-features-and-limitations). El control de acceso basado en rol (RBAC) se aplica solo al portal y no se propaga a SQL Server. Además, el usuario o grupo seleccionado es el usuario que podrá crear el **usuario de base de datos independiente** en la sección siguiente.

1. Seleccione **Guardar** en la página **Administrador de Active Directory**. El proceso para cambiar el administrador tarda unos minutos.

   Al configurar el administrador de Azure AD, el nuevo nombre del administrador (usuario o grupo) no puede estar en la base de datos maestra virtual como usuario de autenticación de SQL Server. Si estuviera presente, se producirá un error en el programa de instalación del administrador de Azure AD y revertirá su creación, para indicar que ese administrador (nombre) ya existe. Como el usuario de autenticación de SQL Server no forma parte de Azure AD, se producirá un error cada vez que intente conectarse al servidor mediante la autenticación de Azure AD como ese usuario. 

## <a name="create-a-contained-database-user"></a>Creación de un usuario de base de datos independiente

A continuación, cree un usuario de base de datos independiente en su instancia de SQL Database que esté asignado a la identidad de Azure Active Directory. El usuario de base de datos independiente no tiene un inicio de sesión para la base de datos maestra, sino que se asigna a una identidad del directorio que está asociada a la base de datos. La identidad de Azure Active Directory puede ser una cuenta de usuario individual o un grupo. En este caso, puede crear un usuario de base de datos independiente para el trabajo de Stream Analytics. 

1. Conéctese a SQL Database mediante SQL Server Management Studio. El **nombre de usuario** es un usuario de Azure Active Directory con el permiso **ALTER ANY USER**. El administrador que estableció en SQL Server es un ejemplo. Utilice la opción de autenticación **Azure Active Directory - Universal con MFA** 

   ![Conectar a SQL Server](./media/sql-db-output-managed-identity/connect-sql-server.png)

   El nombre del servidor `<SQL Server name>.database.windows.net` puede ser diferente en distintas regiones. Por ejemplo, la región de China debe utilizar `<SQL Server name>.database.chinacloudapi.cn`.
 
   Para especificar una instancia de SQL Database específica, vaya a **Opciones > Propiedades de conexión > Conectar con base de datos**.  

   ![Propiedades de conexión de SQL Server:](./media/sql-db-output-managed-identity/sql-server-connection-properties.png)

1. Cuando se conecta por primera vez, puede que se muestre la siguiente ventana:

   ![Ventana Nueva regla de firewall](./media/sql-db-output-managed-identity/new-firewall-rule.png)

   1. En ese caso, vaya al recurso de SQL Server en Azure Portal. En la sección **Seguridad**, abra la página **Firewalls y redes virtuales**. 
   1. Agregue una nueva regla con cualquier nombre.
   1. Use la dirección IP *De* de la ventana **Nueva regla de firewall** para *IP de inicio*.
   1. Use la dirección IP de *A* de la ventana **Nueva regla de firewall** para *IP final*. 
   1. Seleccione **Guardar** e intente conectarse de nuevo desde SQL Server Management Studio. 

1. Una vez que esté conectado, cree el usuario de base de datos independiente. El siguiente comando de SQL crea un usuario de base de datos independiente que tiene el mismo nombre que el trabajo de Stream Analytics. Asegúrese de incluir los corchetes alrededor de *ASA_JOB_NAME*. Use la siguiente sintaxis de T-SQL y ejecute la consulta. 

   ```sql
   CREATE USER [ASA_JOB_NAME] FROM EXTERNAL PROVIDER; 
   ```

1. Para que Azure Active Directory de Microsoft compruebe si el trabajo de Stream Analytics tiene acceso a la instancia de SQL Database, es necesario concederle permiso para comunicarse con la base de datos. Para ello, vaya de nuevo a la página "Firewalls y red virtual" de Azure Portal y habilite "Permitir que los servicios y recursos de Azure accedan a este servidor". 

   ![Firewall y red virtual](./media/sql-db-output-managed-identity/allow-access.png)

## <a name="grant-stream-analytics-job-permissions"></a>Concesión de permisos del trabajo de Stream Analytics

Una vez que se haya creado un usuario de base de datos independiente y se le haya proporcionado acceso a los servicios de Azure en el portal como se ha descrito en la sección anterior, el trabajo de Stream Analytics tiene permiso de identidad administrada para **CONECTARSE** al recurso de base de datos de SQL a través de la identidad administrada. Se recomienda conceder los permisos SELECT e INSERT al trabajo de Stream Analytics, ya que se necesitarán más adelante en el flujo de trabajo de Stream Analytics. El permiso **SELECT** permite que el trabajo pruebe su conexión con la tabla de la instancia de SQL Database. El permiso **INSERT** permite probar consultas Stream Analytics de un extremo a otro una vez que haya configurado una entrada y la salida de SQL Database. Puede conceder estos permisos al trabajo de Stream Analytics mediante SQL Server Management Studio. Para obtener más información, consulte la referencia de GRANT (Transact-SQL).

Para conceder solo permiso a una tabla o un objeto determinado de la base de datos, use la siguiente sintaxis de T-SQL y ejecute la consulta. 

```sql
GRANT SELECT, INSERT ON OBJECT::TABLE_NAME TO ASA_JOB_NAME; 
```

Como alternativa, puede hacer clic con el botón derecho en su instancia de SQL Database en SQL Server Management Studio y seleccionar **Propiedades > Permisos**. En el menú de permisos, puede ver el trabajo de Stream Analytics que agregó anteriormente y conceder o denegar los permisos manualmente según considere oportuno.

## <a name="create-an-azure-sql-database-output"></a>Creación de una salida de Azure SQL Database

Ahora que la identidad administrada está configurada, está listo para agregar Azure SQL Database como salida a su trabajo de Stream Analytics.

Asegúrese de haber creado una tabla en la instancia de SQL Database con el esquema de salida adecuado. El nombre de esta tabla es una de las propiedades necesarias que se debe rellenar cuando se agrega la salida de SQL Database al trabajo de Stream Analytics. Además, asegúrese de que el trabajo tiene permisos **SELECT** e **INSERT** para probar la conexión y ejecutar consultas de Stream Analytics. Consulte la sección [Concesión de permisos de trabajos de Stream Analytics](#grant-stream-analytics-job-permissions) si todavía no lo ha hecho. 

1. Vuelva al trabajo de Stream Analytics y vaya a la página **Salidas** en **Topología de trabajo**. 

1. Seleccione **Agregar > SQL Database**. En la ventana de propiedades de salida del receptor de salida de SQL Database, seleccione **Identidad administrada** en la lista desplegable Modo de autenticación.

1. Rellene el resto de las propiedades. Para más información acerca de cómo crear una salida de SQL Database, consulte [Creación de una salida de SQL Database con Stream Analytics](stream-analytics-define-outputs.md#sql-database). Cuando haya terminado, seleccione **Guardar**. 

## <a name="next-steps"></a>Pasos siguientes

* [Información sobre las salidas desde Azure Stream Analytics](stream-analytics-define-outputs.md)
* [Salida de Azure Stream Analytics a Azure SQL Database](stream-analytics-sql-output-perf.md)
