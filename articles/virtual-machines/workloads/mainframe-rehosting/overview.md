---
title: Rehospedaje del sistema central en Azure Virtual Machines
description: Rehospede las cargas de trabajo del sistema central, como los sistemas basados en IBM Z con máquinas virtuales (VM) en Microsoft Azure.
services: virtual-machines-linux
documentationcenter: ''
author: njray
manager: edprice
editor: edprice
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: multiple
ms.openlocfilehash: d754f27a1510d9db0837eabb96b3a7cf75f76fc1
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835534"
---
# <a name="mainframe-rehosting-on-azure-virtual-machines"></a>Rehospedaje del sistema central en Azure Virtual Machines

Las cargas de trabajo de migración de entornos de sistema central a la nube le permiten modernizar su infraestructura y a menudo reducir los costos. Muchas cargas de trabajo pueden transferirse a Azure con solo leves cambios en el código, como la actualización de los nombres de las bases de datos.

En general, el término *sistema central* hace referencia a un sistema informático de gran tamaño. En concreto, la inmensa mayoría de sistemas centrales que se usan son servidores IBM Z o sistemas compatibles con complementos de IBM que ejecutan MVS, DOS, VSE, OS/390 o z/OS.

Una máquina virtual (VM) de Azure se utiliza para aislar y administrar los recursos de una aplicación específica en una sola instancia. Los sistemas centrales, como IBM z/OS utilizan particiones lógicas (LPAR) para este propósito. Un sistema central puede usar una partición LPAR para una región CICS con programas COBOL asociados, y una partición LPAR independiente para la base de datos IBM Db2. Una [aplicación típica de n niveles en Azure](/azure/architecture/reference-architectures/n-tier/n-tier-sql-server) implementa VM de Azure en una red virtual que se puede dividir en subredes para cada nivel.

Las VM de Azure pueden ejecutar entornos de emulación de sistemas centrales y compiladores que admitan escenarios de migración mediante lift-and-shift. El desarrollo y las pruebas suelen estar entre las primeras cargas de trabajo para migrar desde un sistema central a un entorno de desarrollo y pruebas de Azure. Los componentes de servidor habituales que puede emular son procesamiento de transacciones en línea (OLTP), lotes y sistemas de ingesta de datos, como se muestra en la figura siguiente.

![Los entornos de emulación en Azure le permiten ejecutar sistemas basados en z/OS.](media/01-overview.png)

Algunas cargas de trabajo del sistema central se pueden migrar a Azure con relativa facilidad, mientras que otras pueden rehospedarse en Azure mediante una solución de asociado. Para obtener instrucciones detalladas sobre cómo elegir una solución de asociado, busque ayuda en [Migración del sistema central en Azure Migration Center](https://azure.microsoft.com/migration/mainframe/).

## <a name="mainframe-migration"></a>Migración del sistema central

¿Rehospedar, recompilar, reemplazar o retirar? ¿IaaS o PaaS? Para determinar la estrategia de migración adecuada para su aplicación del sistema central, consulte [Migración del sistema central](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/overview) en el Centro de arquitectura de Azure.

## <a name="micro-focus-rehosting-platform"></a>Plataforma de rehospedaje de Micro Focus

Micro Focus Enterprise Server es una de las mayores plataformas de rehospedaje de sistemas centrales disponibles. Se puede usar para ejecutar las cargas de trabajo de z/OS en una plataforma x86 menos costosa en Azure.

Primeros pasos:

- [Instalación de Enterprise Server y Enterprise Developer en Azure](./microfocus/set-up-micro-focus-azure.md)
- [Configuración de CICS BankDemo para Enterprise Developer en Azure](./microfocus/demo.md)
- [Ejecución de Enterprise Server en un contenedor de Docker en Azure](./microfocus/run-enterprise-server-container.md)


## <a name="tmaxsoft-openframe-on-azure"></a>TmaxSoft OpenFrame en Azure

TmaxSoft OpenFrame es una solución de rehospedaje de sistemas centrales popular que se usa en escenarios de migración mediante lift-and-shift. Un entorno OpenFrame en Azure es adecuado para cargas de trabajo de desarrollo, demostraciones, pruebas o producción.

Primeros pasos:

- [Introducción a TmaxSoft OpenFrame](./tmaxsoft/get-started.md)
- [Descargar el libro electrónico](https://azure.microsoft.com/resources/install-tmaxsoft-openframe-on-azure/)

## <a name="ibm-zdt-120"></a>IBM zD&T 12.0

El entorno de desarrollo y pruebas IBM Z (IBM zD&T) configura un entorno que no es de producción en Azure, que se puede usar para desarrollo, pruebas y demostraciones de aplicaciones basadas en z/OS.

El entorno de emulación en Azure puede hospedar diferentes tipos de instancias de Z a través de distribuciones controladas por los desarrolladores de aplicaciones (ADCD). Puede ejecutar zD&T Personal Edition, zD&T Parallel Sysplex y zD&T Enterprise Edition en Azure y Azure Stack.

Primeros pasos:

- [Configuración de IBM zD&T 12.0 en Azure](./ibm/install-ibm-z-environment.md)
- [Configuración de ADCD en zD&T](./ibm/demo.md)

## <a name="ibm-db2-purescale-on-azure"></a>IBM DB2 pureScale en Azure

El entorno IBM DB2 pureScale proporciona un clúster de base de datos para Azure. No es idéntico al entorno original, pero ofrece escalabilidad y una disponibilidad y una escala similares a IBM DB2 para z/OS que se ejecuta en una configuración de Parallel Sysplex.

Para empezar, consulte [IBM DB2 pureScale en Azure](../../linux/ibm-db2-purescale-azure.md).

## <a name="considerations"></a>Consideraciones

Al migrar cargas de trabajo del sistema central a la infraestructura como servicio (IaaS) de Azure, puede elegir entre varios tipos de recursos informáticos a petición y escalables, incluidas las VM de Azure. Azure ofrece una variedad de VM [Linux](../../linux/overview.md) y [Windows](../../windows/overview.md).

### <a name="compute"></a>Proceso

La eficacia de proceso de Azure se compara favorablemente con la capacidad de un sistema central. Si está pensando en mover una carga de trabajo del sistema central a Azure, compare la métrica del sistema central de un millón de instrucciones por segundo (MIPS) a las CPU virtuales. 

Aprenda a [mover el proceso del sistema central a Azure](./concepts/mainframe-compute-azure.md).

### <a name="high-availability-and-failover"></a>Alta disponibilidad y conmutación por error

Azure ofrece contratos de nivel de servicio (SLA) basados en compromisos. La disponibilidad de varios nueves es el valor predeterminado, y los SLA se pueden optimizar con replicación local o basada en la ubicación de los servicios. En el [SLA de Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/) completo se explica la disponibilidad garantizada de Azure como un conjunto.

Con IaaS de Azure como una VM, funciones del sistema específicas proporcionan compatibilidad con la conmutación por error; por ejemplo, instancias de clústeres de conmutación por error y conjuntos de disponibilidad. Cuando se usan recursos de la plataforma como servicio (PaaS) de Azure, la plataforma controla automáticamente la conmutación por error. Algunos ejemplos son [Azure SQL Database](../../../azure-sql/database/sql-database-paas-overview.md) y [Azure Cosmos DB](../../../cosmos-db/introduction.md).

### <a name="scalability"></a>Escalabilidad

Los sistemas centrales suelen escalarse verticalmente, mientras que los entornos de nube se escalan horizontalmente. Azure ofrece una variedad de tamaños de [Linux](../../sizes.md) y [Windows](../../sizes.md) para satisfacer sus necesidades. La nube también se puede escalar o reducir verticalmente para ajustarse a las especificaciones exactas del usuario. La eficacia de proceso, el almacenamiento y los servicios se [escalan](/azure/architecture/best-practices/auto-scaling) a petición en un modelo de facturación basado en uso.

### <a name="storage"></a>Storage

En la nube, tiene una variedad de opciones de almacenamiento flexibles y escalables, y solo paga por lo que necesita. [Azure Storage](../../../storage/common/storage-introduction.md) ofrece un almacén de objetos que se puede escalar de forma masiva destinado a objetos de datos, un servicio de sistema de archivos para la nube, un almacén de mensajería para mensajería confiable y un almacén NoSQL. En cuanto a las máquinas virtuales, los discos administrados y no administrados proporcionan un almacenamiento de disco persistente y seguro.

Aprenda a [mover el almacenamiento del sistema central a Azure](./concepts/mainframe-storage-azure.md).

### <a name="backup-and-recovery"></a>Copia de seguridad y recuperación

El mantenimiento de un sitio de recuperación ante desastres propio puede ser una propuesta costosa. Azure tiene opciones fáciles de implementar y muy rentables para realizar la [copia de seguridad](../../../backup/backup-overview.md), la [recuperación](../../../site-recovery/site-recovery-overview.md) y la [redundancia](../../../storage/common/storage-redundancy.md) a nivel local o regional, o por medio de la redundancia geográfica.

## <a name="azure-government-for-mainframe-migrations"></a>Azure Government para las migraciones del sistema central

A muchas entidades del sector público les encantaría mover sus aplicaciones del sistema central a una plataforma flexible más moderna. Microsoft Azure Government es una instancia separada físicamente de la plataforma global de Microsoft Azure: empaqueta para sistemas gubernamentales federales, estatales y locales. Proporciona excelentes servicios de seguridad, protección y cumplimiento específicamente para las agencias gubernamentales de Estados Unidos y sus asociados.

Azure Government ha obtenido una aprobación inicial P-ATO (Provisional Authority to Operate) para el nivel de gran impacto de FedRAMP para los servicios que necesitan este tipo de entorno.

Para empezar, descargue [Microsoft Azure Government cloud for mainframe applications](https://azure.microsoft.com/resources/microsoft-azure-government-cloud-for-mainframe-applications/en-us/) (Nube de Microsoft Azure Government para aplicaciones de sistema central).

## <a name="next-steps"></a>Pasos siguientes

Solicite a nuestros [asociados](partner-workloads.md) que le ayuden a migrar o rehospedar sus aplicaciones del sistema central. 

Consulte también:

- [Notas del producto sobre los temas de sistemas centrales](mainframe-white-papers.md)
- [Migración del sistema central](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/overview)
- [Solución de problemas](../../troubleshooting/index.yml)
- [Demystifying mainframe to Azure migration](https://azure.microsoft.com/resources/demystifying-mainframe-to-azure-migration/) (Desmitificación de la migración del sistema central a Azure)

<!-- INTERNAL LINKS -->
[microfocus-get-started]: /microfocus/get-started.md
[microfocus-setup]: /microfocus/set-up-micro-focus-azure.md
[microfocus-demo]: /microfocus/demo.md
[ibm-get-started]: /ibm/get-started.md
[ibm-install-z]: /ibm/install-ibm-z-environment.md
[ibm-demo]: /ibm/demo.md