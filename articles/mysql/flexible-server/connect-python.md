---
title: 'Inicio rápido: Conexión mediante Python a la instancia de Azure Database for MySQL con la opción Servidor flexible'
description: En este inicio rápido se proporcionan ejemplos de código de Python que se pueden usar para conectarse a Azure Database for MySQL con la opción Servidor flexible y consultar datos en este servicio.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.custom: mvc
ms.devlang: python
ms.topic: quickstart
ms.date: 9/21/2020
ms.openlocfilehash: ff14bb1daeef6fc54ee5d11632ad98a29db2a172
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945250"
---
# <a name="quickstart-use-python-to-connect-and-query-data-in-azure-database-for-mysql---flexible-server"></a>Inicio rápido: Use Python para conectarse a datos y consultarlos en Azure Database for MySQL con la opción Servidor flexible.

> [!IMPORTANT] 
> Actualmente, Azure Database for MySQL con la opción Servidor flexible se encuentra en versión preliminar pública.

En este inicio rápido se va a conectar a una instancia de Azure Database for MySQL con la opción Servidor flexible mediante Python. Puede usar instrucciones SQL para consultar, insertar, actualizar y eliminar datos de la base de datos en las plataformas Mac, Ubuntu Linux y Windows. 

En este artículo se da por hecho que está familiarizado con el desarrollo mediante Python y que nunca ha usado la instancia de Azure Database for MySQL con la opción Servidor flexible.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
* Azure Database for MySQL con la opción Servidor flexible. Para crear un servidor flexible, consulte [Creación de una instancia de Azure Database for MySQL con la opción Servidor flexible mediante Azure Portal](./quickstart-create-server-portal.md) o [Creación de una instancia de Azure Database for MySQL con la opción Servidor flexible mediante la CLI de Azure](./quickstart-create-server-cli.md).

## <a name="preparing-your-client-workstation"></a>Preparación de la estación de trabajo de cliente
- Si creó el servidor flexible con la opción *Acceso privado (integración con red la virtual)* , deberá conectarse a su servidor desde un recurso de la misma red virtual que el servidor. Puede crear una máquina virtual y agregarla a la red virtual creada con el servidor flexible. Consulte [Creación y administración de la red virtual de Azure Database for MySQL con la opción Servidor flexible mediante la CLI de Azure](./how-to-manage-virtual-network-cli.md).
- Si creó el servidor flexible con la opción *Acceso público (direcciones IP permitidas)* , puede agregar la dirección IP local a la lista de reglas de firewall del servidor. Consulte [Creación y administración de reglas de firewall de Azure Database for MySQL con la opción Servidor flexible mediante la CLI de Azure](./how-to-manage-firewall-cli.md).

## <a name="install-python-and-the-mysql-connector"></a>Instalación de Python y el conector de MySQL

Instale Python y el conector de MySQL para Python en su propio equipo de la forma siguiente: 

> [!NOTE]
> En este inicio rápido se usa un enfoque de consulta SQL sin procesar para conectarse a MySQL. Si usa una plataforma web, utilice el conector recomendado para ella, por ejemplo, [mysqlclient](https://pypi.org/project/mysqlclient/) para Django.

1. Descargue e instale [Python 3.7 o superior](https://www.python.org/downloads/) para su sistema operativo. Asegúrese de agregar Python a `PATH`, porque el conector de MySQL lo requiere.
   
1. Abra un símbolo del sistema o un shell de `bash` y compruebe la versión de Python mediante la ejecución de `python -V` con el modificador V mayúscula.
   
1. El instalador de paquetes de `pip` se incluye en las versiones más recientes de Python. Actualice `pip` a la versión más reciente mediante la ejecución de `pip install -U pip`. 
   
   Si `pip` no está instalado, puede descargarlo e instalarlo con `get-pip.py`. Para más información, consulte [Instalación](https://pip.pypa.io/en/stable/installing/). 
   
1. Utilice `pip` para instalar el conector de MySQL para Python y sus dependencias:
   
   ```bash
   pip install mysql-connector-python
   ```
   
   También puede instalar el conector de Python para MySQL en [mysql.com](https://dev.mysql.com/downloads/connector/python/). Para más información sobre el conector de MySQL para Python, consulte la [guía del desarrollador de Python o del conector de MySQL](https://dev.mysql.com/doc/connector-python/en/). 

## <a name="get-connection-information"></a>Obtención de información sobre la conexión

Obtenga la información de conexión necesaria para conectarse a Azure Database for MySQL con la opción Servidor flexible en Azure Portal. Necesitará el nombre del servidor, el nombre de la base de datos y las credenciales de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
   
1. En la barra de búsqueda del portal, busque y seleccione la instancia de Azure Database for MySQL con la opción Servidor flexible que ha creado, como **mydemoserver**.
   
   <!---:::image type="content" source="./media/connect-python/1_server-overview-name-login.png" alt-text="Azure Database for MySQL Flexible Server name":::-->
   
1. En la página **Información general** del servidor, anote el **nombre del servidor** y el **nombre de inicio de sesión del administrador del servidor**. Si olvida la contraseña, puede restablecerla también en esta página.
   
   <!---:::image type="content" source="./media/connect-python/azure-database-for-mysql-server-overview-name-login.png" alt-text="Azure Database for MySQL Flexible Server name":::-->

## <a name="code-samples"></a>Ejemplos de código

### <a name="run-below-mentioned-python-code-samples"></a>Ejecute los siguientes ejemplos de código de Python.
Para cada ejemplo de código de este artículo:

1. Cree un nuevo archivo en un editor de texto.
1. Agregue el ejemplo de código al archivo. En el código, reemplace los marcadores de posición `<mydemoserver>`, `<myadmin>`, `<mypassword>`y `<mydatabase>` por los valores de la base de datos y el servidor de MySQL.
1. Guarde el archivo en una carpeta de proyecto con una extensión *.py*, como *C:\pythonmysql\createtable.py* o */home/username/pythonmysql/createtable.py*.
1. Para ejecutar el código, abra un símbolo del sistema o el shell de `bash` y luego cambie el directorio a la carpeta del proyecto, por ejemplo `cd pythonmysql`. Escriba el comando `python createtable.py` seguido del nombre de archivo, por ejemplo `python`, y presione Entrar. 
   
   > [!NOTE]
   > En Windows, si no se encuentra *python.exe*, puede que tenga que agregar la ruta de acceso de Python a la variable de entorno PATH o proporcione la ruta de acceso completa a *python.exe*, por ejemplo `C:\python27\python.exe createtable.py`.

### <a name="create-a-table-and-insert-data"></a>Crear una tabla e insertar datos

Use el código siguiente para conectarse al servidor y a la base de datos, crear una tabla y cargar los datos mediante una instrucción SQL **INSERT**. 

El código importa la biblioteca mysql.connector y utiliza la función [connect()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysql-connector-connect.html) para conectarse al servidor flexible mediante los [argumentos](https://dev.mysql.com/doc/connector-python/en/connector-python-connectargs.html) de la colección de configuración. El código usa un cursor en la conexión, y el método [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) ejecuta la consulta SQL en la base de datos MySQL. 

```python
import mysql.connector
from mysql.connector import errorcode

# Obtain connection string information from the portal
config = {
  'host':'<mydemoserver>.mysql.database.azure.com',
  'user':'<myadmin>',
  'password':'<mypassword>',
  'database':'<mydatabase>'
}

# Construct connection string
try:
   conn = mysql.connector.connect(**config)
   print("Connection established")
except mysql.connector.Error as err:
  if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
    print("Something is wrong with the user name or password")
  elif err.errno == errorcode.ER_BAD_DB_ERROR:
    print("Database does not exist")
  else:
    print(err)
else:
  cursor = conn.cursor()

  # Drop previous table of same name if one exists
  cursor.execute("DROP TABLE IF EXISTS inventory;")
  print("Finished dropping table (if existed).")

  # Create table
  cursor.execute("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
  print("Finished creating table.")

  # Insert some data into table
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("banana", 150))
  print("Inserted",cursor.rowcount,"row(s) of data.")
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("orange", 154))
  print("Inserted",cursor.rowcount,"row(s) of data.")
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("apple", 100))
  print("Inserted",cursor.rowcount,"row(s) of data.")

  # Cleanup
  conn.commit()
  cursor.close()
  conn.close()
  print("Done.")
```

### <a name="read-data"></a>Lectura de datos

Use el código siguiente para conectarse y leer los datos mediante la instrucción SQL **SELECT**. 

El código importa la biblioteca mysql.connector y utiliza la función [connect()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysql-connector-connect.html) para conectarse al servidor flexible mediante los [argumentos](https://dev.mysql.com/doc/connector-python/en/connector-python-connectargs.html) de la colección de configuración. El código usa un cursor en la conexión, y el método [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) ejecuta la consulta SQL en la base de datos MySQL. 

El código lee las filas de datos con el método [fetchall()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-fetchall.html), mantiene el conjunto de resultados en una fila de colección y usa un iterador `for` para recorrer las filas.

```python
import mysql.connector
from mysql.connector import errorcode

# Obtain connection string information from the portal
config = {
  'host':'<mydemoserver>.mysql.database.azure.com',
  'user':'<myadmin>',
  'password':'<mypassword>',
  'database':'<mydatabase>'
}

# Construct connection string
try:
   conn = mysql.connector.connect(**config)
   print("Connection established")
except mysql.connector.Error as err:
  if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
    print("Something is wrong with the user name or password")
  elif err.errno == errorcode.ER_BAD_DB_ERROR:
    print("Database does not exist")
  else:
    print(err)
else:
  cursor = conn.cursor()

  # Read data
  cursor.execute("SELECT * FROM inventory;")
  rows = cursor.fetchall()
  print("Read",cursor.rowcount,"row(s) of data.")

  # Print all rows
  for row in rows:
    print("Data row = (%s, %s, %s)" %(str(row[0]), str(row[1]), str(row[2])))

  # Cleanup
  conn.commit()
  cursor.close()
  conn.close()
  print("Done.")
```

### <a name="update-data"></a>Actualización de datos

Use el código siguiente para conectarse y actualizar los datos mediante la instrucción SQL **UPDATE**. 

El código importa la biblioteca mysql.connector y utiliza la función [connect()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysql-connector-connect.html) para conectarse al servidor flexible mediante los [argumentos](https://dev.mysql.com/doc/connector-python/en/connector-python-connectargs.html) de la colección de configuración. El código usa un cursor en la conexión, y el método [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) ejecuta la consulta SQL en la base de datos MySQL. 

```python
import mysql.connector
from mysql.connector import errorcode

# Obtain connection string information from the portal
config = {
  'host':'<mydemoserver>.mysql.database.azure.com',
  'user':'<myadmin>',
  'password':'<mypassword>',
  'database':'<mydatabase>'
}

# Construct connection string
try:
   conn = mysql.connector.connect(**config)
   print("Connection established")
except mysql.connector.Error as err:
  if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
    print("Something is wrong with the user name or password")
  elif err.errno == errorcode.ER_BAD_DB_ERROR:
    print("Database does not exist")
  else:
    print(err)
else:
  cursor = conn.cursor()

  # Update a data row in the table
  cursor.execute("UPDATE inventory SET quantity = %s WHERE name = %s;", (200, "banana"))
  print("Updated",cursor.rowcount,"row(s) of data.")

  # Cleanup
  conn.commit()
  cursor.close()
  conn.close()
  print("Done.")
```

### <a name="delete-data"></a>Eliminación de datos

Use el código siguiente para conectarse y eliminar datos mediante la instrucción SQL **DELETE**. 

El código importa la biblioteca mysql.connector y utiliza la función [connect()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysql-connector-connect.html) para conectarse al servidor flexible mediante los [argumentos](https://dev.mysql.com/doc/connector-python/en/connector-python-connectargs.html) de la colección de configuración. El código usa un cursor en la conexión, y el método [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) ejecuta la consulta SQL en la base de datos MySQL. 

```python
import mysql.connector
from mysql.connector import errorcode

# Obtain connection string information from the portal
config = {
  'host':'<mydemoserver>.mysql.database.azure.com',
  'user':'<myadmin>',
  'password':'<mypassword>',
  'database':'<mydatabase>'
}

# Construct connection string
try:
   conn = mysql.connector.connect(**config)
   print("Connection established.")
except mysql.connector.Error as err:
  if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
    print("Something is wrong with the user name or password.")
  elif err.errno == errorcode.ER_BAD_DB_ERROR:
    print("Database does not exist.")
  else:
    print(err)
else:
  cursor = conn.cursor()

  # Delete a data row in the table
  cursor.execute("DELETE FROM inventory WHERE name=%(param1)s;", {'param1':"orange"})
  print("Deleted",cursor.rowcount,"row(s) of data.")

  # Cleanup
  conn.commit()
  cursor.close()
  conn.close()
  print("Done.")
```

## <a name="next-steps"></a>Pasos siguientes
- [Conectividad cifrada con Seguridad de la capa de transporte (TLS 1.2) en Azure Database for MySQL con la opción Servidor flexible](./how-to-connect-tls-ssl.md).
- Obtenga más información sobre las [redes de Azure Database for MySQL con la opción Servidor flexible](./concepts-networking.md).
- [Creación y administración de reglas de firewall de Azure Database for MySQL con la opción Servidor flexible mediante Azure Portal](./how-to-manage-firewall-portal.md).
- [Creación y administración de la red virtual de Azure Database for MySQL con la opción Servidor flexible mediante Azure Portal](./how-to-manage-virtual-network-portal.md).
