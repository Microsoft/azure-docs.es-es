---
title: 'Entrega de una prueba de concepto: Azure DevTest Labs | Microsoft Docs'
description: Obtenga información sobre cómo entregar una prueba de concepto para que Azure DevTest Labs se pueda incorporar correctamente en un entorno empresarial.
ms.topic: article
ms.date: 06/2/2020
ms.openlocfilehash: 9c28cf9eebd8a39a2edce48e4fb8b96dc7608d80
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288024"
---
# <a name="deliver-a-proof-of-concept"></a>Entrega de una prueba de concepto 

Uno de los escenarios clave para Azure DevTest Labs es habilitar los entornos de desarrollo y pruebas en la nube. Algunos ejemplos son:

* Creación de escritorios para desarrolladores en la nube.
* Configuración de entornos para pruebas.
* Habilitación del acceso a máquinas virtuales y otros recursos de Azure.
* Configuración de un área de espacio aislado para que los desarrolladores aprendan y experimenten.

DevTest Labs también proporciona las medidas protectoras de la directiva y controles de costos para que la empresa pueda proporcionar "autoservicio de Azure" a los desarrolladores que se adhieren a las directivas corporativas de seguridad, normativas y de cumplimiento. 

Cada empresa tiene requisitos diferentes para el modo en que Azure DevTest Labs puede incorporarse correctamente en su entorno. En este artículo se describen los pasos más comunes que las empresas deben completar para garantizar una prueba de concepto correcta. Una prueba de concepto es el primer paso para una implementación de un extremo a otro correcta. 

## <a name="getting-started"></a>Introducción 

Para empezar a entregar una prueba de concepto, es importante dedicar algún tiempo a obtener información sobre Azure y DevTest Labs.  Estos son algunos recursos de inicio: 

* [Información sobre Azure Portal](https://azure.microsoft.com/features/azure-portal/)
* [Aspectos básicos de DevTest Labs](devtest-lab-overview.md)
* [Escenarios conocidos de uso de Azure DevTest Labs](devtest-lab-guidance-get-started.md)
* [DevTest Labs en la empresa](devtest-lab-guidance-prescriptive-adoption.md)
* [¿Qué es Azure Virtual Network?](../virtual-network/virtual-networks-overview.md)

## <a name="prerequisites"></a>Prerrequisitos 

Para completar correctamente una prueba piloto o una prueba de concepto con DevTest Labs, hay algunos requisitos previos: 

* **Suscripción de Azure**: A menudo, las empresas tienen un [contrato Enterprise](https://azure.microsoft.com/pricing/purchase-options/enterprise-agreement/) en vigor que permite el acceso a Azure, y pueden usar una suscripción nueva o existente para DevTest Labs. Como alternativa, las empresas pueden usar una [suscripción de Visual Studio](https://azure.microsoft.com/pricing/member-offers/visual-studio-subscriptions/) durante la fase piloto (aprovechando los créditos gratuitos de Azure). Si ninguna de estas opciones está disponible, una empresa puede crear y usar una [cuenta de Azure gratuita](https://azure.microsoft.com/free/search/?&OCID=AID719825_SEM_g4lyBqgB&lnkd=Bing_Azure_Brand&msclkid=ecc4275a31b61375749e7a5322c20de8&dclid=CMGW5-m78-ICFaLt4QodmUwGtQ). Si hay un Contrato Enterprise en su lugar, el uso de una [suscripción de Desarrollo/pruebas - Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p/) es una excelente opción para obtener acceso a los sistemas operativos de cliente Windows 10/Windows 8.1 y a las tarifas con descuento para las cargas de trabajo de desarrollo y pruebas. 
* **Inquilino de Azure Active Directory**: Para habilitar la administración de usuarios (por ejemplo, agregar usuarios o agregar propietarios de laboratorios), esos usuarios deben formar parte del [inquilino de Azure Active Directory](https://azure.microsoft.com/services/active-directory/) que se usa en la suscripción a Azure para el piloto. A menudo, las empresas configurarán la [identidad híbrida](../active-directory/hybrid/whatis-hybrid-identity.md) para permitir que los usuarios utilicen su identidad local en la nube, pero esto no es necesario para el piloto de DevTest Labs. 

## <a name="scoping-of-the-pilot"></a>Ámbito del piloto 

Es importante planear un piloto antes de iniciar la implementación. Saber de antemano que los recursos no se mantendrán indefinidamente establece las expectativas adecuadas para los usuarios del piloto. 

> [!IMPORTANT]
> Es imprescindible establecer cuidadosamente el ámbito del piloto y la expectativa por adelantado.

Responda a estas preguntas clave antes de comenzar el piloto: 

* ¿Qué quiere aprender y cómo concluirá que el piloto ha finalizado con éxito? 
* ¿Qué cargas de trabajo o escenarios se tratarán en el piloto? Es importante definir solo un conjunto pequeño para asegurarse de que se pueda establecer y completar rápidamente el ámbito del piloto. 
* ¿Qué recursos deben estar disponibles en el laboratorio? Por ejemplo: imágenes personalizadas, imágenes de Marketplace, directivas, topología de red. 
* ¿Quiénes son los usuarios y los equipos que participarán en el piloto para comprobar la experiencia?  
* ¿Cuál es la duración del piloto? Elija un período de tiempo que se alinee bien con el ámbito planeado, como dos semanas o un mes. 
* Una vez completado el piloto, ¿qué ocurrirá con los recursos asignados que se usaron durante este período? ¿Tiene pensado descartar los recursos del piloto? Puede pensar lo siguiente:
   
   "Si podemos planear el descarte de las máquinas virtuales y los laboratorios al final del piloto, podemos configurar una suscripción única para el piloto y hacer todo nuestro trabajo allí mientras resolvemos las preguntas del lanzamiento de escala en paralelo". 

Existe la tendencia general de configurar el piloto "perfecto" para que represente de forma idéntica el estado final que tendrá una vez que se lance el servicio en la empresa. Esta es una suposición falsa. Cuanto más se acerque a la versión "perfecta", más deberá completar *antes* de comenzar a trabajar en el piloto. La finalidad del piloto es tomar las decisiones correctas sobre la escalabilidad y el lanzamiento del servicio final. 

El objetivo del piloto es elegir las cargas de trabajo y dependencias mínimas necesarias para responder a la pregunta de si Azure DevTest Labs es el servicio adecuado para la empresa. Es aconsejable elegir la carga de trabajo más sencilla con las dependencias mínimas para garantizar un éxito más rápido y claro. Si eso no es posible, elija una carga de trabajo más representativa que exponga las posibles complejidades para que el éxito de la fase piloto se pueda replicar en la fase de escalabilidad horizontal. 

En el ejemplo siguiente se muestra una prueba de concepto con el ámbito correcto. 

## <a name="example-proof-of-concept-plan"></a>Ejemplo: plan de prueba de concepto 

En esta sección se muestra un ejemplo que se usa para el ámbito de una prueba de concepto del piloto de DevTest Labs. 

> [!TIP]
> Para minimizar la posibilidad de configurar el proyecto para que se produzca un error, se recomienda encarecidamente no omitir el ejemplo que se describe en esta sección. 

### <a name="overview"></a>Información general 

Estamos planeando desarrollar un nuevo entorno en Azure basado en DevTest Labs para que los proveedores lo usen como un entorno aislado de la red corporativa. Para determinar si la solución cumplirá los requisitos, desarrollaremos una prueba de concepto para validar la solución de un extremo a otro. Hemos incluido varios proveedores para probar y verificar la experiencia. 

### <a name="outcomes"></a>Resultados 

Al compilar una prueba de concepto, nos centramos primero en los resultados (qué intentamos lograr). Al final de la prueba de concepto, esperamos lo siguiente: 

* Una solución de un extremo a otro en funcionamiento para que los proveedores utilicen las cuentas de invitado en Azure Active Directory (Azure AD) para tener acceso a un entorno aislado en Azure. El entorno tiene los recursos necesarios para que sean productivos. 
* Se enumeran y comprenden los posibles problemas de bloqueo que afectan al uso y la adopción de una escala más amplia.
* Las personas implicadas en el desarrollo de la prueba de concepto tienen un buen conocimiento de todo el código. También comprenden la documentación y material adjunto implicados y están seguros en una adopción más amplia.

### <a name="open-questions-and-prerequisites"></a>Preguntas abiertas y requisitos previos 

* ¿Se ha creado una suscripción que podemos usar para este proyecto? 
* ¿Tenemos un inquilino de Azure AD y un administrador global de Azure AD identificados que puedan proporcionar ayuda e instrucciones para preguntas relacionadas con Azure AD? 
* ¿Tenemos un lugar donde colaborar para las personas que trabajan en el proyecto? 

   * Código fuente y scripts (como Azure Repos) 
   * Documentos (como Microsoft Teams o SharePoint)  
   * Conversaciones (como Microsoft Teams) 
   * Elementos de trabajo (como Azure Boards) 
* ¿Cuáles son los recursos necesarios para los proveedores? Esto incluye aplicaciones disponibles en la red, tanto localmente en las máquinas virtuales como en otros servidores necesarios. 
* ¿Las máquinas virtuales se unirán a un dominio en Azure? En ese caso, ¿será Azure Active Directory Domain Services (Azure AD DS) u otra opción? 
* ¿Tenemos identificados el equipo o los proveedores que serán el objetivo de la prueba de concepto? ¿Quiénes serán los clientes del entorno?
* ¿Qué región de Azure usaremos para la prueba de concepto? 
* ¿Tenemos una lista de servicios que los proveedores pueden usar a través de DevTest Labs además de IaaS (VM)? 
* ¿Cómo se planea el entrenamiento de los proveedores y los usuarios sobre el uso del laboratorio? 

### <a name="components-of-the-proof-of-concept-solution"></a>Componentes de la solución de prueba de concepto 

Esperamos que la solución tenga los componentes siguientes: 

* Varios equipos de proveedores utilizarán un conjunto de laboratorios en Azure.
* Los laboratorios están conectados a una infraestructura de red compatible con los requisitos.
* Los proveedores tienen acceso a los laboratorios a través de Azure AD y las asignaciones de roles.
* Los proveedores cuentan con una manera de conectarse correctamente a sus recursos. En concreto, una VPN de sitio a sitio permite el acceso directo a máquinas virtuales sin direcciones IP públicas.
* Un conjunto de artefactos cubre el software requerido que necesitan los proveedores en las máquinas virtuales.

## <a name="additional-planning-and-design-decisions"></a>Decisiones de planeamiento y diseño adicionales 

Antes de lanzar una solución completa de DevTest Labs, hay que tomar algunas decisiones importantes sobre planeamiento y diseño. La experiencia de trabajar en una prueba de concepto puede ayudarle a tomar estas decisiones. Tenga en cuenta lo siguiente: 

* **Topología de la suscripción**: Los requisitos de nivel empresarial para los recursos de Azure pueden extenderse más allá de las [cuotas disponibles dentro de una sola suscripción](../azure-resource-manager/management/azure-subscription-service-limits.md). Se requerirían así varias suscripciones o solicitudes de servicio de Azure para aumentar los límites iniciales de la suscripción. Es importante decidir con antelación cómo distribuir los recursos entre las suscripciones. Un recurso valioso es la [guía para la toma de decisiones sobre las suscripciones](/azure/architecture/cloud-adoption/decision-guides/subscriptions/) porque es difícil trasladar los recursos a otra suscripción más adelante. Por ejemplo, un laboratorio no se puede trasladar a otra suscripción después de crearlo.  
* **Topología de red**: La [infraestructura de red predeterminada](../app-service/networking-features.md) creada automáticamente por DevTest Labs puede no ser suficiente para cumplir los requisitos y las restricciones de los usuarios empresariales. Es habitual ver [redes virtuales conectadas de Azure ExpressRoute](/azure/architecture/reference-architectures/hybrid-networking/), [conexiones de concentrador y radio](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) para la conectividad entre suscripciones e incluso el [enrutamiento forzado](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md) para garantizar únicamente la conectividad local. DevTest Labs permite que la redes virtuales existentes se conecten al laboratorio para habilitar su uso al crear máquinas virtuales nuevas en el laboratorio. 
* **Acceso remoto de máquinas virtuales**: Hay muchas opciones para obtener acceso de forma remota a las máquinas virtuales que se encuentran en DevTest Labs. Lo más sencillo es usar direcciones IP públicas o direcciones IP públicas compartidas. Estos son [los valores de configuración disponibles en el laboratorio](devtest-lab-shared-ip.md). Si estas opciones no son suficientes, el uso de una puerta de enlace de acceso remoto también es una opción, tal como se muestra en la [arquitectura de referencia empresarial de DevTest Labs](devtest-lab-reference-architecture.md) y se describe más adelante en la [documentación de puerta de enlace de Escritorio remoto de DevTest Labs](configure-lab-remote-desktop-gateway.md). Las empresas también pueden usar ExpressRoute o una VPN de sitio a sitio para conectar sus laboratorios a su red local. Esta opción permite conexiones directas de Escritorio remoto o SSH a máquinas virtuales en función de su dirección IP privada sin exposición en Internet. 
* **Tratamiento de permisos**: Los dos permisos clave que se usan habitualmente en DevTest Labs son [Propietario y Usuario del laboratorio](devtest-lab-add-devtest-user.md). Es importante decidir antes de implementar DevTest Labs ampliamente en quién se confiará en cada nivel de acceso en el laboratorio. Un modelo común es el propietario del presupuesto (líder de equipo, por ejemplo) como propietario del laboratorio y los miembros del equipo como usuarios del laboratorio. Este modelo permite a la persona responsable del presupuesto (líder de equipo) ajustar la configuración de la directiva para mantener el equipo dentro del presupuesto.  

## <a name="completing-the-proof-of-concept"></a>Finalización de la prueba de concepto 

Una vez que se han tratado los aprendizajes esperados, es el momento de completar el piloto. Este es el momento de recopilar comentarios de los usuarios, determinar si el piloto se realizó correctamente y decidir si la organización va a avanzar en un lanzamiento de escalabilidad de DevTest Labs en la empresa. También es un buen momento para considerar la automatización de la implementación de DevTest Labs y los recursos asociados para garantizar la coherencia a lo largo del lanzamiento de escalabilidad. 

## <a name="next-steps"></a>Pasos siguientes 

* [DevTest Labs en la empresa](devtest-lab-guidance-prescriptive-adoption.md)
* [Arquitectura de referencia para la empresa](devtest-lab-reference-architecture.md)
* [Scaling up your DevTest Labs deployment](devtest-lab-guidance-orchestrate-implementation.md) (Escalado vertical de la implementación de DevTest Labs)
* [Orquestación de la implementación de Azure DevTest Labs](devtest-lab-guidance-orchestrate-implementation.md)
