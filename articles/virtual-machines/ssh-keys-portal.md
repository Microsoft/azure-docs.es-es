---
title: Creación de claves SSH en Azure Portal
description: Aprenda a generar y almacenar claves SSH en Azure Portal para conectar las VM de Linux.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: article
ms.date: 08/25/2020
ms.author: cynthn
ms.openlocfilehash: abc9a2ae130d987c90ce87ffaecbf2bb44b06010
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88929452"
---
# <a name="generate-and-store-ssh-keys-in-the-azure-portal"></a>Generar y almacenar claves SSH en Azure Portal

Si usa con frecuencia el portal para implementar VM de Linux, puede hacer que el uso de claves SSH sea más sencillo al crearlas directamente en el portal o al cargarlas desde el equipo.

Puede crear claves SSH cuando cree una VM por primera vez y reutilizarlas para otras VM. Igualmente, también puede crear claves SSH por separado, de modo que tenga un conjunto de claves almacenadas en Azure que se adapten a las necesidades de su organización. 

Si tiene claves existentes y quiere simplificar su uso en el portal, puede cargarlas y almacenarlas en Azure para reutilizarlas.

Para obtener información más detallada sobre la creación y el uso de claves SSH con VM de Linux, consulte [Uso de claves SSH para conectarse a VM Linux](./linux/ssh-from-windows.md).

## <a name="generate-new-keys"></a>Generar claves nuevas

1. Abra [Azure Portal](https://portal.azure.com).

1. En la parte superior de la página, escriba *SSH* para realizar la búsqueda. En **Marketplace**, seleccione **Claves SSH**.

1. En la página de **claves SSH**, seleccione **Crear**.

   :::image type="content" source="./media/ssh-keys/portal-sshkey.png" alt-text="Creación de un nuevo grupo de recursos y generación de un par de claves SSH":::

1. En el **grupo de recursos** seleccione **Crear nuevo** para crear un nuevo grupo de recursos para almacenar las claves. Escriba un nombre para el grupo de recursos y seleccione **Aceptar**.

1. En la **región** seleccione una región para almacenar las claves. Puede usar las claves en cualquier región, es decir, solo la región donde se almacenarán.

1. Escriba un nombre para la clave en el **nombre del par de claves**.

1. En el **origen de la clave pública SSH**, seleccione **Generar origen de clave pública**. 

1. Cuando haya terminado, seleccione **Revisar y crear**.

1. Una vez que pasa la validación, seleccione **Crear**.

1. A continuación, aparecerá una ventana emergente, y deberá seleccionar **Descargar clave privada y crear recurso**. Esta opción descargará la clave SSH como archivo .pem.

   :::image type="content" source="./media/ssh-keys/download-key.png" alt-text="Descarga de la clave pública como un archivo .pem":::

1. Una vez descargado el archivo. pem, es posible que quiera moverlo a algún lugar del equipo desde el cual le sea fácil apuntar desde el cliente SSH.


## <a name="connect-to-the-vm"></a>Conexión a la máquina virtual

En el equipo local, abra un símbolo del sistema de PowerShell y escriba:

```powershell
ssh -i <path to the .pem file> username@<ipaddress of the VM>
```

Por ejemplo, escriba: `ssh -i /Downloads/mySSHKey.pem azureuser@123.45.67.890`


## <a name="upload-an-ssh-key"></a>Carga de una clave SSH

También puede cargar una clave SSH pública para almacenarla en Azure. Para obtener información sobre cómo crear un par de claves SSH, vea [Uso de claves SSH para conectarse a VM Linux](./linux/ssh-from-windows.md).

1. Abra [Azure Portal](https://portal.azure.com).

1. En la parte superior de la página, escriba *SSH* para realizar la búsqueda. En **Marketplace*, seleccione **Claves SSH**.

1. En la página de **claves SSH**, seleccione **Crear**.

   :::image type="content" source="./media/ssh-keys/upload.png" alt-text="Carga de una clave pública SSH para almacenarla en Azure":::

1. En el **grupo de recursos** seleccione **Crear nuevo** para crear un nuevo grupo de recursos para almacenar las claves. Escriba un nombre para el grupo de recursos y seleccione **Aceptar**.

1. En la **región** seleccione una región para almacenar las claves. Puede usar las claves en cualquier región, es decir, solo la región donde se almacenarán.

1. Escriba un nombre para la clave en el **nombre del par de claves**.

1. En el **origen de la clave pública SSH**, seleccione **Cargar clave pública existente**. 

1. Pegue el contenido completo de la clave pública en la **clave de carga** y, a continuación, seleccione **Revisar y crear**.

1. Una vez completada la validación, seleccione **Crear**. 

Una vez que se ha cargado la clave, puede elegir usarla cuando cree una VM.

## <a name="list-keys"></a>Enumeración de claves

Las claves SSH creadas en el portal se almacenan como recursos, por lo que puede filtrar la vista de recursos para verlas todas.

1. En el portal, seleccione **Todos los recursos**.
1. En los filtros, seleccione **Tipo**, anule la selección de la opción **Seleccionar todo** para borrar la lista.
1. Escriba **SSH** en el filtro y seleccione **Clave SSH**.

   :::image type="content" source="./media/ssh-keys/filter.png" alt-text="Captura de pantalla sobre cómo filtrar la lista para ver todas las claves SSH.":::

## <a name="get-the-public-key"></a>Obtención de la clave pública

Si necesita la clave pública, puede copiarla fácilmente desde la página del portal de la clave. Solo tiene que mostrar las claves (mediante el proceso de la última sección) y, a continuación, seleccionar una clave en la lista. Se abrirá la página de la clave y podrá hacer clic en el icono **Copiar al portapapeles** junto a la clave para copiarla.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el uso de claves SSH con VM de Azure, vea [Uso de claves SSH para conectarse a VM Linux](./linux/ssh-from-windows.md).
