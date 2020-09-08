---
title: Creación de un servicio Private Link en Azure Private Link
description: En esta guía de inicio rápido, usará una plantilla de Azure Resource Manager (plantilla de ARM) para crear un servicio de Private Link.
services: private-link
author: mblanco77
ms.service: private-link
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 05/29/2020
ms.author: allensu
ms.openlocfilehash: dce80d70af7cd711cf852a60b98ad65b6d21117f
ms.sourcegitcommit: e0785ea4f2926f944ff4d65a96cee05b6dcdb792
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88705255"
---
# <a name="quickstart-create-a-private-link-service-by-using-an-arm-template"></a>Inicio rápido: Creación de un servicio de Private Link mediante una plantilla de ARM

En esta guía de inicio rápido, usará una plantilla de Azure Resource Manager (plantilla de ARM) para crear un servicio de Private Link.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

También se puede completar este inicio rápido mediante [Azure Portal](create-private-link-service-portal.md), [Azure PowerShell](create-private-link-service-powershell.md) o la [CLI de Azure](create-private-link-service-cli.md).

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-privatelink-service%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Necesita una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Revisión de la plantilla

Esta plantilla crea un servicio Private Link.

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-privatelink-service/).

:::code language="json" source="~/quickstart-templates/101-privatelink-service/azuredeploy.json":::

En la plantilla se definen varios recursos de Azure:

- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks): Hay una red virtual para cada máquina virtual.
- [**Microsoft.Network/loadBalancers**](/azure/templates/microsoft.network/loadBalancers): equilibrador de carga que expone las máquinas virtuales en las que se hospeda el servicio.
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces): hay dos interfaces de red, una para cada máquina virtual.
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines): hay dos máquinas virtuales, una en la que se hospeda el servicio y otra que prueba la conexión al punto de conexión privado.
- [**Microsoft.Compute/virtualMachines/extensions**](/azure/templates/Microsoft.Compute/virtualMachines/extensions): extensión que instala el servidor web.
- [**Microsoft.Network/privateLinkServices**](/azure/templates/microsoft.network/privateLinkServices): servicio Private Link para exponer el servicio.
- [**Microsoft.Network/publicIpAddresses**](/azure/templates/microsoft.network/publicIpAddresses): hay dos direcciones IP públicas, una para cada máquina virtual.
- [**Microsoft.Network/privateendpoints**](/azure/templates/microsoft.network/privateendpoints): punto de conexión privado para acceder al servicio.

## <a name="deploy-the-template"></a>Implementación de la plantilla

Aquí se muestra cómo implementar la plantilla de ARM en Azure:

1. Seleccione **Implementar en Azure** para iniciar sesión en Azure y abrir la plantilla. La plantilla crea una máquina virtual, un equilibrador de carga estándar, un servicio Private Link, un punto de conexión privado, redes y una máquina virtual para validar.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-privatelink-service%2Fazuredeploy.json)

2. Elija su grupo de recursos o cree uno.
3. Escriba el nombre de usuario y la contraseña del administrador de la máquina virtual.
4. Lea la declaración de los términos y condiciones. Si está de acuerdo con ellos, seleccione **Acepto los términos y condiciones indicados anteriormente** > **Comprar**.

## <a name="validate-the-deployment"></a>Validación de la implementación

> [!NOTE]
> La plantilla de ARM genera un nombre único para el recurso de la máquina virtual myConsumerVm<b>{uniqueid}</b>. Sustituya el valor generado para **{uniqueid}** .

### <a name="connect-to-a-vm-from-the-internet"></a>Conexión a una máquina virtual desde Internet

Conéctese a la máquina virtual _myConsumerVm{uniqueid}_ desde Internet de esta forma:

1.  En la barra de búsqueda del portal, escriba _myConsumerVm{uniqueid}_ .

2.  Seleccione **Conectar**. Se abre **Connect to virtual machine** (Conectarse a una máquina virtual).

3.  Seleccione **Descargar archivo RDP**. Azure crea un archivo de Protocolo de Escritorio remoto ( _.rdp_) y lo descarga en su equipo.

4.  Abra el archivo .rdp descargado.

    a. Cuando se le pida, seleccione **Conectar**.

    b. Escriba el nombre de usuario y la contraseña que especificó cuando creó la máquina virtual.
    
    > [!NOTE]
    > Puede que deba seleccionar **Más opciones** > **Usar otra cuenta** para especificar las credenciales que escribió al crear la máquina virtual.

5.  Seleccione **Aceptar**.

6.  Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe una advertencia de certificado, seleccione **Sí** o **Continuar**.

7.  Cuando aparezca el escritorio de la máquina virtual, minimícelo para volver al escritorio local.

### <a name="access-the-http-service-privately-from-the-vm"></a>Acceso al servicio http de forma privada desde la máquina virtual

A continuación, se muestra cómo conectarse al servicio HTTP desde la máquina virtual mediante el punto de conexión privado.

1.  Vaya al Escritorio remoto de _myConsumerVm{uniqueid}_ .
2.  Abra un explorador y escriba la dirección del punto de conexión privado: `http://10.0.0.5/`.
3.  Aparece la página predeterminada de IIS.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con el servicio Private Link, elimine el grupo de recursos. Esta acción quita el servicio Private Link y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre [Azure Private Link](private-link-overview.md).
