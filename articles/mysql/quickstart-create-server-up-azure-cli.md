---
title: 'Inicio rápido: Creación de una instancia de Azure Database for MySQL mediante az mysql up'
description: Inicio rápido para crear un servidor Azure Database for MySQL mediante el comando up de la CLI (interfaz de la línea de comandos) de Azure.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 3/18/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 4f2c0a51f7c8d2441c9ddff2520dae2f5cc13565
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495357"
---
# <a name="quickstart-create-an-azure-database-for-mysql-using-a-simple-azure-cli-command---az-mysql-up-preview"></a>Inicio rápido: Creación de una instancia de Azure Database for MySQL mediante un sencillo comando de la CLI de Azure: az mysql up (versión preliminar)

> [!IMPORTANT]
> El comando [az mysql up](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-up) de la CLI de Azure se encuentra en versión preliminar.

Azure Database for MySQL es un servicio administrado que le permite ejecutar, administrar y escalar bases de datos de MySQL de alta disponibilidad en la nube. La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. En este inicio rápido se muestra cómo usar el comando [az mysql up](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-up) para crear un servidor Azure Database for MySQL mediante la CLI de Azure. Además de crear el servidor, el comando `az mysql up` crea una base de datos de ejemplo, un usuario raíz en la base de datos, abre el firewall para servicios de Azure y crea reglas de firewall predeterminadas para el equipo cliente. Esto ayuda a acelerar el proceso de desarrollo.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

En este artículo es necesario que ejecute la versión 2.0 de la CLI de Azure, o cualquier versión posterior, de forma local. Para ver la versión instalada, ejecute el comando `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Deberá iniciar sesión en su cuenta mediante el comando [az login](/cli/azure/authenticate-azure-cli?view=interactive-log-in). Anote la propiedad **id** de la salida del comando para el nombre de la suscripción correspondiente.

```azurecli
az login
```

Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione el identificador de suscripción específico en su cuenta mediante el comando [az account set](/cli/azure/account). Sustituya la propiedad **subscription ID** de la salida de **az login** de su suscripción en el marcador de posición de identificador de suscripción.

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-an-azure-database-for-mysql-server"></a>Creación de un servidor de Azure Database for MySQL

Para usar los comandos, instale la extensión [db-up](/cli/azure/ext/db-up). Si se devuelve un error, asegúrese de que ha instalado la versión más reciente de la CLI de Azure. Consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

```azurecli
az extension add --name db-up
```

Cree un servidor Azure Database for MySQL con el comando siguiente:

```azurecli
az mysql up
```

El servidor se crea con los siguientes valores predeterminados (a menos que los invalide manualmente):

**Configuración** | **Valor predeterminado** | **Descripción**
---|---|---
server-name | Generado por el sistema | Un nombre único que identifica al servidor de Azure Database for MySQL.
resource-group | Generado por el sistema | Un nuevo grupo de recursos de Azure.
sku-name | GP_Gen5_2 | El nombre de la SKU. Sigue la convención {plan de tarifa}\_{generación de procesos}\_{núcleos virtuales} en forma abreviada. El valor predeterminado es un servidor de uso general Gen5 con 2 núcleos virtuales. Consulte nuestra [página de precios](https://azure.microsoft.com/pricing/details/mysql/) para más información acerca de los niveles.
backup-retention | 7 | Cuánto tiempo deben conservarse las copias de seguridad. La unidad es días.
geo-redundant-backup | Disabled | Si se deben habilitar las copias de seguridad con redundancia geográfica en este servidor o no.
ubicación | westus2 | La ubicación de Azure para el servidor.
ssl-enforcement | habilitado | Si SSL debe habilitarse o no en este servidor.
storage-size | 5120 | La capacidad de almacenamiento del servidor (la unidad es megabytes).
version | 5.7 | La versión principal de MySQL.
admin-user | Generado por el sistema | El nombre del usuario del inicio de sesión del administrador.
admin-password | Generado por el sistema | La contraseña del usuario administrador.

> [!NOTE]
> Para más información acerca del comando `az mysql up` y sus parámetros adicionales, consulte la [documentación de la CLI de Azure](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-up).

Una vez que se crea el servidor, incluye la siguiente configuración:

- Se crea una regla de firewall denominada "devbox". La CLI de Azure intenta detectar la dirección IP de la máquina desde la que se ejecuta el comando `az mysql up` e incluye esa dirección IP en la lista de permitidos.
- "Permitir el acceso a servicios de Azure" está activado. Este valor configura el firewall del servidor para que acepte conexiones de todos los recursos de Azure, incluidos aquellos que no están en la suscripción.
- El parámetro `wait_timeout` se establece en 8 horas.
- Se crea una base de datos vacía denominada "sampledb"
- Se crea un usuario denominado "root" con privilegios para "sampledb"

> [!NOTE]
> Azure Database for MySQL se comunica a través del puerto 3306. Al conectarse desde una red corporativa, es posible que el firewall de la red no permita el tráfico saliente a través de dicho puerto. Indique al departamento de TI que abran el puerto 3306 para conectarse al servidor.

## <a name="get-the-connection-information"></a>Obtención de la información de conexión

Una vez finalizado el comando `az mysql up`, se le presenta una lista de cadenas de conexión de lenguajes de programación conocidos. Estas cadenas de conexión están preconfiguradas con atributos específicos del servidor Azure Database for MySQL recién creado.

Puede usar el comando [az mysql show-connection-string](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-show-connection-string) para mostrar de nuevo estas cadenas de conexión.

## <a name="clean-up-resources"></a>Limpieza de recursos

Limpie todos los recursos que creó en el inicio rápido mediante el siguiente comando. Este comando elimina el servidor Azure Database for MySQL y el grupo de recursos.

```azurecli
az mysql down --delete-group
```

Si solo quiere eliminar el servidor recién creado, puede ejecutar el comando [az mysql down](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-down).

```azurecli
az mysql down
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Diseño de una base de datos MySQL con la CLI de Azure](./tutorial-design-database-using-cli.md)
