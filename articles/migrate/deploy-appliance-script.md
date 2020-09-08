---
title: Configuración de un dispositivo de Azure Migrate con un script
description: Aprenda a configurar un dispositivo de Azure Migrate con un script
ms.topic: article
ms.date: 04/16/2020
ms.openlocfilehash: 43e47c6b4121a2f389dfec27873b90b7031c4f2c
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88929982"
---
# <a name="set-up-an-appliance-with-a-script"></a>Configuración de un dispositivo con un script

Siga este artículo para crear una [aplicación de Azure Migrate](./migrate-appliance-architecture.md) para evaluar o migrar máquinas virtuales de VMware y máquinas virtuales de Hyper-V. Ejecute un script para crear una aplicación y compruebe que se puede conectar a Azure. 

Puede implementar la aplicación para máquinas virtuales de Hyper-V y VMware mediante un script o mediante una plantilla que se descarga de Azure Portal. El uso de un script es útil si no se puede crear una máquina virtual mediante la plantilla descargada.

- Para usar una plantilla, siga los tutoriales de [VMware](tutorial-prepare-vmware.md) o [Hyper-V](tutorial-prepare-hyper-v.md).
- Para configurar una aplicación para servidores físicos, solo puede usar un script. Siga [este artículo](how-to-set-up-appliance-physical.md).
- Para configurar una aplicación en una nube de Azure Government, siga [este artículo](deploy-appliance-script-government.md).

## <a name="prerequisites"></a>Requisitos previos

El script configura el dispositivo de Azure Migrate en un equipo físico o máquina virtual existente.

- El equipo que actuará como dispositivo debe cumplir los siguientes requisitos de hardware y sistema operativo:

Escenario | Requisitos
--- | ---
VMware | Windows Server 2016, con 32 GB de memoria, ocho vCPU, alrededor de 80 GB de almacenamiento en disco
Hyper-V | Windows Server 2016, con 16 GB de memoria, ocho vCPU, alrededor de 80 GB de almacenamiento en disco
- El equipo también necesita un conmutador virtual externo. Requiere una dirección IP estática o dinámica y acceso a Internet.
- Antes de implementar el dispositivo, consulte los requisitos detallados del dispositivo para las [máquinas virtuales de VMware](migrate-appliance.md#appliance---vmware) y las [máquinas virtuales de Hyper-V](migrate-appliance.md#appliance---hyper-v).
- No ejecute el script en un dispositivo de Azure Migrate existente.

## <a name="set-up-the-appliance-for-vmware"></a>Instalación de la aplicación para VMware

Para configurar el dispositivo para VMware, descargue el archivo comprimido denominado AzureMigrateInstaller-Server-Public.zip desde el portal o desde [aquí](https://go.microsoft.com/fwlink/?linkid=2140334) y extraiga el contenido. Ejecute el script de PowerShell para iniciar la aplicación web de la aplicación. Instale la aplicación y configúrela por primera vez. Después, registre la aplicación en el proyecto de Azure Migrate.


### <a name="verify-file-security"></a>Comprobación de la seguridad del archivo

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En la máquina en la que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public.zip SHA256```
3. Compruebe la versión y script más recientes del dispositivo para la nube pública de Azure:

    **Algoritmo** | **Descargar** | **SHA256**
    --- | --- | ---
    VMware (85 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140334) | 5d0a3dbce4b5010980d59d49859f809acfeb17f5a36f57af4dac44a0a62dde1f



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

    ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public> .\AzureMigrateInstaller.ps1 -scenario VMware ```
   
5. Una vez que el script se haya ejecutado correctamente, inicia la aplicación web del dispositivo para que pueda configurarlo. Si detecta algún problema, consulte los registros del script en C:\Archivos de programa\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Comprobación del acceso

Asegúrese de que la aplicación puede conectarse a las direcciones URL de Azure para la nube [pública](migrate-appliance.md#public-cloud-urls).

## <a name="set-up-the-appliance-for-hyper-v"></a>Configure la aplicación para Hyper-V

Para configurar el dispositivo para Hyper-V, descargue el archivo comprimido denominado AzureMigrateInstaller-Server-Public.zip desde el portal o desde [aquí](https://go.microsoft.com/fwlink/?linkid=2105112) y extraiga el contenido. Ejecute el script de PowerShell para iniciar la aplicación web de la aplicación. Instale la aplicación y configúrela por primera vez. Después, registre la aplicación en el proyecto de Azure Migrate.


### <a name="verify-file-security"></a>Comprobación de la seguridad del archivo

Compruebe que el archivo comprimido es seguro, antes de implementarlo.

1. En la máquina en la que descargó el archivo, abra una ventana de comandos de administrador.
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public.zip SHA256```

3. Compruebe la versión y script más recientes del dispositivo para la nube pública de Azure:

    **Escenario** | **Descargar** | **SHA256**
    --- | --- | ---
    Hyper-V (85 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2140334) |  5d0a3dbce4b5010980d59d49859f809acfeb17f5a36f57af4dac44a0a62dde1f

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

    ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public> .\AzureMigrateInstaller.ps1 -scenario Hyperv ```
   
5. Una vez que el script se haya ejecutado correctamente, inicia la aplicación web del dispositivo para que pueda configurarlo. Si detecta algún problema, consulte los registros del script en C:\Archivos de programa\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Comprobación del acceso

Asegúrese de que la aplicación puede conectarse a las direcciones URL de Azure para la nube [pública](migrate-appliance.md#public-cloud-urls).

## <a name="next-steps"></a>Pasos siguientes

Después de implementar la aplicación, es preciso que la configure y la registre en el proyecto de Azure Migrate.

- Configure el dispositivo para [VMware](how-to-set-up-appliance-vmware.md#configure-the-appliance).
- Configure el dispositivo para [Hyper-V](how-to-set-up-appliance-hyper-v.md#configure-the-appliance).
