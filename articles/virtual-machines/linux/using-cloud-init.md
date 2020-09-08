---
title: Información general de la compatibilidad de cloud-init con máquinas virtuales Linux en Azure
description: Información general de las funcionalidades cloud-init para configurar una máquina virtual en tiempo de aprovisionamiento en Azure.
author: danielsollondon
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 06/15/2020
ms.author: danis
ms.openlocfilehash: 7ddbb48f3598780988feb25a11729a5086d31fde
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88869276"
---
# <a name="cloud-init-support-for-virtual-machines-in-azure"></a>Compatibilidad con cloud-init para máquinas virtuales en Azure
En este artículo se explica la compatibilidad que existe para [cloud-init](https://cloudinit.readthedocs.io) para configurar una máquina virtual (VM) o conjuntos de escalado de máquinas virtuales en el momento del aprovisionamiento en Azure. Estas configuraciones de cloud-init se ejecutan durante el primer arranque una vez que Azure ha aprovisionado los recursos.  

El aprovisionamiento de máquinas virtuales es el proceso en el que Azure pasará los valores de los parámetros de creación de la máquina virtual, como nombre de host, nombre de usuario, contraseña, etc., y los pone a disposición de la máquina virtual cuando se inicia. Un "agente de aprovisionamiento" consumirá esos valores, configurará la máquina virtual y devolverá una notificación cuando se complete. 

Azure admite dos agentes de aprovisionamiento [cloud-init](https://cloudinit.readthedocs.io) y el [agente de Linux de Azure (WALA)](../extensions/agent-linux.md).

## <a name="cloud-init-overview"></a>Introducción a cloud-init
[cloud-init](https://cloudinit.readthedocs.io) es un enfoque ampliamente usado para personalizar una máquina virtual Linux la primera vez que se arranca. Puede usar cloud-init para instalar paquetes y escribir archivos o para configurar los usuarios y la seguridad. Dado que se llama a cloud-init durante el proceso de arranque inicial, no se necesitan pasos adicionales ni agentes para aplicar la configuración.  Para más información sobre cómo dar formato correctamente a los archivos `#cloud-config` u otras entradas, consulte el [sitio de documentación de cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/format.html#cloud-config-data).  Los archivos `#cloud-config` son archivos de texto codificados en base64.

cloud-init también funciona entre distribuciones. Por ejemplo, no use **apt-get install** o **yum install** para instalar un paquete. En su lugar, puede definir una lista de paquetes que se van a instalar. cloud-init usará automáticamente la herramienta de administración de paquetes nativos para la distribución de Linux (distro) que seleccione.

Estamos trabajando activamente con nuestros asociados de distribuciones de Linux certificadas para disponer de imágenes con cloud-init habilitado en Azure Marketplace. Estas imágenes harán que las implementaciones y configuraciones de cloud-init funcionen perfectamente con las máquinas virtuales y los conjuntos de escalado de máquinas virtuales. Inicialmente colaboramos con los partners de distribución de Linux aprobados y el nivel superior para garantizar las funciones de cloud-init con el sistema operativo en Azure; posteriormente, los paquetes se actualizan y están disponibles públicamente en los repositorios de paquetes de distribución. 

Hay dos fases para que cloud-init esté disponible para el sistema operativo de distribución de Linux aprobado en Azure, la compatibilidad con paquetes y, después, la compatibilidad con imágenes:
* La "compatibilidad con paquetes de cloud-init en Azure" documenta que paquetes de cloud-init se admiten o están en versión preliminar, por lo que puede usar estos paquetes con el sistema operativo en una imagen personalizada.
* "Imagen preparada para cloud-init" documenta si la imagen ya está configurada para usar cloud-init.


### <a name="canonical"></a>Canonical
| Publicador o versión| Oferta | SKU | Versión | Imagen preparada para cloud-init | compatibilidad con paquetes de cloud-init en Azure|
|:--- |:--- |:--- |:--- |:--- |:--- |
|Canonical 20.04 |UbuntuServer |18.04-LTS |latest |sí | sí |
|Canonical 18.04 |UbuntuServer |18.04-LTS |latest |sí | sí |
|Canonical 16.04|UbuntuServer |16.04-LTS |latest |sí | sí |
|Canonical 14.04|UbuntuServer |14.04.5-LTS |latest |sí | sí |

### <a name="rhel"></a>RHEL
| Publicador o versión | Oferta | SKU | Versión | Imagen preparada para cloud-init | compatibilidad con paquetes de cloud-init en Azure|
|:--- |:--- |:--- |:--- |:--- |:--- |
|RedHat 7.6 |RHEL |7-RAW-CI |7.6.2019072418 |sí | sí, compatibilidad desde la versión del paquete: *18.2-1.el7_6.2*|
|RedHat 7.7 |RHEL |7-RAW-CI |7.7.2019081601 | sí (nota: se trata de una imagen en versión preliminar y **no debe** usarse ya más, pues se suprimirá el 1 de septiembre de 2020) | N/D |
|RedHat 7.7 (GEN1)|RHEL |7,7 | 7.7.2020051912 | sí | sí, compatibilidad desde la versión del paquete: *18.5-6.el7*|
|RedHat 7.7 (Gen2)|RHEL | 77-gen2 | 7.7.2020051913 | sí | sí, compatibilidad desde la versión del paquete: *18.5-6.el7*|
|RedHat 7.7 (GEN1)|RHEL |7-LVM | 7.7.2020051921 | sí | sí, compatibilidad desde la versión del paquete: *18.5-6.el7*|
|RedHat 7.7 (Gen2)|RHEL | 7lvm-gen2 | 7.7.2020051922  | sí | sí, compatibilidad desde la versión del paquete: *18.5-6.el7*|
|RedHat 7.7 (GEN1) |rhel-byos | rhel-lvm77 | 7.7.20200416 | sí  | sí, compatibilidad desde la versión del paquete: *18.5-6.el7*|
|RedHat 8.1 (GEN1) |RHEL |8.1-ci |8.1.2020042511 | sí (nota: se trata de una imagen en versión preliminar y, una vez que todas las imágenes de RHEL 8.1 admitan cloud-init, se suprimirá el 1 de agosto de 2020) | No, ETA para obtener soporte técnico completo junio de 2020|
|RedHat 8.1 (Gen2) |RHEL |81-ci-gen2 |8.1.2020042524 | sí (nota: se trata de una imagen en versión preliminar y, una vez que todas las imágenes de RHEL 8.1 admitan cloud-init, se suprimirá el 1 de agosto de 2020) | No, ETA para obtener soporte técnico completo junio de 2020 |

* Todo RedHat: las imágenes RHEL 7.8 y 8.2 (Gen1 y Gen2) se aprovisionan mediante cloud-init.

### <a name="centos"></a>CentOS

| Publicador o versión | Oferta | SKU | Versión | Imagen preparada para cloud-init | compatibilidad con paquetes de cloud-init en Azure|
|:--- |:--- |:--- |:--- |:--- |:--- |
|OpenLogic 7.7 |CentOS |7-CI |7.7.20190920 |sí (nota: se trata de una imagen en versión preliminar y **no debe** usarse ya más, pues se suprimirá el 1 de septiembre de 2020) | N/D |
|OpenLogic 7.7 |CentOS | 7,7 |7.7.2020062400 |sí | sí, compatibilidad desde la versión del paquete: `18.5-6.el7.centos.5`|
|OpenLogic 7.7 (Gen2) |CentOS | 7_7-gen2 |7.7.2020062401 |sí | sí, compatibilidad desde la versión del paquete: `18.5-6.el7.centos.5`|
|OpenLogic 7.7 |CentOS-HPC | 7,7 |7.6.2020062600 |sí | sí, compatibilidad desde la versión del paquete: `18.5-6.el7.centos.5`|
|OpenLogic 7.7 (Gen2) |CentOS-HPC | 7_7-gen2 |7.6.2020062601 |sí | sí, compatibilidad desde la versión del paquete: `18.5-6.el7.centos.5`|
|OpenLogic 8.1 |CentOS | 8_1 |8.1.2020062400 |sí | sí, compatibilidad desde la versión del paquete: `18.5-7.el8_1.1`|
|OpenLogic 8.1 (Gen2) |CentOS | 8_1-gen2 |8.1.2020062401 |sí | sí, compatibilidad desde la versión del paquete: `18.5-7.el8_1.1`|
|OpenLogic 8.1 |CentOS-HPC | 8_1 |8.1.2020062400 |sí | sí, compatibilidad desde la versión del paquete: `18.5-7.el8_1.1`|
|OpenLogic 8.1 (Gen2) |CentOS-HPC:8_1-gen2 | 8_1-gen2 |8.1.2020062401 |sí | sí, compatibilidad desde la versión del paquete: `18.5-7.el8_1.1`|

* Todo OpenLogic: las imágenes de CentOS 7.8 y 8.2 (Gen1 y Gen 2) se aprovisionan mediante cloud-init.

### <a name="oracle"></a>Oracle

| Publicador o versión | Oferta | SKU | Versión | Imagen preparada para cloud-init | compatibilidad con paquetes de cloud-init en Azure|
|:--- |:--- |:--- |:--- |:--- |:--- |
|Oracle 7.7 |Oracle-Linux |77-ci |7.7.01| Imagen en versión preliminar (nota: se trata de una imagen en versión preliminar y, una vez que todas las imágenes de Oracle 7.7 admitan cloud-init, se suprimirá a mediados de 2020; se informará de ello) | no, en la versión preliminar, el paquete es: *18.5-3.0.1.el7*

### <a name="suse-sles"></a>SUSE SLES
Estas imágenes de SLES se han actualizado para aprovisionar mediante cloud-init. También se han actualizado las variantes de la imagen de Gen2.
* suse:sles-15-sp1-{basic/byos/hpc/hpc-byos/chost-byos}:gen1:2020.06.10
* suse:sles-sap-15-sp1:gen1:2020.06.10
* suse:sles-sap-15-sp1-byos:gen1:2020.06.10
* suse:manager-proxy-4-byos:gen1:2020.06.10
* suse:manager-server-4-byos:gen1:2020.06.10
* suse:sles-{byos/sap/sap-byos}:15:2020.06.10
* suse:sles-12-sp5:gen1:2020.06.10
* suse:sles-12-sp5{-byos/basic/hpc-byos/hpc}:gen1:2020.06.10
* suse:sles-{byos/sap/sap-byos}:12-sp4:2020.06.10
* suse:sles-{byos/sap/sap-byos}:12-sp3:2020.06.10
* suse:sles-{byos/sap/sap-byos}:12-sp2:2020.06.10


### <a name="debian"></a>Debian
| Publicador o versión | Oferta | SKU | Versión | Imagen preparada para cloud-init | compatibilidad con paquetes de cloud-init en Azure|
|:--- |:--- |:--- |:--- |:--- |:--- |
| debian (Gen1) |debian-10 | 10-cloudinit |cloud-init-preview| sí (solo vista previa) | No, en versión preliminar. |
| debian (Gen2) |debian-10 | 10-cloudinit-gen2 |cloud-init-preview| sí (solo vista previa) | No, en versión preliminar. |




Actualmente, Azure Stack admitirá el aprovisionamiento de imágenes habilitadas para cloud-init.

## <a name="what-is-the-difference-between-cloud-init-and-the-linux-agent-wala"></a>¿Cuál es la diferencia entre cloud-init y el agente de Linux (WALA)?
WALA es un agente específico de la plataforma de Azure que se usa para aprovisionar y configurar máquinas virtuales y controlar las [extensiones de Azure](../extensions/features-linux.md). 

Estamos mejorando la tarea de configuración de las máquinas virtuales para usar cloud-init en lugar del agente de Linux con objeto de permitir que los clientes existentes de cloud-init usen sus scripts actuales de cloud-init o que los nuevos clientes se aprovechen de la rica funcionalidad de configuración de cloud-init. Si tiene inversiones existentes en scripts de cloud-init para configurar sistemas Linux, **no se necesitan configuraciones adicionales** para permitir que cloud-init las procese. 

cloud-init no puede procesar las extensiones de Azure, por lo que todavía se requiere WALA en la imagen para procesar extensiones, pero debe tener deshabilitado su código de aprovisionamiento para que las imágenes de distribuciones de Linux aprobadas que se van a convertir en cloud-init tengan WALA instalado y configurado correctamente.

Al crear una máquina virtual, si no se incluye el modificador `--custom-data` de la CLI de Azure en el momento del aprovisionamiento, cloud-init o WALA toma los parámetros de aprovisionamiento de máquina virtual mínimos necesarios para aprovisionar la máquina virtual y completar la implementación con los valores predeterminados.  Si hace referencia a la configuración de cloud-init con el modificador `--custom-data`, el contenido de los datos personalizados estará disponible en cloud-init cuando arranque la máquina virtual.

Las configuraciones de cloud-init que se aplican a las máquinas virtuales no tienen restricciones de tiempo y no producirán un error de implementación porque se cumpla el tiempo de espera. Esto no es cierto para WALA, si cambia los valores predeterminados de WALA para procesar los datos personalizados, no puede superar el tiempo total de aprovisionamiento de la máquina virtual de 40 min; si es así, se producirá un error al crear máquina virtual.

## <a name="deploying-a-cloud-init-enabled-virtual-machine"></a>Implementación de una máquina Virtual con cloud-init habilitado
La implementación de una máquina virtual con cloud-init habilitado es tan simple como hacer referencia a una distribución con cloud-init habilitado durante la implementación.  Los encargados de mantener la distribución de Linux tienen que elegir habilitar e integrar cloud-init en sus imágenes básicas publicadas de Azure. Cuando haya confirmado que la imagen que quiere implementar tiene habilitado cloud- init, puede usar la CLI de Azure para implementar la imagen. 

El primer paso para implementar esta imagen es crear un grupo de recursos con el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

El paso siguiente es crear un archivo en el shell actual, denominado *cloud-init.txt* y pegar la siguiente configuración. Para este ejemplo, cree el archivo en Cloud Shell, no en la máquina local. Puede utilizar el editor que prefiera. Escriba `sensible-editor cloud-init.txt` para crear el archivo y ver una lista de editores disponibles. Elija el número 1 para utilizar el editor **nano**. Asegúrese de que todo el archivo cloud-init se copia correctamente, especialmente la primera línea:

```yaml
#cloud-config
package_upgrade: true
packages:
  - httpd
```
Presione `ctrl-X` para salir del archivo, escriba `y` para guardar el archivo y presione `enter` para confirmar el nombre del archivo al salir.

El paso final es crear la máquina virtual con el comando [az vm create](/cli/azure/vm). 

En el ejemplo siguiente, se crea una máquina virtual denominada *centos74* y las claves SSH si aún no existen en una ubicación de claves predeterminada. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`.  Use el parámetro `--custom-data` para pasar su archivo de configuración cloud-init. Proporcione la ruta de acceso completa a la configuración de *cloud-init.txt* si guardó el archivo fuera de su directorio de trabajo actual. 

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS-CI:7-CI:latest \
  --custom-data cloud-init.txt \
  --generate-ssh-keys 
```

Cuando se haya creado la VM, la CLI de Azure mostrará información específica a la implementación. Anote el valor de `publicIpAddress`. Esta dirección se utiliza para tener acceso a la máquina virtual.  Transcurre algo de tiempo hasta que la máquina virtual se crea, los paquetes se instalan y la aplicación se inicia. Hay tareas en segundo plano que continúan ejecutándose después de que la CLI de Azure vuelve a abrir el símbolo del sistema. Puede usar SSH en la VM y usar los pasos descritos en la sección de solución de problemas para ver los registros de cloud-init. 

También puede implementar una máquina virtual con cloud-init habilitado pasando los [parámetros en la plantilla de Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-cli#inline-parameters).

## <a name="troubleshooting-cloud-init"></a>Solución de problemas de cloud-init
Una vez que se ha aprovisionado la máquina virtual, cloud-init se ejecuta en todos los módulos y en el script definido en `--custom-data` para configurar dicha máquina virtual.  Si tiene que solucionar algún error u omisión de la configuración, debe buscar el nombre del módulo (`disk_setup` o `runcmd`,0 por ejemplo) en el registro de cloud-init, ubicado en **/var/log/cloud-init.log**.

> [!NOTE]
> No todos los errores de módulo dan como resultado un error irrecuperable en la configuración global de cloud-init. Por ejemplo, si se usa el módulo `runcmd` y se produce un error en el script, cloud-init seguirá notificando un aprovisionamiento correcto porque se ejecutó el módulo runcmd.

Para más detalles sobre el registro de cloud-init, consulte la [documentación de cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/logging.html). 

## <a name="next-steps"></a>Pasos siguientes

[Solución de problemas con cloud-init](cloud-init-troubleshooting.md)


Para ejemplos de cloud-init de cambios de configuración, vea los siguientes documentos:
 
- [Incorporación de otro usuario de Linux a una máquina virtual](cloudinit-add-user.md)
- [Ejecución de un administrador de paquetes para actualizar los existentes durante el primer arranque](cloudinit-update-vm.md)
- [Cambio del nombre de host de la máquina virtual local](cloudinit-update-vm-hostname.md) 
- [Instalación de un paquete de aplicación, actualización de los archivos de configuración e inserción de claves](tutorial-automate-vm-deployment.md)
 
