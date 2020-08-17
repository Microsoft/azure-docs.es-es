---
title: 'Uso de Azure Active Directory: Azure Database for MySQL'
description: Información sobre cómo configurar Azure Active Directory (Azure AD) para la autenticación con Azure Database for MySQL
author: lfittl-msft
ms.author: lufittl
ms.service: mysql
ms.topic: how-to
ms.date: 07/23/2020
ms.openlocfilehash: 0418785fe558503b716ff1e798446fb64db998b1
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87799845"
---
# <a name="use-azure-active-directory-for-authentication-with-mysql"></a>Uso de Azure Active Directory para la autenticación con MySQL

Este artículo le guiará por los pasos para configurar el acceso a Azure Active Directory con Azure Database for MySQL, y cómo conectarse con un token de Azure AD.

> [!IMPORTANT]
> La autenticación de Azure Active Directory solo está disponible para MySQL 5.7 y versiones más recientes.

## <a name="setting-the-azure-ad-admin-user"></a>Configuración del usuario administrador de Azure AD

Solo un usuario administrador de Azure AD puede crear o habilitar usuarios para la autenticación basada en Azure AD. Para crear un usuario administrador de Azure AD, haga lo siguiente

1. En Azure Portal, seleccione la instancia de Azure Database for MySQL que desea habilitar para Azure AD.
2. En Configuración, seleccione Administrador de Active Directory:

![establecer administrador de Azure AD][2]

3. Seleccione un usuario de Azure AD válido en el suscriptor del cliente para que sea administrador de Azure AD.

> [!IMPORTANT]
> Al establecer el administrador, se agrega un nuevo usuario al servidor de Azure Database for MySQL con permisos de administrador totales.

Solo se puede crear un administrador de Azure AD por servidor MySQL y la selección de otro sobrescribirá al administrador de Azure AD existente configurado para el servidor.

En una versión futura, se admitirá especificar un grupo de Azure AD en lugar de un usuario individual para que tenga varios administradores, pero actualmente no se admite.

Después de configurar el administrador, ahora puede iniciar sesión:

## <a name="connecting-to-azure-database-for-mysql-using-azure-ad"></a>Conexión a Azure Database for MySQL con Azure AD

En el siguiente diagrama de alto nivel se resume el flujo de trabajo del uso de la autenticación de Azure AD con Azure Database for MySQL:

![flujo de autenticación][1]

Hemos diseñado la integración de Azure AD para que funcione con herramientas comunes de MySQL, como la CLI mysql, que no son compatibles con Azure AD y solo admiten la especificación de nombre de usuario y contraseña al conectarse a MySQL. Pasamos el token de Azure AD como la contraseña tal como se muestra en la imagen anterior.

Actualmente hemos probado a los siguientes clientes:

- MySQLWorkbench 
- CLI mysql

También hemos probado los controladores de aplicación más comunes, puede ver los detalles al final de esta página.

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

Invoque la herramienta CLI de Azure para adquirir un token de acceso para el usuario autenticado de Azure AD en el paso 1 para tener acceso a Azure Database for MySQL.

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

El token es una cadena de Base 64 que codifica toda la información sobre el usuario autenticado, y que está dirigida al servicio de Azure Database for MySQL.

> [!NOTE]
> La validez del token de acceso es de entre 5 y 60 minutos. Se recomienda obtener el token de acceso justo antes del inicio de sesión en Azure Database for MySQL.

### <a name="step-3-use-token-as-password-for-logging-in-with-mysql"></a>Paso 3: Usar el token como contraseña para iniciar sesión con MySQL

Al conectarse, debe usar el token de acceso como contraseña de usuario de MySQL. Al usar clientes de GUI, como MySQLWorkbench, puede usar el método anterior para recuperar el token. 

Al usar la CLI, puede usar este atajo para conectarse: 

**Ejemplo (Linux/macOS):**
```
mysql -h mydb.mysql.database.azure.com \ 
  --user user@tenant.onmicrosoft.com@mydb \ 
  --enable-cleartext-plugin \ 
  --password=`az account get-access-token --resource-type oss-rdbms --output tsv --query accessToken`
```

Consideraciones importantes al conectarse:

* `user@tenant.onmicrosoft.com` es el nombre del usuario de Azure AD o grupo al que está intentando conectarse.
* Anexe siempre el nombre del servidor después del nombre del grupo o usuario de Azure AD (por ejemplo, `@mydb`).
* Asegúrese de usar la forma exacta en que se ha escrito el nombre de usuario o grupo de Azure AD.
* Los nombres de usuario y grupo de Azure AD distinguen mayúsculas de minúsculas.
* Al conectarse como un grupo, use solo el nombre de grupo (por ejemplo, `GroupName@mydb`).
* Si el nombre contiene espacios, utilice `\` antes de cada espacio para escaparlo.

Tenga en cuenta el valor "enable-cleartext-plugin", debe usar una configuración similar con otros clientes para asegurarse de que el token se envíe al servidor sin síntesis del mensaje.

Ahora está autenticado en el servidor MySQL con la autenticación de Azure AD.

## <a name="creating-azure-ad-users-in-azure-database-for-mysql"></a>Creación de usuarios Azure AD en Azure Database for MySQL

Para agregar un usuario Azure AD a la base de datos de Azure Database for MySQL, haga lo siguiente después de la conexión (consulte la sección más adelante sobre cómo conectarse):

1. En primer lugar, asegúrese de que el usuario Azure AD `<user>@yourtenant.onmicrosoft.com` un usuario válido en el suscriptor de Azure AD.
2. Inicie sesión en la instancia de Azure Database for MySQL como usuario administrador de Azure AD.
3. Cree el usuario `<user>@yourtenant.onmicrosoft.com` en Azure Database for MySQL.

**Ejemplo**:

```sql
CREATE AADUSER 'user1@yourtenant.onmicrosoft.com';
```

En el caso de los nombres de usuario que superen los 32 caracteres, se recomienda usar un alias en su lugar, para usarlo al conectarse: 

Ejemplo:

```sql
CREATE AADUSER 'userWithLongName@yourtenant.onmicrosoft.com' as 'userDefinedShortName'; 
```

> [!NOTE]
> La autenticación de un usuario a través de Azure AD no concede al usuario ningún permiso para tener acceso a los objetos en la base de datos de Azure Database for MySQL. Asegúrese de concederle al usuario los permisos necesarios de forma manual.

## <a name="creating-azure-ad-groups-in-azure-database-for-mysql"></a>Creación de grupos de Azure AD en Azure Database for MySQL

Para habilitar el acceso de un grupo de Azure AD a la base de datos, use el mismo mecanismo que para los usuarios, pero en su lugar especifique el nombre del grupo:

**Ejemplo**:

```sql
CREATE AADUSER 'Prod_DB_Readonly';
```

Al iniciar sesión, los miembros del grupo usarán sus token de acceso personal, pero firmarán con el nombre del grupo especificado como nombre de usuario.

## <a name="token-validation"></a>Validación de token

La autenticación de Azure AD en Azure Database for MySQL asegura que el usuario existe en el servidor MySQL, y comprueba la validez del token con la validación del contenido del token. Se realizan los siguientes pasos de validación de token:

-   El Token se firmó por Azure AD y no se manipuló
-   El token se emitió por Azure AD para el suscriptor asociado con el servidor
-   El token sigue vigente
-   El token es para el recurso Azure Database for MySQL (y no para otro recurso de Azure)

## <a name="compatibility-with-application-drivers"></a>Compatibilidad con controladores de aplicaciones

Se admite la mayoría de los controladores; sin embargo, asegúrese de usar la configuración para enviar la contraseña en texto sin cifrar, de modo que el token se envíe sin modificaciones.

* C/C++
  * libmysqlclient: Compatible
  * mysql-connector-c++: Compatible
* Java
  * Connector/J (mysql-connector-java): Compatible, debe usar la configuración `useSSL`
* Python
  * Connector/Python: Compatible
* Ruby
  * mysql2: Compatible
* .NET
  * mysql-connector-net: Compatible, debe agregar complementos para mysql_clear_password
  * mysql-net/MySqlConnector: Compatible
* Node.js
  * mysqljs: No compatible (no envía el token en texto no cifrado sin revisión)
  * node-mysql2: Compatible
* Perl
  * DBD::mysql: Compatible
  * Net::MySQL: No compatible
* Go
  * go-sql-driver: Compatible, agregue `?tls=true&allowCleartextPasswords=true` a la cadena de conexión

## <a name="next-steps"></a>Pasos siguientes

* Revisión de los conceptos generales para la [Autenticación de Azure Active Directory con Azure Database for MySQL](concepts-azure-ad-authentication.md)

<!--Image references-->

[1]: ./media/concepts-azure-ad-authentication/authentication-flow.png
[2]: ./media/concepts-azure-ad-authentication/set-azure-ad-admin.png
