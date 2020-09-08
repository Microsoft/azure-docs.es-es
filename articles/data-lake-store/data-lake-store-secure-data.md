---
title: Protección de los datos almacenados en Azure Data Lake Storage Gen1 | Microsoft Docs
description: Aprenda a proteger los datos de Azure Data Lake Storage Gen1 mediante grupos y listas de control de acceso.
services: data-lake-store
documentationcenter: ''
author: twooley
manager: mtillman
editor: cgronlun
ms.assetid: ca35e65f-3986-4f1b-bf93-9af6066bb716
ms.service: data-lake-store
ms.devlang: na
ms.topic: how-to
ms.date: 03/26/2018
ms.author: twooley
ms.openlocfilehash: b1da644d8aca0b197e21ec03c7d0ac0b454f92a9
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926304"
---
# <a name="securing-data-stored-in-azure-data-lake-storage-gen1"></a>Protección de los datos almacenados en Azure Data Lake Storage Gen1
Para proteger los datos en Azure Data Lake Storage Gen1, se adopta un enfoque de tres pasos.  Es necesario que esté configurado tanto el control de acceso basado en rol como las listas de control de acceso para permitir un acceso total a los datos para los usuarios y los grupos de seguridad.

1. Comience creando grupos de seguridad en Azure Active Directory (AAD). Estos grupos de seguridad se usan para implementar el control de acceso basado en roles de Azure (Azure RBAC) en Azure Portal. Para más información, consulte [Azure RBAC](../role-based-access-control/role-assignments-portal.md).
2. Asigne los grupos de seguridad de AAD a la cuenta de Data Lake Storage Gen1. Esto controla el acceso a la cuenta de Data Lake Storage Gen1 desde el portal y las operaciones de administración desde el portal o las API.
3. Asigne los grupos de seguridad de AAD como listas de control de acceso (ACL) en el sistema de archivos de Data Lake Storage Gen1.
4. Además, también puede establecer un intervalo de direcciones IP para los clientes que pueden acceder a los datos de Data Lake Storage Gen1.

En este artículo se proporcionan instrucciones sobre cómo usar el Portal de Azure para realizar las tareas anteriores. Para obtener más información sobre cómo Data Lake Storage Gen1 implementa la seguridad en el nivel de cuenta y datos, consulte [Security in Azure Data Lake Storage Gen1](data-lake-store-security-overview.md)(Seguridad en Azure Data Lake Storage Gen1). Para obtener información detallada acerca de cómo se implementan las ACL en Azure Data Lake Storage Gen1, consulte el artículo [Introducción al control de acceso en Azure Data Lake Storage Gen1](data-lake-store-access-control.md).

## <a name="prerequisites"></a>Prerrequisitos
Antes de empezar este tutorial, debe contar con lo siguiente:

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* **Cuenta de Data Lake Storage Gen1**. Para instrucciones sobre cómo crear una, consulte la [introducción a Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)

## <a name="create-security-groups-in-azure-active-directory"></a>Creación de grupos de seguridad en Azure Active Directory
Para obtener instrucciones sobre cómo crear grupos de seguridad de AAD y cómo agregar usuarios al grupo, consulte [Administración de grupos de seguridad en Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

> [!NOTE] 
> Puede agregar usuarios y otros grupos a un grupo de Azure AD mediante Azure Portal. Pero para agregar una entidad de servicio a un grupo, use el [módulo PowerShell de Azure AD](../active-directory/users-groups-roles/groups-settings-v2-cmdlets.md).
> 
> ```powershell
> # Get the desired group and service principal and identify the correct object IDs
> Get-AzureADGroup -SearchString "<group name>"
> Get-AzureADServicePrincipal -SearchString "<SPI name>"
> 
> # Add the service principal to the group
> Add-AzureADGroupMember -ObjectId <Group object ID> -RefObjectId <SPI object ID>
> ```
 
## <a name="assign-users-or-security-groups-to-data-lake-storage-gen1-accounts"></a>Asignación de grupos de seguridad o usuarios a cuentas de Data Lake Storage Gen1
Cuando asigna usuarios o grupos de seguridad a las cuentas de Data Lake Storage Gen1, controla el acceso a las operaciones de administración en la cuenta mediante Azure Portal y las API de Azure Resource Manager. 

1. Abra una cuenta de Data Lake Storage Gen1. En el panel izquierdo, haga clic en **Todos los recursos** y, en la hoja que se abre, haga clic en el nombre de la cuenta a la que desea asignar un usuario o grupo de seguridad.

2. En la hoja de su cuenta de Data Lake Storage Gen1, haga clic en **Control de acceso (IAM)** . La hoja de forma predeterminada muestra a los propietarios de suscripción en el rol de propietario.
   
    ![Asignación de un grupo de seguridad a la cuenta de Azure Data Lake Storage Gen1](./media/data-lake-store-secure-data/adl.select.user.icon1.png "Asignación de un grupo de seguridad a la cuenta de Azure Data Lake Storage Gen1")

3. En la hoja **Control de acceso (IAM)** , haga clic en **Agregar** para abrir la hoja **Agregar permisos**. En la hoja **Agregar permisos**, seleccione un **Rol** para el usuario o grupo. Busque el grupo de seguridad que creó antes en Azure Active Directory y selecciónelo. Si tiene muchos grupos en los que buscar, use el cuadro de texto **Seleccionar** para filtrar según el nombre del grupo. 
   
    ![Incorporación de un rol para el usuario](./media/data-lake-store-secure-data/adl.add.user.1.png "Agregar un rol para el usuario")
   
    Los roles **Propietario** y **Colaborador** proporcionan acceso a diversas funciones de administración en la cuenta de Data Lake. En el caso de los usuarios que interactúan con datos en Data Lake pero siguen necesitando ver la información de administración de la cuenta, puede agregarlos al rol **Lector**. El ámbito de estos roles se limita a las operaciones de administración relacionadas con la cuenta de Data Lake Storage Gen1.
   
    Para las operaciones de datos, los permisos individuales del sistema de archivos definen lo que los usuarios pueden hacer. Por lo tanto, un usuario con el rol Lector solamente ve la configuración administrativa asociada a la cuenta pero potencialmente puede leer y escribir datos en función de los permisos del sistema de archivos que tengan asignados. Los permisos del sistema de archivos de Data Lake Storage Gen1 se describen en [Asignación de usuarios o grupos de seguridad como ACL al sistema de archivos de Data Lake Storage Gen1](#filepermissions).

    > [!IMPORTANT]
    > El rol **Propietario** habilita automáticamente el acceso al sistema de archivos. Los roles **Colaborador**, **Lector** y otros requieren ACL para habilitar cualquier nivel de acceso a las carpetas y los archivos.  El rol **Propietario** proporciona permisos de superusuario a archivos y carpetas que no se pueden invalidar mediante ACL. Para obtener más información sobre cómo se asigna el acceso a los datos en función de las directivas de control de acceso basado en rol, vea [RBAC para la administración de cuentas](data-lake-store-security-overview.md#rbac-for-account-management).

4. Si desea agregar un grupo o usuario que no aparece en la hoja **Agregar permisos**, puede invitarlo escribiendo su dirección de correo electrónico en el cuadro de texto **Seleccionar**; a continuación, selecciónelo en la lista.
   
    ![Incorporación de un grupo de seguridad](./media/data-lake-store-secure-data/adl.add.user.2.png "Agregar un grupo de seguridad")
   
5. Haga clic en **Save**(Guardar). Debería ver el grupo de seguridad que se agregó como se muestra a continuación.
   
    ![Grupo de seguridad agregado](./media/data-lake-store-secure-data/adl.add.user.3.png "Grupo de seguridad agregado")

6. Ahora el usuario o el grupo de seguridad tienen acceso a la cuenta de Data Lake Storage Gen1. Si desea proporcionar acceso a usuarios específicos, puede agregarlos al grupo de seguridad. De forma similar, si desea revocar el acceso de un usuario, puede quitarle del grupo de seguridad. También puede asignar varios grupos de seguridad a una cuenta. 

## <a name="assign-users-or-security-groups-as-acls-to-the-data-lake-storage-gen1-file-system"></a><a name="filepermissions"></a>Asignación de usuarios o grupos de seguridad como ACL al sistema de archivos de Data Lake Storage Gen1
Al asignar usuarios o grupos de seguridad al sistema de archivos de Data Lake Storage Gen1, establece el control de acceso sobre los datos almacenados en Data Lake Storage Gen1.

1. En la hoja de su cuenta de Data Lake Storage Gen1, haga clic en **Explorador de datos**.
   
    ![Visualización de datos en Data Explorer](./media/data-lake-store-secure-data/adl.start.data.explorer.png "Visualización de datos en Data Explorer")
2. En la hoja **Explorador de datos**, haga clic en la carpeta para las que desea configurar la ACL y, después, haga clic en **Acceso**. Para asignar las ACL a un archivo, primero debe hacer clic en el archivo para obtener una vista previa y, a continuación, haga clic en **Acceso** en la hoja **Vista previa de archivo**.
   
    ![Establecimiento de las ACL en el sistema de archivos de Data Lake Storage Gen1](./media/data-lake-store-secure-data/adl.acl.1.png "Establecimiento de las ACL en el sistema de archivos de Data Lake Storage Gen1")
3. La hoja **Acceso** muestra los propietarios y los permisos ya asignados a la raíz. Haga clic en el icono **Agregar** para agregar las ACL de acceso adicionales.
    > [!IMPORTANT]
    > El establecimiento de los permisos de acceso para un único archivo no concede necesariamente a un usuario o grupo acceso a ese archivo. La ruta de acceso al archivo debe ser accesible para el usuario o grupo asignado. Para obtener más información y ejemplos, vea [Escenarios comunes relacionados con los permisos](data-lake-store-access-control.md#common-scenarios-related-to-permissions).
   
    ![Enumeración de acceso estándar y personalizado](./media/data-lake-store-secure-data/adl.acl.2.png "Mostrar acceso estándar y personalizado")
   
   * **Propietarios** y **Todos los demás** ofrecen acceso de estilo UNIX, donde se especifican lectura, escritura y ejecución (rwx) para tres clases de usuario distintos: propietario, grupo y otros.
   * **Permisos asignados** corresponde a las ACL de POSIX que le permiten establecer permisos para usuarios o grupos designados específicos más allá del propietario o el grupo del archivo. 
     
     Para obtener más información, consulte la página sobre las [ACL de HDFS](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html#ACLs_Access_Control_Lists). Para obtener más información sobre cómo se implementan las ACL en Data Lake Storage Gen1, consulte [Control de acceso en Azure Data Lake Storage Gen1](data-lake-store-access-control.md).
4. Haga clic en el icono **Agregar** para abrir la hoja **Asignar permisos**. En esta hoja, haga clic en **Seleccionar usuario o grupo** y, después, en la hoja **Seleccionar usuario o grupo**, busque el grupo de seguridad que creó antes en Azure Active Directory. Si tiene muchos grupos en los que buscar, use el cuadro de texto en la parte superior para filtrar según el nombre del grupo. Haga clic en el grupo que desee agregar y después en **Seleccionar**.
   
    ![Incorporación de un grupo](./media/data-lake-store-secure-data/adl.acl.3.png "Agregar un grupo")
5. Haga clic en **Seleccionar permisos**, seleccione los permisos, si desea que los permisos se apliquen periódicamente y si desea asignar los permisos como una ACL de acceso, una ACL predeterminada o ambos. Haga clic en **OK**.
   
    ![Asignación de permisos al grupo](./media/data-lake-store-secure-data/adl.acl.4.png "Asignar permisos al grupo")
   
    Para obtener más información sobre los permisos de Data Lake Storage Gen1 y las ACL predeterminadas y de acceso, consulte [Control de acceso en Azure Data Lake Storage Gen1](data-lake-store-access-control.md).
6. Después de hacer clic en **Aceptar** en la hoja **Seleccionar permisos**, los permisos de grupo y asociados recién agregados aparecerán ahora en la hoja **Acceso**.
   
    ![Asignación de permisos al grupo](./media/data-lake-store-secure-data/adl.acl.5.png "Asignar permisos al grupo")
   
   > [!IMPORTANT]
   > En la versión actual, puede tener hasta 28 entradas bajo **Permisos asignados**. Si quiere agregar más de 28 usuarios, debe crear grupos de seguridad, agregar usuarios a los grupos de seguridad y proporcionar acceso a esos grupos de seguridad para la cuenta de Data Lake Storage Gen1.
   > 
   > 
7. Si es necesario, también puede modificar los permisos de acceso después de agregar el grupo. Active o desactive la casilla para cada tipo de permiso (lectura, escritura, ejecución) en función de si desea quitarlo o asignarlo al grupo de seguridad. Haga clic en **Guardar** para guardar los cambios o en **Descartar** para deshacerlos.

## <a name="set-ip-address-range-for-data-access"></a>Configuración del intervalo de direcciones IP para el acceso a los datos
Data Lake Storage Gen1 le permite bloquear aún más el acceso a su almacén de datos en el nivel de red. Puede habilitar el firewall, especificar una dirección IP o definir un intervalo de direcciones IP para los clientes de confianza. Una vez habilitado, solo los clientes que tienen las direcciones IP dentro del intervalo definido pueden conectarse al almacén.

![Configuración del firewall y el acceso a IP](./media/data-lake-store-secure-data/firewall-ip-access.png "Configuración del firewall y dirección IP")

## <a name="remove-security-groups-for-a-data-lake-storage-gen1-account"></a>Eliminación de grupos de seguridad de una cuenta de Data Lake Storage Gen1
Cuando quita grupos de seguridad de cuentas de Data Lake Storage Gen1, solamente está cambiando el acceso a las operaciones de administración en la cuenta mediante Azure Portal y las API de Azure Resource Manager.  

El acceso a datos no se ha modificado y sigue administrado por las ACL de acceso.  La excepción a esto son los usuarios o grupos en el rol Propietarios.  Los usuarios o grupos que se quiten del rol Propietarios dejarán de ser superusuarios y su acceso retrocede a la configuración de acceso de ACL. 

1. En la hoja de su cuenta de Data Lake Storage Gen1, haga clic en **Control de acceso (IAM)** . 
   
    ![Asignación de un grupo de seguridad a la cuenta de Data Lake Storage Gen1](./media/data-lake-store-secure-data/adl.select.user.icon.png "Asignación de un grupo de seguridad a la cuenta de Data Lake Storage Gen1")
2. En la hoja **Control de acceso (IAM)** , haga clic en el grupo de seguridad que desea quitar. Haga clic en **Quitar**.
   
    ![Grupo de seguridad quitado](./media/data-lake-store-secure-data/adl.remove.group.png "Grupo de seguridad quitado")

## <a name="remove-security-group-acls-from-a-data-lake-storage-gen1-file-system"></a>Eliminación de las ACL de grupos de seguridad del sistema de archivos de Data Lake Storage Gen1
Cuando quita las ACL de grupos de seguridad de un sistema de archivos de Data Lake Storage Gen1, cambia el acceso a los datos de la cuenta de Data Lake Storage Gen1.

1. En la hoja de su cuenta de Data Lake Storage Gen1, haga clic en **Explorador de datos**.
   
    ![Creación de directorios en la cuenta de Data Lake Storage Gen1](./media/data-lake-store-secure-data/adl.start.data.explorer.png "Creación de directorios en la cuenta de Data Lake Storage Gen1")
2. En la hoja **Explorador de datos**, haga clic en la carpeta para los que desea quitar la ACL y, después, haga clic en **Acceso**. Para quitar las ACL para un archivo, primero debe hacer clic en el archivo para obtener una vista previa y, a continuación, haga clic en **Acceso** en la hoja **Vista previa de archivo**. 
   
    ![Establecimiento de las ACL en el sistema de archivos de Data Lake Storage Gen1](./media/data-lake-store-secure-data/adl.acl.1.png "Establecimiento de las ACL en el sistema de archivos de Data Lake Storage Gen1")
3. En la hoja **Acceso**, haga clic en el grupo de seguridad que desea quitar. En la hoja **Detalles de acceso**, haga clic en **Quitar**.
   
    ![Asignación de permisos al grupo](./media/data-lake-store-secure-data/adl.remove.acl.png "Asignar permisos al grupo")

## <a name="see-also"></a>Consulte también
* [Introducción a Azure Data Lake Storage Gen1](data-lake-store-overview.md)
* [Copia de datos de los blobs de Azure Storage en Data Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md)
* [Use Azure Data Lake Analytics with Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md) (Uso de Azure Data Lake Analytics con Data Lake Storage Gen1)
* [Use Azure HDInsight with Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md) (Uso de Azure HDInsight con Data Lake Storage Gen1)
* [Introducción a Data Lake Storage Gen1 con PowerShell](data-lake-store-get-started-powershell.md)
* [InGet Started with Data Lake Storage Gen1 using .NET SDK](data-lake-store-get-started-net-sdk.md) (Introducción a Data Lake Storage Gen1 con el SDK de .NET)
* [Access diagnostic logs for Data Lake Storage Gen1](data-lake-store-diagnostic-logs.md) (Acceso a los registros de diagnóstico de Data Lake Storage Gen1)
