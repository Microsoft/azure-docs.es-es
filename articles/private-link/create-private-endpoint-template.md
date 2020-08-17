---
title: Creación de un punto de conexión privado en Azure Private Link
description: En este inicio rápido, se usa una plantilla de Azure Resource Manager para crear un punto de conexión privado.
services: private-link
author: mblanco77
ms.service: private-link
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 05/26/2020
ms.author: allensu
ms.openlocfilehash: 9fde76b86b290e1271f408cb7810e549dd9502a8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87071489"
---
# <a name="quickstart-create-a-private-endpoint-by-using-an-arm-template"></a>Inicio rápido: Creación de un punto de conexión privado mediante una plantilla de Resource Manager

En este inicio rápido, se usa una plantilla de Azure Resource Manager para crear un punto de conexión privado.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

También se puede completar este inicio rápido mediante [Azure Portal](create-private-endpoint-portal.md), [Azure PowerShell](create-private-endpoint-powershell.md) o la [CLI de Azure](create-private-endpoint-cli.md).

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-private-endpoint-sql%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Necesita una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Revisión de la plantilla

Esta plantilla crea un punto de conexión privado para una instancia de Azure SQL Database.

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-private-endpoint-sql/).

:::code language="json" source="~/quickstart-templates/101-private-endpoint-sql/azuredeploy.json" range="001-295" highlight="131-156":::

En la plantilla se definen varios recursos de Azure:

- [**Microsoft.Sql/servers**](/azure/templates/microsoft.sql/servers): la instancia de SQL Database con la base de datos de ejemplo.
- [**Microsoft.Sql/servers/databases**](/azure/templates/microsoft.sql/servers/databases): la base de datos de ejemplo.
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks): la red virtual donde se implementa el punto de conexión privado.
- [**Microsoft.Network/privateEndpoints**](/azure/templates/microsoft.network/privateendpoints): el punto de conexión privado para acceder a la instancia de SQL Database.
- [**Microsoft.Network/privateDnsZones**](/azure/templates/microsoft.network/privatednszones): la zona usada para resolver la dirección IP del punto de conexión privado.
- [**Microsoft.Network/privateDnsZones/virtualNetworkLinks**](/azure/templates/microsoft.network/privatednszones/virtualnetworklinks)
- [**Microsoft.Network/privateEndpoints/privateDnsZoneGroups**](/azure/templates/microsoft.network/privateendpoints/privateDnsZoneGroups): el grupo de zonas usado para asociar el punto de conexión privado con una zona DNS privada.
- [**Microsoft.Network/publicIpAddresses**](/azure/templates/microsoft.network/publicIpAddresses): la dirección IP pública para acceder a la máquina virtual.
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces): la interfaz de red de la máquina virtual.
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines): la máquina virtual usada para probar la conexión privada con un punto de conexión privado a la instancia de SQL Database.

## <a name="deploy-the-template"></a>Implementación de la plantilla

Aquí se muestra cómo implementar la plantilla de ARM en Azure:

1. Seleccione **Implementar en Azure** para iniciar sesión en Azure y abrir la plantilla. La plantilla crea el punto de conexión privado, la instancia de SQL Database, la infraestructura de red y las máquinas virtuales que se van a validar.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-private-endpoint-sql%2Fazuredeploy.json)

2. Elija su grupo de recursos o cree uno.
3. Escriba las credenciales de inicio de sesión y contraseña del administrador de SQL.
4. Escriba el nombre de usuario y la contraseña del administrador de la máquina virtual.
5. Lea la declaración de los términos y condiciones. Si está de acuerdo con ellos, seleccione **Acepto los términos y condiciones indicados anteriormente** > **Comprar**. La implementación puede tardar 20 minutos o más en completarse.

## <a name="validate-the-deployment"></a>Validación de la implementación

> [!NOTE]
> La plantilla de Resource Manager genera un nombre único para el recurso de la máquina virtual myVm<b>{uniqueid}</b> y para el recurso sqlserver<b>{uniqueid}</b> de SQL Database. Sustituya el valor generado para **{uniqueid}** .

### <a name="connect-to-a-vm-from-the-internet"></a>Conexión a una máquina virtual desde Internet

Conéctese a la VM _myVm{uniqueid}_ desde Internet de la siguiente manera:

1. En la barra de búsqueda del portal, escriba _myVm{uniqueid}_ .

2. Seleccione **Conectar**. Se abre **Connect to virtual machine** (Conectarse a una máquina virtual).

3. Seleccione **Descargar archivo RDP**. Azure crea un archivo de Protocolo de Escritorio remoto ( _.rdp_) y lo descarga en su equipo.

4. Abra el archivo .rdp descargado.

   a. Cuando se le pida, seleccione **Conectar**.

   b. Escriba el nombre de usuario y la contraseña que especificó cuando creó la máquina virtual.

      > [!NOTE]
      > Puede que deba seleccionar **Más opciones** > **Usar otra cuenta** para especificar las credenciales que escribió al crear la máquina virtual.

5. Seleccione **Aceptar**.

6. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe una advertencia de certificado, seleccione **Sí** o **Continuar**.

7. Cuando aparezca el escritorio de la máquina virtual, minimícelo para volver al escritorio local.

### <a name="access-the-sql-database-server-privately-from-the-vm"></a>Acceso al servidor de SQL Database de forma privada desde la máquina virtual

A continuación, se muestra cómo conectarse al servidor de SQL Database desde la máquina virtual mediante el punto de conexión privado.

1.  En el Escritorio remoto de _myVm{uniqueid}_ , abra PowerShell.
2.  Escriba lo siguiente: nslookup sqlserver{uniqueid}.database.windows.net. 
    Recibirá un mensaje similar a este:

    ```
      Server:  UnKnown
      Address:  168.63.129.16
      Non-authoritative answer:
      Name:    sqlserver.privatelink.database.windows.net
      Address:  10.0.0.5
      Aliases:  sqlserver.database.windows.net
    ```

3.  Instale SQL Server Management Studio.
4.  En  **Conectar con el servidor**, escriba o seleccione esta información:
    - **Tipo de servidor**: Seleccione **Motor de base de datos**.
    - **Nombre del servidor**: seleccione **sqlserver{uniqueid}.database.windows.net**.
    - **Nombre de usuario**: escriba un nombre de usuario proporcionado durante la creación.
    - **Contraseña**: escriba una contraseña proporcionada durante la creación.
    - **Recordar contraseña**: Seleccione **Sí**.

5.  Seleccione **Conectar**.
6.  En el menú de la izquierda, vaya a **Bases de datos**.
7.  También tiene la opción de crear o consultar información desde _sample-db_.
8.  Cierre la conexión a Escritorio remoto a _myVm{uniqueid}_ .

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con el punto de conexión privado, elimine el grupo de recursos. Esta acción quita el punto de conexión privado y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre [Azure Private Link](private-link-overview.md).
