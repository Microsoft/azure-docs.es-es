---
title: 'Uso de Azure Active Directory en Azure Database for PostgreSQL: servidor único'
description: 'Aprenda a configurar Azure Active Directory (AAD) para la autenticación con Azure Database for PostgreSQL: servidor único.'
author: lfittl
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.date: 07/23/2020
ms.openlocfilehash: 178c339f6f47569160a9a748794678c610f35734
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87171634"
---
# <a name="use-azure-active-directory-for-authentication-with-postgresql"></a>Uso de Azure Active Directory para la autenticación con PostgreSQL

Este artículo le guiará a través de los pasos para configurar el acceso a Azure Active Directory con Azure Database for PostgreSQL, y cómo conectarse con un token de Azure AD.

## <a name="setting-the-azure-ad-admin-user"></a>Configuración del usuario administrador de Azure AD

Solo los usuarios administradores de Azure AD pueden crear o habilitar usuarios para la autenticación basada en Azure AD. Se recomienda no usar el administrador de Azure AD para las operaciones normales de base de datos, ya que tiene permisos de usuario elevados (por ejemplo, CREATEDB).

Para configurar el administrador de Azure AD (puede usar un usuario o un grupo), siga estos pasos

1. En el Azure Portal, seleccione la instancia de Azure Database for PostgreSQL que desea habilitar para Azure AD.
2. En Configuración, seleccione Administrador de Active Directory:

![establecer administrador de Azure AD][2]

3. Seleccione un usuario de Azure AD válido en el suscriptor del cliente para que sea administrador de Azure AD.

> [!IMPORTANT]
> Al establecer el administrador, se agrega un nuevo usuario al servidor de Azure Database for PostgreSQL con permisos de administrador totales. El usuario administrador de Azure AD en Azure Database for PostgreSQL tendrá el rol `azure_ad_admin`.

Solo se puede crear un administrador de Azure AD por servidor PostgreSQL y la selección de otro sobrescribirá al administrador de Azure AD existente configurado para el servidor. Puede especificar un grupo de Azure AD en lugar de un usuario individual para que tenga varios administradores. Tenga en cuenta que, después iniciará sesión con el nombre de grupo para fines de administración.

## <a name="connecting-to-azure-database-for-postgresql-using-azure-ad"></a>Conexión a Azure Database for PostgreSQL con Azure AD

En el siguiente diagrama de alto nivel se resume el flujo de trabajo del uso de la autenticación de Azure AD con Azure Database for PostgreSQL:

![flujo de autenticación][1]

Hemos diseñado la integración de Azure AD para trabajar con herramientas comunes de PostgreSQL, como psql, que no son compatibles con Azure AD y solo admiten la especificación de nombre de usuario y contraseña al conectarse a PostgreSQL. Pasamos el token de Azure AD como la contraseña tal como se muestra en la imagen anterior.

Actualmente hemos probado a los siguientes clientes:

- psql commandline (use la variable PGPASSWORD para pasar el token, ver a continuación)
- Azure Data Studio (con la extensión PostgreSQL)
- Otros clientes se basan en libpq (por ejemplo, marcos de trabajo de la aplicación comunes y ORM)

> [!NOTE]
> Tenga en cuenta que actualmente no se admite el uso del token de Azure AD con pgAdmin, ya que tiene una limitación codificada de forma rígida de 256 caracteres para las contraseñas (que supera el token).

Los pasos que se describen a continuación son los que debe seguir un usuario o una aplicación para autenticarse con Azure AD:

### <a name="prerequisites"></a>Requisitos previos

Puede seguir en Azure Cloud Shell, en una máquina virtual de Azure o en su equipo local. Asegúrese de que tiene instalado el [CLI de Azure](/cli/azure/install-azure-cli).

### <a name="step-1-authenticate-with-azure-ad"></a>Paso 1: Autenticación mediante Azure Active Directory

Empiece autenticándose en Azure AD por medio de la herramienta CLI de Azure. Este paso no es necesario en Azure Cloud Shell.

```
az login
```

El comando iniciará una ventana del explorador en la página de autenticación de Azure AD. Requiere que proporcione el identificador de usuario y la contraseña de Azure AD.

### <a name="step-2-retrieve-azure-ad-access-token"></a>Paso 2: Recuperar el token de acceso a Azure AD

Invoque la herramienta Azure CLI-Interfaz de la línea de comandos de Azure para adquirir un token de acceso para el usuario autenticado de Azure AD en el paso 1 para tener acceso a Azure Database for PostgreSQL.

Ejemplo (para la nube pública):

```azurecli-interactive
az account get-access-token --resource https://ossrdbms-aad.database.windows.net
```

El valor del recurso anterior debe especificarse exactamente como se muestra. En el caso de otras nubes, el valor del recurso se puede buscar usando:

```azurecli-interactive
az cloud show
```

En la versión 2.0.71 de CLI de Azure y posteriores, el comando se puede especificar en la siguiente versión más conveniente para todas las nubes:

```azurecli-interactive
az account get-access-token --resource-type oss-rdbms
```

Una vez que la autenticación se realiza correctamente, Azure AD devolverá un token de acceso:

```json
{
  "accessToken": "TOKEN",
  "expiresOn": "...",
  "subscription": "...",
  "tenant": "...",
  "tokenType": "Bearer"
}
```

El token es una cadena de Base 64 que codifica toda la información sobre el usuario autenticado, y que está dirigida al servicio de Azure Database for PostgreSQL.

> [!NOTE]
> La validez del token de acceso es de entre 5 y 60 minutos. Se recomienda obtener el token de acceso justo antes del inicio de sesión en Azure Database for PostgreSQL.

### <a name="step-3-use-token-as-password-for-logging-in-with-postgresql"></a>Paso 3: Uso del token como contraseña para iniciar sesión con PostgreSQL

Al conectarse, debe usar el token de acceso como contraseña de usuario de PostgreSQL.

Cuando se usa el cliente de línea de comandos `psql`, el token de acceso debe pasarse a través de la variable de entorno `PGPASSWORD`, ya que el token de acceso supera la longitud de la contraseña que `psql` puede aceptar directamente:

Ejemplo de Windows:

```cmd
set PGPASSWORD=<copy/pasted TOKEN value from step 2>
```

```PowerShell
$env:PGPASSWORD='<copy/pasted TOKEN value from step 2>'
```

Ejemplo de Linux/macOS:

```shell
export PGPASSWORD=<copy/pasted TOKEN value from step 2>
```

Ahora puede iniciar una conexión con Azure Database for PostgreSQL como lo haría normalmente:

```shell
psql "host=mydb.postgres... user=user@tenant.onmicrosoft.com@mydb dbname=postgres sslmode=require"
```

Consideraciones importantes al conectarse:

* `user@tenant.onmicrosoft.com` es el nombre del usuario de Azure AD o grupo al que está intentando conectarse.
* Anexe siempre el nombre del servidor después del nombre del grupo o usuario de Azure AD (por ejemplo, `@mydb`).
* Asegúrese de usar la forma exacta en que se ha escrito el nombre de usuario o grupo de Azure AD.
* Los nombres de usuario y grupo de Azure AD distinguen mayúsculas de minúsculas.
* Al conectarse como un grupo, use solo el nombre de grupo (por ejemplo, `GroupName@mydb`).
* Si el nombre contiene espacios, utilice `\` antes de cada espacio para escaparlo.

Ahora está autenticado en el servidor de PostgreSQL con la autenticación de Azure AD.

## <a name="creating-azure-ad-users-in-azure-database-for-postgresql"></a>Creación de usuarios Azure AD en Azure Database for PostgreSQL

Para agregar un usuario Azure AD a la base de datos de Azure Database for PostgreSQL, haga lo siguiente después de la conexión (consulte la sección más adelante sobre cómo conectarse):

1. En primer lugar, asegúrese de que el usuario Azure AD `<user>@yourtenant.onmicrosoft.com` un usuario válido en el suscriptor de Azure AD.
2. Inicie sesión en la instancia de Azure Database for PostgreSQL como usuario administrador de Azure AD.
3. Creación del rol `<user>@yourtenant.onmicrosoft.com` en Azure Database for PostgreSQL.
4. Cree `<user>@yourtenant.onmicrosoft.com` un miembro del rol azure_ad_user. Esto solo se debe proporcionar a los usuarios de Azure AD.

**Ejemplo**:

```sql
CREATE ROLE "user1@yourtenant.onmicrosoft.com" WITH LOGIN IN ROLE azure_ad_user;
```

> [!NOTE]
> La autenticación de un usuario a través de Azure AD no concede al usuario ningún permiso para tener acceso a los objetos en la base de datos de Azure Database for PostgreSQL. Asegúrese de concederle al usuario los permisos necesarios de forma manual.

## <a name="creating-azure-ad-groups-in-azure-database-for-postgresql"></a>Creación de grupos Azure AD en Azure Database for PostgreSQL

Para habilitar el acceso de un grupo de Azure AD a la base de datos, use el mismo mecanismo que para los usuarios, pero en su lugar especifique el nombre del grupo:

**Ejemplo**:

```sql
CREATE ROLE "Prod DB Readonly" WITH LOGIN IN ROLE azure_ad_user;
```

Al iniciar sesión, los miembros del grupo usarán sus token de acceso personal, pero firmarán con el nombre del grupo especificado como nombre de usuario.

## <a name="token-validation"></a>Validación de token

La autenticación de Azure AD en Azure Database for PostgreSQL asegura que el usuario existe en el servidor de PostgreSQL, y comprueba la validez del token con la validación del contenido del token. Se realizan los siguientes pasos de validación de token:

- El Token se firmó por Azure AD y no se manipuló
- El token se emitió por Azure AD para el suscriptor asociado con el servidor
- El token sigue vigente
- El token es para el recurso Azure Database for PostgreSQL (y no para otro recurso Azure)

## <a name="migrating-existing-postgresql-users-to-azure-ad-based-authentication"></a>Migración de usuarios existentes de PostgreSQL hacia la autenticación basada en Azure AD

Puede habilitar la autenticación de Azure AD para los usuarios existentes. Estos son dos casos a considerar:

### <a name="case-1-postgresql-username-matches-the-azure-ad-user-principal-name"></a>Caso 1: El nombre de usuario de PostgreSQL coincide con el nombre principal de usuario de Azure AD

En el caso improbable de que los usuarios existentes ya coincidan con los nombres de usuario de Azure AD, puede concederles el `azure_ad_user` rol para habilitarles la autenticación Azure AD:

```sql
GRANT azure_ad_user TO "existinguser@yourtenant.onmicrosoft.com";
```

Ahora podrán iniciar sesión con las credenciales de Azure AD en lugar de usar la contraseña de usuario de PostgreSQL previamente configurada.

### <a name="case-2-postgresql-username-is-different-than-the-azure-ad-user-principal-name"></a>Caso 2: El nombre de usuario de PostgreSQL no coincide con el nombre principal de usuario de Azure AD

Si un usuario de PostgreSQL no existe en Azure AD o tiene un nombre de usuario diferente, puede usar grupos de Azure AD para autenticarse como este usuario de PostgreSQL. Puede migrar los usuarios existentes de Azure Database for PostgreSQL hacia Azure AD con la creación de un grupo Azure AD que tenga un nombre que coincida con el del usuario de PostgreSQL y, después otorgar el rol azure_ad_user al usuario de PostgreSQL existente:

```sql
GRANT azure_ad_user TO "DBReadUser";
```

Esto asume que se ha creado un grupo "DBReadUser" en Azure AD. Los usuarios que pertenezcan a ese grupo ahora podrán iniciar sesión en la base de datos como este usuario.

## <a name="next-steps"></a>Pasos siguientes

* Revisión de los conceptos generales para [Ala autenticación de Azure Active Directory con Azure Database for PostgreSQL - Servidor único](concepts-aad-authentication.md)

<!--Image references-->

[1]: ./media/concepts-aad-authentication/authentication-flow.png
[2]: ./media/concepts-aad-authentication/set-aad-admin.png
