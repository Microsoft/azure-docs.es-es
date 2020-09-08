---
title: 'Certificación de máquinas virtuales: problemas y soluciones'
description: En este artículo se explican los mensajes de error comunes de las imágenes de máquina virtual. También se describen las soluciones relacionadas.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: troubleshooting
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 06/16/2020
ms.openlocfilehash: 5878ea6a554439c261399706eec708b06ed59b11
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88225395"
---
# <a name="issues-and-solutions-during-virtual-machine-certification"></a>Problemas y soluciones durante la certificación de máquinas virtuales 

Cuando publica sus imágenes de máquina virtual en Azure Marketplace, el equipo de Azure la valida para garantizar su arranque, seguridad y compatibilidad con Azure. Si se produce un error en alguna de las pruebas de alta calidad, se producirá un error en la publicación y recibirá un mensaje de error que describe el problema.

En este artículo se explican los mensajes de error comunes durante la publicación de imágenes de máquina virtual, junto con soluciones relacionadas.

> [!NOTE]
> Si tiene preguntas o comentarios sobre mejoras, póngase en contacto con el [soporte técnico del Centro de partners](https://partner.microsoft.com/support/v2/?stage=1).

## <a name="approved-base-image"></a>Imagen base aprobada

Cuando envía una solicitud para volver a publicar su imagen con actualizaciones, el caso de prueba de comprobación del número de pieza puede fallar. Si se produce un error, la imagen no se aprobará.

Este error se produce al utilizar una imagen base que pertenece a otro publicador y ha actualizado la imagen. En esta situación, no se le permitirá publicar la imagen.

Para corregir el problema, recupere la imagen de Azure Marketplace y haga cambios en ella. Para más información, consulte los siguientes artículos.

- [Imágenes de Linux](../../virtual-machines/linux/endorsed-distros.md?toc=/azure/virtual-machines/linux/toc.json)
- [Imágenes de Windows](create-azure-vm-technical-asset.md#create-a-vm-image-using-an-approved-base)

## <a name="vm-extension-failure"></a>Error de la extensión de máquina virtual

Compruebe si la imagen es compatible con las extensiones de máquina virtual.

Para habilitar las extensiones de máquina virtual, haga lo siguiente:

1. Seleccione la máquina virtual Linux.
1. Vaya a **Configuración de diagnóstico**.
1. Habilite las matrices base mediante la actualización de la **cuenta de almacenamiento**.
1. Seleccione **Guardar**.

   ![Habilitación de la supervisión a nivel de invitado](./media/vm-certification-issues-solutions-1.png)

Para comprobar que las extensiones de máquina virtual están activadas correctamente, haga lo siguiente:

1. Vaya a la pestaña **Extensiones de máquina virtual** y, a continuación, verifique el estado de la **Extensión de diagnósticos de Linux**.
    * Si el estado es *Aprovisionamiento realizado correctamente*, el caso de prueba de las extensiones se ha realizado correctamente.  
    * Si el estado es *Error de aprovisionamiento*, el caso de prueba de extensiones ha dado error y es necesario configurar la marca reforzada.

      ![Captura de pantalla que muestra que el aprovisionamiento se ha realizado correctamente](./media/vm-certification-issues-solutions-2.png)

      Si se produce un error en la extensión de máquina virtual, consulte [Uso de la extensión de diagnósticos de Linux para supervisar métricas y registros](../../virtual-machines/extensions/diagnostics-linux.md) para habilitarla. Si no desea habilitar la extensión de máquina virtual, póngase en contacto con el equipo de soporte técnico y pídale que la deshabilite.

## <a name="vm-provisioning-issue"></a>Problema de aprovisionamiento de máquinas virtuales

Asegúrese de que ha seguido el proceso de aprovisionamiento de la máquina virtual rigurosamente antes de enviar la oferta. Para ver el formato JSON y aprovisionar la máquina virtual, consulte [Certificación de imágenes de máquinas virtuales de Azure](azure-vm-image-certification.md).

Entre los problemas de aprovisionamiento, se incluyen los siguientes escenarios:

|Escenario|Error|Motivo|Solución|
|---|---|---|---|
|1|Disco duro virtual (VHD) no válido:|Si el valor de cookie especificado en el pie de página del VHD no es correcto, el VHD se considerará no válido.|Vuelva a crear la imagen y envíe la solicitud.|
|2|Tipo de blob no válido|La máquina virtual no se pudo aprovisionar porque el bloque utilizado es de tipo blob en lugar de tipo página.|Vuelva a crear la imagen y envíe la solicitud.|
|3|Tiempo de espera de aprovisionamiento agotado o no generalizado correctamente|Hay un problema con la generalización de máquina virtual.|Vuelva a crear la imagen con generalización y envíe la solicitud.|

> [!NOTE]
> Para obtener más información sobre la generalización de máquinas virtuales, consulte:
> - [Documentación de Linux](create-azure-vm-technical-asset.md#generalize-the-image)
> - [Documentación de Windows](../../virtual-machines/windows/capture-image-resource.md#generalize-the-windows-vm-using-sysprep)

## <a name="software-compliance-for-windows"></a>Compatibilidad de software para Windows

Si se rechaza su solicitud de imagen de Windows debido a un problema de cumplimiento, es posible que haya creado una imagen de Windows con la instancia de SQL Server instalada, en lugar de utilizar la imagen base de la versión de SQL pertinente de Azure Marketplace.

No cree su imagen de Windows con SQL Server instalado. En lugar de ello, use las imágenes base de SQL aprobadas (Enterprise/Standard/web) de Azure Marketplace.

Si intenta instalar Visual Studio o cualquier producto con licencia de Office, póngase en contacto con el equipo de soporte técnico para obtener aprobación previamente.

Para obtener más información sobre seleccionar una base aprobada, consulte [Creación de recursos técnicos para máquinas virtuales de Azure](create-azure-vm-technical-asset.md#create-a-vm-image-using-an-approved-base).

## <a name="tool-kit-test-case-execution-failed"></a>Error de ejecución del caso de prueba del kit de herramientas

El kit de herramientas de certificación de Microsoft le puede ayudarle a ejecutar casos de prueba y comprobar que su disco duro virtual o imagen es compatible con el entorno de Azure.

Descargue el [kit de herramientas de certificación de Microsoft](azure-vm-image-certification.md).

## <a name="linux-test-cases"></a>Casos de prueba de Linux

En la tabla siguiente se enumeran los casos de prueba de Linux que ejecutará el kit de herramientas. La validación de prueba se indica en la descripción.

|Escenario|Caso de prueba|Descripción|
|---|---|---|
|1|Historial de Bash|Los archivos de historial de Bash deben borrarse antes de que cree la imagen de máquina virtual.|
|2|Versión del agente Linux|Se debe instalar el agente Linux de Azure 2.2.41 o más reciente.|
|3|Parámetros de kernel obligatorios|Comprueba que se han configurado los siguientes parámetros de kernel: <br>console=ttyS0<br>earlyprintk=ttyS0<br>rootdelay=300|
|4|Partición de intercambio en el disco del sistema operativo|Comprueba que no se crean particiones de intercambio en el disco del sistema operativo.|
|5|Partición raíz en el disco del sistema operativo|Crear una sola partición raíz para el disco de SO.|
|6|Versión de OpenSSL|La versión de OpenSSL debe ser v0.9.8 o más reciente.|
|7|Versión de Python|Se recomienda usar la versión de Python 2.6 o más reciente.|
|8|ClientAliveInterval|Establece ClientAliveInterval en 180. En cuanto al requisito de la aplicación, se puede establecer de 30 a 235. Si va a habilitar SSH para sus usuarios finales, este valor debe establecerse tal y como se ha explicado.|
|9|Arquitectura del SO|Solo se admiten los sistemas operativos de 64 bits.|
|10|Actualización automática|Determina si la actualización automática del agente Linux está habilitada.|

### <a name="common-errors-found-while-executing-previous-test-cases"></a>Errores comunes encontrados al ejecutar los casos de prueba anteriores.

En la tabla siguiente se enumeran los errores comunes que se encuentran al ejecutar casos de prueba anteriores:
 
|Escenario|Caso de prueba|Error|Solución|
|---|---|---|---|
|1|Caso de prueba de la versión del agente Linux|La versión mínima del agente Linux es 2.241 o más reciente. Este requisito es obligatorio desde el 1 de mayo de 2020.|La imagen debe actualizarse con la versión necesaria para [enviar la solicitud](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).|
|2|Caso de prueba del historial de Bash|Si el tamaño del historial de Bash de la imagen enviada es superior a 1 kilobyte (KB), se mostrará un error. El tamaño está limitado a 1 KB para garantizar que cualquier información potencialmente sensible no sea capturada en el archivo de historial de Bash.|Para solucionar este problema, monte el disco duro virtual en cualquier otra máquina virtual en funcionamiento y haga los cambios que desee (por ejemplo, eliminar los archivos de historial *.bash*) para reducir el tamaño a 1 KB o menos.|
|3|Caso de prueba del parámetro de kernel obligatorio|Recibirá este error si el valor de **console** no se ha establecido en **ttyS0**. Haga una comprobación al ejecutar el comando siguiente:<br>`cat /proc/cmdline`|Establezca el valor de **console** en **ttyS0** y vuelva a enviar la solicitud.|
|4|Caso de prueba de ClientAliveInterval|Si el resultado del kit de herramientas genera error para este caso de prueba, el valor de **ClientAliveInterval** no es adecuado.|Establezca **ClientAliveInterval** en un valor igual o inferior a 235 y, a continuación, vuelva a enviar la solicitud.|

### <a name="windows-test-cases"></a>Casos de prueba de Windows

En la tabla siguiente se enumeran los casos de prueba de Windows que ejecutará el kit de herramientas, junto con una descripción de la validación de prueba:

|Escenario |Casos de prueba|Descripción|
|---|---|---|---|
|1|Arquitectura del SO|Azure solo admite los sistemas operativos de 64 bits.|
|2|Dependencia de la cuenta de usuario|La ejecución de aplicaciones no debería depender de la cuenta de administrador.|
|3|Clúster de conmutación por error|La característica de clústeres de conmutación por error de Windows Server aún no se admite. La aplicación no debería depender de esta característica.|
|4|IPV6|IPv6 aún no se admite en el entorno de Azure. La aplicación no debería depender de esta característica.|
|5|DHCP|El rol de servidor del protocolo de configuración dinámica de host aún no se admite. La aplicación no debería depender de esta característica.|
|6|Hyper-V|El rol de servidor de Hyper-V aún no se admite. La aplicación no debería depender de esta característica.|
|7|Acceso remoto|El rol de servidor de acceso remoto (acceso directo) aún no se admite. La aplicación no debería depender de esta característica.|
|8|Rights Management Services|Rights Management Services. El rol de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|9|Servicios de implementación de Windows|Servicios de implementación de Windows. El rol de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|10|Cifrado BitLocker de unidades|El Cifrado de unidad BitLocker no se admite en el disco duro del sistema operativo, pero se podría utilizar en discos de datos.|
|11|Servidor de nombres de almacenamiento de Internet|La característica Servidor de nombres de almacenamiento de Internet aún no se admite. La aplicación no debería depender de esta característica.|
|12|E/S de múltiples rutas|E/S de múltiples rutas. Esta característica de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|13|Equilibrio de carga de red|Equilibrio de carga de red. Esta característica de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|14|Protocolo de resolución de nombres de mismo nivel|Protocolo de resolución de nombres de mismo nivel. Esta característica de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|15|Servicios SNMP|Todavía no se admite la característica servicios de Protocolo simple de administración de redes (SNMP). La aplicación no debería depender de esta característica.|
|16|Servicio de nombres Internet de Windows|Servicio de nombres Internet de Windows. Esta característica de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|17|Servicio WLAN|Servicio WLAN. Esta característica de servidor aún no se admite. La aplicación no debería depender de esta característica.|

Si se producen errores con los casos de prueba mencionados, consulte la columna de **Descripción** en la tabla para ver la solución. Para más información, póngase en contacto con el equipo de soporte técnico. 

## <a name="data-disk-size-verification"></a>Comprobación del tamaño del disco de datos

Si el tamaño de cualquier solicitud enviada con el disco de datos es superior a 1023 gigabytes (GB), la solicitud no se aprobará. Esta regla se aplica a Windows y Linux.

Vuelva a enviar la solicitud con un tamaño que sea inferior o igual a 1023 GB.

## <a name="os-disk-size-validation"></a>Validación del tamaño del disco del sistema operativo

Consulte las siguientes reglas para conocer las limitaciones del tamaño del disco del sistema operativo. Al enviar una solicitud, compruebe que el tamaño del disco del sistema operativo cumple el requisito para Linux o Windows.

|SO|Tamaño recomendado del disco duro virtual|
|---|---|
|Linux|De 30 a 1023 GB|
|Windows|De 30 a 250 GB|

Como las máquinas virtuales permiten acceder al sistema operativo subyacente, asegúrese de que el tamaño del disco duro virtual sea lo suficientemente grande. Dado que los discos no se pueden expandir sin que haya tiempo de inactividad, use un tamaño de disco de 30 a 50 GB.

|Tamaño de VHD|Tamaño real ocupado|Solución|
|---|---|---|
|>500 tebibytes (TiB)|N/D|Póngase en contacto con el equipo de soporte técnico para obtener una aprobación extraordinaria.|
|De 250 a 500 TiB|diferencia de > 200 gibibytes (GiB) del tamaño del blob|Póngase en contacto con el equipo de soporte técnico para obtener una aprobación extraordinaria.|

> [!NOTE]
> Los discos de mayor tamaño tienen costos más elevados y provocarán un retraso durante los pasos de configuración y replicación. Debido a este retraso y al costo, el equipo de soporte técnico podría solicitar una justificación para la aprobación extraordinaria.

## <a name="wannacry-patch-verification-test-for-windows"></a>Prueba de comprobación de la revisión de WannaCry para Windows

Para evitar un posible ataque relacionado con el virus WannaCry, asegúrese de que todas las solicitudes de imagen de Windows estén actualizadas con la revisión más reciente.

Para comprobar la versión de revisión aplicada a Windows Server, consulte la tabla siguiente con los detalles del sistema operativo y la versión mínima admitida. 

La versión del archivo de imagen se puede comprobar en `C:\windows\system32\drivers\srv.sys` o `srv2.sys`.

> [!NOTE]
> WindowsServer2019 no tiene requisitos de versión obligatoria.

|SO|Versión|
|---|---|
|Windows Server 2008 R2|6.1.7601.23689|
|Windows Server 2012|6.2.9200.22099|
|Windows Server 2012 R2|6.3.9600.18604|
|Windows Server 2016|10.0.14393.953|
|Windows Server 2019|N/D|

## <a name="sack-vulnerability-patch-verification"></a>Comprobación de la revisión para la vulnerabilidad SACK

Al enviar una imagen de Linux, es posible que se rechace su solicitud debido a problemas con la versión del kernel.

Actualice el kernel con una versión aprobada y vuelva a enviar la solicitud. Puede encontrar la versión aprobada de kernel en la tabla siguiente. El número de versión debe ser igual o superior al número que se muestra a continuación.

Si su imagen no tiene instalada una de las siguientes versiones de kernel, actualícela con las revisiones correctas. Solicite la aprobación correspondiente del equipo de soporte técnico después de actualizar la imagen con estas revisiones necesarias:

- CVE-2019-11477 
- CVE-2019-11478 
- CVE-2019-11479

|Familia de SO|Versión|Kernel|
|---|---|---|
|Ubuntu|14.04 LTS|4.4.0-151| 
||14.04 LTS|4.15.0-1049-*-azure|
||16.04 LTS|4.15.0-1049|
||18.04 LTS|4.18.0-1023|
||18.04 LTS|5.0.0-1025|
||18.10|4.18.0-1023|
||19.04|5.0.0-1010|
||19.04|5.3.0-1004|
|RHEL y Cent OS|6.10|2.6.32-754.15.3|
||7.2|3.10.0-327.79.2|
||7.3|3.10.0-514.66.2|
||7.4|3.10.0-693.50.3|
||7.5|3.10.0-862.34.2|
||7.6|3.10.0-957.21.3|
||7,7|3.10.0-1062.1.1|
||8.0|4.18.0-80.4.2|
||8.1|4.18.0-147|
||"7-RAW" (7.6)||
||"7-LVM" (7.6)|3.10.0-957.21.3|
||RHEL-SAP 7.4|TBD|
||RHEL-SAP 7.5|TBD|
|SLES|SLES11SP4 (incluido SAP)|3.0.101-108.95.2|
||SLES12SP1 para SAP|3.12.74-60.64.115.1|
||SLES12SP2 para SAP|4.4.121-92.114.1|
||SLES12SP3|4.4180-4.31.1 (kernel-azure)|
||SLES12SP3 para SAP|4.4.180-94.97.1|
||SLES12SP4|4.12.14-6.15.2 (kernel-azure)|
||SLES12SP4 para SAP|4.12.14-95.19.1|
||SLES15|4.12.14-5.30.1 (kernel-azure)|
||SLES15 para SAP|4.12.14-5.30.1 (kernel-azure)|
||SLES15SP1|4.12.14-5.30.1 (kernel-azure)|
|Oracle|6.10|UEK2 2.6.39-400.312.2<br>UEK3 3.8.13-118.35.2<br>RHCK 2.6.32-754.15.3 
||7.0-7.5|UEK3 3.8.13-118.35.2<br>UEK4 4.1.12-124.28.3<br>RHCK sigue a RHEL (más arriba)|
||7.6|RHCK 3.10.0-957.21.3<br>UEK5 4.14.35-1902.2.0|
|CoreOS Stable 2079.6.0|4.19.43*|
||Beta 2135.3.1|4.19.50*|
||Alpha 2163.2.1|4.19.50*|
|Debian|jessie (seguridad)|3.16.68-2|
||jessie (backports)|4.9.168-1+deb9u3|
||stretch (seguridad)|4.9.168-1+deb9u3|
||Debian GNU/Linux 10 (buster)|Debian 6.3.0-18+deb9u1|
||buster, sid (stretch backports)|4.19.37-5|

## <a name="image-size-should-be-in-multiples-of-megabytes"></a>El tamaño de la imagen debe expresarse en múltiplos de megabytes

En Azure, todos los discos duros virtuales deben tener un tamaño virtual expresado en múltiplos de 1 megabyte (MB). Si el disco duro virtual no tiene el tamaño recomendado, es posible que la solicitud se rechace.

Cuando convierta un disco sin formato en un disco duro virtual siga las recomendaciones y asegúrese de que el tamaño del disco sin formato es un múltiplo de 1 MB. Para más información, consulte [Información para las distribuciones no aprobadas](../../virtual-machines/linux/create-upload-generic.md).

## <a name="vm-access-denied"></a>Acceso denegado a la máquina virtual

Si surgen problemas de acceso denegado al ejecutar los casos de prueba en la máquina virtual, podría deberse a que no haya privilegios suficientes para ejecutar los casos de prueba.

Compruebe si el acceso adecuado está habilitado para la cuenta en la que se estén ejecutando los casos de prueba automáticos. Si el acceso no está habilitado, habilítelo para ejecutar los casos de prueba. Si no desea habilitar el acceso, podría compartir los resultados de los casos de prueba automáticos con el equipo de soporte técnico.

## <a name="download-failure"></a>Error de descarga
    
Consulte la tabla siguiente para ver los problemas que surgen al descargar la imagen de máquina virtual mediante una dirección URL de firma de acceso compartido (SAS).

|Escenario|Error|Motivo|Solución|
|---|---|---|---|
|1|Blob no encontrado|El disco duro virtual podría eliminarse o moverse de la ubicación especificada.|| 
|2|Blob en uso|Otro proceso interno utiliza el disco duro virtual.|El disco duro virtual debe estar en uso cuando lo descargue mediante la dirección URL de SAS.|
|3|Dirección URL de SAS no válida|La dirección URL de SAS asociada al disco duro virtual es incorrecta.|Obtenga la dirección URL de SAS correcta.|
|4|Firma incorrecta|La dirección URL de SAS asociada al disco duro virtual es incorrecta.|Obtenga la dirección URL de SAS correcta.|
|6|Encabezado condicional HTTP|La dirección URL no es válida.|Obtenga la dirección URL de SAS correcta.|
|7|El nombre del disco duro virtual no es válido|Compruebe si hay algún carácter especial, como un signo de porcentaje (%) o comillas ("), existen en el nombre del disco duro virtual.|Cambie el nombre del archivo del disco duro virtual y quite los caracteres especiales.|

## <a name="first-1-mb-partition"></a>Primera partición de 1 MB

Al enviar el disco duro virtual, asegúrese de que la primera partición de 1 MB esté vacía. De lo contrario, se producirá un error en la solicitud.

## <a name="default-credentials"></a>Credenciales predeterminadas

Asegúrese siempre de que las credenciales predeterminadas no se envíen con el disco duro virtual. Agregar credenciales predeterminadas hace que el disco duro virtual sea más vulnerable a las amenazas de seguridad. En lugar de ello, cree sus propias credenciales al enviar el disco duro virtual.
  
## <a name="datadisk-mapped-incorrectly"></a>DataDisk se asignó incorrectamente

Cuando se envía una solicitud con varios discos de datos pero el orden no aparece en secuencia, se considera un problema de asignación. Por ejemplo, si hay tres discos de datos, el orden de numeración debe ser *0, 1, 2*. Cualquier otro orden se interpreta como un problema de asignación.

Vuelva a enviar la solicitud con una secuencia adecuada de los discos de datos.

## <a name="incorrect-os-mapping"></a>Asignación de SO incorrecta

Cuando se crea una imagen, se podría asignar a una etiqueta de sistema operativo incorrecta. Por ejemplo, cuando selecciona **Windows** como parte del nombre del sistema operativo al crear la imagen, el disco del sistema operativo debe instalarse solo con Windows. El mismo requisito se aplica a Linux.

## <a name="vm-not-generalized"></a>Máquina virtual no generalizada

Si se van a volver a usar todas las imágenes tomadas de Azure Marketplace, el disco duro virtual del sistema operativo debe generalizarse.

* En el caso de **Linux**, el siguiente proceso generaliza una máquina virtual Linux y la vuelve a implementar como una máquina virtual independiente.

  En la ventana de SSH, escriba el siguiente comando: `sudo waagent -deprovision+user`

* Para **Windows**, puede generalizar las imágenes de Windows mediante `sysreptool`.

Para obtener más información acerca de esta herramienta, consulte [Información general de la preparación del sistema (Sysprep) ]( https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

## <a name="datadisk-errors"></a>Errores de DataDisk

Consulte la tabla siguiente a fin de obtener soluciones para los errores relativos al disco de datos:

|Error|Motivo|Solución|
|---|---|---|
|`DataDisk- InvalidUrl:`|Este error puede producirse porque se ha especificado un número no válido para el número de unidad lógica (LUN) al enviar la oferta.|Compruebe que la secuencia de números de LUN del disco de datos se encuentra en el Centro de partners.|
|`DataDisk- NotFound:`|Este error podría deberse a que un disco de datos no se ha encontrado en una dirección URL de SAS especificada.|Compruebe que el disco de datos se encuentra en la dirección URL de SAS que se especificó en la solicitud.|

## <a name="remote-access-issue"></a>Problema de acceso remoto

Si la opción de Protocolo de escritorio remoto (RDP) no está habilitada para la imagen de Windows, recibirá este error. 

Habilite el acceso RDP para imágenes de Windows antes de enviarlas.

## <a name="bash-history-failed"></a>Error en el historial de Bash

Si el tamaño del historial de Bash de la imagen enviada es superior a 1 kilobyte (KB), se mostrará un error. El tamaño está limitado a 1 KB para garantizar que cualquier información potencialmente sensible no sea capturada en el archivo de historial de Bash.

A continuación se indican los pasos para eliminar el "historial de Bash".

Paso 1. Implemente la máquina virtual y haga clic en la opción "Ejecutar comando" en Azure Portal.
![Ejecutar comando en Azure Portal](./media/vm-certification-issues-solutions-3.png)

Paso 2. Seleccione la primera opción "RunShellScript" y ejecute el siguiente comando.

Comando: cat /dev/null > ~/.bash_history && history -c" ![Comando de historial de Bash en Azure Portal](./media/vm-certification-issues-solutions-4.png)

Paso 3. Después de ejecutar correctamente el comando, reinicie la máquina virtual.

Paso 4. Generalice la máquina virtual, tome la imagen VHD y detenga la máquina virtual.

Paso 5.     Vuelva a enviar la imagen generalizada.

## <a name="requesting-exceptions-custom-templates-on-vm-images-for-selective-tests"></a>Solicitud de excepciones (plantillas personalizadas) en imágenes de máquinas virtuales para pruebas selectivas

Los publicadores pueden solicitar excepciones para algunas pruebas realizadas durante la certificación de la máquina virtual. Las excepciones se proporcionan en casos muy raros cuando el publicador proporciona evidencias para admitir la solicitud.
El equipo de certificación se reserva el derecho a denegar o aprobar excepciones en cualquier momento.

En las secciones siguientes, hablaremos sobre los escenarios principales en los que se solicitan las excepciones y cómo se solicita una excepción.

Escenarios para la excepción

Hay tres escenarios/casos en los que los publicadores suelen solicitar estas excepciones. 

* **Excepción para uno o varios casos de prueba:** los publicadores pueden ponerse en contacto con el [ soporte técnico para publicadores de Marketplace](https://aka.ms/marketplacepublishersupport) para solicitar excepciones para casos de prueba. 

* **Máquinas virtuales bloqueadas/sin acceso raíz:** algunos publicadores tienen escenarios donde es necesario bloquear las máquinas virtuales, ya que tienen software como firewalls instalados en la máquina virtual. 
       En este caso, los publicadores pueden descargar la [herramienta de prueba de certificados](https://aka.ms/AzureCertificationTestTool) aquí y proporcionar el informe en el [ soporte técnico para publicadores de Marketplace](https://aka.ms/marketplacepublishersupport)


* **Plantillas personalizadas:** algunos publicadores publican imágenes de máquinas virtuales que requieren una plantilla de ARM personalizada para implementar las máquinas virtuales. En este caso, se solicita a los publicadores que proporcionen las plantillas personalizadas al [ soporte técnico para publicadores de Marketplace](https://aka.ms/marketplacepublishersupport) para que el equipo de certificación pueda usarlas para la validación. 

### <a name="information-to-provide-for-exception-scenarios"></a>Información que se debe proporcionar para escenarios de excepción

Los publicadores deben ponerse en contacto con el [soporte técnico para publicadores de Marketplace](https://aka.ms/marketplacepublishersupport) para solicitar excepciones para el escenario anterior con la información adicional siguiente:

   1.   Identificador del publicador: el identificador del publicador en el portal del Centro de partners
   2.   Identificador/nombre de oferta: el identificador o el nombre de la oferta para el que se solicita la excepción 
   3.   SKU/identificador de plan: el SKU/identificador de plan de la oferta de máquina virtual para la que se solicita la excepción
   4.    Versión: la versión de la oferta de máquina virtual para la que se solicita la excepción
   5.   Tipo de excepción: pruebas, máquina virtual bloqueada, plantillas personalizadas
   6.   Motivo de la solicitud: motivo de esta excepción e información sobre las pruebas que se van a excluir 
   7.   Datos adjuntos: posibles documentos importantes de evidencia En el caso de las máquinas virtuales bloqueadas, adjunte el informe de prueba y, para las plantillas personalizadas, proporcione la plantilla de ARM personalizada como dato adjunto. Si no adjunta el informe para las máquinas virtuales bloqueadas y la plantilla de ARM personalizada para las plantillas personalizadas se producirá una denegación de solicitud.


## <a name="next-steps"></a>Pasos siguientes

Si tiene preguntas o comentarios sobre mejoras, póngase en contacto con el [soporte técnico del Centro de partners](https://partner.microsoft.com/support/v2/?stage=1).
