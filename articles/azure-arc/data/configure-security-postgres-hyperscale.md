---
title: Configuración de la seguridad del grupo de servidores de PostgreSQL Hyperscale habilitado para Azure Arc
description: Configuración de la seguridad del grupo de servidores de PostgreSQL Hyperscale habilitado para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: b166348031e9f72e8005e866a198855db9c01a9c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932247"
---
# <a name="configure-security-for-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Configuración de la seguridad del grupo de servidores de PostgreSQL Hyperscale habilitado para Azure Arc

En este documento se describen diversos aspectos relacionados con la seguridad de su grupo de servidores:
- Cifrado en reposo
- Administración de usuarios
   - Perspectivas generales
   - Cambio de la contraseña del usuario administrativo de _Postgres_

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="encryption-at-rest"></a>Cifrado en reposo
Puede implementar el cifrado en reposo mediante el cifrado de los discos en los que se almacenan las bases de datos o mediante las funciones de base de datos para cifrar los datos que se insertan o actualizan.

### <a name="hardware-linux-host-volume-encryption"></a>Hardware: Cifrado del volumen del host Linux
Implemente el cifrado de datos del sistema para proteger todos los datos que residen en los discos usados por su configuración de Data Services habilitado para Azure Arc. Puede leer más información sobre este tema aquí:
- [Cifrado de datos en reposo](https://wiki.archlinux.org/index.php/Data-at-rest_encryption) en Linux en general 
- Cifrado de discos con el comando de cifrado LUKS `cryptsetup` (Linux) https://www.cyberciti.biz/security/howto-linux-hard-disk-encryption-with-luks-cryptsetup-command/) específicamente desde que Data Services habilitado para Azure Arc se ejecuta en la infraestructura física que proporcione, se encarga de proteger la infraestructura.

### <a name="software-use-the-postgresql-pgcrypto-extension-in-your-server-group"></a>Software: Uso de la extensión `pgcrypto` de PostgreSQL en el grupo de servidores
Además de cifrar los discos usados para hospedar la configuración de Azure Arc, puede configurar el grupo de servidores de PostgreSQL Hyperscale habilitado para Azure Arc para exponer los mecanismos que las aplicaciones pueden usar para cifrar los datos de las bases de datos. La extensión de `pgcrypto` forma parte de las extensiones de `contrib` de Postgres y está disponible en el grupo de servidores de PostgreSQL Hyperscale habilitado para Azure Arc. [Aquí](https://www.postgresql.org/docs/current/pgcrypto.html) encontrará detalles sobre la extensión de `pgcrypto`.
En resumen, con los siguientes comandos, se habilita la extensión, se crea y se usa:


#### <a name="create-the-pgcrypto-extension"></a>Cree la extensión de `pgcrypto`
Conéctese al grupo de servidores con la herramienta cliente que prefiera y ejecute la consulta estándar de PostgreSQL:
```console
CREATE EXTENSION pgcrypto;
```

> [Aquí](get-connection-endpoints-and-connection-strings-postgres-hyperscale.md) encontrará detalles sobre cómo conectarse.

#### <a name="verify-the-list-the-extensions-ready-to-use-in-your-server-group"></a>Comprobación de la lista de las extensiones listas para su uso en el grupo de servidores
Puede comprobar que la extensión de `pgcrypto` está lista para usarse mediante la enumeración de las extensiones disponibles en su grupo de servidores.
Conéctese al grupo de servidores con la herramienta cliente que prefiera y ejecute la consulta estándar de PostgreSQL:
```console
select * from pg_extension;
```
Debería ver `pgcrypto` si lo habilitó y lo creó con los comandos indicados anteriormente.

#### <a name="use-the-pgcrypto-extension"></a>Uso de la extensión de `pgcrypto`
Ahora puede ajustar el código de las aplicaciones para que utilicen cualquiera de las funciones que ofrece `pgcrypto`:
- Funciones hash generales
- Funciones hash de contraseñas
- Funciones de cifrado de PGP
- Funciones de cifrado sin formato
- Funciones de datos aleatorios

Por ejemplo, para generar valores hash. Ejecute el comando:

```console
Select crypt('Les sanglots longs des violons de l_automne', gen_salt('md5'));
```

Devuelve el siguiente valor hash:

```console
              crypt
------------------------------------
 $1$/9ACBYOV$z52PAGjQ5WTU9xvEECBNv/   
```

O, por ejemplo:

```console
select hmac('Les sanglots longs des violons de l_automne', 'md5', 'sha256');
```

devuelve el siguiente valor hash:

```console
                                hmac
--------------------------------------------------------------------
 \xd4e4790b69d2cc8dbce3385ee63272bc7760f1603640bb211a7b864e695570c5
```

O, por ejemplo, para almacenar datos cifrados como una contraseña:

Supongamos que mi aplicación almacena los secretos en la siguiente tabla:

```console
create table mysecrets(USERid int, USERname char(255), USERpassword char(512));
```

Y cifro su contraseña al crear un usuario:

```console
insert into mysecrets values (1, 'Me', crypt('MySecretPasswrod', gen_salt('md5')));
```

Ahora, veo que mi contraseña está cifrada:

```console
select * from mysecrets;
```

Salida:

- USERid: 1
- USERname: Yo
- USERpassword: $1$Uc7jzZOp$NTfcGo7F10zGOkXOwjHy31

Cuando me conecto con mi aplicación y paso una contraseña, buscará en la tabla de `mysecrets` y devolverá el nombre del usuario si hay una coincidencia entre la contraseña que se proporciona a la aplicación y las contraseñas almacenadas en la tabla. Por ejemplo:

- Paso la contraseña equivocada:
   ```console
   select USERname from mysecrets where (USERpassword = crypt('WrongPassword', USERpassword));
   ```

   Resultados 

   ```returns
    USERname
   ---------
   (0 rows)
   ```
- Paso la contraseña correcta:

   ```console
   select USERname from mysecrets where (USERpassword = crypt('MySecretPasswrod', USERpassword));
   ``` 

   Salida:

   ```output
    USERname
   ---------
   Me
   (1 row)
   ```

En este pequeño ejemplo se muestra que puede cifrar datos en reposo (almacenar datos cifrados) en PostgreSQL Hyperscale habilitado para Azure Arc mediante la extensión de `pgcrypto` de Postgres y las aplicaciones pueden usar las funciones que ofrece `pgcrypto` para manipular estos datos cifrados.

## <a name="user-management"></a>Administración de usuarios
### <a name="general-perspectives"></a>Perspectivas generales
Puede usar la forma estándar Postgres para crear usuarios o roles. Sin embargo, si lo hace, estos artefactos solo estarán disponibles en el rol de coordinador. Durante la versión preliminar, estos usuarios o roles todavía no podrán tener acceso a los datos que se distribuyen fuera del nodo Coordinador y en los nodos Trabajador del grupo de servidores. La razón es que, en la versión preliminar, la definición de usuario no se replica en los nodos Trabajador.

### <a name="change-the-password-of-the-_postgres_-administrative-user"></a>Cambio de la contraseña del usuario administrativo de _Postgres_
Hiperescala de PostgreSQL habilitado para Azure Arc incluye el usuario administrativo de Postgres estándar _postgres_, para el que se establece la contraseña al crear el grupo de servidores.
El formato general del comando para cambiar su contraseña es:
```console
azdata arc postgres server edit --name <server group name> --admin-password <new password>
```
La contraseña se establecerá en el valor de la variable de entorno de la **sesión** de  AZDATA_PASSWORD si existe. Si no es así, se solicitará al usuario un valor.
Para comprobar qué sucede si existe la variable de entorno de la sesión de AZDATA_PASSWORD o el valor que se establece, ejecute:
```console
printenv AZDATA_PASSWORD
```
Es posible que desee eliminar su valor si prefiere que se le pida que escriba una nueva contraseña.


## <a name="next-steps"></a>Pasos siguientes
- Podrá leer detalles sobre la extensión de `pgcrypto`[aquí](https://www.postgresql.org/docs/current/pgcrypto.html).
- Lea los detalles sobre cómo usar las extensiones de Postgres [aquí](using-extensions-in-postgresql-hyperscale-server-group.md).

