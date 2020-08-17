---
title: 'Guía de inicio rápido: creación de una máquina virtual Windows en Azure Portal'
description: En esta guía de inicio rápido aprenderá a usar Azure Portal para crear una máquina virtual Windows.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: quickstart
ms.workload: infrastructure
ms.date: 11/05/2019
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 46119b9f7cdac47920d4bba8e00c3fc56b0edc78
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494526"
---
# <a name="quickstart-create-a-windows-virtual-machine-in-the-azure-portal"></a>Inicio rápido: Creación de una máquina virtual de Windows en Azure Portal

Las máquinas virtuales de Azure pueden crearse mediante Azure Portal. Este método proporciona una interfaz de usuario basada en explorador para crear máquinas virtuales y sus recursos asociados. En esta guía de inicio rápido se muestra cómo usar Azure Portal para implementar una máquina virtual (VM) en Azure que ejecuta Windows Server 2019. Para ver la máquina virtual en acción, conéctese a la máquina virtual mediante RDP e instale al servidor web IIS.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-virtual-machine"></a>Crear máquina virtual

1. Escriba **máquinas virtuales** en la búsqueda.
1. En **Servicios**, seleccione **Máquinas virtuales**.
1. En la página **Máquinas virtuales**, seleccione **Agregar**. 
1. En la pestaña **Aspectos básicos**, en **Detalles del proyecto**, asegúrese de que esté seleccionada la suscripción correcta y luego elija **Crear nuevo** grupo de recursos. Escriba *myResourceGroup* para el nombre. 

    ![Captura de pantalla de la sección Detalles del proyecto en la que se muestra dónde se selecciona la suscripción de Azure y el grupo de recursos de la máquina virtual](./media/quick-create-portal/project-details.png)

1. En **Detalles de instancia**, escriba *myVM* en **Nombre de máquina virtual** y elija *Este de EE. UU.* como **Región**. Después, seleccione *Windows Server 2019 Datacenter* para la **Imagen**. Deje los demás valores predeterminados.

    ![Captura de pantalla de la sección Detalles de instancia, en la que se especifica el nombre de la máquina virtual y se selecciona su región, imagen y tamaño](./media/quick-create-portal/instance-details.png)

1. En **Cuenta de administrador**, proporcione un nombre de usuario, como *azureuser*, y una contraseña. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](faq.md#what-are-the-password-requirements-when-creating-a-vm).

    ![Captura de pantalla de la sección Cuenta de administrador, en la que se especifican el nombre de usuario y la contraseña del administrador](./media/quick-create-portal/administrator-account.png)

1. En **Reglas de puerto de entrada**, elija **Permitir los puertos seleccionados** y luego seleccione **RDP (3389)** y **HTTP (80)** en la lista desplegable.

    ![Captura de pantalla de la sección Reglas de puerto de entrada, donde se seleccionan los puertos en los que se permiten conexiones entrantes](./media/quick-create-portal/inbound-port-rules.png)

1. Deje los valores predeterminados restantes y luego seleccione el botón **Revisar + crear** en la parte inferior de la página.

    ![Captura de pantalla en que se muestra el botón Revisar y crear de la parte inferior de la página](./media/quick-create-portal/review-create.png)


## <a name="connect-to-virtual-machine"></a>Conexión a la máquina virtual

Cree una conexión a Escritorio remoto en la máquina virtual. Estas instrucciones indican cómo conectarse a la máquina virtual desde un equipo Windows. En un equipo Mac, necesita un cliente RDP como este [Cliente de Escritorio remoto](https://apps.apple.com/app/microsoft-remote-desktop/id1295203466?mt=12) de Mac App Store.

1. Seleccione el botón **Conectar** en la página de información general de la máquina virtual. 

    ![Captura de pantalla de la página de información general de la máquina virtual que muestra la ubicación del botón Conectar](./media/quick-create-portal/portal-quick-start-9.png)
    
2. En la página **Conectarse a una máquina virtual**, mantenga las opciones predeterminadas para conectarse por dirección IP a través del puerto 3389 y haga clic en **Descargar archivo RDP**.

2. Abra el archivo RDP que descargó y haga clic en **Conectar** cuando se le solicite. 

3. En la ventana **Seguridad de Windows**, seleccione **Más opciones** y, después, **Usar otra cuenta**. Escriba el nombre de usuario como **localhost**\\*username*, escriba la contraseña que creó para la máquina virtual y luego haga clic en **Aceptar**.

4. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Haga clic en **Sí** o en **Continuar** para crear la conexión.

## <a name="install-web-server"></a>Instalación del servidor web

Para ver la máquina virtual en acción, instale el servidor web IIS. Abra un símbolo del sistema de PowerShell en la máquina virtual y ejecute el siguiente comando:

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

Cuando haya terminado, cierre la conexión RDP con la máquina virtual.


## <a name="view-the-iis-welcome-page"></a>Página principal de IIS

En el portal, seleccione la maquina virtual y, en su información general, utilice el botón **Haga clic para copiar** a la derecha de la dirección IP para copiarla y pegarla en una pestaña del explorador. Se abrirá la página de bienvenida de IIS predeterminada, y debería tener el siguiente aspecto:

![Captura de pantalla del sitio predeterminado de IIS en un explorador](./media/quick-create-powershell/default-iis-website.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede eliminar el grupo de recursos, la máquina virtual y todos los recursos relacionados. 

Seleccione el grupo de recursos de la máquina virtual y haga clic en **Eliminar**. Confirme el nombre del grupo de recursos para terminar de eliminar los recursos.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha implementado una máquina virtual sencilla, ha abierto un puerto de red para el tráfico web y ha instalado un servidor web básico. Para más información acerca de las máquinas virtuales de Azure, continúe con el tutorial de máquinas virtuales Windows.

> [!div class="nextstepaction"]
> [Tutoriales de máquinas virtuales Windows de Azure](./tutorial-manage-vm.md)
