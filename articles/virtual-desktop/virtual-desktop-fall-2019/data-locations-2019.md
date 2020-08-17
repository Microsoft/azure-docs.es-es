---
title: 'Ubicaciones de datos para Windows Virtual Desktop (clásico): Azure'
description: Una breve introducción a las ubicaciones en las que se almacenan los datos y metadatos de Windows Virtual Desktop (clásico).
author: Heidilohr
ms.topic: conceptual
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 869defde657c9cb4c8bea6bbacebb9458e5a2b96
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008821"
---
# <a name="data-locations-for-windows-virtual-desktop-classic"></a>Ubicaciones de datos de Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager, consulte [este artículo](../data-locations.md).

Windows Virtual Desktop está disponible actualmente para todas las ubicaciones geográficas. Inicialmente, los metadatos del servicio solo se pueden almacenar en el área geográfica de Estados Unidos (EE. UU.). Los administradores pueden elegir la ubicación para almacenar los datos de usuario cuando creen las máquinas virtuales del grupo de hosts y los servicios asociados, como los servidores de archivos. Obtenga más información sobre las ubicaciones geográficas de Azure en el [mapa de centros de datos de Azure](https://azuredatacentermap.azurewebsites.net/).

>[!NOTE]
>Microsoft no controla ni limita las regiones en las que usted o sus usuarios pueden acceder a los datos específicos de la aplicación o del usuario.

>[!IMPORTANT]
>Windows Virtual Desktop almacena información de metadatos globales como los nombres de inquilinos, nombres de grupos de hosts, nombres de grupos de aplicaciones y nombres principales de usuario en un centro de datos ubicado en Estados Unidos. Los metadatos almacenados se cifran en reposo y los reflejos con redundancia geográfica se mantienen en Estados Unidos. Todos los datos del cliente, como la configuración de la aplicación y los datos de usuario, residen en la ubicación que el cliente elige y no los administra el servicio.

Los metadatos del servicio se replican en Estados Unidos con fines de recuperación ante desastres.