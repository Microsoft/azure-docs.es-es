---
title: Dispositivo con Azure Migrate
description: Proporciona información general sobre el dispositivo de Azure Migrate usado en la evaluación y migración del servidor.
ms.topic: conceptual
ms.date: 05/04/2020
ms.openlocfilehash: a313d7b964e6ea849acb5b034ed55975b8c5b524
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88927528"
---
# <a name="azure-migrate-appliance"></a>Dispositivo con Azure Migrate

En este artículo se resumen los requisitos previos y los requisitos de compatibilidad del dispositivo de Azure Migrate. 

## <a name="deployment-scenarios"></a>Escenarios de implementación

El dispositivo Azure Migrate se usa en los escenarios siguientes.

**Escenario** | **Herramienta** | **Se usa para** 
--- | --- | ---
**Valoración de máquinas virtuales de VMware** | Server Assessment de Azure Migrate | Detectar máquinas virtuales de VMware<br/><br/> Detectar aplicaciones y dependencias de la máquina<br/><br/> Recopilar metadatos de máquinas y metadatos de rendimiento para las evaluaciones
**Migración sin agentes de máquinas virtuales de VMware** | Server Migration de Azure Migrate | Detectar máquinas virtuales de VMware <br/><br/> Replicar máquinas virtuales de VMware con la migración sin agentes
**Valoración de máquinas virtuales de Hyper-V** | Server Assessment de Azure Migrate | Detectar máquinas virtuales de Hyper-V<br/><br/> Recopilar metadatos de máquinas y metadatos de rendimiento para las evaluaciones
**Valoración de máquina física** |  Server Assessment de Azure Migrate |  Descubra servidores físicos (o máquinas virtuales que se tratan como servidores físicos).<br/><br/> Recopilar metadatos de máquinas y metadatos de rendimiento para las evaluaciones

## <a name="deployment-methods"></a>Métodos de implementación

Para implementar el dispositivo se pueden usar un par de métodos:

- El dispositivo se puede implementar mediante una plantilla para máquinas virtuales de VMware y máquinas virtuales de Hyper-V (plantilla OVA en el caso de VMware o VHD en el de Hyper-V).
- Si no desea usar una plantilla, puede implementar el dispositivo para VMware o Hyper-V mediante un script de PowerShell.
- En Azure Government, el dispositivo se debe implementar mediante un script.
- En el caso de los servidores físicos, el dispositivo siempre se implementa mediante un script.
- Los vínculos de descarga están disponibles en las tablas siguientes.


## <a name="appliance---vmware"></a>Dispositivo: VMware 

En la tabla siguiente se resumen los requisitos del dispositivo de Azure Migrate para VMware.

**Requisito** | **VMware** 
--- | ---
**Permisos** | Para acceder a la aplicación web del dispositivo de forma local o remota, debe ser un administrador de dominio o un administrador local en la máquina del dispositivo.
**Componentes del dispositivo** | El dispositivo tiene los componentes siguientes:<br/><br/> - **Aplicación de administración**: una aplicación web para la entrada del usuario durante la implementación del dispositivo. Se usa a la hora de evaluar las máquinas para su migración a Azure.<br/> - **Agente de detección**: un agente que reúne los datos de configuración de la máquina. Se usa a la hora de evaluar las máquinas para su migración a Azure.<br/>- **Agente recopilador**: un agente que recopila datos de rendimiento. Se usa a la hora de evaluar las máquinas para su migración a Azure.<br/>- **Servicio de actualización automática**: actualiza los componentes del dispositivo (se ejecuta cada 24 horas).<br/>- **Agente de recuperación de datos (DRA)** : organiza la replicación de VM y coordina la comunicación entre las máquinas replicadas y Azure. Solo se usa al replicar máquinas virtuales de VMware en Azure mediante la migración sin agentes.<br/>- **Puerta de enlace**: Envía los datos replicados a Azure. Solo se usa al replicar máquinas virtuales de VMware en Azure mediante la migración sin agentes.
**Implementación admitida** | Impleméntelo como máquina virtual de VMware mediante la plantilla de OVA.<br/><br/> Impleméntelo como máquina física o máquina virtual de VMware mediante el script de instalación de PowerShell.
**Compatibilidad con proyectos** |  Un dispositivo solo puede estar asociado a un proyecto. <br/> Se puede asociar cualquier número de dispositivos a un solo proyecto.<br/> 
**Límites de detección** | Un dispositivo puede detectar hasta 10 000 máquinas virtuales de VMware en una instancia de vCenter Server.<br/> Un dispositivo solo puede conectarse a una instancia de vCenter Server.
**Plantilla de OVA** | Descarga desde el portal o desde [aquí](https://go.microsoft.com/fwlink/?linkid=2140333).<br/><br/> El tamaño de la descarga es de 11,6 GB.<br/><br/> La plantilla del dispositivo descargada incluye una licencia de evaluación de Windows Server 2016 que es válida durante 180 días. Si el período de evaluación está a punto de expirar, se recomienda descargar e implementar un nuevo dispositivo, o bien activar la licencia del sistema operativo de la máquina virtual del dispositivo.
**Script de PowerShell** | Consulte este [artículo](./deploy-appliance-script.md#set-up-the-appliance-for-vmware).<br/><br/> 
**Software/hardware** |  El dispositivo debe ejecutarse en una máquina con Windows Server 2016, 32 GB de RAM, 8 vCPU, alrededor de 80 GB de almacenamiento en disco y un conmutador virtual externo.<br/> El dispositivo requiere acceso a Internet, ya sea directamente o a través de un proxy.<br/><br/> Si ejecuta el dispositivo en una máquina virtual de VMware, necesitará recursos suficientes en vCenter Server para asignar una máquina virtual que cumpla los requisitos.<br/><br/> Si ejecuta el dispositivo en una máquina física, asegúrese de que está ejecutando Windows Server 2016 y que cumple los requisitos de hardware.
**Requisitos de VMware** | Si el dispositivo se implementa como una máquina virtual de VMware, debe implementarse en un host ESXi que ejecute la versión 5.5 o una posterior.<br/><br/> vCenter Server que ejecute la versión 5.5, 6.0, 6.5 o 6.7.
**VDDK (migración sin agentes)** | Si el dispositivo se implementa como una máquina virtual de VMware y ejecuta una migración sin agentes, es necesario tener instalado el VDDK de VMware vSphere en la máquina virtual del dispositivo.
**Valor del código hash: OVA** | [Compruebe](tutorial-assess-vmware.md#verify-security) los valores hash de la plantilla de OVA.
**Valor del código hash: script de PowerShell** | [Compruebe](deploy-appliance-script.md#verify-file-security) los valores hash del script de PowerShell.




## <a name="appliance---hyper-v"></a>Dispositivo: Hyper-V

**Requisito** | **Hyper-V** 
--- | ---
**Permisos** | Para acceder a la aplicación web del dispositivo de forma local o remota, debe ser un administrador de dominio o un administrador local en la máquina del dispositivo.
**Componentes del dispositivo** | El dispositivo tiene los componentes siguientes:<br/><br/>- **Aplicación de administración**: una aplicación web para la entrada del usuario durante la implementación del dispositivo. Se usa a la hora de evaluar las máquinas para su migración a Azure.<br/> - **Agente de detección**: un agente que reúne los datos de configuración de la máquina. Se usa a la hora de evaluar las máquinas para su migración a Azure.<br/>- **Agente recopilador**: un agente que recopila datos de rendimiento. Se usa a la hora de evaluar las máquinas para su migración a Azure.<br/>- **Servicio de actualización automática**: actualiza los componentes del dispositivo (se ejecuta cada 24 horas).
**Implementación admitida** | Impleméntelo como máquina virtual de Hyper-V mediante una plantilla de VHD.<br/><br/> Impleméntelo como máquina física o máquina virtual de Hyper-V mediante el script de instalación de PowerShell.
**Compatibilidad con proyectos** |  Un dispositivo solo puede estar asociado a un proyecto. <br/> Se puede asociar cualquier número de dispositivos a un solo proyecto.<br/> 
**Límites de detección** | Un dispositivo puede detectar hasta 5000 VM de Hyper-V.<br/> Un dispositivo puede conectarse hasta a 300 hosts de Hyper-V.
**Plantilla de VHD** | Carpeta comprimida que incluye el VHD. Descarga desde el portal o desde [aquí](https://go.microsoft.com/fwlink/?linkid=2140422).<br/><br/> El tamaño de la descarga es de 10,4 GB.<br/><br/> La plantilla del dispositivo descargada incluye una licencia de evaluación de Windows Server 2016 que es válida durante 180 días. Si el período de evaluación está a punto de expirar, se recomienda descargar e implementar un nuevo dispositivo, o bien activar la licencia del sistema operativo de la máquina virtual del dispositivo.
**Script de PowerShell** | Consulte este [artículo](./deploy-appliance-script.md#set-up-the-appliance-for-hyper-v).<br/><br/> 
**Software/hardware***   |  El dispositivo debe ejecutarse en una máquina con Windows Server 2016, 16 GB de RAM, 8 vCPU, alrededor de 80 GB de almacenamiento en disco y un conmutador virtual externo.<br/> El dispositivo necesita una dirección IP estática o dinámica y requiere acceso a Internet, ya sea directamente o a través de un proxy.<br/><br/> Si se ejecuta el dispositivo como una máquina virtual de Hyper-V, se necesitan recursos suficientes en el host de Hyper-V para asignar 16 GB de RAM, 8 vCPU, alrededor de 80 GB de espacio de almacenamiento y un conmutador externo para la máquina virtual del dispositivo.<br/><br/> Si ejecuta el dispositivo en una máquina física, asegúrese de que está ejecutando Windows Server 2016 y que cumple los requisitos de hardware. 
**Requisitos de Hyper-V** | Si se implementa el dispositivo con la plantilla de VHD, la máquina virtual del dispositivo que proporciona Azure Migrate es la versión 5.0 de la máquina virtual de Hyper-V.<br/><br/> El host de Hyper-V debe ejecutarse en Windows Server 2012 R2 o posterior. 
**Valor del código hash: VHD** | [Compruebe](tutorial-assess-hyper-v.md#verify-security) los valores hash de la plantilla de VHD.
**Valor del código hash: script de PowerShell** | [Compruebe](deploy-appliance-script.md#verify-file-security) los valores hash del script de PowerShell.


## <a name="appliance---physical"></a>Dispositivo: físico

**Requisito** | **Físico** 
--- | ---
**Permisos** | Para acceder a la aplicación web del dispositivo de forma local o remota, debe ser un administrador de dominio o un administrador local en la máquina del dispositivo.
**Componentes del dispositivo** | El dispositivo tiene los componentes siguientes: <br/><br/> - **Aplicación de administración**: una aplicación web para la entrada del usuario durante la implementación del dispositivo. Se usa a la hora de evaluar las máquinas para su migración a Azure.<br/> - **Agente de detección**: un agente que reúne los datos de configuración de la máquina. Se usa a la hora de evaluar las máquinas para su migración a Azure.<br/>- **Agente recopilador**: un agente que recopila datos de rendimiento. Se usa a la hora de evaluar las máquinas para su migración a Azure.<br/>- **Servicio de actualización automática**: actualiza los componentes del dispositivo (se ejecuta cada 24 horas).
**Implementación admitida** | Impleméntelo como una máquina física dedicada o una máquina virtual mediante el script de instalación de PowerShell. El script está disponible para su descarga desde el portal.
**Compatibilidad con proyectos** |  Un dispositivo solo puede estar asociado a un proyecto. <br/> Se puede asociar cualquier número de dispositivos a un solo proyecto.<br/> 
**Límites de detección** | Un dispositivo puede detectar hasta 1000 servidores físicos.
**Script de PowerShell** | Descarga del script (AzureMigrateInstaller.ps1) en una carpeta comprimida desde el portal o desde [aquí](https://go.microsoft.com/fwlink/?linkid=2140334). [Más información](tutorial-assess-physical.md#set-up-the-azure-migrate-appliance).<br/><br/> El tamaño de la descarga es de 85 MB.
**Software/hardware** |  El dispositivo debe ejecutarse en una máquina con Windows Server 2016, 16 GB de RAM, 8 vCPU, alrededor de 80 GB de almacenamiento en disco y un conmutador virtual externo.<br/> El dispositivo necesita una dirección IP estática o dinámica y requiere acceso a Internet, ya sea directamente o a través de un proxy.<br/><br/> Si ejecuta el dispositivo en una máquina física, asegúrese de que está ejecutando Windows Server 2016 y que cumple los requisitos de hardware.<br/> No se admite la ejecución del dispositivo en una máquina con Windows Server 2019.
**Valor del código hash** | [Compruebe](deploy-appliance-script.md#verify-file-security) los valores hash del script de PowerShell.

## <a name="url-access"></a>acceso URL

El dispositivo de Azure Migrate necesita conectividad a Internet.

- Al implementar el dispositivo, Azure Migrate realiza una comprobación de conectividad con las direcciones URL necesarias.
- Debe permitir el acceso a todas las direcciones URL de la lista. Si solo está realizando la evaluación, solo puede omitir las direcciones URL marcadas como necesarias para la migración sin agente de VMware.
-  Si usa un proxy basado en URL para conectarse a Internet, asegúrese de que el proxy resuelve los registros CNAME recibidos al buscar estas direcciones.

### <a name="public-cloud-urls"></a>Direcciones URL de la nube pública

**URL** | **Detalles**  
--- | --- |
*.portal.azure.com  | Acceda a Azure Portal.
\* .windows.net <br/> *.msftauth.net <br/> *.msauth.net <br/> *.microsoft.com <br/> *.live.com | Inicie sesión en la suscripción de Azure.
*.microsoftonline.com <br/> *.microsoftonline-p.com | Cree aplicaciones de Azure Active Directory (AD) para que el dispositivo se comunique con Azure Migrate.
management.azure.com | Cree aplicaciones de Azure AD para que el dispositivo se comunique con el servicio Azure Migrate.
*.services.visualstudio.com | Cargue los registros de aplicaciones que se usan para la supervisión interna.
*.vault.azure.net | Administre secretos en Azure Key Vault. Nota: Asegúrese de que las máquinas que se replican tengan acceso a ella.
aka.ms/* | Permiso de acceso a vínculos aka. Se usa para las actualizaciones del dispositivo de Azure Migrate.
download.microsoft.com/download | Permita descargas de Microsoft.
*.servicebus.windows.net | Comunicación entre el dispositivo y el servicio Azure Migrate.
*.discoverysrv.windowsazure.com <br/> *.migration.windowsazure.com | Conéctese a las direcciones URL del servicio Azure Migrate.
*.hypervrecoverymanager.windowsazure.com | **Se usa para la migración sin agentes de VMware.**<br/><br/> Conéctese a las direcciones URL del servicio Azure Migrate.
*.blob.core.windows.net |  **Se usa para la migración sin agentes de VMware.**<br/><br/>Cargue los datos al almacenamiento para la migración.

### <a name="government-cloud-urls"></a>Direcciones URL de la nube del gobierno

**URL** | **Detalles**  
--- | --- |
*.portal.azure.us  | Acceda a Azure Portal.
graph.windows.net | Inicie sesión en la suscripción de Azure.
login.microsoftonline.us  | Cree aplicaciones de Azure Active Directory (AD) para que el dispositivo se comunique con Azure Migrate.
management.usgovcloudapi.net | Cree aplicaciones de Azure AD para que el dispositivo se comunique con el servicio Azure Migrate.
dc.services.visualstudio.com | Cargue los registros de aplicaciones que se usan para la supervisión interna.
*.vault.usgovcloudapi.net | Administre secretos en Azure Key Vault.
aka.ms/* | Permiso de acceso a vínculos aka. Se usa para las actualizaciones del dispositivo de Azure Migrate.
download.microsoft.com/download | Permita descargas de Microsoft.
*.servicebus.usgovcloudapi.net  | Comunicación entre el dispositivo y el servicio Azure Migrate.
*.discoverysrv.windowsazure.us <br/> *.migration.windowsazure.us | Conéctese a las direcciones URL del servicio Azure Migrate.
*.hypervrecoverymanager.windowsazure.us | **Se usa para la migración sin agentes de VMware.**<br/><br/> Conéctese a las direcciones URL del servicio Azure Migrate.
*.blob.core.usgovcloudapi.net  |  **Se usa para la migración sin agentes de VMware.**<br/><br/>Cargue los datos al almacenamiento para la migración.
*.applicationinsights.us | Cargue los registros de aplicaciones que se usan para la supervisión interna.





## <a name="collected-data---vmware"></a>Datos recopilados: VMware

El dispositivo recopila metadatos, datos de rendimiento y datos de análisis de dependencias (si se usa [análisis de dependencias](concepts-dependency-visualization.md) sin agente).

### <a name="metadata"></a>Metadatos

Los metadatos que descubre el dispositivo de Azure Migrate ayudan a averiguar si las máquinas y las aplicaciones están listas para la migración a Azure, el tamaño correcto de las máquinas y las aplicaciones, los planes de costos y el análisis de las dependencias de la aplicación. Microsoft no usa estos datos en ninguna auditoría de cumplimiento de licencias.

Esta es la lista completa de metadatos de VM de VMware que el dispositivo recopila y envía a Azure.

**DATOS** | **CONTADOR**
--- | --- 
**Detalles de la máquina** | 
Id. de VM | vm.Config.InstanceUuid 
Nombre de la máquina virtual | vm.Config.Name
Id. de vCenter Server | VMwareClient.Instance.Uuid
Descripción de la VM | vm.Summary.Config.Annotation
Nombre del producto de licencia | vm.Client.ServiceContent.About.LicenseProductName
Tipo de sistema operativo | vm.SummaryConfig.GuestFullName
Tipo de arranque | vm.Config.Firmware
Número de núcleos | vm.Config.Hardware.NumCPU
Memoria (MB) | vm.Config.Hardware.MemoryMB
Número de discos | vm.Config.Hardware.Device.ToList().FindAll(x => es VirtualDisk).count
Lista de tamaños de los discos | vm.Config.Hardware.Device.ToList().FindAll(x => es VirtualDisk)
Lista de adaptadores de red | vm.Config.Hardware.Device.ToList().FindAll(x => es VirtualEthernet).count
Uso de CPU | cpu.usage.average
Uso de memoria |mem.usage.average
**Detalles por disco** | 
Valor de clave del disco | disk.Key
Número de unidades de disco | disk.UnitNumber
Valor de clave de controladora de disco | disk.ControllerKey.Value
Gigabytes aprovisionados | virtualDisk.DeviceInfo.Summary
Nombre del disco | Valor generado mediante disk.UnitNumber, disk.Key y disk.ControllerKey.Value
Operaciones de lectura por segundo | virtualDisk.numberReadAveraged.average
Operaciones de escritura por segundo | virtualDisk.numberWriteAveraged.average
Rendimiento de lectura (MB por segundo) | virtualDisk.read.average
Rendimiento de escritura (MB por segundo) | virtualDisk.write.average
**Detalles por NIC** | 
Nombre del adaptador de red | nic.Key
Dirección MAC | ((VirtualEthernetCard)nic).MacAddress
Direcciones IPv4 | vm.Guest.Net
Direcciones IPv6 | vm.Guest.Net
Rendimiento de lectura (MB por segundo) | net.received.average
Rendimiento de escritura (MB por segundo) | net.transmitted.average
**Detalles de la ruta de acceso de inventario** | 
Nombre | container.GetType().Name
Tipo de objeto secundario | container.ChildType
Información de referencia | container.MoRef
Detalles del objeto primario | Container.Parent
Detalles de la carpeta por VM | ((Folder)container).ChildEntity.Type
Detalles del centro de datos por VM | ((Datacenter)container).VmFolder
Detalles del centro de datos por carpeta de host | ((Datacenter)container).HostFolder
Detalles del clúster por host | ((ClusterComputeResource)container).Host
Detalles del host por VM | ((HostSystem)container).VM

### <a name="performance-data"></a>Datos de rendimiento


Estos son los datos de rendimiento de las VM de VMware que el dispositivo recopila y envía a Azure.

**Data** | **Contador** | **Impacto en la evaluación**
--- | --- | ---
Uso de CPU | cpu.usage.average | Tamaño/costo de VM recomendado
Uso de memoria | mem.usage.average | Tamaño/costo de VM recomendado
Rendimiento de lectura de disco (MB por segundo) | virtualDisk.read.average | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de VM
Rendimiento de escrituras de discos (MB por segundo) | virtualDisk.write.average | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de VM
Operaciones de lectura de disco por segundo | virtualDisk.numberReadAveraged.average | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de VM
Operaciones de escritura de discos por segundo | virtualDisk.numberWriteAveraged.average  | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de VM
Rendimiento de lectura de NIC (MB por segundo) | net.received.average | Cálculo del tamaño de VM
Rendimiento de escrituras de adaptadores de red (MB por segundo) | net.transmitted.average  |Cálculo del tamaño de VM


### <a name="installed-apps-metadata"></a>Metadatos de las aplicaciones instaladas

La detección de aplicaciones recopila los datos del sistema operativo y las aplicaciones instaladas.

#### <a name="windows-vm-apps-data"></a>Datos de aplicaciones de VM Windows

Estos son los datos de aplicaciones instaladas que el dispositivo recopila de cada VM habilitada para la detección de aplicaciones. Estos datos se envían a Azure.

**Data** | **Ubicación del registro** | **Clave**
--- | --- | ---
Nombre de la aplicación  | HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* <br/> HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*  | DisplayName
Versión  | HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*  <br/> HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*  | DisplayVersion 
Proveedor  | HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*  <br/> HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*  | Publicador

#### <a name="windows-vm-features-data"></a>Datos de características de VM Windows

Estos son los datos de características que el dispositivo recopila de cada VM habilitada para la detección de aplicaciones. Estos datos se envían a Azure.

**Data**  | **Cmdlet de PowerShell** | **Propiedad**
--- | --- | ---
Nombre  | Get-WindowsFeature  | Nombre
Tipo de característica | Get-WindowsFeature  | FeatureType
Parent  | Get-WindowsFeature  | Parent

#### <a name="windows-vm-sql-server-metadata"></a>Metadatos de SQL Server para la VM Windows

Estos son los metadatos de SQL Server que el dispositivo recopila de las VM que ejecutan Microsoft SQL Server habilitadas para la detección de aplicaciones. Estos datos se envían a Azure.

**Data**  | **Ubicación del registro**  | **Clave**
--- | --- | ---
Nombre  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\Instance Names\SQL  | installedInstance
Edición  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\\\<InstanceName>\Setup  | Edición 
Service Pack  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\\\<InstanceName>\Setup  | SP
Versión  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\\\<InstanceName>\Setup  | Versión 

#### <a name="windows-vm-operating-system-data"></a>Datos del sistema operativo de VM Windows

Estos son los datos del sistema operativo que el dispositivo recopila de cada VM habilitada para la detección de aplicaciones. Estos datos se envían a Azure.

data  | Clase WMI  | Propiedad de clase WMI
--- | --- | ---
Nombre  | Win32_operatingsystem  | Caption
Versión  | Win32_operatingsystem  | Versión
Architecture  | Win32_operatingsystem  | OSArchitecture

#### <a name="linux-vm-apps-data"></a>Datos de aplicaciones de VM Linux

Estos son los datos de aplicaciones instaladas que el dispositivo recopila de cada VM habilitada para la detección de aplicaciones. En función del sistema operativo de la VM, se ejecutan uno o varios comandos. Estos datos se envían a Azure.

data  | Get-Help
--- | --- 
Nombre | rpm, dpkg-query, snap
Versión | rpm, dpkg-query, snap
Proveedor | rpm, dpkg-query, snap

#### <a name="linux-vm-operating-system-data"></a>Datos del sistema operativo de VM Linux

Estos son los datos del sistema operativo que el dispositivo recopila de cada VM habilitada para la detección de aplicaciones. Estos datos se envían a Azure.

**Data**  | **Comando** 
--- | --- | ---
Nombre <br/> version | Recopilado de uno o varios de los archivos siguientes:<br/> <br/>/etc/os-release  <br> /usr/lib/os-release  <br> /etc/enterprise-release  <br> /etc/redhat-release  <br> /etc/oracle-release  <br> /etc/SuSE-release  <br> /etc/lsb-release  <br> /etc/debian_version 
Architecture | uname


### <a name="app-dependencies-metadata"></a>Metadatos de dependencias de la aplicación

El análisis de dependencias sin agente recopila datos de conexión y de proceso.

#### <a name="windows-vm-app-dependencies-data"></a>Datos de dependencias de aplicaciones de VM Windows

Estos son los datos de conexión que el dispositivo recopila de cada máquina virtual habilitada para el análisis de dependencias sin agente. Estos datos se envían a Azure.

**Data** | **Comando utilizado** 
--- | --- 
Puerto local | netstat
Dirección IP local | netstat
Puerto remoto | netstat
Dirección IP remota | netstat
Estado de conexión de TCP | netstat
Identificador del proceso | netstat
Número de conexiones activas | netstat


Estos son los datos de proceso que el dispositivo recopila de cada máquina virtual habilitada para el análisis de dependencias sin agente. Estos datos se envían a Azure.

**Data** | **Clase WMI** | **Propiedad de clase WMI**
--- | --- | ---
Nombre del proceso | Win32_Process | ExecutablePath _s
Argumentos de procesos | Win32_Process | CommandLine
Nombre de la aplicación | Win32_Process | Parámetro VersionInfo.ProductName de la propiedad ExecutablePath

#### <a name="linux-vm-app-dependencies-data"></a>Datos de dependencias de aplicaciones de VM Linux

Estos son los datos de conexión y de proceso que el dispositivo recopila de cada máquina virtual Linux habilitada para el análisis de dependencias sin agente. Estos datos se envían a Azure.

**Data** | **Comando utilizado** 
--- | ---
Puerto local | netstat 
Dirección IP local | netstat 
Puerto remoto | netstat 
Dirección IP remota | netstat 
Estado de conexión de TCP | netstat 
Número de conexiones activas | netstat
Identificador del proceso  | netstat 
Nombre del proceso | ps
Argumentos de procesos | ps
Nombre de la aplicación | dpkg o rpm



## <a name="collected-data---hyper-v"></a>Datos recopilados: Hyper-V

El dispositivo recopila metadatos, datos de rendimiento y datos de análisis de dependencias (si se usa [análisis de dependencias](concepts-dependency-visualization.md) sin agente).

### <a name="metadata"></a>Metadatos
Los metadatos que descubre el dispositivo de Azure Migrate ayudan a averiguar si las máquinas y las aplicaciones están listas para la migración a Azure, el tamaño correcto de las máquinas y las aplicaciones, los planes de costos y el análisis de las dependencias de la aplicación. Microsoft no usa estos datos en ninguna auditoría de cumplimiento de licencias.

Esta es la lista completa de metadatos de VM de Hyper-V que el dispositivo recopila y envía a Azure.

**DATOS** | **CLASE WMI** | **PROPIEDAD DE CLASE WMI**
--- | --- | ---
**Detalles de la máquina** | 
Número de serie de BIOS _ Msvm_BIOSElement | BIOSSerialNumber
Tipo de VM (gen. 1 o 2) | Msvm_VirtualSystemSettingData | VirtualSystemSubType
Nombre para mostrar de VM | Msvm_VirtualSystemSettingData | ElementName
Versión de VM | Msvm_ProcessorSettingData | VirtualQuantity
Memoria (bytes) | Msvm_MemorySettingData | VirtualQuantity
Memoria máxima que se puede consumir por VM | Msvm_MemorySettingData | Límite
Memoria dinámica habilitada | Msvm_MemorySettingData | DynamicMemoryEnabled
Nombre/versión/FQDN del sistema operativo | Msvm_KvpExchangeComponent | Datos de nombre de GuestIntrinsicExchangeItems
Estado de energía de VM | Msvm_ComputerSystem | EnabledState
**Detalles por disco** | 
Identificador de disco | Msvm_VirtualHardDiskSettingData | VirtualDiskId
Tipo de disco duro virtual | Msvm_VirtualHardDiskSettingData | Tipo
Tamaño de disco duro virtual | Msvm_VirtualHardDiskSettingData | MaxInternalSize
Primario de disco duro virtual | Msvm_VirtualHardDiskSettingData | ParentPath
**Detalles por NIC** | 
Direcciones IP (NIC sintéticas) | Msvm_GuestNetworkAdapterConfiguration | IPAddresses
DHCP habilitado (NIC sintéticas) | Msvm_GuestNetworkAdapterConfiguration | DHCPEnabled
Id. de NIC (NIC sintéticas) | Msvm_SyntheticEthernetPortSettingData | InstanceID
Dirección MAC de NIC (NIC sintéticas) | Msvm_SyntheticEthernetPortSettingData | Dirección
Id. de NIC (NIC heredadas) | Datos de MsvmEmulatedEthernetPortSetting | InstanceID
Id. de MAC de NIC (NIC heredadas) | Datos de MsvmEmulatedEthernetPortSetting | Dirección

### <a name="performance-data"></a>Datos de rendimiento

Estos son los datos de rendimiento de las máquinas virtuales de Hyper-V que el dispositivo recopila y envía a Azure.

**Clase de contador de rendimiento** | **Contador** | **Impacto en la evaluación**
--- | --- | ---
Procesador virtual del hipervisor de Hyper-V | Tiempo de ejecución de invitado (%) | Tamaño/costo de VM recomendado
VM de Memoria dinámica de Hyper-V | Presión actual (%)<br/> Memoria física visible de invitado (MB) | Tamaño/costo de VM recomendado
Dispositivo de almacenamiento virtual de Hyper-V | Bytes de lectura/segundo | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de VM
Dispositivo de almacenamiento virtual de Hyper-V | Bytes de escritura/segundo | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de VM
Adaptador de red virtual de Hyper-V | Bytes recibidos/segundo | Cálculo del tamaño de VM
Adaptador de red virtual de Hyper-V | Bytes enviados/segundo | Cálculo del tamaño de VM

- El uso de CPU es la suma de todo el uso, para todos los procesadores virtuales conectados a una VM.
- El uso de memoria es (presión actual * memoria física visible del invitado)/100.
- Los valores de uso de disco y red se recopilan de los contadores de rendimiento de Hyper-V enumerados.


## <a name="collected-data---physical"></a>Datos recopilados: físicos

El dispositivo recopila metadatos, datos de rendimiento y datos de análisis de dependencias (si se usa [análisis de dependencias](concepts-dependency-visualization.md) sin agente).

### <a name="windows-metadata"></a>Metadatos de Windows

Los metadatos que descubre el dispositivo de Azure Migrate ayudan a averiguar si las máquinas y las aplicaciones están listas para la migración a Azure, el tamaño correcto de las máquinas y las aplicaciones, los planes de costos y el análisis de las dependencias de la aplicación. Microsoft no usa estos datos en ninguna auditoría de cumplimiento de licencias.

Esta es la lista completa de metadatos de Windows Server que el dispositivo recopila y envía a Azure.

**DATOS** | **CLASE WMI** | **PROPIEDAD DE CLASE WMI**
--- | --- | ---
FQDN | Win32_ComputerSystem | Dominio, nombre, PartOfDomain
Número de núcleos del procesador | Win32_PRocessor | NumberOfCores
Memoria asignada | Win32_ComputerSystem | TotalPhysicalMemory
Número de serie del BIOS | Win32_ComputerSystemProduct | IdentifyingNumber
GUID del BIOS | Win32_ComputerSystemProduct | UUID
Tipo de arranque | Win32_DiskPartition | Busque la partición con el tipo = **GPT:System** para EFI/BIOS
Nombre del SO | Win32_OperatingSystem | Caption
Versión del SO |Win32_OperatingSystem | Versión
Arquitectura del SO | Win32_OperatingSystem | OSArchitecture
Número de discos | Win32_DiskDrive | Modelo, tamaño, DeviceID, MediaType, nombre
Tamaño del disco | Win32_DiskDrive | Size
Lista de NIC | Win32_NetworkAdapterConfiguration | Descripción, índice
Dirección IP de NIC | Win32_NetworkAdapterConfiguration | IPAddress
Dirección MAC de NIC | Win32_NetworkAdapterConfiguration | MACAddress

### <a name="linux-metadata"></a>Metadatos de Linux

Esta es la lista completa de metadatos del servidor de Linux que el dispositivo recopila y envía a Azure.

**DATOS** | **LINUX** 
--- | --- 
FQDN | cat /proc/sys/kernel/hostname, hostname -f
Número de núcleos del procesador |  /proc/cpuinfo \| awk "/^processor/{print $3}" \| wc -l
Memoria asignada | cat /proc/meminfo \| grep MemTotal \| awk "{printf "%.0f", $2/1024}"
Número de serie del BIOS | lshw \| grep "serial:" \| head -n1 \| awk "{print $2}" <br/> /usr/sbin/dmidecode -t 1 \| grep 'Serial' \| awk '{ $1="" ; $2=""; print}'
GUID del BIOS | cat /sys/class/dmi/id/product_uuid
Tipo de arranque | [ -d /sys/firmware/efi ] && echo EFI \|\| echo BIOS
Versión o nombre del SO | Obtenemos acceso a estos archivos para el nombre y la versión del SO:<br/><br/> /etc/os-release<br/> /usr/lib/os-release <br/> /etc/enterprise-release <br/> /etc/redhat-release<br/> /etc/oracle-release<br/>  /etc/SuSE-release<br/>  /etc/lsb-release  <br/> /etc/debian_version
Arquitectura del SO | Uname -m
Número de discos | fdisk -l \| egrep "Disk.*bytes" \| awk "{print $2}" \| cut -f1 -d ":"
Disco de arranque | df /boot \| sed -n 2p \| awk "{print $1}"
Tamaño del disco | fdisk -l \| egrep "Disk.*bytes" \| egrep $disk: \| awk "{print $5}"
Lista de NIC | ip -o -4 addr show \| awk "{print $2}"
Dirección IP de NIC | ip addr show $nic \| grep inet \| awk "{print $2}" \| cut -f1 -d "/" 
Dirección MAC de NIC | ip addr show $nic \| grep ether  \| awk "{print $2}"

### <a name="windows-performance-data"></a>Datos de rendimiento de Windows

Estos son los datos de rendimiento de Windows Sever que el dispositivo recopila y envía a Azure.

**Data** | **Clase WMI** | **Propiedad de clase WMI**
--- | --- | ---
Uso de CPU | Win32_PerfFormattedData_PerfOS_Processor | PercentIdleTime
Uso de la memoria | Win32_PerfFormattedData_PerfOS_Memory | AvailableMBytes
NIC (recuento) | Win32_PerfFormattedData_Tcpip_NetworkInterface | Obtiene el número de dispositivos de red.
Datos recibidos por NIC | Win32_PerfFormattedData_Tcpip_NetworkInterface  | BytesReceivedPerSec
Datos transmitidos por NIC | BWin32_PerfFormattedData_Tcpip_NetworkInterface | BytesSentPersec
Número de discos | BWin32_PerfFormattedData_PerfDisk_PhysicalDisk | Número de discos
Detalles del disco | Win32_PerfFormattedData_PerfDisk_PhysicalDisk | DiskWritesPerSec, DiskWriteBytesPerSec, DiskReadsPerSec, DiskReadBytesPerSec.

### <a name="linux-performance-data"></a>Datos de rendimiento de Linux

Estos son los datos de rendimiento de Linux que el dispositivo recopila y envía a Azure.

**Data** | **Linux** 
--- | --- 
Uso de CPU | cat /proc/stat/| grep "cpu" /proc/stat
Uso de la memoria | free \| grep Mem \| awk "{print $3/$2 * 100.0}"
NIC (recuento) | lshw -class network \| eth[0-60] \| wc -l
Datos recibidos por NIC | cat /sys/class/net/eth$nic/statistics/rx_bytes
Datos transmitidos por NIC | cat /sys/class/net/eth$nic/statistics/tx_bytes
Número de discos | fdisk -l \| egrep "Disk.*bytes" \| awk "{print $2}" \| cut -f1 -d ":"
Detalles del disco | cat /proc/diskstats


## <a name="appliance-upgrades"></a>Actualizaciones del dispositivo

El dispositivo se actualiza a medida que se actualizan los agentes de Azure Migrate que se ejecutan en el dispositivo. Esto sucede automáticamente porque la actualización automática está habilitada en el dispositivo de forma predeterminada. Puede cambiar esta configuración predeterminada para actualizar los servicios de dispositivo manualmente.

### <a name="turn-off-auto-update"></a>Desactivación de la actualización automática

1. En la máquina que ejecuta el dispositivo, abra el Editor del Registro.
2. Vaya a **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureAppliance**.
3. Para desactivar la actualización automática, cree una clave **AutoUpdate** de clave del Registro con un valor de DWORD de 0.

    ![Definición de clave del Registro](./media/migrate-appliance/registry-key.png)


### <a name="turn-on-auto-update"></a>Activación de la actualización automática

Puede activar la actualización automática mediante cualquiera de estos métodos:

- Eliminando la clave del Registro de AutoUpdate de HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureAppliance.
- Haga clic en **Ver servicios del dispositivo** desde las comprobaciones de actualizaciones más recientes en el panel **Configurar los requisitos previos** para activar la actualización automática.

Para eliminar la clave del Registro:

1. En la máquina que ejecuta el dispositivo, abra el Editor del Registro.
2. Vaya a **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureAppliance**.
3. Elimine la clave del registro **AutoUpdate** que se creó anteriormente para desactivar la actualización automática.

Para realizar la activación desde Appliance Configuration Manager, una vez completada la detección:

1. En el administrador de configuración del dispositivo, vaya al panel **Configurar los requisitos previos**.
2. En la comprobación de actualizaciones más recientes, haga clic en **Ver servicios del dispositivo** y haga clic en el vínculo para activar la actualización automática.

    ![Activar las actualizaciones automáticas](./media/migrate-appliance/autoupdate-off.png)

### <a name="check-the-appliance-services-version"></a>Comprobación de la versión de servicios del dispositivo

Puede comprobar la versión de los servicios del dispositivo con cualquiera de estos métodos:

- En el administrador de configuración del dispositivo, vaya al panel **Configurar los requisitos previos**.
- En la máquina del dispositivo, en el **Panel de control** > **Programas y características**.

Para registrar el administrador de configuración del dispositivo:

1. En el administrador de configuración del dispositivo, vaya al panel **Configurar los requisitos previos**.
2. En la comprobación de actualizaciones más recientes, haga clic en **Ver servicios del dispositivo**.

    ![Comprobación de la versión](./media/migrate-appliance/versions.png)

Para consultarlo en el Panel de control.

1. En el dispositivo, haga clic en **Inicio** > **Panel de control** > **Programas y características**
2. Compruebe las versiones de los servicios de dispositivo en la lista.

    ![Comprobación de la versión en el Panel de control](./media/migrate-appliance/programs-features.png)

### <a name="manually-update-an-older-version"></a>Actualización manual de una versión anterior

Si está ejecutando una versión anterior para cualquiera de los componentes, debe desinstalar el servicio y actualizar manualmente a la versión más reciente.

1. Para buscar las versiones de servicio de dispositivo más recientes, [descargue](https://aka.ms/latestapplianceservices) el archivo LatestComponents.json.
2.    Después de la descarga, abra el archivo LatestComponents.json en el Bloc de notas.
3. Busque la última versión del servicio en el archivo y el vínculo de descarga correspondiente. Por ejemplo:

    "Name": "ASRMigrationWebApp", "DownloadLink": "https://download.microsoft.com/download/f/3/4/f34b2eb9-cc8d-4978-9ffb-17321ad9b7ed/MicrosoftAzureApplianceConfigurationManager.msi", "Version": "6.0.211.2", "Md5Hash": "e00a742acc35e78a64a6a81e75469b84"

4.    Descargue la versión más reciente de un servicio obsoleto mediante el vínculo de descarga del archivo.
5. Después de la descarga, ejecute el siguiente comando en una ventana de comandos de administrador para comprobar la integridad del MSI descargado.

    Por ejemplo: ``` C:\>Get-FileHash -Path <file_location> -Algorithm [Hashing Algorithm] ```  C:\>CertUtil -HashFile C:\Users\public\downloads\MicrosoftAzureApplianceConfigurationManager.MSI MD5

5. Compruebe que la salida del comando coincide con la entrada de valor hash del servicio en el archivo (por ejemplo, el valor hash MD5 anterior).
6. Ahora, ejecute el archivo MSI para instalar el servicio. Se trata de una instalación silenciosa y la ventana de instalación se cierra una vez finalizada.
7. Una vez finalizada la instalación, compruebe la versión del servicio en **Panel de control** > **Programas y características**. Ahora, la versión del servicio debe actualizarse a la última que se muestra en el archivo json.



## <a name="next-steps"></a>Pasos siguientes

- [Obtenga información](how-to-set-up-appliance-vmware.md) sobre cómo configurar el dispositivo para VMware.
- [Obtenga información](how-to-set-up-appliance-hyper-v.md) sobre cómo configurar el dispositivo para Hyper-V.
- [Obtenga información](how-to-set-up-appliance-physical.md) sobre cómo configurar el dispositivo para servidores físicos.

