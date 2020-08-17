---
title: Archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 03/05/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 138e7efd33ee36a77fd56deda155a29017432302
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420299"
---
- Solo se puede vincular una red virtual a un objeto de acceso a disco.
- La red virtual debe estar en la misma suscripción que el objeto de acceso a disco para vincularlas.
- Se pueden importar o exportar hasta 10 discos o instantáneas al mismo tiempo con el mismo objeto de acceso a disco.
- No se puede solicitar la aprobación manual para vincular una red virtual a un objeto de acceso al disco.
- La funcionalidad diferencial no se admite para las instantáneas incrementales que están asociadas a un objeto de acceso a disco.
- AzCopy no se puede usar para descargar un VHD de un disco o instantánea protegidos mediante vínculos privados a una cuenta de almacenamiento. Sin embargo, se puede usar para descargar VHD en máquinas virtuales.
