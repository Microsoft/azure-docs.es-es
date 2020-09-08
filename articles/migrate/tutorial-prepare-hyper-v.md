---
title: Preparación de máquinas virtuales de Hyper-V para evaluación y migración con Azure Migrate
description: Aprenda a prepararse para la evaluación y migración de máquinas virtuales de Hyper-V con Azure Migrate.
ms.topic: tutorial
ms.date: 04/15/2020
ms.custom: mvc
ms.openlocfilehash: 8ecb886b5d5cd9d6811788043b924880b4c366c4
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88928928"
---
# <a name="prepare-for-assessment-and-migration-of-hyper-v-vms-to-azure"></a>Preparación de la evaluación y migración de máquinas virtuales de Hyper-V a Azure

En este artículo se describe cómo preparar la evaluación y la migración de máquinas virtuales Hyper-V locales a Azure con [Azure Migrate:Server Assessment](migrate-services-overview.md#azure-migrate-server-assessment-tool) y [Azure Migrate:Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool).


Este tutorial es el primero de una serie que muestra cómo evaluar máquinas virtuales de Hyper-V y migrarlas a Azure. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Preparar Azure para trabajar con Azure Migrate.
> * Preparar la evaluación de las máquinas virtuales Hyper-V.
> * Preparar la migración de las máquinas virtuales Hyper-V. 

> [!NOTE]
> En los tutoriales se muestra la ruta de implementación más sencilla para un escenario, de modo que pueda configurar rápidamente una prueba de concepto. En ellos se usan las opciones predeterminadas siempre que es posible y no muestran todos los valores y rutas de acceso posibles.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.


## <a name="prepare-azure"></a>Preparación de Azure

En la tabla se resumen las tareas que debe completar en Azure. Encontrará instrucciones a continuación de la tabla.

**Task** | **Detalles** | **Permisos**
--- | --- | ---
**Crear un proyecto de Azure Migrate** | Un proyecto de Azure Migrate proporciona una ubicación central para orquestar y administrar las evaluaciones y migraciones con herramientas de Azure Migrate, herramientas de Microsoft y otras ofertas de terceros. | La cuenta de Azure necesita permisos de colaborador o propietario en el grupo de recursos en el que reside el proyecto.
**Registrar dispositivo** | Azure Migrate usa un dispositivo Azure Migrate ligero para detectar y evaluar máquinas virtuales Hyper-V. [Más información](migrate-appliance-architecture.md#appliance-registration). | Para registrar el dispositivo, la cuenta de Azure necesita permisos de colaborador o propietario en la suscripción de Azure.
**Creación de una aplicación de Azure AD** | Al registrar el dispositivo, Azure Migrate crea una aplicación de Azure Active Directory (Azure AD) que se usa para la comunicación entre los agentes que se ejecutan en el dispositivo y Azure Migrate. | La cuenta de Azure necesita permisos para crear aplicaciones de Azure AD.
**Crear un almacén de claves** | La instancia de Key Vault se crea como parte del registro del dispositivo y se usa para la administración del certificado que se descargó en el dispositivo durante su configuración.| Para permitir que Azure Migrate cree la instancia de Key Vault, la cuenta de Azure necesita permisos de colaborador en el grupo de recursos en el que reside el proyecto de Azure Migrate.
**Crear una máquina virtual** | Necesita permisos para crear una máquina virtual en el grupo de recursos y la red virtual y para escribir en un disco administrado de Azure. | La cuenta de Azure necesita el rol Colaborador de la máquina virtual.


### <a name="assign-permissions-to-create-project"></a>Asignación de permisos para crear un proyecto

Compruebe que tenga permiso para crear un proyecto de Azure Migrate.

1. En Azure Portal, abra la suscripción y seleccione **Control de acceso (IAM)** .
2. En **Comprobar acceso**, busque la cuenta correspondiente y haga clic en ella para ver los permisos.
3. Debe tener permisos de **Colaborador** o **Propietario**.
    - Si acaba de crear una cuenta de Azure gratuita, es el propietario de la suscripción.
    - Si no es el propietario, trabaje con él para asignar el rol.


### <a name="assign-permissions-to-create-azure-ad-apps"></a>Asignación de permisos para crear aplicaciones de Azure AD

Puede asignar permisos para Azure Migrate con el fin de crear las aplicaciones de Azure AD durante el registro del dispositivo mediante uno de los métodos siguientes:

- Un administrador de inquilinos o administrador global puede conceder permisos a los usuarios del inquilino para crear y registrar aplicaciones de Azure AD.
- Un administrador de inquilinos o administrador global puede asignar el rol de desarrollador de aplicaciones (que tiene los permisos) a la cuenta.

> [!NOTE]
> - Las aplicaciones no tienen otros permisos de acceso en la suscripción distintos de los descritos anteriormente.
> - Solo necesita estos permisos al registrar un nuevo dispositivo. Puede quitar los permisos una vez configurado el dispositivo.


#### <a name="grant-account-permissions"></a>Concesión de permisos de cuenta

El administrador de inquilinos o global puede conceder permisos como se indica:

1. En Azure AD, el administrador de inquilinos o global debe ir a **Azure Active Directory** > **Usuarios** > **Configuración de usuario**.
2. El administrador debe establecer **Registros de aplicaciones** en **Sí**.

    ![Permisos de Azure AD](./media/tutorial-prepare-hyper-v/aad.png)

> [!NOTE]
> Se trata de una configuración predeterminada que no es confidencial. [Más información](../active-directory/develop/active-directory-how-applications-are-added.md#who-has-permission-to-add-applications-to-my-azure-ad-instance).



#### <a name="assign-application-developer-role"></a>Asignación del rol de desarrollador de aplicaciones

El administrador de inquilinos o administrador global puede asignar el rol de desarrollador de aplicaciones a una cuenta. [Más información](../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md).


### <a name="assign-azure-account-permissions"></a>Asignación de los permisos de la cuenta de Azure

Asigne a la cuenta el rol colaborador de máquina virtual, con el fin de que tenga permisos para:

- Crear una máquina virtual en el grupo de recursos seleccionado.
- Crear una máquina virtual en la red virtual seleccionada.
- Escribir en un disco administrado de Azure. 


### <a name="set-up-an-azure-network"></a>Configurar una red de Azure

[Configure una red de Azure](../virtual-network/manage-virtual-network.md#create-a-virtual-network). Las máquinas locales se replican en los discos administrados de Azure. Al conmutar por error a Azure para la migración, las máquinas virtuales de Azure se crean a partir de estos discos administrados y se unen a una red de Azure que configuró.


## <a name="prepare-for-assessment"></a>Preparación para la evaluación

Puede preparar Hyper-V para la evaluación de máquinas virtuales manualmente o mediante un script de configuración. Estos son los pasos de preparación. Si realiza la preparación con un script, estos se configuran automáticamente.

**Step** | **Script** | **Manual**
--- | --- | ---
**Comprobación de los requisitos del host de Hyper-V** | El script comprueba que el host ejecuta una versión compatible de Hyper-V y el rol de Hyper-V.<br/><br/> Habilita el servicio WinRM y abre los puertos 5985 (HTTP) y 5986 (HTTPS) en el host (necesario para la recopilación de metadatos). | Compruebe los [requisitos del host de Hyper-V](migrate-support-matrix-hyper-v.md#hyper-v-host-requirements) para la evaluación de los servidores.<br/><br/> Asegúrese de que los [puertos necesarios](migrate-support-matrix-hyper-v.md#port-access) están abiertos en los hosts de Hyper-V.
**Comprobación de la versión de PowerShell** | Comprueba que se está ejecutando el script en una versión compatible de PowerShell. | Compruebe que ejecuta la versión 4.0 o posterior de PowerShell en el host de Hyper-V.
**Crear una cuenta** | Comprueba que usted (el usuario que ejecuta el script) tiene privilegios administrativos en el host de Hyper-V.<br/><br/>  Permite crear una cuenta de usuario local (no administrador) que se usa para que el servicio Azure Migrate se comunique con el host de Hyper-V. Esta cuenta de usuario se agrega a estos grupos en el host:<br/><br/> - Usuarios de Administración remota<br/><br/> - Administradores de Hyper-V<br/><br/>- Usuarios de Monitor de rendimiento | Configure una cuenta de usuario local o de dominio con permisos de administrador en el clúster o los hosts de Hyper-V.<br/><br/> - Necesita una sola cuenta para todos los hosts y clústeres que desee incluir en la detección.<br/><br/> - La cuenta puede ser una cuenta local o de dominio. Se recomienda que tenga permisos de administrador en los hosts o clústeres de Hyper-V.<br/><br/> O bien, si no desea asignar permisos de administrador, se necesitan los siguientes permisos: Usuarios de administración remota, Administradores de Hyper-V y Usuarios de Monitor de rendimiento.
**Habilitación de la comunicación remota con PowerShell** | Habilita la comunicación remota de PowerShell en el host de modo que el dispositivo Azure Migrate pueda ejecutar comandos de PowerShell en el host mediante una conexión de WinRM.| Para configurarlo, en cada uno de los hosts, abra una consola de PowerShell como administrador y ejecute este comando:<br/><br/>``` Enable-PSRemoting -force ```
**Configuración de los servicios de integración de Hyper-V** | Comprueba que los servicios de integración de Hyper-V está habilitado en todas las máquinas virtuales administradas por el host. |  [ Habilite los servicios de integración de Hyper-V](/windows-server/virtualization/hyper-v/manage/manage-hyper-v-integration-services) en cada máquina virtual.<br/><br/> Si ejecuta Windows Server 2003, [siga estas instrucciones](prepare-windows-server-2003-migration.md).
**Delegación de credenciales si los discos de máquina virtual se encuentran en recursos compartidos SMB remotos** | El script delega las credenciales. | [Habilite CredSSP](#enable-credssp-to-delegate-credentials) para delegar las credenciales.

### <a name="run-the-script"></a>Ejecute el script.

Ejecute el script como se indica a continuación:

1. Asegúrese de que tiene instalada la versión de PowerShell 4.0 o posterior en el host de Hyper-V.
2. Descargue el script del [Centro de descarga de Microsoft](https://aka.ms/migrate/script/hyperv). Microsoft firma criptográficamente el script.
3. Valide la integridad del script mediante archivos hash MD5 o SHA256. A continuación se muestran los valores de hashtag. Ejecute este comando para generar el código hash para el script:
    ```
    C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]
    ```
    Ejemplo de uso:
    ```
    C:\>CertUtil -HashFile C:\Users\Administrators\Desktop\ MicrosoftAzureMigrate-Hyper-V.ps1 SHA256
    ```

4. Después de validar la integridad del script, ejecútelo en cada host de Hyper-V con este comando de PowerShell:
    ```
    PS C:\Users\Administrators\Desktop> MicrosoftAzureMigrate-Hyper-V.ps1
    ```

#### <a name="hashtag-values"></a>Valores de hashtag

Los valores de hash son:

| **Hash** | **Valor** |
| --- | --- |
| **MD5** | 0ef418f31915d01f896ac42a80dc414e |
| **SHA256** | 0ad60e7299925eff4d1ae9f1c7db485dc9316ef45b0964148a3c07c80761ade2 |



### <a name="enable-credssp-to-delegate-credentials"></a>Habilitación de CredSSP para delegar las credenciales

Si el host tiene discos de las máquinas virtuales en recursos compartidos SMB, complete este paso en cada host.

- Puede ejecutar este comando de forma remota en todos los hosts de Hyper-V.
- Si agrega nuevos nodos de host en un clúster, se agregan automáticamente para la detección, pero debe habilitar CredSSP manualmente en los nuevos nodos si es necesario.

Habilite de la siguiente manera:

1. Identifique los hosts de Hyper-V que ejecutan máquinas virtuales de Hyper-V con discos en recursos compartidos SMB.
2. Ejecute el siguiente comando en cada host de Hyper-V identificado:

    ```
    Enable-WSManCredSSP -Role Server -Force
    ```

Al configurar el dispositivo, termine de configurar CredSSP mediante su [habilitación en el dispositivo](tutorial-assess-hyper-v.md#delegate-credentials-for-smb-vhds). Esto se describe en el siguiente tutorial de la serie.


## <a name="prepare-for-appliance-deployment"></a>Preparación para la implementación del dispositivo

Antes de configurar el dispositivo de Azure Migrate y comenzar la evaluación en el siguiente tutorial, prepare la implementación del dispositivo.

1. [Compruebe](migrate-appliance.md#appliance---hyper-v) los requisitos del dispositivo.
2. Revise las direcciones URL de Azure a las que el dispositivo necesitará acceder en las nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls). Si usa un proxy de firewall basado en direcciones URL, asegúrese de que permiten el acceso a las URL necesarias.
3. Revise los datos que el dispositivo va a recopilar durante la detección y la evaluación.
4. [Revise](migrate-appliance.md#collected-data---hyper-v) los requisitos de acceso a los puertos del dispositivo.


## <a name="prepare-for-hyper-v-migration"></a>Preparación de la migración de Hyper-V

1. [Revise](migrate-support-matrix-hyper-v-migration.md#hyper-v-host-requirements) los requisitos de los hosts de Hyper-V para la migración y las direcciones URL de Azure a las que los hosts y clústeres de Hyper-V necesitan acceder para la migración de las máquinas virtuales.
2. [Consulte](migrate-support-matrix-hyper-v-migration.md#hyper-v-vms) los requisitos de las máquinas virtuales de Hyper-V que desea migrar a Azure.
3. Hay algunos cambios necesarios en las máquinas virtuales antes de migrarlas a Azure.
    - Es importante realizar estos cambios antes de comenzar la migración. Si migra la máquina virtual antes de realizar el cambio, es posible que la máquina virtual no arranque en Azure.
    - Revise los cambios en [Windows](prepare-for-migration.md#windows-machines) y [Linux](prepare-for-migration.md#linux-machines) que debe realizar.



## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Configurar los permisos de la cuenta de Azure.
> * Preparar las máquinas virtuales y los hosts de Hyper-V para la evaluación y migración.
> * Se preparó para la implementación del dispositivo de Azure Migrate.

Continúe con el siguiente tutorial para crear un proyecto de Azure Migrate, implementar el dispositivo y detectar y evaluar las máquinas virtuales de Hyper-V para la migración a Azure.

> [!div class="nextstepaction"]
> [Evaluación de máquinas virtuales de Hyper-V](./tutorial-assess-hyper-v.md)
