---
title: Instalación del agente de Microsoft Azure Recovery Services (MARS)
description: Aprenda a instalar el agente de Microsoft Azure Recovery Services (MARS) para realizar copias de seguridad de máquinas Windows.
ms.topic: conceptual
ms.date: 03/03/2020
ms.openlocfilehash: 715153d445acbc372f8305ca39f5276bf8a39773
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533485"
---
# <a name="install-the-azure-backup-mars-agent"></a>Instalación del agente de MARS de Azure Backup

En este artículo se explica cómo instalar el agente de Microsoft Azure Recovery Services (MARS). MARS también se conoce como agente de Azure Backup.

## <a name="about-the-mars-agent"></a>Acerca del agente de MARS

Azure Backup utiliza el agente de MARS para realizar copias de seguridad de archivos, carpetas y el estado del sistema de máquinas locales y máquinas virtuales de Azure. Esas copias de seguridad se almacenan en un almacén de Recovery Services en Azure. Puede ejecutar el agente:

* Directamente en máquinas Windows locales. Se pueden realizar copias de seguridad de estas máquinas directamente en un almacén de Recovery Services en Azure.
* En máquinas virtuales de Azure que ejecutan Windows en paralelo con la extensión de copia de seguridad de máquina virtual de Azure. El agente realiza copias de seguridad de archivos y carpetas específicos de la máquina virtual.
* En una instancia de Microsoft Azure Backup Server (MABS) o en un servidor System Center Data Protection Manager (DPM). En este escenario, las copias de seguridad de las máquinas y cargas de trabajo se crean en MABS o Data Protection Manager. A continuación, MABS o Data Protection Manager usa el agente de MARS para realizar una copia de seguridad en un almacén de Azure.

Los datos disponibles para la copia de seguridad dependen de la ubicación en la que está instalado el agente.

> [!NOTE]
> Por lo general, una copia de seguridad de una máquina virtual de Azure se realiza mediante una extensión de Azure Backup en la máquina virtual. Este método realiza la copia de seguridad de toda la máquina virtual. Si desea realizar una copia de seguridad de archivos y carpetas específicos de la máquina virtual, instale y use el agente de MARS junto con la extensión. Para más información, consulte [Arquitectura de una copia de seguridad de máquina virtual de Azure integrada](backup-architecture.md#architecture-built-in-azure-vm-backup).

![Pasos del proceso de copia de seguridad](./media/backup-configure-vault/initial-backup-process.png)

## <a name="before-you-start"></a>Antes de comenzar

* Obtenga información acerca de cómo [Azure Backup usa el agente de MARS para realizar copias de seguridad de máquinas Windows](backup-architecture.md#architecture-direct-backup-of-on-premises-windows-server-machines-or-azure-vm-files-or-folders).
* Obtenga información acerca de la [arquitectura de copia de seguridad](backup-architecture.md#architecture-back-up-to-dpmmabs) que ejecuta el agente de MARS en un servidor MABS secundario o un servidor Data Protection Manager.
* Revise [qué admite y de qué puede hacer una copia de seguridad](backup-support-matrix-mars-agent.md) el agente de MARS.
* Si necesita realizar una copia de seguridad de un servidor o cliente en Azure, asegúrese de que tiene una cuenta de Azure. Si no tiene ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/) en tan solo unos minutos.
* Compruebe el acceso a Internet en las máquinas de las que desee realizar copias de seguridad.
* Asegúrese de que el usuario que llevará a cabo la instalación y la configuración del agente de MARS tiene privilegios de administrador local en el servidor que se va a proteger.

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="modify-storage-replication"></a>Modificar la replicación de almacenamiento

De forma predeterminada, los almacenes usan el [almacenamiento con redundancia geográfica (GRS)](../storage/common/storage-redundancy.md).

* Si el almacén es su mecanismo principal de copia de seguridad, le recomendamos que use GRS.
* Puede usar [almacenamiento con redundancia local (LRS)](../storage/common/storage-redundancy.md?toc=/azure/storage/blobs/toc.json) para reducir los costos de almacenamiento de Azure.

Para modificar el tipo de replicación de almacenamiento:

1. En la sección **Configuración** del nuevo almacén, seleccione **Propiedades**.

1. En la hoja **Propiedades**, haga clic en **Actualizar** en la hoja **Configuración de copia de seguridad**.

1. Seleccione el tipo de replicación almacenamiento y seleccione **Guardar**.

    ![Actualización de la configuración de copia de seguridad](./media/backup-afs/backup-configuration.png)

> [!NOTE]
> No puede modificar el tipo de replicación de almacenamiento después de configurar el almacén y si este contiene elementos de copia de seguridad. Si quiere hacer esto, tiene que volver a crear el almacén.
>

### <a name="verify-internet-access"></a>Comprobación del acceso a Internet

Si la máquina tiene limitado el acceso a Internet, asegúrese de que su configuración de firewall en la máquina o el proxy está establecida para permitir las direcciones URL y direcciones IP:

* URLs
  * `www.msftncsi.com`
  * `*.Microsoft.com`
  * `*.WindowsAzure.com`
  * `*.microsoftonline.com`
  * `*.windows.net`
  * `www.msftconnecttest.com`
* Direcciones IP
  * 20.190.128.0/18
  * 40.126.0.0/18

### <a name="use-azure-expressroute"></a>Uso de Azure ExpressRoute

Puede realizar una copia de seguridad de los datos mediante Azure ExpressRoute con emparejamiento público (disponible para circuitos antiguos) y emparejamiento de Microsoft. La copia de seguridad por emparejamiento privado no se admite.

Para usar el emparejamiento público, asegúrese primero de tener acceso a los siguientes dominios y direcciones:

* `http://www.msftncsi.com/ncsi.txt`
* `http://www.msftconnecttest.com/connecttest.txt`
* `microsoft.com`
* `.WindowsAzure.com`
* `.microsoftonline.com`
* `.windows.net`

Para usar el emparejamiento de Microsoft, seleccione los siguientes servicios, regiones y los valores de comunidad correspondientes:

* Azure Active Directory (12076:5060)
* Región de Azure (según la ubicación del almacén de Recovery Services)
* Azure Storage (según la ubicación del almacén de Recovery Services)

Para más información, consulte los [requisitos de enrutamiento de ExpressRoute](../expressroute/expressroute-routing.md).

> [!NOTE]
> El emparejamiento público está en desuso para circuitos nuevos.

Todas las direcciones URL e IP anteriores usan el protocolo HTTPS en el puerto 443.

### <a name="private-endpoints"></a>Puntos de conexión privados

[!INCLUDE [Private Endpoints](../../includes/backup-private-endpoints.md)]

## <a name="download-the-mars-agent"></a>Descarga del agente de MARS

Descargue el agente de MARS para poder instalarlo en las máquinas de las que desea realizar copias de seguridad.

Si ya ha instalado el agente en alguna máquina, asegúrese de que está ejecutando la versión más reciente del agente. Busque la versión más reciente en el portal o vaya directamente a la [descarga](https://aka.ms/azurebackup_agent).

1. En el almacén, en **Introducción**, seleccione **Copia de seguridad**.

    ![Abrir el objetivo de copia de seguridad](./media/backup-try-azure-backup-in-10-mins/open-backup-settings.png)

1. En **¿Dónde se ejecuta su carga de trabajo?** , seleccione **Local**. Seleccione esta opción incluso si desea instalar el agente de MARS en una máquina virtual de Azure.
1. En **¿De qué desea hacer una copia de seguridad?** , seleccione **Archivos y carpetas**. También puede seleccionar **Estado del sistema**. Hay varias opciones disponibles más, pero solo se admiten si ejecuta un servidor de copia de seguridad secundario. Seleccione **Preparar infraestructura**.

    ![Configuración de archivos y carpetas](./media/backup-try-azure-backup-in-10-mins/set-file-folder.png)

1. En **Preparar infraestructura**, en **Instalar el agente de Recovery Services**, descargue el agente de MARS.

    ![Preparación de la infraestructura](./media/backup-try-azure-backup-in-10-mins/choose-agent-for-server-client.png)

1. En el menú de descargas, seleccione **Guardar**. De forma predeterminada, se guarda el archivo *MARSagentinstaller.exe* en la carpeta de descargas.

1. Seleccione **Ya se ha descargado o se usa el agente de Recovery Services más reciente** y, a continuación, descargue las credenciales del almacén.

    ![Descarga de las credenciales de almacén](./media/backup-try-azure-backup-in-10-mins/download-vault-credentials.png)

1. Seleccione **Guardar**. El archivo se descargará en la carpeta Descargas. El archivo de credenciales del almacén no se puede abrir.

## <a name="install-and-register-the-agent"></a>Instalación y registro del agente

1. Ejecute el archivo *MARSagentinstaller.exe* en las máquinas de las que desea realizar una copia de seguridad.
1. En el Asistente para la instalación del agente de MARS, seleccione **Configuración de la instalación**. Una vez ahí, especifique dónde desea instalar el agente y una ubicación para la memoria caché. Luego, seleccione **Siguiente**.
   * Azure Backup usa la memoria caché para almacenar instantáneas de datos antes de enviarlas a Azure.
   * La ubicación de la memoria caché debe tener un espacio libre igual a al menos el 5 % del tamaño de los datos de los que se va a realizar la copia de seguridad.

    ![Selección de la configuración de la instalación en el Asistente para la instalación del agente de MARS](./media/backup-configure-vault/mars1.png)

1. En **Configuración del proxy**, especifique cómo se conectará el agente que se ejecuta en la máquina Windows a Internet. Luego, seleccione **Siguiente**.

   * Si utiliza un proxy personalizado, especifique la configuración del proxy y las credenciales necesarias.
   * Recuerde que el agente necesita acceso a [direcciones URL concretas](#before-you-start).

    ![Configuración del acceso a Internet en el Asistente para MARS](./media/backup-configure-vault/mars2.png)

1. En **Instalación**, compruebe los requisitos previos y seleccione **Instalar**.
1. Una vez instalado el agente, seleccione **Proceder al registro**.
1. En el **Asistente para registrar servidor** > **Identificación del almacén**, busque y seleccione el archivo de credenciales descargado. Luego, seleccione **Siguiente**.

    ![Adición de credenciales de almacén mediante el Asistente para registrar servidor](./media/backup-configure-vault/register1.png)

1. En la página **Configuración de cifrado**, especifique una frase de contraseña que se usará para cifrar y descifrar copias de seguridad de la máquina. [Consulte aquí](backup-azure-file-folder-backup-faq.md#what-characters-are-allowed-for-the-passphrase) más información sobre los caracteres de frase de contraseña permitidos.

    * Guarde la frase de contraseña en una ubicación segura. Es necesario para restaurar una copia de seguridad.
    * Si pierde u olvida la frase de contraseña, Microsoft no podrá ayudarle a recuperar los datos de copia de seguridad.

1. Seleccione **Finalizar**. Ahora está instalado el agente y el equipo está registrado en el almacén. Está listo para configurar y programar la copia de seguridad.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre la [Copia de seguridad de máquinas Windows con el agente de MARS de Azure Backup](backup-windows-with-mars-agent.md)
