---
title: 'Tutorial: Configuración de bases de datos en Azure Database for MySQL con Ansible | Microsoft Docs'
description: Obtenga información acerca de cómo usar Ansible para crear y configurar un servidor de Azure Database for MySQL
keywords: ansible, azure, devops, bash, cuaderno de estrategias, mysql, base de datos
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 1170ae9d609a07dbdaebf50e145de65faefa60ec
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230918"
---
# <a name="tutorial-configure-databases-in-azure-database-for-mysql-using-ansible"></a>Tutorial: Configuración de bases de datos en Azure Database for MySQL con Ansible

[!INCLUDE [ansible-27-note.md](../../includes/ansible-27-note.md)]

[Azure Database for MySQL](/azure/mysql/overview) es un servicio de base de datos relacional basado en MySQL Community Edition. Azure Database for MySQL le permite administrar bases de datos MySQL en sus aplicaciones web.

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Crear un servidor MySQL
> * Crear una base de datos MySQL
> * Configurar una regla de firewall para que una aplicación externa se pueda conectar al servidor
> * Conectarse al servidor MySQL desde Azure Cloud Shell
> * Consultar los servidores de MySQL disponibles
> * Enumerar todas las bases de datos de los servidores conectados

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

El código del cuaderno de estrategias de esta sección crea un grupo de recursos de Azure. Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure.  

Guarde el siguiente cuaderno de estrategias como `rg.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    location: eastus 
  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
```

Antes de ejecutar el cuaderno de estrategias, consulte las notas siguientes:

* Se crea un grupo de recursos llamado `myResourceGroup`.
* Este grupo de recursos se crea en la ubicación `eastus`:

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook rg.yml
```

## <a name="create-a-mysql-server-and-database"></a>Creación de un servidor de MySQL y una base de datos

El código del cuaderno de estrategias de esta sección crea un servidor MySQL y una instancia de Azure Database for MySQL. El nuevo servidor MySQL es un servidor Gen 5 de uso general con un núcleo virtual y se llama `mysqlserveransible`. La instancia de la base de datos se denomina `mysqldbansible`.

Para obtener más información sobre los planes de tarifa, consulte [Planes de tarifa de Azure Database for MySQL](/azure/mysql/concepts-pricing-tiers). 

Guarde el siguiente cuaderno de estrategias como `mysql_create.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    location: eastus
    mysqlserver_name: mysqlserveransible
    mysqldb_name: mysqldbansible
    admin_username: mysqladmin
    admin_password: <server_admin_password> 
  tasks:
    - name: Create MySQL Server
      azure_rm_mysqlserver:
        resource_group: "{{ resource_group }}"
        name: "{{ mysqlserver_name }}"
        sku:
          name: B_Gen5_1
          tier: Basic
        location: "{{ location }}"
        version: 5.6
        enforce_ssl: True
        admin_username: "{{ admin_username }}"
        admin_password: "{{ admin_password }}"
        storage_mb: 51200
    - name: Create instance of MySQL Database
      azure_rm_mysqldatabase:
        resource_group: "{{ resource_group }}"
        server_name: "{{ mysqlserver_name }}"
        name: "{{ mysqldb_name }}"
```

Antes de ejecutar el cuaderno de estrategias, consulte las notas siguientes:

* En la sección `vars`, el valor de `mysqlserver_name` debe ser único.
* En la sección `vars`, reemplace `<server_admin_password>` por una contraseña.

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook mysql_create.yml
```

## <a name="configure-a-firewall-rule"></a>Configuración de una regla de firewall

Una regla de firewall de nivel de servidor permite que una aplicación externa se conecte al servidor a través del firewall del servicio MySQL de Azure. Ejemplos de aplicaciones externas son la herramienta de línea de comandos `mysql` y MySQL Workbench.

El código del cuaderno de estrategias de esta sección crea una regla de firewall denominada `extenalaccess` que permite las conexiones desde cualquier dirección IP externa. 

Guarde el siguiente cuaderno de estrategias como `mysql_firewall.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    mysqlserver_name: mysqlserveransible
  tasks:
  - name: Open firewall to access MySQL Server from outside
    azure_rm_resource:
      api_version: '2017-12-01'
      resource_group: "{{ resource_group }}"
      provider: dbformysql
      resource_type: servers
      resource_name: "{{ mysqlserver_name }}"
      subresource:
        - type: firewallrules
          name: externalaccess
      body:
        properties:
          startIpAddress: "0.0.0.0"
          endIpAddress: "255.255.255.255"
```

Antes de ejecutar el cuaderno de estrategias, consulte las notas siguientes:

* En la sección de variables, reemplace `startIpAddress` y `endIpAddress`. Utilice el intervalo de direcciones IP que se correspondan con el intervalo desde el que se está conectando.
* Las conexiones a Azure Database for MySQL se comunican a través del puerto 3306. Si intenta conectarse desde una red corporativa, es posible que no se permita el tráfico saliente a través del puerto 3306. En ese caso no podrá conectarse al servidor, salvo que el departamento de TI abra el puerto 3306.
* El cuaderno de estrategias usa el módulo `azure_rm_resource`, que permite el uso directo de la API REST.

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook mysql_firewall.yml
```

## <a name="connect-to-the-server"></a>Conexión al servidor

En esta sección, usará Azure Cloud Shell para conectarse al servidor que creó anteriormente.

1. Seleccione el botón **Pruébelo** en el código siguiente:

    ```azurecli-interactive
    mysql -h mysqlserveransible.mysql.database.azure.com -u mysqladmin@mysqlserveransible -p
    ```

1. En el símbolo del sistema, escriba el comando siguiente para consultar el estado del servidor:

    ```sql
    mysql> status
    ```
    
    Si todo va bien, verá una salida parecida a los siguientes resultados:
    
    ```
    demo@Azure:~$ mysql -h mysqlserveransible.mysql.database.azure.com -u mysqladmin@mysqlserveransible -p
    Enter password:
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 65233
    Server version: 5.6.39.0 MySQL Community Server (GPL)
    
    Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.
    
    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.
    
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    
    mysql> status
    --------------
    mysql  Ver 14.14 Distrib 5.7.23, for Linux (x86_64) using  EditLine wrapper
    
    Connection id:          65233
    Current database:
    Current user:           mysqladmin@13.76.42.93
    SSL:                    Cipher in use is AES256-SHA
    Current pager:          stdout
    Using outfile:          ''
    Using delimiter:        ;
    Server version:         5.6.39.0 MySQL Community Server (GPL)
    Protocol version:       10
    Connection:             mysqlserveransible.mysql.database.azure.com via TCP/IP
    Server characterset:    latin1
    Db     characterset:    latin1
    Client characterset:    utf8
    Conn.  characterset:    utf8
    TCP port:               3306
    Uptime:                 36 min 21 sec
    
    Threads: 5  Questions: 559  Slow queries: 0  Opens: 96  Flush tables: 3  Open tables: 10  Queries per second avg: 0.256
    --------------
    ```
    
## <a name="query-mysql-servers"></a>Realizar consultas en servidores MySQL

El código del cuaderno de estrategias de esta sección realiza consultas en los servidores MySQL en `myResourceGroup` y enumera las bases de datos de los servidores encontrados.

Guarde el siguiente cuaderno de estrategias como `mysql_query.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    mysqlserver_name: mysqlserveransible
  tasks:
    - name: Query MySQL Servers in current resource group
      azure_rm_mysqlserver_facts:
        resource_group: "{{ resource_group }}"
      register: mysqlserverfacts

    - name: Dump MySQL Server facts
      debug:
        var: mysqlserverfacts

    - name: Query MySQL Databases
      azure_rm_mysqldatabase_facts:
        resource_group: "{{ resource_group }}"
        server_name: "{{ mysqlserver_name }}"
      register: mysqldatabasefacts

    - name: Dump MySQL Database Facts
      debug:
        var: mysqldatabasefacts
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook mysql_query.yml
```

Tras ejecutar el cuaderno de estrategias, debería ver resultados similares a los siguientes:

```json
"servers": [
    {
        "admin_username": "mysqladmin",
        "enforce_ssl": false,
        "fully_qualified_domain_name": "mysqlserveransible.mysql.database.azure.com",
        "id": "/subscriptions/685ba005-af8d-4b04-8f16-a7bf38b2eb5a/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/mysqlserveransible",
        "location": "eastus",
        "name": "mysqlserveransible",
        "resource_group": "myResourceGroup",
        "sku": {
            "capacity": 1,
            "family": "Gen5",
            "name": "B_Gen5_1",
            "tier": "Basic"
        },
        "storage_mb": 5120,
        "user_visible_state": "Ready",
        "version": "5.6"
    }
]
```

También puede ver la siguiente salida para la base de datos de MySQL:

```json
"databases": [
    {
        "charset": "utf8",
        "collation": "utf8_general_ci",
        "name": "information_schema",
        "resource_group": "myResourceGroup",
        "server_name": "mysqlserveransible"
    },
    {
        "charset": "latin1",
        "collation": "latin1_swedish_ci",
        "name": "mysql",
        "resource_group": "myResourceGroup",
        "server_name": "mysqlserveransibler"
    },
    {
        "charset": "latin1",
        "collation": "latin1_swedish_ci",
        "name": "mysqldbansible",
        "resource_group": "myResourceGroup",
        "server_name": "mysqlserveransible"
    },
    {
        "charset": "utf8",
        "collation": "utf8_general_ci",
        "name": "performance_schema",
        "resource_group": "myResourceGroup",
        "server_name": "mysqlserveransible"
    }
]
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los recursos creados en este artículo. 

Guarde el siguiente cuaderno de estrategias como `cleanup.yml`:

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
  tasks:
    - name: Delete a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        state: absent
```

Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

```bash
ansible-playbook cleanup.yml
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Ansible en Azure](/azure/ansible/)