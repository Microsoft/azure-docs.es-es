---
title: 'Administración de redes virtuales de Azure Database for MySQL mediante la CLI de Azure: servidor flexible'
description: 'Creación y administración de redes virtuales de Azure Database for MySQL: servidor flexible mediante la CLI de Azure'
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: 4e7d75bc1a2acd805e573d4c6ad80f9892045551
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931903"
---
# <a name="create-and-manage-virtual-networks-for-azure-database-for-mysql---flexible-server-using-the-azure-cli"></a>Creación y administración de redes virtuales de Azure Database for MySQL: servidor flexible mediante la CLI de Azure

> [!IMPORTANT]
> Azure Database for MySQL: servidor flexible está actualmente en versión preliminar pública.

Se admiten dos métodos de conectividad de red mutuamente excluyentes para conectarse a su servidor flexible de Azure Database for MySQL. Las dos opciones son las siguientes:

- Acceso público (direcciones IP permitidas)
- Acceso privado (integración con red virtual)

En este artículo nos vamos a centrar en la creación de un servidor de MySQL con **acceso privado (integración con red virtual)** mediante la CLI de Azure. Con *acceso privado (Integración con red virtual)* , puede implementar su servidor flexible en su propio [Azure Virtual Network](../../virtual-network/virtual-networks-overview.md). Las redes de Azure Virtual Network proporcionan una comunicación de red privada y segura. En el acceso privado, las conexiones con el servidor de MySQL están restringidas a dentro de la red virtual. Para obtener más información al respecto, consulte [Acceso privado (Integración con red virtual)](./concepts-networking.md#private-access-vnet-integration).

En Azure Database for MySQL: servidor flexible, solo puede implementar el servidor en una red virtual y una subred durante la creación del servidor. Una vez implementado el servidor flexible en una red virtual y una subred, no se puede trasladar a otra red virtual, a una subred o a *acceso público (direcciones IP permitidas)* .

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

[Azure Cloud Shell](../../cloud-shell/overview.md) es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta.

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede abrir Cloud Shell en una pestaña independiente acudiendo a [https://shell.azure.com/bash](https://shell.azure.com/bash). Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, después, seleccione **Entrar** para ejecutarlos.

Si prefiere instalar y usar la CLI de forma local, en este inicio rápido se requiere la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="prerequisites"></a>Requisitos previos

Deberá iniciar sesión en la cuenta con el comando [az login](https://docs.microsoft.com/cli/azure/reference-index?view=azure-cli-latest#az-login). Tenga en cuenta la propiedad **id**, que hace referencia al **id. de suscripción** de su cuenta de Azure.

```azurecli-interactive
az login
```

Seleccione la suscripción específica en su cuenta mediante el comando [az account set](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az-account-set). Anote el valor de **id** de la salida de **az login** para usarlo como valor del argumento **subscription** del comando. Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Para obtener todas las suscripciones, use [az account list](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az-account-list).

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-azure-database-for-mysql-flexible-server-using-cli"></a>Creación de Azure Database for MySQL: servidor flexible mediante la CLI
Puede usar el comando `az mysql flexible-server` para crear el servidor flexible con *acceso privado (Integración con red virtual)* . Este comando usa acceso privado (Integración con red virtual) como método de conectividad predeterminado. Si no se proporciona ninguna, se creará una red virtual y una subred. También puede proporcionar la red virtual y la subred ya existentes con el id. de subred. <!-- You can provide the **vnet**,**subnet**,**vnet-address-prefix** or**subnet-address-prefix** to customize the virtual network and subnet.--> Existen varias opciones para crear un servidor flexible mediante la CLI, tal y como se muestra en los ejemplos siguientes.

>[!Important]
> El uso de este comando delega la subred a **Microsoft.DBforMySQL/flexibleServers**. Esta delegación significa que solo los servidores flexibles de Azure Database for MySQL pueden usar esa subred. No puede haber otros tipos de recursos de Azure en la subred delegada.
>

Consulte la documentación de referencia de la CLI de Azure <!--FIXME --> para obtener la lista completa de parámetros configurables de la CLI. Por ejemplo, en los siguientes comandos puede especificar opcionalmente el grupo de recursos.

- Creación de un servidor flexible mediante la red virtual predeterminada y la subred con prefijo de dirección predeterminado
    ```azurecli-interactive
    az mysql flexible-server create
    ```
<!--- Create a flexible server using already existing virtual network and subnet
    ```azurecli-interactive
    az mysql flexible-server create --vnet myVnet --subnet mySubnet
    ```-->
- Cree un servidor flexible mediante la red virtual existente, la subred y el id. de subred. La subred proporcionada no debe tener ningún otro recurso implementado y se delega a **Microsoft.DBforMySQL/flexibleServers**, si aún no se ha delegado.
    ```azurecli-interactive
    az mysql flexible-server create --subnet /subscriptions/{SubID}/resourceGroups/{ResourceGroup}/providers/Microsoft.Network/virtualNetworks/{VNetName}/subnets/{SubnetName}
    ```
    > [!Note]
    > La red virtual y la subred deben estar en la misma región y suscripción que el servidor flexible.
<!--
- Create a flexible server using new virtual network, subnet with non-default address prefix
    ```azurecli-interactive
    az mysql flexible-server create --vnet myVnet --vnet-address-prefix 10.0.0.0/24 --subnet mySubnet --subnet-address-prefix 10.0.0.0/24
    ```-->
Consulte la documentación de referencia de la CLI de Azure <!--FIXME --> para obtener la lista completa de parámetros configurables de la CLI.


## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre las [redes de Azure Database for MySQL: servidor flexible](./concepts-networking.md).
- [Cree y administre redes virtuales de Azure Database for MySQL: servidor flexible mediante Azure Portal](./how-to-manage-virtual-network-portal.md).
- Obtenga más información sobre las [redes virtuales de Azure Database for MySQL: servidor flexible](./concepts-networking.md#private-access-vnet-integration).
