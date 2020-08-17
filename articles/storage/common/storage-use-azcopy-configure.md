---
title: Configuración, optimización y solución de problemas de AzCopy con Azure Storage | Microsoft Docs
description: Configure, optimice y solucione problemas de AzCopy.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 07/27/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: 7e79f186688f3b6531ac24df4e3ae4201cf1903c
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282439"
---
# <a name="configure-optimize-and-troubleshoot-azcopy"></a>Configuración, optimización y solución de problemas de AzCopy

AzCopy es una utilidad de línea de comandos que puede usar para copiar blobs o archivos a una cuenta de almacenamiento o desde una cuenta de almacenamiento. Este artículo le ayuda a realizar tareas de configuración avanzadas y a solucionar los problemas que puedan surgir al usar AzCopy.

> [!NOTE]
> Si busca contenido que le ayude a empezar a trabajar con AzCopy, vea cualquiera de los artículos siguientes:
> - [Introducción a AzCopy](storage-use-azcopy-v10.md)
> - [Transferencia de datos con AzCopy y Blob Storage](storage-use-azcopy-blobs.md)
> - [Transferencia de datos con AzCopy y File Storage](storage-use-azcopy-files.md)
> - [Transferencia de datos con AzCopy y cubos de Amazon S3](storage-use-azcopy-s3.md)

## <a name="configure-proxy-settings"></a>Configuración de los valores de proxy

Para configurar las opciones de proxy para AzCopy, establezca la variable de entorno `https_proxy`. Si ejecuta AzCopy en Windows, AzCopy detecta automáticamente la configuración de proxy, por lo que no tiene que usar esta opción en Windows. Si decide usar esta opción en Windows, invalidará la detección automática.

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | En un símbolo del sistema, use: `set https_proxy=<proxy IP>:<proxy port>`<br> En PowerShell, use: `$env:https_proxy="<proxy IP>:<proxy port>"`|
| **Linux** | `export https_proxy=<proxy IP>:<proxy port>` |
| **macOS** | `export https_proxy=<proxy IP>:<proxy port>` |

En la actualidad, AzCopy no admite servidores proxy que requieren autenticación con NTLM o Kerberos.

### <a name="bypassing-a-proxy"></a>Omisión de un proxy ###

Si está ejecutando AzCopy en Windows y desea indicarle que _no_ use ningún proxy en absoluto (en lugar de detectar automáticamente la configuración), use estos comandos. Con esta configuración, AzCopy no buscará ni intentará usar ningún proxy.

| Sistema operativo | Entorno | Comandos:  |
|--------|-----------|----------|
| **Windows** | Símbolo del sistema (CMD) | `set HTTPS_PROXY=dummy.invalid` <br>`set NO_PROXY=*`|
| **Windows** | PowerShell | `$env:HTTPS_PROXY="dummy.invalid"` <br>`$env:NO_PROXY="*"`<br>|

En otros sistemas operativos, simplemente deje la variable HTTPS_PROXY sin establecer si no desea usar ningún proxy.

## <a name="optimize-performance"></a>Optimización del rendimiento

Puede realizar un banco de pruebas de rendimiento y, después, usar comandos y variables de entorno para encontrar un equilibrio óptimo entre el rendimiento y el consumo de recursos.

Esta sección le ayuda a realizar estas tareas de optimización:

> [!div class="checklist"]
> * Ejecución de pruebas del banco de pruebas
> * Optimización del rendimiento
> * Optimización del uso de memoria 
> * Optimización de la sincronización de archivos

### <a name="run-benchmark-tests"></a>Ejecución de pruebas del banco de pruebas

Puede ejecutar una prueba del banco de pruebas de rendimiento en contenedores de blobs o recursos compartidos de archivos específicos para ver las estadísticas generales de rendimiento y para identificar los cuellos de botella de rendimiento. Puede ejecutar la prueba al cargar o descargar los datos de prueba generados. 

Utilice el siguiente comando para ejecutar un banco de pruebas de rendimiento.

|    |     |
|--------|-----------|
| **Sintaxis** | `azcopy benchmark 'https://<storage-account-name>.blob.core.windows.net/<container-name>'` |
| **Ejemplo** | `azcopy benchmark 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D'` |

> [!TIP]
> En este ejemplo los argumentos de ruta de acceso se encierran entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

Este comando ejecuta un banco de pruebas de rendimiento mediante la carga los datos de prueba en un destino especificado. Los datos de prueba se generan en la memoria, se cargan en el destino y, a continuación, se eliminan del destino una vez completada la prueba. Puede especificar el número de archivos que se van a generar y el tamaño que desea que se utilicen mediante parámetros de comando opcionales.

Si prefiere descargar datos para ejecutar esta prueba, establezca el parámetro `mode` en `download`. Para ver documentos de referencia detallados, consulte [azcopy benchmark](storage-ref-azcopy-bench.md). 

### <a name="optimize-throughput"></a>Optimización del rendimiento

Puede usar la marca `cap-mbps` en los comandos para colocar un límite superior en la velocidad de datos de rendimiento. Por ejemplo, el siguiente comando reanuda un trabajo y limita el rendimiento a `10` megabits (MB) por segundo. 

```azcopy
azcopy jobs resume <job-id> --cap-mbps 10
```

El rendimiento puede disminuir al transferir archivos pequeños. Puede aumentar el rendimiento si establece la variable de entorno `AZCOPY_CONCURRENCY_VALUE`. Esta variable especifica el número de solicitudes simultáneas que pueden producirse.  

Si el equipo tiene menos de cinco CPU, el valor de esta variable se establece en `32`. En caso contrario, el valor predeterminado es igual a 16 multiplicado por el número de CPU. El valor máximo predeterminado de esta variable es `3000`, pero puede establecerlo manualmente en un valor superior o inferior. 

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | `set AZCOPY_CONCURRENCY_VALUE=<value>` |
| **Linux** | `export AZCOPY_CONCURRENCY_VALUE=<value>` |
| **macOS** | `export AZCOPY_CONCURRENCY_VALUE=<value>` |

Use `azcopy env` para comprobar el valor actual de esta variable. Si el valor está en blanco, puede leer el valor que se está usando al examinar el principio de cualquier archivo de registro de AzCopy. El valor seleccionado, y el motivo por el que se seleccionó, se indican allí.

Antes de establecer esta variable, se recomienda ejecutar una prueba del banco de pruebas. El proceso de prueba del banco de pruebas informará del valor de simultaneidad recomendado. Como alternativa, si las condiciones de la red y las cargas varían, establezca esta variable en la palabra `AUTO` en lugar de en un número determinado. Esto hará que AzCopy ejecute siempre el mismo proceso de ajuste automático que utiliza en las pruebas del banco de pruebas.

### <a name="optimize-memory-use"></a>Optimización del uso de memoria

Establezca la variable de entorno `AZCOPY_BUFFER_GB` para especificar la cantidad máxima de memoria del sistema que desea que AzCopy use al descargar y cargar archivos.
Exprese este valor en gigabytes (GB).

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | `set AZCOPY_BUFFER_GB=<value>` |
| **Linux** | `export AZCOPY_BUFFER_GB=<value>` |
| **macOS** | `export AZCOPY_BUFFER_GB=<value>` |

### <a name="optimize-file-synchronization"></a>Optimización de la sincronización de archivos

El comando [sync](storage-ref-azcopy-sync.md) identifica todos los archivos en el destino y, luego, compara los nombres de archivo y las marcas de tiempo de última modificación antes de iniciar la operación de sincronización. Si tiene un gran número de archivos, puede eliminar este procesamiento por adelantado para mejorar el rendimiento. 

Para ello, use en su lugar el comando [azcopy copy](storage-ref-azcopy-copy.md) y establezca la marca `--overwrite` en `ifSourceNewer`. AzCopy comparará los archivos a medida que se copien sin necesidad de realizar exámenes y comparaciones por adelantado. Esto proporciona una ventaja para el rendimiento en los casos en los que hay un gran número de archivos que se van a comparar.

El comando [azcopy copy](storage-ref-azcopy-copy.md) no elimina archivos del destino, por lo que si desea eliminar archivos en el destino cuando ya no existan en el origen, use el comando [azcopy sync](storage-ref-azcopy-sync.md) con la marca `--delete-destination` establecida en el valor `true` o `prompt`. 

## <a name="troubleshoot-issues"></a>Solución de problemas

AzCopy crea archivos de registro y de plan para cada trabajo. Puede usar los registros para investigar y solucionar los problemas potenciales. 

Los registros contendrán el estado de error (`UPLOADFAILED`, `COPYFAILED` y `DOWNLOADFAILED`), la ruta de acceso completa y el motivo del error.

De forma predeterminada, los archivos de registro y de plan se encuentran en el directorio `%USERPROFILE%\.azcopy` de Windows o en el directorio `$HOME$\.azcopy` en Mac y Linux, pero puede cambiar la ubicación si lo desea.

El error pertinente no es necesariamente el primer error que aparece en el archivo. En el caso de errores como errores de red, tiempos de espera y errores de servidor ocupado, AzCopy volverá a intentarlo hasta 20 veces y, normalmente, el proceso de reintento se realizará correctamente.  El primer error que se ve podría ser algo inofensivo que se reintentara correctamente.  Por lo tanto, en lugar de examinar el primer error del archivo, busque los errores cercanos `UPLOADFAILED`, `COPYFAILED` o `DOWNLOADFAILED`. 

> [!IMPORTANT]
> Al enviar una solicitud a Soporte técnico de Microsoft (o al solucionar el problema con la participación de terceros), comparta la versión censurada del comando que quiere ejecutar. Esto garantiza que la SAS no se comparta de forma accidental con nadie. Puede encontrar la versión censurada al principio del archivo de registro.

### <a name="review-the-logs-for-errors"></a>Revisión de los registros en busca errores

El comando siguiente obtiene todos los errores con el estado `UPLOADFAILED` del registro `04dc9ca9-158f-7945-5933-564021086c79`:

**Windows (PowerShell)**

```
Select-String UPLOADFAILED .\04dc9ca9-158f-7945-5933-564021086c79.log
```

**Linux**

```
grep UPLOADFAILED .\04dc9ca9-158f-7945-5933-564021086c79.log
```

### <a name="view-and-resume-jobs"></a>Visualización y reanudación de trabajos

Cada operación de transferencia creará un trabajo de AzCopy. Use el comando siguiente para ver el historial de trabajos:

```
azcopy jobs list
```

Para ver las estadísticas del trabajo, use el siguiente comando:

```
azcopy jobs show <job-id>
```

Para filtrar a las transferencias por estado, use el siguiente comando:

```
azcopy jobs show <job-id> --with-status=Failed
```

Use el comando siguiente para reanudar un trabajo con error o cancelado. Este comando usa su identificador, junto con el token de SAS, ya que no es persistente por motivos de seguridad:

```
azcopy jobs resume <job-id> --source-sas="<sas-token>"
azcopy jobs resume <job-id> --destination-sas="<sas-token>"
```

> [!TIP]
> Encierre los argumentos de ruta de acceso, como el token de SAS, entre comillas simples ('). Use comillas simples en todos los shells de comandos excepto en el shell de comandos de Windows (cmd.exe). Si usa un shell de comandos de Windows (cmd.exe), incluya los argumentos de la ruta de acceso entre comillas dobles ("") en lugar de comillas simples ('').

Al reanudar un trabajo, AzCopy examina el archivo de plan de trabajo. En el archivo de plan se enumeran todos los archivos que se han identificado para el procesamiento al crear el trabajo por primera vez. Al reanudar un trabajo, AzCopy intentará transferir todos los archivos que aparecen en el archivo de plan que no se han transferido todavía.

## <a name="change-the-location-of-the-plan-and-log-files"></a>Cambie las ubicaciones de los archivos de registro y de plan.

De forma predeterminada, los archivos de registro y de plan se encuentran en el directorio `%USERPROFILE%\.azcopy` de Windows o en el directorio `$HOME$\.azcopy` en Mac y Linux. Pero puede cambiar esta ubicación.

### <a name="change-the-location-of-plan-files"></a>Cambio de la ubicación de los archivos de plan

Use cualquiera de estos comandos.

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | PowerShell:`$env:AZCOPY_JOB_PLAN_LOCATION="<value>"` <br> En un símbolo del sistema, use: `set AZCOPY_JOB_PLAN_LOCATION=<value>` |
| **Linux** | `export AZCOPY_JOB_PLAN_LOCATION=<value>` |
| **macOS** | `export AZCOPY_JOB_PLAN_LOCATION=<value>` |

Use `azcopy env` para comprobar el valor actual de esta variable. Si el valor está en blanco, los registros se escriben en la ubicación predeterminada.

### <a name="change-the-location-of-log-files"></a>Cambio de la ubicación de los archivos de registro

Use cualquiera de estos comandos.

| Sistema operativo | Get-Help  |
|--------|-----------|
| **Windows** | PowerShell:`$env:AZCOPY_LOG_LOCATION="<value>"` <br> En un símbolo del sistema, use: `set AZCOPY_LOG_LOCATION=<value>`|
| **Linux** | `export AZCOPY_LOG_LOCATION=<value>` |
| **macOS** | `export AZCOPY_LOG_LOCATION=<value>` |

Use `azcopy env` para comprobar el valor actual de esta variable. Si el valor está en blanco, los registros se escriben en la ubicación predeterminada.

## <a name="change-the-default-log-level"></a>Cambiar el nivel de registro predeterminado

De forma predeterminada, el nivel de registro de AzCopy se establece en `INFO`. Si quiere reducir el nivel de detalle del registro para ahorrar espacio en disco, sobrescriba este valor mediante la opción ``--log-level``. 

Los niveles de registro disponibles son: `NONE`, `DEBUG`, `INFO`, `WARNING`, `ERROR`, `PANIC` y `FATAL`.

## <a name="remove-plan-and-log-files"></a>Eliminación de archivos de registro y de plan

Si quiere quitar todos los archivos de registro y de plan de la máquina local para ahorrar espacio en disco, use el comando `azcopy jobs clean`.

Para quitar los archivos de registro y de plan asociados a un solo trabajo, use `azcopy jobs rm <job-id>`. Reemplace el marcador de posición `<job-id>` en este ejemplo por el identificador del trabajo.


