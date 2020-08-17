---
title: Distribuciones de Linux aprobadas en Azure
description: Aprenda sobre Linux en las distribuciones aprobadas de Azure, incluyendo instrucciones para Ubuntu, CentOS, Oracle y SUSE.
services: virtual-machines-linux
documentationcenter: ''
author: gbowerman
manager: gwallace
tags: azure-service-management,azure-resource-manager
ms.assetid: 2777a526-c260-4cb9-a31a-bdfe1a55fffc
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: guybo
ms.openlocfilehash: fd21170c4edc1ed0587ea4d4e067e61590530623
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87283323"
---
# <a name="endorsed-linux-distributions-on-azure"></a>Distribuciones de Linux aprobadas en Azure

Los asociados ofrecen imágenes de Linux en Azure Marketplace. Microsoft trabaja con distintas comunidades de Linux para agregar aún más tipos a la lista de distribuciones aprobadas. Para las distribuciones que no están disponibles en Marketplace, siempre puede aportar su propia imagen de Linux mediante las directrices descritas en [Creación y carga de un disco duro virtual que contiene el sistema operativo Linux](./create-upload-generic.md).

## <a name="supported-distributions-and-versions"></a>Distribuciones y versiones admitidas

En la tabla siguiente se enumeran las distribuciones y versiones de Linux que se admiten en Azure. Consulte el [soporte para imágenes Linux en Microsoft Azure](https://support.microsoft.com/help/2941892/support-for-linux-and-open-source-technology-in-azure) para obtener información más detallada sobre el soporte para Linux y la tecnología de código abierto en Azure.

Los controladores de los Servicios de integración de Linux (LIS) para Hyper-V y Azure son módulos del kernel que Microsoft aporta directamente al kernel de Linux del canal de subida. Algunos controladores LIS están integrados de forma predeterminada en el kernel de la distribución. Hay distribuciones anteriores basadas en Red Hat Enterprise (RHEL)/CentOS disponibles como descarga independiente en [Linux Integration Services versión 4.2 para Hyper- V y Azure](https://www.microsoft.com/download/details.aspx?id=55106). Vea [Requisitos para el kernel de Linux](create-upload-generic.md#linux-kernel-requirements) para obtener más información sobre los controladores de LIS.

El Agente de Linux de Azure ya está preinstalado en las imágenes de Azure Marketplace y generalmente está disponible en el repositorio de paquetes de la distribución. El código fuente se puede encontrar en [GitHub](https://github.com/azure/walinuxagent).

| Distribución | Versión | Controladores | Agente |
| --- | --- | --- | --- |
| CentOS de Rogue Wave Software |CentOS 6.x, 7.x, 8.x |CentOS 6.3: [Descarga de LIS](https://www.microsoft.com/download/details.aspx?id=55106)<p>CentOS 6.4 o superior: En kernel |Paquete: en el [repositorio](http://olcentgbl.trafficmanager.net/openlogic/6/openlogic/x86_64/RPMS/), en "WALinuxAgent" <br/>Código fuente: [GitHub](https://github.com/Azure/WALinuxAgent) |
| [CoreOS](https://coreos.com/docs/running-coreos/cloud-providers/azure/)<p> CoreOS ahora está en el [final del ciclo de vida](https://coreos.com/os/eol/) a partir del 26 de mayo de 2020. |Ya no está disponible | | |
| Debian por Credativ |8.x, 9.x |En kernel |Paquete: en el repositorio, en "waagent" <br/>Código fuente: [GitHub](https://github.com/Azure/WALinuxAgent) |
|Flatcar Container Linux de Kinvolk| Estable, Edge| | |
| Oracle Linux de Oracle |6.x, 7.x, 8.x |En kernel |Paquete: en el repositorio, en "WALinuxAgent" <br/>Código fuente: [GitHub](https://go.microsoft.com/fwlink/p/?LinkID=250998) |
| Red Hat Enterprise Linux de Red Hat |6.x, 7.x, 8.x |En kernel |Paquete: en el repositorio, en "WALinuxAgent" <br/>Código fuente: [GitHub](https://github.com/Azure/WALinuxAgent) |
| SUSE Linux Enterprise de SUSE |SLES/SLES para SAP 11.x, 12.x, 15.x <br/> [Ciclo de vida de imagen en la nube pública de SUSE](https://www.suse.com/c/suse-public-cloud-image-life-cycle/) |En kernel |Paquete:<p> para 11 en el repositorio [Cloud:Tools](https://build.opensuse.org/project/show/Cloud:Tools)<br>para 12, que se incluye en el módulo "Nube pública" en "python-azure-agent"<br/>Código fuente: [GitHub](https://go.microsoft.com/fwlink/p/?LinkID=250998) |
| openSUSE de SUSE |openSUSE Leap 15.x |En kernel |Paquete: en el repositorio [Cloud:Tools](https://build.opensuse.org/project/show/Cloud:Tools), en "python-azure-agent" <br/>Código fuente: [GitHub](https://github.com/Azure/WALinuxAgent) |
| Ubuntu de Canonical |Ubuntu Server y Pro. 16.x, 18.x, 20.x<p>Puede encontrar información sobre la compatibilidad ampliada con Ubuntu 12.04 y 14.04 aquí: [Extended Security Maintenance de Ubuntu](https://www.ubuntu.com/esm). |En kernel |Paquete: en el repositorio, en "walinuxagent" <br/>Código fuente: [GitHub](https://github.com/Azure/WALinuxAgent) |

## <a name="image-update-cadence"></a>Cadencia de actualización de imagen

Azure requiere que los publicadores de las distribuciones aprobadas de Linux actualicen con regularidad sus imágenes en Azure Marketplace con las últimas revisiones y correcciones de seguridad, en una cadencia trimestral o más rápida. Las imágenes actualizadas en Azure Marketplace están disponibles automáticamente para los clientes como las nuevas versiones de una SKU de imagen. Más información sobre cómo encontrar imágenes de Linux: [Búsqueda de imágenes de VM Linux en Azure Marketplace](./cli-ps-findimage.md).

## <a name="azure-tuned-kernels"></a>Kernels optimizados para Azure

Azure trabaja en estrecha colaboración con diversas distribuciones de Linux aprobadas para la optimización de imágenes que publican en Azure Marketplace. Un aspecto de esta colaboración es el desarrollo de los kernels "optimizados" de Linux que están optimizados para la plataforma Azure y se entregan como componentes totalmente compatibles de la distribución de Linux. Los kernels optimizados para Azure incorporan nuevas características y mejoras de rendimiento, así como una cadencia más rápida (normalmente trimestral) en comparación con el valor predeterminado o los kernels genéricos que están disponibles en la distribución.

En la mayoría de los casos, encontrará estos kernels preinstalados en las imágenes predeterminadas de Azure Marketplace, por lo que los clientes de Azure obtendrán inmediatamente las ventajas de estos kernels optimizados. Puede encontrar más información sobre los kernels optimizados para Azure en los enlaces siguientes:

- [Kernel de CentOS optimizado para Azure: disponible mediante CentOS Virtualization SIG](https://wiki.centos.org/SpecialInterestGroup/Virtualization)
- [Kernel de Debian en la nube: disponible con la imagen de los "backports" de Debian 10 y Debian 9 en Azure](https://wiki.debian.org/Cloud/MicrosoftAzure)
- [Kernel de SLES optimizado para Azure](https://www.suse.com/c/a-different-builtin-kernel-for-azure-on-demand-images/)
- [Kernel de Ubuntu optimizado para Azure](https://blog.ubuntu.com/2017/09/21/microsoft-and-canonical-increase-velocity-with-azure-tailored-kernel)

## <a name="partners"></a>Asociados

### <a name="coreos"></a>CoreOS

La [finalización del ciclo de vida](https://coreos.com/os/eol/) de CoreOS está programada para el 26 de mayo de 2020.
Microsoft tiene dos (2) canales de migración para los usuarios de CoreOS.

- Flatcar de Kinvolk (consulte la entrada "Flatcar Container Linux de Kinvolk").
- [Fedora Core OS](https://docs.fedoraproject.org/en-US/fedora-coreos/provisioning-azure/) (los clientes deben cargar su propia imagen. Esta es la [documentación de migración](https://docs.fedoraproject.org/en-US/fedora-coreos/migrate-cl/)).

### <a name="credativ"></a>Credativ

[https://www.credativ.co.uk/credativ-blog/debian-images-microsoft-azure](https://www.credativ.co.uk/credativ-blog/debian-images-microsoft-azure)

Credativ es una empresa de servicios y consultoría independiente que se especializa en el desarrollo y la implementación de soluciones profesionales mediante el uso de software gratuito. Como especialistas en código abierto líderes en el sector, Credative goza de reconocimiento en el ámbito internacional y numerosos departamentos de TI recurren a su soporte técnico. Actualmente, Credativ está preparando con Microsoft imágenes de Debian correspondientes a Debian 8 (Jessie) y Debian antes de 7 (Wheezy). Ambas imágenes están diseñadas especialmente para ejecutarse en Azure y pueden administrarse fácilmente a través de la plataforma. Credativ también será compatible con el mantenimiento y la actualización a largo plazo de las imágenes de Debian para Azure a través de sus centros de soporte técnico de código abierto.

### <a name="oracle"></a>Oracle

[https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html](https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html)

La estrategia de Oracle consiste en ofrecer una amplia cartera de soluciones para nubes públicas y privadas. Esta estrategia ofrece e los clientes posibilidad de elección y flexibilidad en la forma en que distribuyen software de Oracle en nubes de Oracle y en otras nubes. La asociación de Oracle y Microsoft permite a los clientes implementar software de Oracle en nubes públicas y privadas de Microsoft con la confianza que brinda la certificación y el respaldo de Oracle.  El compromiso y la inversión de Oracle en las soluciones de nubes públicas y privadas de Oracle siguen intactos.

### <a name="red-hat"></a>Red Hat

[https://www.redhat.com/en/partners/strategic-alliance/microsoft](https://www.redhat.com/en/partners/strategic-alliance/microsoft)

Red Hat, proveedor líder mundial de soluciones de código abierto, ayuda a más del 90 % de las empresas Fortune 500 a resolver desafíos empresariales, a alinear sus estrategias empresariales y de TI y a prepararse para el futuro de la tecnología. Red Hat hace esto ofreciendo soluciones seguras a través de un modelo de negocio abierto y un modelo de suscripción asequible y predecible.

### <a name="suse"></a>SUSE

[https://www.suse.com/suse-linux-enterprise-server-on-azure](https://www.suse.com/suse-linux-enterprise-server-on-azure)

SUSE Linux Enterprise Server en Azure es una plataforma probada que brinda confiabilidad y seguridad de nivel superior para la informática en nube. La versátil plataforma Linux de SUSE se integra de manera fluida con los servicios en la nube de Azure para brindar un entorno de nube fácilmente administrable. Con más de 9200 aplicaciones certificadas de más de 1800 fabricantes de software independiente para SUSE Linux Enterprise Server, SUSE garantiza que las cargas de trabajo que se ejecutan de manera compatible en el centro de datos pueden implementarse con confianza en Azure.

### <a name="canonical"></a>Canonical

[https://www.ubuntu.com/cloud/azure](https://www.ubuntu.com/cloud/azure)

La ingeniería de Canonical y la gobernanza de comunidad abierta impulsan el éxito de Ubuntu en la informática en la nube, servidor y cliente, incluidos servicios personales en la nube para consumidores. La visión de Canonical de una plataforma unificada y gratuita en Ubuntu, del teléfono a la nube, ofrece una serie de interfaces coherentes para teléfono, tableta, TV y escritorio. Esta visión convierte a Ubuntu en la primera opción para diversas instituciones, desde proveedores en la nube pública hasta los fabricantes de aparatos electrónicos de consumo, y el producto favorito entre tecnólogos individuales.

Con desarrolladores y centros de ingeniería por todo el mundo, Canonical ocupa una posición privilegiada para asociarse con fabricantes de hardware, proveedores de contenido y desarrolladores de software para llevar al mercado las soluciones de Ubuntu para equipos, servidores y dispositivos portátiles.
