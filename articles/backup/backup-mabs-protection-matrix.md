---
title: Matriz de protección de MABS (Azure Backup Server) V3 UR1
description: En este artículo se incluye una matriz de compatibilidad en la que se indican todas las cargas de trabajo, los tipos de datos y las instalaciones que protege Azure Backup Server.
ms.date: 03/19/2020
ms.topic: conceptual
ms.openlocfilehash: 974820389e84f727a9aab284716b3be9020c6dd8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87032602"
---
# <a name="mabs-azure-backup-server-v3-ur1-protection-matrix"></a>Matriz de protección de MABS (Azure Backup Server) V3 UR1

En este artículo se indican los diferentes servidores y cargas de trabajo que pueden protegerse con Azure Backup Server. En la siguiente matriz se indican los elementos que pueden protegerse con Azure Backup Server.

Use la siguiente matriz para MABS v3 UR1:

* Cargas de trabajo: el tipo de carga de trabajo de la tecnología.

* Versión: versión de MABS compatible con las cargas de trabajo.

* Instalación de MABS: el equipo o la ubicación donde desea instalar MABS.

* Protección y recuperación: muestra la información detallada sobre las cargas de trabajo, como el contenedor de almacenamiento admitido o la implementación compatible.

## <a name="protection-support-matrix"></a>Matriz de compatibilidad de protección

En las secciones siguientes se detalla la matriz de compatibilidad de protección para MABS:

* [Copia de seguridad de aplicaciones](#applications-backup)
* [Copia de seguridad de máquinas virtuales](#vm-backup)
* [Linux](#linux)

## <a name="applications-backup"></a>Copia de seguridad de aplicaciones

| **Carga de trabajo**               | **Versión**                                                  | **Instalación de Azure Backup Server**                       | **Compatibilidad con Azure Backup Server** | **Protección y recuperación**                                  |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------- | ------------------------------------------------------------ |
| Equipos cliente (64 bits) | Windows 10                                                  | Servidor físico  <br><br>    Máquina virtual de Hyper-V   <br><br>   Máquina virtual de VMware | V3 UR1                            | Volumen, recurso compartido, carpeta, archivos, volúmenes desduplicados   <br><br>   Los volúmenes protegidos deben ser NTFS. No se admiten FAT ni FAT32.  <br><br>    Los volúmenes deben ser de al menos 1 GB. Azure Backup Server usa el Servicio de instantáneas de volumen (VSS) para tomar la instantánea de datos y esta solo funciona si el volumen es de al menos 1 GB. |
| Servidores (64 bits)          | Windows Server 2019, 2016, 2012 R2, 2012                    | Máquina virtual de Azure (cuando la carga de trabajo se ejecute como máquina virtual de Azure)  <br><br>    Servidor físico  <br><br>    Máquina virtual de Hyper-V <br><br>     Máquina virtual de VMware  <br><br>    Azure Stack | V3 UR1                            | Volumen, recurso compartido, carpeta, archivos, volúmenes desduplicados (NTFS y ReFS)  <br><br>   Estado del sistema y reconstrucción completa (no se admite cuando la carga de trabajo se ejecuta como máquina virtual de Azure) |
| Servidores (64 bits)          | Windows Server 2008 R2 SP1, Windows Server 2008 SP2 (debe instalar [Windows Management Frame 4.0](https://www.microsoft.com/download/details.aspx?id=40855)) | Servidor físico  <br><br>    Máquina virtual de Hyper-V  <br><br>      Máquina virtual de VMware  <br><br>   Azure Stack | V3 UR1                            | Volumen, recurso compartido, carpeta, archivo, estado del sistema y reconstrucción completa        |
| SQL Server                | SQL Server 2019, 2017, 2016 y [Service Packs compatibles](https://support.microsoft.com/lifecycle/search?alpha=SQL%20Server%202016), 2014 y [Service Packs](https://support.microsoft.com/lifecycle/search?alpha=SQL%20Server%202014) compatibles | Servidor físico  <br><br>     Máquina virtual de Hyper-V   <br><br>     Máquina virtual de VMware  <br><br>   Máquina virtual de Azure (cuando la carga de trabajo se ejecute como máquina virtual de Azure)  <br><br>     Azure Stack | V3 UR1                            | Todos los escenarios de implementación: base de datos       <br><br>  MABS V3 UR1 admite la copia de seguridad de bases de datos SQL en volúmenes ReFS                  |
| Exchange                   | Exchange 2019, 2016                                         | Servidor físico   <br><br>   Máquina virtual de Hyper-V  <br><br>      Máquina virtual de VMware  <br><br>   Azure Stack  <br><br>    Máquina virtual de Azure (cuando la carga de trabajo se ejecute como máquina virtual de Azure) | V3 UR1                            | Protección (todos los escenarios de implementación): instancia independiente de Exchange Server y base de datos en grupo de disponibilidad de base de datos (DAG)  <br><br>    Recuperación (todos los escenarios de implementación): buzón de correo y bases de datos de buzón de correo en un grupo de disponibilidad de base de datos (DAG)    <br><br>  La copia de seguridad de Exchange a través de ReFS es compatible con MABS V3 UR1 |
| SharePoint                 | SharePoint 2019, 2016 con el Service Pack más reciente                       | Servidor físico  <br><br>    Máquina virtual de Hyper-V <br><br>    Máquina virtual de VMware  <br><br>   Máquina virtual de Azure (cuando la carga de trabajo se ejecute como máquina virtual de Azure)   <br><br>   Azure Stack | V3 UR1                            | Protección (todos los escenarios de implementación): granja de servidores, contenido del servidor web front-end  <br><br>    Recuperación (todos los escenarios de implementación): granja, base de datos, aplicación web, elemento de lista o archivo, búsqueda de SharePoint y servidor web front-end  <br><br>    No se admite la protección para una granja de SharePoint que use la característica Always On de SQL Server 2012 para las bases de datos de contenido. |

## <a name="vm-backup"></a>Copia de seguridad de máquinas virtuales

| **Carga de trabajo**                                                 | **Versión**                                             | **Instalación de Azure Backup Server**                      | **Compatibilidad con Azure Backup Server** | **Protección y recuperación**                                 |
| ------------------------------------------------------------ | ------------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------- | ------------------------------------------------------------ |
| Host de Hyper-V: agente de protección MABS en el servidor host de Hyper-V, clúster o máquina virtual | Windows Server 2019, 2016, 2012 R2, 2012               | Servidor físico  <br><br>    Máquina virtual de Hyper-V <br><br>    Máquina virtual de VMware | V3 UR1                             | Protección:  equipos Hyper-V y volúmenes compartidos de clúster (CSV)  <br><br>    Recuperación: máquina virtual, recuperación a nivel de elemento de archivos y carpetas solo disponible para Windows, volúmenes y discos duros virtuales |
| Máquinas virtuales de VMware                                                  | VMware Server 5.5, 6.0 o 6.5, 6.7 (versión con licencia) | Máquina virtual de Hyper-V  <br><br>   Máquina virtual de VMware         | V3 UR1                             | Protección:  Máquinas virtuales de VMware en volúmenes compartidos de clúster (CSV), NFS y almacenamiento de SAN   <br><br>     Recuperación:  máquina virtual, recuperación a nivel de elemento de archivos y carpetas solo disponible para Windows, volúmenes y discos duros virtuales <br><br>    VMware vApps no compatible. |

## <a name="linux"></a>Linux

| **Carga de trabajo** | **Versión**                               | **Instalación de Azure Backup Server**                      | **Compatibilidad con Azure Backup Server** | **Protección y recuperación**                                 |
| ------------ | ----------------------------------------- | ------------------------------------------------------------ | ---------------------------------- | ------------------------------------------------------------ |
| Linux        | Linux ejecutado como invitado de Hyper-V o VMware | Servidor físico, máquina virtual de Hyper-V local, máquina virtual Windows en VMWare | V3 UR1                             | Hyper-V debe ejecutarse en Windows Server 2012 R2, Windows Server 2016 o Windows Server 2019. Protección:  máquina virtual completa   <br><br>   Recuperación: máquina virtual completa   <br><br>    Solo se admiten instantáneas coherentes con archivos.    <br><br>   Para obtener una lista completa de las distribuciones y las versiones de Linux compatibles, vea el artículo [Linux en distribuciones aprobadas por Azure](../virtual-machines/linux/endorsed-distros.md). |

## <a name="azure-expressroute-support"></a>Compatibilidad con Azure ExpressRoute

Puede realizar una copia de seguridad de los datos mediante Azure ExpressRoute con emparejamiento público (disponible para circuitos antiguos) y emparejamiento de Microsoft. La copia de seguridad por emparejamiento privado no se admite.

Con el emparejamiento público: asegúrese de tener acceso a los siguientes dominios y direcciones:

* `http://www.msftncsi.com/ncsi.txt`
* `microsoft.com`
* `.WindowsAzure.com`
* `.microsoftonline.com`
* `.windows.net`

Con el emparejamiento de Microsoft, seleccione los siguientes servicios o regiones y los valores de comunidad correspondientes:

* Azure Active Directory (12076:5060)
* Región de Microsoft Azure (según la ubicación del almacén de Recovery Services)
* Azure Storage (según la ubicación del almacén de Recovery Services)

Para más información, consulte los [requisitos de enrutamiento de ExpressRoute](../expressroute/expressroute-routing.md).

>[!NOTE]
>El emparejamiento público está en desuso para circuitos nuevos.

## <a name="cluster-support"></a>Compatibilidad con clústeres

Azure Backup Server puede proteger datos en las siguientes aplicaciones agrupadas en clúster:

* Servidores de archivos

* SQL Server

* Hyper-V: si se protege un clúster de Hyper-V mediante el agente de protección MABS escalado horizontalmente, no es posible agregar protección secundaria a las cargas de trabajo de Hyper-V protegidas.

* Exchange Server: Azure Backup Server puede proteger clústeres de discos no compartidos para versiones de Exchange Server admitidas (replicación continua en clúster) y también puede proteger instancias de Exchange Server configuradas para replicación continua local.

* SQL Server : Azure Backup Server no admite la realización de copias de seguridad de bases de datos de SQL Server hospedadas en volúmenes compartidos de clúster (CSV).

Azure Backup Server puede proteger cargas de trabajo de clúster que se encuentren en el mismo dominio que el servidor MABS y en un dominio secundario o de confianza. Si quiere proteger los orígenes de datos de dominios o grupos de trabajo que no son de confianza, use NTLM o una autenticación de certificado para un único servidor, o bien una autenticación de certificado para un único clúster.
