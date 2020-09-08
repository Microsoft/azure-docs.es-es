---
title: Verificación de la configuración de Azure Traffic Manager
description: En este artículo, aprenderá a comprobar la configuración de Traffic Manager y a probar el método de enrutamiento del tráfico.
services: traffic-manager
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/16/2017
ms.author: duau
ms.openlocfilehash: 4bb7458d4258dffb074d663e5cf712c7a0245cc7
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401527"
---
# <a name="verify-traffic-manager-settings"></a>Comprobación de la configuración de Traffic Manager

Para probar la configuración de Traffic Manager, debe tener varios clientes, en diversas ubicaciones, desde los que pueda ejecutar las pruebas. Después, desactive los puntos de conexión en el perfil de Traffic Manager de uno en uno.

* Establezca un valor bajo para TTL de DNS, de forma que los cambios se propaguen rápidamente (por ejemplo, 30 segundos).
* Conozca las direcciones IP de los servicios en la nube de Azure y los sitios web del perfil que prueba.
* Use herramientas que le permitan resolver un nombre de DNS en una dirección IP y muestre dicha dirección.

Compruebe que los nombres DNS se resuelvan en las direcciones IP de los puntos de conexión del perfil. Deberían resolverse de manera congruente con el método de enrutamiento del tráfico definido en el perfil de Traffic Manager. Puede usar herramientas como **nslookup** o **dig** para resolver nombres DNS.

Los siguientes ejemplos lo ayudarán a probar el perfil de Traffic Manager.

### <a name="check-traffic-manager-profile-using-nslookup-and-ipconfig-in-windows"></a>Comprobación del perfil de Traffic Manager con nslookup e ipconfig en Windows

1. Abra un símbolo del sistema o de Windows PowerShell como administrador.
2. Escriba `ipconfig /flushdns` para vaciar la memoria caché de resolución DNS.
3. Escriba `nslookup <your Traffic Manager domain name>`. Por ejemplo, el siguiente comando comprueba el nombre de dominio con el prefijo *myapp.contoso*.

    ```powershell
    nslookup myapp.contoso.trafficmanager.net
    ```

    Un resultado típico muestra la siguiente información:

    + El nombre de DNS y la dirección IP del servidor DNS al que se accede para resolver este nombre de dominio del Administrador de tráfico.
    + El nombre de dominio del Administrador de tráfico que especificó en la línea de comandos después de "nslookup" y la dirección IP en la que se resuelve el dominio del Administrador de tráfico. La segunda dirección IP es la que es importante comprobar. Debe coincidir con una dirección IP virtual (VIP) pública de uno de los servicios en la nube o los sitios web del perfil del Administrador de tráfico que prueba.

## <a name="how-to-test-the-failover-traffic-routing-method"></a>Pruebas del método de enrutamiento del tráfico de conmutación por error

1. Deje todos los extremos activados.
2. Con un solo cliente, solicite la resolución DNS del nombre de dominio de la empresa con la utilidad nslookup u otra parecida.
3. Asegúrese de que la dirección IP resuelta coincida con el punto de conexión principal.
4. Desactive el punto de conexión principal o quite el archivo de supervisión para que Traffic Manager considere que la aplicación está desactivada.
5. Espere durante el período de vida (TTL) de DNS del perfil de Traffic Manager, más dos minutos adicionales. Por ejemplo, si el TTL de DNS es de 300 segundos (5 minutos), debe esperar 7 minutos.
6. Vacíe la memoria caché del cliente DNS y solicite la resolución DNS mediante nslookup. En Windows, puede vaciar la caché DNS con el comando ipconfig /flushdns.
7. Asegúrese de que la dirección IP resuelta coincida con el punto de conexión secundario.
8. Repita el proceso desactivando progresivamente cada punto de conexión. Compruebe que el DNS devuelva la dirección IP del siguiente punto de conexión de la lista. Cuando haya desactivado todos los extremos, debería volver a obtener la dirección IP del extremo principal.

## <a name="how-to-test-the-weighted-traffic-routing-method"></a>Pruebas del método de enrutamiento del tráfico ponderado

1. Deje todos los extremos activados.
2. Con un solo cliente, solicite la resolución DNS del nombre de dominio de la empresa con la utilidad nslookup u otra parecida.
3. Asegúrese de que la dirección IP resuelta coincida con uno de los puntos de conexión.
4. Vacíe la memoria caché del cliente DNS y repita los pasos 2 y 3 para cada punto de conexión. Debería ver diferentes direcciones IP devueltas para cada uno de los extremos.

## <a name="how-to-test-the-performance-traffic-routing-method"></a>Pruebas del método de enrutamiento del tráfico de rendimiento

Para probar eficazmente un método de enrutamiento del tráfico de rendimiento, deberá tener clientes situados en distintas partes del mundo. Puede crear clientes en diferentes regiones de Azure para usarlos para probar los servicios. Si tiene una red global, puede iniciar sesión de forma remota en los clientes de otras zonas del mundo y ejecutar las pruebas desde ellos.

Como alternativa, existen servicios gratuitos en la Web para indagación y búsqueda DNS. Algunas de estas herramientas proporcionan la capacidad de comprobar la resolución de nombres DNS desde varias ubicaciones del mundo. Busque con "búsqueda DNS" para obtener ejemplos. Los servicios de terceros como, por ejemplo, Gomez o Keynote, sirven para confirmar que los perfiles estén distribuyendo el tráfico según lo esperado.

## <a name="next-steps"></a>Pasos siguientes

* [Información acerca de los métodos de enrutamiento del tráfico del Administrador de tráfico](traffic-manager-routing-methods.md)
* [Consideraciones de rendimiento sobre el Administrador de tráfico](traffic-manager-performance-considerations.md)
* [Solución de problemas de estado degradado del Administrador de tráfico](traffic-manager-troubleshooting-degraded.md)
