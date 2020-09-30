---
title: 'Inicio rápido: Creación de un servidor mediante la CLI de Azure en Azure Database for MySQL con la opción Servidor flexible'
description: En este inicio rápido se describe cómo usar la CLI de Azure para crear una instancia de Azure Database for MySQL con la opción Servidor flexible en un grupo de recursos de Azure.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 9/21/2020
ms.custom: mvc
ms.openlocfilehash: bae6e9f04eced02130ae628d5308a87a1baaa8fa
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946202"
---
# <a name="quickstart-create-an-azure-database-for-mysql-flexible-server-using-azure-cli"></a>Inicio rápido: Creación de una instancia de Azure Database for MySQL con la opción Servidor flexible mediante la CLI de Azure

En este inicio rápido se muestra cómo usar los comandos de la [CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) en [Azure Cloud Shell](https://shell.azure.com) para crear una instancia de Azure Database for MySQL con la opción Servidor flexible en cinco minutos. Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

> [!IMPORTANT] 
> Actualmente, Azure Database for MySQL con la opción Servidor flexible está en versión preliminar pública.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

[Azure Cloud Shell](../../cloud-shell/overview.md) es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta.

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede abrir Cloud Shell en una pestaña independiente acudiendo a [https://shell.azure.com/bash](https://shell.azure.com/bash). Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, después, seleccione **Entrar** para ejecutarlos.

Si prefiere instalar y usar la CLI de forma local, en este inicio rápido se requiere la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="prerequisites"></a>Requisitos previos

Será preciso que inicie sesión en su cuenta mediante el comando [az login](https://docs.microsoft.com/cli/azure/reference-index?view=azure-cli-latest#az-login). Tenga en cuenta la propiedad **id**, que hace referencia al **identificador de suscripción** para su cuenta de Azure.

```azurecli-interactive
az login
```

Seleccione la suscripción específica en su cuenta mediante el comando [az account set](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az-account-set). Anote el valor de **id** de la salida de **az login** para usarlo como valor del argumento **subscription** del comando. Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Para obtener todas las suscripciones, use [az account list](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az-account-list).

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-a-flexible-server"></a>Creación de un servidor flexible

Cree un [grupo de recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/overview) mediante el comando `az group create` y, después, cree un servidor flexible de MySQL en dicho grupo de recursos. Debe proporcionar un nombre único. En el ejemplo siguiente, se crea un grupo de recursos denominado `myresourcegroup` en la ubicación `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

Cree un servidor flexible con el comando `az mysql flexible-server create`. Un servidor puede contener varias bases de datos. El siguiente comando crea un servidor con los valores predeterminados de servicio y los valores del [contexto local](https://docs.microsoft.com/cli/azure/local-context?view=azure-cli-latest) de la CLI de Azure: 

```azurecli
az mysql flexible-server create
```

El servidor creado tiene los siguientes atributos: 
- Nombre del servidor generado automáticamente, nombre de usuario de administrador, contraseña de administrador, nombre del grupo de recursos (si aún no se ha especificado en el contexto local) y de la misma ubicación que el grupo de recursos. 
- Valores predeterminados de servicio para las configuraciones de servidor restantes: nivel de proceso (Flexible), tamaño de proceso/SKU (B1MS), período de retención de copia de seguridad (7 días) y versión de MySQL (5.7)
- El método de conectividad predeterminado es el acceso privado (integración con red virtual) con una red virtual y una subred generadas automáticamente.

> [!NOTE] 
> El método de conectividad no se puede cambiar después de crear el servidor. Por ejemplo, si seleccionó *Acceso privado (integración con red virtual)* durante la creación, no podrá cambiar a *Acceso público (direcciones IP permitidas)* después de la creación. Se recomienda encarecidamente crear un servidor con acceso privado para acceder de forma segura a su servidor mediante la integración con la red virtual. Obtenga más información sobre el acceso privado en el [artículo de conceptos](./concepts-networking.md).

Si quiere cambiar los valores predeterminados, consulte la documentación de referencia de la CLI de Azure <!--FIXME --> para obtener la lista completa de parámetros configurables de la CLI. 

> [!NOTE]
> Las conexiones a Azure Database for MySQL se comunican a través del puerto 3306. Si intenta conectarse desde una red corporativa, es posible que no se permita el tráfico saliente a través del puerto 3306. En ese caso no podrá conectarse al servidor, salvo que el departamento de TI abra el puerto 3306.

## <a name="get-the-connection-information"></a>Obtención de la información de conexión

Para conectarse al servidor, debe proporcionar las credenciales de acceso y la información del host.

```azurecli-interactive
az mysql flexible-server show --resource-group myresourcegroup --name mydemoserver
```

El resultado está en formato JSON. Tome nota de los valores de **fullyQualifiedDomainName** y **administratorLogin**.

<!--FIXME-->
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/flexibleServers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 1,
    "name": "Standard_B1ms",
    "size": null,
    "tier": "Burstable"
  },
  "publicAccess": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 5120
  },
  "tags": null,
  "type": "Microsoft.DBforMySQL/flexibleServers",
  "userVisibleState": "Ready",
  "version": "5.7"
}
```

## <a name="connect-using-mysql-command-line-client"></a>Conexión mediante el cliente de línea de comandos mysql

Dado que el servidor flexible se creó con la opción *Acceso privado (integración con red virtual)* , deberá conectarse a su servidor desde un recurso de la misma red virtual que el servidor. Puede crear una máquina virtual y agregarla a la red virtual creada. 

Una vez creada la máquina virtual, puede aplicar SSH en la máquina e instalar la herramienta de línea de comandos de cliente conocida **[mysql.exe](https://dev.mysql.com/downloads/)** .

Con mysql.exe, conéctese con el siguiente comando. Reemplace los valores por el nombre real del servidor y la contraseña. 

```bash
 mysql -h mydemoserver.mysql.database.azure.com -u mydemouser -p
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no necesita estos recursos para otra guía de inicio rápido o tutorial, puede eliminarlos con el siguiente comando:

```azurecli-interactive
az group delete --name myresourcegroup
```

Si solo quiere eliminar el servidor recién creado, puede ejecutar el comando `az mysql server delete`.

```azurecli-interactive
az mysql flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
>[Compilación de una aplicación web PHP (Laravel) con MySQL](tutorial-php-database-app.md)