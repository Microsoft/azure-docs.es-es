---
title: Guía de solución de problemas de rendimiento de Azure Files
description: Solución de problemas de rendimiento conocidos con los recursos compartidos de archivos de Azure. Detecte las posibles causas y soluciones alternativas asociadas cuando se producen estos problemas.
author: gunjanj
ms.service: storage
ms.topic: troubleshooting
ms.date: 08/24/2020
ms.author: gunjanj
ms.subservice: files
ms.openlocfilehash: fe1460d4353addff1b8e3095cfe06c1fcb3b7bd0
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88782377"
---
# <a name="troubleshoot-azure-files-performance-issues"></a>Solucionar problemas de rendimiento de Azure Files

En este artículo se enumeran algunos problemas habituales relacionados con los recursos compartidos de archivos de Azure. Se proporcionan las posibles causas y soluciones alternativas cuando se producen estos problemas.

## <a name="high-latency-low-throughput-and-general-performance-issues"></a>Alta latencia, rendimiento bajo y problemas de rendimiento general

### <a name="cause-1-share-was-throttled"></a>Causa 1: Se ha limitado el recurso compartido de archivos

Las solicitudes se limitan cuando se alcanzan los límites de IOPS, entradas o salidas de un recurso compartido de archivos. Para conocer los límites de los recursos compartidos de archivos de nivel Estándar y Premium, consulte [Objetivos de escalabilidad de archivos y recursos compartidos de archivos](https://docs.microsoft.com/azure/storage/files/storage-files-scale-targets#file-share-and-file-scale-targets).

Para confirmar si se está limitando el recurso compartido, puede aprovechar las métricas de Azure en el portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione **Todos los servicios** y busque **Métricas**.

1. Seleccione **Métricas**.

1. Seleccione la cuenta de almacenamiento como recurso.

1. Seleccione **Archivo** como espacio de nombres de la métrica.

1. Seleccione **Transacciones** como métrica.

1. Agregue un filtro para **ResponseType** y compruebe si alguna solicitud tiene un código de respuesta de **SuccessWithThrottling** (para SMB) o **ClientThrottlingError** (para REST).

![Opciones de métricas para recursos compartidos de archivos Premium](media/storage-troubleshooting-premium-fileshares/metrics.png)

> [!NOTE]
> Para recibir una alerta si un recurso compartido de archivos se limita, consulte [Creación de una alerta si un recurso compartido de archivos se limita](#how-to-create-an-alert-if-a-file-share-is-throttled).

### <a name="solution"></a>Solución

- Si usa un recurso compartido de archivos Estándar, habilite [Recursos compartidos de archivos grandes](https://docs.microsoft.com/azure/storage/files/storage-files-how-to-create-large-file-share?tabs=azure-portal) en su cuenta de almacenamiento. Los recursos compartidos de archivos grandes admiten hasta 10 000 IOPS por recurso compartido.
- Si usa un recurso compartido de archivos Premium, aumente el tamaño del recurso compartido de archivos aprovisionado para aumentar el límite de IOPS. Para más información, consulte la sección [Descripción del aprovisionamiento de recursos compartidos de archivos prémium](https://docs.microsoft.com/azure/storage/files/storage-files-planning#understanding-provisioning-for-premium-file-shares) en la guía de plan de Azure Files.

### <a name="cause-2-metadatanamespace-heavy-workload"></a>Causa 2: Cargas de trabajo pesadas del espacio de nombres o los metadatos

Si la mayoría de las solicitudes se centran en metadatos (por ejemplo, createfile/openfile/closefile/queryinfo/querydirectory), la latencia será peor en comparación con las operaciones de lectura y escritura.

Para confirmar si la mayoría de las solicitudes se centran en metadatos, puede usar los mismos pasos que antes. Excepto que, en lugar de agregar un filtro para **ResponseType**, deberá agregar un filtro para **Nombre de la API**.

![Filtrar por Nombre de la API en las métricas](media/storage-troubleshooting-premium-fileshares/MetadataMetrics.png)

### <a name="workaround"></a>Solución alternativa

- Compruebe si la aplicación se puede modificar para reducir el número de operaciones de metadatos.
- Agregue un disco duro virtual en el recurso compartido de archivos y móntelo a través de SMB desde el cliente para realizar operaciones de archivos en los datos. Este enfoque funciona para los escenarios de un solo escritor y varios lectores y permite que las operaciones de metadatos sean locales, lo que ofrece un rendimiento similar a un almacenamiento de conexión directa local.

### <a name="cause-3-single-threaded-application"></a>Causa 3: Aplicación de un único subproceso

Si la aplicación que usa el cliente tiene un único subproceso, esto puede dar lugar a un IOPS/rendimiento significativamente menor que el máximo posible en función del tamaño del recurso compartido aprovisionado.

### <a name="solution"></a>Solución

- Aumente el paralelismo de la aplicación aumentando el número de subprocesos.
- Cambie a aplicaciones en las que el paralelismo sea posible. Por ejemplo, para las operaciones de copia, los clientes podrían utilizar AzCopy o RoboCopy desde clientes de Windows o el comando **paralelo** en los clientes de Linux.

## <a name="very-high-latency-for-requests"></a>Latencia muy alta para las solicitudes

### <a name="cause"></a>Causa

La máquina virtual del cliente podría encontrarse en una región diferente de la del recurso compartido de archivos.

### <a name="solution"></a>Solución

- Ejecute la aplicación desde una máquina virtual que se encuentre en la misma región que el recurso compartido de archivos.

## <a name="client-unable-to-achieve-maximum-throughput-supported-by-the-network"></a>El cliente no puede lograr el rendimiento máximo admitido por la red

Una posible causa es la ausencia de compatibilidad multicanal SMB. Actualmente, los recursos compartidos de archivos de Azure solo admiten un único canal, por lo que solo hay una conexión desde la máquina virtual del cliente al servidor. Esta conexión única se fija a un único núcleo en la máquina virtual del cliente, por lo que el rendimiento máximo alcanzable desde una máquina virtual está limitado por un solo núcleo.

### <a name="workaround"></a>Solución alternativa

- Obtener una máquina virtual con un núcleo más grande puede ayudar a mejorar el rendimiento.
- Ejecutar la aplicación cliente desde varias máquinas virtuales aumentará el rendimiento.

- Siempre que sea posible, use las API de REST.

## <a name="throughput-on-linux-clients-is-significantly-lower-when-compared-to-windows-clients"></a>El rendimiento en los clientes de Linux es significativamente menor en comparación con los clientes de Windows.

### <a name="cause"></a>Causa

Se trata de un problema conocido relacionado con la implementación del cliente de SMB en Linux.

### <a name="workaround"></a>Solución alternativa

- Distribuir la carga entre varias máquinas virtuales.
- En la misma máquina virtual, use varios puntos de montaje con la opción **nosharesock** y propague la carga entre estos puntos de montaje.
- En Linux, intente montar la opción **nostrictsync** para evitar forzar el vaciado de SMB en cada llamada a **fsync**. Para Azure Files, esta opción no afecta a la coherencia de los datos, pero puede producir metadatos de archivos obsoletos en la lista de directorios (comando **ls -l**). La consulta directa de metadatos del archivo (comando **stat**) devolverá los metadatos de archivo más recientes.

## <a name="high-latencies-for-metadata-heavy-workloads-involving-extensive-openclose-operations"></a>Latencias altas para cargas de trabajo con muchos metadatos que implican operaciones de apertura y cierre extensas.

### <a name="cause"></a>Causa

Falta de compatibilidad para las concesiones de directorios.

### <a name="workaround"></a>Solución alternativa

- Si es posible, evite las aperturas o cierres excesivos en el mismo directorio en un período de tiempo breve.
- Para las máquinas virtuales de Linux, aumente el tiempo de espera de caché de entrada de directorio especificando **actimeo=\<sec>** como opción de montaje. De forma predeterminada es un segundo, por lo que un valor mayor, como tres o cinco, podría ayudar.
- Para las máquinas virtuales de Linux, actualice el kernel a 4.20 o una versión posterior.

## <a name="low-iops-on-centosrhel"></a>Baja tasa de IOPS en RHEL/CentOS

### <a name="cause"></a>Causa

No se admite una profundidad de IO mayor que uno en RHEL/CentOS.

### <a name="workaround"></a>Solución alternativa

- Actualice a CentOS 8 o RHEL 8.
- Cambie a Ubuntu.

## <a name="slow-file-copying-to-and-from-azure-files-in-linux"></a>Copia de archivos lenta en y desde Azure Files en Linux

Si experimenta una copia de archivos lenta desde y hacia Azure Files, consulte la sección [Copia de archivos lenta en y desde Azure Files en Linux](storage-troubleshoot-linux-file-connection-problems.md#slow-file-copying-to-and-from-azure-files-in-linux) de la guía de solución de problemas de Linux.

## <a name="jitterysaw-tooth-pattern-for-iops"></a>Patrón de vibración/sierra para IOPS

### <a name="cause"></a>Causa

La aplicación cliente supera constantemente el IOPS de base de referencia. Actualmente, no hay ningún suavizado en el lado del servicio para la carga de solicitudes, por lo que, si el cliente supera el IOPS de base de referencia, se verá limitado por el servicio. Dicha limitación puede provocar que el cliente experimente un patrón de IOPS de vibración/sierra. En este caso, el IOPS promedio alcanzable por el cliente podría ser inferior al IOPS de base de referencia.

### <a name="workaround"></a>Solución alternativa

- Reduzca la carga de solicitudes desde la aplicación cliente, de modo que el recurso compartido no se vea limitado.
- Aumente la cuota del recurso compartido para que el recurso compartido no se vea limitado.

## <a name="excessive-directoryopendirectoryclose-calls"></a>Exceso de llamadas DirectoryOpen/DirectoryClose

### <a name="cause"></a>Causa

Si el número de llamadas DirectoryOpen/DirectoryClose se encuentra entre las llamadas de API principales y no espera que el cliente realice esa cantidad de llamadas, podría tratarse de un problema con el antivirus instalado en la máquina virtual del cliente de Azure.

### <a name="workaround"></a>Solución alternativa

- La [Actualización de la plataforma para Windows de abril](https://support.microsoft.com/help/4052623/update-for-windows-defender-antimalware-platform) incluye una corrección para este problema.

## <a name="file-creation-is-slower-than-expected"></a>La creación de archivos es más lenta de lo esperado

### <a name="cause"></a>Causa

Las cargas de trabajo que se basan en la creación de un gran número de archivos no verán una diferencia importante entre el rendimiento de los recursos compartidos de archivos Premium y recursos compartidos de archivos estándar.

### <a name="workaround"></a>Solución alternativa

- Ninguno.

## <a name="slow-performance-from-windows-81-or-server-2012-r2"></a>Rendimiento lento desde Windows 8.1 o Server 2012 R2

### <a name="cause"></a>Causa

Latencia mayor a la esperada al acceder a Azure Files para cargas de trabajo intensivas de IO.

### <a name="workaround"></a>Solución alternativa

- Instale la [revisión](https://support.microsoft.com/help/3114025/slow-performance-when-you-access-azure-files-storage-from-windows-8-1) disponible.

## <a name="how-to-create-an-alert-if-a-file-share-is-throttled"></a>Creación de una alerta si un recurso compartido de archivos se limita

1. Vaya a la **cuenta de almacenamiento** en **Azure Portal**.
2. En la sección Supervisión, haga clic en **Alertas** y, después, haga clic en **+ Nueva regla de alertas**.
3. Haga clic en **Editar recurso**, seleccione el **tipo de recurso de archivo** para la cuenta de almacenamiento y, a continuación, haga clic en **Listo**. Por ejemplo, si el nombre de la cuenta de almacenamiento es contoso, seleccione el recurso contoso/archivo.
4. Haga clic en **Seleccionar condición** para agregar una condición.
5. Verá una lista de señales admitidas para la cuenta de almacenamiento, seleccione la métrica **Transacciones**.
6. En la hoja **Configurar lógica de señal**, haga clic en la lista desplegable **Nombre de la dimensión** y seleccione **Tipo de respuesta**.
7. Haga clic en el menú desplegable **Valores de la dimensión** y seleccione **SuccessWithThrottling** (para SMB) o **ClientThrottlingError** (para REST).

  > [!NOTE]
  > Si el valor de la dimensión SuccessWithThrottling o ClientThrottlingError no aparece en la lista, significa que el recurso no se ha limitado. Para agregar el valor de dimensión, haga clic en **Agregar valor personalizado** junto a la lista desplegable **Valores de dimensión**, escriba **SuccessWithThrottling** o **ClientThrottlingError**, haga clic en **Aceptar** y, a continuación, repita el paso 7.

8. Haga clic en la lista desplegable **Nombre de la dimensión** y seleccione **Recurso compartido de archivos**.
9. Haga clic en la lista desplegable **Valores de dimensión** y seleccione los recursos compartidos de archivos en los que desea generar alertas.

  > [!NOTE]
  > Si el recurso compartido de archivos es un recurso compartido de archivos estándar, seleccione **Todos los valores actuales y futuros**. El menú desplegable de valores de dimensión no mostrará los recursos compartidos de archivos porque las métricas por recurso compartido no están disponibles para los recursos compartidos de archivos estándar. Las alertas de limitación de los recursos compartidos de archivos estándar se desencadenarán si algún recurso compartido de archivos de la cuenta de almacenamiento está limitado y la alerta no identificará qué recurso compartido de archivos se ha limitado. Dado que las métricas por recurso compartido no están disponibles para los recursos compartidos de archivos estándar, se recomienda tener un recurso compartido de archivos por cada cuenta de almacenamiento.

10. Defina los **parámetros de alerta** (umbral, operador, granularidad de agregación y frecuencia de evaluación) y haga clic en **Listo**.

  > [!TIP]
  > Si usa un umbral estático, el gráfico de métricas puede ayudar a determinar un valor de umbral razonable si el recurso compartido de archivos se está limitando actualmente. Si usa un umbral dinámico, el gráfico de métricas mostrará los umbrales calculados según los datos recientes.

11. Haga clic en **Seleccionar el grupo de acciones** para agregar un **grupo de acciones** (correo electrónico, SMS, etc.) a la alerta, para lo que puede seleccionar un grupo de acciones existente o crear uno nuevo.
12. Rellene los **detalles de la alerta**, como el **nombre de la regla de alertas**, la **descripción** y la **gravedad**.
13. Haga clic en **Crear regla de alerta** para crear la alerta.

Para obtener más información sobre cómo configurar alertas en Azure Monitor, consulte [Introducción sobre las alertas en Microsoft Azure]( https://docs.microsoft.com/azure/azure-monitor/platform/alerts-overview).

## <a name="see-also"></a>Consulte también
* [Solución de problemas de Azure Files en Windows](storage-troubleshoot-windows-file-connection-problems.md)
* [Solución de problemas de Azure Files en Linux](storage-troubleshoot-linux-file-connection-problems.md)
* [Preguntas más frecuentes (P+F) sobre Azure Files](storage-files-faq.md)
