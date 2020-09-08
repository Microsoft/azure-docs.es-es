---
title: Configuración de un dispositivo de Azure Migrate en Azure Government
description: Aprenda a configurar un dispositivo de Azure Migrate en Azure Government
ms.topic: article
ms.date: 04/16/2020
ms.openlocfilehash: ffea966c58909ecaab0da13a4204295ecb193895
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936799"
---
# <a name="set-up-an-appliance-in-azure-government"></a>Configuración de una aplicación en Azure Government 

Siga este artículo para implementar un [dispositivo de Azure Migrate](./migrate-appliance-architecture.md) para máquinas virtuales de VMware, máquinas virtuales de Hyper-V y servidores físicos, en una nube de Azure Government. Ejecute un script para crear el dispositivo y compruebe que se pueda conectar a Azure. Si quiere configurar una aplicación en la nube pública, siga [este artículo](deploy-appliance-script.md).


> [!NOTE]
> La opción para implementar un dispositivo mediante una plantilla (para máquinas virtuales de VMware y máquinas virtuales de Hyper-V) no se admite en Azure Government.


## <a name="prerequisites"></a>Requisitos previos

El script configura el dispositivo de Azure Migrate en un equipo físico o máquina virtual existente.

- El equipo que hará de dispositivo debe ejecutar Windows Server 2016, con 32 GB de memoria, ocho CPU virtuales, cerca de 80 GB de almacenamiento en disco y un conmutador virtual externo. Requiere una dirección IP estática o dinámica y acceso a Internet.
- Antes de implementar el dispositivo, consulte los requisitos detallados del dispositivo para las [máquinas virtuales de VMware](migrate-appliance.md#appliance---vmware), las [máquinas virtuales de Hyper-V](migrate-appliance.md#appliance---hyper-v)y los [servidores físicos](migrate-appliance.md#appliance---physical).
- No ejecute el script en un dispositivo de Azure Migrate existente.

## <a name="set-up-the-appliance-for-vmware"></a>Instalación de la aplicación para VMware

Para instalar la aplicación para VMware, descargue un archivo comprimido de Azure Portal y extraiga el contenido. Ejecute el script de PowerShell para iniciar la aplicación web de la aplicación. Instale la aplicación y configúrela por primera vez. Después, registre la aplicación en el proyecto de Azure Migrate.

### <a name="download-the-script"></a>Descarga del script

1.  En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Assessment**, haga clic en **Detectar**.
2.  En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , haga clic en **Sí, con VMware vSphere Hypervisor**.
3.  Haga clic en **Descargar** para descargar el archivo comprimido. 


### <a name="verify-file-security"></a>Comprobación de la seguridad del archivo

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En la máquina en la que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-VMWare-USGov.zip SHA256```

3. Compruebe la versión más reciente del dispositivo y el valor hash:

    **Algoritmo** | **Descargar** | **SHA256**
    --- | --- | ---
    VMware (85 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140337) | 31b1bfdd4fc29b3eb923c7c6e7a898af79b7cac0404426bea18809def2284188


### <a name="run-the-script"></a>Ejecute el script.

Funcionamiento del script:

- Instala agentes y una aplicación web.
- Instala roles de Windows, como el servicio de activación de Windows, IIS y PowerShell ISE.
- Descarga e instala un módulo de reescritura de IIS. [Más información](https://www.microsoft.com/download/details.aspx?id=7435).
- Actualiza una clave del registro (HKLM) con una configuración persistente para Azure Migrate.
- Crea archivos de registro y de configuración del siguiente modo:
    - **Archivos de configuración**: %ProgramData%\Microsoft Azure\Config
    - **Archivos de registro**: %ProgramData%\Microsoft Azure\Logs

Para ejecutar el script:

1. Extraiga el archivo comprimido en una carpeta del equipo que vaya a hospedar el dispositivo. No ejecute el script en una máquina en una aplicación de Azure Migrate existente.
2. Inicie PowerShell en el equipo con privilegios de administrador (elevados).
3. Cambie el directorio de PowerShell a la carpeta que contiene el contenido extraído del archivo comprimido descargado.
4. Ejecute el script **AzureMigrateInstaller.ps1** como se indica a continuación: 
    
    ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-VMWare-USGov>.\AzureMigrateInstaller.ps1 ```
1. Una vez que el script se haya ejecutado correctamente, inicia la aplicación web del dispositivo para que pueda configurarlo. Si detecta algún problema, consulte los registros del script en C:\Archivos de programa\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Comprobación del acceso

Asegúrese de que el dispositivo pueda conectarse a las direcciones URL de Azure para las [nubes gubernamentales](migrate-appliance.md#government-cloud-urls).


## <a name="set-up-the-appliance-for-hyper-v"></a>Configuración del dispositivo para Hyper-V

Para configurar la aplicación para Hyper-V, descargue un archivo comprimido de Azure Portal y extraiga el contenido. Ejecute el script de PowerShell para iniciar la aplicación web de la aplicación. Instale la aplicación y configúrela por primera vez. Después, registre la aplicación en el proyecto de Azure Migrate.

### <a name="download-the-script"></a>Descarga del script

1.  En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Assessment**, haga clic en **Detectar**.
2.  En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , seleccione **Sí, con Hyper-V**.
3.  Haga clic en **Descargar** para descargar el archivo comprimido. 


### <a name="verify-file-security"></a>Comprobación de la seguridad del archivo

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En la máquina en la que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-HyperV-USGov.zip SHA256```

3. Compruebe la versión más reciente del dispositivo y el valor hash:

    **Escenario** | **Descargar** | **SHA256**
    --- | --- | ---
    Hyper-V (85 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140424) |  db5311de3d1d4a1167183a94e8347456db9c5749c7332ff2eb4b777798765e48

          

### <a name="run-the-script"></a>Ejecute el script.

Funcionamiento del script:

- Instala agentes y una aplicación web.
- Instala roles de Windows, como el servicio de activación de Windows, IIS y PowerShell ISE.
- Descarga e instala un módulo de reescritura de IIS. [Más información](https://www.microsoft.com/download/details.aspx?id=7435).
- Actualiza una clave del registro (HKLM) con una configuración persistente para Azure Migrate.
- Crea archivos de registro y de configuración del siguiente modo:
    - **Archivos de configuración**: %ProgramData%\Microsoft Azure\Config
    - **Archivos de registro**: %ProgramData%\Microsoft Azure\Logs

Para ejecutar el script:

1. Extraiga el archivo comprimido en una carpeta del equipo que vaya a hospedar el dispositivo. No ejecute el script en una máquina en una aplicación de Azure Migrate existente.
2. Inicie PowerShell en el equipo con privilegios de administrador (elevados).
3. Cambie el directorio de PowerShell a la carpeta que contiene el contenido extraído del archivo comprimido descargado.
4. Ejecute el script **AzureMigrateInstaller.ps1** como se indica a continuación: 

    ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-HyperV-USGov>.\AzureMigrateInstaller.ps1 ``` 
1. Una vez que el script se haya ejecutado correctamente, inicia la aplicación web del dispositivo para que pueda configurarlo. Si detecta algún problema, consulte los registros del script en C:\Archivos de programa\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Comprobación del acceso

Asegúrese de que el dispositivo pueda conectarse a las direcciones URL de Azure para las [nubes gubernamentales](migrate-appliance.md#government-cloud-urls).


## <a name="set-up-the-appliance-for-physical-servers"></a>Configuración del dispositivo para servidores físicos

Para configurar el dispositivo para VMware, descargue un archivo comprimido de Azure Portal y extraiga el contenido. Ejecute el script de PowerShell para iniciar la aplicación web de la aplicación. Instale la aplicación y configúrela por primera vez. Después, registre la aplicación en el proyecto de Azure Migrate.

### <a name="download-the-script"></a>Descarga del script

1.  En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Assessment**, haga clic en **Detectar**.
2.  En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , seleccione **No virtualizado/Otro**.
3.  Haga clic en **Descargar** para descargar el archivo comprimido. 


### <a name="verify-file-security"></a>Comprobación de la seguridad del archivo

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En la máquina en la que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-USGov.zip SHA256```

3. Compruebe la versión más reciente del dispositivo y el valor hash:

    **Escenario** | **Descargar*** | **Valor del código hash**
    --- | --- | ---
    Físico (85 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140338) | 1545f9ce8874cedef6347c1a1332f8b5eabd6811a017440a2382525fb0430309
          

### <a name="run-the-script"></a>Ejecute el script.

Funcionamiento del script:

- Instala agentes y una aplicación web.
- Instala roles de Windows, como el servicio de activación de Windows, IIS y PowerShell ISE.
- Descarga e instala un módulo de reescritura de IIS. [Más información](https://www.microsoft.com/download/details.aspx?id=7435).
- Actualiza una clave del registro (HKLM) con una configuración persistente para Azure Migrate.
- Crea archivos de registro y de configuración del siguiente modo:
    - **Archivos de configuración**: %ProgramData%\Microsoft Azure\Config
    - **Archivos de registro**: %ProgramData%\Microsoft Azure\Logs

Para ejecutar el script:

1. Extraiga el archivo comprimido en una carpeta del equipo que vaya a hospedar el dispositivo. No ejecute el script en una máquina en una aplicación de Azure Migrate existente.
2. Inicie PowerShell en el equipo con privilegios de administrador (elevados).
3. Cambie el directorio de PowerShell a la carpeta que contiene el contenido extraído del archivo comprimido descargado.
4. Ejecute el script **AzureMigrateInstaller.ps1** como se indica a continuación: 

    ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-Server-USGov>.\AzureMigrateInstaller.ps1 ```
1. Una vez que el script se haya ejecutado correctamente, inicia la aplicación web del dispositivo para que pueda configurarlo. Si detecta algún problema, consulte los registros del script en C:\Archivos de programa\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Comprobación del acceso

Asegúrese de que el dispositivo pueda conectarse a las direcciones URL de Azure para las [nubes gubernamentales](migrate-appliance.md#government-cloud-urls).

## <a name="next-steps"></a>Pasos siguientes

Después de implementar la aplicación, es preciso que la configure y la registre en el proyecto de Azure Migrate.

- Configure el dispositivo para [VMware](how-to-set-up-appliance-vmware.md#configure-the-appliance).
- Configure el dispositivo para [Hyper-V](how-to-set-up-appliance-hyper-v.md#configure-the-appliance).
- Configure el dispositivo para [servidores físicos](how-to-set-up-appliance-physical.md).
