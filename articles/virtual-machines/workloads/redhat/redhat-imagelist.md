---
title: Imágenes de Red Hat Enterprise Linux disponibles en Azure
description: Obtenga información sobre las imágenes de Red Hat Enterprise Linux en Microsoft Azure.
services: virtual-machines-linux
documentationcenter: ''
author: asinn826
manager: BorisB2015
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 04/16/2020
ms.author: alsin
ms.openlocfilehash: f4ebf433237193c0db7859b029bb224c72c92eae
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87291022"
---
# <a name="red-hat-enterprise-linux-rhel-images-available-in-azure"></a>Imágenes de Red Hat Enterprise Linux (RHEL) disponibles en Azure
Azure ofrece una variedad de imágenes de RHEL para diferentes casos de uso.

> [!NOTE]
> Todas las imágenes de RHEL están disponibles en las nubes públicas de Azure y de Azure Government. No están disponibles en las nubes de Azure China.

## <a name="list-of-rhel-images"></a>Lista de imágenes de RHEL
Esta es una lista de imágenes de RHEL disponibles en Azure. A menos que se indique lo contrario, todas las imágenes tienen particiones LVM y se acoplarán en los repositorios de RHEL convencionales (no EUS ni E4S). Las imágenes siguientes están disponibles actualmente para uso general:

> [!NOTE]
> Ya no se generan imágenes sin procesar en favor de las imágenes con particiones LVM. LVM ofrece varias ventajas en comparación con el esquema de partición sin formato (que no es LVM) anterior, incluidas las opciones de cambio de tamaño de partición significativamente más flexibles.

Oferta| SKU | Creación de particiones | Aprovisionamiento | Notas
:----|:----|:-------------|:-------------|:-----
RHEL          | 6.7      | RAW    | Agente Linux |
|             | 6,8      | RAW    | Agente Linux |
|             | 6.9      | RAW    | Agente Linux |
|             | 6.10     | RAW    | Agente Linux |
|             | 7-RAW    | RAW    | Agente Linux | Familia de imágenes de RHEL 7.x. <br> Se acopla a repositorios convencionales de forma predeterminada (no EUS).
|             | 7-LVM    | LVM    | Agente Linux | Familia de imágenes de RHEL 7.x. <br> Se acopla a repositorios convencionales de forma predeterminada (no EUS). Si está buscando una imagen de RHEL estándar para implementarla, utilice este conjunto de imágenes o su homólogo de segunda generación.
|             | 7lvm-gen2| LVM    | Agente Linux | Familia de imágenes de RHEL 7.x de segunda generación. <br> Se acopla a repositorios convencionales de forma predeterminada (no EUS). Si está buscando una imagen de RHEL estándar para implementarla, utilice este conjunto de imágenes o su homólogo de primera generación.
|             | 7-RAW-CI | RAW-CI | Cloud-init  | Familia de imágenes de RHEL 7.x. <br> Se acopla a repositorios convencionales de forma predeterminada (no EUS).
|             | 7.2      | RAW    | Agente Linux |
|             | 7.3      | RAW    | Agente Linux |
|             | 7.4      | RAW    | Agente Linux | Adjuntada a repositorios de EUS de forma predeterminada a partir de abril de 2019.
|             | 74-gen2  | RAW    | Agente Linux | Adjuntada a repositorios de EUS de forma predeterminada.
|             | 7.5      | RAW    | Agente Linux | Adjuntada a repositorios de EUS de forma predeterminada a partir de junio de 2019.
|             | 75-gen2  | RAW    | Agente Linux | Adjuntada a repositorios de EUS de forma predeterminada.
|             | 7.6      | RAW    | Agente Linux | Adjuntada a repositorios de EUS de forma predeterminada a partir de mayo de 2019.
|             | 76-gen2  | RAW    | Agente Linux | Adjuntada a repositorios de EUS de forma predeterminada.
|             | 7,7      | LVM    | Agente Linux | Adjuntada a repositorios de EUS de forma predeterminada.
|             | 77-gen2  | LVM    | Agente Linux | Adjuntada a repositorios de EUS de forma predeterminada.
|             | 7.8      | LVM    | Agente Linux | Conectada a repositorios normales (EUS no está disponible para RHEL 7.8)
|             | 78-gen2  | LVM    | Agente Linux | Conectada a repositorios normales (EUS no está disponible para RHEL 7,8)
|             | 8-LVM    | LVM    | Agente Linux | Familia de imágenes de RHEL 8.x. Se adjunta a repositorios convencionales.
|             | 8-lvm-gen2| LVM    | Agente Linux | Generación 2 de Hyper-V: familia de imágenes de RHEL 8.x. Se adjunta a repositorios convencionales.
|             | 8        | LVM    | Agente Linux | Imágenes de RHEL 8.0.
|             | 8-gen2   | LVM    | Agente Linux | Hyper-V Generación 2: imágenes de RHEL 8.0.
|             | 8.1      | LVM    | Agente Linux | Imágenes de RHEL 8.2. Actualmente adjunto a repositorios convencionales.
|             | 81gen2   | LVM    | Agente Linux | Hyper-V Generación 2: imágenes de RHEL 8.1. Actualmente adjunto a repositorios convencionales.
|             | 8.1-ci   | LVM    | Agente Linux | Imágenes de RHEL 8.1 que usan cloud-init como agente de aprovisionamiento. Actualmente adjunto a repositorios convencionales.
|             | 81-ci-gen2| LVM    | Agente Linux | Hyper-V de segunda generación: imágenes de RHEL 8.1 que usan cloud-init como agente de aprovisionamiento. Actualmente adjunto a repositorios convencionales.
|             | 8,2      | LVM    | Agente Linux | Imágenes de RHEL 8.2. Actualmente adjunto a repositorios convencionales.
|             | 82gen2   | LVM    | Agente Linux | Hyper-V Generación 2: imágenes de RHEL 8.1. Actualmente adjunto a repositorios convencionales.
RHEL-SAP      | 7.4      | LVM    | Agente Linux | RHEL 7.4 for SAP HANA y Business Apps. Se acopla a los repositorios de E4S, cobrará un recargo para SAP y RHEL, así como la tarifa de proceso básica.
|             | 74sap-gen2| LVM    | Agente Linux | RHEL 7.4 for SAP HANA y Business Apps. Imagen de segunda generación. Se acopla a los repositorios de E4S, cobrará un recargo para SAP y RHEL, así como la tarifa de proceso básica.
|             | 7.5       | LVM    | Agente Linux | RHEL 7.5 for SAP HANA y Business Apps. Se acopla a los repositorios de E4S, cobrará un recargo para SAP y RHEL, así como la tarifa de proceso básica.
|             | 75sap-gen2| LVM    | Agente Linux | RHEL 7.5 for SAP HANA y Business Apps. Imagen de segunda generación. Se acopla a los repositorios de E4S, cobrará un recargo para SAP y RHEL, así como la tarifa de proceso básica.
|             | 7.6       | LVM    | Agente Linux | RHEL 7.6 for SAP HANA y Business Apps. Se acopla a los repositorios de E4S, cobrará un recargo para SAP y RHEL, así como la tarifa de proceso básica.
|             | 76sap-gen2| LVM    | Agente Linux | RHEL 7.6 for SAP HANA y Business Apps. Imagen de segunda generación. Se acopla a los repositorios de E4S, cobrará un recargo para SAP y RHEL, así como la tarifa de proceso básica.
|             | 7,7       | LVM    | Agente Linux | RHEL 7.7 for SAP HANA y Business Apps. Se acopla a los repositorios de E4S, cobrará un recargo para SAP y RHEL, así como la tarifa de proceso básica.
RHEL-SAP-HANA | 6.7       | RAW    | Agente Linux | RHEL 6.7 for SAP HANA. Obsoleto en favor de las imágenes de RHEL-SAP.
|             | 7.2       | LVM    | Agente Linux | RHEL 7.2 for SAP HANA. Obsoleto en favor de las imágenes de RHEL-SAP.
|             | 7.3       | LVM    | Agente Linux | RHEL 7.3 for SAP HANA. Obsoleto en favor de las imágenes de RHEL-SAP.
RHEL-SAP-APPS | 6,8       | RAW    | Agente Linux | RHEL 6.8 for SAP Business Applications. Obsoleto en favor de las imágenes de RHEL-SAP.
|             | 7.3       | LVM    | Agente Linux | RHEL 7.3 for SAP Business Applications. Obsoleto en favor de las imágenes de RHEL-SAP.
RHEL-HA       | 7.4       | LVM    | Agente Linux | RHEL 7.4 con el complemento de alta disponibilidad. Cobrará un recargo por alta disponibilidad y RHEL además de la tarifa de proceso básica.
|             | 7.5       | LVM    | Agente Linux | RHEL 7.5 con el complemento de alta disponibilidad. Cobrará un recargo por alta disponibilidad y RHEL además de la tarifa de proceso básica.
|             | 7.6       | LVM    | Agente Linux | RHEL 7.6 con el complemento de alta disponibilidad. Cobrará un recargo por alta disponibilidad y RHEL además de la tarifa de proceso básica.
RHEL-SAP-HA   | 7.4          | LVM    | Agente Linux | RHEL 7.4 for SAP con alta disponibilidad y servicios de actualización. Se acopla a los repositorios de E4S. Cobrará un recargo para los repositorios de SAP y alta disponibilidad así como RHEL, además de las tarifas de proceso básicas.
|             | 74sapha-gen2 | LVM    | Agente Linux | RHEL 7.4 for SAP con alta disponibilidad y servicios de actualización. Imagen de segunda generación. Se acopla a los repositorios de E4S. Cobrará un recargo para los repositorios de SAP y alta disponibilidad así como RHEL, además de las tarifas de proceso básicas.
|             | 7.5          | LVM    | Agente Linux | RHEL 7.5 for SAP con alta disponibilidad y servicios de actualización. Se acopla a los repositorios de E4S. Cobrará un recargo para los repositorios de SAP y alta disponibilidad así como RHEL, además de las tarifas de proceso básicas.
|             | 7.6          | LVM    | Agente Linux | RHEL 7.6 for SAP con alta disponibilidad y servicios de actualización. Se acopla a los repositorios de E4S. Cobrará un recargo para los repositorios de SAP y alta disponibilidad así como RHEL, además de las tarifas de proceso básicas.
|             | 76sapha-gen2 | LVM    | Agente Linux | RHEL 7.6 for SAP con alta disponibilidad y servicios de actualización. Imagen de segunda generación. Se acopla a los repositorios de E4S. Cobrará un recargo para los repositorios de SAP y alta disponibilidad así como RHEL, además de las tarifas de proceso básicas.
|             | 7,7          | LVM    | Agente Linux | RHEL 7.7 for SAP con alta disponibilidad y servicios de actualización. Se acopla a los repositorios de E4S. Cobrará un recargo para los repositorios de SAP y alta disponibilidad así como RHEL, además de las tarifas de proceso básicas.
|             | 77sapha-gen2 | LVM    | Agente Linux | RHEL 7.7 for SAP con alta disponibilidad y servicios de actualización. Imagen de segunda generación. Se acopla a los repositorios de E4S. Cobrará un recargo para los repositorios de SAP y alta disponibilidad así como RHEL, además de las tarifas de proceso básicas.
rhel-byos     |rhel-lvm74| LVM    | Agente Linux | Las imágenes BYOS de RHEL 7.4, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm75| LVM    | Agente Linux | Las imágenes BYOS de RHEL 7.5, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm76| LVM    | Agente Linux | Las imágenes BYOS de RHEL 7.6, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm76-gen2| LVM    | Agente Linux | Las imágenes BYOS de RHEL 7.6 de segunda generación, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm77| LVM    | Agente Linux | Las imágenes BYOS de RHEL 7.7, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm77-gen2| LVM    | Agente Linux | Las imágenes BYOS de RHEL 7.7 de segunda generación, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm78| LVM    | Agente Linux | Las imágenes BYOS de RHEL 7.8, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm78-gen2| LVM    | Agente Linux | Las imágenes BYOS de RHEL 7.8 de segunda generación, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm8 | LVM    | Agente Linux | Las imágenes BYOS de RHEL 8.0, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm8-gen2 | LVM    | Agente Linux | Las imágenes BYOS de RHEL 8.0 de segunda generación, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm81 | LVM    | Agente Linux | Las imágenes BYOS de RHEL 8.1, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm81-gen2 | LVM    | Agente Linux | Las imágenes BYOS de RHEL 8.1 de segunda generación, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm82 | LVM    | Agente Linux | Las imágenes BYOS de RHEL 8.2, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.
|             |rhel-lvm82-gen2 | LVM    | Agente Linux | Las imágenes BYOS de RHEL 8.2 de segunda generación, no acopladas a ningún origen de actualizaciones, no cobrarán un recargo de RHEL.

> [!NOTE]
> Red Hat considera fin de la vida útil la oferta de productos RHEL-SAP-HANA. Las implementaciones existentes seguirán funcionando con normalidad, pero Red Hat recomienda que los clientes migren de las imágenes RHEL-SAP-HANA a las imágenes RHEL-SAP-HANA que incluyen los repositorios SAP HANA y el complemento HA. Más detalles sobre las ofertas de la nube SAP de Red Hat se encuentran disponibles [aquí](https://access.redhat.com/articles/3751271).

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre las [imágenes de Red Hat en Azure](./redhat-images.md).
* Obtenga más información sobre la [infraestructura de actualización de Red Hat](./redhat-rhui.md).
* Obtenga más información sobre la [oferta BYOS de RHEL](./byos.md).
* Puede encontrar información sobre las directivas de soporte técnico de Red Hat para todas las versiones de RHEL en la página [Red Hat Enterprise Linux Life Cycle](https://access.redhat.com/support/policy/updates/errata) (Ciclo de vida de Red Hat Enterprise Linux).
