---
title: Aprovisionamiento de una máquina virtual Windows con Azure Portal
description: En esta guía se describen las opciones disponibles para usar Azure Portal en el aprovisionamiento de SQL Server en una máquina virtual Windows.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.assetid: 1aff691f-a40a-4de2-b6a0-def1384e086e
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: infrastructure-services
ms.date: 11/07/2019
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: e315f49cb0b78e13c4b6132f844397d1261ff0f9
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88652024"
---
# <a name="how-to-use-the-azure-portal-to-provision-a-windows-virtual-machine-with-sql-server"></a>Uso de Azure Portal para aprovisionar una máquina virtual Windows con SQL Server

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En esta guía se describen las opciones disponibles para usar Azure Portal en el aprovisionamiento de SQL Server en una máquina virtual Windows. En este artículo se describen más opciones de configuración que en el [inicio rápido de máquinas virtuales con SQL Server](sql-vm-create-portal-quickstart.md), que se centra con más detalle en una única configuración. 

Use esta guía para crear su propia VM con SQL Server. O bien úsela como referencia para las opciones disponibles en Azure Portal.

> [!TIP]
> Si tiene alguna pregunta sobre las máquinas virtuales de SQL Server, consulte las [Preguntas más frecuentes](frequently-asked-questions-faq.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sql-server-virtual-machine-gallery-images"></a><a id="select"></a> Imágenes de la galería de máquinas virtuales de SQL Server

Cuando crea una máquina virtual de SQL Server, puede seleccionar una de varias imágenes configuradas previamente en la galería de máquinas virtuales. Los pasos siguientes muestran cómo seleccionar una de las imágenes de SQL Server 2017.

1. Seleccione **Azure SQL** en el menú izquierdo de Azure Portal. Si **Azure SQL** no está en la lista, seleccione **Todos los servicios** y, luego, escriba *Azure SQL* en el cuadro de búsqueda. 

   También puede seleccionar la estrella junto a **Azure SQL** para marcarlo como favorito y agregarlo como un elemento en el panel de navegación izquierdo. 

1. Seleccione **+ Agregar** para abrir la página **Select SQL deployment option** (Seleccionar la opción de implementación de SQL). Para ver información adicional, seleccione **Mostrar detalles**. 
1. Escriba *2017* en el cuadro de búsqueda de imágenes de SQL Server en el icono **Máquinas virtuales SQL** y, a continuación, seleccione **Free SQL Server License: SQL Server 2017 Developer en Windows Server 2016** de la lista desplegable. 

   ![Seleccionar imagen de máquina virtual de SQL](./media/create-sql-vm-portal/select-sql-vm-image-portal.png)

   > [!TIP]
   > La edición Developer se usa en este artículo porque se trata de una edición gratuita completa de SQL Server para pruebas de desarrollo. Solo paga por el costo de ejecutar la máquina virtual. Sin embargo, es libre de elegir cualquiera de las imágenes para usarla en este tutorial. Para una descripción de las imágenes disponibles, consulte la [introducción a SQL Server en máquinas virtuales Windows de Azure](sql-server-on-azure-vm-iaas-what-is-overview.md#payasyougo).

   > [!TIP]
   > Los costos de licencia de SQL Server se incorporan en el precio por segundo de la VM que se crea y varía según edición y núcleos. Sin embargo, la edición SQL Server Developer es gratis para desarrollo y pruebas, no para producción. Además, SQL Express es gratis para cargas de trabajo ligeras (menos de 1 GB de memoria, menos de 10 GB de almacenamiento). También existe la opción de traiga su propia licencia (BYOL) y que solo pague por la máquina virtual. Esos nombres de imagen tienen el prefijo {BYOL}. 
   >
   > Para más información sobre estas opciones, consulte [Pricing guidance for SQL Server Azure VMs](pricing-guidance.md) (Orientación de precios de máquinas virtuales de SQL Server en Azure).


1. Seleccione **Crear**.


## <a name="1-configure-basic-settings"></a>1. Configuración básica

En la pestaña **Básico**, especifique la siguiente información:

* En **Detalles del proyecto**, asegúrese de que está seleccionada la suscripción correcta. 
* En la sección **Grupo de recursos**, seleccione un grupo de recursos existente en la lista o elija **Crear nuevo** para crear un grupo de recursos. Un grupo de recursos es una colección de recursos relacionados de Azure (máquinas virtuales, cuentas de almacenamiento, redes virtuales, etc.). 

  ![Suscripción](./media/create-sql-vm-portal/basics-project-details.png)

  > [!NOTE]
  > El uso de un grupo de recursos resulta útil si solo está probando o aprendiendo sobre las implementaciones de SQL Server en Azure. Cuando haya terminado la prueba, elimine el grupo de recursos. De ese modo, se eliminará automáticamente la máquina virtual y los recursos asociados a ese grupo. Para más información sobre los grupos de recursos, consulte [Información general de Azure Resource Manager](../../../active-directory-b2c/overview.md).


* En **Detalles de instancia**:

    1. Escriba un **nombre de máquina virtual** único.  
    1. Elija una ubicación para su **Región**. 
    1. Para esta guía, en **Opciones de disponibilidad** deje _No se requiere redundancia de la infraestructura_. Para más información acerca de las opciones de disponibilidad, consulte [Disponibilidad](../../../virtual-machines/windows/availability.md). 
    1. En la lista **Imagen** seleccione la denominada _Free SQL Server License: SQL Server 2017 Developer on Windows Server 2016_ (Licencia gratuita de SQL Server: SQL Server 2017 Developer en Windows Server 2016).  
    1. Elija **Cambiar el tamaño** para el **Tamaño** de la máquina virtual y seleccione la oferta **Básico A2**. Asegúrese de limpiar los recursos cuando haya terminado con ellos para evitar cargos inesperados. Para cargas de trabajo de producción, consulte las recomendaciones de tamaños de máquina y la configuración en [Procedimientos recomendados de SQL Server en Azure Virtual Machines](performance-guidelines-best-practices.md).

    ![Detalles de instancia](./media/create-sql-vm-portal/basics-instance-details.png)

> [!IMPORTANT]
> El costo mensual estimado que se muestra en la ventana **Elegir un tamaño** no incluye los costos de licencias de SQL Server. Esta estimación es el costo solo de la máquina virtual. Para las ediciones Express y Developer de SQL Server, esta estimación es el costo total estimado. Para conocer el costo de otras ediciones, consulte la [página de precios de máquinas virtuales Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) y seleccione la edición de destino de SQL Server. Consulte también [Orientación de precios de SQL Server para máquinas virtuales de Azure](pricing-guidance.md) y [Tamaños de las máquinas virtuales](../../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

* En **Cuenta de administrador**, especifique un nombre de usuario y una contraseña. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../../../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).

   ![Cuenta de administrador](./media/create-sql-vm-portal/basics-administrator-account.png)

* En **Reglas de puerto de entrada**, elija **Permitir los puertos seleccionados** y luego seleccione **RDP (3389)** en la lista desplegable. 

   ![Reglas de puerto de entrada](./media/create-sql-vm-portal/basics-inbound-port-rules.png)


## <a name="2-configure-optional-features"></a>2. Configuración de características opcionales

### <a name="disks"></a>Discos

En la pestaña **Discos**, configure las opciones de disco. 

* En **Tipo de disco del sistema operativo** , seleccione en la lista desplegable el tipo de disco que desea para el sistema operativo. Se recomienda Premium para los sistemas de producción, pero no está disponible para las máquinas virtuales básicas. Para usar SSD Premium, cambie el tamaño de la máquina virtual. 
* En **Avanzado**, seleccione **Sí** en uso de **Managed Disks**.

   > [!NOTE]
   > Microsoft recomienda el uso de Managed Disks para SQL Server. Managed Disks controla el almacenamiento en segundo plano. Además, cuando las máquinas virtuales con Managed Disks están en el mismo conjunto de disponibilidad, Azure distribuye los recursos de almacenamiento para proporcionar la redundancia adecuada. Para más información, consulte [Introducción a Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md). Para obtener información específica acerca de Managed Disks en un conjunto de disponibilidad, consulte [Uso de Managed Disks para las máquinas virtuales de un conjunto de disponibilidad](../../../virtual-machines/windows/manage-availability.md).

![Configuración de disco de máquina virtual SQL](./media/create-sql-vm-portal/azure-sqlvm-disks.png)
  
  
### <a name="networking"></a>Redes

En la pestaña **Redes**, configure las opciones de las red. 

* Cree una **red virtual** o use una existente para su VM con SQL Server. Designe también una **subred**. 

* En **Grupo de seguridad de red de NIC**, seleccione un grupo de seguridad básica o el grupo de seguridad avanzada. La opción básica permite seleccionar los puertos de entrada de la VM con SQL Server, que son los mismos valores que se configuraron en la pestaña **Básico**. La opción avanzada permite elegir un grupo de seguridad de red existente o crear uno nuevo. 

* Puede realizar otros cambios en valores de configuración de la red o mantener los valores predeterminados.

![Configuración de red de máquina virtual SQL](./media/create-sql-vm-portal/azure-sqlvm-networking.png)

#### <a name="monitoring"></a>Supervisión

En la pestaña **Supervisión**, configure la supervisión y el apagado automático. 

* De forma predeterminada, Azure habilita los **diagnósticos de arranque** con la misma cuenta de almacenamiento que se designó para la máquina virtual. En esta pestaña, puede cambiar esta configuración y habilitar **Diagnósticos del SO invitado**. 
* En esta pestaña también se pueden habilitar **Identidad administrada asignada por el sistema** y **apagado automático**. 

![Configuración de administración de máquina virtual SQL](./media/create-sql-vm-portal/azure-sqlvm-management.png)


## <a name="3-configure-sql-server-settings"></a>3. Configuración de SQL Server

En la pestaña **Configuración de SQL Server**, configure las optimizaciones y los valores específicos de SQL Server. Puede configurar los siguientes valores para SQL Server:

- [Conectividad](#connectivity)
- [Autenticación](#authentication)
- [Integración de Azure Key Vault](#azure-key-vault-integration)
- [Configuración de almacenamiento](#storage-configuration)
- [Aplicación de revisión automatizada](#automated-patching)
- [Copia de seguridad automatizada](#automated-backup)
- [Machine Learning Services](#machine-learning-services)


### <a name="connectivity"></a>Conectividad

En **Conectividad SQL**, especifique el tipo de acceso que desea para la instancia de SQL Server de esta máquina virtual. En este tutorial, seleccione **Público (Internet)** para permitir que se establezcan conexiones a SQL Server desde equipos o servicios de Internet. Si esta opción está seleccionada, Azure configurará automáticamente el firewall y el grupo de seguridad de red para permitir el tráfico en el puerto seleccionado.

> [!TIP]
> De forma predeterminada, SQL Server escucha en un puerto conocido, **1433**. Para mayor seguridad, cambie el puerto en el cuadro de diálogo anterior para que escuche en un puerto no predeterminado, como 1401. Si cambia el puerto, debe conectarse con ese puerto desde cualquiera de las herramientas de cliente, como SQL Server Management Studio (SSMS).

![Seguridad de VM de SQL](./media/create-sql-vm-portal/azure-sqlvm-security.png)

Para conectarse a SQL Server a través de Internet, también debe habilitar la autenticación de SQL Server, que se describe en la sección siguiente.

Si prefiere no permitir las conexiones al motor de base de datos a través de Internet, elija una de las siguientes opciones:

* **Local (solo dentro de la máquina virtual)** : elija esta opción para permitir conexiones a SQL Server únicamente desde dentro de la máquina virtual.
* **Privado (dentro de la red virtual)** : elija esta opción para permitir conexiones a SQL Server desde máquinas o servicios que se encuentren en la misma red virtual.

En general, mejore la seguridad al elegir la conectividad más restrictiva que permita su escenario. No obstante, todas las opciones se pueden proteger mediante reglas del grupo de seguridad de red (NSG) y la autenticación de SQL o Windows. Puede editar el NSG una vez creada la máquina virtual. Para obtener más información, consulte [Consideraciones de seguridad para SQL Server en Azure Virtual Machines](security-considerations-best-practices.md).

### <a name="authentication"></a>Authentication

Si requiere autenticación de SQL Server, seleccione **Habilitar** en **Autenticación SQL** en la pestaña **Configuración de SQL Server**.

![Autenticación de SQL Server](./media/create-sql-vm-portal/azure-sqlvm-authentication.png)

> [!NOTE]
> Si piensa acceder a SQL Server a través de Internet (la opción de conectividad pública), debe habilitar aquí la autenticación de SQL. El acceso público a SQL Server requiere la autenticación de SQL.

Si habilita la autenticación de SQL Server, especifique los valores de **Nombre de inicio de sesión** y **Contraseña**. Este nombre de inicio de sesión se configura como inicio de sesión de Autenticación de SQL Server y es miembro del rol fijo de servidor **sysadmin**. Para más información sobre los modos de autenticación, consulte [Elegir un modo de autenticación](https://docs.microsoft.com/sql/relational-databases/security/choose-an-authentication-mode).

Si prefiere no habilitar la autenticación de SQL Server, puede utilizar la cuenta de administrador local en la máquina virtual para conectarse a la instancia de SQL Server.

### <a name="azure-key-vault-integration"></a>Integración de Azure Key Vault

Para almacenar secretos de seguridad en Azure para el cifrado, seleccione **Configuración de SQL Server**y desplácese hacia abajo hasta **Integración de Azure Key Vault**. Seleccione **Habilitar** y rellene la información solicitada. 

![Integración de Azure Key Vault](./media/create-sql-vm-portal/azure-sqlvm-akv.png)

En la tabla siguiente se enumeran los parámetros necesarios para configurar la integración de Azure Key Vault (AKV).

| PARÁMETRO | DESCRIPTION | EJEMPLO: |
| --- | --- | --- |
| **Dirección URL de Key Vault** |La ubicación del almacén de claves. |`https://contosokeyvault.vault.azure.net/` |
| **Nombre de entidad de seguridad** |Nombre de la entidad de servicio de Azure Active Directory Esto se conoce también como Id. Este nombre también se conoce como «identificador de cliente». |`fde2b411-33d5-4e11-af04eb07b669ccf2` |
| **Secreto de entidad de seguridad** |Secreto de la entidad de seguridad de servicio de Azure Active Directory Este secreto también se conoce como «secreto de cliente». |`9VTJSQwzlFepD8XODnzy8n2V01Jd8dAjwm/azF1XDKM=` |
| **Nombre de credencial** |**Nombre de credencial**: La integración de AKV crea una credencial en SQL Server y permite el acceso de la máquina virtual al almacén de claves. Elija un nombre para esta credencial. |`mycred1` |

Para más información, consulte [Configuración de la integración de Azure Key Vault para SQL Server en máquinas virtuales de Azure](azure-key-vault-integration-configure.md).

### <a name="storage-configuration"></a>Configuración de almacenamiento

En la pestaña **Configuración de SQL Server**, en **Configuración de almacenamiento**, seleccione **Cambiar configuración** para abrir la página Performance Optimized Storage Configuration (Configuración de almacenamiento optimizada para rendimiento) y especificar los requisitos de almacenamiento.

![Configuración de almacenamiento de máquina virtual SQL](./media/create-sql-vm-portal/sql-vm-storage-configuration-provisioning.png)

En **Storage optimized for**(Optimización de almacenamiento para), seleccione una de las siguientes opciones:

* **General** es la configuración predeterminada y es compatible con la mayoría de las cargas de trabajo.
* **Procesamiento de transacciones** optimiza el almacenamiento en las cargas de trabajo OLTP de las bases de datos tradicionales.
* **Almacenamiento de datos** optimiza el almacenamiento para cargas de trabajo de informes y análisis.

![Configuración de almacenamiento de máquina virtual SQL](./media/create-sql-vm-portal/sql-vm-storage-configuration.png)

Puede dejar los valores predeterminados, o bien puede cambiar manualmente la topología de almacenamiento para que se adapte a sus necesidades de IOPS. Para más información, consulte la [configuración del almacenamiento](storage-configuration.md). 

### <a name="sql-server-license"></a>Licencia de SQL Server

Si es cliente de Software Assurance, puede utiliza [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) para utilizar su propia licencia de SQL Server y ahorrar en recursos. 

![Licencia de máquina virtual SQL](./media/create-sql-vm-portal/azure-sqlvm-license.png)

### <a name="automated-patching"></a>Aplicación de revisiones automatizada

**Automated patching** está habilitada de forma predeterminada. La aplicación de revisiones automatizada permite a Azure aplicar automáticamente las revisiones de SQL Server y del sistema operativo. Especifique un día de la semana, la hora y la duración de una ventana de mantenimiento. Azure realiza la aplicación de revisión en esta ventana de mantenimiento. La programación de la ventana de mantenimiento utiliza la configuración regional de la máquina virtual. Si no desea que Azure efectúe automáticamente la aplicación de revisiones de SQL Server y del sistema operativo, seleccione **Deshabilitar**.  

![Aplicación de revisiones automatizada de máquina virtual SQL](./media/create-sql-vm-portal/azure-sqlvm-automated-patching.png)

Para más información, consulte [Automated Patching para SQL Server en Azure Virtual Machines](automated-patching.md).

### <a name="automated-backup"></a>Copia de seguridad automatizada

Habilite las copias de seguridad automáticas en todas las bases de datos en **Copia de seguridad automatizada**. La copia de seguridad automatizada está deshabilitada de forma predeterminada.

Al habilitar la copia de seguridad automatizada de SQL, puede configurar las siguientes opciones:

* El período de retención (días) de las copias de seguridad
* La cuenta de almacenamiento que se utilizará para las copias de seguridad
* La opción de cifrado y la contraseña de las copias de seguridad
* La copia de seguridad de bases de datos del sistema
* La configuración de la programación de copia de seguridad

Para cifrar la copia de seguridad, seleccione **Habilitar**. Después, especifique un valor en **Contraseña**. Azure crea un certificado para cifrar las copias de seguridad y utiliza la contraseña especificada para proteger ese certificado. De manera predeterminada, la programación se establece automáticamente, pero se puede crear una programación manual si se selecciona **Manual**. 

![Copias de seguridad automáticas de la máquina virtual SQL](./media/create-sql-vm-portal/automated-backup.png)

Para obtener más información, vea [Automated Backup para SQL Server en Azure Virtual Machines](automated-backup-sql-2014.md).


### <a name="machine-learning-services"></a>Machine Learning Services

Tiene la opción de habilitar [Machine Learning Services](/sql/advanced-analytics/). Esta opción le permite usar el aprendizaje automático con Python y R en SQL Server 2017. Seleccione **Habilitar** en la ventana **Configuración de SQL Server**.


## <a name="4-review--create"></a>4. Revisar y crear

En la pestaña **Revisar y crear**:
1. Revise el resumen.
1. Seleccione **Crear** para crear la instancia de SQL Server, el grupo de recursos y los recursos especificados para esta máquina virtual.

Puede supervisar la implementación desde Azure Portal. En el botón **Notificaciones** de la parte superior de la pantalla, se muestra el estado básico de la implementación.

> [!NOTE]
> Un ejemplo de tiempo para que Azure implemente una VM con SQL Server: Una VM con SQL Server de prueba aprovisionada en la región Este de EE. UU. con la configuración predeterminada tarda aproximadamente 12 minutos en completarse. Podría experimentar tiempos de implementación más rápidos o más lentos en función de su región y de la configuración seleccionada.

## <a name="open-the-vm-with-remote-desktop"></a><a id="remotedesktop"></a>Apertura de la máquina virtual con Escritorio remoto

Use los pasos siguientes para conectarse a la máquina virtual de SQL Server con Protocolo de Escritorio remoto (RDP):

[!INCLUDE [Connect to SQL Server VM with remote desktop](../../../../includes/virtual-machines-sql-server-remote-desktop-connect.md)]

Después de conectarse a la máquina virtual de SQL Server, puede iniciar SQL Server Management Studio y conectarse con la autenticación de Windows mediante sus credenciales de administrador local. Si ha habilitado la autenticación de SQL Server, también puede conectarse con la autenticación de SQL mediante el inicio de sesión de SQL y la contraseña configurada durante el aprovisionamiento.

El acceso a la máquina le permite cambiar directamente la máquina y la configuración de SQL Server según sus necesidades. Por ejemplo, podría configurar el firewall o cambiar la configuración de SQL Server.

## <a name="connect-to-sql-server-remotely"></a><a id="connect"></a> Conexión remota a SQL Server

En este tutorial, seleccionó el acceso **Público** para la máquina virtual y **Autenticación de SQL Server**. Estos valores configuran automáticamente la máquina virtual para permitir conexiones de SQL Server desde cualquier cliente a través de Internet (suponiendo que tengan el inicio de sesión SQL correcto).

> [!NOTE]
> Si no ha seleccionado la opción Pública durante el aprovisionamiento, puede cambiar la configuración de la conectividad SQL a través del portal después de aprovisionar. Para más información, consulte [Cambio de la configuración de conectividad SQL](ways-to-connect-to-sql.md#change).

En las secciones siguientes se muestra cómo conectarse a la instancia de VM con SQL Server a través de Internet.

[!INCLUDE [Connect to SQL Server in a VM Resource Manager](../../../../includes/virtual-machines-sql-server-connection-steps-resource-manager.md)]

  > [!NOTE]
  > En este ejemplo se usa el puerto común 1433. Sin embargo, este valor deberá modificarse si se especificó otro puerto (por ejemplo, 1401) durante la implementación de la VM con SQL Server. 


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el uso de SQL Server en Azure, consulte Ia página sobre [SQL Server en Azure Virtual Machines](sql-server-on-azure-vm-iaas-what-is-overview.md) y las [Preguntas más frecuentes](frequently-asked-questions-faq.md).
