---
title: Uso de Azure Files con Linux | Microsoft Docs
description: Aprenda a montar un recurso compartido de archivos de Azure mediante SMB en Linux. Consulte la lista de requisitos previos. Revise las consideraciones de seguridad de SMB en los clientes de Linux.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 10/19/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: d00b0558f85e18dfb53736d89fead953cc01ee60
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88053174"
---
# <a name="use-azure-files-with-linux"></a>Uso de Azure Files con Linux
[Azure Files](storage-files-introduction.md) es el sencillo sistema de archivos en la nube de Microsoft. Los recursos compartidos de archivos de Azure se pueden montar en distribuciones de Linux mediante el [cliente kernel de SMB](https://wiki.samba.org/index.php/LinuxCIFS). En este artículo se muestran dos maneras de montar un recurso compartido de archivos de Azure: a petición, con el comando `mount` y al inicio, mediante la creación de una entrada en `/etc/fstab`.

El método recomendado para montar un recurso compartido de archivos de Azure en Linux es usar SMB 3.0. De forma predeterminada, Azure Files requiere cifrado en tránsito, que solo es compatible con SMB 3.0. Azure Files también admite SMB 2.1, que no admite el cifrado en tránsito, pero no puede montar recursos compartidos de archivos de Azure con SMB 2.1 desde otra región de Azure o de forma local por motivos de seguridad. A menos que la aplicación requiera específicamente SMB 2.1, hay pocas razones para usarlo, ya que las distribuciones de Linux más populares publicadas recientemente admiten SMB 3.0:  

| Distribución de Linux | SMB 2.1 <br>(Puede montar en máquinas virtuales dentro de la misma región de Azure) | SMB 3.0 <br>(Puede montar desde el nivel local a entre regiones) |
| --- | :---: | :---: |
| Ubuntu | 14.04+ | 16.04 (o posterior) |
| Red Hat Enterprise Linux (RHEL) | 7 (o posterior) | 7.5 (o posterior) |
| CentOS | 7 (o posterior) |  7.5 (o posterior) |
| Debian | 8 (o posterior) | 10+ |
| openSUSE | 13.2 (o posterior) | 42.3+ |
| SUSE Linux Enterprise Server | 12+ | 12 SP2+ |

Si usa una distribución de Linux que no aparece en la tabla anterior, puede comprobar si la distribución de Linux admite SMB 3.0 con cifrado comprobando la versión de kernel de Linux. SMB 3.0 con cifrado se ha agregado a la versión 4.11 de kernel de Linux. El comando `uname` devolverá la versión de kernel de Linux en uso:

```bash
uname -r
```

## <a name="prerequisites"></a>Prerrequisitos
<a id="smb-client-reqs"></a>

* <a id="install-cifs-utils"></a>**Asegúrese de que el paquete cifs-utils está instalado.**  
    El paquete cifs-utils se puede instalar con el administrador de paquetes en la distribución de Linux de su elección. 

    En distribuciones de **Ubuntu** y **Debian**, use el administrador de paquetes `apt`:

    ```bash
    sudo apt update
    sudo apt install cifs-utils
    ```

    En **Fedora**, **Red Hat Enterprise Linux 8 +**, y **CentOS 8 +**, use el administrador de paquetes `dnf`:

    ```bash
    sudo dnf install cifs-utils
    ```

    En las versiones anteriores de **Red Hat Enterprise Linux** y **CentOS**, use el administrador de paquetes `yum`:

    ```bash
    sudo yum install cifs-utils 
    ```

    En **openSUSE**, use el administrador de paquetes `zypper`:

    ```bash
    sudo zypper install cifs-utils
    ```

    En otras distribuciones, use el administrador de paquetes apropiado o [compile desde el origen](https://wiki.samba.org/index.php/LinuxCIFS_utils#Download)

* **La versión más reciente de la interfaz de la línea de comandos (CLI).** Para más información sobre cómo instalar la CLI de Azure, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) y seleccione el sistema operativo. Si prefiere usar el módulo de Azure PowerShell en PowerShell 6+, puede hacerlo. Sin embargo, las instrucciones siguientes son para la CLI de Azure.

* **Asegurarse de que el puerto 445 está abierto**: SMB se comunica a través del puerto TCP 445, así que compruebe que el firewall no bloquea el puerto TCP 445 en la máquina cliente.  Remplace **<your-resource-group>** y **<your-storage-account>**
    ```bash
    resourceGroupName="<your-resource-group>"
    storageAccountName="<your-storage-account>"
    
    # This command assumes you have logged in with az login
    httpEndpoint=$(az storage account show \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | tr -d '"')
    smbPath=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint))
    fileHost=$(echo $smbPath | tr -d "/")

    nc -zvw3 $fileHost 445
    ```

    Si la conexión se realizó correctamente, verá algo parecido a la siguiente salida:

    ```
    Connection to <your-storage-account> 445 port [tcp/microsoft-ds] succeeded!
    ```

    Si no puede abrir el puerto 445 en su red corporativa o si un ISP le impide hacerlo, puede utilizar una conexión VPN o ExpressRoute para solucionar el puerto 445. Para más información, consulte [Consideraciones de red para el acceso directo a los recursos compartidos de archivos de Azure](storage-files-networking-overview.md).

## <a name="mounting-azure-file-share"></a>Montando el recurso compartido de archivos de Azure
Para usar un recurso compartido de archivos de Azure con la distribución de Linux, debe crear un directorio para que sirva como punto de montaje para el recurso compartido de archivos de Azure. El punto de montaje se puede crear en cualquier parte del sistema de Linux, pero es habitual crearlo en /mnt. Después del punto de montaje, use el comando `mount` para tener acceso al recurso compartido de archivos de Azure.

Si lo desea, puede montar el mismo recurso compartido de archivos de Azure en varios puntos de montaje.

### <a name="mount-the-azure-file-share-on-demand-with-mount"></a>Montaje del recurso compartido de archivos de Azure a petición con `mount`
1. **Cree una carpeta para un punto de montaje**: Reemplace `<your-resource-group>`, `<your-storage-account>` y `<your-file-share>` por la información correcta para su entorno:

    ```bash
    resourceGroupName="<your-resource-group>"
    storageAccountName="<your-storage-account>"
    fileShareName="<your-file-share>"

    mntPath="/mnt/$storageAccountName/$fileShareName"

    sudo mkdir -p $mntPath
    ```

1. **Use el comando siguiente para montar el recurso compartido de archivos de Azure**. En el ejemplo siguiente, el valor predeterminado de los permisos de archivo y carpeta de Linux es 0755, que significa lectura, escritura y ejecución para el propietario (según el propietario de Linux de archivo o directorio), lectura y ejecución para los usuarios en el grupo de propietarios y lectura y ejecución para otros en el sistema. Puede usar las opciones de montaje `uid` y `gid` para establecer el identificador de usuario y el identificador de grupo para el montaje. También puede usar `dir_mode` y `file_mode` para establecer permisos personalizados según sea necesario. Para obtener más información sobre cómo establecer permisos, consulte [notación numérica de UNIX](https://en.wikipedia.org/wiki/File_system_permissions#Numeric_notation) en Wikipedia. 

    ```bash
    httpEndpoint=$(az storage account show \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | tr -d '"')
    smbPath=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint))$fileShareName

    storageAccountKey=$(az storage account keys list \
        --resource-group $resourceGroupName \
        --account-name $storageAccountName \
        --query "[0].value" | tr -d '"')

    sudo mount -t cifs $smbPath $mntPath -o vers=3.0,username=$storageAccountName,password=$storageAccountKey,serverino
    ```

    > [!Note]  
    > El comando de montaje anterior se monta con SMB 3.0. Si la distribución de Linux no admite SMB 3.0 con cifrado o si solo admite SMB 2.1, solo puede montar desde una máquina virtual de Azure dentro de la misma región que la cuenta de almacenamiento. Para montar el recurso compartido de archivos de Azure en una distribución de Linux que no admita SMB 3.0 con cifrado, necesitará [deshabilitar el cifrado en tránsito para la cuenta de almacenamiento](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

Cuando haya terminado de usar el recurso compartido de archivos de Azure, puede usar `sudo umount $mntPath` para desmontarlo.

### <a name="create-a-persistent-mount-point-for-the-azure-file-share-with-etcfstab"></a>Creación de un punto de montaje persistente para el recurso compartido de archivos de Azure con `/etc/fstab`
1. **Cree una carpeta para un punto de montaje**: La carpeta para el punto de montaje se puede crear en cualquier parte del sistema de archivos, pero es habitual crearla en la carpeta /mnt. Por ejemplo, con el siguiente comando se crea un directorio; reemplace `<your-resource-group>`, `<your-storage-account>` y `<your-file-share>` por la información de su entorno correspondiente:

    ```bash
    resourceGroupName="<your-resource-group>"
    storageAccountName="<your-storage-account>"
    fileShareName="<your-file-share>"

    mntPath="/mnt/$storageAccountName/$fileShareName"

    sudo mkdir -p $mntPath
    ```

1. **Cree un archivo de credenciales para almacenar el nombre de usuario (el nombre de cuenta de almacenamiento) y la contraseña (la clave de cuenta de almacenamiento) para el recurso compartido de archivos.** 

    ```bash
    if [ ! -d "/etc/smbcredentials" ]; then
        sudo mkdir "/etc/smbcredentials"
    fi

    storageAccountKey=$(az storage account keys list \
        --resource-group $resourceGroupName \
        --account-name $storageAccountName \
        --query "[0].value" | tr -d '"')
    
    smbCredentialFile="/etc/smbcredentials/$storageAccountName.cred"
    if [ ! -f $smbCredentialFile ]; then
        echo "username=$storageAccountName" | sudo tee $smbCredentialFile > /dev/null
        echo "password=$storageAccountKey" | sudo tee -a $smbCredentialFile > /dev/null
    else 
        echo "The credential file $smbCredentialFile already exists, and was not modified."
    fi
    ```

1. **Cambie los permisos en el archivo de credenciales para que solo la raíz pueda leer o modificar el archivo de contraseña.** Puesto que la clave de la cuenta de almacenamiento es básicamente una contraseña de superadministrador para la cuenta de almacenamiento, el establecimiento de los permisos en el archivo de forma que solo la raíz pueda acceder a ella es importante para que los usuarios con privilegios reducidos no puedan recuperarla.   

    ```bash
    sudo chmod 600 $smbCredentialFile
    ```

1. **Use el siguiente comando para anexar la siguiente línea a `/etc/fstab`** : En el ejemplo siguiente, el valor predeterminado de los permisos de archivo y carpeta de Linux es 0755, que significa lectura, escritura y ejecución para el propietario (según el propietario de Linux de archivo o directorio), lectura y ejecución para los usuarios en el grupo de propietarios y lectura y ejecución para otros en el sistema. Puede usar las opciones de montaje `uid` y `gid` para establecer el identificador de usuario y el identificador de grupo para el montaje. También puede usar `dir_mode` y `file_mode` para establecer permisos personalizados según sea necesario. Para obtener más información sobre cómo establecer permisos, consulte [notación numérica de UNIX](https://en.wikipedia.org/wiki/File_system_permissions#Numeric_notation) en Wikipedia.

    ```bash
    httpEndpoint=$(az storage account show \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | tr -d '"')
    smbPath=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint))$fileShareName

    if [ -z "$(grep $smbPath\ $mntPath /etc/fstab)" ]; then
        echo "$smbPath $mntPath cifs nofail,vers=3.0,credentials=$smbCredentialFile,serverino" | sudo tee -a /etc/fstab > /dev/null
    else
        echo "/etc/fstab was not modified to avoid conflicting entries as this Azure file share was already present. You may want to double check /etc/fstab to ensure the configuration is as desired."
    fi

    sudo mount -a
    ```
    
    > [!Note]  
    > El comando de montaje anterior se monta con SMB 3.0. Si la distribución de Linux no admite SMB 3.0 con cifrado o si solo admite SMB 2.1, solo puede montar desde una máquina virtual de Azure dentro de la misma región que la cuenta de almacenamiento. Para montar el recurso compartido de archivos de Azure en una distribución de Linux que no admita SMB 3.0 con cifrado, necesitará [deshabilitar el cifrado en tránsito para la cuenta de almacenamiento](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

### <a name="using-autofs-to-automatically-mount-the-azure-file-shares"></a>Uso de autofs para montar automáticamente los recursos compartidos de archivos de Azure

1. **Asegúrese de que el paquete autofs está instalado**.  

    El paquete autofs se puede instalar con el administrador de paquetes en la distribución de Linux de su elección. 

    En distribuciones de **Ubuntu** y **Debian**, use el administrador de paquetes `apt`:
    ```bash
    sudo apt update
    sudo apt install autofs
    ```
    En **Fedora**, **Red Hat Enterprise Linux 8 +**, y **CentOS 8 +**, use el administrador de paquetes `dnf`:
    ```bash
    sudo dnf install autofs
    ```
    En las versiones anteriores de **Red Hat Enterprise Linux** y **CentOS**, use el administrador de paquetes `yum`:
    ```bash
    sudo yum install autofs 
    ```
    En **openSUSE**, use el administrador de paquetes `zypper`:
    ```bash
    sudo zypper install autofs
    ```
2. **Cree un punto de montaje para los recursos compartidos**:
   ```bash
    sudo mkdir /fileshares
    ```
3. **Cree un archivo de configuración de autofs personalizado**.
    ```bash
    sudo vi /etc/auto.fileshares
    ```
4. **Agregue las siguientes entradas a /etc/auto.fileshares**.
   ```bash
   echo "$fileShareName -fstype=cifs,credentials=$smbCredentialFile :$smbPath"" > /etc/auto.fileshares
   ```
5. **Agregue la siguiente entrada a /etc/auto.master**.
   ```bash
   /fileshares /etc/auto.fileshares --timeout=60
   ```
6. **Reinicie autofs**.
    ```bash
    sudo systemctl restart autofs
    ```
7.  **Acceda a la carpeta designada para el recurso compartido**.
    ```bash
    cd /fileshares/$filesharename
    ```
## <a name="securing-linux"></a>Protección de Linux
Para montar un recurso compartido de archivos de Azure en Linux, el puerto 445 debe estar accesible. Muchas organizaciones bloquean este puerto debido a los riesgos de seguridad inherentes a SMB 1. SMB 1, también conocido como CIFS (Sistema de archivos de Internet común), es un protocolo de sistema de archivos heredado que se incluye en muchas distribuciones de Linux. SMB 1 es un protocolo obsoleto, ineficaz y, lo más importante, no seguro. La buena noticia es que Azure Files no es compatible con SMB 1 y, a partir de la versión 4.18 del kernel de Linux, Linux hace posible deshabilitar SMB 1. Siempre [recomendamos encarecidamente](https://aka.ms/stopusingsmb1) deshabilitar SMB 1 en los clientes Linux antes de usar recursos compartidos de archivos SMB en producción.

A partir del kernel de Linux 4.18, el módulo de kernel de SMB, denominado `cifs` por motivos de herencia, expone un nuevo parámetro de módulo (a menudo conocido como *parm* en diversos documentos externos), denominado `disable_legacy_dialects`. Aunque se presentó en el kernel de Linux 4.18, algunos proveedores han trasladado este cambio a los kernels más antiguos que admiten. Para mayor comodidad, en la tabla siguiente se detalla la disponibilidad de este parámetro de módulo en distribuciones de Linux comunes.

| Distribución | Puede deshabilitar SMB 1 |
|--------------|-------------------|
| Ubuntu 14.04-16.04 | No |
| Ubuntu 18.04 | Sí |
| Ubuntu 19.04+ | Sí |
| Debian 8-9 | No |
| Debian 10+ | Sí |
| Fedora 29+ | Sí |
| CentOS 7 | No | 
| CentOS 8+ | Sí |
| Red Hat Enterprise Linux 6.x-7.x | No |
| Red Hat Enterprise Linux 8+ | Sí |
| openSUSE Leap 15.0 | No |
| openSUSE Leap 15.1+ | Sí |
| openSUSE Tumbleweed | Sí |
| SUSE Linux Enterprise 11.x-12.x | No |
| SUSE Linux Enterprise 15 | No |
| SUSE Linux Enterprise 15.1 | No |

Puede comprobar si la distribución de Linux es compatible con el parámetro del módulo `disable_legacy_dialects` a través del comando siguiente.

```bash
sudo modinfo -p cifs | grep disable_legacy_dialects
```

Este comando debe generar el siguiente mensaje:

```output
disable_legacy_dialects: To improve security it may be helpful to restrict the ability to override the default dialects (SMB2.1, SMB3 and SMB3.02) on mount with old dialects (CIFS/SMB1 and SMB2) since vers=1.0 (CIFS/SMB1) and vers=2.0 are weaker and less secure. Default: n/N/0 (bool)
```

Antes de deshabilitar SMB 1, debe comprobar para asegurarse de que el módulo SMB no está cargado actualmente en el sistema (esto sucede automáticamente si ha montado un recurso compartido de SMB). Puede hacerlo con el siguiente comando, que no debe generar nada si no se carga SMB:

```bash
lsmod | grep cifs
```

Para descargar el módulo, desmonte primero todos los recursos compartidos de SMB (con el comando `umount` como se describió anteriormente). Puede identificar todos los recursos compartidos de SMB montados en el sistema con el siguiente comando:

```bash
mount | grep cifs
```

Una vez que haya desmontado todos los recursos compartidos de archivos SMB, es seguro descargar el módulo. Para ello, puede usar el comando `modprobe` :

```bash
sudo modprobe -r cifs
```

Puede cargar manualmente el módulo con SMB 1 descargado mediante el comando `modprobe`:

```bash
sudo modprobe cifs disable_legacy_dialects=Y
```

Por último, puede comprobar que el módulo SMB se ha cargado con el parámetro, para ello, examine los parámetros cargados en `/sys/module/cifs/parameters`:

```bash
cat /sys/module/cifs/parameters/disable_legacy_dialects
```

Para deshabilitar de forma persistente SMB 1 en las distribuciones basadas en Ubuntu y Debian, debe crear un nuevo archivo (si aún no tiene opciones personalizadas para otros módulos) denominado `/etc/modprobe.d/local.conf` con la configuración. Para ello, use el siguiente comando:

```bash
echo "options cifs disable_legacy_dialects=Y" | sudo tee -a /etc/modprobe.d/local.conf > /dev/null
```

Para comprobar que esto ha funcionado, cargue el módulo SMB:

```bash
sudo modprobe cifs
cat /sys/module/cifs/parameters/disable_legacy_dialects
```

## <a name="next-steps"></a>Pasos siguientes
Consulte los vínculos siguientes para más información sobre Azure Files:

* [Planeamiento de una implementación de Azure Files](storage-files-planning.md)
* [P+F](../storage-files-faq.md)
* [Solución de problemas](storage-troubleshoot-linux-file-connection-problems.md)
