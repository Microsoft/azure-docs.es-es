---
title: Solución alternativa de la configuración del firewall de almacenamiento
description: Una configuración del firewall de red de la cuenta de almacenamiento puede producir un error al crear un destino de Azure Blob Storage en Azure HPC Cache. En este artículo se ofrece una solución alternativa para esta limitación hasta que se aplique una corrección de software.
author: ekpgh
ms.service: hpc-cache
ms.topic: troubleshooting
ms.date: 11/7/2019
ms.author: v-erkel
ms.openlocfilehash: 6916c79e9110a88beff65d487fac72441382c2f4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092379"
---
# <a name="work-around-blob-storage-account-firewall-settings"></a>Solución alternativa para la configuración del firewall de la cuenta de Blob Storage

Un valor determinado utilizado en los firewalls de la cuenta de almacenamiento puede provocar un error en la creación del destino de Blob Storage. El equipo de Azure HPC Cache trabaja en una corrección de software para este problema, pero puede solucionarlo según las instrucciones de este artículo.

La configuración de firewall que permite el acceso únicamente desde "redes seleccionadas" puede impedir que la memoria caché cree un destino de Blob Storage. Esta configuración se encuentra en la página de configuración de **Firewalls y redes virtuales** de la cuenta de almacenamiento.

El problema es que el servicio de caché usa una red virtual de servicio oculta que es independiente de los entornos de cliente. No es posible autorizar explícitamente esta red para que acceda a la cuenta de almacenamiento.

Cuando se crea un destino de Blob Storage, el servicio de caché usa esta red para comprobar si el contenedor está vacío o no. Si el firewall no permite el acceso desde la red oculta, se produce un error en la comprobación y se produce un error en la creación del destino de almacenamiento.

Para solucionar el problema, cambie temporalmente la configuración del firewall al crear el destino de almacenamiento:

1. Vaya a la página **Firewalls y redes virtuales** de la cuenta de almacenamiento y cambie la configuración "Permitir el acceso desde" a **Todas las redes**.
1. Cree el destino de Blob Storage en Azure HPC Cache.
1. Una vez que el destino de almacenamiento se haya creado correctamente, vuelva a cambiar la configuración del firewall de la cuenta a **Redes seleccionadas**.

Azure HPC Cache no usa la red virtual de servicio para acceder al destino de almacenamiento finalizado.

Para obtener ayuda con esta solución alternativa, [póngase en contacto con el servicio de soporte técnico de Microsoft](hpc-cache-support-ticket.md).
