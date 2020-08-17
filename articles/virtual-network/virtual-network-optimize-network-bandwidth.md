---
title: Optimización del rendimiento de la red de una máquina virtual | Microsoft Docs
description: Optimice el rendimiento de la red de las VM Windows y Linux de Microsoft Azure, incluidas las distribuciones principales, como Ubuntu, CentOS y Red Hat.
services: virtual-network
documentationcenter: na
author: steveesp
manager: Gerald DeGrace
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/15/2017
ms.author: steveesp
ms.openlocfilehash: 7b693d10b4e1925e9c07111982a616b56b77e5b6
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87265150"
---
# <a name="optimize-network-throughput-for-azure-virtual-machines"></a>Optimización del rendimiento de red en las máquinas virtuales de Azure

Las máquinas virtuales de Azure (VM) tienen una configuración de red predeterminada que se puede optimizar para mejorar aún más el rendimiento de la red. En este artículo se describe cómo optimizar el rendimiento de la red de las máquinas virtuales Windows y Linux de Microsoft Azure, incluidas las distribuciones principales como Ubuntu, CentOS y Red Hat.

## <a name="windows-vm"></a>Máquina virtual de Windows

Si la máquina virtual Windows es compatible con [redes aceleradas](create-vm-accelerated-networking-powershell.md), habilite esta característica para conseguir un rendimiento óptimo. En el caso de otras máquinas virtuales Windows, el uso de escalado en el lado de la recepción (RSS) pueden logar un rendimiento máximo mayor que las que no lo usan. En las máquinas virtuales Windows, RSS se puede deshabilitar de forma predeterminada. Para determinar si RSS está habilitado y para habilitarlo si no lo está en la actualidad, complete los siguientes pasos:

1. Compruebe si RSS está habilitado para un adaptador de red con el comando `Get-NetAdapterRss` de PowerShell. En la siguiente salida de ejemplo que devuelve `Get-NetAdapterRss`, RSS no está habilitado.

    ```powershell
    Name                    : Ethernet
    InterfaceDescription    : Microsoft Hyper-V Network Adapter
    Enabled                 : False
    ```
2. Para habilitar RSS escriba el siguiente comando:

    ```powershell
    Get-NetAdapter | % {Enable-NetAdapterRss -Name $_.Name}
    ```
    El comando anterior no tiene ninguna salida. El comando cambió la configuración de NIC, lo que ha provocado una pérdida temporal de la conectividad durante aproximadamente un minuto. Durante la pérdida de conectividad aparece un cuadro de diálogo de reconexión. La conectividad se suele restaurar al tercer intento.
3. Confirme que RSS está habilitado en la máquina virtual, para lo que debe volver a escribir el comando `Get-NetAdapterRss`. Si se realiza correctamente, se devuelve la siguiente salida de ejemplo:

    ```powershell
    Name                    : Ethernet
    InterfaceDescription    : Microsoft Hyper-V Network Adapter
    Enabled                  : True
    ```

## <a name="linux-vm"></a>Máquina virtual de Linux

De manera predeterminada, en las máquinas virtuales Linux de Azure RSS está siempre habilitado. Los kernels de Linux lanzados desde octubre de 2017 incluyen nuevas opciones de optimización de red que permiten que las máquinas virtuales Linux logren un mayor rendimiento de la red.

### <a name="ubuntu-for-new-deployments"></a>Ubuntu para las nuevas implementaciones

El kernel de Azure de Ubuntu proporciona el mejor rendimiento de la red en Azure y ha sido el kernel predeterminado desde el 21 de septiembre de 2017. Para obtener este kernel, primero instale la última versión compatible de 16.04-LTS, como se describe a continuación:

```json
"Publisher": "Canonical",
"Offer": "UbuntuServer",
"Sku": "16.04-LTS",
"Version": "latest"
```

Una vez que la creación finaliza, escriba los siguientes comandos para obtener el actualizaciones más recientes. Estos pasos también funcionan en las máquinas virtuales que ejecutan actualmente el kernel de Azure de Ubuntu.

```bash
#run as root or preface with sudo
apt-get -y update
apt-get -y upgrade
apt-get -y dist-upgrade
```

El siguiente conjunto de comandos opcional puede ser útil para las implementaciones de Ubuntu existentes que ya tiene el kernel de Azure, pero que no han podido realizar actualizaciones debido a errores.

```bash
#optional steps may be helpful in existing deployments with the Azure kernel
#run as root or preface with sudo
apt-get -f install
apt-get --fix-missing install
apt-get clean
apt-get -y update
apt-get -y upgrade
apt-get -y dist-upgrade
```

#### <a name="ubuntu-azure-kernel-upgrade-for-existing-vms"></a>Actualización del kernel de Azure de Ubuntu para las máquinas virtuales existentes

Se puede lograr un rendimiento significativo instalando el kernel de Linux de Azure propuesto. Para comprobar si tienen este kernel, compruebe la versión del kernel.

```bash
#Azure kernel name ends with "-azure"
uname -r

#sample output on Azure kernel:
#4.13.0-1007-azure
```

Si la máquina virtual no tiene el kernel de Azure, el número de versión comenzará normalmente con "4.4." Si la máquina virtual no tiene el kernel de Azure, ejecute los comandos siguientes como raíz:

```bash
#run as root or preface with sudo
apt-get update
apt-get upgrade -y
apt-get dist-upgrade -y
apt-get install "linux-azure"
reboot
```

### <a name="centos"></a>CentOS

Para obtener las optimizaciones más recientes, se recomienda crear una máquina virtual con la versión más reciente compatible mediante la especificación de los siguientes parámetros:

```json
"Publisher": "OpenLogic",
"Offer": "CentOS",
"Sku": "7.4",
"Version": "latest"
```

Tanto las máquinas virtuales nuevas como las existentes se pueden beneficiar de la versión más reciente de Linux Integration Services (LIS). La optimización del rendimiento es en LIS, a partir de 4.2.2-2, aunque las versiones posteriores contienen más mejoras. Escriba los siguientes comandos para instalar el LIS más reciente:

```bash
sudo yum update
sudo reboot
sudo yum install microsoft-hyper-v
```

### <a name="red-hat"></a>Red Hat

Para obtener las optimizaciones, se recomienda crear una máquina virtual con la versión más reciente compatible mediante la especificación de los siguientes parámetros:

```json
"Publisher": "RedHat"
"Offer": "RHEL"
"Sku": "7-RAW"
"Version": "latest"
```

Tanto las máquinas virtuales nuevas como las existentes se pueden beneficiar de la versión más reciente de Linux Integration Services (LIS). La optimización del rendimiento se realiza en LIS a partir de la versión 4.2. Escriba los siguientes comandos para descargar e instalar LIS:

```bash
wget https://aka.ms/lis
tar xvf lis
cd LISISO
sudo ./install.sh #or upgrade.sh if prior LIS was previously installed
```

Para más información sobre la versión 4.2 de Linux Integration Services para Hyper-V, vea la [página de descarga](https://www.microsoft.com/download/details.aspx?id=55106).

## <a name="next-steps"></a>Pasos siguientes
* Vea el resultado con las [pruebas de ancho de banda y rendimiento de Azure VM](virtual-network-bandwidth-testing.md) para su escenario.
* Obtenga información acerca de cómo [se asigna el ancho de banda a las máquinas virtuales](virtual-machine-network-throughput.md)
* Obtenga más información con las [Preguntas más frecuentes (P+F) acerca de Azure Virtual Network](virtual-networks-faq.md)
