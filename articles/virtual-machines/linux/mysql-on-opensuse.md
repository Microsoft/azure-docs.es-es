---
title: Instalación de MySQL en una máquina virtual con OpenSUSE en Azure
description: Aprenda a instalar MySQL en una máquina virtual con OpenSUSE Linux en Azure.
services: virtual-machines-linux
author: cynthn
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: how-to
ms.date: 07/11/2018
ms.author: cynthn
ms.openlocfilehash: eee32dc7edd4256dd2bd120609504042d7ab78ea
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836893"
---
# <a name="install-mysql-on-a-virtual-machine-running-opensuse-linux-in-azure"></a>Instalación de MySQL en una máquina virtual que ejecuta OpenSUSE Linux en Azure

[MySQL](https://www.mysql.com) es una conocida base de datos SQL de código abierto. En este tutorial se muestra cómo crear una máquina virtual que disponga de OpenSUSE Linux y, a continuación, instale MySQL.


Si decide instalar y usar la CLI localmente, necesitará la CLI de Azure versión 2.0 o posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-virtual-machine-running-opensuse-linux"></a>Creación de una máquina virtual que ejecuta OpenSUSE Linux

En primer lugar, cree un grupo de recursos. En este ejemplo, el grupo de recursos se denomina *mySQSUSEResourceGroup* y se crea en la región *Este de EE. UU.* .

```azurecli-interactive
az group create --name mySQLSUSEResourceGroup --location eastus
```

Cree la máquina virtual. En este ejemplo, la máquina virtual se denomina *myVM* y el tamaño de la máquina virtual es *Standard_D2s_v3*, pero debe elegir el [Tamaño de VM](../sizes.md) que crea más adecuado para la carga de trabajo.

```azurecli-interactive
az vm create --resource-group mySQLSUSEResourceGroup \
   --name myVM \
   --image openSUSE-Leap \
   --size Standard_D2s_v3 \
   --generate-ssh-keys
```

También debe agregar una regla al grupo de seguridad de red para permitir el tráfico a través del puerto 3306 para MySQL.

```azurecli-interactive
az vm open-port --port 3306 --resource-group mySQLSUSEResourceGroup --name myVM
```

## <a name="connect-to-the-vm"></a>Conexión a la máquina virtual

Usará SSH para conectarse a la máquina virtual. En este ejemplo, la dirección IP pública de la máquina virtual es *10.111.112.113*. Puede ver la dirección IP en la salida de cuando se creó la VM.

```azurecli-interactive  
ssh 10.111.112.113
```

 
## <a name="update-the-vm"></a>Actualizar la VM
 
Después de conectarse a la VM, instale las actualizaciones y revisiones del sistema. 
   
```bash
sudo zypper update
```

Siga las indicaciones para actualizar la VM.

## <a name="install-mysql"></a>Instalar MySQL 


Instale MySQL en la VM a través de SSH. Responda a las solicitudes según corresponda.

```bash
sudo zypper install mysql
```
 
Establezca MySQL para que se inicie con el arranque del sistema. 

```bash
sudo systemctl enable mysql
```
Compruebe que MySQL esté habilitado.

```bash
systemctl is-enabled mysql
```

Debe devolver: enabled.

Reinicie el servidor.

```bash
sudo reboot
```


## <a name="mysql-password"></a>Contraseña de MySQL

después de la instalación, la contraseña raíz de MySQL se encuentra vacía de forma predeterminada. Ejecute el script **mysql\_secure\_installation** para proteger MySQL. El script le solicitará que cambie la contraseña raíz de MySQL, quite las cuentas de usuario anónimas, deshabilite los datos de inicio de sesión raíz remotos, quite las bases de datos de prueba y vuelva a cargar la tabla de privilegios. 

Una vez que se reinicie el servidor, haga ssh a la máquina virtual de nuevo.

```azurecli-interactive  
ssh 10.111.112.113
```



```bash
mysql_secure_installation
```

## <a name="sign-in-to-mysql"></a>Iniciar sesión en MySQL

Ahora puede iniciar sesión e introducir el símbolo del sistema de MySQL.

```bash  
mysql -u root -p
```
Esto cambia al símbolo del sistema de MySQL, donde puede emitir instrucciones SQL para interactuar con la base de datos.

Ahora, cree un nuevo usuario de MySQL.

```sql
CREATE USER 'mysqluser'@'localhost' IDENTIFIED BY 'password';
```
   
El punto y coma (;) al final de la línea es fundamental para terminar el comando.


## <a name="create-a-database"></a>Crear una base de datos


Cree una base de datos y conceda los permisos de usuario `mysqluser`.

```sql
CREATE DATABASE testdatabase;
GRANT ALL ON testdatabase.* TO 'mysqluser'@'localhost' IDENTIFIED BY 'password';
```
   
Los nombres de usuario y contraseñas de la base de datos solo los usan los scripts que se conectan a la base de datos.  Los nombres de cuenta de usuario de la base de datos no representan necesariamente las cuentas de usuario reales en el sistema.

Habilite el inicio de sesión desde otro equipo. En este ejemplo, la dirección IP del equipo desde el que se va a permitir iniciar sesión es *10.112.113.114*.

```sql
GRANT ALL ON testdatabase.* TO 'mysqluser'@'10.112.113.114' IDENTIFIED BY 'password';
```
   
Para salir de la utilidad administración de base de datos MySQL, escriba:

```    
quit
```


## <a name="next-steps"></a>Pasos siguientes
Para consultar detalles sobre MySQL, vea la [documentación de MySQ](https://dev.mysql.com/doc).