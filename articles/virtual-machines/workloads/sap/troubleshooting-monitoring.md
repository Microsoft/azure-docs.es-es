---
title: Supervisión de SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Supervisión de SAP HANA en Azure (Instancias grandes).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d168ff36e63cc07dc1b334ae7817bfed45b4fcd0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87042274"
---
# <a name="how-to-monitor-sap-hana-large-instances-on-azure"></a>Procedimiento para supervisar SAP HANA en Azure (instancias grandes)

SAP HANA en Azure (Instancias grandes) no se diferencia de cualquier otra implementación de IaaS: necesita supervisar lo que están haciendo el sistema operativo y la aplicación y cómo consumen las aplicaciones los siguientes recursos:

- CPU
- Memoria
- Ancho de banda de red
- Espacio en disco

Con Azure Virtual Machines, debe averiguar si las clases de recursos indicadas anteriormente serán suficientes o se agotarán. A continuación se proporcionan más detalles sobre cada una de las clases:

**Consumo de recursos de CPU:** se fuerza la relación definida por SAP para una determinada carga de trabajo con HANA con el fin de asegurarse de que haya suficientes recursos de CPU disponibles para trabajar con los datos almacenados en la memoria. No obstante, puede haber casos en los que HANA consuma una gran cantidad de CPU al ejecutar consultas debido a que falten índices o problemas similares. Esto significa que debe supervisar el consumo de los recursos de CPU por parte de la unidad de Instancias grandes de HANA, así como los recursos de CPU utilizados por servicios específicos de HANA.

**Consumo de memoria:** es importante supervisarlo desde dentro y desde fuera de HANA (en la unidad). Dentro de HANA, supervise cómo los datos utilizan la memoria asignada a HANA con el fin de que se mantengan dentro de las directrices de tamaño obligatorias de SAP. También puede supervisar el consumo de memoria en el nivel de Instancias grandes para asegurarse de que el software instalado no perteneciente a HANA no utiliza demasiada memoria y compite por la memoria con HANA.

**Ancho de banda de red:** la puerta de enlace de red virtual de Azure tiene una limitación en el ancho de banda de los datos que se transfieren a la red virtual de Azure, por lo que es útil supervisar los datos recibidos por todas las máquinas virtuales de Azure dentro de una red virtual para averiguar si está cerca de los límites de la SKU de puerta de enlace de Azure que seleccionó. En la unidad de Instancias grandes de HANA, tiene sentido supervisar también el tráfico de red de entrada y de salida, así como realizar un seguimiento de los volúmenes que se administran a lo largo del tiempo.

**Espacio en disco:** normalmente su uso aumenta a lo largo del tiempo. Las causas más comunes son el aumento del volumen de datos, la ejecución de copias de seguridad de los registros de transacciones, el almacenamiento de los archivos de seguimiento y la realización de instantáneas de almacenamiento. Por lo tanto, es importante supervisarlo y administrar el espacio en disco asociado a la unidad de Instancias grandes de HANA.

En las **SKU de tipo II** de las instancias grandes de HANA, el servidor viene ya con las herramientas de diagnóstico del sistema cargadas. Puede usar estas herramientas de diagnóstico para realizar la comprobación de mantenimiento del sistema. Ejecute el siguiente comando para generar el archivo de registro de comprobación de mantenimiento en /var/log/health_check.
```
/opt/sgi/health_check/microsoft_tdi.sh
```
Cuando trabaje con el equipo de Soporte técnico de Microsoft para solucionar un problema, también se le pedirá que proporcione los archivos de registro mediante el uso de estas herramientas de diagnóstico. Puede comprimir el archivo con el comando siguiente.
```
tar  -czvf health_check_logs.tar.gz /var/log/health_check
```

**Pasos siguientes**

- Consulte [Procedimiento para supervisar SAP HANA en Azure (instancias grandes)](./hana-monitor-troubleshoot.md).
