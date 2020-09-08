---
title: Configurar un dispositivo de Azure Migrate para VMware
description: Obtenga información sobre cómo configurar un dispositivo de Azure Migrate para evaluar y migrar VM de VMware.
ms.topic: article
ms.date: 04/16/2020
ms.openlocfilehash: c9a9f1567f984fc5770b47d3998610cb69643360
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88923637"
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
- Proporcione un nombre de dispositivo y genere una clave del proyecto de Azure Migrate en el portal.
- Descargue una plantilla OVA e impórtela en vCenter Server.
- Crear el dispositivo y comprobar que se puede conectar a Azure Migrate Server Assessment.
- Configure el dispositivo por primera vez y regístrelo en el proyecto de Azure Migrate mediante la clave del proyecto de Azure Migrate.

### <a name="generate-the-azure-migrate-project-key"></a>Generación de la clave del proyecto de Azure Migrate

1. En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Assessment**, seleccione **Detectar**.
2. En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , haga clic en **Sí, con VMware vSphere Hypervisor**.
3. En **1: Generar la clave del proyecto de Azure Migrate**, proporcione un nombre para el dispositivo de Azure Migrate que configurará para la detección de máquinas virtuales de VMware. El nombre debe ser alfanumérico con 14 caracteres como máximo.
1. Haga clic en **Generar clave** para iniciar la creación de los recursos de Azure necesarios. No cierre la página Detectar máquinas durante la creación de recursos.
1. Después de la creación correcta de los recursos de Azure, se genera una **clave del proyecto Azure Migrate**.
1. Copie la clave, ya que la necesitará para completar el registro del dispositivo durante su configuración.

### <a name="download-the-ova-template"></a>Descarga de la plantilla OVA
En **2: Descargar dispositivo de Azure Migrate**, seleccione el archivo .OVA y haga clic en **Descargar**. 


   ![Selecciones para Detectar máquinas](./media/tutorial-assess-vmware/servers-discover.png)


   ![Selecciones para Generar clave](./media/tutorial-assess-vmware/generate-key-vmware.png)

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


### <a name="configure-the-appliance"></a>Configuración del dispositivo

Configure el dispositivo por primera vez.

> [!NOTE]
> Si configura la aplicación mediante un [script de PowerShell](deploy-appliance-script.md) en lugar de la OVA descargada, los dos primeros pasos de este procedimiento no son pertinentes.

1. En la consola de cliente de vSphere, haga clic con el botón derecho en la máquina virtual y, luego, seleccione **Open Console** (Abrir consola).
2. Especifique el idioma, la zona horaria y la contraseña del dispositivo.
3. Abra un explorador en cualquier equipo que pueda conectarse a la máquina virtual y abra la dirección URL de la aplicación web del dispositivo: **https://*nombre o dirección IP del dispositivo*: 44368**.

   Como alternativa, puede abrir la aplicación desde el escritorio del dispositivo, para lo que debe seleccionar el acceso directo de la aplicación.
1. Acepte los **términos de licencia** y lea la información de terceros.
1. En la aplicación web > **Set up prerequisites** (Configurar los requisitos previos ), realice las siguientes operaciones:
   - **Connectivity** (Conectividad): la aplicación comprueba que la máquina virtual tiene acceso a Internet. Si la máquina virtual usa un proxy:
     - Haga clic en **Configurar el proxy** y especifique la dirección del proxy (con los formatos http://ProxyIPAddress o http://ProxyFQDN) ) y el puerto de escucha.
     - Especifique las credenciales si el proxy requiere autenticación.
     - Solo se admite un proxy HTTP.
     - Si ha agregado detalles del proxy o ha deshabilitado el proxy o la autenticación, haga clic en **Guardar** para desencadenar la comprobación de conectividad.
   - **Time sync** (Sincronización de hora): Para que la detección funcione correctamente, la hora del dispositivo debe estar sincronizada con la hora de Internet.
   - **Instalación de actualizaciones**: el dispositivo garantiza que se instalan las actualizaciones más recientes. Una vez finalizada la comprobación, puede hacer clic en **Ver servicios del dispositivo** para ver el estado y las versiones de los componentes que se ejecutan en el dispositivo.
   - **Install VDDK** (Instalar VDDK): El dispositivo comprueba si VMware vSphere Virtual Disk Development Kit (VDDK) está instalado. Si no está instalado, descargue VDDK 6.7 de VMware y extraiga el contenido del archivo ZIP descargado en la ubicación especificada del dispositivo, tal como se indica en las **instrucciones de instalación**.

     Server Migration de Azure Migrate usa VDDK para replicar las máquinas durante la migración a Azure. 
1. Si quiere, puede **volver a ejecutar los requisitos previos** en cualquier momento durante la configuración del dispositivo para comprobar si el dispositivo cumple todos los requisitos previos.

### <a name="register-the-appliance-with-azure-migrate"></a>Registro del dispositivo en Azure Migrate

1. Pegue la **clave del proyecto de Azure Migrate** copiada desde el portal. Si no tiene la clave, vaya a **Server Assessment > Detectar > Administrar los dispositivos existentes**, seleccione el nombre del dispositivo que proporcionó en el momento de la generación de la clave y copie la clave correspondiente.
1. Haga clic en **Iniciar sesión**. Se abrirá un mensaje de inicio de sesión de Azure en una nueva pestaña del explorador. Si no aparece, asegúrese de que ha deshabilitado el bloqueador de elementos emergentes en el explorador.
1. En la nueva pestaña, inicie sesión con su nombre de usuario y contraseña de Azure.
   
   No se admite el inicio de sesión con un PIN.
3. Después de iniciar sesión correctamente, vuelva a la aplicación web. 
4. Si la cuenta de usuario de Azure que se usa para el registro tiene los [permisos](tutorial-prepare-vmware.md#prepare-azure) adecuados en los recursos de Azure creados durante la generación de la clave, se iniciará el registro del dispositivo.
1. Una vez que el dispositivo se ha registrado correctamente, puede ver los detalles de registro haciendo clic en **Ver detalles**.


## <a name="start-continuous-discovery"></a>Inicio de detección continua

El dispositivo necesita conectarse a vCenter Server para detectar los datos de configuración y rendimiento de las máquinas virtuales.

1. En **Paso 1: Proporcionar las credenciales de vCenter Server**, haga clic en **Agregar credenciales** para especificar un nombre descriptivo para las credenciales, agregue un **nombre de usuario** y una **contraseña** para la cuenta de vCenter Server que utilizará el dispositivo para detectar máquinas virtuales en la instancia de vCenter Server.
    - Debe haber configurado una cuenta con los permisos necesarios en el [tutorial anterior](tutorial-prepare-vmware.md#set-up-permissions-for-assessment).
    - Si desea limitar el ámbito de la detección a objetos específicos de VMware (centros de datos de vCenter Server, clústeres, una carpeta de clústeres, hosts, una carpeta de hosts o máquinas virtuales individuales), consulte las instrucciones de [este artículo](set-discovery-scope.md) para restringir la cuenta usada por Azure Migrate.
1. En el **Paso 2: Proporcionar los detalles de vCenter Server**, haga clic en **Agregar origen de detección** para seleccionar el nombre descriptivo de las credenciales en la lista desplegable, especifique la **dirección IP o el FQDN** de la instancia de vCenter Server. Puede dejar el **puerto** en el valor predeterminado (443) o especificar un puerto personalizado en el que vCenter Server escuche y hacer clic en **Guardar**.
1. Al hacer clic en Guardar, el dispositivo intentará validar la conexión a vCenter Server con las credenciales proporcionadas y mostrar el **estado de validación** en la tabla con la dirección IP o el FQDN de vCenter Server.
1. Puede **volver a validar** la conectividad a vCenter Server en cualquier momento antes de iniciar la detección.
1. En **Paso 3: Proporcionar las credenciales de la máquina virtual para detectar las aplicaciones instaladas y para realizar la asignación de dependencias sin agente**, haga clic en **Agregar credenciales** y especifique el sistema operativo para el que se proporcionan las credenciales, el nombre descriptivo de las credenciales y el **nombre de usuario** y la **contraseña**. A continuación, haga clic en **Guardar**.

    - Si lo desea, agregue las credenciales aquí si ha creado una cuenta para utilizarla en la [característica de detección de aplicaciones](how-to-discover-applications.md) o [en la característica de análisis de dependencias sin agente](how-to-create-group-machine-dependencies-agentless.md).
    - Si no quiere usar estas características, puede hacer clic en el control deslizante para omitir el paso. Puede invertir la intención en cualquier momento más tarde.
    - Consulte cuáles son las credenciales necesarias para la [detección de aplicaciones](migrate-support-matrix-vmware.md#application-discovery-requirements) o para el [análisis de dependencias sin agente](migrate-support-matrix-vmware.md#dependency-analysis-requirements-agentless).

5. Haga clic en **Iniciar detección** para iniciar la detección de máquinas virtuales. Una vez que la detección se ha iniciado correctamente, puede comprobar el estado de detección de la dirección IP o el FQDN de vCenter en la tabla.

La detección funciona como se indica a continuación:
- Los metadatos de las máquinas virtuales detectadas tardan unos 15 minutos en aparecer en el portal.
- La detección de las aplicaciones, los roles y las características instaladas tarda un rato. Todo depende del número de máquinas virtuales que se detectan. En el caso de 500 máquinas virtuales, el inventario de la aplicación tarda aproximadamente una hora en aparecer en el portal de Azure Migrate.

## <a name="next-steps"></a>Pasos siguientes

Revise los tutoriales para la [valoración de VMware](tutorial-assess-vmware.md) y la [migración sin agente](tutorial-migrate-vmware.md).
