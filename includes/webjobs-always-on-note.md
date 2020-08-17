---
title: archivo de inclusión
description: archivo de inclusión
services: app-service
author: ggailey777
ms.service: app-service
ms.topic: include
ms.date: 06/26/2020
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: c255be53a1809bf5dd3fc6b184852767dfec9c66
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009809"
---
> [!NOTE]
> Una aplicación web puede agotar el tiempo de espera después de 20 minutos de inactividad. Solo las solicitudes a la aplicación web real pueden restablecer el temporizador. Al ver la configuración de la aplicación en Azure Portal o realizar solicitudes en el sitio de herramientas avanzadas (`https://<app_name>.scm.azurewebsites.net`), no se restablece el temporizador. Si configura la aplicación web para que ejecute trabajos web continuos o programados (desencadenador de temporizador), habilite la opción **AlwaysOn** en la página **Configuración** de Azure de la aplicación web para asegurarse de que los trabajos web se ejecuten de forma confiable. Esta característica solo está disponible en los [planes de tarifa](https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) Básico, Estándar y Premium.
