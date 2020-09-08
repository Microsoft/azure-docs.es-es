---
title: Soluciones de Oracle en máquinas virtuales de Azure | Microsoft Docs
description: Obtenga información sobre las configuraciones admitidas y las limitaciones de las imágenes de máquinas virtuales de Oracle en Microsoft Azure.
services: virtual-machines-linux
documentationcenter: ''
author: rgardler
manager: ''
tags: azure-resource-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 05/12/2020
ms.author: rogardle
ms.openlocfilehash: c22c5fa9c1a1680734a8cf5d8f4079e8c7b5c5f7
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723423"
---
# <a name="oracle-vm-images-and-their-deployment-on-microsoft-azure"></a>Imágenes de máquinas virtuales de Oracle y su implementación en Microsoft Azure

En este artículo se incluye información sobre soluciones de Oracle basadas en las imágenes de máquinas virtuales que publica Oracle en Azure Marketplace. Si está interesado en soluciones de aplicaciones entre nubes con Oracle Cloud Infrastructure, vea [Soluciones de aplicaciones de Oracle que integran Microsoft Azure y Oracle Cloud Infrastructure](oracle-oci-overview.md).

Para obtener una lista de imágenes disponibles actualmente, ejecute el siguiente comando:

```azurecli-interactive
az vm image list --publisher oracle -o table --all
```

A partir de junio de 2020 están disponibles las siguientes imágenes:

```bash
Offer                   Publisher    Sku                     Urn                                                          Version
----------------------  -----------  ----------------------  -----------------------------------------------------------  -------------
oracle-database-19-3    Oracle       oracle-db-19300         Oracle:oracle-database-19-3:oracle-db-19300:19.3.0           19.3.0
Oracle-Database-Ee      Oracle       12.1.0.2                Oracle:Oracle-Database-Ee:12.1.0.2:12.1.20170220             12.1.20170220
Oracle-Database-Ee      Oracle       12.2.0.1                Oracle:Oracle-Database-Ee:12.2.0.1:12.2.20180725             12.2.20180725
Oracle-Database-Ee      Oracle       18.3.0.0                Oracle:Oracle-Database-Ee:18.3.0.0:18.3.20181213             18.3.20181213
Oracle-Database-Se      Oracle       12.1.0.2                Oracle:Oracle-Database-Se:12.1.0.2:12.1.20170220             12.1.20170220
Oracle-Database-Se      Oracle       12.2.0.1                Oracle:Oracle-Database-Se:12.2.0.1:12.2.20180725             12.2.20180725
Oracle-Database-Se      Oracle       18.3.0.0                Oracle:Oracle-Database-Se:18.3.0.0:18.3.20181213             18.3.20181213
Oracle-Linux            Oracle       6.10                    Oracle:Oracle-Linux:6.10:6.10.00                             6.10.00
Oracle-Linux            Oracle       6.8                     Oracle:Oracle-Linux:6.8:6.8.0                                6.8.0
Oracle-Linux            Oracle       6.8                     Oracle:Oracle-Linux:6.8:6.8.20190529                         6.8.20190529
Oracle-Linux            Oracle       6.9                     Oracle:Oracle-Linux:6.9:6.9.0                                6.9.0
Oracle-Linux            Oracle       6.9                     Oracle:Oracle-Linux:6.9:6.9.20190529                         6.9.20190529
Oracle-Linux            Oracle       7.3                     Oracle:Oracle-Linux:7.3:7.3.0                                7.3.0
Oracle-Linux            Oracle       7.3                     Oracle:Oracle-Linux:7.3:7.3.20190529                         7.3.20190529
Oracle-Linux            Oracle       7.4                     Oracle:Oracle-Linux:7.4:7.4.1                                7.4.1
Oracle-Linux            Oracle       7.4                     Oracle:Oracle-Linux:7.4:7.4.20190529                         7.4.20190529
Oracle-Linux            Oracle       7.5                     Oracle:Oracle-Linux:7.5:7.5.1                                7.5.1
Oracle-Linux            Oracle       7.5                     Oracle:Oracle-Linux:7.5:7.5.2                                7.5.2
Oracle-Linux            Oracle       7.5                     Oracle:Oracle-Linux:7.5:7.5.20181207                         7.5.20181207
Oracle-Linux            Oracle       7.5                     Oracle:Oracle-Linux:7.5:7.5.20190529                         7.5.20190529
Oracle-Linux            Oracle       7.6                     Oracle:Oracle-Linux:7.6:7.6.2                                7.6.2
Oracle-Linux            Oracle       7.6                     Oracle:Oracle-Linux:7.6:7.6.3                                7.6.3
Oracle-Linux            Oracle       7.6                     Oracle:Oracle-Linux:7.6:7.6.4                                7.6.4
Oracle-Linux            Oracle       77                      Oracle:Oracle-Linux:77:7.7.1                                 7.7.1
Oracle-Linux            Oracle       77                      Oracle:Oracle-Linux:77:7.7.2                                 7.7.2
Oracle-Linux            Oracle       77                      Oracle:Oracle-Linux:77:7.7.3                                 7.7.3
Oracle-Linux            Oracle       77                      Oracle:Oracle-Linux:77:7.7.4                                 7.7.4
Oracle-Linux            Oracle       77                      Oracle:Oracle-Linux:77:7.7.5                                 7.7.5
Oracle-Linux            Oracle       77-ci                   Oracle:Oracle-Linux:77-ci:7.7.01                             7.7.01
Oracle-Linux            Oracle       77-ci                   Oracle:Oracle-Linux:77-ci:7.7.02                             7.7.02
Oracle-Linux            Oracle       77-ci                   Oracle:Oracle-Linux:77-ci:7.7.03                             7.7.03
Oracle-Linux            Oracle       78                      Oracle:Oracle-Linux:78:7.8.01                                7.8.01
Oracle-Linux            Oracle       8                       Oracle:Oracle-Linux:8:8.0.2                                  8.0.2
Oracle-Linux            Oracle       8-ci                    Oracle:Oracle-Linux:8-ci:8.0.11                              8.0.11
Oracle-Linux            Oracle       81                      Oracle:Oracle-Linux:81:8.1.0                                 8.1.0
Oracle-Linux            Oracle       81                      Oracle:Oracle-Linux:81:8.1.2                                 8.1.2
Oracle-Linux            Oracle       81-ci                   Oracle:Oracle-Linux:81-ci:8.1.0                              8.1.0
Oracle-Linux            Oracle       81-gen2                 Oracle:Oracle-Linux:81-gen2:8.1.11                           8.1.11
Oracle-Linux            Oracle       ol77-ci-gen2            Oracle:Oracle-Linux:ol77-ci-gen2:7.7.1                       7.7.1
Oracle-Linux            Oracle       ol77-gen2               Oracle:Oracle-Linux:ol77-gen2:7.7.01                         7.7.01
Oracle-Linux            Oracle       ol77-gen2               Oracle:Oracle-Linux:ol77-gen2:7.7.02                         7.7.02
Oracle-Linux            Oracle       ol78-gen2               Oracle:Oracle-Linux:ol78-gen2:7.8.11                         7.8.11
Oracle-WebLogic-Server  Oracle       Oracle-WebLogic-Server  Oracle:Oracle-WebLogic-Server:Oracle-WebLogic-Server:12.1.2  12.1.2
weblogic-122130-jdk8u3  Oracle       owls-122130-8u131-ol73  Oracle:weblogic-122130-jdk8u131-ol73:owls-122130-8u131-ol7   1.1.6
weblogic-122130-jdk8u4  Oracle       owls-122130-8u131-ol74  Oracle:weblogic-122130-jdk8u131-ol74:owls-122130-8u131-ol7   1.1.1
weblogic-122140-jdk8u6  Oracle       owls-122140-8u251-ol76  Oracle:weblogic-122140-jdk8u251-ol76:owls-122140-8u251-ol7   1.1.1
weblogic-141100-jdk116  Oracle       owls-141100-11_07-ol76  Oracle:weblogic-141100-jdk11_07-ol76:owls-141100-11_07-ol7   1.1.1
weblogic-141100-jdk8u6  Oracle       owls-141100-8u251-ol76  Oracle:weblogic-141100-jdk8u251-ol76:owls-141100-8u251-ol7   1.1.1
```

Estas imágenes se consideran "Traiga su propia licencia" y, por tanto, solo se le cobrarán los costos de proceso, almacenamiento y red derivados de la ejecución de una máquina virtual.  Se supone que tiene la licencia correcta para usar software Oracle y que cuenta con un contrato de soporte técnico actual con Oracle. Oracle ha garantizado la movilidad de licencias del entorno local a Azure. Para obtener información detallada sobre la movilidad de licencias, vea la nota publicada de [Oracle y Microsoft](https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html).

Los individuos pueden elegir también basar sus soluciones en una imagen personalizada que crean desde cero en Azure o cargar una imagen personalizada de su entorno local.

## <a name="oracle-database-vm-images"></a>Imágenes de máquinas virtuales de base de datos de Oracle

Oracle admite la ejecución de Oracle Database 12.1 y versiones posteriores de edición Standard y Enterprise en Azure en imágenes de máquina virtual basadas en Oracle Linux.  A fin de conseguir el mejor rendimiento para las cargas de trabajo de producción de Oracle Database en Azure, asegúrese de ajustar correctamente el tamaño de la imagen de máquina virtual y usar discos administrados SSD Premium o SSD Ultra. Para obtener instrucciones sobre cómo conseguir poner en marcha rápidamente una instancia de Oracle Database en Azure mediante la imagen de máquina virtual publicada de Oracle, [pruebe el tutorial de inicio rápido de Oracle Database](oracle-database-quick-create.md).

### <a name="attached-disk-configuration-options"></a>Opciones de configuración de discos conectados

Los discos conectados se basan en el servicio de almacenamiento de blobs de Azure. Teóricamente, cada disco estándar puede realizar aproximadamente 500 operaciones de entrada/salida por segundo (IOPS). Nuestra oferta de disco premium es preferible para las cargas de trabajo de base de datos de alto rendimiento y puede alcanzar las 5000 IOPS por disco. Se puede usar un único disco si con ello cubre sus necesidades de rendimiento, pero se puede mejorar el rendimiento de IOPS efectivo si se usan varios discos conectados, se reparten los datos de la base de datos entre ellos y, después, se usa Oracle Automatic Storage Management (ASM). Consulte la [información general sobre Oracle Automatic Storage](https://www.oracle.com/technetwork/database/index-100339.html) para obtener más información específica de Oracle ASM. Para ver un ejemplo de cómo instalar y configurar Oracle ASM en una máquina virtual Linux de Azure, vea el tutorial [Instalación y configuración de Oracle Automated Storage Management](configure-oracle-asm.md).

### <a name="shared-storage-configuration-options"></a>Opciones de configuración de almacenamiento compartido

Azure NetApp Files se diseñó para satisfacer los requisitos principales de la ejecución de cargas de trabajo de alto rendimiento, como bases de datos, en la nube y proporciona lo siguiente:

- Un servicio de almacenamiento nativo NFS compartido para ejecutar cargas de trabajo de Oracle mediante un cliente NFS nativo de máquina virtual o de Oracle dNFS.
- Niveles de rendimiento escalables que reflejan el rango real de demandas de IOPS.
- Baja latencia.
- Alta disponibilidad, durabilidad y capacidad de administración a escala, que normalmente exigen las cargas de trabajo empresariales críticas (como SAP y Oracle).
- Copia de seguridad y recuperación rápidas y eficaces, para lograr el RTO y RPO del Acuerdo de Nivel de Servicio más agresivo.

Estas capacidades son posibles porque Azure NetApp Files está basado en sistemas all-flash de NetApp® ONTAP® que se ejecutan en el entorno del centro de datos de Azure como un servicio nativo de Azure. El resultado es una tecnología de almacenamiento de base de datos idónea que se puede aprovisionar y utilizar como otras opciones de almacenamiento de Azure. Consulte la [documentación sobre Azure NetApp Files](../../../azure-netapp-files/index.yml) para obtener más información sobre cómo implementar y acceder a los volúmenes NFS de Azure NetApp Files. Consulte [Guía de procedimientos recomendados para la implementación de Oracle en Azure con Azure NetApp Files](https://www.netapp.com/us/media/tr-4780.pdf) para conocer los procedimientos recomendados a fin de manejar una base de datos de Oracle en Azure NetApp Files.

## <a name="licensing-oracle-database--software-on-azure"></a>Licencias para Oracle Database y software en Azure

Microsoft Azure es un entorno de nube autorizado para ejecutar Oracle Database. La tabla de factores de núcleos de Oracle no se aplica al conceder licencias a las bases de datos de Oracle en la nube. En su lugar, al usar las máquinas virtuales con la tecnología Hyper-Threading habilitada para las bases de datos de la edición Enterprise, cuente dos vCPU como equivalentes a una licencia de procesador de Oracle si el hyperthreading está habilitado (como se indica en el documento de directiva). Puede consultar los detalles de la directiva [aquí](http://www.oracle.com/us/corporate/pricing/cloud-licensing-070579.pdf).
Las bases de datos de Oracle suelen requerir una mayor cantidad de memoria y E/S. Por esta razón, se recomienda usar [máquinas virtuales optimizadas para memoria](../../sizes-memory.md) en estas cargas de trabajo. A fin de optimizar aún más las cargas de trabajo, se recomiendan las [CPU virtuales restringidas en núcleos](../../constrained-vcpu.md) para las cargas de trabajo Oracle Database que requieran gran cantidad de memoria, almacenamiento y ancho de banda de E/S, pero no un gran número de núcleos.

Al migrar el software y las cargas de trabajo de Oracle del entorno local a Microsoft Azure, Oracle ofrece movilidad de licencias, tal como se indica en las [Preguntas más frecuentes sobre Oracle en Azure](https://www.oracle.com/cloud/technologies/oracle-azure-faq.html).

## <a name="oracle-real-application-cluster-oracle-rac"></a>Oracle Real Application Cluster (Oracle RAC)

Oracle Real Application Clusters (Oracle RAC) se ha diseñado para mitigar el error de un nodo individual en una configuración local de clúster de varios nodos. Se basa en dos tecnologías locales que no son nativas para los entornos de nube pública de hiperescala: multidifusión y disco compartido de red. Si la solución de base de datos requiere Oracle RAC en Azure, se necesita software de terceros para habilitar estas tecnologías. Para obtener más información sobre Oracle RAC, vea la [página de FlashGrid SkyCluster](https://www.flashgrid.io/oracle-rac-in-azure/).

## <a name="high-availability-and-disaster-recovery-considerations"></a>Consideraciones sobre la alta disponibilidad y la recuperación ante desastres

Si usa bases de datos de Oracle en Azure, es responsable de implementar una solución de recuperación ante desastres y alta disponibilidad para evitar los tiempos de inactividad.

La alta disponibilidad y recuperación ante desastres para Oracle Database Enterprise Edition (sin basarse en Oracle RAC) se puede lograr en Azure mediante [Data Guard, Active Data Guard](https://www.oracle.com/database/technologies/high-availability/dataguard.html) u [Oracle Golden Gate](https://www.oracle.com/technetwork/middleware/goldengate), con dos bases de datos en dos máquinas virtuales independientes. Ambas máquinas virtuales deben estar en la misma [red virtual](https://azure.microsoft.com/documentation/services/virtual-network/) para asegurarse de que pueden acceder entre sí a través de la dirección IP privada persistente.  Además, recomendamos situar las máquinas virtuales en el mismo conjunto de disponibilidad para permitir a Azure colocarlas en dominios de error y de actualización independientes. Si quiere tener redundancia geográfica, puede configurar las dos bases de datos para que se repliquen entre dos regiones diferentes y conectar las dos instancias con una instancia de VPN Gateway.

En el tutorial [Implementación de Data Guard de Oracle en Azure](configure-oracle-dataguard.md) se describe el procedimiento de configuración básico en Azure.  

Con Data Guard de Oracle, se puede lograr alta disponibilidad con una base de datos principal en una máquina virtual, una base de datos secundaria (de reserva) en otra máquina virtual y la replicación unidireccional establecida entre ellos. El resultado es un acceso de lectura a la copia de la base de datos. Con Oracle GoldenGate, puede configurar la replicación bidireccional entre las dos bases de datos. Para obtener más información sobre cómo configurar una solución de alta disponibilidad para bases de datos mediante estas herramientas, vea la documentación de [Active Data Guard](https://www.oracle.com/database/technologies/high-availability/dataguard.html) y [GoldenGate](https://docs.oracle.com/goldengate/1212/gg-winux/index.html) en el sitio web de Oracle. Si se necesita acceso de lectura-escritura a la copia de la base de datos, puede usar [Active Data Guard de Oracle](https://www.oracle.com/uk/products/database/options/active-data-guard/overview/index.html).

En el tutorial [Implementación de Oracle GoldenGate en Azure](configure-oracle-golden-gate.md) se describe el procedimiento de configuración básico en Azure.

Además de tener una solución de alta disponibilidad y recuperación ante desastres diseñada en Azure, debe contar con una estrategia de copia de seguridad en vigor para restaurar la base de datos. En el tutorial [Copia de seguridad y recuperación de una instancia de Oracle Databas](oracle-backup-recovery.md) se describe el procedimiento básico para establecer una copia de seguridad coherente.

## <a name="support-for-jd-edwards"></a>Compatibilidad con JD Edwards

De acuerdo con la nota de compatibilidad de Oracle con el [id. de documento 2178595.1](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=573435677515785&id=2178595.1&_afrWindowMode=0&_adf.ctrl-state=o852dw7d_4), las versiones de JD Edwards EnterpriseOne 9.2 y superiores se admiten en **cualquier oferta de nube pública** que satisfaga sus `Minimum Technical Requirements` (MTR) específicos.  Debe crear imágenes personalizadas que cumplan sus especificaciones de MTR para la compatibilidad con el sistema operativo y la aplicación de software.

## <a name="oracle-weblogic-server-virtual-machine-offers"></a>Ofertas de máquina virtual de Oracle WebLogic Server

Oracle y Microsoft están colaborando para llevar WebLogic Server a Azure Marketplace en forma de una colección de ofertas de Aplicación de Azure.  Estas ofertas se describen en el artículo [Aplicaciones de Azure de Oracle WebLogic Server](oracle-weblogic.md).

### <a name="oracle-weblogic-server-virtual-machine-images"></a>Imágenes de máquina virtual de Oracle WebLogic Server

- **La agrupación en clústeres solo se admite en Enterprise Edition.** Tiene licencia para usar la agrupación en clústeres de WebLogic solo cuando utiliza la versión Enterprise Edition de Oracle WebLogic Server. No use la agrupación en clústeres con Oracle WebLogic Server Standard Edition.
- **Tampoco se admite la multidifusión UDP.** Azure admite la unidifusión UDP, pero no admite la multidifusión ni la difusión. Oracle WebLogic Server es capaz de depender de las capacidades de unidifusión UDP de Azure. Para obtener mejores resultados dependiendo de la unidifusión UDP, recomendamos que el tamaño del clúster WebLogic se mantenga estático o con no más de 10 servidores administrados.
- **Oracle WebLogic Server espera que los puertos públicos y privados sean los mismos para el acceso T3 (por ejemplo, al usar Enterprise JavaBeans).** Considere un escenario de varios niveles, en el que una aplicación de capa de servicio (EJB) se ejecuta en un clúster de Oracle WebLogic Server compuesto por dos o más máquinas virtuales, en una red virtual llamada *SLWLS*. El nivel de cliente está en una subred diferente de la misma red virtual, ejecutando un programa Java simple que intenta llamar a EJB en el nivel de servicio. Dado que es necesario equilibrar la carga de la capa de servicio, debe crearse un punto de conexión público con equilibrio de carga para las máquinas virtuales en el clúster de Oracle WebLogic Server. Si el puerto privado que especifique es diferente del puerto público (por ejemplo, 7006:7008), se producirá un error como el siguiente:

```bash
   [java] javax.naming.CommunicationException [Root exception is java.net.ConnectException: t3://example.cloudapp.net:7006:

   Bootstrap to: example.cloudapp.net/138.91.142.178:7006' over: 't3' got an error or timed out]
```

   Esto se debe a que, para cualquier acceso remoto T3, Oracle WebLogic Server espera que el puerto con equilibrador de carga y el puerto del servidor WebLogic administrado sean el mismo. En el caso anterior, el cliente tiene acceso al puerto 7006 (el puerto del equilibrador de carga) y el servidor administrado está escuchando en 7008 (el puerto privado). Esta restricción se aplica solo al acceso T3, no a HTTP.

   Para evitar este problema, use una de las soluciones siguientes:

- Use los mismos números de puerto públicos y privados para los puntos de conexión de carga equilibrada dedicados al acceso a T3.
- Incluya el parámetro siguiente de JVM al iniciar Oracle WebLogic Server:

```bash
   -Dweblogic.rjvm.enableprotocolswitch=true
```

Para obtener información relacionada, vea el artículo de KB **860340.1** en <https://support.oracle.com>.

- **Agrupación en clústeres dinámica y limitaciones de equilibrio de carga.** Supongamos que quiere usar un clúster dinámico en Oracle WebLogic Server y exponerlo mediante un único punto de conexión con equilibrio de carga público en Azure. Esto puede hacerse siempre que use un número de puerto fijo para cada uno de los servidores administrados (no asignados dinámicamente a partir de un rango) y no inicie más servidores administrados que máquinas esté siguiendo el administrador. Es decir, que no haya más de un servidor administrado por máquina virtual. Si la configuración provoca que se inicien más servidores Oracle WebLogic que máquinas virtuales (es decir, donde varias instancias de Oracle WebLogic Server compartirán la misma máquina virtual), entonces no será posible para más de una de esas instancias de servidores de Oracle WebLogic Server establecer un enlace a un número de puerto determinado. Los demás de esa máquina virtual generan un error.

   Si configura el servidor de administración a fin de asignar automáticamente números de puerto exclusivos para los servidores administrados, el equilibrio de carga no será posible porque Azure no admite la asignación de un único puerto público a varios puertos privados, tal como sería necesario para esta configuración.
- **Varias instancias de Oracle WebLogic Server en una máquina virtual.** En función de los requisitos de su implementación, puede considerar la opción de ejecutar varias instancias de Oracle WebLogic Server en la misma máquina virtual, si la máquina virtual es suficientemente grande. Por ejemplo, en un máquina virtual de tamaño medio que contiene dos núcleos, puede elegir ejecutar dos instancias de Oracle WebLogic Server. Pero tenga en cuenta que todavía se recomienda evitar introducir únicos puntos de error en la arquitectura, que sería el caso si usase una sola máquina virtual que ejecuta varias instancias de Oracle WebLogic Server. Usar al menos dos máquinas virtuales podría ser un mejor enfoque y, después, cada máquina virtual podría ejecutar varias instancias de Oracle WebLogic Server. Cada instancia de Oracle WebLogic Server podría continuar formando parte del mismo clúster. Pero tenga en cuenta que actualmente no resulta posible usar Azure para efectuar el equilibrio de carga de los puntos de conexión que están expuestos por tales implementaciones de Oracle WebLogic Server dentro de la misma máquina virtual, porque el equilibrador de carga de Azure requiere que los servidores con equilibrio de carga se distribuyan entre máquinas virtuales únicas.

## <a name="oracle-jdk-virtual-machine-images"></a>Imágenes de máquina virtual de Oracle JDK

- **Actualizaciones más recientes de JDK 6 y 7.** Aunque se recomienda usar la última versión compatible pública de Java (actualmente Java 8), Azure también ofrece imágenes de JDK 6 y 7. Esto está pensado para las aplicaciones heredadas que aún no están preparadas para actualizarse a JDK 8. Mientras que es posible que las actualizaciones a imágenes de JDK anteriores ya no estén disponibles para el público general, dada la asociación de Microsoft con Oracle, las imágenes de JDK 6 y 7 que ofrece Azure están diseñadas a fin de contener una actualización más reciente no pública que la que ofrece normalmente Oracle para tan solo un grupo exclusivo de clientes compatibles con Oracle. Con el tiempo habrá disponibles nuevas versiones de las imágenes de JDK con versiones actualizadas de JDK 6 y 7.

   Tenga en cuenta que el JDK disponible en estas imágenes de JDK 6 y 7 y las máquinas virtuales e imágenes derivadas de estas solo se podrán usar dentro de Azure.
- **JDK de 64 bits.** Las imágenes de máquina virtual de Oracle WebLogic Server y las imágenes de máquina virtual de JDK de Oracle que proporciona Azure contienen las versiones de 64 bits de Windows Server y el JDK.

## <a name="next-steps"></a>Pasos siguientes

Ahora tiene una visión general de las soluciones actuales de Oracle basadas en imágenes de máquinas virtuales de Microsoft Azure. El paso siguiente es implementar su primera base de datos de Oracle Database en Azure.

> [!div class="nextstepaction"]
> [Creación de una base de datos de Oracle Database en Azure](oracle-database-quick-create.md)
