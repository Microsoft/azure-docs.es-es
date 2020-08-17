---
title: Información sobre la nube por niveles de Azure File Sync | Microsoft Docs
description: Obtenga información sobre la característica Nube por niveles de Azure File Sync
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 06/15/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 74887e6ee4656091aa647b481bc406dcc23b9c12
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87460089"
---
# <a name="cloud-tiering-overview"></a>Información general de nube por niveles
La nube por niveles es una característica opcional de Azure File Sync por la que los archivos a los que se tiene acceso con frecuencia se almacenan en caché localmente en el servidor mientras que todos los demás archivos se organizan en niveles en Azure Files, según la configuración de directiva. Cuando un archivo está en capas, el filtro del sistema de archivos de Azure File Sync (StorageSync.sys) sustituye al archivo localmente por un puntero o punto de repetición de análisis. El punto de repetición de análisis representa una dirección URL del archivo en Azure Files. Un archivo con niveles tiene los atributos “sin conexión” y FILE_ATTRIBUTE_RECALL_ON_DATA_ACCESS establecidos en NTFS para que las aplicaciones de terceros puedan identificar de forma segura archivos con niveles.
 
Cuando un usuario abre un archivo con niveles, Azure File Sync recupera completamente los datos de archivo de Azure Files sin necesidad de que el usuario sepa que el archivo está almacenado en Azure. 
 
 > [!Important]  
 > La nube por niveles no se admite en el volumen del sistema de Windows.
    
 > [!Important]  
 > Para recuperar los archivos que están organizados en niveles, el ancho de banda de red debe ser de al menos 1 Mbps. Si el ancho de banda de red es menor que 1 Mbps, puede ocurrir un error de tiempo de expiración al recuperar los archivos.

## <a name="cloud-tiering-faq"></a>Preguntas frecuentes de la nube por niveles

<a id="afs-cloud-tiering"></a>
### <a name="how-does-cloud-tiering-work"></a>¿Cómo funciona la nube por niveles?
El filtro del sistema de Azure File Sync se basa en un “mapa térmico” del espacio de nombres en cada punto de conexión de servidor. Supervisa los accesos (operaciones de lectura y escritura) a lo largo del tiempo y después, en función de la frecuencia y la cercanía en el tiempo del acceso, asigna una puntuación de calor a todos los archivos. Un archivo al que se accede con frecuencia que se ha abierto recientemente se considerará como caliente, mientras que se considerará frío un archivo que apenas se toca y al que no se ha accedido desde hace algún tiempo. Cuando el volumen de archivos en un servidor supera el umbral de espacio disponible del volumen que estableció, organizará por niveles los archivos más fríos en Azure Files hasta que se cumpla el porcentaje de espacio libre.

Además, se puede especificar una directiva de fecha en cada punto de conexión de servidor que coloque por niveles los archivos a los que no se haya accedido en un número de días especificado, independientemente de la capacidad de almacenamiento local disponible. Esta es una buena opción para liberar espacio en disco local de forma proactiva si se sabe que los archivos de ese punto de conexión de servidor no necesitan conservarse en local más allá de una determinada antigüedad. Eso libera una valiosa capacidad de disco local para que otros puntos de conexión del mismo volumen almacenen en caché más archivos.

El mapa térmico de la nube por niveles es básicamente una lista ordenada de todos los archivos que se están sincronizando y que están en una ubicación con la nube por niveles habilitada. Para determinar la posición relativa de un archivo individual en ese mapa térmico, el sistema usa el valor máximo de cualquiera de las siguientes marcas de tiempo, en ese orden: MAX(Hora del último acceso, Hora de la última modificación, Hora de creación). Normalmente, la hora del último acceso está registrada y disponible. Pero cuando se crea un nuevo punto de conexión de servidor con la nube por niveles habilitada, al principio no ha pasado suficiente tiempo para observar el acceso al archivo. En ausencia de una hora de último acceso, se usa la hora de la última modificación para evaluar la posición relativa en el mapa térmico. Lo mismo se aplica a la directiva de fecha. Sin una hora de último acceso, la directiva de fecha actúa en función de la hora de la última modificación. Si no estuviera disponible, se pasa a la hora de creación de un archivo. Con el tiempo, el sistema observa más y más solicitudes de acceso al archivo y vuelve a usar principalmente la hora de último acceso autorregistrada.

La nube por niveles no depende de la característica NTFS para el registro de la hora del último acceso. Esta característica NTFS está desactivada de forma predeterminada y, debido a consideraciones de rendimiento, no se recomienda habilitarla manualmente. La nube por niveles registra la hora del último acceso por separado y de manera muy eficaz.

<a id="tiering-minimum-file-size"></a>
### <a name="what-is-the-minimum-file-size-for-a-file-to-tier"></a>¿Cuál es el tamaño de archivo mínimo para organizar un archivo en niveles?

En las versiones del agente 9 y más recientes, el tamaño de archivo mínimo para organizar un archivo por niveles se basa en el tamaño del clúster del sistema de archivos. El tamaño mínimo del archivo que se selecciona para la realizar agrupación en niveles en la nube se calcula en función del doble del tamaño del clúster y en un mínimo de 8 KB. En la tabla siguiente se muestran los tamaños de archivo mínimos que se pueden organizar por niveles, en función del tamaño del clúster del volumen:

|Tamaño del clúster del volumen (bytes) |Los archivos de este tamaño o mayores se pueden organizar por niveles  |
|----------------------------|---------|
|4 KB o más pequeño (4096)      | 8 KB    |
|8 KB (8192)                 | 16 KB   |
|16 KB (16384)               | 32 KB   |
|32 KB (32768)               | 64 KB   |
|64 KB (65536)               | 128 KB  |

Con Windows Server 2019 y el agente de Azure File Sync (versión 12 y posteriores), también se admiten los tamaños de clúster de hasta 2 MB, y los niveles de clústeres de mayor tamaño funcionan de la misma manera. Las versiones anteriores del sistema operativo o del agente admiten tamaños de clúster de hasta 64 KB.

Todos los sistemas de archivos que usa Windows organizan el disco duro en función del tamaño del clúster (lo que también se conoce como tamaño de la unidad de asignación). El tamaño del clúster representa la cantidad más pequeña de espacio en disco que se puede usar para conservar un archivo. Cuando los tamaños de archivo no son un múltiplo par del tamaño del clúster, se debe usar espacio adicional para conservar el archivo (hasta el siguiente múltiplo del tamaño del clúster).

Azure File Sync se admite en volúmenes NTFS con Windows Server 2012 R2 y versiones más recientes. En la tabla siguiente se describen los tamaños de clúster predeterminados cuando se crea un nuevo volumen NTFS. 

|Tamaño del volumen    |Windows Server 2012 R2 y versiones más recientes |
|---------------|---------------|
|7 MB – 16 TB   | 4 KB          |
|16 TB – 32 TB   | 8 KB          |
|32 TB – 64 TB   | 16 KB         |
|64 TB – 128 TB  | 32 KB         |
|128 TB – 256 TB | 64 KB         |
|> 256 TB       | No compatible |

Es posible que, después de crear el volumen, se le haya aplicado formato manualmente con otro tamaño de clúster. Si el volumen procede de una versión anterior de Windows, los tamaños de clúster predeterminados también pueden diferir. [En este artículo se ofrecen más detalles sobre los tamaños de clúster predeterminados.](https://support.microsoft.com/help/140365/default-cluster-size-for-ntfs-fat-and-exfat) Incluso si elige un tamaño de clúster inferior a 4 KB, se seguirá aplicando un límite de 8 KB como el tamaño de archivo más pequeño que se puede organizar en niveles. (Aunque el tamaño doble del clúster técnicamente sea menor que 8 KB).

El motivo del mínimo absoluto se encuentra en la forma en que NTFS almacena archivos extremadamente pequeños; por ejemplo, de 1 KB a 4 KB de tamaño. En función de otros parámetros del volumen, es posible que los archivos pequeños no se almacenen en un clúster del disco. Igualmente, es posible que sea más eficaz almacenar dichos archivos directamente en la tabla de archivos maestros del volumen o en "Registro de MFT". El punto de reanálisis de los niveles de la nube siempre se almacena en el disco y toma exactamente un clúster. La organización en niveles de estos pequeños archivos podría hacer que acabe sin espacio libre. Los casos extremos pueden incluso acabar usando más espacio con la opción de organizar la nube por niveles habilitada. Para evitar esto, el tamaño mínimo de un archivo que la nube por niveles distribuirá en niveles es de 8 KB en un clúster de 4 KB o inferior.

<a id="afs-volume-free-space"></a>
### <a name="how-does-the-volume-free-space-tiering-policy-work"></a>¿Cómo funciona la directiva de organización por niveles de espacio disponible del volumen?
El espacio disponible del volumen es la cantidad de espacio libre que desea reservar en el volumen en el que reside un punto de conexión de servidor. Por ejemplo, si el espacio disponible del volumen se establece en 20 % en un volumen que tiene un punto de conexión de servidor, hasta un 80 % del espacio de volumen permanecerá ocupado por los archivos a los que se ha accedido más recientemente, mientras que los archivos restantes que no caben dentro de este espacio se organizan en niveles en Azure. El espacio disponible del volumen se aplica en el nivel de volumen en lugar de en el nivel de directorios individuales o en lo grupos de sincronización. 

<a id="volume-free-space-fastdr"></a>
### <a name="how-does-the-volume-free-space-tiering-policy-work-with-regards-to-new-server-endpoints"></a>¿Cómo funciona la directiva de organización en niveles del espacio disponible de volumen con respecto a los nuevos puntos de conexión de servidor?
Cuando un punto de conexión de servidor acaba de aprovisionarse y conectarse a un recurso compartido de archivos de Azure, el servidor primero extraerá el espacio de nombres y después los archivos hasta que llegue a su umbral de espacio disponible del volumen. Este proceso también se denomina restauración rápida del espacio de nombres o recuperación ante desastres rápida.

<a id="afs-effective-vfs"></a>
### <a name="how-is-volume-free-space-interpreted-when-i-have-multiple-server-endpoints-on-a-volume"></a>¿Cómo se interpreta el espacio disponible del volumen cuando tengo varios puntos de conexión de servidor en un volumen?
Cuando hay más de un punto de conexión de servidor en un volumen, el umbral de espacio disponible del volumen efectivo es el mayor espacio disponible del volumen especificado en cualquier punto de conexión de servidor de ese volumen. Los archivos se pueden almacenar en capas según sus patrones de uso, independientemente de qué punto de conexión de servidor al que pertenezcan. Por ejemplo, si tiene dos puntos de conexión de servidor en un volumen, Endpoint1 y Endpoint2, donde Endpoint1 tiene un umbral de espacio disponible del volumen del 25 %, y Endpoint2 tiene un umbral de espacio disponible del volumen de 50 %, el umbral del volumen de espacio disponible para ambos puntos de conexión de servidor será el 50 %. 

<a id="date-tiering-policy"></a>
### <a name="how-does-the-date-tiering-policy-work-in-conjunction-with-the-volume-free-space-tiering-policy"></a>¿Cómo funciona la directiva de organización por niveles basada en fechas junto con la directiva de organización por niveles del espacio disponible de volumen? 
Cuando se habilita la nube por niveles en el punto de conexión de un servidor, se establece una directiva de espacio disponible en el volumen. Esta directiva siempre tiene prioridad sobre cualquier otra, incluidas las directivas de fecha. Opcionalmente, puede habilitar una directiva de fecha para cada punto de conexión de servidor de ese volumen. Esta directiva administra que solo los archivos a los que se tiene acceso (es decir, los que se leen o en los que se escribe) dentro del intervalo de días que describe esta directiva se conservarán de forma local. Los archivos a los que no se tiene acceso con el número de días especificado se almacenarán en niveles. 

La organización en niveles en la nube usa la hora de último acceso para determinar qué archivos deben estar organizados en niveles. El controlador de filtro de la nube por niveles (storagesync.sys) realiza un seguimiento de la hora del último acceso y registra la información en el almacén térmico de la nube por niveles. Puede ver el almacén térmico mediante un cmdlet de PowerShell local.

```powershell
Import-Module '<SyncAgentInstallPath>\StorageSync.Management.ServerCmdlets.dll'
Get-StorageSyncHeatStoreInformation '<LocalServerEndpointPath>'
```

> [!IMPORTANT]
> La marca de tiempo de último acceso no es una propiedad de la que NTFS hace seguimiento y, por lo tanto, no está visible de forma predeterminada en el Explorador de archivos. No use la marca de tiempo de última modificación en un archivo para comprobar si la directiva de fecha funciona según lo previsto. Esta marca de tiempo solo realiza el seguimiento de escrituras, no de lecturas. Use el cmdlet que se muestra para obtener la marca de tiempo del último acceso para esta evaluación.

> [!WARNING]
> No active la característica de NTFS de seguimiento de la marca de tiempo del último acceso para archivos y carpetas. Esta característica está desactivada de forma predeterminada porque incide en gran medida en el rendimiento. Azure File Sync realizará un seguimiento de las horas de acceso más reciente de forma automática y eficaz, y no utiliza esta característica de NTFS.

Tenga en cuenta que la directiva de espacio disponible en el volumen siempre tiene prioridad, y cuando no haya espacio suficiente en el volumen para conservar todos los archivos que especifica la directiva de fechas, Azure File Sync seguirá organizando por niveles los archivos más antiguos hasta alcanzar el porcentaje de espacio libre en el volumen.

Por ejemplo, supongamos que tiene una directiva de organización por niveles basada en fechas con un valor de 60 días y una directiva de espacio disponible en el volumen del 20 %. Si tras aplicar la directiva de fechas el espacio disponible en el volumen es menor al 20 %, la directiva de espacio libre se pondrá en marcha y anulará la directiva de fechas. Como resultado, se organizarán por niveles más archivos, de forma tal que la cantidad de datos almacenados en el servidor se reduzca quizá de 60 a 45 días de datos. En cambio, la directiva forzará la organización por niveles de los archivos que no coincidan con el intervalo de tiempo, incluso si no ha alcanzado el umbral de espacio libre; es decir, un archivo con una antigüedad de 61 días se organizará por niveles incluso si el volumen está vacío.

<a id="volume-free-space-guidelines"></a>
### <a name="how-do-i-determine-the-appropriate-amount-of-volume-free-space"></a>¿Cómo se puede determinar la cantidad adecuada de espacio disponible del volumen?
La cantidad de datos que se debe mantener en el entorno local viene determinada por una serie de factores: el ancho de banda, el patrón de acceso del conjunto de datos y su presupuesto. Si tiene una conexión con un ancho de banda bajo, desea mantener más datos en su entorno local para asegurarse de que hay un retraso mínimo para los usuarios. En caso contrario, puede basarlo en la tasa de renovación durante un período determinado. Por ejemplo, si sabe que se accede activamente o cambia aproximadamente el 10 % del conjunto de datos de 1 TB cada mes, le interesa mantener en el entorno local 100 GB para no tener que recuperar archivos con frecuencia. Si el volumen es de 2 TB, desea mantener el 5 % (o 100 GB) en el entorno local, lo que significa que el 95 % restante es el porcentaje de espacio disponible del volumen. No obstante, se recomienda que agregue un búfer para tener en cuenta los períodos de mayor renovación, es decir, partir de un porcentaje de espacio disponible del volumen inferior y ajustarlo posteriormente si es necesario. 

Mantener más datos en el entorno local implica menores costos de salida ya que se recuperarán menos archivos de Azure, pero también es necesario mantener una mayor cantidad de almacenamiento local, lo que tiene su propio costo. Una vez que tenga una instancia de Azure File Sync implementada, puede mirar la salida de su cuenta de almacenamiento para valorar aproximadamente si la configuración de espacio disponible del volumen es adecuada para su uso. Suponiendo que la cuenta de almacenamiento contiene solo el archivo punto de conexión de nube de Azure File Sync (es decir, el recurso compartido de sincronización), una salida alta significa que mucho archivos se recuperan desde la nube y debe plantearse aumentar la memoria caché local.

<a id="how-long-until-my-files-tier"></a>
### <a name="ive-added-a-new-server-endpoint-how-long-until-my-files-on-this-server-tier"></a>He agregado un nuevo punto de conexión de servidor. ¿Cuánto tiempo transcurrirá para que los archivos de este servidor se organicen por niveles?

El hecho de que los archivos deban organizarse por niveles según directivas establecidas se evalúa una vez a la hora. Al crear un nuevo punto de conexión de servidor pueden darse dos situaciones:

1. Al agregar un nuevo punto de conexión de servidor, a menudo existen archivos en esa ubicación de servidor. Deben cargarse para que la nube por niveles se pueda iniciar. La directiva de espacio disponible en el volumen no comienza a trabajar hasta que termina la carga inicial de todos los archivos. Pero la directiva de fecha opcional empieza a trabajar archivo a archivo, en cuanto se carga un archivo. El intervalo de una hora también se aplica aquí. 
2. Al agregar un nuevo punto de conexión de servidor, es posible que se conecte una ubicación de servidor vacía a un recurso compartido de archivos de Azure que contiene los datos. Tanto si es para un segundo servidor como durante una situación de recuperación ante desastres, si decide descargar el espacio de nombres y recuperar el contenido durante la descarga inicial en el servidor, después de que el espacio de nombres quede inactivo, se recuperan los archivos en función de la marca de tiempo de última modificación. Se recuperan los archivos que quepan según la directiva de espacio disponible en el volumen y la directiva de fecha opcional.

<a id="is-my-file-tiered"></a>
### <a name="how-can-i-tell-whether-a-file-has-been-tiered"></a>¿Cómo se puede saber si un archivo se ha organizado en niveles?
Hay varias maneras de comprobar si un archivo se ha organizado en niveles en el recurso compartido de archivos de Azure:
    
   *  **Compruebe los atributos de archivo en el archivo.**
     Haga clic con el botón derecho en un archivo, vaya a **Detalles** y desplácese hacia abajo a la propiedad **Atributos**. Un archivo en niveles tendrá los siguientes atributos establecidos:     
        
        | Letra del atributo | Atributo | Definición |
        |:----------------:|-----------|------------|
        | A | Archivar | Indica que se debe realizar la copia de seguridad del archivo con un software de copia de seguridad. Este atributo siempre se establece, independientemente de si el archivo está organizado en niveles o está completamente almacenado en el disco. |
        | P | Archivos dispersos | Indica que el archivo es un archivo disperso. Un archivo disperso es un tipo especializado de archivo que ofrece NTFS para un uso eficaz cuando el archivo en el flujo del disco está en gran parte vacío. Azure File Sync usa archivos dispersos porque un archivo o está completamente organizado en niveles o parcialmente recuperado. En un archivo completamente organizado en niveles, su flujo de archivos se almacena en la nube. En un archivo parcialmente recuperado, esa parte del archivo ya está en el disco. Si un archivo está totalmente recuperado en el disco, Azure File Sync lo convierte de un archivo disperso a un archivo normal. Este atributo solo está establecido en Windows Server 2016 y versiones anteriores.|
        | M | Recuperación del acceso a datos | Indica que los datos del archivo no están totalmente presentes en el almacenamiento local. La lectura del archivo hará que se capture al menos parte del contenido del archivo desde un recurso compartido de archivos de Azure al que está conectado el punto de conexión del servidor. Este atributo solo está establecido en Windows Server 2019. |
        | L | Punto de repetición de análisis | Indica que el archivo tiene un punto de repetición de análisis. Un punto de repetición de análisis es un puntero especial para su uso en un filtro del sistema de archivos. Azure File Sync usa puntos de repetición de análisis a fin de definir para el filtro del sistema de archivos de Azure File Sync (StorageSync.sys) la ubicación en la nube donde está almacenado el archivo. Esto permite el acceso sin problemas. Los usuarios no necesitan saber que se está usando Azure File Sync ni cómo obtener acceso al archivo en el recurso compartido de archivos de Azure. Cuando un archivo completo se recupera, Azure File Sync quita el punto de repetición de análisis del archivo. |
        | O | Sin conexión | Indica que parte del contenido del archivo, o la totalidad, no se ha almacenado en el disco. Cuando un archivo completo se recupera, Azure File Sync quita este atributo. |

        ![Cuadro de diálogo Propiedades de un archivo con la pestaña Detalles seleccionada](media/storage-files-faq/azure-file-sync-file-attributes.png)
        
        Puede ver los atributos de todos los archivos en una carpeta mediante la adición del campo **Atributos** a la presentación de la tabla del Explorador de archivos. Para ello, haga clic con el botón derecho en una columna existente (por ejemplo, **Tamaño**), seleccione **Más** y, después, **Atributos** en la lista desplegable.
        
   * **Use `fsutil` para buscar puntos de repetición de análisis en un archivo.**
       Como se indica en la opción anterior, un archivo organizado en niveles siempre tiene establecido un punto de repetición de análisis. Un puntero de repetición de análisis es un puntero especial para el filtro del sistema de archivos de Azure File Sync (StorageSync.sys). Para comprobar si un archivo tiene un punto de repetición de análisis, en un símbolo del sistema con privilegios elevados o en una ventana de PowerShell, ejecute la utilidad `fsutil`:
    
        ```powershell
        fsutil reparsepoint query <your-file-name>
        ```

        Si el archivo tiene un punto de repetición de análisis, debería ver **Valor de la etiqueta de análisis: 0x8000001e**. Este valor hexadecimal es el valor del punto de repetición de análisis que pertenece a Azure File Sync. La salida también contendrá los datos de repetición de análisis que representan la ruta de acceso al archivo en el recurso compartido de archivos de Azure.

        > [!WARNING]  
        > El comando de la utilidad `fsutil reparsepoint` también puede eliminar un punto de repetición de análisis. No ejecute este comando a menos que el equipo de ingeniería de Azure File Sync se lo indique. Si lo ejecuta, podría producirse una pérdida de datos. 

<a id="afs-recall-file"></a>
### <a name="a-file-i-want-to-use-has-been-tiered-how-can-i-recall-the-file-to-disk-to-use-it-locally"></a>Uno de los archivos que quiero usar se ha organizado en niveles. ¿Cómo puedo recuperarlo en el disco para usarlo de forma local?
La manera más fácil de recuperar un archivo en el disco es abrirlo. El filtro del sistema de archivos de Azure File Sync (StorageSync.sys) descarga sin problemas el archivo desde el recurso compartido de archivos de Azure de forma automática. Para los tipos de archivo que se pueden leer parcialmente, como los archivos multimedia o los archivos .zip, si se abre un archivo, este no se descargará entero.

También puede usar PowerShell para forzar que se recupere un archivo. Esta opción puede resultar útil cuando se quiere recuperar muchos archivos a la vez, como todos los archivos de una carpeta. Abra una sesión de PowerShell en el nodo del servidor donde está instalado Azure File Sync y ejecute los siguientes comandos de PowerShell:
    
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <path-to-to-your-server-endpoint>
```
Parámetros opcionales:
* `-Order CloudTieringPolicy` recupera primero los archivos modificados o a los que se ha accedido más recientemente permitidos por la directiva de niveles actual. 
    * Si la directiva de espacio disponible en el volumen está configurada, se recuperan archivos hasta que se alcanza el valor de esta. Por ejemplo, si el valor de la directiva es 20 %, la recuperación se detiene una vez que el espacio disponible en el volumen alcanza el 20 %.  
    * Si la directiva de espacio disponible en el volumen y la directiva de fecha están configuradas, se recuperan archivos hasta que se alcanzan los valores de estas. Por ejemplo, si el valor de la directiva de espacio disponible es 20 % y el de la directiva de fecha es 7 días, la recuperación se detiene una vez que el espacio disponible en el volumen alcanza el 20 % o si todos los archivos a los que se ha accedido o que se han modificado en los últimos 7 días son locales.
* `-ThreadCount` determina el número de archivos que se pueden recuperar en paralelo.
* `-PerFileRetryCount` determina la frecuencia con que se intentará recuperar un archivo que está bloqueado actualmente.
* `-PerFileRetryDelaySeconds` determina el tiempo en segundos entre los reintentos de recuperación y siempre se debe usar en combinación con el parámetro anterior.

Ejemplo:
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <path-to-to-your-server-endpoint> -ThreadCount 8 -Order CloudTieringPolicy -PerFileRetryCount 3 -PerFileRetryDelaySeconds 10
``` 

> [!Note]  
> - El cmdlet Invoke-StorageSyncFileRecall también se puede usar para mejorar el rendimiento de la descarga de archivos al agregar un nuevo punto de conexión de servidor a un grupo de sincronización existente.  
>- Si el volumen local que hospeda el servidor no tiene suficiente espacio disponible para recuperar todos los datos con niveles, el cmdlet `Invoke-StorageSyncFileRecall` dará error.  

<a id="sizeondisk-versus-size"></a>
### <a name="why-doesnt-the-size-on-disk-property-for-a-file-match-the-size-property-after-using-azure-file-sync"></a>¿Por qué la propiedad *Tamaño en disco* de un archivo no coincide con la propiedad *Tamaño* después de usar Azure File Sync? 
El Explorador de archivos de Windows expone dos propiedades para representar el tamaño de un archivo: **Tamaño** y **Tamaño en disco**. Estas propiedades difieren ligeramente en significado. **Tamaño** representa el tamaño completo del archivo. **Tamaño en disco** representa el tamaño del flujo de archivos que se almacena en el disco. Los valores de estas propiedades pueden diferir por diversos motivos, como la compresión, el uso de la desduplicación de datos o los niveles en la nube con Azure File Sync. Si un archivo está organizado en niveles en un recurso compartido de archivos de Azure, el tamaño en disco será cero, ya que la secuencia de archivos se almacena en el recurso compartido de archivos de Azure, no en el disco. También es posible que un archivo se encuentre parcialmente organizado en niveles (o parcialmente recuperado). En el caso de un archivo parcialmente organizado en niveles, parte del archivo está en el disco. Esto puede ocurrir cuando aplicaciones como reproductores multimedia o utilidades zip leen parcialmente los archivos. 

<a id="afs-force-tiering"></a>
### <a name="how-do-i-force-a-file-or-directory-to-be-tiered"></a>¿Cómo puedo forzar la organización en niveles de un archivo o directorio?

> [!NOTE]
> Cuando se selecciona un directorio para organizarlo por niveles, solo los archivos que se encuentran en ese momento en el directorio se organizan por niveles. Los archivos creados después de ese momento no se organizan por niveles automáticamente.

Cuando se habilita la característica de organización en niveles en la nube, se crean automáticamente niveles en los archivos según la última hora de acceso y de modificación a fin de conseguir el porcentaje de espacio libre en el volumen especificado en el punto de conexión en la nube. Aun así, en ocasiones podría querer forzar un archivo para organizarse en niveles de forma manual. Esto puede resultar útil cuando se guarda un archivo grande que no piensa volver a usar durante mucho tiempo y quiere espacio libre en el volumen ahora para usarlo para otros archivos o carpetas. Puede forzar la organización en niveles con los siguientes comandos de PowerShell:

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncCloudTiering -Path <file-or-directory-to-be-tiered>
```

<a id="afs-image-thumbnail"></a>
### <a name="why-are-my-tiered-files-not-showing-thumbnails-or-previews-in-windows-explorer"></a>¿Por qué mis archivos con niveles no muestran miniaturas ni vistas previas en el Explorador de Windows?
En el caso de los archivos con niveles, las vistas previas y las miniaturas no estarán visibles en el punto de conexión del servidor. Este comportamiento es el esperado, ya que la característica de caché de vistas en miniatura de Windows omite intencionadamente la lectura de archivos con el atributo sin conexión. Con Niveles de la nube habilitado, la lectura de archivos con niveles provocaría su descarga (recuperación).

Este comportamiento no es específico de Azure File Sync, el Explorador de Windows muestra una "X gris" para los archivos que tienen establecido el atributo sin conexión. Verá el icono X al acceder a los archivos a través de SMB. Para obtener una explicación detallada de este comportamiento, consulte [https://blogs.msdn.microsoft.com/oldnewthing/20170503-00/?p=96105](https://blogs.msdn.microsoft.com/oldnewthing/20170503-00/?p=96105).

<a id="afs-tiering-disabled"></a>
### <a name="i-have-cloud-tiering-disabled-why-are-there-tiered-files-in-the-server-endpoint-location"></a>Tengo deshabilitada la nube por niveles, ¿por qué hay archivos por niveles en la ubicación del punto de conexión de servidor?

Existen dos razones por las que pueden existir archivos por niveles en la ubicación del punto de conexión de servidor:

- Al agregar un nuevo punto de conexión de servidor a un grupo de sincronización existente, los metadatos se sincronizan primero con el servidor y, después, los archivos se descargan en el servidor en segundo plano. Los archivos se muestran por niveles hasta que se descargan en local. Para mejorar el rendimiento de la descarga de archivos al agregar un nuevo servidor a un grupo de sincronización, use el cmdlet [Invoke-StorageSyncFileRecall](storage-sync-cloud-tiering.md#afs-recall-file).

- Si se ha habilitado la nube por niveles en el punto de conexión de servidor y luego se ha deshabilitado, los archivos permanecen por niveles hasta que se accede a ellos.


## <a name="next-steps"></a>Pasos siguientes
* [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md)
