---
title: Cómo reemplazar la infraestructura de cintas
description: Aprenda la forma en que Azure Backup proporciona una semántica similar a la cinta que le permite hacer copias de seguridad y restaurar datos en Azure
ms.topic: conceptual
ms.date: 04/30/2017
ms.openlocfilehash: 695cc2644521384527ecd871f3613a078e987aa7
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378447"
---
# <a name="move-your-long-term-storage-from-tape-to-the-azure-cloud"></a>Traslado del almacenamiento a largo plazo de la cinta a la nube de Azure

Los clientes de Microsoft Azure Backup y System Center Data Protection Manager pueden:

* Realizar copias de seguridad de datos en las programaciones que mejor se adapten a las necesidades de su organización.
* Conservar los datos de copia de seguridad durante períodos más largos.
* Hacer a Azure partícipe de sus necesidades de retención a largo plazo (en lugar de la cinta).

Este artículo explica cómo los clientes pueden habilitar las directivas de copia de seguridad y retención. Los clientes que utilicen las cintas para abordar sus necesidades de retención a largo plazo ahora tienen una alternativa viable y eficaz con la disponibilidad de esta característica. La característica está habilitada en la versión más reciente de Azure Backup (que está disponible [aquí](https://aka.ms/azurebackup_agent)). Debe actualizarse a los clientes System Center DPM, como mínimo, DPM 2012 R2 UR5 para usar DPM con el servicio de Azure Backup.

## <a name="what-is-the-backup-schedule"></a>¿Cuál es la programación de copia de seguridad?

La programación de copia de seguridad indica la frecuencia de la operación de copia de seguridad. Por ejemplo, la configuración de la pantalla siguiente indica que las copias de seguridad se tomarán diariamente a las 18:00 h y a medianoche.

![Programación diaria](./media/backup-azure-backup-cloud-as-tape/dailybackupschedule.png)

Los clientes también pueden programar una copia de seguridad semanal. Por ejemplo, la configuración de la pantalla siguiente indica que las copias de seguridad se realizarán alternando entre cada domingo y miércoles a las 09:30 h y a la 01: 00 h.

![Programación semanal](./media/backup-azure-backup-cloud-as-tape/weeklybackupschedule.png)

## <a name="what-is-the-retention-policy"></a>¿Qué es la directa de retención?

La directiva de retención especifica el tiempo durante el que debe almacenarse la copia de seguridad. En vez de especificar solo una "directiva" para todos los puntos de copia de seguridad, los clientes pueden especificar directivas de retención diferentes en función de cuándo se realizó la copia de seguridad. Por ejemplo, un punto de copia de seguridad diario (que sirve como punto de recuperación operativo) podría conservarse durante 90 días. El punto de copia de seguridad realizado al final de cada trimestre con fines de auditoría se conserva durante un período prolongado.

![Directiva de retención](./media/backup-azure-backup-cloud-as-tape/retentionpolicy.png)

El número total de "puntos de retención" especificado en esta directiva es de 90 (puntos diarios) + 40 (uno cada trimestre durante 10 años) = 130.

## <a name="example--putting-both-together"></a>Ejemplo: reunir ambos

![Pantalla de ejemplo](./media/backup-azure-backup-cloud-as-tape/samplescreen.png)

1. **Directiva de retención diaria**: Lasas copias de seguridad realizadas diariamente se almacenan durante siete días.
2. **Directiva de retención semanal**: Las copias de seguridad realizadas a medianoche y el sábado a las 18:00 h se conservan durante cuatro semanas.
3. **Directiva de retención mensual**: Las copias de seguridad realizadas a medianoche y a las 18:00 h del último sábado de cada mes se conservan durante 12 meses.
4. **Directiva de retención anual**: Las copias de seguridad realizadas a medianoche el último sábado de cada mes de marzo se conservan durante 10 años.

El número total de "puntos de retención" (desde los que un cliente puede restaurar datos) en el diagrama anterior se calcula de la forma siguiente:

* 2 puntos por día durante 7 días = 14 puntos de recuperación
* 2 puntos por día durante 4 semanas = 8 puntos de recuperación
* 2 puntos por mes durante 12 meses = 24 puntos de recuperación
* 1 punto por año durante 10 años = 10 puntos de recuperación

El número total de puntos de recuperación es 56.

> [!NOTE]
> Con Azure Backup, puede crear hasta 9999 puntos de recuperación por instancia protegida. Una instancia protegida es un equipo, un servidor (físico o virtual) o una carga de trabajo que realiza copias de seguridad en Azure.
>

## <a name="advanced-configuration"></a>Configuración avanzada

Al seleccionar **Modificar** en la pantalla anterior, los clientes tienen más flexibilidad para especificar programaciones de retención.

![Ventana Modificar directiva](./media/backup-azure-backup-cloud-as-tape/modify.png)

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre Azure Backup, vea

* [Introducción a Azure Backup](./backup-overview.md)
* [Probar Azure Backup](./backup-windows-with-mars-agent.md)
