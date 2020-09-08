---
title: Puntuación de seguridad de Azure Security Center
description: Descripción de Puntuación de seguridad de Azure Security Center y sus controles de seguridad
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetd: c42d02e4-201d-4a95-8527-253af903a5c6
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/15/2020
ms.author: memildin
ms.openlocfilehash: 596fc44cb3d449b73b681d17db879746ce3051d3
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89277755"
---
# <a name="enhanced-secure-score-in-azure-security-center"></a>Puntuación de seguridad mejorada de Azure Security Center

## <a name="introduction-to-secure-score"></a>Introducción a Puntuación de seguridad

Azure Security Center tiene dos objetivos principales: ayudarle a entender la situación de su seguridad actual y a mejorar esta de forma eficaz. La característica principal de Security Center que le ayuda a conseguir estos objetivos es Puntuación de seguridad.

Security Center evalúa continuamente los recursos, suscripciones y la organización en busca de problemas de seguridad. A continuación, agrega todos los resultados a una sola puntuación para que pueda conocer de un vistazo la situación de la seguridad actual: cuanto mayor sea la puntuación, menor será el nivel de riesgo identificado.

La página Puntuación de seguridad de Security Center incluye lo siguiente:

- **La puntuación**: la puntuación de seguridad se muestra como un valor de porcentaje, pero los valores subyacentes también están claros:

    [![Puntuación de seguridad mostrada como un valor de porcentaje con los números subyacentes claros](media/secure-score-security-controls/secure-score-with-percentage.png)](media/secure-score-security-controls/secure-score-with-percentage.png#lightbox)

- **Controles de seguridad**: cada control es un grupo lógico de recomendaciones de seguridad relacionadas y refleja las superficies de ataque vulnerables. Un control es un conjunto de recomendaciones de seguridad con instrucciones que le ayudan a implementar esas recomendaciones. La puntuación solo mejora cuando corrige *todas* las recomendaciones para un solo recurso de un control.

    Para ver de inmediato cómo protege su organización cada superficie de ataque individual, revise las puntuaciones de cada control de seguridad.

    Para más información, consulte [Cálculo de la puntuación de seguridad](secure-score-security-controls.md#how-your-secure-score-is-calculated) a continuación. 


>[!TIP]
> Las versiones anteriores de Security Center otorgaban puntos en el nivel de recomendación: cuando se corregía una recomendación para un único recurso, la puntuación de seguridad mejoraba. Actualmente, la puntuación solo mejora si se corrigen *todas* las recomendaciones para un solo recurso de un control. Por tanto, la puntuación solo mejora si mejora la seguridad de un recurso.


## <a name="access-your-secure-score"></a>Acceso a la puntuación de seguridad

Encontrará la puntuación de seguridad general, así como la puntuación por suscripción, a través de Azure Portal o mediante programación con la API de REST de Azure Security Center.

### <a name="get-your-secure-score-from-the-portal"></a>Obtención de la puntuación de seguridad desde el portal

Security Center muestra la puntuación de forma destacada en el portal: es lo primero que aparece en la página de información general. Si hace clic en la página dedicada de puntuación de seguridad, verá la puntuación desglosada por suscripción. Haga clic en una suscripción única para ver la lista detallada de las recomendaciones prioritarias y del posible impacto que su corrección tendrá en la puntuación de la suscripción.

![Puntuación de seguridad general como se muestra en el portal](media/secure-score-security-controls/single-secure-score-via-ui.png)

### <a name="get-your-secure-score-from-the-rest-api"></a>Obtención de la puntuación de seguridad desde la API de REST

Ahora puede acceder a la puntuación a través de la [API de puntuación de seguridad](https://docs.microsoft.com/rest/api/securitycenter/securescores/) (actualmente en versión preliminar). Los métodos de API proporcionan la flexibilidad necesaria para consultar los datos y crear su propio mecanismo de creación de informes de las puntuaciones seguras a lo largo del tiempo. Por ejemplo, puede usar la API **Secure Scores** para obtener la puntuación de una suscripción específica. Además, puede usar la API **Secure Score Controls** para mostrar los controles de seguridad y la puntuación actual de las suscripciones.

![Recuperación de una sola puntuación de seguridad a través de la API](media/secure-score-security-controls/single-secure-score-via-api.png)

Para ver ejemplos de herramientas basadas en la API de puntuación de seguridad, consulte el [área de puntuación de seguridad de la comunidad de GitHub](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score). 

## <a name="how-your-secure-score-is-calculated"></a>Cálculo de la puntuación de seguridad 

La contribución de cada control de seguridad en la puntuación de seguridad total aparece con claridad en la página de recomendaciones.

[![La puntuación de seguridad mejorada presenta controles de seguridad](media/secure-score-security-controls/security-controls.png)](media/secure-score-security-controls/security-controls.png#lightbox)

Para obtener todos los puntos posibles de un control de seguridad, todos sus recursos deben cumplir todas las recomendaciones de seguridad de dicho control. Por ejemplo, Security Center tiene varias recomendaciones sobre cómo proteger los puertos de administración. En el pasado, podía corregir algunas de esas recomendaciones relacionadas e interdependientes y dejar otras sin resolver, y con ello, la puntuación de seguridad mejoraba. Analizado objetivamente, era fácil argumentar que la seguridad no mejoraría realmente hasta que se resolvieran todas. Ahora, es necesario corregirlas todas para que la puntuación de seguridad mejore.

Por ejemplo, el control de seguridad denominado "Aplicar actualizaciones del sistema" tiene una puntuación máxima de seis puntos, como puede ver en la información sobre herramientas sobre el potencial valor de mejora del control:

[![Control de seguridad "Aplicar actualizaciones del sistema"](media/secure-score-security-controls/apply-system-updates-control.png)](media/secure-score-security-controls/apply-system-updates-control.png#lightbox)

La puntuación máxima para este control, "Aplicar actualizaciones del sistema", es siempre 6. En este ejemplo, hay 50 recursos. Por tanto, dividimos la puntuación máxima entre 50 y el resultado es que cada recurso aporta 0,12 puntos. 

* **Mejora potencial** (0,12 x 8 recursos con estado incorrecto = 0,96): los puntos restantes disponibles para el usuario en el control. Esto significa que si corrige todas las recomendaciones de este control, la puntuación aumentará en un 2 % (en este caso, 0,96 puntos, que se redondean en 1 punto). 
* **Puntuación actual** (0,12 x 42 recursos con estado correcto = 5,04): puntuación actual para este control. Cada control contribuye a la puntuación total. En este ejemplo, el control contribuye con 5,04 puntos a la puntuación total de seguridad actual.
* **Puntuación máxima**: el número máximo de puntos que puede obtener al completar todas las recomendaciones de un control. La puntuación máxima de un control indica la importancia relativa de ese control. Use los valores de puntuación máxima para evaluar la prioridad de los problemas. 


### <a name="calculations---understanding-your-score"></a>Cálculos: Descripción de la puntuación

|Métrica|Fórmula y ejemplo|
|-|-|
|**Puntuación actual del control de seguridad**|<br>![Ecuación para calcular la puntuación actual de un control de seguridad](media/secure-score-security-controls/security-control-scoring-equation.png)<br><br>Cada control de seguridad individual contribuye a la puntuación de seguridad. Cada recurso afectado por una recomendación dentro del control contribuye a la puntuación actual de este. La puntuación actual de cada control es una medida del estado de los recursos que están *dentro* del control.<br>![Información sobre herramientas que muestra los valores usados al calcular la puntuación actual del control de seguridad](media/secure-score-security-controls/security-control-scoring-tooltips.png)<br>En este ejemplo, la puntuación máxima de 6 se divide entre 78, ya que esta es la suma de los recursos correctos e incorrectos.<br>6 / 78 = 0,0769<br>La multiplicación de esa cifra por el número de recursos correctos (4) da como resultado la puntuación actual:<br>0.0769 * 4 = **0.31**<br><br>|
|**Puntuación segura**<br>Suscripción única|<br>![Ecuación para calcular la puntuación de seguridad actual](media/secure-score-security-controls/secure-score-equation.png)<br><br>![Puntuación de seguridad de una suscripción única con todos los controles habilitados](media/secure-score-security-controls/secure-score-example-single-sub.png)<br>En este ejemplo, hay una suscripción única con todos los controles de seguridad disponibles (una puntuación máxima posible de 60 puntos). La puntuación muestra 28 puntos de los 60 posibles y los 32 puntos restantes se reflejan en las cifras de "Posible aumento de puntuación" de los controles de seguridad.<br>![Lista de controles y posible aumento de puntuación](media/secure-score-security-controls/secure-score-example-single-sub-recs.png)|
|**Puntuación segura**<br>Varias suscripciones|<br>La puntuación actual de todos los recursos de todas las suscripciones se suma y el cálculo es el mismo que para una suscripción única.<br><br>Cuando se ven varias suscripciones, la puntuación de seguridad evalúa todos los recursos de todas las directivas habilitadas y agrupa su impacto combinado en la puntuación máxima de cada control de seguridad.<br>![Puntuación de seguridad de varias suscripciones con todos los controles habilitados](media/secure-score-security-controls/secure-score-example-multiple-subs.png)<br>La puntuación combinada **no** es un promedio; en vez de eso, es la posición evaluada del estado de todos los recursos de todas las suscripciones.<br>También en este caso, si va a la página de recomendaciones y suma los puntos posibles disponibles, observará que esta es la diferencia entre la puntuación actual (24) y la puntuación máxima disponible (60).|
||||

## <a name="improve-your-secure-score"></a>Mejora de su puntuación de seguridad

Para mejorar la puntuación de seguridad, corrija las recomendaciones de seguridad de la lista de recomendaciones. Puede corregir cada recomendación manualmente para cada recurso o utilizar la opción **Corrección rápida** (si está disponible) para aplicar rápidamente una corrección de una recomendación a un grupo de recursos. Para más información, consulte [Corrección de recomendaciones](security-center-remediate-recommendations.md).

>[!IMPORTANT]
> Solo las recomendaciones integradas afectan a la puntuación de seguridad.


## <a name="security-controls-and-their-recommendations"></a>Controles de seguridad y sus recomendaciones

En la tabla siguiente se enumeran los controles de seguridad de Azure Security Center. Para cada control, puede ver el número máximo de puntos que puede sumar a la puntuación de seguridad si corrige *todas* las recomendaciones enumeradas en el control, para *todos* los recursos. 

<div class="foo">

<style type="text/css"> .tg  {border-collapse:collapse;border-spacing:0;} .tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px; overflow:hidden;padding:10px 5px;word-break:normal;} .tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:18px; font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;} .tg .tg-cly1{text-align:left;vertical-align:middle} .tg .tg-lboi{border-color:inherit;text-align:left;vertical-align:middle} </style>
<table class="tg">
<thead>
  <tr>
    <th class="tg-cly1"><b>Control de seguridad, puntuación y descripción</b><br></th>
    <th class="tg-cly1"><b>Recomendaciones</b></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Habilitar MFA (puntuación máxima: 10)</p></strong>Si para autenticar a un usuario solo usa una contraseña, deja un vector desprotegido frente a los ataques. Si la contraseña es débil o se ha expuesto en otro lugar, ¿cómo saber si es el usuario quien inicia sesión realmente con el nombre de usuario y la contraseña?<br>Con <a href="https://www.microsoft.com/security/business/identity/mfa">MFA</a> habilitada, las cuentas son más seguras y los usuarios aún podrán autenticarse en casi cualquier aplicación con inicio de sesión único (SSO).</td>
    <td class="tg-lboi"; width=55%>- MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción.<br>- MFA debe estar habilitada en las cuentas con permisos de escritura de la suscripción.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Proteger puertos de administración (puntuación máxima: 8)</p></strong>Los ataques por fuerza bruta tienen como destino los puertos de administración para obtener acceso a una máquina virtual. Dado que no siempre es necesario abrir los puertos, una estrategia de mitigación consiste en reducir la exposición a los puertos mediante controles de acceso a la red Just-in-Time, grupos de seguridad de red y administración de puertos de máquina virtual.<br>Dado que muchas organizaciones de TI no bloquean las comunicaciones SSH salientes de su red, los atacantes pueden crear túneles cifrados que permitan a los puertos RDP de los sistemas infectados comunicarse con el comando del atacante para controlar los servidores. Los atacantes pueden usar el subsistema de Administración remota de Windows para moverse lateralmente por el entorno y usar credenciales robadas para acceder a otros recursos de una red.</td>
    <td class="tg-lboi"; width=55%>- Los puertos de administración de las máquinas virtuales deben protegerse con el control de acceso de red Just-In-Time.<br>- Las máquinas virtuales deben estar asociadas a un grupo de seguridad de red.<br>- Se deben cerrar los puertos de administración en las máquinas virtuales.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Aplicar actualizaciones del sistema (puntuación máxima: 6)</p></strong>Las actualizaciones del sistema proporcionan a las organizaciones la capacidad de mantener la eficacia operativa, reducir las vulnerabilidades de seguridad y proporcionar un entorno más estable para los usuarios finales. No aplicar actualizaciones deja vulnerabilidades sin revisiones y da lugar a entornos susceptibles a ataques. Estas vulnerabilidades se pueden aprovechar y provocar problemas como la pérdida de datos, la filtración de datos, el ransomware y el uso abusivo de recursos. Para implementar actualizaciones del sistema, puede usar la solución <a href="https://docs.microsoft.com/azure/automation/automation-update-management">Update Management para administrar las actualizaciones y las revisiones</a> de las máquinas virtuales. La administración de actualizaciones es el proceso mediante el que se controla la implementación y el mantenimiento de las versiones de software.</td>
    <td class="tg-lboi"; width=55%>- Los problemas de estado del agente de supervisión se deben resolver en las máquinas.<br>- El agente de supervisión debe instalarse en conjuntos de escalado de máquinas virtuales.<br>- El agente de supervisión debe instalarse en las máquinas.<br>- La versión del sistema operativo debe actualizarse para los roles de servicio en la nube.<br>- Se deben instalar las actualizaciones del sistema en los conjuntos de escalado de máquinas virtuales.<br>- Se deben instalar las actualizaciones del sistema en las máquinas.<br>- Las máquinas deben reiniciarse para aplicar las actualizaciones del sistema.<br>- Kubernetes Services se debe actualizar a una versión de Kubernetes no vulnerable.<br>- Un agente de supervisión debe estar instalado en las máquinas virtuales.<br>- El agente de Log Analytics debe estar instalado en las máquinas de Azure Arc basadas en Windows (versión preliminar).<br>- El agente de Log Analytics debe estar instalado en las máquinas de Azure Arc basadas en Linux (versión preliminar).</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Corregir vulnerabilidades (puntuación máxima: 6)</p></strong>Una vulnerabilidad es un punto débil que un actor de amenazas podría aprovechar para poner en peligro la confidencialidad, la disponibilidad o la integridad de un recurso. La <a href="https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/next-gen-threat-and-vuln-mgt">administración de vulnerabilidades</a> reduce la exposición de la organización, protege el área expuesta del punto de conexión, aumenta la resistencia de la organización y reduce la superficie de ataque de los recursos. La administración de amenazas y vulnerabilidades permite detectar errores de configuración de software y seguridad y proporciona recomendaciones para mitigarlos.</td>
    <td class="tg-lboi"; width=55%>- Se debe habilitar Seguridad avanzada de datos en SQL Database.<br>- Las vulnerabilidades de las imágenes de Azure Container Registry deben corregirse.<br>- Se deben corregir las vulnerabilidades de las bases de datos SQL.<br>- Se deben corregir las vulnerabilidades mediante una solución de evaluación de vulnerabilidades.<br>- Se debe habilitar la valoración de vulnerabilidades en Instancia administrada de SQL.<br>- La evaluación de vulnerabilidades debe estar habilitada en sus servidores de SQL Server.<br>- La solución de evaluación de vulnerabilidades debe instalarse en sus máquinas virtuales.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Habilitar cifrado de datos en reposo (puntuación máxima: 4)</p></strong>El <a href="https://docs.microsoft.com/azure/security/fundamentals/encryption-atrest">cifrado en reposo</a> proporciona protección de datos para los datos almacenados. Los ataques contra los datos en reposo incluyen intentos de obtener acceso físico al hardware en el que se almacenan los datos. Azure usa el cifrado simétrico para cifrar y descifrar grandes cantidades de datos en reposo. Se usa una clave de cifrado simétrico para cifrar datos mientras se escriben en el almacenamiento. Esa clave de cifrado también se utiliza para descifrar los datos tal y como se prepararon para su uso en la memoria. Las claves deben almacenarse en una ubicación segura con el control de acceso basado en identidades y directivas de auditoría. Una ubicación segura es Azure Key Vault. Si un atacante obtiene los datos cifrados pero no las claves de cifrado, no podrá acceder a los datos sin romper el cifrado.</td>
    <td class="tg-lboi"; width=55%>- El cifrado de discos debe aplicarse en las máquinas virtuales.<br>- El cifrado de datos transparente en SQL Database debe estar habilitado.<br>- Las variables de cuenta de Automation deben cifrarse.<br>- Se debe establecer la propiedad ClusterProtectionLevel en EncryptAndSign en los clústeres de Service Fabric.<br>- El protector de TDE de SQL Server se debe cifrar con su propia clave.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Cifrado de los datos en tránsito (puntuación máxima: 4)</p></strong>Cuando se transfieren datos entre componentes, ubicaciones o programas, están "en tránsito". Las organizaciones que no protegen los datos en tránsito son susceptibles a los ataques del tipo "Man in the middle", a la interceptación y al secuestro de sesión. Para intercambiar datos, se deben usar protocolos SSL/TLS y se recomienda utilizar una VPN. Al enviar datos cifrados entre una máquina virtual de Azure y una ubicación local a través de Internet, puede usar una puerta de enlace de red virtual como <a href="https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways">Azure VPN Gateway</a> para enviar tráfico cifrado.</td>
    <td class="tg-lboi"; width=55%>- Se debe acceder a la aplicación de API App solo a través de HTTPS.<br>- Acceso a Function App solo a través de HTTPS.<br>- Solo se deben habilitar las conexiones seguras a Redis Cache.<br>- Se debe habilitar la transferencia segura a las cuentas de almacenamiento.<br>- Acceso a la aplicación web solo a través de HTTPS.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Administrar el acceso y los permisos (puntuación máxima: 4)</p></strong>Una parte fundamental de un programa de seguridad es asegurarse de que los usuarios tengan el acceso necesario para realizar sus trabajos, pero no más: el <a href="https://docs.microsoft.com/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models">modelo de acceso con privilegios mínimos</a>.<br>Controle el acceso a los recursos mediante la creación de asignaciones de roles con <a href="https://docs.microsoft.com/azure/role-based-access-control/overview">control de acceso basado en rol (RBAC)</a>. Una asignación de roles consta de tres elementos:<br>- <strong>Entidad de seguridad</strong>: el objeto al que el usuario solicita acceso.<br>- <strong>Definición de roles</strong>: los permisos.<br>- <strong>Ámbito</strong>: conjunto de recursos a los que se aplican los permisos.</td>
    <td class="tg-lboi"; width=55%>- Las cuentas en desuso se deben eliminar de la suscripción (versión preliminar).<br>- Las cuentas en desuso con permisos de propietario se deben eliminar de la suscripción (versión preliminar).<br>- Las cuentas externas con permisos de propietario se deben eliminar de la suscripción (versión preliminar).<br>- Las cuentas externas con permisos de escritura se deben eliminar de la suscripción (versión preliminar).<br>- Debe haber más de un propietario asignado a la suscripción.<br>- Se debe usar el control de acceso basado en rol (RBAC) en los servicios de Kubernetes (versión preliminar).<br>- Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente.<br>- Para proteger las suscripciones, deben usarse entidades de servicio en lugar de certificados de administración.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Corregir configuraciones de seguridad (puntuación máxima: 4)</p></strong>Los recursos de TI incorrectamente configurados presentan un riesgo mayor de sufrir ataques. Al implementar recursos e intentar ajustarse a unos plazos determinados, suelen pasarse por alto acciones de protección básicas. Puede haber configuraciones de seguridad incorrectas a cualquier nivel de la infraestructura: desde los sistemas operativos y los dispositivos de red hasta los recursos en la nube.<br>Azure Security Center compara continuamente la configuración de los recursos con los requisitos de los estándares del sector, las regulaciones y los bancos de pruebas. Una vez que haya configurado los "paquetes de cumplimiento" (estándares y de referencia) pertinentes para su organización, las carencias darán lugar a recomendaciones de seguridad que incluyen el CCEID y una explicación del posible impacto en la seguridad.<br><a href="https://docs.microsoft.com/azure/security/benchmarks/introduction">Azure Security Benchmark</a> y <a href="https://www.cisecurity.org/benchmark/azure/">CIS Microsoft Azure Foundations Benchmark versión 1.1.0</a> son dos paquetes que se usan con frecuencia.</td>
    <td class="tg-lboi"; width=55%>- Las directivas de seguridad de pod deben definirse en los servicios de Kubernetes.<br>- Se deben corregir las vulnerabilidades en las configuraciones de seguridad de contenedor.<br>- Se deben corregir las vulnerabilidades en la configuración de seguridad de las máquinas.<br>- Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales.<br>- Un agente de supervisión debe estar instalado en las máquinas virtuales.<br>- El agente de supervisión debe instalarse en las máquinas.<br>- El agente de Log Analytics debe estar instalado en las máquinas de Azure Arc basadas en Windows (versión preliminar).<br>- El agente de Log Analytics debe estar instalado en las máquinas de Azure Arc basadas en Linux (versión preliminar).<br>- El agente de supervisión debe instalarse en conjuntos de escalado de máquinas virtuales.<br>- Los problemas de estado del agente de supervisión se deben resolver en las máquinas.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Restringir los accesos de red no autorizados (puntuación máxima: 4)</p></strong>Los puntos de conexión de una organización proporcionan una conexión directa desde la red virtual a los servicios de Azure compatibles. Las máquinas virtuales de una subred pueden comunicarse con todos los recursos. Para limitar la comunicación hacia y desde todos los recursos de una subred, cree un grupo de seguridad de red y asócielo a la subred. Las organizaciones pueden limitar el tráfico no autorizado y protegerse mediante la creación de reglas de entrada y salida.</td>
    <td class="tg-lboi"; width=55%>- El reenvío de IP en la máquina virtual debe estar deshabilitado.<br>- Deben definirse los intervalos IP autorizados en los servicios de Kubernetes (versión preliminar).<br>- (EN DESUSO) El acceso a App Services debe estar restringido (versión preliminar).<br>- (EN DESUSO) Se deben proteger las reglas de las aplicaciones web en los grupos de seguridad de red de IaaS.<br>- Las máquinas virtuales deben estar asociadas a un grupo de seguridad de red.<br>- CORS no debe permitir que todos los recursos accedan a la aplicación de API.<br>- CORS no debe permitir que todos los recursos accedan a Function App.<br>- CORS no debe permitir que todos los recursos accedan a las aplicaciones web.<br>- Se debe desactivar la depuración remota para la aplicación de API.<br>- Se debe desactivar la depuración remota para Function App.<br>- Se debe desactivar la depuración remota para las aplicaciones web.<br>- Se debe restringir el acceso a los grupos de seguridad de red permisivos con máquinas virtuales accesibles desde Internet.<br>- Se deben proteger las reglas del grupo de seguridad de red para máquinas virtuales accesibles desde Internet.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Aplicar controles de aplicaciones adaptables (puntuación máxima: 3)</p></strong>El control de aplicaciones adaptables (AAC) es una solución de un extremo a otro inteligente y automatizada que permite controlar qué aplicaciones se pueden ejecutar en las máquinas virtuales de Azure y que no son de Azure. También ayuda a proteger los equipos frente a malware.<br>Security Center usa el aprendizaje automático para crear una lista de aplicaciones seguras conocidas para un grupo de máquinas.<br>Este enfoque innovador de inclusión de aplicaciones en listas ofrece ventajas de seguridad sin la complejidad de la administración.<br>El control de aplicaciones adaptables es especialmente adecuado para servidores específicos que necesitan ejecutar un conjunto concreto de aplicaciones.</td>
    <td class="tg-lboi"; width=55%>- Se deben habilitar los controles de aplicaciones adaptables en las máquinas virtuales.<br>- Un agente de supervisión debe estar instalado en las máquinas virtuales.<br>- El agente de supervisión debe instalarse en las máquinas.<br>- El agente de Log Analytics debe estar instalado en las máquinas de Azure Arc basadas en Windows (versión preliminar).<br>- El agente de Log Analytics debe estar instalado en las máquinas de Azure Arc basadas en Linux (versión preliminar).<br>- Los problemas de estado del agente de supervisión se deben resolver en las máquinas.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Aplicar la clasificación de datos (puntuación máxima: 2)</p></strong>La clasificación de los datos de su organización por confidencialidad e impacto empresarial le permite determinar y asignar valor a los datos, y proporciona la estrategia y la base para la gobernanza.<br><a href="https://docs.microsoft.com/azure/information-protection/what-is-information-protection">Azure Information Protection</a> puede ayudarle a clasificar los datos. Este servicio usa directivas de cifrado, identidad y autorización para proteger los datos y restringir el acceso a ellos. Algunas de las clasificaciones que Microsoft utiliza son No empresarial, Público, General, Confidencial y Extremadamente confidencial.</td>
    <td class="tg-lboi"; width=55%>- Los datos confidenciales de las bases de datos SQL deben clasificarse (versión preliminar).</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Proteger aplicaciones contra ataques DDoS (puntuación máxima: 2)</p></strong>Los ataques de denegación de servicio distribuido (DDoS) sobrecargan los recursos y dejan inutilizables las aplicaciones. Use <a href="https://docs.microsoft.com/azure/virtual-network/ddos-protection-overview">Azure DDoS Protection Standard</a> para defender su organización de los tres tipos principales de ataques DDoS:<br>Los - <strong>ataques volumétricos</strong> desbordan la red con tráfico legítimo. DDoS Protection Standard mitiga estos ataques al absorberlos o limpiarlos automáticamente.<br>Los - <strong>ataques a protocolos</strong> impiden el acceso a un destino al aprovechar una debilidad en la pila de protocolo en los niveles 3 y 4. DDoS Protection Standard mitiga estos ataques bloqueando el tráfico malintencionado.<br>Los - <strong>ataques de nivel de recurso (aplicación)</strong> van dirigidos a paquetes de aplicación web. Para defenderse de este tipo de ataque, use un firewall de aplicaciones web y DDoS Protection Standard.</td>
    <td class="tg-lboi"; width=55%>- Se debe habilitar DDoS Protection Standard.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Habilitar la protección de los puntos de conexión (puntuación máxima: 2)</p></strong>Para asegurarse de que los puntos de conexión están protegidos contra malware, los sensores de comportamiento recopilan y procesan datos de los sistemas operativos de los puntos de conexión y envían estos datos a la nube privada para su análisis. El análisis de seguridad utiliza Big Data, el aprendizaje automático y otras fuentes para recomendar respuestas a las amenazas. Por ejemplo, <a href="https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection">ATP de Microsoft Defender</a> usa la inteligencia sobre amenazas para identificar métodos de ataque y generar alertas de seguridad.<br>Security Center admite las siguientes soluciones de protección de los puntos de conexión: Windows Defender, System Center Endpoint Protection, Trend Micro, Symantec v12.1.1.1100, McAfee v10 para Windows, McAfee v10 para Linux y Sophos v9 para Linux. Si Security Center detecta alguna de estas soluciones, dejará de mostrarse la recomendación de instalar protección de puntos de conexión.</td>
    <td class="tg-lboi"; width=55%>- Los errores de estado de protección de puntos de conexión se deben corregir en los conjuntos de escalado de máquinas virtuales.<br>- Los problemas de estado de protección de puntos de conexión se deben resolver en las máquinas.<br>- La solución de protección de puntos de conexión se debe instalar en los conjuntos de escalado de máquinas virtuales.<br>- Instale la solución de protección de puntos de conexión en máquinas virtuales.<br>- Los problemas de estado del agente de supervisión se deben resolver en las máquinas.<br>- El agente de supervisión debe instalarse en conjuntos de escalado de máquinas virtuales.<br>- El agente de supervisión debe instalarse en las máquinas.<br>- Un agente de supervisión debe estar instalado en las máquinas virtuales.<br>- El agente de Log Analytics debe estar instalado en las máquinas de Azure Arc basadas en Windows (versión preliminar).<br>- El agente de Log Analytics debe estar instalado en las máquinas de Azure Arc basadas en Linux (versión preliminar).<br>- Instale la solución de protección de puntos de conexión en las máquinas.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Habilitar auditoría y registro (puntuación máxima: 1)</p></strong>Los datos de registro proporcionan conclusiones sobre problemas anteriores, evitan problemas potenciales, pueden mejorar el rendimiento de las aplicaciones y proporcionan la capacidad de automatizar acciones que de otro modo deberían realizarse manualmente.<br>Los - <strong>registros de control y administración</strong> proporcionan información sobre las operaciones de <a href="https://docs.microsoft.com/azure/azure-resource-manager/management/overview">Azure Resource Manager</a>.<br>Los - <strong>registros del plano de datos</strong> proporcionan información sobre los eventos desencadenados como parte del uso de los recursos de Azure.<br>Los - <strong>eventos procesados</strong> proporcionan información sobre eventos y alertas analizados que se han procesado.</td>
    <td class="tg-lboi"; width=55%>- La auditoría de SQL Server debe estar habilitada.<br>- Los registros de diagnóstico de App Services deben estar habilitados.<br>- Los registros de diagnóstico de Azure Data Lake Store deben estar habilitados.<br>- Los registros de diagnóstico de Azure Stream Analytics deben estar habilitados.<br>- Los registros de diagnóstico de las cuentas de Batch deben estar habilitados.<br>- Los registros de diagnóstico de Data Lake Analytics deben estar habilitados.<br>- Los registros de diagnóstico de Event Hubs deben estar habilitados.<br>- Los registros de diagnóstico de IoT Hub deben estar habilitados.<br>- Los registros de diagnóstico de Key Vault deben estar habilitados.<br>- Los registros de diagnóstico de Logic Apps deben estar habilitados.<br>- Los registros de diagnóstico del servicio Search deben estar habilitados.<br>- Los registros de diagnóstico de Service Bus deben estar habilitados.<br>- Los registros de diagnóstico de Virtual Machine Scale Sets deben estar habilitados.<br>- Las reglas de alerta de métricas deben configurarse en las cuentas de Batch.<br>- La configuración de auditoría de SQL debe tener grupos de acción configurados para capturar actividades críticas.<br>- Debe configurar los servidores SQL Server para que realicen una retención de la auditoría durante más de 90 días.</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Habilitar Advanced Threat Protection (puntuación máxima 0)</p></strong>La protección contra amenazas de Azure Security Center proporciona defensas completas para el entorno. Cuando Security Center detecta una amenaza en cualquiera de las áreas del entorno, genera una alerta. Estas alertas describen los detalles de los recursos afectados, los pasos de corrección sugeridos y, en algunos casos, una opción para desencadenar una aplicación lógica como respuesta.<br>Cada paquete de protección contra amenazas es una oferta independiente y opcional que puede habilitar con la recomendación correspondiente en este control de seguridad.<br><a href="https://docs.microsoft.com/azure/security-center/threat-protection">Más información sobre la protección contra amenazas en Security Center</a>.</td>
    <td class="tg-lboi"; width=55%>- Advanced Data Security debe estar habilitado en los servidores de Azure SQL Database<br>- Advanced Data Security debe estar habilitado en los servidores SQL Server en las máquinas<br>- Advanced Threat Protection debe estar habilitado en Virtual Machines<br>- Advanced Threat Protection debe estar habilitado en los planes de Azure App Service<br>- Advanced Threat Protection debe estar habilitado en las cuentas de almacenamiento de Azure<br>- Advanced Threat Protection debe estar habilitado en los clústeres de Azure Kubernetes Service<br>- Advanced Threat Protection debe estar habilitado en los registros de Azure Container Registry<br>- Advanced Threat Protection debe estar habilitado en los almacenes de Azure Key Vault</td>
  </tr>
  <tr>
    <td class="tg-lboi"><strong><p style="font-size: 16px">Implementar procedimientos recomendados de seguridad (puntuación máxima: 0)</p></strong>Prácticas de seguridad modernas: "presunción de brecha" del perímetro de red. Por ese motivo, muchos de los procedimientos recomendados de este control se centran en la administración de identidades.<br>La pérdida de claves y credenciales es un problema común. <a href="https://docs.microsoft.com/azure/key-vault/key-vault-overview">Azure Key Vault</a> protege las claves y los secretos mediante el cifrado de claves, archivos .pfx y contraseñas.<br>Las redes privadas virtuales (VPN) son una forma segura de acceder a las máquinas virtuales. Si no hay VPN disponibles, asegúrese de usar frases de contraseña complejas y la autenticación en dos fases (como <a href="https://docs.microsoft.com/azure/active-directory/authentication/concept-mfa-howitworks">Azure Multi-Factor Authentication</a>). La autenticación en dos fases evita las debilidades inherentes al uso exclusivo de nombres de usuario y contraseñas.<br>El uso de plataformas sólidas de autenticación y autorización es otro procedimiento recomendado. El uso de identidades federadas permite a las organizaciones delegar la administración de identidades autorizadas. Esto también es importante cuando los empleados abandonan la empresa y es necesario revocar su acceso.</td>
    <td class="tg-lboi"; width=55%>- Debe designar un máximo de tres propietarios para la suscripción.<br>- Las cuentas externas con permisos de lectura se deben eliminar de la suscripción.<br>- MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción.<br>- Se debe restringir el acceso a las cuentas de almacenamiento con configuraciones de red virtual y firewall.<br>- Todas las reglas de autorización, excepto RootManageSharedAccessKey, se deben eliminar del espacio de nombres de Event Hubs.<br>- Se debe aprovisionar el administrador de Azure Active Directory para servidores SQL Server.<br>- La seguridad avanzada de datos debe estar habilitada en las instancias administradas.<br>- Las reglas de autorización de la instancia del centro de eventos deben definirse.<br>- Se deben migrar las cuentas de almacenamiento a los nuevos recursos de Azure Resource Manager.<br>- Se deben migrar las máquinas virtuales a los nuevos recursos de Azure Resource Manager.<br>- Las subredes deben estar asociadas con un grupo de seguridad de red.<br>- [Versión preliminar] La protección contra vulnerabilidades de Windows debe estar habilitada. <br>- [Versión preliminar] El agente de configuración de invitado debe estar instalado.<br>- Las máquinas virtuales sin conexión a Internet deben protegerse con grupos de seguridad de red.</td>
  </tr>
</tbody>
</table>


</div>




## <a name="secure-score-faq"></a>Preguntas frecuentes sobre la puntuación de seguridad

### <a name="why-has-my-secure-score-gone-down"></a>¿Por qué ha dejado de funcionar mi puntuación de seguridad?
Security Center ha cambiado a una puntuación de seguridad mejorada que incluye los cambios en la forma de calcular la puntuación. Ahora, debe resolver todas las recomendaciones de un recurso para recibir puntos. Las puntuaciones también han cambiado a una escala de 0-10.

### <a name="if-i-address-only-three-out-of-four-recommendations-in-a-security-control-will-my-secure-score-change"></a>Si solo se corrigen tres de las cuatro recomendaciones de un control de seguridad, ¿cambiará mi puntuación de seguridad?
No. No cambiará hasta que corrija todas las recomendaciones para un único recurso. Para obtener la puntuación máxima de un control, debe corregir todas las recomendaciones de todos los recursos.

### <a name="is-the-previous-experience-of-the-secure-score-still-available"></a>¿La experiencia de Puntuación de seguridad anterior todavía está disponible? 
No. Durante un tiempo se ejecutaron en paralelo para facilitar la transición. El modelo anterior ya está en desuso. 

### <a name="if-a-recommendation-isnt-applicable-to-me-and-i-disable-it-in-the-policy-will-my-security-control-be-fulfilled-and-my-secure-score-updated"></a>Si una recomendación no es aplicable a mí y la deshabilito en la directiva, ¿se cumplirá mi control de seguridad y se actualizará mi puntuación de seguridad?
Sí. Se recomienda deshabilitar las recomendaciones cuando no son aplicables a su entorno. Para obtener instrucciones sobre cómo deshabilitar una recomendación específica, consulte [Deshabilitar las directivas de seguridad](https://docs.microsoft.com/azure/security-center/tutorial-security-policy#disable-security-policies).

### <a name="if-a-security-control-offers-me-zero-points-towards-my-secure-score-should-i-ignore-it"></a>Si un control de seguridad ofrece cero puntos a mi puntuación de seguridad, ¿debería ignorarlo?
En algunos casos verá una puntuación máxima del control mayor que cero, pero el impacto es nulo. Cuando la puntuación incremental para corregir recursos es insignificante, se redondea a cero. Aún así, no ignore estas recomendaciones ya que pueden aportarle mejoras en seguridad. La única excepción es el control de "procedimiento recomendado adicional". La corrección de estas recomendaciones no aumentará la puntuación, pero mejorará la seguridad en general.

## <a name="next-steps"></a>Pasos siguientes

En este artículo se describe la puntuación de seguridad y los controles de seguridad que presenta. Para obtener material relacionado, consulte los siguientes artículos:

- [Más información sobre los distintos elementos de una recomendación](security-center-recommendations.md)
- [Recomendaciones de corrección](security-center-remediate-recommendations.md)
