---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 05/14/2019
ms.author: cynthn;kareni
ms.custom: include file
ms.openlocfilehash: ba41f6cce5233491020a0b42f4fd40dac060be57
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65815536"
---
**Última actualización del documento**: 14 de mayo de 2019 10:00 A.M. hora del Pacífico.

La divulgación de una [nueva clase de vulnerabilidades de CPU](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180002), conocidas como ataques de canal lateral de ejecución especulativa, han generado preguntas de los clientes que buscan mayor claridad.  

Microsoft ha implementado mitigaciones en todos nuestros servicios en la nube. La infraestructura que ejecuta Azure y aísla las cargas de trabajo de los clientes entre sí está protegida. Esto significa que un atacante potencial que use la misma infraestructura no puede atacar su aplicación mediante estas vulnerabilidades.

Azure utiliza un [mantenimiento con conservación de memoria](https://docs.microsoft.com/azure/virtual-machines/windows/maintenance-and-updates#maintenance-not-requiring-a-reboot) siempre que es posible para minimizar el impacto en el cliente y eliminar la necesidad de reinicios. Azure seguirá utilizando estos métodos al realizar actualizaciones del host en todo el sistema y proteger a los clientes.

Encontrará más información acerca de cómo integrar la seguridad en todos los aspectos de Azure en el sitio de [documentación de Azure Security Center](https://docs.microsoft.com/azure/security/). 

> [!NOTE] 
> Desde que este documento se publicó por primera vez, se han divulgado varias variantes de esta clase de vulnerabilidad. Microsoft continúa realizando grandes esfuerzos por proteger a nuestros clientes y proporcionarles orientación. Esta página se actualizará a medida que se publiquen más correcciones. 
> 
> En el 14 de mayo de 2019 [Intel revelada](https://www.intel.com/content/www/us/en/security-center/advisory/intel-sa-00233.html) un nuevo conjunto de vulnerabilidad de canal lateral de ejecución especulativa conocido como muestreo de datos de la microarquitectura (MDS consulte las instrucciones de seguridad de Microsoft [ADV190013](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV190013)), que se ha asignado varios identificadores: 
> - CVE-2019-11091 - datos de la microarquitectura muestreo de memoria Uncacheable (MDSUM)
> - CVE-2018-12126 - Store microarquitectura búfer los datos de muestreo (MSBDS) 
> - CVE-2018-12127 - los datos de puerto de carga de la microarquitectura de muestreo (MLPDS)
> - CVE-2018-12130 - relleno de la microarquitectura búfer los datos de muestreo (MFBDS)
>
> Esta vulnerabilidad afecta a los procesadores Intel® Core® y los procesadores Intel® Xeon®.  Microsoft Azure ha publicado actualizaciones del sistema operativo y se está implementando nuevas microcódigo, cuando vaya a estar disponible por Intel, en toda nuestra flota para proteger a nuestros clientes contra estas nuevas vulnerabilidades.   Azure está trabajando estrechamente con Intel para probar y validar la nueva habilitación del microcódigo antes de su lanzamiento oficial en la plataforma. 
>
> **Los clientes que ejecutan confianza código dentro de su máquina virtual** deba tomar medidas para protegerse frente a estas vulnerabilidades, se lee a continuación para obtener instrucciones acerca de todas las vulnerabilidades de canal lateral de ejecución especulativa (ADV de avisos de Microsoft [180002](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180002), [180018](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/adv180018), y [190013](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV190013)).
>
> Otros clientes deben evaluar estas vulnerabilidades de una defensa de la perspectiva de profundidad y tenga en cuenta las implicaciones de seguridad y el rendimiento de su configuración elegida.



## <a name="keeping-your-operating-systems-up-to-date"></a>Mantener actualizados sus sistemas operativos

Aunque no se necesita una actualización del sistema operativo para aislar de otros clientes las aplicaciones que se ejecutan en Azure, mantener el software actualizado siempre es un procedimiento recomendado. Los últimos paquetes acumulativos de actualizaciones de seguridad para Windows contienen mitigaciones para varias vulnerabilidades de canal lateral de ejecución especulativa. De forma similar, las distribuciones de Linux han publicado varias actualizaciones para corregir estas vulnerabilidades. Estas son las acciones recomendadas para actualizar el sistema operativo:

| Oferta | Acción recomendada  |
|----------|---------------------|
| Azure Cloud Services  | Habilite las [actualizaciones automáticas](https://docs.microsoft.com/azure/cloud-services/cloud-services-how-to-configure-portal) o asegúrese de que está ejecutando la versión más reciente del sistema operativo invitado. |
| Azure Linux Virtual Machines | Instale las actualizaciones de su proveedor de sistema operativo. Para más información, consulte [Linux](#linux) más adelante en este documento. |
| Máquinas virtuales Windows en Azure  | Instale las últimas actualizaciones acumulativas de seguridad.
| Otros servicios PaaS de Azure | No es necesaria ninguna acción para los clientes que usan estos servicios. Azure mantiene actualizadas las versiones de sistema operativo de forma automática. |

## <a name="additional-guidance-if-you-are-running-untrusted-code"></a>Instrucciones adicionales si está ejecutando código que no es de confianza 

Los clientes que permiten a los usuarios que no son de confianza ejecutar código arbitrario tal vez deseen implementar algunas características de seguridad adicionales en las máquinas virtuales o en los servicios en la nube de Azure. Estas características protegen frente a los vectores de divulgación entre procesos que describen varias vulnerabilidades de ejecución especulativa.

Escenarios de ejemplo donde se recomiendan las características de seguridad adicionales:

- Permite la ejecución de código que no es de confianza en la máquina virtual.  
    - *Por ejemplo, permite que uno de sus clientes cargue un archivo binario o un script que, a continuación, se ejecutará en la aplicación*. 
- Permite iniciar sesión en la máquina virtual con cuentas con privilegios reducidos a usuarios que no son de confianza.   
    - *Por ejemplo, permite iniciar sesión en una de las máquinas virtuales mediante escritorio remoto o SSH a un usuario con privilegios reducidos*.  
- Permite acceder a las máquinas virtuales implementadas mediante virtualización anidada a usuarios que no son de confianza.  
    - *Por ejemplo, controla el host de Hyper-V, pero asigna las máquinas virtuales a usuarios que no son de confianza*. 

Los clientes que no implementan un escenario relacionado con código que no es de confianza no necesitan habilitar estas características de seguridad adicionales. 

## <a name="enabling-additional-security"></a>Habilitación de la seguridad adicional 

Puede habilitar características de seguridad adicionales dentro de la máquina virtual o servicio en la nube si está ejecutando código no seguro. En paralelo, asegúrese de que el sistema operativo está actualizado para habilitar las características de seguridad dentro de la máquina virtual o servicio en la nube

### <a name="windows"></a> Windows 

El sistema operativo de destino debe estar actualizado para habilitar estas características de seguridad adicionales. Aunque hay habilitadas numerosas mitigaciones del canal lateral de ejecución especulativa de forma predeterminada, las características adicionales que se describen aquí deben habilitarse manualmente y pueden afectar al rendimiento. 


**Paso 1: Deshabilitar el hyperthreading en la máquina virtual** : los clientes que ejecutan código no seguro en un hiperproceso VM deberá deshabilitar el hyperthreading o mover a un tamaño de máquina virtual no hyperthreading. Para comprobar si la máquina virtual tiene habilitado el hiperproceso, consulte el siguiente script mediante la línea de comandos de Windows desde la máquina virtual.

Tipo `wmic` para entrar en la interfaz interactiva. A continuación, escriba el siguiente para ver la cantidad de física y lógica procesadores en la máquina virtual.

```console
CPU Get NumberOfCores,NumberOfLogicalProcessors /Format:List
```

Si el número de procesadores lógicos es mayor de procesadores físicos (núcleos), se habilita el hyperthreading.  Si está ejecutando una máquina virtual hiperproceso, inicie [póngase en contacto con soporte técnico de Azure](https://aka.ms/MicrocodeEnablementRequest-SupportTechnical) para obtener el hyperthreading deshabilitado.  Una vez que el hyperthreading está deshabilitado, **soporte requerirá un reinicio de máquina virtual completo**. 


**Paso 2**: En paralelo al paso 1, siga las instrucciones de [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) para comprobar las protecciones se habilitan mediante la [SpeculationControl](https://aka.ms/SpeculationControlPS) módulo de PowerShell.

> [!NOTE]
> Si descargó este módulo anteriormente, deberá instalar la versión más reciente.
>


La salida del script de PowerShell debe tener el siguiente para validar los valores habilitados protecciones frente a estas vulnerabilidades:

```
Windows OS support for branch target injection mitigation is enabled: True
Windows OS support for kernel VA shadow is enabled: True
Windows OS support for speculative store bypass disable is enabled system-wide: False
Windows OS support for L1 terminal fault mitigation is enabled: True
Windows OS support for MDS mitigation is enabled: True
```

Si el resultado muestra `MDS mitigation is enabled: False`, por favor, [póngase en contacto con soporte técnico de Azure](https://aka.ms/MicrocodeEnablementRequest-SupportTechnical) para las opciones de mitigación disponibles.



**Paso 3**: Para habilitar el sombreado de dirección Virtual (KVAS) de Kernel y soporte técnico de inyección de código de destino de rama (BTI) del sistema operativo, siga las instrucciones de [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) para habilitar la protección mediante el `Session Manager` claves del registro. Es necesario reiniciar.


**Paso 4**: Para las implementaciones que usan [virtualización anidada](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization) (D3 y E3 solo): Estas instrucciones se aplican dentro de la máquina virtual que se va a usar como un host de Hyper-V.

1.  Siga las instrucciones de [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) para habilitar la protección mediante el `MinVmVersionForCpuBasedMitigations` claves del registro.
2.  Establezca el tipo de hipervisor programador en `Core` siguiendo las instrucciones [aquí](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/manage-hyper-v-scheduler-types).


### <a name="linux"></a>Linux

<a name="linux"></a>Se requiere que el sistema operativo de destino esté completamente actualizado para habilitar el conjunto de características de seguridad adicionales. Algunas de las mitigaciones estarán habilitadas de forma predeterminada. La siguiente sección describe las características que están desactivadas de forma predeterminada y sujetas a la compatibilidad de hardware (microcódigo). La habilitación de estas características puede afectar al rendimiento. Consulte la documentación del proveedor de su sistema operativo para más instrucciones


**Paso 1: Deshabilitar el hyperthreading en la máquina virtual** : los clientes que ejecutan código no seguro en un hiperproceso VM deberá deshabilitar el hyperthreading o mover a una VM no hyperthreading.  Para comprobar si está ejecutando una máquina virtual hiperproceso, ejecute el `lscpu` comando en la VM de Linux. 

Si `Thread(s) per core = 2`, a continuación, se ha habilitado el hyperthreading. 

Si `Thread(s) per core = 1`, a continuación, se ha deshabilitado el hyperthreading. 

 
Ejemplo de salida para una máquina virtual con el hyperthreading habilitado: 

```console
CPU Architecture:      x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                8
On-line CPU(s) list:   0,2,4,6
Off-line CPU(s) list:  1,3,5,7
Thread(s) per core:    2
Core(s) per socket:    4
Socket(s):             1
NUMA node(s):          1

```

Si está ejecutando una máquina virtual hiperproceso, inicie [póngase en contacto con soporte técnico de Azure](https://aka.ms/MicrocodeEnablementRequest-SupportTechnical) para obtener el hyperthreading deshabilitado.  Una vez que el hyperthreading está deshabilitado, **soporte requerirá un reinicio de máquina virtual completo**.


**Paso 2**: Para mitigar el riesgo cualquiera de las siguientes vulnerabilidades de canal lateral de ejecución especulativa, consulte la documentación del proveedor de su sistema operativo:   
 
- [RedHat y CentOS](https://access.redhat.com/security/vulnerabilities) 
- [SUSE](https://www.suse.com/support/kb/?doctype%5B%5D=DT_SUSESDB_PSDB_1_1&startIndex=1&maxIndex=0) 
- [Ubuntu](https://wiki.ubuntu.com/SecurityTeam/KnowledgeBase/) 

## <a name="next-steps"></a>Pasos siguientes

Este artículo proporcionan instrucciones para la debajo de los ataques de canal lateral de ejecución especulativa que afectan a muchos de los procesadores modernos:

[Spectre Meltdown](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV180002):
- CVE-2017-5715 - inserción de rama de destino (BTI)  
- CVE-2017-5754 - aislamiento de la tabla de páginas de Kernel (KPTI)
- CVE-2018-3639 – Store especulativa omisión (KPTI) 
 
[L1 Error de Terminal (L1TF)](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV180018):
- CVE-2018-3615 - Intel Software Guard Extensions (SGX de Intel)
- CVE-2018-3620 - sistemas operativos (SO) y modo de administración del sistema (SMM)
- CVE-2018-3646 – impactos de Virtual Machine Manager (VMM)

[Muestreo de datos de la microarquitectura](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV190013): 
- CVE-2019-11091 - datos de la microarquitectura muestreo de memoria Uncacheable (MDSUM)
- CVE-2018-12126 - Store microarquitectura búfer los datos de muestreo (MSBDS)
- CVE-2018-12127 - los datos de puerto de carga de la microarquitectura de muestreo (MLPDS)
- CVE-2018-12130 - relleno de la microarquitectura búfer los datos de muestreo (MFBDS)








