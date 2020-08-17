---
title: 'Diagnóstico de problemas de rendimiento de gráficos en Escritorio remoto: Azure'
description: Este artículo describe cómo usar contadores de gráficos de RemoteFX en sesiones del protocolo de escritorio remoto para diagnosticar problemas de rendimiento con gráficos en Windows Virtual Desktop.
author: Heidilohr
ms.topic: troubleshooting
ms.date: 05/23/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: dc5c6499fa47a1e32a517032d5cc2a97b3f2677f
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88007173"
---
# <a name="diagnose-graphics-performance-issues-in-remote-desktop"></a>Diagnóstico de problemas de rendimiento de gráficos en Escritorio remoto

Para diagnosticar problemas de calidad con las sesiones remotas, se han proporcionado contadores en la sección Gráficos RemoteFX de Monitor de rendimiento. Este artículo le ayuda a identificar y corregir cuellos de botella de rendimiento relacionados con gráficos durante sesiones del Protocolo de escritorio remoto (RDP) mediante estos contadores.

## <a name="find-your-remote-session-name"></a>Búsqueda del nombre de la sesión remota

Necesitará el nombre de la sesión remota para identificar los contadores de rendimiento de gráficos. Siga las instrucciones de esta sección para identificar su instancia de cada contador.

1. Abra el símbolo del sistema de Windows desde la sesión remota.
2. Ejecute el comando **qwinsta** y busque el nombre de la sesión.
    - Si la sesión se hospeda en una máquina virtual de sesión múltiple: La instancia de cada contador tiene como sufijo el mismo número que el sufijo de su nombre de sesión, como "rdp-tcp 37".
    - Si la sesión se hospeda en una máquina virtual compatible con unidades de procesamiento gráfico virtuales (vGPU): La instancia de cada contador se almacena en el servidor en lugar de en la máquina virtual. Las instancias de contador incluyen el nombre de la máquina virtual en lugar del número que hay en el nombre de la sesión, como "Win8 Enterprise VM".

>[!NOTE]
> Aunque los contadores tienen RemoteFX en el nombre, también incluyen gráficos de escritorio remoto en escenarios de vGPU.

## <a name="access-performance-counters"></a>Acceso a los contadores de rendimiento

Una vez determinado el nombre de la sesión remota, siga estas instrucciones para recopilar los contadores de rendimiento de los gráficos de RemoteFX para su sesión remota.

1. Seleccione **Inicio** > **Herramientas administrativas** > **Monitor de rendimiento**.
2. En el cuadro de diálogo **Monitor de rendimiento**, expanda **Herramientas de supervisión**, seleccione **Monitor de rendimiento**y, a continuación, seleccione **Agregar**.
3. En el cuadro de diálogo **Agregar contadores** de la lista **Contadores disponibles**, expanda la sección Gráficos RemoteFX.
4. Seleccione los contadores que desea supervisar.
5. En la lista **Instancias del objeto seleccionado**, seleccione las instancias específicas que quiere supervisar para los contadores seleccionados y, a continuación, seleccione **Agregar**. Para seleccionar todas las instancias de contadores disponibles, seleccione **Todas las instancias**.
6. Después de agregar los contadores, seleccione **Aceptar**.

Los contadores de rendimiento seleccionados aparecerán en la pantalla Monitor de rendimiento.

>[!NOTE]
>Cada sesión activa en un host tiene su propia instancia de cada contador de rendimiento.

## <a name="diagnose-issues"></a>Diagnosticar problemas

Por lo general, los problemas de rendimiento relacionados con gráficos se dividen en cuatro categorías:

- Baja velocidad de fotogramas
- Pausas aleatorias
- Alta latencia de entrada
- Baja calidad de fotograma

### <a name="addressing-low-frame-rate-random-stalls-and-high-input-latency"></a>Solución de problemas con baja velocidad de fotogramas, pausas aleatorias y alta latencia de entrada

En primer lugar, compruebe el contador Fotogramas de salida/segundo. Este contador mide el número de fotogramas que se ponen a disposición del cliente. Si este valor es inferior al del contador Fotogramas de entrada/segundo, se estarán omitiendo fotogramas. Para identificar el cuello de botella, use los contadores Fotogramas omitidos/segundo.

Hay tres tipos de contadores Fotogramas omitidos/segundo:

- Fotogramas omitidos/segundo: recursos de servidor insuficientes
- Fotogramas omitidos/segundo: recursos de red insuficientes
- Fotogramas omitidos/segundo: recursos de cliente insuficientes

Un valor alto de cualquiera de estos contadores implica que el problema está relacionado con el recurso que supervisa el contador. Por ejemplo, si el cliente no descodifica y presenta fotogramas a la misma velocidad que el servidor los proporciona, el contador Fotogramas omitidos/segundo: recursos de cliente insuficientes tendrá un valor elevado.

Si el contador Fotogramas de salida/segundo coincide con el contador Fotogramas de entrada/segundo, pero aún experimenta retrasos o pausas inusuales, el problema puede ser del promedio de tiempo de codificación. La codificación es un proceso sincrónico que se produce en el servidor en un escenario de sesión única (vGPU) y en la máquina virtual en un escenario de sesión múltiple. El promedio de tiempo de codificación debe ser inferior a 33 ms. Si el promedio de tiempo de codificación es inferior a 33 ms pero aún tiene problemas de rendimiento, puede haber un problema con la aplicación o el sistema operativo que está usando.

Para más información acerca de cómo diagnosticar problemas relacionados con la aplicación, consulte [Contadores de rendimiento de retraso de entrada del usuario](/windows-server/remote/remote-desktop-services/rds-rdsh-performance-counters/).

Dado que RDP admite un promedio de tiempo de codificación de 33 ms, también admite una velocidad de fotogramas de entrada de hasta 30 fotogramas por segundo. Tenga en cuenta que 33 ms es la velocidad de fotogramas máxima permitida. En muchos casos, la velocidad de fotogramas experimentada por el usuario será menor, en función de la frecuencia con la que el origen proporciona un fotograma a RDP. Por ejemplo, tareas como ver un vídeo requieren una velocidad de fotogramas de entrada completa de 30 fotogramas por segundo, pero las tareas con un proceso menos intensivo, como la edición de un documento con poca frecuencia, dan como resultado un valor de fotogramas de entrada/segundo mucho menor sin degradar la calidad de la experiencia del usuario.

### <a name="addressing-poor-frame-quality"></a>Solución de problemas de baja calidad de los fotogramas

Utilice el contador Calidad de fotograma para diagnosticar problemas de calidad de fotograma. Este contador expresa la calidad del fotograma de salida como un porcentaje de la calidad del fotograma de origen. La pérdida de calidad puede deberse a RemoteFX o puede ser inherente al origen de los gráficos. Si RemoteFX es la causa de la pérdida de calidad, el problema puede ser una falta de recursos de red o de servidor para enviar contenido de una mayor fidelidad.

## <a name="mitigation"></a>Mitigación

Si los recursos del servidor son la causa del cuello de botella, pruebe uno de los siguientes enfoques para mejorar el rendimiento:

- Reduzca el número de sesiones por host.
- Aumente la memoria y los recursos de proceso en el servidor.
- Baje la resolución de la conexión.

Si los recursos de red son la causa del cuello de botella, pruebe uno de los siguientes enfoques para mejorar la disponibilidad de la red por sesión:

- Reduzca el número de sesiones por host.
- Utilice una red con mayor ancho de banda.
- Baje la resolución de la conexión.

Si los recursos del cliente son la causa del cuello de botella, pruebe uno de los siguientes enfoques para mejorar el rendimiento:

- Instale el cliente de Escritorio remoto más reciente.
- Aumente la memoria y los recursos de proceso en la máquina cliente.

> [!NOTE]
> Actualmente no se admite el contador Fotogramas de origen/segundo. Por el momento, el contador Fotogramas de salida/segundo siempre se mostrará como 0.

## <a name="next-steps"></a>Pasos siguientes

- Para crear una máquina virtual de Azure optimizada para GPU, consulte [Configuración de la aceleración de la unidad de procesamiento de gráficos (GPU) para Windows Virtual Desktop](configure-vm-gpu.md).
- Para obtener información general sobre cómo solucionar problemas y las pistas de escalación, consulte [Información general sobre solución de problemas, comentarios y soporte técnico](troubleshoot-set-up-overview.md).
- Para más información sobre el servicio, consulte [Entorno de Windows Virtual Desktop](environment-setup.md).
