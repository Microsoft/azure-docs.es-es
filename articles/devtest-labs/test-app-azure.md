---
title: Prueba de la aplicación en Azure | Microsoft Docs
description: Aprenda a crear un recurso compartido de archivos en un laboratorio y montarlo en su máquina local y en una máquina virtual del laboratorio para, a continuación, implementar aplicaciones web o de escritorio en el recurso compartido de archivos y probarlas.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: b2dbbf349da4e352fe20a22db03cc9063d801990
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282252"
---
# <a name="test-your-app-in-azure"></a>Prueba de las aplicaciones en Azure 
En este artículo se proporciona los pasos para probar la aplicación en Azure con DevTest Labs. En primer lugar, configurará un recurso compartido de archivos en un laboratorio y lo montará como una unidad en la máquina de desarrollo local y en una máquina virtual de un laboratorio. A continuación, usará Visual Studio 2019 para implementar la aplicación en el recurso compartido de archivos de forma que pueda ejecutar la aplicación en la máquina virtual del laboratorio.  

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos 
1. [Cree una suscripción a Azure](https://azure.microsoft.com/free/) si no tiene una e inicie sesión en [Azure Portal](https://portal.azure.com).
2. Siga las instrucciones de [este artículo](devtest-lab-create-lab.md) para crear un laboratorio con Azure DevTest Labs. Ancle el laboratorio al panel para que pueda encontrarlo fácilmente la próxima vez que inicie sesión. Azure DevTest Labs le permite crear recursos en Azure rápidamente, reducir los desperdicios y controlar los costos. Para más información sobre DevTest Labs, consulte [Introducción](devtest-lab-overview.md). 
3. Cree una cuenta de Azure Storage en el grupo de recursos del laboratorio mediante las instrucciones del artículo [Creación de una cuenta de almacenamiento](../storage/common/storage-account-create.md). En la página **Crear cuenta de almacenamiento**, seleccione **Usar existente** en **Grupo de recursos** y seleccione el **grupo de recursos del laboratorio**. 
4. Cree un recurso compartido de archivos en su instancia de Azure Storage mediante las instrucciones del artículo [Creación de un recurso compartido de Azure Files](../storage/files/storage-how-to-create-file-share.md). 

## <a name="mount-the-file-share-on-your-local-machine"></a>Montaje del recurso compartido de archivos en la máquina local
1. En la máquina local, use el script de la sección [Montaje del recurso compartido de archivos de Azure](../storage/files/storage-how-to-use-files-windows.md#mount-the-azure-file-share) del artículo [Uso de un recurso compartido de archivos de Azure con Windows](../storage/files/storage-how-to-use-files-windows.md). 
2. A continuación, use el comando `net use` para montar el recurso compartido de archivos en su máquina. Este es el comando de ejemplo: especifique el nombre de su almacenamiento de Azure y el nombre del recurso compartido de archivos antes de ejecutar el comando. 

    `net use Z: \\<YOUR AZURE STORAGE NAME>.file.core.windows.net\<YOUR FILE SHARE NAME> /persistent:yes`

## <a name="create-a-vm-in-the-lab"></a>Creación de una máquina virtual en el laboratorio
1. En la página **Recurso compartido de archivos**, seleccione el **grupo de recursos** en el menú de la ruta de navegación de la parte superior. Verá la página **Grupo de recursos**. 
    
    ![Selección del grupo de recursos en el menú de navegación](media/test-app-in-azure/select-resource-group-bread-crump.png)
2. En la página **Grupo de recursos**, seleccione el **laboratorio** que creó en DevTest Labs.

    ![Selección del laboratorio](media/test-app-in-azure/select-devtest-lab-in-resource-group.png)
3. En la página **DevTest Lab** de su laboratorio, seleccione **+ Agregar** en la barra de herramientas. 

    ![Agregar botón para el laboratorio](media/test-app-in-azure/add-button-in-lab.png)
4. En la página **Choose a base** (Elegir una base), busque **smalldisk** y seleccione **[smalldisk] Windows Server 2016 Data Center**. 

    ![Elegir el servidor de Windows smalldisk](media/test-app-in-azure/choose-small-disk-windows-server.png)
5. En la página **Máquina virtual**, especifique el **nombre de la máquina virtual**, el **nombre de usuario**, la **contraseña** y seleccione **Crear**.    
    
    ![Página de creación de la máquina virtual](media/test-app-in-azure/create-virtual-machine-page.png)    

## <a name="mount-the-file-share-on-your-vm"></a>Montaje del recurso compartido de archivos en la máquina virtual
1. Una vez que la máquina virtual se ha creado correctamente, seleccione la **máquina virtual** de la lista.    

    ![Seleccionar la máquina virtual de laboratorio](media/test-app-in-azure/select-lab-vm.png)
2. Seleccione **Conectar** en la barra de herramientas para conectarse a la máquina virtual. 
3. [Instale Azure PowerShell](/powershell/azure/install-az-ps).
4. Siga las instrucciones de la sección Montaje del recurso compartido de archivos. 

## <a name="publish-your-app-from-visual-studio"></a>Publicación de la aplicación desde Visual Studio
En esta sección, publicará la aplicación desde Visual Studio en una máquina virtual de prueba en la nube.

1. Cree una aplicación web o de escritorio mediante Visual Studio 2019.
2. Compile la aplicación.
3. Para publicar la aplicación, haga clic con el botón derecho en el proyecto en el **Explorador de soluciones** y seleccione **Publicar**. 
4. En el **Asistente para publicación**, escriba la **unidad** asignada al recurso compartido de archivos.

    **Aplicación de escritorio:**

    ![Aplicación de escritorio](media/test-app-in-azure/desktop-app.png)

    **Aplicación web:**

    ![Aplicación web](media/test-app-in-azure/web-app.png)

1. Seleccione **Siguiente** para finalizar el flujo de trabajo de publicación y seleccione **Finalizar**. Al finalizar los pasos del asistente, Visual Studio compila la aplicación y la publica en el recurso compartido de archivos. 


## <a name="test-the-app-on-your-test-vm-in-the-lab"></a>Prueba de la aplicación en la máquina virtual de prueba del laboratorio

1. Vaya a la página de la máquina virtual del laboratorio. 
2. Seleccione **Iniciar** en la barra de herramientas para iniciar la máquina virtual si está en estado detenido. Puede configurar directivas de inicio y apagado automáticos para la máquina virtual a fin de evitar iniciarla y detenerla en cada ocasión. 
3. Seleccione **Conectar**.

    ![Página de máquina virtual](media/test-app-in-azure/virtual-machine-page.png)
4. Dentro de la máquina virtual, inicie el **Explorador de archivos**y seleccione **Este equipo** para encontrar el recurso compartido de archivos.

    ![Buscar el recurso compartido en la máquina virtual](media/test-app-in-azure/find-share-on-vm.png)

    > [!NOTE]
    > Si, por algún motivo, no puede encontrar el recurso compartido de archivos en la máquina virtual o en la máquina local, puede volver a montarlo mediante la ejecución del comando `net use`. Puede encontrar el comando `net use` en el **Asistente para conexión** del **recurso compartido de archivos** en Azure Portal.
1. Abra el recurso compartido de archivos y confirme que ve la aplicación que ha implementado desde Visual Studio. 

    ![Abrir el recurso compartido en la máquina virtual](media/test-app-in-azure/open-file-share.png)

    Ahora puede acceder a la aplicación y probarla dentro de la máquina virtual de prueba que creó en Azure.

## <a name="next-steps"></a>Pasos siguientes
Consulte los siguientes artículos para aprender a usar máquinas virtuales de un laboratorio. 

- [Adición de una máquina virtual a un laboratorio](devtest-lab-add-vm.md)
- [Reinicio de una máquina virtual de laboratorio](devtest-lab-restart-vm.md)
- [Cambio de tamaño de una máquina virtual de laboratorio](devtest-lab-resize-vm.md)
