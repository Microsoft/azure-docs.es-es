---
title: Uso de Windows Virtual Desktop con Azure NetApp Files | Microsoft Docs
description: Proporciona orientación sobre los procedimientos recomendados y planos técnicos de ejemplo sobre la implementación de Windows Virtual Desktop con Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/13/2020
ms.author: b-juche
ms.openlocfilehash: a003090fd610f2ac75895cccbf97750adbd4cfcd
ms.sourcegitcommit: ef055468d1cb0de4433e1403d6617fede7f5d00e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/16/2020
ms.locfileid: "88258332"
---
# <a name="benefits-of-using-azure-netapp-files-with-windows-virtual-desktop"></a>Ventajas de usar Azure NetApp Files con Windows Virtual Desktop 

En este artículo se proporciona orientación sobre los procedimientos recomendados para implementar Windows Virtual Desktop (WVD) con Azure NetApp Files.

Azure NetApp Files es un servicio de almacenamiento de archivos de Azure que ofrece un alto rendimiento. Puede proporcionar hasta 450.000 IOPS y una latencia inferior a milisegundos, capaz de admitir implementaciones de Windows Virtual Desktop a escalas muy grandes. Es posible ajustar el ancho de banda y cambiar el nivel de servicio de los volúmenes de Azure NetApp Files a petición, casi de forma instantánea, sin necesidad de pausar las operaciones de E/S ni perder el acceso al plano de datos. Esta funcionalidad permite optimizar fácilmente la escalabilidad de la implementación de WVD para ahorrar costos. También se pueden crear instantáneas de volúmenes en un momento dado, eficientes desde el punto de vista del espacio, sin que el rendimiento de los volúmenes se vea afectado. Esta funcionalidad permite revertir [contenedores de perfiles de usuario de FSLogix](https://docs.microsoft.com/azure/virtual-desktop/store-fslogix-profile) individuales a través de una copia del directorio `~snapshot`, o revertir instantáneamente el volumen completo en una sola operación mediante la funcionalidad de reversión del volumen.  Al disponer de hasta 255 instantáneas (rotativas) para proteger un volumen frente a la pérdida de datos u otros daños, los administradores tienen numerosas posibilidades de deshacer operaciones.

## <a name="sample-blueprints"></a>Planos técnicos de ejemplo

En los siguientes planos técnicos de ejemplo se muestra la integración de Windows Virtual Desktop con Azure NetApp Files. En un escenario de escritorio agrupado, los usuarios son dirigidos al mejor host de sesión del grupo que esté disponible ([ modo en amplitud](https://docs.microsoft.com/azure/virtual-desktop/host-pool-load-balancing#breadth-first-load-balancing-method)) mediante [máquinas virtuales multisesión](https://docs.microsoft.com/azure/virtual-desktop/windows-10-multisession-faq#what-is-windows-10-enterprise-multi-session). Por otro lado, los escritorios personales se reservan para escenarios en los que cada usuario dispone de su propia máquina virtual.

### <a name="pooled-desktop-scenario"></a>Escenario de escritorio agrupado

Para el escenario agrupado, el equipo de Windows Virtual Desktop ofrece [estas recomendaciones](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/virtual-machine-recs#multi-session-recommendations) en función del número de usuarios para vCPU. Tenga en cuenta que en la recomendación no se especifica ningún tamaño de máquina virtual.

|     Tipo de carga de trabajo     |     Claro    |     Media    |     Pesado    |
|-----------------------|--------------|---------------|--------------|
|     Usuarios por vCPU    |     6        |     4         |     2        |


Esta recomendación se confirma mediante una prueba LoginVSI de 500 usuarios, que registra aproximadamente 62 "usuarios medios/del conocimiento" en cada máquina virtual D16as_V4. 

Por ejemplo, con 62 usuarios por máquina virtual D16as_V4, Azure NetApp Files puede admitir fácilmente a 60 000 usuarios por entorno. La prueba para evaluar el límite superior de la máquina virtual D32as_v4 es continua. Si la recomendación de usuario de WVD por vCPU se cumple para la máquina virtual D32as_v4, se admitirán más de 120 000 usuarios en 1000 máquinas virtuales antes de abordar el [límite de red virtual de 1000 direcciones IP](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-network-topologies), tal y como se muestra en la siguiente ilustración.  

![Escenario de escritorio agrupado en Windows Virtual Desktop](../media/azure-netapp-files/solutions-pooled-desktop-scenario.png)   

### <a name="personal-desktop-scenario"></a>Escenario de escritorio personal 

La siguiente ilustración muestra la arquitectura de uso general recomendada en un escenario de escritorio personal. Los usuarios se asignan a pods de escritorio específicos y cada pod tiene menos de 1000 máquinas virtuales, lo que deja espacio para que las direcciones IP se propaguen desde la red virtual de administración. Azure NetApp Files puede administrar fácilmente más de 900 escritorios personales por cada red virtual de grupo host de sesión única, y el número real de máquinas virtuales es igual a 1000 menos el número de hosts de administración que se encuentran en la red virtual del concentrador. Si se necesitan más escritorios personales, se pueden añadir fácilmente otros pods (grupos de hosts y redes virtuales), tal y como se muestra en la ilustración siguiente. 

![Escenario de escritorio personal en Windows Virtual Desktop](../media/azure-netapp-files/solutions-personal-desktop-scenario.png)  

Cuando se crear una arquitectura basada en pods como esta, es importante asignar usuarios al pod correcto al iniciar sesión para garantizar que los usuarios siempre encuentren sus perfiles de usuario. 

## <a name="next-steps"></a>Pasos siguientes

- [Arquitecturas de las soluciones con Azure NetApp Files](azure-netapp-files-solution-architectures.md)
