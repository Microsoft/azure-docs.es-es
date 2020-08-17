---
title: Administración del acceso a áreas de trabajo, datos y canalizaciones
description: Aprenda a administrar el control de acceso a áreas de trabajo, datos y canalizaciones en un área de trabajo de Azure Synapse Analytics (versión preliminar).
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: ''
ms.date: 04/15/2020
ms.author: v-stazar
ms.reviewer: jrasnick
ms.openlocfilehash: 6dc0bb08e3d824c1d3fea155d1b3d4c52e1af4a6
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499847"
---
# <a name="manage-access-to-workspaces-data-and-pipelines"></a>Administración del acceso a áreas de trabajo, datos y canalizaciones

Aprenda a administrar el control de acceso a áreas de trabajo, datos y canalizaciones en un área de trabajo de Azure Synapse Analytics (versión preliminar).

> [!NOTE]
> En el caso de la disponibilidad general, RBAC se desarrollará más a través de la introducción de roles de Azure específicos de Synapse

## <a name="access-control-for-workspace"></a>Control de acceso para un área de trabajo

Para una implementación de producción en un área de trabajo de Azure Synapse, se recomienda organizar el entorno de forma que sea fácil aprovisionar usuarios y administradores.

> [!NOTE]
> El enfoque que se usa aquí es crear varios grupos de seguridad y, después, configurar el área de trabajo para que los utilice de forma coherente. Una vez configurados los grupos, los administradores solo necesitan administrar la pertenencia a los grupos de seguridad.

### <a name="step-1-set-up-security-groups-with-names-following-this-pattern"></a>Paso 1: configuración de grupos de seguridad con nombres que siguen este patrón

1. Cree un grupo de seguridad llamado `Synapse_WORKSPACENAME_Users`
2. Cree un grupo de seguridad llamado `Synapse_WORKSPACENAME_Admins`
3. Se ha agregado `Synapse_WORKSPACENAME_Admins` a `Synapse_WORKSPACENAME_Users`

> [!NOTE]
> Obtenga información sobre cómo crear un grupo de seguridad en [este artículo](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal).
>
> Obtenga información sobre cómo agregar un grupo de seguridad de otro grupo de seguridad en [este artículo](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-membership-azure-portal).
>
> WORKSPACENAME: debe reemplazar esta parte por el nombre de área de trabajo real.

### <a name="step-2-prepare-the-default-adls-gen2-account"></a>Paso 2: preparación de la cuenta de ADLS Gen2 predeterminada

Al aprovisionar el área de trabajo, tenía que elegir una cuenta de [Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-introduction) y un contenedor para el sistema de archivos del área de trabajo que se va a usar.

1. Abra [Azure Portal](https://portal.azure.com).
2. Desplazamiento a la cuenta de Azure Data Lake Storage Gen2
3. Vaya al contenedor (filesystem) que eligió para el área de trabajo de Azure Synapse
4. Haga clic en **Control de acceso (IAM)**
5. Asigne los siguientes roles:
   1. Rol de **lector** para: `Synapse_WORKSPACENAME_Users`
   2. Rol de **propietario de datos de Storage Blob** para: `Synapse_WORKSPACENAME_Admins`
   3. Rol de **colaborador de datos de Storage Blob** para: `Synapse_WORKSPACENAME_Users`
   4. Rol de **propietario de datos de Storage Blob** para: `WORKSPACENAME`

> [!NOTE]
> WORKSPACENAME: debe reemplazar esta parte por el nombre de área de trabajo real.

### <a name="step-3-configure-the-workspace-admin-list"></a>Paso 3: configuración de la lista de administradores del área de trabajo

1. Vaya a la [**interfaz de usuario web de Azure Synapse**](https://web.azuresynapse.net)
2. Vaya a **Manage**  > **Security** > **Access control** (Administrar > Seguridad > Control de acceso)
3. Haga clic en **Add Admin** (Agregar administrador) y seleccione `Synapse_WORKSPACENAME_Admins`

### <a name="step-4-configure-sql-admin-access-for-the-workspace"></a>Paso 4: configuración del acceso de administrador de SQL para el área de trabajo

1. Vaya a [Azure Portal](https://portal.azure.com).
2. Vaya a su área de trabajo
3. Vaya a **Configurar** > **Administrador de Active Directory**
4. Haga clic en **Establecer administrador**
5. Seleccione `Synapse_WORKSPACENAME_Admins`.
6. Haga clic en **Seleccionar**
7. Haga clic en **Guardar**

> [!NOTE]
> WORKSPACENAME: debe reemplazar esta parte por el nombre de área de trabajo real.

### <a name="step-5-add-and-remove-users-and-admins-to-security-groups"></a>Paso 5: incorporación y eliminación de usuarios y administradores en grupos de seguridad

1. Agregue los usuarios que necesitan acceso administrativo a `Synapse_WORKSPACENAME_Admins`
2. Agregue todos los demás usuarios a `Synapse_WORKSPACENAME_Users`

> [!NOTE]
> Obtenga información sobre cómo agregar usuarios como miembros a un grupo de seguridad en [este artículo](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-members-azure-portal).
> 
> WORKSPACENAME: debe reemplazar esta parte por el nombre de área de trabajo real.

## <a name="access-control-to-data"></a>Control de acceso a los datos

El control de acceso a los datos subyacentes se divide en tres partes:

- Acceso del plano de datos a la cuenta de almacenamiento (ya configurado en el paso 2)
- Acceso del plano de datos a las bases de datos SQL (tanto para grupos de SQL como para SQL a petición)
- Creación de una credencial para bases de datos de SQL a petición a través de la cuenta de almacenamiento

## <a name="access-control-to-sql-databases"></a>Control de acceso a bases de datos SQL

> [!TIP]
> Los pasos siguientes deben ejecutarse para **cada** base de datos SQL con el fin de conceder a los usuarios acceso a todas las bases de datos SQL, excepto en la sección [Permisos de nivel de servidor](#server-level-permission), donde puede asignar un rol sysadmin a un usuario.

### <a name="sql-on-demand"></a>SQL a petición

En esta sección, puede encontrar ejemplos sobre cómo conceder a un usuario un permiso para una base de datos determinada o permisos de servidor completos.

#### <a name="database-level-permission"></a>Permiso de nivel de base de datos

Para conceder acceso a un usuario a una base de datos **única** de SQL a petición, siga los pasos de este ejemplo:

1. Cree LOGIN

    ```sql
    use master
    go
    CREATE LOGIN [alias@domain.com] FROM EXTERNAL PROVIDER;
    go
    ```

2. Cree USER

    ```sql
    use yourdb -- Use your DB name
    go
    CREATE USER alias FROM LOGIN [alias@domain.com];
    ```

3. Agregue USER a los miembros del rol especificado

    ```sql
    use yourdb -- Use your DB name
    go
    alter role db_owner Add member alias -- Type USER name from step 2
    ```

> [!NOTE]
> Reemplace alias por el alias del usuario al que le gustaría conceder acceso y dominio con el dominio de la compañía que está usando.

#### <a name="server-level-permission"></a>Permiso de nivel de servidor

Para conceder acceso total a un usuario a **todas** las bases de datos de SQL On-Demand, siga el paso de este ejemplo:

```sql
CREATE LOGIN [alias@domain.com] FROM EXTERNAL PROVIDER;
ALTER SERVER ROLE  sysadmin  ADD MEMBER [alias@domain.com];
```

### <a name="sql-pools"></a>Grupos de SQL

Para conceder acceso a un usuario a una base de datos **única** de SQL, siga estos pasos:

1. Cree el usuario en la base de datos. Para ello, debe ejecutar el siguiente comando, que va dirigido a la base de datos deseada del selector de contexto (lista desplegable para seleccionar bases de datos):

    ```sql
    --Create user in SQL DB
    CREATE USER [<alias@domain.com>] FROM EXTERNAL PROVIDER;
    ```

2. Conceda al usuario un rol para acceder a la base de datos:

    ```sql
    --Create user in SQL DB
    EXEC sp_addrolemember 'db_owner', '<alias@domain.com>';
    ```

> [!IMPORTANT]
> *db_datareader* y *db_datawriter* pueden funcionar para los permisos de lectura y escritura si no se desea conceder el permiso *db_owner*.
> Para que los usuarios de Spark lean directamente desde Spark de un grupo de SQL, o escriban en él, se requiere el permiso *db_owner*.

Después de crear los usuarios, compruebe que SQL a petición puede consultar la cuenta de almacenamiento.

## <a name="access-control-to-workspace-pipeline-runs"></a>Control de acceso a las ejecuciones de la canalización del área de trabajo

### <a name="workspace-managed-identity"></a>Identidad administrada del área de trabajo

> [!IMPORTANT]
> Para ejecutar correctamente canalizaciones que incluyen conjuntos de datos o actividades que hacen referencia a un grupo de SQL, es preciso que se conceda a la identidad del área de trabajo acceso al grupo de SQL directamente.

Ejecute los siguientes comandos en cada grupo de SQL para permitir que la identidad administrada del área de trabajo ejecute canalizaciones en la base de datos del grupo de SQL:

```sql
--Create user in DB
CREATE USER [<workspacename>] FROM EXTERNAL PROVIDER;

--Granting permission to the identity
GRANT CONTROL ON DATABASE::<SQLpoolname> TO <workspacename>;
```

Este permiso se puede quitar. Para ello, es preciso ejecutar el siguiente script en el mismo grupo de SQL:

```sql
--Revoking permission to the identity
REVOKE CONTROL ON DATABASE::<SQLpoolname> TO <workspacename>;

--Deleting the user in the DB
DROP USER [<workspacename>];
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener información general sobre la identidad administrada del área de trabajo de Synapse, consulte [Identidad administrada del área de trabajo de Azure Synapse (versión preliminar)](../security/synapse-workspace-managed-identity.md). Para más información sobre las entidades de seguridad de bases de datos, consulte [Entidades de seguridad](https://msdn.microsoft.com/library/ms181127.aspx). Puede encontrar información adicional acerca de los roles de base de datos en el artículo [Roles de base de datos](https://msdn.microsoft.com/library/ms189121.aspx).
