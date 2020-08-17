---
title: 'Inicio rápido: Creación de servidor: az postgres up: Azure Database for PostgreSQL (servidor único)'
description: 'Guía de inicio rápido para crear una instancia de Azure Database for PostgreSQL: servidor único con el comando up de la CLI (interfaz de la línea de comandos) de Azure.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 05/06/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1a2b7ceea74091931c92d28f8fa47b9046b545fc
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496583"
---
# <a name="quickstart-use-an-azure-cli-command-az-postgres-up-preview-to-create-an-azure-database-for-postgresql---single-server"></a>Inicio rápido: Usar un comando de la CLI de Azure, az postgres up (versión preliminar), para crear una instancia de Azure Database for PostgreSQL: servidor único

> [!IMPORTANT]
> El comando [az postgres up](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up) de la CLI de Azure se encuentra en versión preliminar.

Azure Database for PostgreSQL es un servicio administrado que le permite ejecutar, administrar y escalar bases de datos de PostgreSQL de alta disponibilidad en la nube. La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. En este inicio rápido se muestra cómo usar el comando [az postgres up](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up) para crear un servidor Azure Database for PostgreSQL mediante la CLI de Azure. Además de crear el servidor, el comando `az postgres up` crea una base de datos de ejemplo, un usuario raíz en la base de datos, abre el firewall para servicios de Azure y crea reglas de firewall predeterminadas para el equipo cliente. Estos valores predeterminados ayudan a acelerar el proceso de desarrollo.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

En este artículo es necesario que ejecute la versión 2.0 de la CLI de Azure, o cualquier versión posterior, de forma local. Para ver la versión instalada, ejecute el comando `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Deberá iniciar sesión en la cuenta con el comando [az login](/cli/azure/authenticate-azure-cli?view=interactive-log-in). Anote la propiedad **ID** de la salida del comando para el nombre de la suscripción correspondiente.

```azurecli
az login
```

Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione el identificador de suscripción específico en su cuenta mediante el comando [az account set](/cli/azure/account). Sustituya la propiedad **subscription ID** de la salida de **az login** de su suscripción en el marcador de posición de identificador de suscripción.

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-an-azure-database-for-postgresql-server"></a>Creación de un servidor de Azure Database for PostgreSQL

Para usar los comandos, instale la extensión [db-up](/cli/azure/ext/db-up). Si se devuelve un error, asegúrese de que ha instalado la versión más reciente de la CLI de Azure. Consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

```azurecli
az extension add --name db-up
```

Cree un servidor Azure Database for PostgreSQL mediante los pasos siguientes:

```azurecli
az postgres up
```

El servidor se crea con los siguientes valores predeterminados (a menos que los invalide manualmente):

**Configuración** | **Valor predeterminado** | **Descripción**
---|---|---
server-name | Generado por el sistema | Un nombre único que identifique al servidor de Azure Database for PostgreSQL.
resource-group | Generado por el sistema | Un nuevo grupo de recursos de Azure.
sku-name | GP_Gen5_2 | El nombre de la SKU. Sigue la convención {plan de tarifa}\_{generación de procesos}\_{núcleos virtuales} en forma abreviada. El valor predeterminado es un servidor de uso general Gen5 con 2 núcleos virtuales. Consulte nuestra [página de precios](https://azure.microsoft.com/pricing/details/postgresql/) para más información acerca de los niveles.
backup-retention | 7 | El tiempo durante el cual se conserva la copia de seguridad. La unidad es días.
geo-redundant-backup | Disabled | Si se deben habilitar las copias de seguridad con redundancia geográfica en este servidor o no.
ubicación | westus2 | La ubicación de Azure para el servidor.
ssl-enforcement | Disabled | Si TLS/SSL debe habilitarse o no en este servidor.
storage-size | 5120 | La capacidad de almacenamiento del servidor (la unidad es megabytes).
version | 10 | La versión principal de PostgreSQL.
admin-user | Generado por el sistema | El nombre de usuario del administrador.
admin-password | Generado por el sistema | La contraseña del usuario administrador.

> [!NOTE]
> Para más información acerca del comando `az postgres up` y sus parámetros adicionales, consulte la [documentación de la CLI de Azure](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up).

Una vez que se crea el servidor, incluye la siguiente configuración:

- Se crea una regla de firewall denominada "devbox". La CLI de Azure intenta detectar la dirección IP de la máquina desde la que se ejecuta el comando `az postgres up` e incluye esa dirección IP en la lista de permitidos.
- "Permitir el acceso a servicios de Azure" está activado. Este valor configura el firewall del servidor para que acepte conexiones de todos los recursos de Azure, incluidos aquellos que no están en la suscripción.
- Se crea una base de datos vacía denominada "sampledb"
- Se crea un usuario denominado "root" con privilegios para "sampledb"

> [!NOTE]
> Azure Database for PostgreSQL se comunica a través del puerto 5432. Al conectarse desde una red corporativa, es posible que el firewall de la red no permita el tráfico saliente a través del puerto 5432. Indique al departamento de TI que abran el puerto 5432 para conectarse al servidor.

## <a name="get-the-connection-information"></a>Obtención de la información de conexión

Una vez finalizado el comando `az postgres up`, se le presenta una lista de cadenas de conexión de lenguajes de programación conocidos. Estas cadenas de conexión están preconfiguradas con atributos específicos del servidor Azure Database for PostgreSQL recién creado.

Puede usar el comando [az postgres show-connection-string](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-show-connection-string) para mostrar de nuevo estas cadenas de conexión.

## <a name="clean-up-resources"></a>Limpieza de recursos

Limpie todos los recursos que creó en el inicio rápido mediante el siguiente comando. Este comando elimina el servidor Azure Database for PostgreSQL y el grupo de recursos.

```azurecli
az postgres down --delete-group
```

Si solo quiere eliminar el servidor recién creado, puede ejecutar el comando [az postgres down](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-down).

```azurecli
az postgres down
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Migración de una base de datos mediante exportación e importación](./howto-migrate-using-export-and-import.md)
