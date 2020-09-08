---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines-linux
author: cynthn
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 07/09/2020
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 612e7dc2f3d1ef6aa5d3598999aa214a2f377e1f
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513200"
---
## <a name="overview-of-ssh-and-keys"></a>Información general sobre SSH y sus claves

[SSH](https://www.ssh.com/ssh/) es un protocolo de conexión cifrada que permite inicios de sesión seguros a través de conexiones no seguras. SSH es el protocolo de conexión predeterminado de las máquinas virtuales Linux hospedadas en Azure. Aunque SSH proporciona una conexión cifrada, el uso de contraseñas con conexiones SSH deja la máquina virtual vulnerable a ataques por fuerza bruta. Se recomienda conectarse a una máquina virtual a través de SSH mediante un par de claves pública y privada, también conocidas como *claves SSH*. 

- La *clave pública* se coloca en la máquina virtual Linux.

- La *clave privada* permanece en el sistema local. Esta clave se debe proteger, por lo que no se debe compartir.

Cuando se usa un cliente SSH para conectarse a la máquina virtual Linux (que tiene la clave pública), la máquina virtual remota comprueba que el cliente posee la clave privada. Si es el caso, se le concede acceso a la máquina virtual. 

En función de las directivas de seguridad de la organización, puede volver a usar un par de claves públicas y privadas único para obtener acceso a varias máquinas virtuales y servicios de Azure. No es necesario usar ningún par de claves independiente para cada máquina virtual o servicio al que se quiera acceder. 

La clave pública se puede compartir con cualquiera. Sin embargo, solo usted (o su infraestructura de seguridad local) debe tener acceso a la clave privada.