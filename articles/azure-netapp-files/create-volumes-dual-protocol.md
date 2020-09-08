---
title: Creación de un volumen de protocolo dual (NFSv3 y SMB) para Azure NetApp Files | Microsoft Docs
description: Describe cómo crear un volumen que usa el protocolo dual NFSv3 y SMB con compatibilidad con la asignación de usuarios LDAP.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 8/11/2020
ms.author: b-juche
ms.openlocfilehash: f4cc253de0de9d099cfc4881f48182cf9b2a1616
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88134588"
---
# <a name="create-a-dual-protocol-nfsv3-and-smb-volume-for-azure-netapp-files"></a>Creación de un volumen de protocolo dual (NFSv3 y SMB) para Azure NetApp Files

Azure NetApp Files admite la creación de volúmenes con NFS (NFSv3 y NFSv4.1), SMBv3 o el protocolo dual. En este artículo se muestra cómo crear un volumen que use el protocolo dual de NFSv3 y SMB con compatibilidad con la asignación de usuarios LDAP.  


## <a name="before-you-begin"></a>Antes de empezar 

* Debe haber configurado un grupo de capacidad.  
    Consulte [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md).   
* Debe haber una subred delegada en Azure NetApp Files.  
    Consulte [Delegación de una subred en Azure NetApp Files](azure-netapp-files-delegate-subnet.md).

## <a name="considerations"></a>Consideraciones

* Asegúrese de cumplir los [requisitos para las conexiones de Active Directory](azure-netapp-files-create-volumes-smb.md#requirements-for-active-directory-connections). 
* Cree una zona de búsqueda inversa en el servidor DNS y, a continuación, agregue un registro de puntero (PTR) de la máquina host de AD en esa zona de búsqueda inversa. De lo contrario, se producirá un error en la creación del volumen de dos protocolos.
* Asegúrese de que el cliente NFS esté actualizado y ejecute las actualizaciones más recientes del sistema operativo.

## <a name="create-a-dual-protocol-volume"></a>Creación de un volumen de protocolo dual

1.  Haga clic en la hoja **Volúmenes** desde la hoja Grupos de capacidad. Haga clic en **+ Agregar volumen** para crear un volumen. 

    ![Vaya a Volúmenes](../media/azure-netapp-files/azure-netapp-files-navigate-to-volumes.png) 

2.  En la ventana Crear un volumen, haga clic en **Crear** y proporcione la información para los campos siguientes bajo la pestaña Aspectos básicos:   
    * **Nombre del volumen**      
        Especifique el nombre para el volumen que va a crear.   

        Un nombre de volumen debe ser único dentro de cada grupo de capacidad. Debe tener tres caracteres de longitud, como mínimo. Puede usar cualquier carácter alfanumérico.   

        No se puede usar `default` como nombre del volumen.

    * **Grupo de capacidad**  
        Especifique el grupo de capacidad en el que desee que el volumen se cree.

    * **Cuota**  
        Especifique la cantidad de almacenamiento lógico que se asigna al volumen.  

        El campo **Cuota disponible** muestra la cantidad de espacio no utilizado en el grupo de capacidad elegido que puede usar para crear un nuevo volumen. El tamaño del volumen nuevo no debe superar la cuota disponible.  

    * **Red virtual**  
        Especifique la red virtual de Azure desde la que desea tener acceso al volumen.  

        La red virtual que especifique debe tener una subred delegada en Azure NetApp Files. Solo puede tener acceso al servicio Azure NetApp Files desde la misma red virtual o desde una red virtual que se encuentre en la misma ubicación que el volumen mediante el emparejamiento de redes virtuales. También puede acceder al volumen desde la red local mediante ExpressRoute.   

    * **Subred**  
        Especifique la subred que desea usar para el volumen.  
        La red virtual que especifique debe estar delegada en Azure NetApp Files. 
        
        Si no ha delegado una subred, haga clic en **Crear nuevo** en el volumen de creación de un volumen. A continuación, en la página de creación de la subred, especifique la información de la subred y seleccione **Microsoft.NetApp/volumes** para delegar la subred para Azure NetApp Files. En cada red virtual, solo puede delegarse una subred a Azure NetApp Files.   
 
        ![Crear un volumen](../media/azure-netapp-files/azure-netapp-files-new-volume.png)
    
        ![Creación de una subred](../media/azure-netapp-files/azure-netapp-files-create-subnet.png)

    * Si desea aplicar una directiva de instantáneas existente al volumen, haga clic en **Mostrar la sección avanzada** para expandirla y seleccione una directiva de instantáneas en el menú desplegable. 

        Para obtener información sobre cómo crear una directiva de instantáneas, consulte [Administración de directivas de instantánea](azure-netapp-files-manage-snapshots.md#manage-snapshot-policies).

        ![Mostrar la sección avanzada](../media/azure-netapp-files/volume-create-advanced-selection.png)

3. Haga clic en **Protocolo** y realice las siguientes acciones:  
    * Seleccione el **protocolo dual (NFSv3 y SMB)** como tipo de protocolo para el volumen.   

    * Seleccione la conexión de **Active Directory** en la lista desplegable.  
    La conexión de Active Directory que use debe tener un certificado de CA raíz del servidor. 

    * Especifique la **ruta de acceso del volumen** para el volumen.   
    Esta ruta de acceso del volumen es el nombre del volumen compartido. El nombre debe comenzar con un carácter alfabético y debe ser único dentro de cada suscripción y región.  

    * Especifique el **estilo de seguridad** que se vaya a usar: NTFS (valor predeterminado) o UNIX.

    * Si lo desea, [configure la directiva de exportación para el volumen](azure-netapp-files-configure-export-policy.md).

    ![Especificación del protocolo dual](../media/azure-netapp-files/create-volume-protocol-dual.png)

4. Haga clic en **Revisar y crear** para revisar los detalles del volumen. Después, haga clic en **Crear** para crear el volumen.

    El volumen que creó aparece en la hoja Volúmenes. 
 
    Un volumen hereda los atributos de ubicación, la suscripción y el grupo de recursos de su grupo de capacidad. Para supervisar el estado de la implementación del volumen, puede usar la pestaña Notificaciones.

## <a name="upload-active-directory-certificate-authority-public-root-certificate"></a>Carga del certificado raíz público de la entidad de certificación de Active Directory  

1.  Siga las indicaciones de [Instalación de la entidad de certificación](https://docs.microsoft.com/windows-server/networking/core-network-guide/cncg/server-certs/install-the-certification-authority) para instalar y configurar la entidad de certificación de ADDS. 

2.  Siga las indicaciones de [Ver certificados con el complemento MMC](https://docs.microsoft.com/dotnet/framework/wcf/feature-details/how-to-view-certificates-with-the-mmc-snap-in) para usar el complemento MMC y la herramienta Administrador de certificados.  
    Use el complemento Administrador de certificados para buscar el certificado raíz o emisor del dispositivo local. Debe ejecutar los comandos del complemento Administración de certificados desde una de las siguientes opciones:  
    * Un cliente basado en Windows que se haya unido al dominio y tenga instalado el certificado raíz. 
    * Otra máquina del dominio que contenga el certificado raíz.  

3. Exporte el certificado raíz.  
    Asegúrese de que el certificado se exporta en el formato X.509 codificado en Base-64 (.CER): 

    ![Asistente para exportación de certificados](../media/azure-netapp-files/certificate-export-wizard.png)

4. Vaya a la cuenta de NetApp del volumen del protocolo dual, haga clic en **Conexiones de Active Directory** y cargue el certificado de la entidad de certificación raíz mediante la ventana **Unir Active Directory**:  

    ![Certificado de CA raíz de servidor](../media/azure-netapp-files/server-root-ca-certificate.png)

    Asegúrese de que el nombre de la entidad de certificación pueda resolverse mediante DNS. Este nombre es el campo "Emitido por" o "Emisor" del certificado:  

    ![Información del certificado](../media/azure-netapp-files/certificate-information.png)

## <a name="manage-ldap-posix-attributes"></a>Administración de los atributos POSIX de LDAP

Puede administrar los atributos POSIX como UID, el directorio particular y otros valores mediante el complemento de MMC Usuarios y equipos de Active Directory.  En el ejemplo siguiente se muestra el editor de atributos de Active Directory:  

![Editor de atributos de Active Directory](../media/azure-netapp-files/active-directory-attribute-editor.png) 


## <a name="configure-the-nfs-client"></a>Configuración del cliente NFS 

Siga las instrucciones de [Configuración de un cliente NFS para Azure NetApp Files](configure-nfs-clients.md) para configurar el cliente NFS.  

## <a name="next-steps"></a>Pasos siguientes  

* [Preguntas más frecuentes sobre el protocolo dual](azure-netapp-files-faqs.md#dual-protocol-faqs)
* [Configuración de un cliente NFS para Azure NetApp Files](configure-nfs-clients.md) 
