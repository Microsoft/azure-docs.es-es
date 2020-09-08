---
title: Red Hat Update Infrastructure | Microsoft Docs
description: Obtenga información acerca de Red Hat Update Infrastructure para instancias de Red Hat Enterprise Linux a petición de Microsoft Azure
services: virtual-machines-linux
documentationcenter: ''
author: asinn826
manager: BorisB2015
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 02/10/2020
ms.author: alsin
ms.openlocfilehash: 641ac1f6a2cc98e48694c42ec1531f679621640d
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88869225"
---
# <a name="red-hat-update-infrastructure-for-on-demand-red-hat-enterprise-linux-vms-in-azure"></a>Red Hat Update Infrastructure para máquinas virtuales Red Hat Enterprise Linux a petición en Azure
 [Red Hat Update Infrastructure](https://access.redhat.com/products/red-hat-update-infrastructure) (RHUI) permite que los proveedores de nube, como Azure, reflejen el contenido del repositorio hospedado en Red Hat, creen repositorios personalizados con contenido específico de Azure y lo pongan a disposición de las máquinas virtuales del usuario final.

Las imágenes de pago por uso (PAYG) de Red Hat Enterprise Linux (RHEL) vienen preconfiguradas para acceder a RHUI de Azure. No se necesita ninguna configuración adicional. Para obtener las actualizaciones más recientes, ejecute `sudo yum update` después de que la instancia RHEL esté lista. Este servicio se incluye como parte de las tarifas de software de PAYG de RHEL.

Hay disponible información adicional sobre las imágenes de RHEL de Azure, incluidas las directivas de publicación y retención, [aquí](./redhat-images.md).

Puede encontrar información sobre las directivas de soporte técnico de Red Hat para todas las versiones de RHEL en la página [Red Hat Enterprise Linux Life Cycle](https://access.redhat.com/support/policy/updates/errata) (Ciclo de vida de Red Hat Enterprise Linux).

> [!IMPORTANT]
> RHUI está pensado únicamente para imágenes de pago por uso (PAYG). En el caso de las imágenes personalizadas y maestras, también conocidas como "traiga su propia suscripción" (BYOS), el sistema debe estar conectado a RHSM o Satellite para recibir actualizaciones. Consulte el [artículo de Red Hat](https://access.redhat.com/solutions/253273) para más información.


## <a name="important-information-about-azure-rhui"></a>Información importante sobre RHUI de Azure

* RHUI de Azure es la infraestructura de actualización que admite todas las máquinas virtuales RHEL de pago por uso creadas en Azure. Esto no impide que registre sus máquinas virtuales RHEL de pago por uso con el Administrador de suscripciones o Satellite, o cualquier otro origen de actualizaciones, pero si lo hace con una máquina virtual de pago por uso provocará una facturación doble indirecta. Para más información, consulte el punto siguiente.
* El acceso a RHUI hospedado en Azure se incluye en el precio de la imagen de PAYG de RHEL. Al anular el registro de una máquina virtual PAYG RHEL en la RHUI hospedada por Azure, no se convierte el tipo de la máquina virtual a "traiga su propia licencia" (BYOL). Si registra la misma máquina virtual con otro origen de actualizaciones, podrían generarse gastos _indirectos_ por duplicado. Se le cobrará la primera vez la cuota de software de Azure RHEL. Se le cobrará la segunda vez para las suscripciones de Red Hat que se adquirieron previamente. Si necesita usar habitualmente una infraestructura de actualizaciones que no sea la RHUI hospedada en Azure, considere la posibilidad de registrarse para usar las [imágenes de RHEL del tipo BYOL](./byos.md).

* Las imágenes de PAYG de RHEL en Azure (RHEL for SAP, RHEL for SAP HANA y RHEL for SAP Business Applications) están conectadas a canales de RHUI dedicados que permanecen en la versión menor específica de RHEL, tal como se requiere para la certificación de SAP.

* El acceso a la RHUI hospedada en Azure se limita a las máquinas virtuales dentro de los [intervalos de direcciones IP del centro de datos de Azure](https://www.microsoft.com/download/details.aspx?id=41653). Si remite todo el tráfico de máquina virtual a través de un proxy mediante la infraestructura de red local, es posible que tenga que configurar rutas definidas por el usuario para que las máquinas virtuales de PAYG de RHEL accedan a RHUI de Azure. En ese caso, será necesario agregar las rutas definidas por el usuario para _todas_ las direcciones IP de RHUI.


## <a name="image-update-behavior"></a>Comportamiento de actualización de imágenes

A partir de abril de 2019, Azure ofrece imágenes de RHEL que están conectadas de forma predeterminada a repositorios de Compatibilidad de actualización extendida (EUS) y las imágenes de RHEL que se conectan a los repositorios normales (no EUS) de forma predeterminada. Puede encontrar más información sobre EUS de RHEL en la [documentación del ciclo de vida de la versión](https://access.redhat.com/support/policy/updates/errata) de Red Hat y en la [documentación de EUS](https://access.redhat.com/articles/rhel-eus). El comportamiento predeterminado de `sudo yum update` variará en función de la imagen de RHEL que haya aprovisionado, ya que las imágenes diferentes se conectan a distintos repositorios.

Para obtener una lista de imágenes completa, ejecute `az vm image list --publisher redhat --all` con la CLI de Azure.

### <a name="images-connected-to-non-eus-repositories"></a>Imágenes conectadas a repositorios que no son de EUS

Si aprovisiona una máquina virtual a partir de una imagen de RHEL que está conectada a repositorios que no son de EUS, se actualizará a la versión secundaria de RHEL más reciente cuando ejecute `sudo yum update`. Por ejemplo, si aprovisiona una máquina virtual desde una imagen de PAYG de RHEL 7.4 y ejecuta `sudo yum update`, terminará con una máquina virtual de RHEL 7.8 (la versión menor más reciente de la familia RHEL7).

Las imágenes que están conectadas a repositorios que no son de EUS no contendrán un número de versión secundaria en la SKU. La SKU es el tercer elemento del URN (nombre completo de la imagen). Por ejemplo, todas las imágenes siguientes se adjuntan a repositorios que no son de EUS:

```text
RedHat:RHEL:7-LVM:7.4.2018010506
RedHat:RHEL:7-LVM:7.5.2018081518
RedHat:RHEL:7-LVM:7.6.2019062414
RedHat:RHEL:7-RAW:7.4.2018010506
RedHat:RHEL:7-RAW:7.5.2018081518
RedHat:RHEL:7-RAW:7.6.2019062120
```

Tenga en cuenta que las SKU son 7-LVM o 7-RAW. La versión secundaria se indica en la versión (cuarto elemento del URN) de estas imágenes.

### <a name="images-connected-to-eus-repositories"></a>Imágenes conectadas a repositorios de EUS

Si aprovisiona una máquina virtual a partir de una imagen de RHEL que está conectada a repositorios de EUS, no se actualizará a la versión secundaria de RHEL más reciente cuando ejecute `sudo yum update`. Esto se debe a que las imágenes conectadas a los repositorios de EUS también tienen bloqueada la versión de la versión secundaria específica.

Las imágenes conectadas a repositorios de EUS contendrán un número de versión secundaria en la SKU. Por ejemplo, todas las imágenes siguientes se adjuntan a los repositorios de EUS:

```text
RedHat:RHEL:7.4:7.4.2019062107
RedHat:RHEL:7.5:7.5.2019062018
RedHat:RHEL:7.6:7.6.2019062116
```

## <a name="rhel-eus-and-version-locking-rhel-vms"></a>Bloqueo de versiones en máquinas virtuales de RHEL y RHEL EUS

Los repositorios de compatibilidad de actualización extendida (EUS) están disponibles para aquellos clientes que es posible que quieran bloquear sus máquinas virtuales de RHEL en una determinada versión inferior de RHEL después de aprovisionar la máquina virtual. Puede bloquear la versión de una máquina virtual de RHEL a una determinada versión secundaria mediante la actualización de los repositorios para que apunten a los repositorios de Extended Update Support. También puede deshacer la operación de bloqueo de versión de EUS.

>[!NOTE]
> EUS no es compatible con RHEL Extras. Esto significa que si va a instalar un paquete que está disponible normalmente en el canal de RHEL Extras, no podrá hacerlo mientras se encuentre en EUS. [Aquí](https://access.redhat.com/support/policy/updates/extras/) se detalla el ciclo de vida del producto Red Hat Extras.

En el momento de redactar este artículo, ha finalizado el soporte técnico de EUS para RHEL < = 7.4. Para más información, consulte la sección "Mantenimiento extendido de Red Hat Enterprise Linux" en la [documentación de Red Hat](https://access.redhat.com/support/policy/updates/errata/#Long_Support).
* El soporte técnico de RHEL 7.4 EUS finaliza el 31 de agosto de 2019
* El soporte técnico de RHEL 7.5 EUS finaliza el 30 de abril de 2020
* El soporte técnico de RHEL 7.6 EUS finaliza el 31 de mayo de 2021
* El soporte técnico de RHEL 7.7 EUS finaliza el 30 de agosto de 2021

### <a name="switch-a-rhel-vm-to-eus-version-lock-to-a-specific-minor-version"></a>Cambio de una máquina virtual RHEL a EUS (bloqueo de versión en una versión menor específica)
Use las instrucciones siguientes para bloquear una máquina virtual RHEL a una determinada versión secundaria (ejecutar como raíz):

>[!NOTE]
> Esto solo se aplica para las versiones de RHEL en que EUS está disponible. En el momento de redactar este artículo, esto incluye RHEL 7.2-7.7. Encontrará más detalles en la página [Red Hat Enterprise Linux Life Cycle](https://access.redhat.com/support/policy/updates/errata) (Ciclo de vida de Red Hat Enterprise Linux).

1. Deshabilitar los repositorios que no sean EUS:
    ```bash
    yum --disablerepo='*' remove 'rhui-azure-rhel7'
    ```

1. Agregar repositorios EUS:
    ```bash
    yum --config='https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel7-eus.config' install 'rhui-azure-rhel7-eus'
    ```

1. Bloquee la variable `releasever` (ejecutar como raíz):
    ```bash
    echo $(. /etc/os-release && echo $VERSION_ID) > /etc/yum/vars/releasever
    ```

    >[!NOTE]
    > La instrucción anterior bloqueará la versión secundaria de RHEL a la versión secundaria actual. Especifique una determinada versión secundaria si quiere actualizar y bloquee en una versión secundaria posterior que no sea la más reciente. Por ejemplo, `echo 7.5 > /etc/yum/vars/releasever` bloqueará la versión de RHEL en RHEL 7.5

1. Actualización de la máquina virtual de RHEL
    ```bash
    sudo yum update
    ```

### <a name="switch-a-rhel-vm-back-to-non-eus-remove-a-version-lock"></a>Cambiar una máquina virtual RHEL a no EUS (quitar un bloqueo de versión)
Ejecute lo siguiente comando como raíz:
1. Quite el archivo `releasever`:
    ```bash
    rm /etc/yum/vars/releasever
     ```

1. Deshabilitar los repositorios EUS:
    ```bash
    yum --disablerepo='*' remove 'rhui-azure-rhel7-eus'
   ```

1. Configuración de la máquina virtual RHEL
    ```bash
    yum --config='https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel7.config' install 'rhui-azure-rhel7'
    ```

1. Actualización de la máquina virtual de RHEL
    ```bash
    sudo yum update
    ```

## <a name="the-ips-for-the-rhui-content-delivery-servers"></a>Las direcciones IP para los servidores de entrega de contenido de RHUI

RHUI está disponible en todas las regiones donde estén disponibles las imágenes a petición RHEL. Actualmente incluye todas las regiones públicas que aparecen en el [panel de estado de Azure](https://azure.microsoft.com/status/) y las regiones del US Gov de Azure y regiones de Microsoft Azure Alemania.

Si va a usar la configuración de red para restringir aún más el acceso desde máquinas virtuales RHEL PAYG, asegúrese de que se permiten las siguientes direcciones IP para `yum update` con el fin de trabajar según el entorno en el que se encuentre:


```
# Azure Global
13.91.47.76
40.85.190.91
52.187.75.218
52.174.163.213
52.237.203.198

# Azure US Government
13.72.186.193
13.72.14.155
52.244.249.194

# Azure Germany
51.5.243.77
51.4.228.145
```
>[!NOTE]
>Las nuevas imágenes de Azure US Government, a partir de enero de 2020, utilizarán la dirección IP pública mencionada en el encabezado global de Azure anterior.

>[!NOTE]
>Además, tenga en cuenta que Azure Alemania está en desuso en favor de las regiones de Alemania públicas. Una recomendación para los clientes de Azure Alemania es empezar a apuntar a RHUI público mediante los pasos descritos [aquí](#manual-update-procedure-to-use-the-azure-rhui-servers).

## <a name="azure-rhui-infrastructure"></a>Infraestructura de RHUI en Azure


### <a name="update-expired-rhui-client-certificate-on-a-vm"></a>Actualización del certificado de cliente de RHUI expirado en una máquina virtual

Si usa una imagen de máquina virtual de RHEL anterior, por ejemplo, RHEL 7.4 (URN de imagen: `RedHat:RHEL:7.4:7.4.2018010506`), experimentará problemas de conectividad a RHUI debido a un certificado de cliente TLS/SSL ahora expirado. El error que vea puede ser similar a _"SSL del mismo nivel rechazó el certificado como expirado"_ o _Error: No se puede recuperar el repositorio de metadatos (repomd.xml) para el repositorio:... Compruebe su ruta de acceso y vuelva a intentarlo"_ . Para solucionar este problema, actualice el paquete de cliente de RHUI en la máquina virtual con el siguiente comando:

```bash
sudo yum update -y --disablerepo='*' --enablerepo='*microsoft*'
```

Como alternativa, al ejecutar `sudo yum update` también se actualiza el paquete del certificado de cliente (según la versión de RHEL) a pesar de los errores de "certificado SSL expirado" que verá para otros repositorios. Si esta actualización es correcta, se debería restaurar la conectividad normal a otros repositorios de RHUI, de modo que podrá ejecutar `sudo yum update` correctamente.

Si aparece el error 404 al ejecutar `yum update`, pruebe lo siguiente para actualizar la caché de yum:
```bash
sudo yum clean all;
sudo yum makecache
```

### <a name="troubleshoot-connection-problems-to-azure-rhui"></a>Solución de problemas de conexión a RHUI de Azure
Si experimenta problemas al conectarse a RHUI de Azure desde la máquina virtual de Azure RHEL PAYG, siga estos pasos:

1. Inspeccione la configuración de máquina virtual del punto de conexión de RHUI de Azure:

    1. Compruebe si el archivo `/etc/yum.repos.d/rh-cloud.repo` contiene referencias a `rhui-[1-3].microsoft.com` en `baseurl` de la sección `[rhui-microsoft-azure-rhel*]` del archivo. Si las tiene, significa que está usando la nueva RHUI de Azure.

    1. Si apunta a una ubicación con el siguiente patrón `mirrorlist.*cds[1-4].cloudapp.net`, es necesario actualizar la configuración. Está usando la instantánea de máquina virtual anterior y tiene que actualizarla para dirigirla al nuevo RHUI de Azure.

1. El acceso a la RHUI hospedada en Azure se limita a las máquinas virtuales dentro de los [intervalos de direcciones IP del centro de datos de Azure](https://www.microsoft.com/download/details.aspx?id=41653).

1. Si está usando la configuración nueva y comprobó que la máquina virtual se conecta desde el intervalo IP de Azure pero sigue sin poder conectarse a RHUI de Azure, presente una incidencia de soporte técnico a Microsoft o Red Hat.

### <a name="infrastructure-update"></a>Actualización de infraestructura

En septiembre de 2016, implementamos una RHUI de Azure actualizada. En abril de 2017, antigua RHUI de Azure dejó de prestar servicio. Si ha usado las imágenes de PAYG de RHEL (o sus instantáneas) desde septiembre de 2016 o después, se conecta automáticamente al nuevo RHUI de Azure. Sin embargo, si instantáneas anteriores en sus máquinas virtuales, debe actualizar manualmente su configuración para acceder a la RHUI de Azure tal como se describe en la siguiente sección.

Los nuevos servidores RHUI de Azure se implementan con [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/). En Traffic Manager, una máquina virtual puede usar cualquier punto de conexión único (rhui-1.micrsoft.com), independientemente de la región.

### <a name="manual-update-procedure-to-use-the-azure-rhui-servers"></a>Procedimiento de actualización manual para usar los servidores de RHUI de Azure
Este procedimiento se proporciona solo como referencia. Las imágenes de RHEL PAYG ya tienen la configuración correcta para conectarse a la RHUI de Azure. Para actualizar manualmente la configuración para usar los servidores de RHUI de Azure, complete los pasos siguientes:

- Para RHEL 6:
  ```bash
  yum --config='https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel6.config' install 'rhui-azure-rhel6'
  ```

- Para RHEL 7:
  ```bash
  yum --config='https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel7.config' install 'rhui-azure-rhel7'
  ```

- Para RHEL 8:
    1. Cree un archivo de configuración:
        ```bash
        vi rhel8.config
        ```
    1. Agregue el siguiente contenido al archivo de configuración:
        ```bash
        [rhui-microsoft-azure-rhel8]
        name=Microsoft Azure RPMs for Red Hat Enterprise Linux 8
        baseurl=https://rhui-1.microsoft.com/pulp/repos/microsoft-azure-rhel8 https://rhui-2.microsoft.com/pulp/repos/microsoft-azure-rhel8 https://rhui-3.microsoft.com/pulp/repos/microsoft-azure-rhel8
        enabled=1
        gpgcheck=1
        gpgkey=https://rhelimage.blob.core.windows.net/repositories/RPM-GPG-KEY-microsoft-azure-release sslverify=1
        ```
    1. Guarde el archivo y ejecute el comando siguiente:
        ```bash
        dnf --config rhel8.config install 'rhui-azure-rhel8'
        ```
    1. Actualización de la máquina virtual
        ```bash
        sudo dnf update
        ```


## <a name="next-steps"></a>Pasos siguientes
* Para crear una máquina virtual Linux de Red Hat Enterprise a partir de la imagen de pago por uso de Azure Marketplace y usar la RHUI hospedada en Azure, vaya a [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/RedHat.RHEL_6).
* Para obtener más información acerca de las imágenes de Red Hat en Azure, consulte la [página de documentación](./redhat-images.md).
* Puede encontrar información sobre las directivas de soporte técnico de Red Hat para todas las versiones de RHEL en la página [Red Hat Enterprise Linux Life Cycle](https://access.redhat.com/support/policy/updates/errata) (Ciclo de vida de Red Hat Enterprise Linux).
