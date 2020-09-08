---
title: Solución de problemas de lentitud en la copia de seguridad de archivos y carpetas
description: Le proporciona una guía para solucionar problemas que le ayudará a diagnosticar la causa de los problemas de rendimiento de Azure Backup
ms.topic: troubleshooting
ms.date: 07/05/2019
ms.openlocfilehash: b3f2ac343ef4a703f347ec8a57f242a636bb32d2
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88824022"
---
# <a name="troubleshoot-slow-backup-of-files-and-folders-in-azure-backup"></a>Solución de problemas de lentitud en la copia de seguridad de archivos y carpetas en Azure Backup

Este artículo proporciona una guía para la solución de problemas que le ayudará a diagnosticar la causa de un rendimiento lento en la copia de seguridad de archivos y carpetas cuando se usa Azure Backup. Si se utiliza el agente de Azure Backup para hacer copia de seguridad de los archivos, el proceso puede tardar más de lo esperado. Este problema puede deberse a uno o a varios de los siguientes motivos:

* [Hay cuellos de botella que afectan al rendimiento del equipo en el que se realiza la copia de seguridad.](#cause1)
* [Otro proceso o software antivirus que interfiere en el proceso de Azure Backup](#cause2)
* [El agente de Copia de seguridad se ejecuta en una máquina virtual (VM) de Azure.](#cause3)  
* [Se realiza una copia de seguridad de un número elevado (varios millones) de archivos.](#cause4)

Antes de empezar a solucionar problemas, se recomienda descargar e instalar el [agente de Azure Backup más reciente](https://aka.ms/azurebackup_agent). Se realizan actualizaciones frecuentes en el agente de copia de seguridad para corregir varios problemas, agregar características y mejorar el rendimiento.

También recomendamos encarecidamente que revise el artículo [P+F de servicio de Azure Backup](backup-azure-backup-faq.md) para asegurarse de que no experimenta alguno de los problemas habituales de la configuración.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="cause-backup-job-running-in-unoptimized-mode"></a>Causa: Trabajo de copia de seguridad en ejecución en modo no optimizado

* El agente de MARS puede ejecutar el trabajo de copia de seguridad en **modo optimizado** con el diario de cambios USN (número de secuencia actualizada) o **modo no optimizado** al comprobar los cambios en los directorios o archivos examinando todo el volumen.
* El modo no optimizado es lento porque el agente tiene que examinar cada uno de los archivos del volumen y compararlos con los metadatos para establecer qué archivos se han modificado.
* Para verificarlo, abra **Detalles del trabajo** desde la consola del agente de MARS y compruebe el estado para ver si indica **Transfiriendo datos (sin optimizar, puede tardar más tiempo)** como se muestra a continuación:

    ![Ejecución en modo no optimizado](./media/backup-azure-troubleshoot-slow-backup-performance-issue/unoptimized-mode.png)

* Las siguientes condiciones pueden hacer que el trabajo de copia de seguridad se ejecute en modo no optimizado:
  * La primera copia de seguridad (también conocida como "Replicación inicial") siempre se ejecutará en modo no optimizado.
  * Si se produce un error en el trabajo de copia de seguridad anterior, el siguiente trabajo de copia de seguridad programado se ejecutará como no optimizado.

<a id="cause1"></a>

## <a name="cause-performance-bottlenecks-on-the-computer"></a>Causa: Cuellos de botella que afectan al rendimiento del equipo.

Los cuellos de botella del equipo en el que se realiza la copia de seguridad pueden provocar retrasos. Por ejemplo, la capacidad del equipo para leer o escribir en el disco, o el ancho de banda disponible para enviar datos a través de la red pueden provocar cuellos de botella.

Windows proporciona una herramienta integrada llamada [Monitor de rendimiento](https://techcommunity.microsoft.com/t5/ask-the-performance-team/windows-performance-monitor-overview/ba-p/375481) (Perfmon) para detectar estos cuellos de botella.

Estos son algunos contadores de rendimiento e intervalos que pueden resultar útiles para diagnosticar cuellos de botella, con el fin de que las copias de seguridad sean óptimas.

| Contador | Estado |
| --- | --- |
| Logical Disk(Physical Disk) [Disco lógico (disco físico)]--% de inactividad |<li> Entre 100 % y 50 % de inactividad = Correcto</br><li> Entre 49 % y 20 % de inactividad = Advertencia o supervisión</br><li> Del 19 % al 0 % de inactividad = Situación crítica o fuera de la especificación |
| Logical Disk(Physical Disk) [Disco lógico (disco físico)]--% promedio Disk Sec Read or Write (Segundos de disco de lectura o escritura) |<li> De 0,001 ms a 0,015 ms  = Correcto</br><li> De 0,015 ms a 0,025 ms = Advertencia o supervisión</br><li> 0,026 o más = Situación crítica o fuera de la especificación |
| Logical Disk(Physical Disk) [Disco lógico (disco físico)] -- Longitud actual de la cola de disco (para todas las instancias) |80 solicitudes durante más de 6 minutos |
| Memoria: Pool Non Paged Bytes (Bytes de bloque no paginado) |<li> Menos del 60 % del grupo consumido = Correcto<br><li> Del 61 % al 80 % del grupo consumido = Advertencia o supervisión</br><li> Más del 80 % del grupo consumido = Situación crítica o fuera de la especificación |
| Memoria: Bytes de bloque paginado |<li> Menos del 60 % del grupo consumido = Correcto</br><li> Del 61 % al 80 % del grupo consumido = Advertencia o supervisión</br><li> Más del 80 % del grupo consumido = Situación crítica o fuera de la especificación |
| Memoria: Megabytes disponibles |<li> 50 % de memoria libre o más disponible = Correcto</br><li> 25 % de memoria libre disponible = Supervisión</br><li>10 % de memoria libre disponible = Advertencia</br><li> Menos de 100 MB o del 5 % de memoria libre disponible = Situación crítica o fuera de la especificación. |
| Procesador: \%Tiempo de procesador (todas las instancias) |<li> Menos del 60 % consumido = Correcto</br><li> Del 61 % al 90 % consumido = Supervisión o precaución</br><li> Del 91 % al 100 % consumido = Situación crítica |

> [!NOTE]
> Si determina que la causa es la infraestructura, se recomienda desfragmentar los discos periódicamente para mejorar su rendimiento.
>
>

<a id="cause2"></a>

## <a name="cause-another-process-or-antivirus-software-interfering-with-azure-backup"></a>Causa: Otro proceso o software antivirus que interfiere con Azure Backup

Se han detectado varios ejemplos en los que otros procesos del sistema de Windows han afectado negativamente al rendimiento del proceso del agente de Azure Backup. Por ejemplo, si utiliza el agente de Azure Backup y otro programa para realizar copias de seguridad de los datos o si se ejecuta un software antivirus que tiene bloqueados los archivos de los que se va a realizar la copia de seguridad, estos bloqueos de varios archivos podrían causar una contención. En esta situación, se podría producir un error en la copia de seguridad o el trabajo podría tardar más de lo esperado.

La mejor recomendación en este escenario consiste en desactivar el otro programa de copia de seguridad para ver si cambia el tiempo de copia de seguridad del agente de Azure Backup. Normalmente, para evitar que unos trabajos de copia de seguridad afecten a otros es suficiente asegurarse de que no se ejecuta varios trabajos al mismo tiempo.

En el caso de los programas antivirus, se recomienda que excluya los siguientes archivos y ubicaciones:

* C:\Archivos de programa\Microsoft Azure Recovery Services Agent\bin\cbengine.exe como proceso.
* Carpetas de C:\Archivos de programa\Microsoft Azure Recovery Services Agent\.
* La ubicación de la carpeta temporal (si no utiliza la ubicación estándar).

<a id="cause3"></a>

## <a name="cause-backup-agent-running-on-an-azure-virtual-machine"></a>Causa: El agente de copia de seguridad que se ejecuta en una máquina virtual de Azure

Si el agente de Copia de seguridad se ejecuta en una máquina virtual, el rendimiento será peor que si lo hace en una máquina física. Esto se debe a limitaciones de IOPS.  Sin embargo, el rendimiento se puede optimizar mediante el cambio de las unidades de datos de las que se realiza la copia de seguridad a Azure Premium Storage. Estamos trabajando para solucionar este problema y dicha solución estará disponible en futuras versiones.

<a id="cause4"></a>

## <a name="cause-backing-up-a-large-number-millions-of-files"></a>Causa: Se realiza una copia de seguridad de un número elevado (varios millones) de archivos.

El movimiento de un gran volumen de datos tarda más tiempo en realizarse el de un volumen más pequeño. En algunos casos, el tiempo que tarda en realizarse una copia de seguridad está relacionado no solo con el tamaño de los datos, sino también con el número de archivos o carpetas. Esto sucede especialmente cuando se realizan copias de seguridad de millones de archivos pequeños (que tienen entre unos pocos bytes y unos pocos kilobytes).

Este comportamiento se produce porque mientras se realiza la copia de seguridad de los datos y su movimiento a Azure, Azure está catalogando los archivos. En algunos casos poco frecuentes, la operación de catálogo puede tardar más tiempo del esperado.

Los siguientes indicadores pueden ayudarle a entender el cuello de botella y trabajar según corresponda en los pasos siguientes:

* **La interfaz de usuario muestra el progreso de la transferencia de datos**. La transferencia de datos no ha finalizado. El ancho de banda de la red o el tamaño de datos pueden estar causando retrasos.
* **La interfaz de usuario no muestra el progreso de la transferencia de datos**. Abra los registros ubicados en "C:\Archivos de programa\Microsoft Azure Recovery Services Agent\Temp" y busque en ellos la entrada FileProvider::EndData. Esta entrada indica que la transferencia de datos ha finalizado y que se está realizando la operación de catálogo. No cancele los trabajos de copia de seguridad. Es preferible que espere hasta que finalice la operación de catálogo. Si el problema persiste, póngase en contacto con el [servicio de soporte técnico de Azure](https://portal.azure.com/#create/Microsoft.Support).

Si está intentando realizar una copia de seguridad de discos de gran tamaño, se recomienda usar [Azure Data Box](./offline-backup-azure-data-box.md) para la primera copia de seguridad (replicación inicial).  Si no puede usar Data Box, los problemas de red transitorios que se produzcan en el entorno durante las transferencias de datos largas a través de la red pueden provocar errores de copia de seguridad.  Para protegerse frente a estos errores, puede agregar algunas carpetas a la copia de seguridad inicial y seguir agregando más carpetas de manera incremental hasta que se haya realizado correctamente la copia de seguridad de todas las carpetas en Azure.  Las copias de seguridad incrementales posteriores serán relativamente más rápidas.

## <a name="next-steps"></a>Pasos siguientes

* [Preguntas comunes acerca de la realización de copias de seguridad de archivos y carpetas](backup-azure-file-folder-backup-faq.md)
