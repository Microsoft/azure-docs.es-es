---
title: Creación y cifrado de una máquina virtual Linux desde Azure Portal
description: En este inicio rápido aprenderá a usar Azure Portal para crear y cifrar una máquina virtual Linux
author: msmbaldwin
ms.author: mbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: quickstart
ms.date: 10/02/2019
ms.openlocfilehash: 563b9c03141357eeacf9a7d890c386faae52c6df
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510712"
---
# <a name="quickstart-create-and-encrypt-a-virtual-machine-with-the-azure-portal"></a>Inicio rápido: Creación y cifrado de una máquina virtual desde Azure Portal

Las máquinas virtuales de Azure pueden crearse mediante Azure Portal. Azure Portal es una interfaz de usuario basada en explorador para crear máquinas virtuales y recursos asociados. En este inicio rápido se usará Azure Portal para implementar una máquina virtual Linux que usa Ubuntu 18.04 LTS, crear un almacén de claves para el almacenamiento de las claves de cifrado y cifrar la máquina virtual.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

1. Elija **Crear un recurso** en la esquina superior izquierda de Azure Portal.
1. En la página Nuevo, en Popular, seleccione **Ubuntu Server 18.04 LTS**.
1. En la pestaña Conceptos básicos, en Detalles del proyecto, asegúrese de que está seleccionada la suscripción correcta.
1. En "Grupo de recursos", seleccione **Crear nuevo**. Como nombre, escriba *myResourceGroup* y seleccione **Aceptar**.
1. En **Nombre de máquina virtual** , escriba *MyVM*.
1. En **Región**, seleccione *(EE. UU.) Este de EE. UU.* .
1. Asegúrese de que el valor de **Tamaño** es *Estándar D2s v3*.
1. En **Cuenta de administrador**, seleccione *Contraseña* en **Tipo de autenticación**. Escriba un nombre de usuario y una contraseña.

    :::image type="content" source="../media/disk-encryption/portal-quickstart-linux-vm-creation.png" alt-text="Pantalla de creación de máquinas virtuales Linux":::

    > [!WARNING]
    > La pestaña "Discos" incluye el campo "Tipo de cifrado" en **Opciones de disco**. Este campo se usa para especificar las opciones de cifrado de [Managed Disks](managed-disks-overview.md) + CMK, **no** para Azure Disk Encryption.
    >
    > Para evitar confusiones, se recomienda omitir la pestaña *Discos* completamente mientras se completa este tutorial.

1. Seleccione la pestaña "Administración" y compruebe que tiene una cuenta de almacenamiento de diagnóstico. Si no tiene cuentas de almacenamiento, seleccione *Crear nuevo*, asigne a la cuenta el nombre *myStorageAccount* y seleccione "Aceptar"

    :::image type="content" source="../media/disk-encryption/portal-quickstart-vm-creation-storage.png" alt-text="Pantalla de creación de grupos de recursos":::

1. Haga clic en "Revisar y crear".
1. En la página **Crear una máquina virtual** verá los detalles de la máquina virtual que va a crear. Cuando esté preparado, seleccione **Crear**.

La implementación de la máquina virtual tardará unos minutos. Cuando finalice la implementación, pase a la siguiente sección.

## <a name="encrypt-the-virtual-machine"></a>Cifrado de la máquina virtual

1. Cuando la implementación de la máquina virtual se complete, seleccione **Ir al recurso**.
1. En la barra lateral izquierda, seleccione **Discos**.
1. En la barra superior, seleccione **Configuración adicional** .
1. En **Configuración de cifrado** > **Discos que se van a cifrar**, seleccione **Discos del SO y de datos**.

    :::image type="content" source="../media/disk-encryption/portal-quickstart-disks-to-encryption.png" alt-text="Selección de discos y de cifrado":::

1. En **Configuración de cifrado**, elija **Seleccionar un almacén de claves y una clave para el cifrado**.
1. En la pantalla **Seleccionar la clave de Azure Key Vault**, seleccione **Crear nuevo**.

    :::image type="content" source="../media/disk-encryption/portal-qs-keyvault-create.png" alt-text="Selección de discos y de cifrado":::

1. A la izquierda de **Almacén de claves y clave**, seleccione **Hacer clic para seleccionar una clave**.
1. En **Seleccionar la clave de Azure Key Vault**, en el campo **Almacén de claves**, seleccione **Crear nuevo**.
1. En la pantalla **Crear almacén de claves**, asegúrese de que el valor de Resource Group es *myResourceGroup* y asigne un nombre al almacén de claves.  Cada almacén de claves en Azure debe tener un nombre único.
1. En la pestaña **Directivas de acceso**, active la casilla **Habilitar el acceso a Azure Disk Encryption para el cifrado de volúmenes**.

    :::image type="content" source="../media/disk-encryption/portal-quickstart-keyvault-enable.png" alt-text="Selección de discos y de cifrado":::

1. Seleccione **Revisar + crear**.  
1. Una vez que el almacén de claves haya superado la validación, seleccione **Crear**. Al hacerlo, regresará a la pantalla **Seleccionar la clave de Azure Key Vault**.
1. Deje en blanco el campo **Clave** y elija **Seleccionar**.
1. En la parte superior de la pantalla de cifrado, haga clic en **Guardar**. Un elemento emergente le avisará que la máquina virtual se va a reiniciar la máquina virtual. Haga clic en **Sí**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede eliminar el grupo de recursos, la máquina virtual y todos los recursos relacionados. Para ello, seleccione el grupo de recursos de la máquina virtual, seleccione Eliminar y confirme el nombre del grupo de recursos que desea eliminar.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado un almacén de claves que se ha habilitado para las claves de cifrado, ha creado una máquina virtual y ha habilitado la máquina virtual para el cifrado.  

> [!div class="nextstepaction"]
> [Introducción a Azure Disk Encryption](disk-encryption-overview.md)
