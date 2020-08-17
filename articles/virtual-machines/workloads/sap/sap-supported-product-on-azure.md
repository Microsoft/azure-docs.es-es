---
title: 'SAP en Azure: Qué software de SAP es compatible con Azure'
description: Explica qué software de SAP es compatible para su implementación en Azure
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: SAP
ms.assetid: d7c59cc1-b2d0-4d90-9126-628f9c7a5538
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 04/21/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 217d5b060ca1b7d32cd334049db9b7c68b7949a0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87065080"
---
# <a name="what-sap-software-is-supported-for-azure-deployments"></a>Qué software de SAP es compatible para su implementación en Azure
En este artículo se describe cómo puede averiguar qué software de SAP es compatible para su implementación en Azure y cuáles son las versiones de sistemas operativos o de DBMS necesarias.

Para evaluar si el software de SAP actual es compatible, y qué versiones de sistemas operativos y de DBMS son compatibles con el software de SAP en Azure, necesitará acceso a:

- Notas sobre compatibilidad con SAP
- Matriz de disponibilidad de productos SAP



## <a name="general-restrictions-for-sap-workload"></a>Restricciones generales para la carga de trabajo de SAP
Los servicios de IaaS de Azure que se pueden usar para la carga de trabajo de SAP se limitan al hardware x86-64 o x64. No hay ofertas basadas en CPU de Sparc o de Power que se apliquen a la carga de trabajo de SAP. Los clientes que ejecutan sus aplicaciones en sistemas operativos propietarios de arquitecturas de hardware, como los mainframes o AS400 de IBM, o en los que se estén utilizando los sistemas operativos HP-UX, Solaris o AIX, deben cambiar sus aplicaciones SAP, incluido DBMS, por uno de los sistemas operativos siguientes:

- Windows Server de 64 bits para la plataforma x86-64
- SUSE Linux de 64 bits para la plataforma x86-64
- Red Hat Linux de 64 bits para la plataforma x86-64
- Oracle Linux de 64 bits para la plataforma x86-64

En combinación con el software de SAP, no son compatibles otras versiones de sistemas operativos ni distribuciones de Linux. Más adelante en el documento se indican los detalles exactos de versiones y casos específicos.


## <a name="you-start-here"></a>Comience aquí
El punto de partida es la [Nota sobre compatibilidad con SAP n.º 1928533](https://launchpad.support.sap.com/#/notes/1928533). A medida que recorra esta nota de compatibilidad con SAP de arriba a abajo, aparecerán varias áreas de software y máquinas virtuales compatibles.

En la primera sección se enumeran los requisitos mínimos para las versiones de sistemas operativos que son compatibles con el software de SAP en máquinas virtuales de Azure en general. Si no alcanza los requisitos mínimos y ejecuta versiones anteriores de estos sistemas operativos, debe actualizar la versión del sistema operativo a una versión mínima o incluso a versiones más recientes. Es cierto que Azure, en general, es compatible con versiones anteriores de algunos de estos sistemas operativos. Sin embargo, las restricciones o versiones mínimas enumeradas se basan en las pruebas y cualificaciones ejecutadas, y no se van a ampliar a versiones más antiguas. 


> [!NOTE]
>Hay algunos tipos específicos de máquinas virtuales, instancias grandes de HANA o cargas de trabajo de SAP que van a requerir versiones más recientes del sistema operativo. Casos como este se van a mencionar en todo el documento. Este tipo de casos se documentan claramente en las notas sobre compatibilidad con SAP u otras publicaciones de SAP.

En la sección siguiente se enumeran las plataformas de SAP generales que son compatibles con las versiones admitidas y, lo que es aún más importante, los kernels de SAP compatibles. Se muestran las pilas de NetWeaver/ABAP o Java que son compatibles Y que necesitan versiones mínimas de kernel. Las pilas de ABAP más recientes son compatibles en Azure, pero no necesitan versiones mínimas de kernel, ya que los cambios de Azure se han implementado desde el inicio del desarrollo de las pilas más recientes.

Debe comprobar:

- Si las aplicaciones de SAP que está ejecutando están cubiertas por las versiones mínimas indicadas. Si no es así, debe definir una nueva versión de destino, comprobar la matriz de disponibilidad de productos SAP, y qué combinaciones de compilaciones de sistemas operativos y de DBMS son compatibles con la nueva versión de destino. Así, podrá elegir la versión correcta del sistema operativo y de DBMS
- Si necesita actualizar los kernels de SAP para moverse a Azure
- Si necesita actualizar los paquetes de soporte técnico de SAP. Especialmente, los paquetes de soporte técnico base que puedan ser necesarios para los casos en los que sea preciso pasar a una versión de DBMS más reciente


En la sección siguiente se incluyen más detalles sobre otras versiones de productos SAP y de DBMS compatibles con SAP en Azure para Windows y Linux. 

> [!NOTE]
> Se eligen cuidadosamente las distintas versiones mínimas de DBMS y es posible que no siempre reflejen el espectro completo de versiones de DBMS que admiten los distintos proveedores de DBMS en Azure en general. Se han tenido en cuenta muchas consideraciones relacionadas con la carga de trabajo de SAP para definir estas versiones mínimas. No se está realizando ningún esfuerzo para probar y cualificar versiones anteriores de DBMS. 

> [!NOTE]
> Las versiones mínimas enumeradas representan la versión más antigua de sistemas operativos y de bases de datos. Es muy recomendable usar las versiones más recientes de sistemas operativos y de bases de datos. En muchos casos, las versiones más recientes de sistemas operativos y de bases de datos han tenido en cuenta el caso de uso de la ejecución en la nube pública y han adaptado el código para optimizar la ejecución en la nube pública o, más específicamente, en Azure.

## <a name="oracle-dbms-support"></a>Compatibilidad de DBMS de Oracle
Las versiones de sistemas operativos, de DBMS de Oracle y la funcionalidad de Oracle que se admiten en Azure aparecen concretamente en la [Nota sobre compatibilidad con SAP n.º 2039619](https://launchpad.support.sap.com/#/notes/2039619). Lo principal de esta nota se puede resumir del modo siguiente:

- La versión mínima de Oracle admitida en las máquinas virtuales de Azure que están certificadas para NetWeaver es la versión 2 revisión 3 de Oracle 11g (11.2.0.4).
- Como sistemas operativos invitados, solo se pueden están cualificados Windows y Oracle Linux. En la nota se muestran las versiones exactas de los sistemas operativos y las versiones mínimas de DBMS relacionadas.
- La compatibilidad de Oracle Linux se extiende también al cliente de DBMS de Oracle. Esto significa que todos los componentes de SAP, como las instancias de cuadro de diálogo de ABAP o la pila de Java, se deben ejecutar también en Oracle Linux. Solo los componentes de SAP dentro de este sistema SAP que no se conecten a DBMS de Oracle podrán ejecutar un sistema operativo diferente de Linux.
- No se admite RAC de Oracle. 
- En algunos de los casos se admite ASM de Oracle. Los detalles aparecen en la nota.
- Los sistemas SAP que no son de Unicode solo se admiten con servidores de aplicaciones que ejecutan el sistema operativo invitado de Windows. El sistema operativo invitado de DBMS puede ser Oracle Linux o Windows. La razón de esta restricción es evidente al comprobar la matriz de disponibilidad de productos (PAM) SAP. Para Oracle Linux, SAP nunca ha publicado kernels de SAP que no sean de Unicode

Como conoce las versiones de DBMS que son compatibles con la infraestructura de Azure de destino, debe comprobar la matriz de disponibilidad de productos SAP sobre si se admiten las versiones de sistemas operativos y DBMS requeridos con las versiones de productos SAP que desea ejecutar. 


## <a name="sap-hana-support"></a>Compatibilidad con SAP HANA
En Azure hay dos servicios que se pueden usar para ejecutar la base de datos de HANA:

- Azure Virtual Machines
- [Instancias grandes de HANA](./hana-overview-architecture.md)

Para ejecutar SAP HANA, SAP tiene que satisfacer unas necesidades de infraestructura cada más grandes para ejecutar NetWeaver u otras aplicaciones de SAP y DBMS. En consecuencia, un número menor de máquinas virtuales de Azure es apto para ejecutar DBMS de SAP HANA. La lista de la infraestructura de Azure compatible con SAP HANA se puede encontrar en el conocido como [Directorio de hardware de SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). 

> [!NOTE]
> Las unidades que comienzan por la letra "S" son unidades de [instancias grandes de HANA](./hana-overview-architecture.md). 

> [!NOTE]
> SAP no tiene ninguna certificación específica que dependa de las versiones principales de SAP HANA. Al contrario de la opinión común, la columna **Escenario de certificación** en las [plataformas IaaS certificadas de HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure), la columna **no tiene ninguna instrucción sobre el certificado de versión principal o secundaria de HANA**. Debe suponer que todas las unidades enumeradas que se pueden usar para HANA 1.0 y HANA 2.0, siempre que las versiones de sistema operativo certificadas para las unidades específicas, también se admitan en las versiones de HANA 1.0. 

Para utilizar SAP HANA, es posible que se apliquen versiones mínimas de sistemas operativos distintas de las de los casos de NetWeaver en general. Debe comprobar los sistemas operativos compatibles para cada unidad individualmente, ya que pueden variar. Para ello, haga clic en cada unidad. Aparecerán más detalles. Uno de los detalles enumerado son los distintos sistemas operativos compatibles con esta unidad específica.

> [!NOTE]
> Las unidades de instancias grandes de HANA de Azure son más restrictivas con los sistemas operativos compatibles que las máquinas virtuales de Azure. Por otra parte, las máquinas virtuales de Azure pueden aplicar versiones operativas más recientes como versiones mínimas. Esto resulta especialmente cierto en algunas de las unidades de máquinas virtuales más grandes que requieren cambios en los kernels de Linux

Si conoce el sistema operativo admitido para la infraestructura de Azure, debe consultar la [Nota sobre compatibilidad con SAP n.º 2235581](https://launchpad.support.sap.com/#/notes/2235581) para conocer las versiones y los niveles de revisión de SAP HANA exactos que se admiten con las unidades de Azure de destino. 

> [!IMPORTANT]
> El paso de comprobación de las versiones y los niveles de revisión de SAP HANA exactos compatibles es muy importante. En muchos casos, la compatibilidad de una determinada versión del sistema operativo depende de un nivel específico de revisión de los ejecutables de SAP HANA.

Como conoce cuáles son las versiones de HANA específicas que puede ejecutar en la infraestructura de Azure de destino, debe comprobar la matriz de disponibilidad de productos SAP para averiguar si existen restricciones con las versiones de productos SAP que admiten las versiones de HANA que ha filtrado.


## <a name="certified-azure-vms-and-hana-large-instance-units-and-business-transaction-throughput"></a>Máquinas virtuales y unidades de instancias grandes de HANA certificadas de Azure, y procesamiento de transacciones empresariales
Además de evaluar las versiones de sistemas operativos compatibles, las versiones de DBMS y las versiones de software de SAP compatibles dependientes para las unidades de infraestructura de Azure, debe cualificar estas unidades por procesamiento de transacciones empresariales, lo que SAP expresa en la unidad "SAP". Todo el ajuste de tamaño de SAP depende de los cálculos de SAP. Al evaluar los sistemas SAP existentes, normalmente puede, con la ayuda del proveedor de infraestructura, calcular los SAPS de las unidades. Tanto para la capa de DBMS como para la capa de la aplicación. En otros casos en los que se crea una nueva funcionalidad, un ejercicio de ajuste de tamaño con SAP puede revelar los números de SAPS necesarios para la capa de la aplicación y la capa de DBMS. Como proveedor de infraestructura, Microsoft tiene la obligación de proporcionar la caracterización del procesamiento de SAP de las distintas unidades que tienen la certificación NetWeaver y/o HANA.

En el caso de las máquinas virtuales de Azure, estos números de procesamiento de SAPS se documentan en la [Nota sobre compatibilidad con SAP n.º 1928533](https://launchpad.support.sap.com/#/notes/1928533). En el caso de las unidades de instancias grandes de HANA de Azure, los números de procesamiento de SAPS se documentan en la [Nota sobre compatibilidad con SAP n.º 2316233](https://launchpad.support.sap.com/#/notes/2316233)

En la [Nota sobre compatibilidad con SAP n.º 1928533](https://launchpad.support.sap.com/#/notes/1928533), se aplican las siguientes observaciones:

- **Para las máquinas virtuales de Azure de la serie M y las máquinas virtuales de Azure de la serie Mv2, se aplican versiones de sistemas operativos mínimas distintas que para otros tipos de máquinas virtuales de Azure**. El requisito de las versiones más recientes de sistemas operativos se basa en los cambios que los distintos proveedores de sistemas operativos han tenido que proporcionar en sus versiones de sistemas operativos para permitir que sus sistemas operativos se ejecuten en tipos de máquinas virtuales de Azure específicos u optimizar el rendimiento y el procesamiento de la carga de trabajo de SAP en estos tipos de máquinas virtuales
- Hay dos tablas que especifican diferentes tipos de máquinas virtuales. La segunda tabla especifica el procesamiento de SAPS para los tipos de máquina virtual de Azure que solo admiten Azure Standard Storage. No se admite la implementación de DBMS en las unidades especificadas en la segunda tabla de la nota


## <a name="other-sap-products-supported-on-azure"></a>Otros productos SAP admitidos en Azure
En general, se da por supuesto que, con el estado de las nubes de hiperescala como Azure, la mayor parte del software de SAP se debería ejecutar sin problemas funcionales en Azure. No obstante y al contrario que la visualización de nubes privadas, SAP sigue admitiendo los diferentes productos SAP explícitamente para los distintos proveedores de nubes de hiperescala. En consecuencia, existen diferentes notas sobre compatibilidad con SAP que indican la compatibilidad de Azure con diferentes productos SAP. 

En el caso de Business Objects BI Platform, la [Nota sobre compatibilidad con SAP n.º 2145537](https://launchpad.support.sap.com/#/notes/2145537) proporciona una lista de los productos SAP Business Objects compatibles con Azure. Si hay preguntas sobre los componentes o las combinaciones de versiones de software y versiones de sistemas operativos que aparentemente no aparecen o no son compatibles y que son más recientes que las versiones mínimas enumeradas, debe abrir una solicitud de soporte técnico de SAP en el componente sobre el que consulta la compatibilidad.

En el caso de Business Objects Data Services, la [Nota sobre compatibilidad con SAP n.º 22288344](https://launchpad.support.sap.com/#/notes/2288344) explica la compatibilidad mínima de SAP Data Services que se ejecuta en Azure. 

> [!NOTE]
> Como se indica en la nota sobre compatibilidad con SAP, debe consultar SAP PAM para identificar el nivel de paquete de soporte técnico correcto que se admitirá en Azure.

La compatibilidad de SAP Datahub/Vora con Azure Kubernetes Services (AKS) se detalla en la [Nota sobre compatibilidad con SAP n.º 2464722](https://launchpad.support.sap.com/#/notes/2464722).

La compatibilidad con SAP BPC 10.1 SP08 se describe en la [Nota sobre compatibilidad con SAP n.º 2451795](https://launchpad.support.sap.com/#/notes/2451795).

La compatibilidad con la plataforma de comercio SAP Hybris en Azure se detalla en la [documentación de Hybris](https://help.sap.com/viewer/a74589c3a81a4a95bf51d87258c0ab15/1811/en-US/8c71300f866910149b40c88dfc0de431.html). A partir del DBMS compatible con la plataforma de comercio SAP Hybris, se muestra como:

- SQL Server y Oracle en la plataforma del sistema operativo Windows. Se aplican las mismas versiones mínimas que para SAP NetWeaver. Para más información, consulte la [nota de soporte técnico de SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533)
- SAP HANA en Red Hat y SUSE Linux. Se requieren tipos de máquina virtual certificados para SAP HANA, tal como se documentó anteriormente en [este documento](#sap-hana-support). La plataforma de comercio SAP (Hybris) se considera una carga de trabajo de OLTP
- Base de datos de SQL Azure a partir de la versión 1811 de la plataforma de comercio SAP (Hybris)




## <a name="next-steps"></a>Pasos siguientes
Consulte los pasos siguientes en [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver](./planning-guide.md).
