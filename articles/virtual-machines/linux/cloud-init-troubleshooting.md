---
title: Solución de problemas con Cloud-init
description: Solución de problemas de aprovisionamiento de una VM de Azure con cloud-init.
author: danielsollondon
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: troubleshooting
ms.date: 07/06/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: 6412036e3f16e2efb3bbf6669f6a31e9dc6e3584
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89434646"
---
# <a name="troubleshooting-vm-provisioning-with-cloud-init"></a>Solución de problemas de aprovisionamiento de VM con cloud-init

Si ha creado imágenes personalizadas generalizadas con cloud-init para el aprovisionamiento, pero ha descubierto que la VM no se creó correctamente, deberá solucionar los problemas de las imágenes personalizadas.

Algunos ejemplos de problemas con el aprovisionamiento:
- La VM se bloquea en la "creación" durante 40 minutos y este proceso devuelve errores.
- `CustomData` no se procesa
- No se puede montar el disco efímero.
- Los usuarios no se crean o hay problemas de acceso del usuario.
- Las redes no están configuradas correctamente
- Errores de archivo de intercambio o partición

En este artículo se explica cómo solucionar problemas de cloud-init. Para obtener más información, consulte [análisis detallado de cloud-init](./cloud-init-deep-dive.md).

## <a name="step-1-test-the-deployment-without-customdata"></a>Paso 1: Prueba de la implementación sin `customData`

Cuando se crea la máquina virtual, cloud-init puede aceptar `customData`, que se le pasa. En primer lugar, debe asegurarse de que esto no cause ningún problema con las implementaciones. Intente aprovisionar la VM sin pasar ninguna configuración. Si se produce un error al aprovisionar la VM, continúe con los pasos siguientes. Si la configuración que está pasando no se está aplicando, vaya al [paso 4](). 

## <a name="step-2-review-image-requirements"></a>Paso 2: Revisión de los requisitos de imagen
La causa principal del error de aprovisionamiento de la VM es que la imagen del sistema operativo no cumple los requisitos previos para ejecutarse en Azure. Asegúrese de que las imágenes estén preparadas correctamente antes de intentar aprovisionarlas en Azure. 


En los artículos siguientes se muestran los pasos para preparar las distintas distribuciones de Linux admitidas en Azure:

- [Distribuciones basadas en CentOS](create-upload-centos.md)
- [Debian Linux](debian-create-upload-vhd.md)
- [Flatcar Container Linux](flatcar-create-upload-vhd.md)
- [Oracle Linux](oracle-create-upload-vhd.md)
- [Red Hat Enterprise Linux](redhat-create-upload-vhd.md)
- [SLES y openSUSE](suse-create-upload-vhd.md)
- [Ubuntu](create-upload-ubuntu.md)
- [Otras: distribuciones no aprobadas](create-upload-generic.md)

En el caso de las [imágenes de cloud-init de Azure admitidas](./using-cloud-init.md), las distribuciones de Linux ya tienen todos los paquetes y configuraciones necesarios para aprovisionar correctamente la imagen en Azure. Si la máquina virtual no se puede crear a partir de su propia imagen creada, pruebe una imagen de Azure Marketplace compatible que ya esté configurada para cloud-init, con el valor de `customData` opcional. Si `customData` funciona correctamente con una imagen de Azure Marketplace, probablemente haya un problema con la imagen creada.

## <a name="step-3-collect--review-vm-logs"></a>Paso 3: Recopilar y revisar los registros de VM

Cuando no se puede aprovisionar la VM, Azure mostrará el estado "creando", durante 20 minutos y, a continuación, reiniciará la VM, espere unos 20 minutos antes de marcar finalmente la implementación de la VM como con errores antes de marcarla con un error de `OSProvisioningTimedOut`.

Mientras se ejecuta la VM, necesitará los registros de la VM para comprender por qué se produjo un error en el aprovisionamiento.  Para comprender por qué se produjo un error en el aprovisionamiento de la VM, no detenga la VM. Mantenga la VM en ejecución. Tendrá que mantener la VM con errores en ejecución para recopilar registros. Para recopilar los registros, use uno de los siguientes métodos:

- [Consola serie](../troubleshooting/serial-console-grub-single-user-mode.md)

- [Habilitar los diagnósticos de arranque](./tutorial-monitor.md#enable-boot-diagnostics) antes de crear la VM y, a continuación, [verlos](./tutorial-monitor.md#view-boot-diagnostics) durante el arranque.

- [Ejecute reparación de máquinas virtuales de AZ](../troubleshooting/repair-linux-vm-using-azure-virtual-machine-repair-commands.md) para conectar y montar el disco del sistema operativo, lo que le permitirá recopilar estos registros:
```bash
/var/log/cloud-init*
/var/log/waagent*
/var/log/syslog*
/var/log/rsyslog*
/var/log/messages*
/var/log/kern*
/var/log/dmesg*
/var/log/boot*
```
Para empezar a solucionar problemas iniciales, empiece con los registros de cloud-init y comprenda dónde se produjo el error, use los otros registros para profundizar y proporcione información adicional. 
* /var/log/cloud-init.log
* /var/log/cloud-init-output.log
* Registros de serie/arranque

En todos los registros, inicie la búsqueda de "Failed", "WARNING", "WARN", "err", "error", "ERROR" Se recomienda establecer la configuración para omitir las búsquedas con distinción de mayúsculas y minúsculas. 

> [!TIP]
> Si va a solucionar problemas de una imagen personalizada, considere la posibilidad de agregar un usuario durante la imagen. Si el aprovisionamiento no puede establecer el usuario administrador, todavía puede iniciar sesión en el sistema operativo.

## <a name="analyzing-the-logs"></a>Analizar los registros

Aquí encontrará más detalles sobre lo que debe buscar en cada registro de cloud-init.

### <a name="varlogcloud-initlog"></a>/var/log/cloud-init.log

De forma predeterminada, todos los eventos de cloud-init con una prioridad de depuración o superior se escriben en `/var/log/cloud-init.log`. Esto proporciona registros detallados de cada evento que se produjo durante la inicialización de cloud-init. 

Por ejemplo:

```console
2019-10-10 04:51:25,321 - util.py[DEBUG]: Failed mount of '/dev/sr0' as 'auto': Unexpected error while running command.
Command: ['mount', '-o', 'ro,sync', '-t', 'auto', u'/dev/sr0', '/run/cloud-init/tmp/tmpLIrklc']
Exit code: 32
Reason: -
Stdout:
Stderr: mount: unknown filesystem type 'udf'
2020-01-31 00:21:53,352 - DataSourceAzure.py[WARNING]: /dev/sr0 was not mountable
```


Una vez que encuentre un error o una advertencia, lea en el registro de cloud-init anterior para saber qué inicio de sesión en cloud-init estaba intentando antes de encontrar el error o la advertencia. En muchos casos, cloud-init habrá ejecutado comandos del sistema operativo o realizado operaciones de aprovisionamiento antes del error, lo que puede proporcionar información sobre por qué los errores aparecían en los registros. El siguiente ejemplo muestra que cloud Init intentó montar un dispositivo justo antes de que se produjera un error.

```output
2019-10-10 04:51:24,010 - util.py[DEBUG]: Running command ['mount', '-o', 'ro,sync', '-t', 'auto', u'/dev/sr0', '/run/cloud-init/tmp/tmpXXXXX'] with allowed return codes [0] (shell=False, capture=True)
```

Si tiene acceso a la [consola serie](../troubleshooting/serial-console-grub-single-user-mode.md), puede intentar volver a ejecutar el comando que cloud-init estaba intentando ejecutar.

El registro de `/var/log/cloud-init.log` también se puede reconfigurar desde /etc/cloud/cloud.cfg.d/05_logging.cfg. Para más detalles sobre el registro de cloud-init, consulte la [documentación de cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/logging.html). 

### <a name="varlogcloud-init-outputlog"></a>/var/log/cloud-init-output.log

Puede obtener información de `stdout` y `stderr` durante las [fases de cloud-init](cloud-init-deep-dive.md). Normalmente, esto implica la información de la tabla de enrutamiento, la información de red, la información de comprobación de la clave de host de SSH, `stdout` y `stderr` para cada fase de cloud-init, junto con la marca de tiempo para cada fase. Si lo desea, el registro de `stderr` y `stdout` se puede volver a configurar desde `/etc/cloud/cloud.cfg.d/05_logging.cfg`.

### <a name="serialboot-logs"></a>Registros de serie/arranque 

Cloud-init tiene varias dependencias. Estas se documentan en los requisitos previos necesarios para las imágenes en Azure, como las redes, el almacenamiento, la capacidad para montar una imagen ISO y el montaje y formateo del disco temporal. Cualquiera de estas puede producir errores y provocar que cloud-init tenga errores. Por ejemplo, si la VM no puede obtener una concesión DHCP, se producirá un error en cloud-init.

Si todavía no puede aislar por qué Cloud-init no se ha aprovisionado, debe comprender las fases de cloud-init y cuándo se ejecutan los módulos. Consulte [Profundizar más en Cloud-init](cloud-init-deep-dive.md) para obtener más detalles.


## <a name="step-4-investigate-why-the-configuration-isnt-being-applied"></a>Paso 4: Investigue por qué no se aplica la configuración
No todos los errores de cloud-init provocan un error de aprovisionamiento irrecuperable. Por ejemplo, si usa el módulo `runcmd` en una configuración de cloud-init, un código de salida distinto de cero del comando que se está ejecutando hará que se produzca un error en el aprovisionamiento de la VM. Esto se debe a que se ejecuta después de la funcionalidad de aprovisionamiento básica que ocurre en las 3 primeras fases de cloud-init. Para solucionar el motivo por el que no se aplicó la configuración, revise los registros en el paso 3 y los módulos Cloud-init manualmente. Por ejemplo:

- `runcmd`: ¿se ejecutan los scripts sin errores? Ejecute la configuración manualmente desde la terminal para asegurarse de que se ejecuta según lo previsto.
- Instalación de paquetes: ¿la VM tiene acceso a los repositorios de paquetes?
- También debe comprobar la configuración de datos `customData` que se proporcionó a la VM, que se encuentra en `/var/lib/cloud/instances/<unique-instance-identifier>/user-data.txt`.


## <a name="next-steps"></a>Pasos siguientes

Si todavía no puede aislar por qué cloud-init no ejecutó la configuración, debe examinar más detenidamente lo que sucede en cada fase de cloud-init y cuándo se ejecutan los módulos. Consulte [profundizar en la configuración de Cloud-init](./cloud-init-deep-dive.md) para obtener más información. 
