---
title: 'Inicio rápido: Conexión mediante Node.js: Azure Database for MySQL'
description: En este tutorial rápido se proporcionan ejemplos de código Node.js que se pueden usar para conectarse a Azure Database for MySQL y consultar datos en este servicio.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019, devx-track-javascript
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 5/26/2020
ms.openlocfilehash: 1f8f4dee2ed1e6dc5cdf6985c1149db0dcaed7c2
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88185934"
---
# <a name="quickstart-use-nodejs-to-connect-and-query-data-in-azure-database-for-mysql"></a>Inicio rápido: Uso de Node.js para conectar y consultar datos en Azure Database for MySQL

En este inicio rápido, se va a conectar a una instancia de Azure Database for MySQL mediante Node.js. Puede usar instrucciones SQL para consultar, insertar, actualizar y eliminar datos de la base de datos en las plataformas Mac, Ubuntu Linux y Windows. 

En este artículo se da por hecho que está familiarizado con el desarrollo mediante Node.js y que nunca ha usado Azure Database for MySQL.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- Un servidor de Azure Database for MySQL. [Creación de un servidor de Azure Database for MySQL mediante Azure Portal](quickstart-create-mysql-server-database-using-azure-portal.md) o [Creación de un servidor de Azure Database for MySQL mediante la CLI de Azure](quickstart-create-mysql-server-database-using-azure-cli.md).

> [!IMPORTANT] 
> Asegúrese de que a la dirección IP desde la que se conecta se le han agregado las reglas de firewall del servidor desde [Azure Portal](./howto-manage-firewall-using-portal.md) o la [CLI de Azure](./howto-manage-firewall-using-cli.md)

## <a name="install-nodejs-and-the-mysql-connector"></a>Instalación de Node.js y el conector de MySQL

Dependiendo de la plataforma, siga las instrucciones de la sección apropiada para instalar [Node.js](https://nodejs.org). Use npm para instalar el paquete de [mysql](https://www.npmjs.com/package/mysql) y sus dependencias en la carpeta de proyecto.

### <a name="windows"></a>Windows

1. Visite la [página de descargas de Node.js](https://nodejs.org/en/download/) y luego seleccione la opción deseada de Windows Installer.
2. Cree una carpeta de proyecto local, como `nodejsmysql`. 
3. Abra el símbolo del sistema y luego cambie el directorio a la carpeta de proyecto, por ejemplo, `cd c:\nodejsmysql\`.
4. Ejecute la herramienta NPM para instalar la biblioteca de mysql en la carpeta de proyecto.

   ```cmd
   cd c:\nodejsmysql\
   "C:\Program Files\nodejs\npm" install mysql
   "C:\Program Files\nodejs\npm" list
   ```

5. Para comprobar la instalación, examine el texto de salida de `npm list`. El número de versión puede variar a medida que se publiquen nuevas revisiones.

### <a name="linux-ubuntu"></a>Linux (Ubuntu)

1. Ejecute los siguientes comandos para instalar **Node.js** y **npm**, el Administrador de paquetes para Node.js.

   ```bash
   sudo apt-get install -y nodejs npm
   ```

2. Ejecute los comandos siguientes para crear una carpeta de proyecto `mysqlnodejs` e instale el paquete mysql en ella.

   ```bash
   mkdir nodejsmysql
   cd nodejsmysql
   npm install --save mysql
   npm list
   ```
3. Para comprobar la instalación, examine el texto de salida de npm list. El número de versión puede variar a medida que se publiquen nuevas revisiones.

### <a name="mac-os"></a>Mac OS

1. Escriba los comandos siguientes para instalar **brew**, un administrador de paquetes fácil de usar para Mac OS X y **Node.js**.

   ```bash
   ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   brew install node
   ```
2. Ejecute los comandos siguientes para crear una carpeta de proyecto `mysqlnodejs` e instale el paquete mysql en ella.

   ```bash
   mkdir nodejsmysql
   cd nodejsmysql
   npm install --save mysql
   npm list
   ```

3. Para comprobar la instalación, examine el texto de salida de `npm list`. El número de versión puede variar a medida que se publiquen nuevas revisiones.

## <a name="get-connection-information"></a>Obtención de información sobre la conexión

Obtenga la información de conexión necesaria para conectarse a Azure Database for MySQL. Necesitará el nombre completo del servidor y las credenciales de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En el menú izquierdo de Azure Portal, seleccione **Todos los recursos** y, luego, busque el servidor que ha creado, por ejemplo, **mydemoserver**.
3. Seleccione el nombre del servidor.
4. En el panel **Información general** del servidor, anote el **nombre del servidor** y el **nombre de inicio de sesión del administrador del servidor**. Si olvida la contraseña, puede restablecerla en este panel.
 ![Nombre del servidor de Azure Database for MySQL](./media/connect-nodejs/server-name-azure-database-mysql.png)

## <a name="running-the-javascript-code-in-nodejs"></a>Ejecución del código de JavaScript en Node.js

1. Pegue el código de JavaScript en archivos de texto y luego guárdelos en una carpeta de proyecto con la extensión .js (por ejemplo, C:\nodejsmysql\createtable.js o /home/username/nodejsmysql/createtable.js).
2. Abra el símbolo del sistema o el shell de Bash y luego cambie el directorio a la carpeta de proyecto `cd nodejsmysql`.
3. Para ejecutar la aplicación, introduzca el comando node seguido del nombre de archivo, por ejemplo, `node createtable.js`.
4. En Windows, si la aplicación de Node no está en la ruta de la variable de entorno, puede que deba usar la ruta de acceso completa para iniciar la aplicación de Node, por ejemplo, `"C:\Program Files\nodejs\node.exe" createtable.js`.

## <a name="connect-create-table-and-insert-data"></a>Conexión, creación de una tabla e inserción de datos

Use el código siguiente para conectarse y cargar los datos mediante las instrucciones SQL **CREATE TABLE** e **INSERT INTO**.

El método [mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) se usa para interactuar con el servidor de MySQL. La función [connect()](https://github.com/mysqljs/mysql#establishing-connections) se usa para establecer la conexión al servidor. La función [query()](https://github.com/mysqljs/mysql#performing-queries) se usa para ejecutar la consulta SQL en la base de datos MySQL. 

Reemplace los parámetros `host`, `user`, `password` y `database` por los valores especificados al crear el servidor y la base de datos.

```javascript
const mysql = require('mysql');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
    if (err) { 
        console.log("!!! Cannot connect !!! Error:");
        throw err;
    }
    else
    {
       console.log("Connection established.");
           queryDatabase();
    }   
});

function queryDatabase(){
       conn.query('DROP TABLE IF EXISTS inventory;', function (err, results, fields) { 
            if (err) throw err; 
            console.log('Dropped inventory table if existed.');
        })
       conn.query('CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);', 
            function (err, results, fields) {
                if (err) throw err;
            console.log('Created inventory table.');
        })
       conn.query('INSERT INTO inventory (name, quantity) VALUES (?, ?);', ['banana', 150], 
            function (err, results, fields) {
                if (err) throw err;
            else console.log('Inserted ' + results.affectedRows + ' row(s).');
        })
       conn.query('INSERT INTO inventory (name, quantity) VALUES (?, ?);', ['orange', 154], 
            function (err, results, fields) {
                if (err) throw err;
            console.log('Inserted ' + results.affectedRows + ' row(s).');
        })
       conn.query('INSERT INTO inventory (name, quantity) VALUES (?, ?);', ['apple', 100], 
        function (err, results, fields) {
                if (err) throw err;
            console.log('Inserted ' + results.affectedRows + ' row(s).');
        })
       conn.end(function (err) { 
        if (err) throw err;
        else  console.log('Done.') 
        });
};
```

## <a name="read-data"></a>Lectura de datos

Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **SELECT**. 

El método [mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) se usa para interactuar con el servidor de MySQL. El método [connect()](https://github.com/mysqljs/mysql#establishing-connections) se usa para establecer la conexión al servidor. El método [query()](https://github.com/mysqljs/mysql#performing-queries) se usa para ejecutar la consulta SQL en la base de datos MySQL. La matriz de resultados se usa para almacenar los resultados de la consulta.

Reemplace los parámetros `host`, `user`, `password` y `database` por los valores especificados al crear el servidor y la base de datos.

```javascript
const mysql = require('mysql');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
        if (err) { 
            console.log("!!! Cannot connect !!! Error:");
            throw err;
        }
        else {
            console.log("Connection established.");
            readData();
        }   
    });

function readData(){
        conn.query('SELECT * FROM inventory', 
            function (err, results, fields) {
                if (err) throw err;
                else console.log('Selected ' + results.length + ' row(s).');
                for (i = 0; i < results.length; i++) {
                    console.log('Row: ' + JSON.stringify(results[i]));
                }
                console.log('Done.');
            })
       conn.end(
           function (err) { 
                if (err) throw err;
                else  console.log('Closing connection.') 
        });
};
```

## <a name="update-data"></a>Actualización de datos

Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **UPDATE**. 

El método [mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) se usa para interactuar con el servidor de MySQL. El método [connect()](https://github.com/mysqljs/mysql#establishing-connections) se usa para establecer la conexión al servidor. El método [query()](https://github.com/mysqljs/mysql#performing-queries) se usa para ejecutar la consulta SQL en la base de datos MySQL. 

Reemplace los parámetros `host`, `user`, `password` y `database` por los valores especificados al crear el servidor y la base de datos.

```javascript
const mysql = require('mysql');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
        if (err) { 
            console.log("!!! Cannot connect !!! Error:");
            throw err;
        }
        else {
            console.log("Connection established.");
            updateData();
        }   
    });

function updateData(){
       conn.query('UPDATE inventory SET quantity = ? WHERE name = ?', [200, 'banana'], 
            function (err, results, fields) {
                if (err) throw err;
                else console.log('Updated ' + results.affectedRows + ' row(s).');
        })
       conn.end(
           function (err) { 
                if (err) throw err;
                else  console.log('Done.') 
        });
};
```

## <a name="delete-data"></a>Eliminación de datos

Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **DELETE**. 

El método [mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) se usa para interactuar con el servidor de MySQL. El método [connect()](https://github.com/mysqljs/mysql#establishing-connections) se usa para establecer la conexión al servidor. El método [query()](https://github.com/mysqljs/mysql#performing-queries) se usa para ejecutar la consulta SQL en la base de datos MySQL. 

Reemplace los parámetros `host`, `user`, `password` y `database` por los valores especificados al crear el servidor y la base de datos.

```javascript
const mysql = require('mysql');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
        if (err) { 
            console.log("!!! Cannot connect !!! Error:");
            throw err;
        }
        else {
            console.log("Connection established.");
            deleteData();
        }   
    });

function deleteData(){
       conn.query('DELETE FROM inventory WHERE name = ?', ['orange'], 
            function (err, results, fields) {
                if (err) throw err;
                else console.log('Deleted ' + results.affectedRows + ' row(s).');
        })
       conn.end(
           function (err) { 
                if (err) throw err;
                else  console.log('Done.') 
        });
};
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Migración de una base de datos mediante exportación e importación](./concepts-migrate-import-export.md)
