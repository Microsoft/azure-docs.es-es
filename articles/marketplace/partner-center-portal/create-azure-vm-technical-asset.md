---
title: Creación de recursos técnicos para máquinas virtuales de Azure
description: Obtenga información sobre cómo crear y configurar recursos técnicos para una oferta de máquina virtual (VM) en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 04/13/2020
ms.openlocfilehash: afc012329d0d9e337dfca93a88615ba7c28f1754
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87460378"
---
# <a name="create-your-azure-virtual-machine-technical-assets"></a>Creación de recursos técnicos para máquinas virtuales de Azure

En este artículo se describe cómo crear y configurar recursos técnicos para una oferta de máquina virtual (VM) para Azure Marketplace. Una máquina virtual contiene dos componentes: el disco duro virtual (VHD) del sistema operativo y los discos duros virtuales de los discos de datos asociados opcionales:

* **Disco duro virtual del sistema operativo**: contiene el sistema operativo y la solución que se implementa con la oferta. El proceso de preparación del disco duro virtual es diferente en función de si se basa en Linux, en Windows o una máquina virtual personalizada.
* **Discos duros virtuales de los discos de datos**: el almacenamiento persistente dedicado para una máquina virtual. No use el disco duro virtual del sistema operativo (por ejemplo, la unidad C:) para almacenar la información persistente.

Una imagen de máquina virtual contiene un disco del sistema operativo y hasta 16 discos de datos. Use un VHD por cada disco de datos, incluso si el disco está vacío.

> [!NOTE]
> Independientemente del sistema operativo que use, agregue solo el número mínimo de discos de datos necesarios para la solución. Los usuarios no pueden quitar los discos que formen parte de una imagen durante la implementación, pero siempre pueden agregar discos durante o después de la implementación.

> [!IMPORTANT]
> Cada imagen de máquina virtual de un plan debe tener el mismo número de discos de datos.

## <a name="fundamental-technical-knowledge"></a>Conocimientos técnicos básicos

El diseño, la compilación y las pruebas de estos recursos lleva tiempo y requiere conocimientos técnicos de la plataforma de Azure y las tecnologías que se usan para crear la oferta. Además del dominio de la solución, el equipo de ingeniería debe tener conocimientos sobre las siguientes tecnologías de Microsoft:

* Conocimientos básicos de los [Servicios de Azure](https://azure.microsoft.com/services/).
* Cómo [diseñar y estructurar las aplicaciones de Azure](https://azure.microsoft.com/solutions/architecture/).
* Conocimientos prácticos de [Azure Virtual Machines](https://azure.microsoft.com/services/virtual-machines/), [Azure Storage](https://azure.microsoft.com/services/?filter=storage) y [redes de Azure](https://azure.microsoft.com/services/?filter=networking).
* Conocimientos prácticos de [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/).
* Conocimientos prácticos de [JSON](https://www.json.org/).

## <a name="suggested-tools--optional"></a>Herramientas sugeridas (opcional)

Considere la posibilidad de usar uno de los siguientes entornos de scripting para ayudar a administrar los discos duros virtuales y las máquinas virtuales:

* [Azure PowerShell](https://docs.microsoft.com/powershell/azure/)
* [CLI de Azure](https://code.visualstudio.com/)

Además, considere la posibilidad de agregar las herramientas siguientes al entorno de desarrollo:

* [Explorador de Azure Storage](../../vs-azure-tools-storage-manage-with-storage-explorer.md)
* [Visual Studio Code](https://code.visualstudio.com/)
  * Extensión: [Herramientas de Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)
  * Extensión: [Beautify](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify)
  * Extensión: [Prettify JSON](https://marketplace.visualstudio.com/items?itemName=mohsen1.prettify-json)

Revise las herramientas disponibles en la página [Herramientas de desarrollo de Azure](https://azure.microsoft.com/product-categories/developer-tools/) y, si usa Visual Studio, en [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

## <a name="create-a-vm-image-using-an-approved-base"></a>Creación de una imagen de máquina virtual mediante una base aprobada

> [!NOTE]
> Para crear recursos técnicos de máquinas virtuales con una imagen creada en sus propios entornos locales, vaya a [Creación de una máquina virtual con su propia imagen](#create-a-vm-using-your-own-image).

En esta sección se describen varios aspectos relativos al uso de una base aprobada, como el uso del Protocolo de escritorio remoto (RDP), la selección de un tamaño para la máquina virtual, la instalación de las actualizaciones de Windows más recientes y cómo generalizar la imagen de disco duro virtual.

Las secciones siguientes se centran principalmente en los discos duros virtuales basados en Windows. Para más información sobre cómo crear un VHD basado en Linux, vea [Distribuciones de Linux aprobadas en Azure](../../virtual-machines/linux/endorsed-distros.md).

> [!WARNING]
> Siga las instrucciones de este tema para usar Azure para crear una máquina virtual que contenga un sistema operativo preconfigurado y aprobado. Si esto no es compatible con la solución, es posible crear y configurar una máquina virtual de manera local mediante un sistema operativo aprobado. Después, puede configurarlo y prepararlo para la carga, como se describe en [Preparación de un VHD o un VHDX de Windows antes de cargarlo en Azure](../../virtual-machines/windows/prepare-for-upload-vhd-image.md).

### <a name="select-an-approved-base"></a>Selección de una base aprobada

Seleccione el sistema operativo Windows o Linux como base.

#### <a name="windows"></a>Windows

El VHD del sistema operativo para su imagen de máquina virtual basada en Windows debe basarse en una imagen base aprobada para Azure que contenga Windows Server o SQL Server. Para comenzar, cree una máquina virtual a partir de alguna de las imágenes de Azure Portal:

* Windows Server ([2016](https://www.microsoft.com/evalcenter/evaluate-windows-server-2016), [2012 R2 Datacenter](https://www.microsoft.com/cloud-platform/windows-server-pricing), [2012 Datacenter](https://www.microsoft.com/cloud-platform/windows-server-pricing), [2008 R2 SP1](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=Overview))
* [Windows 10 IoT Enterprise](/windows/iot-core/windows-iot-enterprise)
* [SQL Server 2014](../../virtual-machines/windows/sql/virtual-machines-windows-sql-server-pricing-guidance.md) (Enterprise, Standard, Web)
* [SQL Server 2012 SP2](../../virtual-machines/windows/sql/virtual-machines-windows-sql-server-pricing-guidance.md) (Enterprise, Standard, Web)

> [!NOTE]
> Si usa la versión actual de Azure Portal o Azure PowerShell, las imágenes de Windows Server publicadas a partir del 8 de septiembre de 2014 están aprobadas.

#### <a name="linux"></a>Linux

Azure ofrece una gama de distribuciones de Linux aprobadas. Para obtener una lista actual, vea [Linux en distribuciones aprobadas por Azure](../../virtual-machines/linux/endorsed-distros.md).

### <a name="create-vm-in-the-azure-portal"></a>Creación de la máquina virtual en Azure Portal

Siga estos pasos para crear una imagen base de máquina virtual en [Azure Portal](https://ms.portal.azure.com/):

1. Inicie sesión en [Azure Portal](https://ms.portal.azure.com/) con la cuenta Microsoft asociada con la suscripción de Azure que quiera usar para publicar la oferta de máquina virtual.
2. Cree un grupo de recursos y proporcione el **Nombre del grupo de recursos**, la **Suscripción** y la **Ubicación del grupo de recursos**. Para conocer los detalles, vea [Administrar recursos](../../azure-resource-manager/resource-group-portal.md).
3. Seleccione **Máquinas virtuales** a la izquierda para mostrar la página de detalles Máquinas virtuales.
4. Seleccione **+ Agregar** para abrir la **experiencia Crear una máquina virtual**.
5. Seleccione la imagen en la lista desplegable o haga clic en **Examinar todas las imágenes públicas y privadas** para buscar o examinar todas las imágenes de máquina virtual disponibles.
6. Seleccione el tamaño de la máquina virtual que se va a implementar mediante las recomendaciones siguientes:
    * Si planea desarrollar el VHD en modo local, el tamaño no importa. Considere usar una de las máquinas virtuales de menor tamaño.
    * Si planea desarrollar la imagen en Azure, considere usar uno de los tamaños de máquina virtual recomendados para la imagen seleccionada.

7. En la sección **Discos**, expanda **Avanzado** y establezca la opción **Usar discos administrados** en **No**.
8. Proporcione los otros detalles necesarios para crear la máquina virtual.
9. Seleccione **Revisar y crear** para revisar sus selecciones. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**.

Azure comienza el aprovisionamiento de la máquina virtual especificada. Puede seguir su progreso si selecciona la pestaña **Máquinas virtuales** de la izquierda. Una vez creada, el estado cambiará a **En ejecución**.

Si tiene dificultades para crear el disco duro virtual basado en Azure, vea [Problemas comunes durante la creación de discos duros virtuales (preguntas frecuentes)](common-issues-during-vhd-creation.md).

### <a name="connect-to-your-azure-vm"></a>Conexión a una máquina virtual de Azure

En esta sección se explica cómo conectarse a las máquinas virtuales (VM) creadas en Azure y cómo iniciar sesión en ellas. Cuando se haya conectado correctamente, puede trabajar con la máquina virtual como si hubiera iniciado sesión localmente en su servidor host.

#### <a name="connect-to-a-windows-based-vm"></a>Conectarse a una máquina virtual basada en Windows

Para conectarse a la máquina virtual basada en Windows hospedada en Azure, use el cliente de escritorio remoto. La mayoría de las versiones de Windows incluyen compatibilidad de forma nativa con el Protocolo de escritorio remoto (RDP). Para otros sistemas operativos, puede encontrar más información sobre los clientes en [Clientes de escritorio remoto](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/clients/remote-desktop-clients).

En este artículo se detalla cómo usar la compatibilidad integrada de Windows RDP para conectarse a una VM: [Conexión a una máquina virtual de Azure donde se ejecuta Windows e inicio de sesión en ella](../../virtual-machines/windows/connect-logon.md).

> [!TIP]
> Puede obtener advertencias de seguridad durante el proceso. Por ejemplo, advertencias como "El archivo .rdp procede de un publicador desconocido" o "No se pueden verificar sus credenciales de usuario". Es seguro hacer caso omiso de estas advertencias.

#### <a name="connect-to-a-linux-based-vm"></a>Conectarse a una máquina virtual basada en Linux

Para conectarse a una máquina virtual basada en Linux, necesita un cliente de protocolo Secure Shell (SSH). Los pasos siguientes usan el terminal SSH [PuTTY](https://www.ssh.com/ssh/putty/) libre.

1. Vaya a [Azure Portal](https://ms.portal.azure.com/).
2. Busque y seleccione **Máquinas virtuales**.
3. Seleccione el nombre de la máquina virtual a la que quiere conectarse.
4. Inicie la máquina virtual si aún no se está ejecutando.
5. Seleccione el nombre de la máquina virtual para abrir su página **Información general**.
6. Tenga en cuenta la dirección IP pública y el nombre DNS de la VM (si no se establecen estos valores, debe [Crear una interfaz de red](../../virtual-network/virtual-network-network-interface.md#create-a-network-interface)).
7. Abra la aplicación PuTTY.
8. En el cuadro de diálogo Configuración de PuTTY, escriba la dirección IP o el nombre DNS de la máquina virtual.

    :::image type="content" source="media/avm-putty.png" alt-text="Muestra la configuración de terminal PuTTy. Se resaltan los cuadros Nombre de host o Dirección IP y Puerto":::.

9. Seleccione **Abrir** para abrir un terminal PuTTY.
10. Cuando se le pida, escriba el nombre y la contraseña de la cuenta de máquina virtual Linux.

Si tiene problemas de conexión, vea la documentación del cliente SSH. Por ejemplo, [Capítulo 10: Mensajes de error habituales](https://www.ssh.com/ssh/putty/putty-manuals).

Para obtener información detallada, por ejemplo, cómo agregar un escritorio a una máquina virtual Linux aprovisionada, vea [Instalación y configuración del escritorio remoto para conectarse a una máquina virtual Linux en Azure](../../virtual-machines/linux/use-remote-desktop.md).

## <a name="create-a-vm-using-your-own-image"></a>Creación de una máquina virtual con su propia imagen

En esta sección se describe cómo crear e implementar una imagen de máquina virtual (VM) proporcionada por el usuario. Para ello, proporcione imágenes del VHD del sistema operativo y del disco de datos desde un disco duro virtual (VHD) implementado en Azure.

> [!NOTE]
> Para usar opcionalmente una imagen base aprobada, siga las instrucciones de [Creación de una imagen de máquina virtual con una base aprobada](#create-a-vm-image-using-an-approved-base).

1. Cargue sus imágenes en la cuenta de Azure Storage.
2. Implemente la imagen de máquina virtual.
3. Capture la imagen de máquina virtual.

### <a name="upload-your-images-to-an-azure-storage-account"></a>Cargue sus imágenes en una cuenta de Azure Storage.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Cargue el VHD del sistema operativo generalizado y los VHD del disco de datos en su cuenta de Azure Storage.

### <a name="deploy-your-image"></a>Implementación de la imagen

Cree la imagen mediante Azure Portal o Azure PowerShell.

#### <a name="deploy-using-the-azure-portal"></a>Implementación mediante Azure Portal

1. En la página principal, seleccione **Crear un recurso**, busque "Template Deployment" y seleccione **Crear**.
2. Seleccione **Cree su propia plantilla en el editor**.

    :::image type="content" source="media/avm-custom-deployment.png" alt-text="Se muestra la página Implementación personalizada":::.

3. Pegue esta [plantilla JSON](../partner-center-portal/azure-vm-image-certification.md) en el editor y seleccione **Guardar**.
4. Proporcione los valores de parámetro de las páginas de propiedades **Implementación personalizada** que se muestran.

    | Parámetro | Descripción |
    | ------------ | ------------- |
    | User Storage Account Name (Nombre de usuario de la cuenta de almacenamiento) | Contenido de la celda 2 |
    | User Storage Container Name (Nombre de usuario del contenedor de almacenamiento) | Nombre de la cuenta de almacenamiento donde se encuentra el disco duro virtual generalizado |
    | DNS Name for Public IP (Nombre DNS para IP pública) | Nombre DNS de IP pública. Defina el nombre DNS para la dirección IP pública en Azure Portal después de implementar la oferta. |
    | Nombre de usuario administrador | Nombre de usuario de la cuenta de administrador de la nueva máquina virtual |
    | Contraseña de administrador | Contraseña de la cuenta de administrador de la nueva máquina virtual |
    | Tipo de SO | Sistema operativo de la máquina virtual: Windows o Linux |
    | Id. de suscripción | Identificador de la suscripción seleccionada |
    | Location | Ubicación geográfica de la implementación |
    | Tamaño de VM | [Tamaño de la máquina virtual de Azure](../../virtual-machines/windows/sizes.md); por ejemplo, Standard_A2 |
    | Nombre de dirección IP pública | Nombre de la dirección IP pública |
    | Nombre de la máquina virtual | Nombre de la nueva máquina virtual |
    | El nombre de la red virtual | Nombre de la red virtual que usa la máquina virtual |
    | Nombre de NIC | Nombre de la tarjeta de interfaz de red que ejecuta la red virtual |
    | URL del VHD | Dirección URL completa del VHD del disco del sistema operativo |
    |  |  |

5. Después de proporcionar estos valores, seleccione **Comprar**.

Azure comenzará la implementación. Crea una máquina virtual con el VHD no administrado especificado en la ruta de acceso de la cuenta de almacenamiento especificada. Puede realizar un seguimiento del progreso en Azure Portal si selecciona **Máquinas virtuales** en el lado izquierdo del portal. Una vez creada la máquina virtual, el estado cambiará de Iniciando a En ejecución.

#### <a name="deploy-using-azure-powershell"></a>Implementación mediante Azure PowerShell

```powershell
    $img = Get-AzureVMImage -ImageName "myVMImage"
    $user = "user123"
    $pass = "adminPassword123"
    $myVM = New-AzureVMConfig -Name "VMImageVM" -InstanceSize "Large" -ImageName $img.ImageName | Add-AzureProvisioningConfig -Windows -AdminUsername $user -Password $pass
    New-AzureVM -ServiceName "VMImageCloudService" -VMs $myVM -Location "West US" -WaitForBoot
```

### <a name="capture-the-vm-image"></a>Capturar la imagen de máquina virtual

Use las instrucciones siguientes que se correspondan con su enfoque:

* Azure PowerShell: [Captura de una imagen administrada de una máquina virtual generalizada en Azure](../../virtual-machines/windows/capture-image-resource.md)
* CLI de Azure: [Creación de una imagen de una máquina virtual o un disco duro virtual](../../virtual-machines/linux/capture-image.md)
* API: [Virtual Machines - Capture](https://docs.microsoft.com/rest/api/compute/virtualmachines/capture) (Máquinas virtuales: Capture)

## <a name="configure-the-virtual-machine"></a>Configuración de la máquina virtual

En esta sección se describe cómo ajustar el tamaño, actualizar y generalizar una máquina virtual de Azure. Estos pasos son necesarios para preparar la máquina virtual para su implementación en Azure Marketplace.

### <a name="sizing-the-vhds"></a>Cambio de tamaño de los VHD

Si selección una de las VM preconfiguradas con un sistema operativo, y, opcionalmente, otros servicios, ya ha seleccionado un tamaño de VM de Azure estándar. Iniciar la solución con un sistema operativo configurado previamente es el enfoque recomendado. Pero si va a instalar un sistema operativo manualmente, debe ajustar el tamaño del disco duro virtual principal en la imagen de máquina virtual:

* En Windows, el VHD del sistema operativo debe crearse como VHD de 127 o 128 GB con formato fijo.
* En Linux, este VHD debe crearse como VHD de 30 a 50 GB con formato fijo.

Si el tamaño físico es inferior a 127 o 128 GB, el VHD debe ser ampliable (disperso o dinámico). Las imágenes base de Windows y SQL Server proporcionadas cumplen ya estos requisitos, por lo que no es necesario que cambie el formato ni el tamaño del VHD.

Los discos de datos pueden tener un tamaño de hasta 1 TB. A la hora de decidir el tamaño, tenga en cuenta que los clientes no pueden cambiar el tamaño de los VHD de una imagen en el momento de la implementación. Los VHD de discos de datos deben crearse como VHD con formato fijo. También deben ser ampliables (dispersos o dinámicos). Los discos de datos inicialmente pueden estar vacíos o contener datos.

### <a name="install-the-most-current-updates"></a>Instalar las actualizaciones más recientes

Las imágenes base de las máquinas virtuales del sistema operativo deben contener las últimas actualizaciones hasta su fecha de publicación. Antes de publicar el VHD del sistema operativo que ha creado, asegúrese de actualizar el sistema operativo y todos los servicios instalados con todas las revisiones de seguridad y mantenimiento más recientes.

En Windows Server, ejecute el comando **Buscar actualizaciones**.

En distribuciones Linux, las actualizaciones se suelen descargar e instalar mediante una herramienta de línea de comandos o una utilidad gráfica. Por ejemplo, Ubuntu Linux proporciona el comando [apt-get](https://manpages.ubuntu.com/manpages/cosmic/man8/apt-get.8.html) y la herramienta [Update Manager](https://manpages.ubuntu.com/manpages/cosmic/man8/update-manager.8.html) para actualizar el sistema operativo.

### <a name="perform-additional-security-checks"></a>Realizar comprobaciones de seguridad adicionales

Mantenga un alto nivel de seguridad para las imágenes de la solución en Azure Marketplace. En el siguiente artículo se proporciona una lista de comprobación de las configuraciones y los procedimientos de seguridad para ayudarle: [Recomendaciones de seguridad para imágenes de Azure Marketplace](../../security/security-recommendations-azure-marketplace-images.md). Algunas de estas recomendaciones son específicas de imágenes basadas en Linux, pero la mayoría se aplica a cualquier imagen de máquina virtual.

### <a name="perform-custom-configuration-and-scheduled-tasks"></a>Realizar una configuración personalizada y tareas programadas

Si es necesaria alguna configuración adicional, use una tarea programada que se ejecute al inicio para realizar posibles cambios finales en la máquina virtual una vez implementada. Tenga en cuenta además las siguientes recomendaciones:

* Si es una tarea de una ejecución, la tarea debe eliminarse a sí misma una vez que se realice correctamente.
* Las configuraciones no deben basarse en unidades distintas de C o D, ya que solo se garantiza la existencia continua de estas dos unidades (la unidad C es el disco del sistema operativo y la unidad D es el disco local temporal).

Para obtener más información sobre las personalizaciones de Linux, vea [Características y extensiones de las máquinas virtuales para Linux](../../virtual-machines/extensions/features-linux.md).

## <a name="generalize-the-image"></a>Generalizar la imagen

Todas las imágenes de Azure Marketplace deben ser reutilizables de forma genérica. Para lograrlo, el VHD del sistema operativo debe estar generalizado, una operación que quita todos los controladores de software y los identificadores específicos de la instancia de una máquina virtual.

### <a name="windows"></a>Windows

Los discos de sistema operativo Windows se generalizan con la [herramienta sysprep](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview). Si posteriormente actualiza o vuelve a configurar el sistema operativo, debe volver a ejecutar sysprep.

> [!WARNING]
> Dado que las actualizaciones se pueden ejecutar automáticamente, una vez que ejecute sysprep, desactive la máquina virtual hasta que se implemente. Este apagado evita que las actualizaciones posteriores realicen cambios específicos de la instancia en el sistema operativo o los servicios instalados. Para más información sobre la ejecución de sysprep, vea [Pasos para generalizar un disco duro virtual](../../virtual-machines/windows/capture-image-resource.md#generalize-the-windows-vm-using-sysprep).

### <a name="linux"></a>Linux

El siguiente proceso generaliza una máquina virtual Linux y la vuelve a implementar como una máquina virtual independiente. Para obtener información detallada, vea [Creación de una imagen de una máquina virtual o un disco duro virtual](../../virtual-machines/linux/capture-image.md). Puede detenerse cuando llegue a la sección "Crear una máquina virtual a partir de la imagen capturada".

1. **Quitar el agente de Linux de Azure**

    1. Conexión a una máquina virtual Linux de Azure mediante un cliente SSH.
    2. En la ventana de SSH, escriba el siguiente comando: `sudo waagent -deprovision+user`.
    3. Escriba **Y** para continuar (puede agregar el parámetro **-force** al comando anterior para evitar el paso de confirmación).
    d. Una vez completado el comando, escriba **Salir** para cerrar el cliente de SSH.

2. **Detener una máquina virtual**

    1. En Azure Portal, seleccione el grupo de recursos (RG) y anule la asignación de la máquina virtual.
    2. El disco duro virtual ahora se ha generalizado y puede crear una máquina virtual mediante este disco duro virtual.

## <a name="next-steps"></a>Pasos siguientes

Si tiene dificultades para crear el disco duro virtual basado en Azure, vea [Common issues during VHD creation](common-issues-during-vhd-creation.md) (Problemas comunes durante la creación de discos duros virtuales).

De lo contrario:

* La [Prueba de máquina virtual implementada desde un disco duro virtual](azure-vm-image-certification.md) explica cómo probar y enviar una imagen de máquina virtual para la certificación de Azure Marketplace, incluido dónde obtener la *herramienta de prueba de certificación para la herramienta Azure Certified* y cómo usarla para certificar la imagen de máquina virtual.
