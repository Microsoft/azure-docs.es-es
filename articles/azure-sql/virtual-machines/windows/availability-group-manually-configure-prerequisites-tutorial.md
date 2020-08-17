---
title: 'Tutorial: Requisitos previos para un grupo de disponibilidad'
description: En este tutorial se muestra cómo configurar los requisitos previos para crear un grupo de disponibilidad AlwaysOn de SQL Server en Azure Virtual Machines.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.assetid: c492db4c-3faa-4645-849f-5a1a663be55a
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/29/2018
ms.author: mathoma
ms.custom: seo-lt-2019
ms.openlocfilehash: 881fa116b1a44d4714002f71e6ebd163279d8c70
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284309"
---
# <a name="prerequisites-for-creating-always-on-availability-groups-on-sql-server-on-azure-virtual-machines"></a>Requisitos previos para crear grupos de disponibilidad AlwaysOn de SQL Server en Azure Virtual Machines

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este tutorial se muestra cómo completar los requisitos previos para crear un [grupo de disponibilidad AlwaysOn de SQL Server en Azure Virtual Machines](availability-group-manually-configure-tutorial.md). Cuando haya completado los requisitos previos, tendrá un controlador de dominio, dos máquinas virtuales SQL Server y un servidor testigo en un único grupo de recursos.

**Tiempo estimado**: los requisitos previos pueden tardar hasta un par de horas en completarse. Gran parte de este tiempo se invierte en crear máquinas virtuales.

En el diagrama se muestra lo que va a crear en el tutorial.

![grupo de disponibilidad](./media/availability-group-manually-configure-prerequisites-tutorial-/00-EndstateSampleNoELB.png)

## <a name="review-availability-group-documentation"></a>Revisión de la documentación del grupo de disponibilidad

En este tutorial se da por supuesto que tiene conocimientos básicos de grupos de disponibilidad de SQL Server AlwaysOn. Si no está familiarizado con esta tecnología, consulte [Información general de los grupos de disponibilidad AlwaysOn (SQL Server)](https://msdn.microsoft.com/library/ff877884.aspx).


## <a name="create-an-azure-account"></a>Crear una cuenta de Azure

Necesitará una cuenta de Azure. Puede [abrir una cuenta gratuita de Azure](https://signup.azure.com/signup?offer=ms-azr-0044p&appId=102&ref=azureplat-generic) o [activar las ventajas que disfrutan los suscriptores de Visual Studio](https://docs.microsoft.com/visualstudio/subscriptions/subscriber-benefits).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **+** para crear un nuevo objeto en el portal.

   ![Nuevo objeto](./media/availability-group-manually-configure-prerequisites-tutorial-/01-portalplus.png)

3. Haga clic en **grupo de recursos** en la ventana de búsqueda de **Marketplace**.

   ![Resource group](./media/availability-group-manually-configure-prerequisites-tutorial-/01-resourcegroupsymbol.png)

4. Seleccione **Grupo de recursos**.
5. Seleccione **Crear**.
6. En **Nombre del grupo de recursos**, escriba un nombre para el grupo de recursos. Por ejemplo, escriba **sql-ha-rg**.
7. Si tiene varias suscripciones de Azure, compruebe que la suscripción es la suscripción de Azure en la que quiere crear el grupo de disponibilidad.
8. Seleccione una ubicación. La ubicación es la región de Azure donde desea crear el grupo de disponibilidad. En este artículo se compilan todos los recursos en una ubicación de Azure.
9. Compruebe que la función **Anclar al panel** está activada. Este parámetro opcional coloca un acceso directo para el grupo de recursos en el panel del Portal de Azure.

   ![Resource group](./media/availability-group-manually-configure-prerequisites-tutorial-/01-resourcegroup.png)

10. Seleccione **Crear** para crear el grupo de recursos.

Azure crea el grupo de recursos y ancla un acceso directo a él en el portal.

## <a name="create-the-network-and-subnets"></a>Creación de redes y subredes

El siguiente paso es crear las redes y subredes en el grupo de recursos de Azure.

La solución usa una red virtual con dos subredes. En [Información general sobre redes virtuales](../../../virtual-network/virtual-networks-overview.md) encontrará más información sobre las redes de Azure.

Para crear la red virtual en Azure Portal:

1. En el grupo de recursos, seleccione **+ Agregar**. 

   ![Nuevo elemento](./media/availability-group-manually-configure-prerequisites-tutorial-/02-newiteminrg.png)
2. Busque **red virtual**.

     ![Búsqueda de una red virtual](./media/availability-group-manually-configure-prerequisites-tutorial-/04-findvirtualnetwork.png)
3. Seleccione **Red virtual**.
4. En **Red virtual**, seleccione el modelo de implementación de **Resource Manager** y, después, **Crear**.

    La siguiente tabla muestra la configuración de la red virtual:

   | **Campo** | Value |
   | --- | --- |
   | **Nombre** |autoHAVNET |
   | **Espacio de direcciones** |10.33.0.0/24 |
   | **Nombre de subred** |Administración |
   | **Intervalo de direcciones de subred** |10.33.0.0/29 |
   | **Suscripción** |Especifique la suscripción que quiere usar. El campo **Suscripción** está en blanco si solo tiene una suscripción. |
   | **Grupos de recursos** |Elija la opción **Usar existente** y seleccione el nombre del grupo de recursos. |
   | **Ubicación** |Especifique la ubicación de Azure. |

   El intervalo de direcciones de subred y el espacio de direcciones podrían ser distintos a los de la tabla. Según su suscripción, el portal recomienda un espacio de direcciones disponible y el intervalo de direcciones de subred correspondiente. Si no hay suficiente espacio de direcciones disponible, use otra suscripción.

   En el ejemplo se usa el nombre de subred **Admin**. Esta subred es para los controladores de dominio.

5. Seleccione **Crear**.

   ![Configuración de la red virtual](./media/availability-group-manually-configure-prerequisites-tutorial-/06-configurevirtualnetwork.png)

Azure le devuelve al panel del portal y le informa cuando se cree la red.

### <a name="create-a-second-subnet"></a>Cree una segunda subred.

La nueva red virtual tiene una subred denominada **Admin**. Los controladores de dominio usan esta subred. Las máquinas virtuales con SQL Server usan una segunda subred denominada **SQL**. Para configurar esta subred, siga estos pasos:

1. En el panel, seleccione el grupo de recursos que ha creado, **SQL-HA-RG**. En **Recursos**, busque la red en el grupo de recursos.

    Si **SQL-HA-RG** no está visible, haga clic en **Grupos de recursos** y filtre por el nombre del grupo de recursos para encontrarlo.

2. Seleccione **autoHAVNET** en la lista de recursos. 
3. En la red virtual **autoHAVNET**, en **Configuración**, seleccione **Subredes**.

    Observe la subred que ya ha creado.

   ![Configuración de la red virtual](./media/availability-group-manually-configure-prerequisites-tutorial-/07-addsubnet.png)

5. Para crear una segunda subred, seleccione **+ Subred**.
6. En **Agregar subred**, configure la subred escribiendo **sqlsubnet** en **Nombre**. Azure especifica automáticamente un valor válido en **Intervalo de direcciones**. Compruebe que este intervalo de direcciones tiene al menos 10 direcciones. En un entorno de producción, puede que necesite más direcciones.
7. Seleccione **Aceptar**.

    ![Configuración de la red virtual](./media/availability-group-manually-configure-prerequisites-tutorial-/08-configuresubnet.png)

En la tabla siguiente se resumen las opciones de configuración de red:

| **Campo** | Value |
| --- | --- |
| **Nombre** |**autoHAVNET** |
| **Espacio de direcciones** |Este valor depende de los espacios de direcciones disponibles en su suscripción. Un valor típico es 10.0.0.0/16. |
| **Nombre de subred** |**admin** |
| **Intervalo de direcciones de subred** |Este valor depende de los intervalos de direcciones disponibles en su suscripción. Un valor típico es 10.0.0.0/24. |
| **Nombre de subred** |**sqlsubnet** |
| **Intervalo de direcciones de subred** |Este valor depende de los intervalos de direcciones disponibles en su suscripción. Un valor típico es 10.0.1.0/24. |
| **Suscripción** |Especifique la suscripción que quiere usar. |
| **Grupo de recursos** |**SQL-HA-RG** |
| **Ubicación** |Especifique la misma ubicación que eligió para el grupo de recursos. |

## <a name="create-availability-sets"></a>Creación de conjuntos de disponibilidad

Antes de crear máquinas virtuales, debe crear conjuntos de disponibilidad. Los conjuntos de disponibilidad reducen el tiempo de inactividad de cara al mantenimiento, tanto planeado como no planeado. Un conjunto de disponibilidad de Azure es un grupo lógico de recursos que Azure coloca en dominios de error y de actualización físicos. Un dominio de error garantiza que los miembros del conjunto de disponibilidad tengan recursos de energía y red independientes. Un dominio de actualización garantiza que los miembros del conjunto de disponibilidad no pasen a inactividad por mantenimiento al mismo tiempo. Para más información, consulte [Administración de la disponibilidad de las máquinas virtuales](../../../virtual-machines/linux/manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Necesita dos conjuntos de disponibilidad. El primero es para los controladores de dominio y el segundo, para las máquinas virtuales con SQL Server.

Para crear un conjunto de disponibilidad, vaya al grupo de recursos y seleccione **Agregar**. Para filtrar los resultados, escriba **conjunto de disponibilidad**. Seleccione **Conjunto de disponibilidad** en los resultados y, a continuación, **Crear**.

Configure dos conjuntos de disponibilidad según los parámetros de la tabla siguiente:

| **Campo** | Conjunto de disponibilidad del controlador de dominio | Conjunto de disponibilidad de SQL Server |
| --- | --- | --- |
| **Nombre** |adavailabilityset |sqlavailabilityset |
| **Grupos de recursos** |SQL-HA-RG |SQL-HA-RG |
| **Dominios de error** |3 |3 |
| **Dominios de actualización** |5 |3 |

Después de crear los conjuntos de disponibilidad, vuelva al grupo de recursos en el Portal de Azure.

## <a name="create-domain-controllers"></a>Creación de controladores de dominio

Después de haber creado la red, las subredes y los conjuntos de disponibilidad, está preparado para crear las máquinas virtuales para los controladores de dominio.

### <a name="create-virtual-machines-for-the-domain-controllers"></a>Creación de máquinas virtuales para controladores de dominio

Para crear y configurar los controladores de dominio, vuelva al grupo de recursos **SQL-HA-RG** .

1. Seleccione **Agregar**. 
2. Escriba **Windows Server 2016 Datacenter**.
3. Seleccione **Windows Server 2016 Datacenter**. En **Windows Server 2016 Datacenter**, compruebe que el modelo de implementación sea **Resource Manager** y seleccione **Crear**. 

Repita los pasos anteriores para crear dos máquinas virtuales. Asigne nombre a las dos máquinas virtuales:

* ad-primary-dc
* ad-secondary-dc

  > [!NOTE]
  > La máquina virtual **ad-secondary-dc** es opcional. Se usa para proporcionar alta disponibilidad para Active Directory Domain Services.
  >

La siguiente tabla muestra la configuración de estas dos máquinas:

| **Campo** | Value |
| --- | --- |
| **Nombre** |Primer controlador de dominio: *ad-primary-dc*.</br>Segundo controlador de dominio: *ad-secondary-dc*. |
| **Tipo de disco de máquina virtual** |SSD |
| **Nombre de usuario** |DomainAdmin |
| **Contraseña** |Contoso!0000 |
| **Suscripción** |*Su suscripción* |
| **Grupos de recursos** |SQL-HA-RG |
| **Ubicación** |*Su ubicación* |
| **Tamaño** |DS1_V2 |
| **Storage** | **Usar discos administrados** - **Sí** |
| **Red virtual** |autoHAVNET |
| **Subred** |admin |
| **Dirección IP pública** |*Mismo nombre que la máquina virtual* |
| **Grupo de seguridad de red** |*Mismo nombre que la máquina virtual* |
| **El conjunto de disponibilidad** |adavailabilityset </br>**Dominios de error**: 2 </br>**Dominios de actualización**: 2|
| **Diagnóstico** |habilitado |
| **Cuenta de almacenamiento de información de diagnóstico** |*Se crea automáticamente* |

   >[!IMPORTANT]
   >Solo puede colocar una máquina virtual en un conjunto de disponibilidad al crearlo. Una vez creada una máquina virtual, no se puede cambiar el conjunto de disponibilidad. Consulte [Administración de la disponibilidad de las máquinas virtuales](../../../virtual-machines/linux/manage-availability.md).

Azure crea las máquinas virtuales.

Después de crear las máquinas virtuales, configure el controlador de dominio.

### <a name="configure-the-domain-controller"></a>Configuración del controlador de dominio

En los pasos siguientes, configurará la máquina **ad-primary-dc** como controlador de dominio para corp.contoso.com.

1. En el portal, abra el grupo de recursos **SQL-HA-RG** y seleccione la máquina **ad-primary-dc**. En **ad-primary-dc**, seleccione **Conectar** para abrir un archivo RDP con el fin de acceder a Escritorio remoto.

    ![Conexión a una máquina virtual](./media/availability-group-manually-configure-prerequisites-tutorial-/20-connectrdp.png)

2. Inicie sesión con su cuenta de administrador configurada ( **\DomainAdmin**) y la contraseña (**Contoso!0000**).
3. De forma predeterminada, se debe mostrar el panel **Administrador del servidor** .
4. Seleccione el vínculo **Agregar roles y características** en el panel.

    ![Administrador del servidor: Agregar roles](./media/availability-group-manually-configure-prerequisites-tutorial-/22-addfeatures.png)

5. Seleccione **Siguiente** hasta llegar a la sección **Roles de servidor**.
6. Seleccione los roles **Active Directory Domain Services** y **Servidor DNS**. Cuando se le pida, agregue características adicionales que son necesarias para estos roles.

   > [!NOTE]
   > Windows le advierte de que no hay ninguna dirección IP estática. Si está probando la configuración, seleccione **Continuar**. En los escenarios de producción, establezca la dirección IP en estática en Azure Portal o [use PowerShell para establecer la dirección IP estática de la máquina del controlador de dominio](../../../virtual-network/virtual-networks-reserved-private-ip.md).
   >

    ![Cuadro de diálogo Agregar roles](./media/availability-group-manually-configure-prerequisites-tutorial-/23-addroles.png)

7. Seleccione **Siguiente** hasta llegar a la sección **Confirmación**. Active la casilla **Reiniciar automáticamente el servidor de destino en caso necesario** .
8. Seleccione **Instalar**.
9. Cuando terminen de instalarse las características, vuelva al panel **Administrador del servidor** .
10. Seleccione la nueva opción **AD DS** en el panel izquierdo.
11. Seleccione el vínculo **Más** en la barra de advertencia amarilla.

    ![Cuadro de diálogo AD DS en la máquina virtual del servidor DNS](./media/availability-group-manually-configure-prerequisites-tutorial-/24-addsmore.png)
    
12. En la columna **Acción** del cuadro de diálogo **Todos los detalles de la tarea del servidor**, seleccione **Promover este servidor a controlador de dominio**.
13. En el **Asistente de configuración de Active Directory Domain Services**, use los siguientes valores:

    | **Page** | Configuración |
    | --- | --- |
    | **Configuración de la implementación** |**Incorporación de nuevos bosques**<br/> **Nombre del dominio raíz** = corp.contoso.com |
    | **Opciones del controlador de dominio** |**Contraseña de DSRM** = Contoso!0000<br/>**Confirmar contraseña** = Contoso!0000 |

14. Seleccione **Siguiente** para avanzar por las otras páginas del asistente. En la página **Comprobación de requisitos previos**, compruebe que ve el mensaje siguiente: **Todas las comprobaciones de requisitos previos se realizaron correctamente**. Puede revisar todos los mensajes de advertencia aplicables, pero es posible continuar con la instalación.
15. Seleccione **Instalar**. La máquina virtual **ad-primary-dc** se reinicia automáticamente.

### <a name="note-the-ip-address-of-the-primary-domain-controller"></a>Anote la dirección IP del controlador de dominio principal.

Utilice el controlador de dominio principal para DNS. Anote la dirección IP del controlador de dominio principal.

Una manera de obtener la dirección IP del controlador de dominio principal es a través de Azure Portal.

1. En Azure Portal, abra el grupo de recursos.

2. Seleccione el controlador de dominio principal.

3. En el controlador del dominio principal, seleccione **Interfaces de red**.

![Interfaces de red](./media/availability-group-manually-configure-prerequisites-tutorial-/25-primarydcip.png)

Anote la dirección IP privada de este servidor.

### <a name="configure-the-virtual-network-dns"></a>Configuración del DNS de la red virtual

Después de crear el primer controlador de dominio y habilitar el DNS en el primer servidor, configure la red virtual para usar este servidor para DNS.

1. En Azure Portal, seleccione la red virtual.

2. En **Configuración**, seleccione **Servidor DNS**.

3. Seleccione **Personalizar** y escriba la dirección IP privada del controlador de dominio principal.

4. Seleccione **Guardar**.

### <a name="configure-the-second-domain-controller"></a>Configuración del segundo controlador de dominio

Una vez reiniciado el controlador de dominio principal, puede configurar el segundo controlador de dominio. Este paso opcional es para lograr alta disponibilidad. Para configurar el segundo controlador de dominio siga estos pasos:

1. En el portal, abra el grupo de recursos **SQL-HA-RG** y seleccione la máquina **ad-secondary-dc**. En **ad-secondary-dc**, seleccione **Conectar** para abrir un archivo RDP con el fin de acceder a Escritorio remoto.
2. Inicie sesión en la máquina virtual con la cuenta de administrador configurada (**BUILTIN\DomainAdmin**) y la contraseña (**Contoso!0000**).
3. Cambie la dirección del servidor DNS preferido por la dirección del controlador de dominio.
4. En **Centro de redes y recursos compartidos**, seleccione la interfaz de red.

   ![interfaz de red](./media/availability-group-manually-configure-prerequisites-tutorial-/26-networkinterface.png)

5. Seleccione **Propiedades**.
6. Seleccione **Protocolo de Internet versión 4 (TCP/IPv4)** y, luego, **Propiedades**.
7. Seleccione **Usar las siguientes direcciones de servidor DNS** y especifique la dirección del controlador de dominio principal en **Servidor DNS preferido**.
8. Seleccione **Aceptar** y **Cerrar** para confirmar los cambios. Ahora ya puede unir la máquina virtual a **corp.contoso.com**.

   >[!IMPORTANT]
   >Si pierde la conexión a Escritorio remoto después de cambiar la configuración de DNS, vaya a Azure Portal y reinicie la máquina virtual.

9. En el escritorio remoto del controlador de dominio secundario, abra el **panel Administrador del servidor**.
10. Seleccione el vínculo **Agregar roles y características** en el panel.

    ![Administrador del servidor: Agregar roles](./media/availability-group-manually-configure-prerequisites-tutorial-/22-addfeatures.png)
11. Seleccione **Siguiente** hasta llegar a la sección **Roles de servidor**.
12. Seleccione los roles **Active Directory Domain Services** y **Servidor DNS**. Cuando se le pida, agregue características adicionales que son necesarias para estos roles.
13. Cuando terminen de instalarse las características, vuelva al panel **Administrador del servidor** .
14. Seleccione la nueva opción **AD DS** en el panel izquierdo.
15. Seleccione el vínculo **Más** en la barra de advertencia amarilla.
16. En la columna **Acción** del cuadro de diálogo **Todos los detalles de la tarea del servidor**, seleccione **Promover este servidor a controlador de dominio**.
17. En **Configuración de implementación**, seleccione **Agregar un controlador de dominio a un dominio existente**.

    ![Configuración de la implementación](./media/availability-group-manually-configure-prerequisites-tutorial-/28-deploymentconfig.png)

18. Haga clic en **Seleccionar**.
19. Conéctese con la cuenta de administrador (**CORP.CONTOSO.COM\domainadmin**) y la contraseña (**Contoso!0000**).
20. En **Seleccionar un dominio del bosque**, elija el dominio y, después, seleccione **Aceptar**.
21. En **Opciones del controlador de dominio**, utilice los valores predeterminados y establezca una contraseña de DSRM.

    >[!NOTE]
    >La página **Opciones de DNS** puede avisarle de que no se puede crear una delegación de este servidor DNS. Puede pasar por alto esta advertencia en entornos de no producción.
    >

22. Seleccione **Siguiente** hasta que se muestre la casilla **Requisitos previos**. A continuación, seleccione **Instalar**.

Después de que el servidor complete los cambios de configuración, reinícielo.

### <a name="add-the-private-ip-address-to-the-second-domain-controller-to-the-vpn-dns-server"></a>Incorporación de la dirección IP privada del controlador de dominio secundario al servidor DNS de VPN

En Azure Portal, en la red virtual, cambie la opción Servidor DNS para que incluya la dirección IP del controlador de dominio secundario. Esta configuración permitirá la redundancia del servicio DNS.

### <a name="configure-the-domain-accounts"></a><a name="DomainAccounts"></a> Configuración de las cuentas de dominio

En los pasos siguientes configurará las cuentas de Active Directory. En la tabla siguiente se muestran las cuentas:

| |Cuenta de instalación<br/> |sqlserver-0 <br/>Cuenta del servicio Agente SQL y SQL Server |sqlserver-1<br/>Cuenta del servicio Agente SQL y SQL Server
| --- | --- | --- | ---
|**Nombre** |Instalar |SQLSvc1 | SQLSvc2
|**Usuario NombreDeCuentaSam** |Instalar |SQLSvc1 | SQLSvc2

Para crear cada cuenta, siga estos pasos.

1. Inicie sesión en la máquina **ad-primary-dc** .
2. En **Administrador del servidor**, seleccione **Herramientas** y, después, **Centro de administración de Active Directory**.   
3. Seleccione **corp (local)** en el panel izquierdo.
4. En el panel **Tareas** de la derecha, seleccione **Nuevo** y, después, **Usuario**.

   ![Centro de administración de Active Directory](./media/availability-group-manually-configure-prerequisites-tutorial-/29-addcnewuser.png)

   >[!TIP]
   >Establezca una contraseña compleja para cada cuenta.<br/> Para entornos de no producción, establezca que la cuenta de usuario no expire nunca.
   >

5. Seleccione **Aceptar** para crear el usuario.
6. Repita los pasos anteriores para cada una de las tres cuentas.

### <a name="grant-the-required-permissions-to-the-installation-account"></a>Concesión de los permisos necesarios para la cuenta de instalación

1. En el **Centro de administración de Active Directory**, seleccione **corp (local)** en el panel izquierdo. Luego, en el panel **Tareas** a la derecha, seleccione **Propiedades**.

    ![Propiedades de usuario CORP](./media/availability-group-manually-configure-prerequisites-tutorial-/31-addcproperties.png)

2. Seleccione **Extensiones** y, luego, el botón **Avanzadas** en la pestaña **Seguridad**.
3. En el cuadro de diálogo **Configuración de seguridad avanzada**, seleccione **Agregar**.
4. Elija **Seleccionar una entidad de seguridad**, busque **CORP\Install** y, a continuación, seleccione **Aceptar**.
5. Active la casilla **Leer todas las propiedades**.

6. Active la casilla **Create Computer objects** (Crear objetos de proceso).

     ![Permisos de usuario Corp](./media/availability-group-manually-configure-prerequisites-tutorial-/33-addpermissions.png)

7. Seleccione **Aceptar** y, después, otra vez **Aceptar**. Cierre la ventana de propiedades **corp**.

Ahora que ha terminado de configurar Active Directory y los objetos de usuario, cree dos máquinas virtuales con SQL Server y una máquina virtual de servidor testigo. Después, una las tres al dominio.

## <a name="create-sql-server-vms"></a>Creación de las máquinas virtuales con SQL Server

Cree tres máquinas virtuales adicionales. La solución requiere dos máquinas virtuales con instancias de SQL Server. Una tercera máquina virtual funciona como testigo. Windows Server 2016 puede usar un [testigo en la nube](https://docs.microsoft.com/windows-server/failover-clustering/deploy-cloud-witness). Sin embargo, para mantener la coherencia con los sistemas operativos anteriores, en este artículo se usa una máquina virtual como testigo.  

Tenga en cuenta las siguientes decisiones de diseño antes de continuar.

* **Almacenamiento: Azure Managed Disks**

   Para el almacenamiento de máquina virtual, use Azure Managed Disks. Microsoft recomienda el uso de Managed Disks para máquinas virtuales de SQL Server. Managed Disks controla el almacenamiento en segundo plano. Además, cuando las máquinas virtuales con Managed Disks están en el mismo conjunto de disponibilidad, Azure distribuye los recursos de almacenamiento para proporcionar la redundancia adecuada. Para más información, consulte [Introducción a Azure Managed Disks](../../../virtual-machines/linux/managed-disks-overview.md). Para obtener información específica acerca de Managed Disks en un conjunto de disponibilidad, consulte [Uso de Managed Disks para las máquinas virtuales de un conjunto de disponibilidad](../../../virtual-machines/linux/manage-availability.md#use-managed-disks-for-vms-in-an-availability-set).

* **Red: direcciones IP privadas en producción**

   Para las máquinas virtuales, este tutorial usa direcciones IP públicas. Una dirección IP pública permite conectarse de forma remota directamente a la máquina virtual a través de Internet y facilita los pasos de configuración. En entornos de producción, Microsoft recomienda usar únicamente direcciones IP privadas con el fin de reducir el grado de vulnerabilidad del recurso de máquina virtual de la instancia de SQL Server.

### <a name="create-and-configure-the-sql-server-vms"></a>Creación y configuración de las máquinas virtuales de SQL Server

A continuación, cree tres máquinas virtuales: dos de SQL Server y una para el nodo de clúster adicional. Para crear cada una de las máquinas virtuales, vuelva al grupo de recursos **SQL-HA-RG** y, después, seleccione **Agregar**. Busque el elemento de la galería adecuado, seleccione **Máquina virtual** y, después, seleccione **De la galería**. Use la información de la tabla siguiente para ayudarlo a crear las máquinas virtuales:


| Página | VM1 | VM2 | VM3 |
| --- | --- | --- | --- |
| Selección del elemento adecuado de la galería |**Windows Server 2016 Datacenter** |**SQL Server 2016 SP1 Enterprise en Windows Server 2016** |**SQL Server 2016 SP1 Enterprise en Windows Server 2016** |
| Configuración de máquina virtual: **Datos básicos** |**Nombre** = clúster-fsw<br/>**Nombre de usuario** = DomainAdmin<br/>**Password** = Contoso!0000<br/>**Suscripción** = su suscripción<br/>**Grupo de recursos** = SQL-HA-GR<br/>**Ubicación** = Su ubicación de Azure |**Nombre** = sqlserver-0<br/>**Nombre de usuario** = DomainAdmin<br/>**Password** = Contoso!0000<br/>**Suscripción** = su suscripción<br/>**Grupo de recursos** = SQL-HA-GR<br/>**Ubicación** = Su ubicación de Azure |**Nombre** = sqlserver-1<br/>**Nombre de usuario** = DomainAdmin<br/>**Password** = Contoso!0000<br/>**Suscripción** = su suscripción<br/>**Grupo de recursos** = SQL-HA-GR<br/>**Ubicación** = Su ubicación de Azure |
| Configuración de máquina virtual: **Tamaño** |**TAMAÑO** = DS1\_V2 (1 vCPU; 3,5 GB) |**TAMAÑO** = DS1\_V2 (2 vCPU, 7 GB)</br>El tamaño debe admitir el almacenamiento SSD (soporte de disco Premium. )) |**TAMAÑO** = DS1\_V2 (2 vCPU, 7 GB) |
| Configuración de máquina virtual: **Configuración** |**Almacenamiento**: Usar discos administrados.<br/>**Red virtual** = autoHAVNET<br/>**Subred** = sqlsubnet(10.1.1.0/24)<br/>**Dirección IP pública** generada automáticamente.<br/>**Grupo de seguridad de red** = ninguno<br/>**Diagnósticos de supervisión** = habilitado<br/>**Cuenta de almacenamiento de diagnósticos** = usar una cuenta de almacenamiento generada automáticamente<br/>**Conjunto de disponibilidad** = sqlAvailabilitySet<br/> |**Almacenamiento**: Usar discos administrados.<br/>**Red virtual** = autoHAVNET<br/>**Subred** = sqlsubnet(10.1.1.0/24)<br/>**Dirección IP pública** generada automáticamente.<br/>**Grupo de seguridad de red** = ninguno<br/>**Diagnósticos de supervisión** = habilitado<br/>**Cuenta de almacenamiento de diagnósticos** = usar una cuenta de almacenamiento generada automáticamente<br/>**Conjunto de disponibilidad** = sqlAvailabilitySet<br/> |**Almacenamiento**: Usar discos administrados.<br/>**Red virtual** = autoHAVNET<br/>**Subred** = sqlsubnet(10.1.1.0/24)<br/>**Dirección IP pública** generada automáticamente.<br/>**Grupo de seguridad de red** = ninguno<br/>**Diagnósticos de supervisión** = habilitado<br/>**Cuenta de almacenamiento de diagnósticos** = usar una cuenta de almacenamiento generada automáticamente<br/>**Conjunto de disponibilidad** = sqlAvailabilitySet<br/> |
| Configuración de máquina virtual: **Configuración de SQL Server** |No aplicable |**Conectividad de SQL** = privado (dentro de la red virtual)<br/>**Puerto** = 1433<br/>**Autenticación SQL** = deshabilitar<br/>**Configuración de almacenamiento** = general<br/>**Aplicación de revisiones automatizada** = el domingo a las 2:00<br/>**Copia de seguridad automatizada** = deshabilitado</br>**Integración de Azure Key Vault** = Deshabilitada |**Conectividad de SQL** = privado (dentro de la red virtual)<br/>**Puerto** = 1433<br/>**Autenticación SQL** = deshabilitar<br/>**Configuración de almacenamiento** = general<br/>**Aplicación de revisiones automatizada** = el domingo a las 2:00<br/>**Copia de seguridad automatizada** = deshabilitado</br>**Integración de Azure Key Vault** = Deshabilitada |

<br/>

> [!NOTE]
> Los tamaños de máquina que se sugieren aquí están diseñados para probar los grupos de disponibilidad en Azure Virtual Machines. Para optimizar el rendimiento de las cargas de trabajo de producción, consulte las recomendaciones de tamaños de máquina y la configuración de SQL Server en [Procedimientos recomendados de SQL Server en Azure Virtual Machines](performance-guidelines-best-practices.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
>

Una vez que las tres máquinas virtuales estén totalmente aprovisionadas, tendrá que unirlas al dominio **corp.contoso.com** y conceder a CORP\Install los derechos administrativos sobre las máquinas.

### <a name="join-the-servers-to-the-domain"></a><a name="joinDomain"></a>Unión de los servidores al dominio

Ahora ya puede unir las máquinas virtuales a **corp.contoso.com**. Realice los pasos siguientes en las máquinas virtuales con SQL Server y el servidor testigo del recurso compartido de archivos:

1. Conéctese de forma remota a la máquina virtual con **BUILTIN\DomainAdmin**.
2. En el **Administrador del servidor**, seleccione **Servidor local**.
3. Seleccione el vínculo **GRUPO DE TRABAJO**.
4. En la sección **Nombre de equipo**, seleccione **Cambiar**.
5. Active la casilla **Dominio** y escriba **corp.contoso.com** en el cuadro de texto. Seleccione **Aceptar**.
6. En el cuadro de diálogo emergente **Seguridad de Windows**, especifique las credenciales de la cuenta de administrador del dominio predeterminado (**CORP\DomainAdmin**) y la contraseña (**Contoso!0000**).
7. Cuando vea el mensaje "Bienvenida al dominio corp.contoso.com", seleccione **Aceptar**.
8. Seleccione **Cerrar** y luego **Reiniciar ahora** en el cuadro de diálogo emergente.

### <a name="add-the-corpinstall-user-as-an-administrator-on-each-cluster-vm"></a>Agregue el usuario Corp\Install como administrador en cada máquina virtual del clúster:

Después de que cada máquina virtual se reinicie como un miembro del dominio, agregue **CORP\Install** como miembro del grupo de administradores local.

1. Espere hasta que se reinicie la máquina virtual y, después, inicie el archivo RDP de nuevo desde el controlador de dominio principal para iniciar sesión en **sqlserver-0** con la cuenta **CORP\DomainAdmin**.

   >[!TIP]
   >Asegúrese de que inicie sesión con la cuenta de administrador del dominio. En los pasos anteriores, usó la cuenta de administrador BUILTIN. Ahora que el servidor está en el dominio, utilice la cuenta de dominio. En la sesión RDP, especifique *DOMINIO*\\*nombre de usuario*.
   >

2. En **Administrador del servidor**, seleccione **Herramientas** y, después, **Administración de equipos**.
3. En la ventana **Administración de equipos**, expanda **Usuarios y grupos locales** y luego seleccione **Grupos**.
4. Haga doble clic en el grupo **Administradores** .
5. En el cuadro de diálogo **Propiedades de administradores**, seleccione el botón **Agregar**.
6. Especifique el usuario **CORP\Install** y seleccione **Aceptar**.
7. Seleccione **Aceptar** para cerrar el cuadro de diálogo **Propiedades de administradores**.
8. Repita los pasos anteriores en **sqlserver-1** y **cluster-fsw**.

### <a name="set-the-sql-server-service-accounts"></a><a name="setServiceAccount"></a>Configuración de las cuentas de servicio de SQL Server

Establezca la cuenta de servicio de SQL Server en cada máquina virtual con SQL Server. Utilice las cuentas que creó cuando configuró las cuentas de dominio.

1. Abra el **Administrador de configuración de SQL Server**.
2. Haga clic con el botón derecho en el servicio SQL Server y seleccione **Propiedades**.
3. Establezca la cuenta y contraseña.
4. Repita estos pasos en la otra máquina virtual con SQL Server.  

En lo que respecta a los grupos de disponibilidad de SQL Server, cada máquina virtual con SQL Server debe ejecutarse como una cuenta de dominio.

### <a name="create-a-sign-in-on-each-sql-server-vm-for-the-installation-account"></a>Creación de un inicio de sesión en cada máquina virtual con SQL Server para la cuenta de instalación

Use la cuenta de instalación (CORP\install) para configurar el grupo de disponibilidad. Esta cuenta debe ser miembro del rol de servidor fijo **sysadmin** en cada máquina virtual con SQL Server. Con los pasos siguientes se crea un inicio de sesión para la cuenta de instalación:

1. Conéctese al servidor mediante el Protocolo de escritorio remoto (RDP) mediante la cuenta *\<MachineName\>\DomainAdmin*.

1. Abra SQL Server Management Studio y conéctese a la instancia local de SQL Server.

1. En el **Explorador de objetos**, seleccione **Seguridad**.

1. Haga clic con el botón derecho en **Inicios de sesión**. Seleccione **Nuevo inicio de sesión**.

1. En **Inicio de sesión: Nuevo**, seleccione **Buscar**.

1. Seleccione **Ubicaciones**.

1. Escriba las credenciales del administrador de dominio.

1. Utilice la cuenta de instalación.

1. Establezca el inicio de sesión como un miembro del rol de servidor fijo **sysadmin**.

1. Seleccione **Aceptar**.

Repita los pasos anteriores en la otra máquina virtual con SQL Server.

## <a name="add-failover-clustering-features-to-both-sql-server-vms"></a>Adición de características de clúster de conmutación por error en ambas máquinas virtuales con SQL Server

Para agregar características de clúster de conmutación por error, realice los pasos siguientes en ambas máquinas virtuales con SQL Server:

1. Conéctese a la máquina virtual de SQL Server con el Protocolo de escritorio remoto (RDP) mediante la cuenta *CORP\install*. Abra el **panel Administrador de servidores**.
2. Seleccione el vínculo **Agregar roles y características** en el panel.

    ![Administrador del servidor: Agregar roles](./media/availability-group-manually-configure-prerequisites-tutorial-/22-addfeatures.png)

3. Seleccione **Siguiente** hasta llegar a la sección **Características de servidor**.
4. En **Características**, seleccione **Clúster de conmutación por error**.
5. Agregue más características necesarias.
6. Seleccione **Instalar** para agregar las características.

Repita los pasos en la otra máquina virtual con SQL Server.

  >[!NOTE]
  > Ahora este paso, junto con la unión real de las máquinas virtuales de SQL Server al clúster de conmutación por error, se puede automatizar con la [CLI de máquina virtual de SQL de Azure](availability-group-az-cli-configure.md) y las [plantillas de inicio rápido de Azure](availability-group-quickstart-template-configure.md).
  >


## <a name="configure-the-firewall-on-each-sql-server-vm"></a><a name="endpoint-firewall"></a> Configuración del firewall en cada máquina virtual con SQL Server

La solución requiere que los siguientes puertos TCP estén abiertos en el firewall:

- **Máquina virtual con SQL Server**: Puerto 1433 para una instancia predeterminada de SQL Server.
- **Sondeo de Azure Load Balancer:** Cualquier puerto disponible. A menudo, los ejemplos utilizan el 59999.
- **Punto de conexión de reflejo de la base de datos:** Cualquier puerto disponible. A menudo, los ejemplos utilizan el 5022.

Los puertos de firewall deben estar abiertos en ambas máquinas virtuales con SQL Server.

El método de abrir los puertos depende de la solución de firewall que use. En la siguiente sección se explica cómo abrir los puertos en Firewall de Windows. Abra los puertos necesarios en cada una de las máquinas virtuales con SQL Server.

### <a name="open-a-tcp-port-in-the-firewall"></a>Apertura de un puerto TCP en el firewall

1. En la pantalla **Inicio** del primer servidor SQL Server, abra **Firewall de Windows con seguridad avanzada**.
2. En el panel izquierdo, seleccione **Reglas de entrada**. En el panel derecho, seleccione **Nueva regla**.
3. En **Tipo de regla**, elija **Puerto**.
4. Para el puerto, especifique **TCP** y escriba los números de puerto adecuados. Observe el ejemplo siguiente:

   ![Firewall de SQL](./media/availability-group-manually-configure-prerequisites-tutorial-/35-tcpports.png)

5. Seleccione **Next** (Siguiente).
6. En la página **Acción**, mantenga seleccionado **Permitir la conexión** y luego seleccione **Siguiente**.
7. En la página **Perfiles**, acepte la configuración predeterminada y luego seleccione **Siguiente**.
8. En la página **Nombre**, especifique el nombre de una regla, como **Sondeo de Azure Load Balancer** en el cuadro de texto **Nombre** y seleccione **Finalizar**.

Repita estos pasos en la segunda máquina virtual con SQL Server.

## <a name="configure-system-account-permissions"></a>Configuración de permisos de cuenta del sistema

Para crear una cuenta para la cuenta del sistema y conceder los permisos adecuados, complete estos pasos en cada instancia de SQL Server:

1. Cree una cuenta para `[NT AUTHORITY\SYSTEM]` en cada instancia de SQL Server. El script siguiente crea esta cuenta:

   ```sql
   USE [master]
   GO
   CREATE LOGIN [NT AUTHORITY\SYSTEM] FROM WINDOWS WITH DEFAULT_DATABASE=[master]
   GO 
   ```

1. Conceda los permisos siguientes a `[NT AUTHORITY\SYSTEM]` en cada instancia de SQL Server:

   - `ALTER ANY AVAILABILITY GROUP`
   - `CONNECT SQL`
   - `VIEW SERVER STATE`

   El script siguiente concede estos permisos:

   ```sql
   GRANT ALTER ANY AVAILABILITY GROUP TO [NT AUTHORITY\SYSTEM]
   GO
   GRANT CONNECT SQL TO [NT AUTHORITY\SYSTEM]
   GO
   GRANT VIEW SERVER STATE TO [NT AUTHORITY\SYSTEM]
   GO 
   ```

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un grupo de disponibilidad AlwaysOn de SQL Server en Azure Virtual Machines](availability-group-manually-configure-tutorial.md)
