---
title: Preparación del servidor DPM para realizar copias de seguridad de las cargas de trabajo
description: En este artículo, aprenderá a preparar las copias de seguridad de System Center Data Protection Manager (DPM) en Azure con el servicio Azure Backup.
ms.topic: conceptual
ms.date: 06/11/2020
ms.openlocfilehash: 4534b75018fb91cedda00b8c1167012858c5e562
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021041"
---
# <a name="prepare-to-back-up-workloads-to-azure-with-system-center-dpm"></a>Preparación para realizar copias de seguridad de las cargas de trabajo en Azure con System Center DPM

En este artículo se explica cómo preparar las copias de seguridad de System Center Data Protection Manager (DPM) en Azure con el servicio Azure Backup.

El artículo ofrece:

- Información general de la implementación de DPM con Azure Backup.
- Requisitos previos y limitaciones para usar Azure Backup con DPM.
- Pasos para preparar Azure, incluida la configuración de un almacén de copia de seguridad de Recovery Services y, opcionalmente, la modificación del tipo de almacenamiento de Azure para el almacén.
- Pasos para preparar el servidor DPM, incluida la descarga de las credenciales del almacén, la instalación del agente de Azure Backup y el registro del servidor DPM en el almacén.
- Consejos para la solución de errores comunes.

## <a name="why-back-up-dpm-to-azure"></a>¿Por qué realizar copias de seguridad de DPM en Azure?

Datos de aplicación y archivo de copia de seguridad de [System Center DPM](/system-center/dpm/dpm-overview). DPM interactúa con Azure Backup de la manera siguiente:

- **Si DPM se ejecuta en un servidor físico o en una máquina virtual local**: puede realizar una copia de seguridad de los datos en un almacén de copia de seguridad en Azure, además de una copia de seguridad en disco y en cinta.
- **Si DPM se ejecuta en una máquina virtual de Azure**: desde System Center 2012 R2 con Update 3 o posterior, puede implementar DPM en una máquina virtual de Azure. Puede realizar una copia de seguridad de los datos en discos de Azure conectados a la máquina virtual, o bien usar Azure Backup para realizar una copia de seguridad de los datos en un almacén de Backup.

Las ventajas empresariales de realizar una copia de seguridad de servidores DPM en Azure son las siguientes:

- En implementaciones de DPM locales, Azure Backup ofrece una alternativa a la implementación a largo plazo en una cinta.
- Si DPM se ejecuta en una máquina virtual de Azure, Azure Backup permite descargar el almacenamiento del disco de Azure. Almacenar los datos más antiguos en el almacén de Backup le permite escalar verticalmente su negocio gracias al almacenamiento de nuevos datos en el disco.

## <a name="prerequisites-and-limitations"></a>Requisitos previos y limitaciones

**Configuración** | **Requisito**
--- | ---
DPM en una máquina virtual de Azure | Paquete acumulativo de actualizaciones 3 de System Center 2012 R2 con DPM 2012 R2 o posterior.
DPM en un servidor físico | System Center 2012 SP1 o posterior; System Center 2012 R2.
DPM en una máquina virtual Hyper-V | System Center 2012 SP1 o posterior; System Center 2012 R2.
DPM en una máquina virtual de VMware | System Center 2012 R2 con el paquete acumulativo de actualizaciones 5 o posterior.
Componentes | El servidor DPM debe tener instalado Windows PowerShell y .NET Framework 4.5.
Aplicaciones admitidas | [Obtenga información](/system-center/dpm/dpm-protection-matrix) sobre los elementos de los que DPM puede realizar una copia de seguridad.
Tipos de archivo admitidos | La copia de seguridad de estos tipos de archivos se puede realizar con Azure Backup:<br> <li>Cifrados (solo copias de seguridad completas)<li> Comprimidos (copias de seguridad incrementales compatibles) <li> Dispersos (copias de seguridad incrementales compatibles)<li> Comprimidos y dispersos (tratados como dispersos)
Tipo de archivo no admitidos | <li>Servidores en sistemas de archivos que distinguen mayúsculas de minúsculas<li> Vínculos físicos (omitidos)<li> Puntos de reanálisis (omitidos)<li> Cifrados y comprimidos (omitidos)<li> Cifrados y dispersos (omitidos)<li> Flujo comprimido<li> Secuencia de análisis
Almacenamiento local | Cada máquina de la que quiera realizar una copia de seguridad debe tener almacenamiento libre local de al menos el 5 % del tamaño de los datos cuya copia se va a hacer. Por ejemplo, realizar una copia de seguridad de 100 GB de datos requiere un mínimo de 5 GB de espacio libre en la ubicación temporal.
Almacenamiento de almacén | No hay ningún límite en la cantidad de datos de los que puede realizar una copia de seguridad en el almacén de Azure Backup; sin embargo, el tamaño de un origen de datos (por ejemplo, una máquina virtual o una base de datos) no debe superar los 54 400 GB.
Azure ExpressRoute | Puede realizar una copia de seguridad de los datos mediante Azure ExpressRoute con emparejamiento público (disponible para circuitos antiguos) y emparejamiento de Microsoft. La copia de seguridad por emparejamiento privado no se admite.<br/><br/> **Con el emparejamiento público**: asegúrese de tener acceso a los siguientes dominios y direcciones:<br/><br/>- `http://www.msftncsi.com/ncsi.txt` <br/><br/>- `microsoft.com` <br/><br/>-`.WindowsAzure.com`<br/><br/>-`.microsoftonline.com`<br/><br/>-`.windows.net`<br/><br/> **Con el emparejamiento de Microsoft**, seleccione los siguientes servicios o regiones y los valores de comunidad correspondientes:<br/><br/>- Azure Active Directory (12076:5060)<br/><br/>- Región de Microsoft Azure (según la ubicación del almacén de Recovery Services)<br/><br/>- Azure Storage (según la ubicación del almacén de Recovery Services)<br/><br/>Para más información, consulte los [requisitos de enrutamiento de ExpressRoute](../expressroute/expressroute-routing.md).<br/><br/>**Nota**: El emparejamiento público está en desuso para circuitos nuevos.
Agente de Azure Backup | Si DPM se ejecuta en System Center 2012 SP1, instale el paquete acumulativo de actualizaciones 2 o posterior para DPM SP1. Esto es necesario para la instalación del agente.<br/><br/> En este artículo se describe cómo implementar la última versión del agente de Azure Backup, también conocido como el agente de Microsoft Azure Recovery Service (MARS). Si tiene una versión anterior implementada, actualice a la última versión para asegurarse de que la copia de seguridad funciona según lo previsto.

Antes de empezar, necesitará una cuenta de Azure con la característica Azure Backup habilitada. En caso de no tener cuenta, puede crear una de evaluación gratuita en tan solo unos minutos. Lea acerca de los [Precios de Backup](https://azure.microsoft.com/pricing/details/backup/).

[!INCLUDE [backup-create-rs-vault.md](../../includes/backup-create-rs-vault.md)]

## <a name="modify-storage-settings"></a>Modificación de la configuración de almacenamiento

Puede elegir entre almacenamiento con redundancia geográfica y almacenamiento con redundancia local.

- De forma predeterminada, el almacén tiene almacenamiento con redundancia geográfica.
- Si el almacén es su copia de seguridad principal, deje la opción establecida como almacenamiento con redundancia geográfica. Si desea una opción más económica que no sea tan duradera, use el siguiente procedimiento para configurar el almacenamiento con redundancia local.
- Obtenga información sobre [Azure Storage](../storage/common/storage-redundancy.md), y las opciones de almacenamiento con [redundancia geográfica](../storage/common/storage-redundancy.md) y [redundancia local](../storage/common/storage-redundancy.md).
- Modifique la configuración de almacenamiento antes de la copia de seguridad inicial. Si ya ha realizado la copia de seguridad de un elemento, detenga la copia de seguridad en el almacén antes de modificar la configuración de almacenamiento.

Para editar la configuración de replicación de almacenamiento:

1. Abra el panel del almacén.

2. En **Administrar**, seleccione **Infraestructura de Backup**.

3. En el menú **Configuración de copia de seguridad**, seleccione una opción de almacenamiento para el almacén.

    ![Lista de copias de seguridad](./media/backup-azure-dpm-introduction/choose-storage-configuration-rs-vault.png)

## <a name="download-vault-credentials"></a>Descarga de las credenciales de almacén

Use las credenciales del almacén cuando registre el servidor DPM en el almacén.

- El archivo de credenciales de almacén es un certificado generado por el portal para cada almacén de copia de seguridad.
- Luego el portal carga la clave pública en Access Control Service (ACS).
- Durante el flujo de trabajo del registro de la máquina, la clave privada del certificado se pone a disposición del usuario, que autentica la máquina.
- En función de la autenticación, el servicio Azure Backup envía los datos al almacén identificado.

### <a name="best-practices-for-vault-credentials"></a>Procedimientos recomendados para las credenciales del almacén

Para obtener las credenciales, descargue el archivo de credenciales del almacén mediante un canal seguro desde Azure Portal:

- Las credenciales del almacén se usan solo durante el flujo de trabajo de registro.
- Es su responsabilidad asegurarse de que el archivo de credenciales del almacén sea seguro y no se vea comprometido.
  - Si se pierde el control de las credenciales, se pueden usar las credenciales de almacén para registrar otras máquinas en el almacén.
  - Sin embargo, los datos de copia de seguridad se cifran mediante una frase de contraseña que pertenece al cliente, por lo que los datos de copia de seguridad existentes no pueden estar en peligro.
- Asegúrese de que el archivo se guarda en una ubicación a la que se pueda acceder desde el servidor DPM. Si se almacenan en un recurso compartido o SMB de archivo, compruebe los permisos de acceso.
- Las credenciales de almacén expiran a las 48 horas. Puede descargar las nuevas credenciales del almacén tantas veces como sea necesario. Sin embargo, se puede usar solo el último archivo de credenciales de almacén durante el flujo de trabajo de registro.
- El servicio Azure Backup no conoce la clave privada del certificado, y la clave privada no está disponible en el portal ni en el servicio.

Descargue el archivo de credenciales del almacén en una máquina local como se indica a continuación:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Abra el almacén en el que desea registrar el servidor DPM.
3. En **Configuración**, seleccione **Propiedades**.

    ![Abrir el menú Almacén](./media/backup-azure-dpm-introduction/vault-settings-dpm.png)

4. En **Propiedades** > **Credenciales de Backup**, seleccione **Descargar**. El portal genera el archivo de credenciales del almacén mediante una combinación de la fecha actual y el nombre del almacén y hace que esté disponible para su descarga.

    ![Descargar credenciales](./media/backup-azure-dpm-introduction/vault-credentials.png)

5. Seleccione **Guardar** para descargar las credenciales del almacén en la carpeta o en **Guardar como** y especifique una ubicación. El archivo tarda un minuto en generarse.

## <a name="install-the-backup-agent"></a>Instalación del agente de copia de seguridad

Todas las máquinas cuya copia de seguridad se realiza con Azure Backup deben tener instalado el agente de copia de seguridad, también conocido como Microsoft Azure Recovery Service (agente MARS). Instale al agente en el servidor DPM como sigue:

1. Abra el almacén en el que desea registrar el servidor DPM.
2. En **Configuración**, seleccione **Propiedades**.

    ![Configuración para abrir el almacén](./media/backup-azure-dpm-introduction/vault-settings-dpm.png)
3. En la página **Propiedades**, descargue el agente de Azure Backup.

    ![Descargar](./media/backup-azure-dpm-introduction/azure-backup-agent.png)

4. Después de la descarga, ejecute MARSAgentInstaller.exe para instalar el agente en la máquina de DPM.
5. Seleccione una carpeta de instalación y una carpeta de caché para el agente. El espacio libre de la ubicación de caché debe ser al menos del 5 % de los datos de copia de seguridad.
6. Si usa un servidor proxy para conectarse a Internet, en la pantalla **Configuración de proxy** , especifique los detalles del servidor proxy. Si utiliza a un servidor proxy autenticado, escriba los detalles de nombre y la contraseña del usuario en esta pantalla.
7. El agente de Azure Backup instala .NET Framework 4.5 y Windows PowerShell (si todavía no están instalados) para completar la instalación.
8. Una vez instalado el agente, **cierre** la ventana.

    ![Cerrar](../../includes/media/backup-install-agent/dpm_FinishInstallation.png)

## <a name="register-the-dpm-server-in-the-vault"></a>Registro del servidor DPM en el almacén

1. En la consola del administrador de DPM > **Administración**, seleccione **En línea**. Seleccione **Registrar**. Se abrirá al Asistente para registrar el servidor.
2. En **Configuración de proxy**, especifique la configuración de proxy según sea necesario.

    ![Configuración de proxy](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_Proxy.png)
3. En **Almacén de Backup**, busque y seleccione el archivo de credenciales del almacén que descargó.

    ![Credenciales de almacén](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_Credentials.jpg)

4. En **Configuración de límite**, opcionalmente puede habilitar el límite de ancho de banda para las copias de seguridad. Puede establecer los límites de velocidad para especificar los días y las horas de trabajo.

    ![Configuración de límite](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_Throttling.png)

5. En **Recovery Folder Setting** (Configuración de la carpeta de recuperación), especifique una ubicación que se pueda usar durante la recuperación de datos.

    - Azure Backup utiliza esta ubicación como un área de almacenamiento temporal para los datos recuperados.
    - Una vez finalizada la recuperación de datos, Azure Backup limpiará los datos de esta área.
    - La ubicación debe tener espacio suficiente para contener los elementos que espera recuperar en paralelo.

    ![Recovery Folder Setting](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_RecoveryFolder.png)

6. En **Configuración de cifrado**, genere o proporcione una frase de contraseña.

    - La frase de contraseña se utiliza para cifrar las copias de seguridad en la nube.
    - Especifique un mínimo de 16 caracteres.
    - Guarde el archivo en una ubicación segura; es necesario para la recuperación.

    ![Cifrado](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_Encryption.png)

    > [!WARNING]
    > El usuario posee la frase de contraseña de cifrado y Microsoft no puede verla.
    > Si la frase de contraseña se pierde u olvida, Microsoft no puede ayudar a recuperar los datos de copia de seguridad.

7. Seleccione **Registrar** para registrar el servidor DPM en el almacén.

Una vez que el servidor se ha registrado correctamente en el almacén, ya está listo para iniciar la copia de seguridad en Microsoft Azure. Deberá configurar el grupo de protección en la consola de DPM para realizar copias de seguridad de las cargas de trabajo en Azure. [Obtenga información](/system-center/dpm/create-dpm-protection-groups?view=sc-dpm-2019) sobre cómo implementar grupos de protección.

## <a name="troubleshoot-vault-credentials"></a>Solución de problemas de las credenciales del almacén

### <a name="expiration-error"></a>Error de expiración

El archivo de almacén de credenciales solo es válido durante 48 horas (después de descargarlo desde el portal). Si encuentra algún error en esta pantalla (por ejemplo "El archivo de credenciales de almacén especificado expiró"), inicie sesión en Azure Portal y vuelva a descargar el archivo de credenciales de almacén.

### <a name="access-error"></a>Error de acceso

Asegúrese de que el archivo de credenciales del almacén está disponible en alguna ubicación a la que pueda acceder la aplicación de instalación. Si encuentra errores relacionados con el acceso, copie el archivo de almacén de credenciales en una ubicación temporal en esta máquina y vuelva a intentar la operación.

### <a name="invalid-credentials-error"></a>Error de credenciales no válidas

Si se produce un error de credenciales de almacén no válidas (por ejemplo, "Las credenciales del almacén no son válidas"), el archivo está dañado o no tiene asociadas las credenciales más recientes con el servicio de recuperación.

- Vuelva a intentar la operación después de descargar un nuevo archivo de credenciales de almacén desde el portal.
- Este error suele aparecer si se selecciona la opción **Descargar credenciales de almacén** en Azure Portal, dos veces en sucesión rápida. En este caso, solo es válido el segundo archivo de credenciales de almacén.
