---
title: Creación de recursos técnicos para máquinas virtuales de Azure
description: Obtenga información sobre cómo crear y configurar recursos técnicos para una oferta de máquina virtual (VM) en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 08/14/2020
ms.openlocfilehash: 07c8de2a9d94b51f7183829466bd68d56e19efba
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89646798"
---
# <a name="create-azure-virtual-machine-technical-assets"></a>Creación de recursos técnicos para máquinas virtuales de Azure

Al publicar sus imágenes de máquina virtual en Azure Marketplace, el equipo de Azure valida la imagen de máquina virtual para garantizar el arranque, la seguridad y la compatibilidad con Azure. Si alguna de las pruebas de alta calidad fracasa, la publicación generará un mensaje con el error y los posibles [pasos de rectificación](https://docs.microsoft.com/azure/marketplace/partner-center-portal/vm-certification-issues-solutions).

En este artículo se describe cómo crear y configurar recursos técnicos para una oferta de máquina virtual (VM) para Azure Marketplace. Una máquina virtual contiene dos componentes: el disco duro virtual (VHD) del sistema operativo y los discos duros virtuales de los discos de datos asociados opcionales:

1. **Disco duro virtual del sistema operativo**: contiene el sistema operativo y la solución que se implementa con la oferta. El proceso de preparación del disco duro virtual es diferente en función de si se basa en Linux, en Windows o una máquina virtual personalizada.

2. **Discos duros virtuales de discos de datos**: el almacenamiento persistente dedicado para una máquina virtual. No use el disco duro virtual del sistema operativo (por ejemplo, la unidad C:) para almacenar la información persistente.

Una imagen de máquina virtual contiene un disco del sistema operativo y hasta 16 discos de datos. Use un VHD por cada disco de datos, incluso si el disco está vacío.

> [!NOTE]
> Independientemente del sistema operativo que use, agregue solo el número mínimo de discos de datos necesarios para la solución. Los usuarios no pueden quitar los discos que formen parte de una imagen durante la implementación, pero siempre pueden agregar discos durante o después de la implementación.

> [!IMPORTANT]
> Cada imagen de máquina virtual de un plan debe tener el mismo número de discos de datos.

## <a name="fundamental-technical-knowledge"></a>Conocimientos técnicos básicos

El diseño, la compilación y las pruebas de estos recursos lleva tiempo y requiere conocimientos técnicos de la plataforma de Azure y las tecnologías que se usan para crear la oferta. Además del dominio de la solución, el equipo de ingeniería debe tener conocimientos sobre las siguientes tecnologías de Microsoft:

- Conocimientos básicos de los [Servicios de Azure](https://azure.microsoft.com/services/).
- Cómo [diseñar y estructurar las aplicaciones de Azure](https://azure.microsoft.com/solutions/architecture/).
- Conocimientos prácticos de [Azure Virtual Machines](https://azure.microsoft.com/services/virtual-machines/), [Azure Storage](https://azure.microsoft.com/services/?filter=storage) y [redes de Azure](https://azure.microsoft.com/services/?filter=networking).
- Conocimientos prácticos de [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/).
- Conocimientos prácticos de [JSON](https://www.json.org/).

### <a name="optional-suggested-tools"></a>Herramientas opcionales sugeridas

Considere la posibilidad de usar uno de los siguientes entornos de scripting para ayudar a administrar los discos duros virtuales y las máquinas virtuales:

- [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview)
- [CLI de Azure](https://code.visualstudio.com/)

Además, considere la posibilidad de agregar las herramientas siguientes al entorno de desarrollo:

- [Explorador de Azure Storage](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer)
- [Visual Studio Code](https://code.visualstudio.com/)

## <a name="create-a-vm-image-using-an-approved-base"></a>Creación de una imagen de máquina virtual mediante una base aprobada

Para crear recursos técnicos de máquinas virtuales con una imagen creada en sus propios entornos locales, consulte [Creación de una imagen de máquina virtual con una base aprobada](#create-a-vm-image-using-an-approved-base).

En esta sección se describen varios aspectos relativos al uso de una base aprobada, como el uso del Protocolo de escritorio remoto (RDP), la selección de un tamaño para la máquina virtual, la instalación de las actualizaciones de Windows más recientes y cómo generalizar la imagen de disco duro virtual.

Siga las instrucciones de este artículo para usar Azure en la creación de una máquina virtual que contenga un sistema operativo preconfigurado y aprobado. Si esto no es compatible con la solución, es posible crear y configurar una máquina virtual de manera local mediante un sistema operativo aprobado. Después, puede configurarlo y prepararlo para la carga, como se describe en [Preparación de un VHD o un VHDX de Windows antes de cargarlo en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/prepare-for-upload-vhd-image).

### <a name="select-an-approved-base-image"></a>Selección de una imagen de base aprobada

Seleccione el sistema operativo Windows o Linux como base.

Los VHD del sistema operativo para su imagen de máquina virtual deben basarse en una imagen aprobada para Azure que contenga Windows Server o SQL Server.

Cuando envía una solicitud para volver a publicar su imagen con actualizaciones, el caso de prueba de comprobación del número de pieza puede dar error. En esa situación, la imagen no se aprobará.

Este error se produce al utilizar una imagen base que pertenece a otro publicador y actualizar la imagen. En este caso, no se le permitirá publicar la imagen.

Para corregir el problema, recupere la imagen más reciente de Azure Marketplace y haga cambios en la imagen. Consulte los siguientes artículos para ver las imágenes base aprobadas en las que puede buscar su imagen:

#### <a name="windows"></a>Windows

- Windows Server ([2019](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=Overview), [2016](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=Overview), [2012 R2 Datacenter](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=Overview), [2012 Datacenter](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=Overview) y [2008 R2 SP1](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=Overview))
- SQL Server 2019 ([Enterprise](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftsqlserver.sql2019-ws2019?tab=Overview), [Standard](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftsqlserver.sql2019-ws2019?tab=Overview) y [Web](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftsqlserver.sql2019-ws2019?tab=Overview))
- SQL Server 2014 ([Enterprise](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-pricing-guidance), [Standard](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-pricing-guidance) y [Web](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-pricing-guidance))
- SQL Server 2012 SP2 ([Enterprise](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-pricing-guidance), [Standard](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-pricing-guidance) y [Web](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-pricing-guidance))

#### <a name="linux"></a>Linux

Azure ofrece una gama de distribuciones de Linux aprobadas. Para obtener una lista actual, vea [Linux en distribuciones aprobadas por Azure](https://docs.microsoft.com/azure/virtual-machines/linux/endorsed-distros).

### <a name="create-vm-on-the-azure-portal"></a>Creación de la máquina virtual en Azure Portal

Siga estos pasos para crear una imagen base de máquina virtual en [Azure Portal](https://ms.portal.azure.com/):

1. Inicie sesión en [Azure Portal](https://ms.portal.azure.com/) con la cuenta Microsoft asociada con la suscripción de Azure que quiera usar para publicar la oferta de máquina virtual.
2. Cree un grupo de recursos y proporcione el **Nombre del grupo de recursos**, la **Suscripción** y la **Ubicación del grupo de recursos**. Para conocer los detalles, vea [Administrar recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-portal).

    :::image type="content" source="media/vm/create-resource-group.png" alt-text="Se muestra cómo comenzar a crear un grupo de recursos.":::

3. Seleccione **Máquinas virtuales** en la navegación de la izquierda para mostrar la página de detalles Máquinas virtuales.
4. Seleccione **+ Agregar** para abrir la **experiencia Crear una máquina virtual**.
5. Seleccione la imagen en la lista desplegable o **Examinar todas las imágenes públicas y privadas** para buscar o examinar todas las imágenes de máquina virtual disponibles. Ejemplo:

    :::image type="content" source="media/vm/create-resource-group-example.png" alt-text="Muestra una imagen de máquina virtual de ejemplo.":::

6. Seleccione el tamaño de la máquina virtual que se va a implementar mediante las recomendaciones siguientes:
    1. Si planea desarrollar el VHD en modo local, el tamaño no importa. Considere usar una de las máquinas virtuales de menor tamaño.
    2. Si planea desarrollar la imagen en Azure, considere usar uno de los tamaños de máquina virtual recomendados para la imagen seleccionada.

    :::image type="content" source="media/vm/create-virtual-machine.png" alt-text="Muestra la selección del tamaño de máquina virtual.":::

7. En la sección **Discos**, expanda **Avanzado** y establezca la opción **Usar discos administrados** en **No**.

    :::image type="content" source="media/vm/use-managed-disks.png" alt-text="Muestra la opción para usar discos administrados.":::

8. Proporcione los otros detalles necesarios para crear la máquina virtual.
9. Seleccione **Revisar y crear** para revisar sus selecciones. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**.

Azure comienza el aprovisionamiento de la máquina virtual especificada. Puede seguir su progreso si selecciona la pestaña **Máquinas virtuales** de la izquierda. Una vez creada, el estado cambiará a **En ejecución**.

### <a name="create-a-generation-2-vm-on-the-azure-portal"></a>Creación una máquina virtual de generación 2 en Azure Portal

Cree una máquina virtual de generación 2 (Gen2) en Azure Portal.

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com/).
2. Seleccione **Crear un recurso**.
3. Seleccione **Ver todo** junto a Azure Marketplace, en el lado izquierdo.
4. Seleccione una imagen compatible con Gen2.
5. Seleccione **Crear**.
6. En la pestaña **Avanzado**, en la sección **Generación de VM**, seleccione la opción **Gen 2**.
7. En la pestaña **Datos básicos**, en **Detalles de instancia**, vaya a **Tamaño** y abra la hoja **Seleccionar un tamaño de máquina virtual**.
8. Seleccione un tamaño recomendado de [máquina virtual Gen 2 compatible](https://docs.microsoft.com/azure/virtual-machines/windows/generation-2#generation-2-vm-sizes) y un tamaño.
9. Realice el [flujo de creación de Azure Portal](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal) para terminar de crear la máquina virtual.

    :::image type="content" source="media/vm/vm-generation.png" alt-text="Muestra la opción para seleccionar la generación de máquina virtual.":::

## <a name="connect-to-your-azure-vm"></a>Conexión a una máquina virtual de Azure

En esta sección se explica cómo conectarse a las máquinas virtuales (VM) que creó en Azure y cómo iniciar sesión en ellas. Cuando se haya conectado correctamente, puede trabajar con la máquina virtual como si hubiera iniciado sesión localmente en su servidor host.

### <a name="connect-to-a-windows-based-vm"></a>Conectarse a una máquina virtual basada en Windows

Para conectarse a la máquina virtual basada en Windows hospedada en Azure, use el cliente de escritorio remoto. La mayoría de las versiones de Windows incluyen compatibilidad de forma nativa con el Protocolo de escritorio remoto (RDP). Para otros sistemas operativos, puede encontrar más información sobre los clientes en [Clientes de escritorio remoto](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/clients/remote-desktop-clients).

En este artículo se detalla cómo usar la compatibilidad integrada de Windows RDP para conectarse a una VM: [Conexión a una máquina virtual de Azure donde se ejecuta Windows e inicio de sesión en ella](https://docs.microsoft.com/azure/virtual-machines/windows/connect-logon).

> [!TIP]
> Puede obtener advertencias de seguridad durante el proceso. Por ejemplo, advertencias como "El archivo .rdp procede de un publicador desconocido" o "No se pueden verificar sus credenciales de usuario". Es seguro hacer caso omiso de estas advertencias.

### <a name="connect-to-a-linux-based-vm"></a>Conectarse a una máquina virtual basada en Linux

Para conectarse a una máquina virtual basada en Linux, necesita un cliente de protocolo Secure Shell (SSH). Los pasos siguientes usan el terminal SSH [PuTTY](https://www.ssh.com/ssh/putty/) libre.

1. Vaya a [Azure Portal](https://ms.portal.azure.com/).
2. Busque y seleccione Máquinas virtuales.
3. Seleccione el nombre de la máquina virtual a la que quiere conectarse.
4. Inicie la máquina virtual si aún no se está ejecutando.
5. Seleccione el nombre de la máquina virtual para abrir su página Información general.
6. Tenga en cuenta la dirección IP pública y el nombre DNS de la VM (si no se establecen estos valores, debe [Crear una interfaz de red](https://docs.microsoft.com/azure/virtual-network/virtual-network-network-interface#create-a-network-interface)).
7. Abra la aplicación PuTTY.
8. En el cuadro de diálogo Configuración de PuTTY, escriba la dirección IP o el nombre DNS de la máquina virtual.

    :::image type="content" source="media/vm/putty-configuration.png" alt-text="Muestra la configuración de terminal PuTTY y se resaltan los campos Nombre de host o Puerto":::.

9. Seleccione **Abrir** para abrir un terminal PuTTY.
10. Cuando se le pida, escriba el nombre y la contraseña de la cuenta de máquina virtual Linux.

## <a name="configure-the-virtual-machine"></a>Configuración de la máquina virtual

En esta sección se describe cómo ajustar el tamaño, actualizar y generalizar una máquina virtual de Azure. Estos pasos son necesarios para preparar la máquina virtual para su implementación en Azure Marketplace.

### <a name="sizing-the-vhds"></a>Cambio de tamaño de los VHD

Las siguientes reglas se aplican a las limitaciones del tamaño del disco del sistema operativo. Al enviar una solicitud, asegúrese de que el tamaño del disco del sistema operativo cumple el requisito para Linux o Windows.

| SO | Tamaño recomendado del disco duro virtual |
| --- | --- |
| Linux | 30 a 1023 GB |
| Windows | 30 a 250 GB |

Como las máquinas virtuales permiten acceder al sistema operativo subyacente, asegúrese de que el tamaño del disco duro virtual sea lo suficientemente grande. Dado que los discos no se pueden expandir sin que haya tiempo de inactividad, use un tamaño de disco comprendido entre 30 y 50&nbsp;GB.

| Tamaño de VHD | Tamaño real ocupado | Solución |
| --- | --- | --- |
| Más de 500 TB | N/D | Póngase en contacto con el equipo de soporte técnico para obtener una aprobación extraordinaria. |
| 250 a 500 TB | Una diferencia de más de 200 GB en relación con el tamaño de blob | Póngase en contacto con el equipo de soporte técnico para obtener una aprobación extraordinaria. |

### <a name="install-the-most-current-updates"></a>Instalar las actualizaciones más recientes

Las imágenes base de las máquinas virtuales del sistema operativo deben contener las últimas actualizaciones hasta su fecha de publicación. Antes de publicar el VHD del sistema operativo que ha creado, asegúrese de actualizar el sistema operativo y todos los servicios instalados con todas las revisiones de seguridad y mantenimiento más recientes.

- En Windows Server, ejecute el comando Buscar actualizaciones.
- En distribuciones Linux, las actualizaciones se suelen descargar e instalar mediante una herramienta de línea de comandos o una utilidad gráfica. Por ejemplo, Ubuntu Linux proporciona el comando [apt-get](https://manpages.ubuntu.com/manpages/cosmic/man8/apt-get.8.html) y la herramienta [Update Manager](https://manpages.ubuntu.com/manpages/cosmic/man8/update-manager.8.html) para actualizar el sistema operativo.

#### <a name="perform-additional-security-checks"></a>Realizar comprobaciones de seguridad adicionales

Mantenga un alto nivel de seguridad para las imágenes de la solución en Azure Marketplace. Para ver una lista de comprobación de las configuraciones y procedimientos de seguridad, consulte [Recomendaciones de seguridad para imágenes de Azure Marketplace](https://docs.microsoft.com/azure/security/security-recommendations-azure-marketplace-images). Algunas de estas recomendaciones son específicas de imágenes basadas en Linux, pero la mayoría se aplica a cualquier imagen de máquina virtual.

#### <a name="perform-custom-configuration-and-scheduled-tasks"></a>Realizar una configuración personalizada y tareas programadas

Si necesita alguna configuración adicional, use una tarea programada que se ejecute al inicio para realizar cambios finales en la máquina virtual una vez implementada. Tenga en cuenta lo siguiente:

- Si es una tarea de una ejecución, la tarea debe eliminarse a sí misma una vez que se realice correctamente.
- Las configuraciones no deben basarse en unidades distintas de C o D, ya que solo se garantiza la existencia continua de estas dos unidades (la unidad C es el disco del sistema operativo y la unidad D es el disco local temporal).

Para obtener más información sobre las personalizaciones de Linux, vea [Características y extensiones de las máquinas virtuales para Linux](https://docs.microsoft.com/azure/virtual-machines/extensions/features-linux).

## <a name="generalize-the-image"></a>Generalizar la imagen

Todas las imágenes de Azure Marketplace deben ser reutilizables de forma genérica. Para lograrlo, el VHD del sistema operativo debe estar generalizado, una operación que quita todos los controladores de software y los identificadores específicos de la instancia de una máquina virtual.

### <a name="for-windows"></a>Para Windows

Los discos de sistema operativo Windows se generalizan con la herramienta [sysprep](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview). Si más adelante actualiza o vuelve a configurar el sistema operativo, debe ejecutar sysprep de nuevo.

> [!WARNING]
> Después de ejecutar sysprep, desactive la máquina virtual hasta que se implemente, ya que las actualizaciones pueden ejecutarse automáticamente. Este apagado evita que las actualizaciones posteriores realicen cambios específicos de la instancia en el sistema operativo o los servicios instalados. Para más información sobre la ejecución de sysprep, vea [Pasos para generalizar un disco duro virtual](https://docs.microsoft.com/azure/virtual-machines/windows/capture-image-resource#generalize-the-windows-vm-using-sysprep).

### <a name="for-linux"></a>Para Linux

El siguiente proceso generaliza una máquina virtual Linux y la vuelve a implementar como una máquina virtual independiente. Para obtener información detallada, vea [Creación de una imagen de una máquina virtual o un disco duro virtual](https://docs.microsoft.com/azure/virtual-machines/linux/capture-image). Puede detenerse cuando llegue a la sección denominada "Crear una máquina virtual a partir de la imagen capturada".

1. Quitar el agente de Linux de Azure

    1. Conexión a una máquina virtual Linux de Azure mediante un cliente SSH
    2. En la ventana de SSH, escriba el siguiente comando: `sudo waagent –deprovision+user`.
    3. Escriba Y para continuar (puede agregar el parámetro -force al comando anterior para evitar el paso de confirmación).
    4. Una vez completado el comando, escriba Salir para cerrar el cliente de SSH.

2. Detener una máquina virtual

    1. En Azure Portal, seleccione el grupo de recursos (RG) y anule la asignación de la máquina virtual.
    2. El disco duro virtual ahora se ha generalizado y puede crear una máquina virtual mediante este disco duro virtual.

## <a name="next-steps"></a>Pasos siguientes

- Si tiene dificultades para crear el disco duro virtual basado en Azure, vea [Common issues during VHD creation](common-issues-during-vhd-creation.md) (Problemas comunes durante la creación de discos duros virtuales).
- De lo contrario, el artículo [Prueba de máquina virtual implementada desde un disco duro virtual](azure-vm-image-certification.md) explica cómo probar y enviar una imagen de máquina virtual para la certificación de Azure Marketplace, incluido dónde obtener la herramienta de prueba de certificación para la herramienta Azure Certified y cómo usarla para certificar la imagen de máquina virtual.
