---
title: Pruebas de penetración | Microsoft Docs
description: En el artículo se proporciona información general sobre el proceso de pruebas de penetración y cómo realizar dichas pruebas en las aplicaciones que se ejecutan en la infraestructura de Azure.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 695d918c-a9ac-4eba-8692-af4526734ccc
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/24/2020
ms.author: terrylan
ms.openlocfilehash: dfacf124f8db0e5323c9abff56c4a78f85f6f014
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816140"
---
# <a name="penetration-testing"></a>Pruebas de penetración

Una de las ventajas del uso de Azure para las pruebas y la implementación de aplicaciones es que puede crear entornos rápidamente. No tiene que preocuparse del pedido, la adquisición, la "instalación en bastidor y apilamiento" de su propio hardware local.

Crear entornos rápidamente está muy bien, pero debe asegurarse de realizar sus diligencias de seguridad normales. Una de las cosas que quizá desee hacer es realizar pruebas de penetración de las aplicaciones que implemente en Azure.

Es posible que ya sepa que realiza Microsoft realiza [pruebas de penetración de nuestro entorno de Azure](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e). Esta prueba permite impulsar las mejoras de Azure.

No realizamos pruebas de penetración de su aplicación, pero sabemos que quiere y necesita realizar dichas pruebas en sus propias aplicaciones. Eso es bueno, ya que al mejorar la seguridad de sus aplicaciones, ayuda a hacer que todo el ecosistema de Azure sea más seguro.

Desde el 15 de junio de 2017, Microsoft ya no requiere la aprobación previa para llevar a cabo pruebas de penetración con recursos de Azure. Este proceso solo está relacionado con Microsoft Azure y no es aplicable ningún otro servicio de Microsoft Cloud.

>[!IMPORTANT]
>Aunque notificar a Microsoft de las actividades de pruebas de penetración ya no es obligatorio, los clientes deben cumplir las [reglas de compromiso de las pruebas de penetración unificadas de Microsoft Cloud](https://technet.microsoft.com/mt784683) de todos modos.

Entre las pruebas estándar que puede realizar se incluyen:

* Pruebas en los puntos de conexión para descubrir las [10 principales vulnerabilidades del Proyecto de seguridad de aplicación web abierta (OWASP)](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)
* [Pruebas de vulnerabilidad ante datos aleatorios o inesperados](https://cloudblogs.microsoft.com/microsoftsecure/2007/09/20/fuzz-testing-at-microsoft-and-the-triage-process/) de los puntos de conexión
* [Exploración de puertos](https://en.wikipedia.org/wiki/Port_scanner) de los puntos de conexión

Un tipo de prueba que no puede realizar es cualquier tipo de ataque de [denegación de servicio (DoS)](https://en.wikipedia.org/wiki/Denial-of-service_attack). Esta prueba incluye iniciar un ataque de denegación de servicio o realizar pruebas relacionadas que puedan determinar, demostrar o simular cualquier tipo de ataque de denegación de servicio.

>[!Note]
>Microsoft se ha asociado con BreakingPoint Cloud para crear una interfaz en la que pueda generar tráfico destinado a las direcciones IP públicas que tengan habilitado el servicio DDoS Protection con fines de simulación. Para más información sobre la simulación de la nube de punto de interrupción, consulte [Validación de la detección de DDoS](../../virtual-network/manage-ddos-protection.md#validate-ddos-detection).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre las [reglas de compromiso de pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=2).
