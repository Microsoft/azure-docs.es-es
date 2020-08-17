---
title: Creación de un volumen SMB para Azure NetApp Files | Microsoft Docs
description: En este artículo se muestra cómo crear un volumen SMBv3 en Azure NetApp Files. Obtenga información acerca de los requisitos para las conexiones de Active Directory y Domain Services.
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
ms.date: 07/24/2020
ms.author: b-juche
ms.openlocfilehash: ba66716abe80a1b12bc64b739f498a0a01d54fe3
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533179"
---
# <a name="create-an-smb-volume-for-azure-netapp-files"></a>Creación de un volumen de SMB para Azure NetApp Files

Azure NetApp Files admite la creación de volúmenes con NFS (NFSv3 y NFSv4.1), SMBv3 o el protocolo dual (NFSv3 y SMB). El consumo de la capacidad de un volumen se descuenta de la capacidad aprovisionada de su grupo. En este artículo se muestra cómo crear un volumen SMBv3.

## <a name="before-you-begin"></a>Antes de empezar 
Debe haber configurado un grupo de capacidad.   
[Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md)   
Debe haber una subred delegada en Azure NetApp Files.  
[Delegación de una subred en Azure NetApp Files](azure-netapp-files-delegate-subnet.md)

## <a name="requirements-for-active-directory-connections"></a>Requisitos para las conexiones de Active Directory

 Debe crear las conexiones de Active Directory antes de crear un volumen SMB. Los requisitos para las conexiones de Active Directory son los siguientes: 

* La cuenta de administrador que use debe ser capaz de crear cuentas de máquina en la ruta de acceso de la unidad organizativa (OU) que se especifique.  

* Los puertos adecuados deben estar abiertos en el servidor de Windows Active Directory (AD) correspondiente.  
    Los puertos necesarios son los siguientes: 

    |     Servicio           |     Port     |     Protocolo     |
    |-----------------------|--------------|------------------|
    |    Servicios web de AD    |    9389      |    TCP           |
    |    DNS                |    53        |    TCP           |
    |    DNS                |    53        |    UDP           |
    |    ICMPv4             |    N/D       |    Echo Reply    |
    |    Kerberos           |    464       |    TCP           |
    |    Kerberos           |    464       |    UDP           |
    |    Kerberos           |    88        |    TCP           |
    |    Kerberos           |    88        |    UDP           |
    |    LDAP               |    389       |    TCP           |
    |    LDAP               |    389       |    UDP           |
    |    LDAP               |    3268      |    TCP           |
    |    Nombre de NetBIOS       |    138       |    UDP           |
    |    SAM/LSA            |    445       |    TCP           |
    |    SAM/LSA            |    445       |    UDP           |
    |    w32time            |    123       |    UDP           |

* La topología del sitio para la instancia de Active Directory Domain Services de destino debe cumplir las directrices, en particular, la red virtual de Azure donde se implementa Azure NetApp Files.  

    El espacio de direcciones de la red virtual donde se implementa Azure NetApp Files debe agregarse a un sitio de Active Directory nuevo o existente (donde se encuentre un controlador de dominio al que Azure NetApp Files pueda acceder). 

* Los servidores DNS especificados deben ser accesibles desde la [subred delegada](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-delegate-subnet) de Azure NetApp Files.  

    Consulte [Directrices para el planeamiento de red de Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-network-topologies) para ver topologías de red admitidas.

    Los grupos de seguridad de red (NSG) y los firewalls deben tener reglas configuradas correctamente para permitir solicitudes de Active Directory y de tráfico DNS. 

* La subred delegada en Azure NetApp Files debe poder tener acceso a todos los controladores de dominio Active Directory Domain Services (ADDS) del dominio, incluidos todos los controladores de dominio locales y remotos. De lo contrario, puede producirse una interrupción del servicio.  

    Si tiene controladores de dominio a los que no se puede acceder mediante la subred delegada de Azure NetApp Files, puede especificar un sitio de Active Directory durante la creación de la conexión a Active Directory.  Azure NetApp Files solo debe comunicarse con los controladores de dominio del sitio en el que se encuentra el espacio de direcciones de la subred delegada de Azure NetApp Files.

    Consulte [Diseño de la topología de sitio](https://docs.microsoft.com/windows-server/identity/ad-ds/plan/designing-the-site-topology) para obtener información acerca de los sitios y servicios de AD. 
    
<!--
* Azure NetApp Files supports DES, Kerberos AES 128, and Kerberos AES 256 encryption types (from the least secure to the most secure). The user credentials used to join Active Directory must have the highest corresponding account option enabled that matches the capabilities enabled for your Active Directory.   

    For example, if your Active Directory has only the AES-128 capability, you must enable the AES-128 account option for the user credentials. If your Active Directory has the AES-256 capability, you must enable the AES-256 account option (which also supports AES-128). If your Active Directory does not have any Kerberos encryption capability, Azure NetApp Files uses DES by default.  

    You can enable the account options in the properties of the Active Directory Users and Computers Microsoft Management Console (MMC):   

    ![Active Directory Users and Computers MMC](../media/azure-netapp-files/ad-users-computers-mmc.png)
-->

Consulte las [Preguntas más frecuentes de SMB](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-faqs#smb-faqs) de Azure NetApp Files para obtener información adicional de AD. 

## <a name="decide-which-domain-services-to-use"></a>Decisión sobre los servicios de dominio que se van a usar 

Azure NetApp Files admite [Active Directory Domain Services](https://docs.microsoft.com/windows-server/identity/ad-ds/plan/understanding-active-directory-site-topology) (ADDS) y Azure Active Directory Domain Services (AADDS) para las conexiones de AD.  Antes de crear una conexión de AD, debe decidir si va a usar ADDS o AADDS.  

Para obtener más información, consulte [Comparación de Active Directory Domain Services autoadministrado, Azure Active Directory y Azure Active Directory Domain Services administrado](https://docs.microsoft.com/azure/active-directory-domain-services/compare-identity-solutions). 

### <a name="active-directory-domain-services"></a>Active Directory Domain Services

Puede usar su ámbito favorito de [Sitios y servicios de Active Directory](https://docs.microsoft.com/windows-server/identity/ad-ds/plan/understanding-active-directory-site-topology) para Azure NetApp Files. Esta opción habilita las lecturas y escrituras en los controladores de dominio de Active Directory Domain Services (ADDS) a los que [tiene acceso Azure NetApp Files](azure-netapp-files-network-topologies.md). También impide que el servicio se comunique con los controladores de dominio que no están en el sitio especificado en Sitios y servicios de Active Directory. 

Para encontrar el nombre del sitio al usar ADDS, puede ponerse en contacto con el grupo administrativo de su organización que se responsabiliza de Active Directory Domain Services. En el ejemplo siguiente se muestra el complemento de Sitios y servicios de Active Directory en el que se muestra el nombre del sitio: 

![Sitios y servicios de Active Directory](../media/azure-netapp-files/azure-netapp-files-active-directory-sites-services.png)

Al configurar una conexión de AD para Azure NetApp Files, especifique el nombre del sitio en el ámbito para el campo **Nombre de sitio de Active Directory**.

### <a name="azure-active-directory-domain-services"></a>Azure Active Directory Domain Services 

Para obtener instrucciones y la configuración de Azure Active Directory Domain Services (AADDS), consulte [Documentación de Azure AD Domain Services](https://docs.microsoft.com/azure/active-directory-domain-services/).

Se aplican consideraciones de AADDS adicionales para Azure NetApp Files: 

* Asegúrese de que la red virtual o la subred donde está implementado AADDS se encuentra en la misma región de Azure que la implementación de Azure NetApp Files.
* Si usa otra red virtual en la región donde está implementado Azure NetApp Files, debe crear un emparejamiento entre las dos redes virtuales.
* Azure NetApp Files admite los tipos `user` y `resource forest`.
* Como tipo de sincronización, puede seleccionar `All` o `Scoped`.   
    Si selecciona `Scoped`, asegúrese de que está seleccionado el grupo de Azure AD correcto para acceder a los recursos compartidos de SMB.  Si no está seguro, puede usar el tipo de sincronización `All`.
* Se requiere el uso de la SKU Enterprise o Premium. No se admite la SKU estándar.

Cuando cree una conexión de Active Directory, tenga en cuenta las siguientes características específicas de AADDS:

* Puede encontrar la información de **DNS principal**, **DNS secundario** y **Nombre de dominio DNS de AD** en el menú AADDS.  
En el caso de los servidores DNS, se usarán dos direcciones IP para configurar la conexión de Active Directory. 
* La **ruta de acceso de unidad organizativa** es `OU=AADDC Computers`.  
Esta opción está configurada en **Conexiones de Active Directory** debajo de **Cuenta de NetApp**:

  ![Ruta de acceso de unidad organizativa](../media/azure-netapp-files/azure-netapp-files-org-unit-path.png)

* Las credenciales de **Nombre de usuario** pueden ser cualquier usuario que sea miembro del grupo de **administradores de controladores de dominio de Azure AD**.


## <a name="create-an-active-directory-connection"></a>Creación de una conexión de Active Directory

1. En la cuenta de NetApp, haga clic en **Conexiones de Active Directory** y, a continuación, haga clic en **Unir**.  

    ![Conexiones de Active Directory](../media/azure-netapp-files/azure-netapp-files-active-directory-connections.png)

2. En la ventana Unir Active Directory, especifique la siguiente información según los servicios de dominio que desee utilizar:  

    Para obtener información específica sobre los servicios de dominio que utiliza, consulte [Decisión sobre los servicios de dominio que se van a usar](#decide-which-domain-services-to-use). 

    * **DNS principal**  
        Se trata del DNS que se requiere para las operaciones de autenticación de SMB y la unión a un dominio de Active Directory. 
    * **DNS secundario**   
        Este es el servidor DNS secundario para garantizar los servicios de nombre redundantes. 
    * **Nombre de dominio DNS de AD**  
        Se trata del nombre de dominio de Active Directory Domain Services al que desea unirse.
    * **Nombre de sitio de AD**  
        Este es el nombre del sitio al que se limitará la detección de controladores de dominio.
    * **Prefijo SMB (cuenta de equipo)**  
        Este es el prefijo de nomenclatura para la cuenta de máquina en Active Directory que Azure NetApp Files va a usar para la creación de nuevas cuentas.

        Por ejemplo, si el estándar de nomenclatura que su organización usa para los servidores de archivos es NAS-01, NAS-02...,NAS-045, escribiría "NAS" para el prefijo. 

        El servicio creará cuentas de máquina adicionales en Active Directory según sea necesario.

        > [!IMPORTANT] 
        > El cambio de nombre del prefijo del servidor de SMB después de crear la conexión de Active Directory es perjudicial. Tendrá que volver a montar los recursos compartidos de SMB existentes después de cambiar el nombre del prefijo del servidor de SMB.

    * **Ruta de acceso de unidad organizativa**  
        Se trata de la ruta de acceso LDAP para la unidad organizativa (UO) donde se crearán las cuentas de máquina del servidor SMB. Es decir, UO=segundo nivel, UO=primer nivel. 

        Si usa Azure NetApp Files con Azure Active Directory Domain Services, la ruta de acceso de la unidad organizativa es `OU=AADDC Computers` cuando configura Active Directory para su cuenta de NetApp.

     * **Usuarios de la directiva de copia de seguridad**  
        Puede incluir cuentas adicionales que requieran privilegios elevados para la cuenta de equipo creada para su uso con Azure NetApp Files. Se permitirá a las cuentas especificadas cambiar los permisos de NTFS en el nivel de archivo o carpeta. Por ejemplo, puede especificar una cuenta de servicio sin privilegios que se usa para migrar los datos a un recurso compartido de archivos de SMB en Azure NetApp Files.  

        La característica **Usuarios de la directiva de copia de seguridad** se encuentra actualmente en la versión preliminar. Si es la primera vez que usa esta característica, regístrela antes de usarla: 

        ```azurepowershell-interactive
        Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFBackupOperator
        ```

        Compruebe el estado del registro de la característica: 

        > [!NOTE]
        > **RegistrationState** puede estar en el estado `Registering` durante varios minutos antes de cambiar a `Registered`. Espere hasta que el estado sea **Registrado** antes de continuar.

        ```azurepowershell-interactive
        Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFBackupOperator
        ```

    * Las credenciales, incluidos el **nombre de usuario** y la **contraseña**

    ![Unir Active Directory](../media/azure-netapp-files/azure-netapp-files-join-active-directory.png)

3. Haga clic en **Unir**.  

    Aparece la conexión de Active Directory que creó.

    ![Conexiones de Active Directory](../media/azure-netapp-files/azure-netapp-files-active-directory-connections-created.png)

## <a name="add-an-smb-volume"></a>Incorporación de un volumen SMB

1. Haga clic en la hoja **Volúmenes** desde la hoja Grupos de capacidad. 

    ![Vaya a Volúmenes](../media/azure-netapp-files/azure-netapp-files-navigate-to-volumes.png)

2. Haga clic en **+ Agregar volumen** para crear un volumen.  
    Aparece la ventana Crear un volumen.

3. En la ventana Crear un volumen, haga clic en **Crear** y proporcione la información para los campos siguientes en la pestaña Aspectos básicos:   
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
        
        Si no ha delegado una subred, puede hacer clic en **Crear nuevo** en la página Crear un volumen. A continuación, en la página de creación de la subred, especifique la información de la subred y seleccione **Microsoft.NetApp/volumes** para delegar la subred para Azure NetApp Files. En cada red virtual, solo puede delegarse una subred a Azure NetApp Files.   
 
        ![Crear un volumen](../media/azure-netapp-files/azure-netapp-files-new-volume.png)
    
        ![Creación de una subred](../media/azure-netapp-files/azure-netapp-files-create-subnet.png)

    * Si desea aplicar una directiva de instantáneas existente al volumen, haga clic en **Mostrar la sección avanzada** para expandirla y seleccione una directiva de instantáneas en el menú desplegable. 

        Para obtener información sobre cómo crear una directiva de instantáneas, consulte [Administración de directivas de instantánea](azure-netapp-files-manage-snapshots.md#manage-snapshot-policies).

        ![Mostrar la sección avanzada](../media/azure-netapp-files/volume-create-advanced-selection.png)

4. Haga clic en **Protocolo** y complete la siguiente información:  
    * Seleccione **SMB** como tipo de protocolo para el volumen. 
    * Seleccione la conexión de **Active Directory** en la lista desplegable.
    * Especifique el nombre del volumen compartido en **Nombre del recurso compartido**.

    ![Especifique el protocolo SMB](../media/azure-netapp-files/azure-netapp-files-protocol-smb.png)

5. Haga clic en **Revisar y crear** para revisar los detalles del volumen.  Haga clic en **Crear** para crear el volumen SMB.

    El volumen que creó aparece en la hoja Volúmenes. 
 
    Un volumen hereda los atributos de ubicación, la suscripción y el grupo de recursos de su grupo de capacidad. Para supervisar el estado de la implementación del volumen, puede usar la pestaña Notificaciones.

## <a name="control-access-to-an-smb-volume"></a>Control del acceso a un volumen SMB  

El acceso a un volumen SMB se administra mediante permisos.  

### <a name="share-permissions"></a>Permisos de recursos compartidos  

De forma predeterminada, un nuevo volumen tiene los permisos de recursos compartidos **Todos/Control total**. Los miembros del grupo Admins. del dominio pueden cambiar los permisos de recursos compartido mediante Administración de equipos en la cuenta de equipo que se usa para el volumen de Azure NetApp Files.

![Ruta de montaje de SMB](../media/azure-netapp-files/smb-mount-path.png) 
![Establecer permisos de recurso compartido](../media/azure-netapp-files/set-share-permissions.png) 

### <a name="ntfs-file-and-folder-permissions"></a>Permisos de carpetas y archivos NTFS  

Puede establecer permisos para un archivo o carpeta con la pestaña **Seguridad** de las propiedades del objeto en el cliente SMB de Windows.
 
![Establecimiento de permisos de carpetas y archivos](../media/azure-netapp-files/set-file-folder-permissions.png) 

## <a name="next-steps"></a>Pasos siguientes  

* [Montaje o desmontaje de un volumen para máquinas virtuales Windows o Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md)
* [Límites de recursos para Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Preguntas más frecuentes de SMB](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-faqs#smb-faqs)
* [Obtenga información sobre la integración de redes virtuales para los servicios de Azure](https://docs.microsoft.com/azure/virtual-network/virtual-network-for-azure-services)
* [Install a new Active Directory forest using Azure CLI](https://docs.microsoft.com/windows-server/identity/ad-ds/deploy/virtual-dc/adds-on-azure-vm) (Instalación de un nuevo bosque de Active Directory en la CLI de Azure)
