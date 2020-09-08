---
title: Matriz de compatibilidad para el agente de MARS
description: En este artículo se resume la compatibilidad con Azure Backup al realizar copias de seguridad de máquinas que ejecutan el agente de Microsoft Azure Recovery Services (MARS).
ms.date: 08/30/2019
ms.topic: conceptual
ms.openlocfilehash: 2b719bd36c27336b3fe24cdb904715bf8194ed70
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87872419"
---
# <a name="support-matrix-for-backup-with-the-microsoft-azure-recovery-services-mars-agent"></a>Matriz de compatibilidad para la copia de seguridad con el agente de Microsoft Azure Recovery Services (MARS)

Puede usar el [servicio Azure Backup](backup-overview.md) para realizar copias de seguridad de aplicaciones y máquinas locales, y de las máquinas virtuales de Azure. En este artículo se resumen las configuraciones de compatibilidad y las limitaciones de uso del agente de Microsoft Azure Recovery Services (MARS) para realizar copias de seguridad de máquinas.

## <a name="the-mars-agent"></a>El agente de MARS

Azure Backup utiliza el agente de MARS para realizar copias de seguridad de máquinas locales y máquinas virtuales de Azure en un almacén de Recovery Services alternativo en Azure. El agente de MARS puede realizar las siguientes acciones:

- Ejecutarse en máquinas Windows locales, para que pueda realizar la copia de seguridad directamente en un almacén de Recovery Services alternativo en Azure.
- Ejecutarse en máquinas virtuales de Azure con Windows para que pueda realizar copias de seguridad directamente en un almacén.
- Ejecutarse en un servidor de Microsoft Azure Backup Server (MABS) o en un servidor de System Center Data Protection Manager (DPM). En este escenario, las copias de seguridad de las máquinas y cargas de trabajo se crean en MABS o en el servidor DPM. El agente de MARS, a continuación, realiza una copia de seguridad de este servidor en un almacén de Azure.

> [!NOTE]
>Azure Backup no admite el ajuste automático del reloj para el horario de verano (DST). Modifique la directiva para asegurarse de que se tenga en cuenta el horario de verano para evitar discrepancias entre el tiempo real y el tiempo de copia de seguridad programado.

Las opciones de copia de seguridad dependen de la ubicación en que está instalado el agente. Para obtener más información, consulte [arquitectura de Azure Backup mediante el agente de MARS](backup-architecture.md#architecture-direct-backup-of-on-premises-windows-server-machines-or-azure-vm-files-or-folders). Para obtener información sobre la arquitectura de copia de seguridad de MABS y DPM, consulte [Copia de seguridad en DPM o MABS](backup-architecture.md#architecture-back-up-to-dpmmabs). Consulte también los [requisitos](backup-support-matrix-mabs-dpm.md) de la arquitectura de copia de seguridad.

**Instalación** | **Detalles**
--- | ---
Descarga de la última versión del agente de MARS | Puede descargar la última versión del agente desde el almacén o [directamente](https://aka.ms/azurebackup_agent).
Instalación directa en una máquina | Puede instalar el agente de MARS directamente en un servidor Windows local o en una máquina virtual con Windows que ejecute cualquiera de los [sistemas operativos compatibles](./backup-support-matrix-mabs-dpm.md#supported-mabs-and-dpm-operating-systems).
Instalación en un servidor de copia de seguridad | Al configurar DPM o MABS para realizar una copia de seguridad en Azure, descargue e instale al agente de MARS en el servidor. Puede instalar el agente en los [sistemas operativos compatibles](backup-support-matrix-mabs-dpm.md#supported-mabs-and-dpm-operating-systems) de la matriz de compatibilidad del servidor de copia de seguridad.

> [!NOTE]
> De forma predeterminada, las máquinas virtuales de Azure habilitadas para copia de seguridad tienen instalada una extensión de Azure Backup. Esta extensión realiza una copia de seguridad de toda la máquina virtual. Puede instalar y ejecutar al agente de MARS en una máquina virtual de Azure junto con la extensión si desea realizar copias de seguridad de carpetas y archivos específicos y no de la máquina virtual completa.
> Cuando ejecuta el agente de MARS en una máquina virtual de Azure, este realiza copias de seguridad de archivos o carpetas ubicados en el almacenamiento temporal de la máquina virtual. Se produce un error de la copia de seguridad si los archivos o las carpetas se eliminan del almacenamiento temporal, o si se elimina dicho almacenamiento temporal.

## <a name="cache-folder-support"></a>Compatibilidad con la carpeta de caché

Cuando usa el agente de MARS para crear copias de seguridad de los datos, este realiza una instantánea de los datos y los almacena en una carpeta de caché local antes de enviarlos a Azure. La carpeta de caché (vacía) tiene varios requisitos:

**Memoria caché** | **Detalles**
--- | ---
Size |  El espacio libre en la carpeta de caché debe ser de entre un 5 % y un 10 % del tamaño total de los datos de copia de seguridad.
Location | La carpeta de caché debe almacenarse de forma local en la máquina de la copia de seguridad y debe estar en línea. La carpeta de caché no debe encontrarse en un recurso compartido de red, en un soporte físico extraíble o en un volumen sin conexión.
Carpeta | La carpeta de caché no debe estar cifrada en un volumen desduplicado o en una carpeta comprimida, dispersa o con un punto de reanálisis.
Cambios de ubicación | Puede cambiar la ubicación de la caché al detener el motor de copia de seguridad (`net stop bengine`) y copiar la carpeta de caché en una nueva unidad. (Asegúrese de que esta tiene espacio suficiente). A continuación, actualice dos entradas del Registro en **HKLM\SOFTWARE\Microsoft\Windows Azure Backup** (**Config/ScratchLocation** and **Config/CloudBackupProvider/ScratchLocation**) en la nueva ubicación y reinicie el motor.

## <a name="networking-and-access-support"></a>Compatibilidad con redes y acceso

### <a name="url-and-ip-access"></a>Acceso a direcciones URL e IP

El agente de MARS necesita acceder a estas direcciones URL:

- `http://www.msftncsi.com/ncsi.txt`
- *.Microsoft.com
- *.WindowsAzure.com
- *.MicrosoftOnline.com
- *.Windows.net

Y a estas direcciones IP:

- 20.190.128.0/18
- 40.126.0.0/18

El acceso a todas las direcciones URL y direcciones IP enumeradas anteriormente usa el protocolo HTTPS en el puerto 443.

### <a name="azure-expressroute-support"></a>Compatibilidad con Azure ExpressRoute

Puede realizar una copia de seguridad de los datos mediante Azure ExpressRoute con emparejamiento público (disponible para circuitos antiguos) y emparejamiento de Microsoft. La copia de seguridad por emparejamiento privado no se admite.

Con el emparejamiento público: asegúrese de tener acceso a los siguientes dominios y direcciones:

- `http://www.msftncsi.com/ncsi.txt`
- `microsoft.com`
- `.WindowsAzure.com`
- `.microsoftonline.com`
- `.windows.net`

Con el emparejamiento de Microsoft, seleccione los siguientes servicios o regiones y los valores de comunidad correspondientes:

- Azure Active Directory (12076:5060)
- Región de Microsoft Azure (según la ubicación del almacén de Recovery Services)
- Azure Storage (según la ubicación del almacén de Recovery Services)

Para más información, consulte los [requisitos de enrutamiento de ExpressRoute](../expressroute/expressroute-routing.md).

>[!NOTE]
>El emparejamiento público está en desuso para circuitos nuevos.

### <a name="private-endpoint-support"></a>Compatibilidad con el punto de conexión privado

Ahora puede usar puntos de conexión privados para realizar copias de seguridad de los datos de los servidores al almacén de Recovery Services. Como Azure Active Directory no es compatible actualmente con puntos de conexión privados, las direcciones IP y los FQDN necesarios para Azure Active Directory tendrán que permitir el acceso de salida por separado.

Al usar el agente de MARS para realizar una copia de seguridad de los recursos locales, asegúrese de que la red local (que contiene los recursos de los que se va a realizar la copia de seguridad) está emparejada con la red virtual de Azure que contiene un punto de conexión privado para el almacén. Después, puede continuar con la instalación del agente de MARS y configurar la copia de seguridad. Sin embargo, debe asegurarse de que toda la comunicación para la copia de seguridad se produzca solo a través de la red emparejada.

Si quita los puntos de conexión privados del almacén después de haber registrado un agente de MARS, deberá volver a registrar el contenedor con el almacén. No es necesario detener la protección de los mismos.

Obtenga más información sobre los [puntos de conexión privados para Azure Backup](private-endpoints.md).

### <a name="throttling-support"></a>Limitaciones de compatibilidad

**Característica** | **Detalles**
--- | ---
Control del ancho de banda | Compatible. En el agente de MARS, use **Cambiar propiedades** para ajustar el ancho de banda.
Limitación de la red | No está disponible para las máquinas de copia de seguridad que ejecutan Windows Server 2008 R2, Windows Server 2008 SP2 o Windows 7.

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

>[!NOTE]
> El agente de MARS no es compatible con las SKU de Windows Server Core.

Puede usar el agente de MARS para realizar copias de seguridad directamente en Azure en los sistemas operativos que se enumeran a continuación y que se ejecutan en:

1. Servidores Windows locales
2. Máquinas virtuales de Azure que ejecutan Windows

Los sistemas operativos deben ser de 64 bits y ejecutar los Service Pack y actualizaciones más recientes. En la tabla siguiente se resumen estos sistemas operativos:

**Sistema operativo** | **Archivos/carpetas** | **Estado del sistema** | **Requisitos de software o módulo**
--- | --- | --- | ---
Windows 10 (Enterprise, Pro, Home) | Sí | No |  Comprobar la versión de servidor correspondiente para los requisitos de software o módulo
Windows 8.1 (Enterprise, Pro)| Sí |No | Comprobar la versión de servidor correspondiente para los requisitos de software o módulo
Windows 8 (Enterprise, Pro) | Sí | No | Comprobar la versión de servidor correspondiente para los requisitos de software o módulo
Windows Server 2016 (Standard, Datacenter, Essentials) | Sí | Sí | - .NET 4.5 <br> Windows PowerShell <br> - Versión compatible más reciente de Microsoft VC++ Redistributable <br> - Microsoft Management Console (MMC) 3.0
Windows Server 2012 R2 (Standard, Datacenter, Foundation, Essentials) | Sí | Sí | - .NET 4.5 <br> Windows PowerShell <br> - Versión compatible más reciente de Microsoft VC++ Redistributable <br> - Microsoft Management Console (MMC) 3.0
Windows Server 2012 (Standard, Datacenter, Foundation) | Sí | Sí |- .NET 4.5 <br> -Windows PowerShell <br> - Versión compatible más reciente de Microsoft VC++ Redistributable <br> - Microsoft Management Console (MMC) 3.0 <br> - Administración y mantenimiento de imágenes de implementación (DISM.exe)
Windows Storage Server 2016/2012 R2/2012 (Standard, Workgroup) | Sí | No | - .NET 4.5 <br> Windows PowerShell <br> - Versión compatible más reciente de Microsoft VC++ Redistributable <br> - Microsoft Management Console (MMC) 3.0
Windows Server 2019 (Standard, Datacenter, Essentials) | Sí | Sí | - .NET 4.5 <br> Windows PowerShell <br> - Versión compatible más reciente de Microsoft VC++ Redistributable <br> - Microsoft Management Console (MMC) 3.0

Para obtener más información, consulte el artículo sobre los [Sistemas operativos de MABS y DPM compatibles](backup-support-matrix-mabs-dpm.md#supported-mabs-and-dpm-operating-systems).

### <a name="operating-systems-at-end-of-support"></a>Sistemas operativos al final del soporte técnico

Los siguientes sistemas operativos se encuentran al final del soporte técnico y se recomienda encarecidamente actualizar el sistema operativo para que siga estando protegido.

Si debido a una serie de compromisos existentes no es posible actualizar el sistema operativo, considere la posibilidad de migrar los servidores Windows a máquinas virtuales de Azure y utilice las copias de seguridad de las máquinas virtuales de Azure para seguir estando protegido. Visite la [página de migración](https://azure.microsoft.com/migration/windows-server/) para obtener más información acerca de la migración de servidores Windows.

En el caso de entornos locales u hospedados, donde no puede actualizar el sistema operativo ni migrar a Azure, active las Actualizaciones de seguridad ampliada para que las máquinas sigan estando protegidas. Tenga en cuenta que las Actualizaciones de seguridad ampliada solo son aplicables para algunas ediciones concretas. Visite la [página de preguntas frecuentes](https://www.microsoft.com/windows-server/extended-security-updates) para más información.

| **Sistema operativo**                                       | **Archivos/carpetas** | **Estado del sistema** | **Requisitos de software o módulo**                           |
| ------------------------------------------------------------ | ----------------- | ------------------ | ------------------------------------------------------------ |
| Windows 7 (Ultimate, Enterprise, Pro, Home Premium/Basic, Starter) | Sí               | No                 | Comprobar la versión de servidor correspondiente para los requisitos de software o módulo |
| Windows Server 2008 R2 (Standard, Enterprise, Datacenter, Foundation) | Sí               | Sí                | - .NET 3.5, .NET 4.5 <br>  Windows PowerShell <br>  - Versión compatible de Microsoft VC++ Redistributable <br>  - Microsoft Management Console (MMC) 3.0 <br>  - Administración y mantenimiento de imágenes de implementación (DISM.exe) |
| Windows Server 2008 SP2 (Standard, Datacenter, Foundation)  | Sí               | No                 | - .NET 3.5, .NET 4.5 <br>  Windows PowerShell <br>  - Versión compatible de Microsoft VC++ Redistributable <br>  - Microsoft Management Console (MMC) 3.0 <br>  - Administración y mantenimiento de imágenes de implementación (DISM.exe) <br>  - Base de Virtual Server 2005 + KB KB948515 |

## <a name="backup-limits"></a>Límites de Backup

### <a name="size-limits"></a>Límites de tamaño

Azure Backup limita el tamaño del origen de datos de archivo o carpeta del que se puede realizar una copia de seguridad. Los elementos para los que realiza una copia de seguridad desde un único volumen no pueden exceder los tamaños resumidos en esta tabla:

**Sistema operativo** | **Límite de tamaño**
--- | ---
Windows Server 2012 o superior |54 400 GB
Windows Server 2008 R2 SP1 |1700 GB
Windows Server 2008 SP2| 1700 GB
Windows 8 o posterior| 54 400 GB
Windows 7| 1700 GB

### <a name="other-limitations"></a>Otras limitaciones

- MARS no admite la protección de varias máquinas con el mismo nombre en un único almacén.

## <a name="supported-file-types-for-backup"></a>Tipos de archivo compatibles para copia de seguridad

**Tipo** | **Soporte técnico**
--- | ---
Cifrado<sup>*</sup>| Compatible.
Compressed | Compatible.
Dispersos | Compatible.
Comprimidos y dispersos |Compatible.
Vínculos físicos| No compatible. Se omite.
Punto de repetición de análisis| No compatible. Se omite.
Cifrados y dispersos |No compatible. Se omite.
Flujo comprimido| No compatible. Se omite.
Flujo disperso| No compatible. Se omite.
OneDrive (los archivos sincronizados son flujos dispersos).| No compatible.
Carpetas con la replicación DFS habilitada | No compatible.

\* Asegúrese de que el agente de MARS tenga acceso a los certificados necesarios para acceder a los archivos cifrados. Los archivos inaccesibles se omitirán.

## <a name="supported-drives-or-volumes-for-backup"></a>Unidades o volúmenes compatibles con la copia de seguridad

**Unidad/volumen** | **Soporte técnico** | **Detalles**
--- | --- | ---
Volúmenes de solo lectura| No compatible | El Servicio de instantánea de copia de volumen (VSS) solo funciona si el volumen es grabable.
Volúmenes sin conexión| No compatible |VSS solo funciona si el volumen está en línea.
Recurso compartido de red| No compatible |El volumen debe ser local en el servidor.
Volúmenes bloqueados por BitLocker| No compatible |El volumen debe desbloquearse antes de que se pueda iniciar la copia de seguridad.
Identificación del sistema de archivos| No compatible |Solo se admite NTFS.
Medios extraíbles| No compatible |Todos los orígenes de elementos de copia de seguridad deben tener el estado *corregido*.
Unidades desduplicadas | Compatible | Azure Backup convierte los datos desduplicados en datos normales. Optimiza, cifra, almacena y envía los datos al almacén.

## <a name="support-for-initial-offline-backup"></a>Compatibilidad con copia de seguridad inicial sin conexión

Azure Backup es compatible con la *propagación sin conexión* para transferir datos de copia de seguridad iniciales a Azure mediante discos. Esta compatibilidad resulta útil si es probable que la copia de seguridad inicial esté en el intervalo de tamaño de terabytes (TB). La copia de seguridad sin conexión es compatible para:

- Copia de seguridad directa de archivos y carpetas en máquinas locales que ejecutan el agente de MARS.
- Copia de seguridad de cargas de trabajo y archivos desde un servidor DPM o MABS.

La copia de seguridad sin conexión no se puede usar para archivos de estado del sistema.

## <a name="support-for-data-restoration"></a>Compatibilidad con restauración de datos

Mediante la característica de [restauración instantánea](backup-instant-restore-capability.md) de Azure Backup, puede restaurar los datos antes de copiarlos en el almacén. La máquina para la que realizará la copia de seguridad debe ejecutar .NET Framework 4.5.2 o posterior.

Las copias de seguridad no se pueden restaurar en una máquina de destino en la que se ejecuta una versión anterior del sistema operativo. Por ejemplo, una copia de seguridad perteneciente a un equipo con Windows 7 se puede restaurar en Windows 8 o posterior. Sin embargo, una copia de seguridad perteneciente a un equipo con Windows 8 no se puede restaurar en un equipo con Windows 7.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre la [arquitectura de copia de seguridad que usa el agente de MARS](backup-architecture.md#architecture-direct-backup-of-on-premises-windows-server-machines-or-azure-vm-files-or-folders).
- Obtenga información sobre lo que admite cuando [ejecuta el agente de MARS en MABS o un servidor DPM](backup-support-matrix-mabs-dpm.md).
