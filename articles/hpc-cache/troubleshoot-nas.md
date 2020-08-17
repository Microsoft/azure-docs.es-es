---
title: Solución de problemas de destinos de almacenamiento NFS de Azure HPC Cache
description: Recomendaciones para evitar y corregir errores de configuración y otros problemas que pueden provocar errores a la hora de crear un destino de almacenamiento NFS
author: ekpgh
ms.service: hpc-cache
ms.topic: troubleshooting
ms.date: 03/18/2020
ms.author: v-erkel
ms.openlocfilehash: efa163a2c10a7dc93bf5d26865a0e7eb43f11dea
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082773"
---
# <a name="troubleshoot-nas-configuration-and-nfs-storage-target-issues"></a>Solución de problemas de configuración de NAS y destinos de almacenamiento NFS

En este artículo se proporcionan soluciones para algunos errores de configuración comunes y otros problemas que podrían evitar que Azure HPC Cache agregara un sistema de almacenamiento NFS como destino de almacenamiento.

En este artículo se incluyen detalles sobre cómo comprobar los puertos y cómo habilitar el acceso raíz a un sistema NAS. También se incluye información detallada sobre problemas menos habituales que podrían provocar errores a la hora de crear un destino de almacenamiento NFS.

> [!TIP]
> Antes de usar esta guía, lea [Requisitos previos de los destinos de almacenamiento NFS](hpc-cache-prerequisites.md#nfs-storage-requirements).

Si la solución al problema no está incluida aquí, [abra una incidencia de soporte técnico](hpc-cache-support-ticket.md) para que el servicio de soporte técnico de Microsoft pueda trabajar con usted a fin de investigar y solucionar el problema.

## <a name="check-port-settings"></a>Comprobación de configuración de puertos

Azure HPC Cache necesita acceso de lectura y escritura a varios puertos UDP/TCP en el sistema de almacenamiento de NAS de back-end. Asegúrese de que se puede acceder a estos puertos en el sistema NAS y también de que se permite el tráfico a estos puertos a través de los firewalls entre el sistema de almacenamiento y la subred de caché. Es posible que deba trabajar con los administradores de firewall y red para que el centro de datos compruebe esta configuración.

Los puertos son diferentes para los sistemas de almacenamiento de los distintos proveedores, así que compruebe los requisitos del sistema al configurar un destino de almacenamiento.

En general, la caché necesita acceso a estos puertos:

| Protocolo | Port  | Servicio  |
|----------|-------|----------|
| TCP/UDP  | 111   | rpcbind  |
| TCP/UDP  | 2049  | NFS      |
| TCP/UDP  | 4045  | nlockmgr |
| TCP/UDP  | 4046  | mountd   |
| TCP/UDP  | 4047  | status   |

Para conocer los puertos concretos necesarios para el sistema, use el siguiente comando ``rpcinfo``. Este comando indica los puertos y presenta los resultados relevantes en una tabla. (Use la dirección IP de su sistema en lugar del término *<storage_IP>* ).

Puede emitir este comando desde cualquier cliente Linux que tenga instalada infraestructura de NFS. Si usa un cliente dentro de la subred del clúster, también puede ayudar a verificar la conectividad entre la subred y el sistema de almacenamiento.

```bash
rpcinfo -p <storage_IP> |egrep "100000\s+4\s+tcp|100005\s+3\s+tcp|100003\s+3\s+tcp|100024\s+1\s+tcp|100021\s+4\s+tcp"| awk '{print $4 "/" $3 " " $5}'|column -t
```

Asegúrese de que todos los puertos devueltos por la consulta ``rpcinfo`` permitan tráfico sin restricciones desde la subred de Azure HPC Cache.

Compruebe estos valores en el propio NAS y en cualquier firewall entre el sistema de almacenamiento y la subred de caché.

## <a name="check-root-access"></a>Comprobación de acceso raíz

Azure HPC Cache necesita acceso a las exportaciones del sistema de almacenamiento para crear el destino de almacenamiento. En concreto, monta las exportaciones como el identificador de usuario 0.

Los distintos sistemas de almacenamiento usan métodos diferentes para habilitar este acceso:

* Normalmente, los servidores Linux agregan ``no_root_squash`` a la ruta de acceso exportada en ``/etc/exports``.
* Los sistemas NetApp y EMC normalmente controlan el acceso con reglas de exportación vinculadas a redes o direcciones IP específicas.

Si usa reglas de exportación, recuerde que la caché puede usar varias direcciones IP diferentes de la subred de caché. Permita el acceso desde el rango completo de posibles direcciones IP de la subred.

> [!NOTE]
> De forma predeterminada, Azure HPC Cache aplica squash al acceso a la raíz. Para obtener más información, lea [Configuración de los valores de caché adicionales](configuration.md#configure-root-squash).

Trabaje con el proveedor de almacenamiento de NAS para habilitar el nivel adecuado de acceso para la caché.

### <a name="allow-root-access-on-directory-paths"></a>Habilitación de acceso raíz en rutas de acceso de directorio
<!-- linked in prereqs article -->

En el caso de los sistemas NAS que exportan directorios jerárquicos, Azure HPC Cache necesita acceso raíz a cada nivel de exportación.

Por ejemplo, un sistema podría mostrar tres exportaciones como estas:

* ``/ifs``
* ``/ifs/accounting``
* ``/ifs/accounting/payroll``

La exportación ``/ifs/accounting/payroll`` es un elemento secundario de ``/ifs/accounting``, y ``/ifs/accounting``, a su vez, es un elemento secundario de ``/ifs``.

Si agrega la exportación ``payroll`` como destino de almacenamiento de HPC Cache, la caché monta realmente ``/ifs/`` y accede al directorio payroll directamente desde allí. Por lo tanto, Azure HPC Cache necesita acceso raíz a ``/ifs`` para acceder a la exportación ``/ifs/accounting/payroll``.

Este requisito está relacionado con la forma en que la caché indexa archivos y evita colisiones de archivos mediante identificadores de archivo que proporciona el sistema de almacenamiento.

Un sistema NAS con exportaciones jerárquicas puede proporcionar diferentes identificadores de archivo para el mismo archivo si este se recupera de distintas exportaciones. Por ejemplo, un cliente podría montar ``/ifs/accounting`` y acceder al archivo ``payroll/2011.txt``. Otro cliente monta ``/ifs/accounting/payroll`` y accede al archivo ``2011.txt``. En función de cómo asigne identificadores de archivo el sistema de almacenamiento, es posible que estos dos clientes reciban el mismo archivo con diferentes identificadores de archivo (uno para ``<mount2>/payroll/2011.txt`` y otro para ``<mount3>/2011.txt``).

El sistema de almacenamiento de back-end mantiene los alias internos de los identificadores de archivo, pero Azure HPC Cache no puede indicar qué identificadores de archivo de su índice hacen referencia al mismo elemento. Por lo tanto, es posible que la caché tenga diferentes escrituras almacenadas en caché del mismo archivo y aplique los cambios de forma incorrecta porque no sabe que se trata del mismo archivo.

Para evitar esta posible colisión de archivos entre archivos de varias exportaciones, Azure HPC Cache monta automáticamente la exportación más superficial disponible en la ruta de acceso (``/ifs`` en el ejemplo) y usa el identificador de archivo proporcionado por esa exportación. Si varias exportaciones usan la misma ruta de acceso base, Azure HPC Cache necesita acceso raíz a esa ruta.

## <a name="enable-export-listing"></a>Habilitación de listas de exportaciones
<!-- link in prereqs article -->

NAS debe indicar sus exportaciones cuando Azure HPC Cache realice una consulta.

En la mayoría de los sistemas de almacenamiento NFS, puede probar esto mediante el envío de la siguiente consulta desde un cliente Linux: ``showmount -e <storage IP address>``

Si es posible, use un cliente Linux de la misma red virtual que la caché.

Si ese comando no muestra las exportaciones, la caché va a tener problemas para conectarse al sistema de almacenamiento. Trabaje con el proveedor de NAS para habilitar las listas de exportaciones.

## <a name="adjust-vpn-packet-size-restrictions"></a>Ajuste de restricciones de tamaño de paquete de VPN
<!-- link in prereqs article and configuration article -->

Si tiene una VPN entre la caché y el dispositivo NAS, la VPN podría bloquear los paquetes de Ethernet de 1500 bytes de tamaño completo. Podría tener este problema si no se completan los intercambios grandes entre NAS y la instancia de Azure HPC Cache, pero las actualizaciones más pequeñas funcionan según lo previsto.

No hay una manera sencilla de saber si el sistema tiene este problema o no, a menos que conozca los detalles de configuración de la VPN. Estos son algunos métodos que pueden ayudarle a detectar este problema.

* Use rastreadores de paquetes en ambos lados de la VPN para detectar qué paquetes se transfieren correctamente.
* Si la VPN permite comandos ping, puede probar el envío de un paquete de tamaño completo.

  Ejecute un comando ping a través de la VPN a NAS con estas opciones. (Use la dirección IP del sistema de almacenamiento en lugar del valor *<storage_IP>* ).

   ```bash
   ping -M do -s 1472 -c 1 <storage_IP>
   ```

  Estas son las opciones del comando:

  * ``-M do``: No fragmentar
  * ``-c 1``: Enviar solo un paquete
  * ``-s 1472``: Establecer el tamaño de la carga en 1472 bytes. Esta es la carga de tamaño máxima de un paquete de 1500 bytes después de tener en cuenta la sobrecarga de Ethernet.

  Una respuesta correcta tiene el siguiente aspecto:

  ```bash
  PING 10.54.54.11 (10.54.54.11) 1472(1500) bytes of data.
  1480 bytes from 10.54.54.11: icmp_seq=1 ttl=64 time=2.06 ms
  ```

  Si se produce un error de ping con 1472 bytes, es probable que se trate de un problema de tamaño del paquete.

Para corregir el problema, es posible que tenga que configurar el bloqueo de MSS en la VPN para que el sistema remoto detecte correctamente el tamaño máximo del marco. Lea la [documentación de parámetros de IPsec/IKE de VPN Gateway](../vpn-gateway/vpn-gateway-about-vpn-devices.md#ipsec) para obtener más información.

En algunos casos, puede resultar útil cambiar la configuración de MTU para Azure HPC Cache a 1400. Sin embargo, si restringe la MTU en la memoria caché, también debe restringir la configuración de MTU para los clientes y sistemas de almacenamiento de back-end que interactúan con la memoria caché. Para obtener más información, lea [Configuración de valores adicionales de Azure HPC Cache](configuration.md#adjust-mtu-value).

## <a name="check-for-acl-security-style"></a>Comprobación del estilo de seguridad de ACL

Algunos sistemas NAS usan un estilo de seguridad híbrido que combina listas de control de acceso (ACL) con seguridad POSIX o UNIX tradicional.

Si el sistema notifica su estilo de seguridad como UNIX o POSIX sin incluir el acrónimo "ACL", este problema no le afecta.

En el caso de los sistemas que usan ACL, Azure HPC Cache debe realizar un seguimiento de los valores adicionales específicos del usuario para controlar el acceso a los archivos. Esto se hace mediante la habilitación de una caché de acceso. No hay un control orientado al usuario para activar la caché de acceso, pero puede abrir una incidencia de soporte técnico para solicitar que se habilite para los destinos de almacenamiento afectados en el sistema de caché.

## <a name="next-steps"></a>Pasos siguientes

Si tiene un problema que no se haya tratado en este artículo, [abra una incidencia de soporte técnico](hpc-cache-support-ticket.md) para obtener ayuda de expertos.
