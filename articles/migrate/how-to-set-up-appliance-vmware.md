---
title: Configurar un dispositivo de Azure Migrate para VMware
description: Obtenga información sobre cómo configurar un dispositivo de Azure Migrate para evaluar y migrar VM de VMware.
ms.topic: article
ms.date: 04/16/2020
ms.openlocfilehash: 24ba978d776da375b417fb67823651727836cb22
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386748"
---
# <a name="set-up-an-appliance-for-vmware-vms"></a>Configuración de un dispositivo para máquinas virtuales de VMware

Siga este artículo para configurar el dispositivo de Azure Migrate para su evaluación con la herramienta [Azure Migrate:Server Assessment](migrate-services-overview.md#azure-migrate-server-assessment-tool) y para la migración sin agente mediante la herramienta [Azure Migrate:Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool).

El [dispositivo de Azure Migrate](migrate-appliance.md) es un dispositivo ligero que usa Azure Migrate:Server Assessment y Server Migration para detectar máquinas virtuales de VMware locales, enviar datos de rendimiento y metadatos de máquinas virtuales a Azure y para la replicación de máquinas virtuales de VMware durante la migración sin agente.

Puede implementar el dispositivo mediante un par de métodos:

- Configúrelo en una VM de VMware mediante una plantilla de OVA descargada. Este es el método que se describe en este artículo.
- Configúrelo en una VM de VMware o en una máquina física con un script del instalador de PowerShell. Debe usarse [este método](deploy-appliance-script.md) si no se puede configurar una VM mediante una plantilla de OVA, o si se encuentra en Azure Government.

Una vez creada la aplicación, compruebe que se puede conectar a Azure Migrate:Server Assessment, configúrela por primera vez y regístrela en el proyecto de Azure Migrate.


## <a name="appliance-deployment-ova"></a>Implementación del dispositivo (OVA)

Para configurar el dispositivo mediante una plantilla de OVA:
- Descargue una plantilla OVA e impórtela en vCenter Server.
- Crear el dispositivo y comprobar que se puede conectar a Azure Migrate Server Assessment.
- Configurar el dispositivo por primera vez y registrarlo en el proyecto de Azure Migrate.

## <a name="download-the-ova-template"></a>Descarga de la plantilla OVA

1. En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Assessment**, haga clic en **Detectar**.
2. En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , haga clic en **Sí, con VMware vSphere Hypervisor**.
3. Haga clic en **Descargar** para descargar el archivo de plantilla .OVA.

  ![Selecciones para descargar un archivo de OVA](./media/tutorial-assess-vmware/download-ova.png)

### <a name="verify-security"></a>Comprobación de la seguridad

Compruebe que el archivo OVA es seguro, antes de implementarlo.

1. En la máquina en la que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo OVA:
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo de uso: ```C:\>CertUtil -HashFile C:\AzureMigrate\AzureMigrate.ova SHA256```
3. El código hash generado debe coincidir con esta [configuración](./tutorial-assess-vmware.md#verify-security) para la versión más reciente del dispositivo.



## <a name="create-the-appliance-vm"></a>Creación de la máquina virtual del dispositivo

Importe el archivo descargado y cree una máquina virtual.

1. En la consola de cliente de vSphere, haga clic en **File** (Archivo) > **Deploy OVF Template** (Implementar plantilla de OVF).
![Comando de menú para implementar una plantilla de OVF](./media/tutorial-assess-vmware/deploy-ovf.png)

2. En el Deploy OVF Template Wizard (Asistente para implementar la plantilla de OVF) > **Source** (Origen), especifique la ubicación del archivo OVA.
3. En **Name** (Nombre) y **Location** (Ubicación), especifique un nombre descriptivo para la máquina virtual. Seleccione el objeto de inventario en el que se hospedará la máquina virtual.
5. En **Host/Cluster** (Host o clúster), especifique el host o clúster en que se ejecutará la máquina virtual.
6. En **Storage**, especifique el destino de almacenamiento de la máquina virtual.
7. En **Disk Format** (Formato de disco), especifique el tamaño y el tipo de disco.
8. En **Network Mapping** (Asignación de red), especifique la red a la que se conectará la máquina virtual. La red necesita conectividad a Internet para enviar metadatos a Azure Migrate Server Assessment.
9. Revise y confirme la configuración y haga clic en **Finish** (Finalizar).


## <a name="verify-appliance-access-to-azure"></a>Comprobación de que el dispositivo puede acceder a Azure

Asegúrese de que la VM del dispositivo pueda conectarse a las direcciones URL de Azure para las nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls).


## <a name="configure-the-appliance"></a>Configuración del dispositivo

Configure el dispositivo por primera vez. Si se implementa el dispositivo mediante un script en lugar de una plantilla de OVA, se tienen que omitir los dos primeros pasos del procedimiento.

1. En la consola del cliente de vSphere, haga clic con el botón derecho en VM > **Open Console** (Abrir consola).
2. Especifique el idioma, la zona horaria y la contraseña del dispositivo.
3. Abra un explorador en cualquier equipo que pueda conectarse a la máquina virtual y abra la dirección URL de la aplicación web del dispositivo: **https://*nombre o dirección IP del dispositivo*: 44368**.

   Como alternativa, puede abrir la aplicación desde el escritorio del dispositivo, para lo que debe hacer clic en el acceso directo de la aplicación.
4. En la aplicación web > **Set up prerequisites** (Configurar los requisitos previos ), realice las siguientes operaciones:
    - **License** (Licencia): Acepte los términos de licencia y lea la información de terceros.
    - **Connectivity** (Conectividad): la aplicación comprueba que la máquina virtual tiene acceso a Internet. Si la máquina virtual usa un proxy:
        - Haga clic en **Configuración de proxy** y especifique el puerto de escucha y la dirección del proxy con los formatos http://ProxyIPAddress o http://ProxyFQDN.
        - Especifique las credenciales si el proxy requiere autenticación.
        - Solo se admite un proxy HTTP.
    - **Time sync** (Sincronización de hora): Se comprueba la hora. Para que la detección funcione correctamente, la hora del dispositivo debe estar sincronizada con la hora de Internet.
    - **Instalación de actualizaciones**: Azure Migrate comprueba si están instaladas las últimas actualizaciones del dispositivo.
    - **Install VDDK** (Instalar VDDK): Azure Migrate comprueba si VMware vSphere Virtual Disk Development Kit (VDDK) está instalado.
        - Azure Migrate usa VDDK para replicar las máquinas durante la migración a Azure.
        - Descargue VDDK 6.7 de VMware y extraiga el contenido del archivo zip descargado en la ubicación especificada del dispositivo.

## <a name="register-the-appliance-with-azure-migrate"></a>Registro del dispositivo en Azure Migrate

1. Haga clic en **Iniciar sesión**. Si no aparece, asegúrese de que ha deshabilitado el bloqueador de elementos emergentes en el explorador.
2. En la pestaña nueva, inicie sesión con sus credenciales de Azure.
    - Inicie sesión con su nombre de usuario y contraseña.
    - No se admite el inicio de sesión con un PIN.
3. Después de iniciar sesión correctamente, vuelva a la aplicación web.
2. Seleccione la suscripción en la que se creó el proyecto de Azure Migrate. Seleccione el proyecto.
3. Escriba un nombre para el dispositivo. Este nombre debe ser alfanumérico y no puede tener más de 14 caracteres.
4. Haga clic en **Registrar**.


## <a name="start-continuous-discovery-by-providing-vcenter-server-and-vm-credential"></a>Inicio de la detección continua con la credencial de máquina virtual y vCenter Server proporcionada

El dispositivo necesita conectarse a vCenter Server para detectar los datos de configuración y rendimiento de las máquinas virtuales.

### <a name="specify-vcenter-server-details"></a>Especificar los detalles de vCenter Server
1. En **Specify vCenter Server details** (Especificar detalles de vCenter Server), especifique el nombre (nombre de dominio completo) o la dirección IP de vCenter Server. Puede dejar el puerto predeterminado o especificar un puerto personalizado en el que vCenter Server escuche.
2. En **User name** (Nombre de usuario) y **Password** (Contraseña), especifique las credenciales de la cuenta de solo lectura que el dispositivo utilizará para detectar las máquinas virtuales en vCenter Server. Puede establecer el ámbito de la detección si limita el acceso a la cuenta de vCenter. [Más información](set-discovery-scope.md).
3. Haga clic en **Validate connection** (Validar conexión) para asegurarse de que el dispositivo puede conectarse a vCenter Server.

### <a name="specify-vm-credentials"></a>Especificación de las credenciales de máquina virtual
Para la detección de aplicaciones, roles y características y la visualización de las dependencias de las máquinas virtuales, puede proporcionar una credencial de máquina virtual que tenga acceso a las máquinas virtuales de VMware. Puede agregar una credencial para máquinas virtuales Windows y otra para máquinas virtuales Linux. [Más información](./migrate-support-matrix-vmware.md) sobre los privilegios de acceso necesarios.

> [!NOTE]
> Esta entrada es opcional y es necesaria para habilitar la detección de aplicaciones y la visualización de dependencias sin agente.

1. En **Discover applications and dependencies on VMs** (Detectar aplicaciones y dependencias en máquinas virtuales) haga clic en **Agregar credenciales**.
2. En **Operating System** (Sistema operativo) seleccione el sistema operativo.
3. Proporcione un nombre descriptivo para la credencial.
4. En **Username** (Nombre de usuario) y **Password** (Contraseña), especifique una cuenta que tenga al menos acceso de invitado en las máquinas virtuales.
5. Haga clic en **Agregar**.

Cuando haya especificado las credenciales de las máquinas virtuales y de vCenter Server (este paso es opcional), haga clic en **Save and start discovery** (Guardar e iniciar la detección) para iniciar la detección del entorno local.

Los metadatos de las máquinas virtuales detectadas tardan unos 15 minutos en aparecer en el portal. La detección de las aplicaciones, las características y los roles instalados lleva tiempo, que depende del número de máquinas virtuales que se detectan. En el caso de 500 máquinas virtuales, el inventario de la aplicación tarda aproximadamente 1 hora en aparecer en el portal de Azure Migrate.

## <a name="next-steps"></a>Pasos siguientes

Revise los tutoriales para la [valoración de VMware](tutorial-assess-vmware.md) y la [migración sin agente](tutorial-migrate-vmware.md).
