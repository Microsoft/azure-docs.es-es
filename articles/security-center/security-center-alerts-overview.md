---
title: Alertas de seguridad en Azure Security Center | Microsoft Docs
description: En este tema se explica qué son las alertas de seguridad y los distintos tipos disponibles en Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 1b71e8ad-3bd8-4475-b735-79ca9963b823
ms.service: security-center
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: memildin
ms.openlocfilehash: 25b5569c123a4c7376aecd01b8e36f68b0a8e9b5
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86519614"
---
# <a name="security-alerts-in-azure-security-center"></a>Alertas de seguridad en Azure Security Center

En Azure Security Center, existe una amplia variedad de alertas para los numerosos y diversos tipos de recursos. Security Center genera alertas para los recursos implementados en Azure, así como para los recursos implementados en entornos locales y entornos en la nube híbrida.

Una serie de medidas de detección avanzadas se ocupan de desencadenar las alertas de seguridad y solo están disponibles en el nivel Estándar de Azure Security Center. Hay una evaluación gratuita disponible. Puede actualizar su versión desde la página **Precios y configuración**. Obtenga más información sobre los [precios de Security Center](https://azure.microsoft.com/pricing/details/security-center/).

## <a name="responding-to-todays-threats"></a>Respuesta a las amenazas actuales <a name="respond-threats"> </a>

En los últimos 20 años, las amenazas han cambiado considerablemente. Por lo general, en el pasado, las compañías solo tenían que preocuparse por las alteraciones que atacantes individuales realizaban en sus sitios web, quienes normalmente solo pretendían comprobar “qué es lo que eran capaces de hacer”. Hoy en día, los atacantes son mucho más sofisticados y están más organizados. Normalmente, se mueven por un fin económico o estratégico. También disponen de más recursos, ya que a veces están financiados por estados o por el crimen organizado.

Estas realidades cambiantes han propiciado un nivel de profesionalidad entre los atacantes que no tiene precedentes. Ya no les interesa alterar los sitios web. Ahora, lo que buscan es robar información, cuentas financieras y datos privados, que pueden usar con fines lucrativos en el mercado libre o para sacar provecho de una determinada situación política, militar o empresarial. Si los atacantes con un objetivo financiero son motivo de preocupación, aún lo son más los que pretenden abrir una brecha en las redes para dañar la infraestructura o infligir daños personales.

Como respuesta, las organizaciones suelen implementar varias soluciones específicas, que se centran en defender el perímetro o los puntos de conexión de la empresa mediante la búsqueda de firmas de ataque conocidas. Estas soluciones suelen generar un gran número de alertas con un bajo nivel de confiabilidad, lo que requiere que un analista de seguridad las cribe e investigue. La mayor parte de las organizaciones no disponen de la experiencia ni del tiempo necesarios para responder a estas alertas, por lo que muchas quedan desatendidas.  

Además, los atacantes han mejorado sus métodos para debilitar muchas de las defensas basadas en firmas y para [adaptarse a los entornos en la nube](https://azure.microsoft.com/blog/detecting-threats-with-azure-security-center/). Por tanto, se necesitan nuevos enfoques que permitan identificar con mayor rapidez las nuevas amenazas, así como acelerar su detección y respuesta.

## <a name="what-are-security-alerts-and-security-incidents"></a>¿Qué son las alertas de seguridad y los incidentes de seguridad? 

Las **alertas** son notificaciones que Security Center genera cuando detecta amenazas en los recursos. Security Center asigna prioridades y enumera las alertas, junto con la información necesaria para que pueda investigar rápidamente el problema. Security Center también proporciona recomendaciones sobre el modo en que puede corregir un ataque.

Un **incidente de seguridad** es una recopilación de alertas relacionadas, en lugar de una enumeración de alertas individuales. Security Center usa la [correlación de alertas inteligentes en la nube](security-center-alerts-cloud-smart.md) para poner en correlación diferentes alertas y señales de baja fidelidad en incidentes de seguridad.

Utilizando los incidentes, Security Center proporciona una vista única de una campaña de ataques y todas las alertas relacionadas. Esta vista le permite comprender rápidamente las acciones que ha realizado el atacante y qué recursos se han visto afectados. Para obtener más información, consulte [Correlación de alertas inteligentes en la nube](security-center-alerts-cloud-smart.md).



## <a name="how-does-security-center-detect-threats"></a>¿Cómo detecta Security Center las amenazas? <a name="detect-threats"> </a>

Los investigadores de seguridad de Microsoft trabajan sin descanso para localizar amenazas. Dada la presencia global de Microsoft en la nube y en sistemas locales, tienen acceso a un amplio conjunto de recursos de telemetría. La amplitud y diversidad de estos conjuntos de datos permite detectar nuevos patrones y tendencias de ataque tanto en sus productos locales, destinados a particulares y empresas, como en sus servicios en línea. Como resultado, Security Center es capaz de actualizar rápidamente los algoritmos de detección a medida que los atacantes idean nuevas y más sofisticadas vulnerabilidades de seguridad. Este enfoque le ayuda a mantenerse al día en entornos llenos de amenazas que cambian continuamente.

Para detectar amenazas reales y reducir los falsos positivos, Security Center recopila, analiza e integra los datos de registro de los recursos de Azure y de la red. También funciona con soluciones de asociados conectados, como firewalls y soluciones de protección de puntos de conexión. Security Center analiza estos datos (a menudo, relacionando la información de diferentes orígenes) para identificar las amenazas.

![Recopilación y presentación de datos de Security Center](./media/security-center-alerts-overview/security-center-detection-capabilities.png)

Security Center utiliza análisis avanzados que superan con creces los enfoques basados en firmas. Los grandes avances registrados en las tecnologías de macrodatos y [aprendizaje automático](https://azure.microsoft.com/blog/machine-learning-in-azure-security-center/) se utilizan para evaluar eventos en todo el tejido de la nube, lo que permite detectar amenazas que no se podrían identificar mediante enfoques manuales, así como predecir la evolución de los ataques. Estas técnicas de análisis son:

* **Información integrada sobre amenazas**: Microsoft dispone de una ingente cantidad de información sobre amenazas globales. Los recursos telemétricos proceden de diferentes fuentes, como Azure, Microsoft 365, Microsoft CRM Online, Microsoft Dynamics AX, outlook.com, MSN.com, la Unidad de crímenes digitales de Microsoft (DCU) y Microsoft Security Response Center (MSRC). Los investigadores también reciben información sobre amenazas que comparten con los principales proveedores de servicios en la nube y que procede de fuentes de otros terceros. Azure Security Center puede usar todos estos datos para avisarle de las amenazas procedentes de actores malintencionados conocidos.

* **Análisis del comportamiento**: El análisis del comportamiento es una técnica que analiza datos y los compara con una serie de patrones conocidos. No obstante, estos patrones no son simples firmas, sino que están determinados por unos complejos algoritmos de aprendizaje automático que se aplican a conjuntos de datos masivos. También se determinan por medio de un análisis cuidadoso, llevado a cabo por analistas expertos, de los comportamientos malintencionados. Azure Security Center puede utilizar el análisis del comportamiento para identificar recursos en peligro a partir del análisis de registros de las máquinas virtuales, registros de los dispositivos de redes virtuales, registros de los tejidos y otros orígenes.

* **Detección de anomalías**: Azure Security Center también utiliza la detección de anomalías para identificar amenazas. A diferencia del análisis del comportamiento, que depende de patrones conocidos obtenidos a partir de grandes conjuntos de datos, la detección de anomalías es una técnica más "personalizada" y se basa en referencias que son específicas de las implementaciones. El aprendizaje automático se aplica para determinar la actividad normal de las implementaciones. A partir de ahí, se generan reglas para definir condiciones de valores atípicos que podrían constituir un evento de seguridad.

## <a name="how-are-alerts-classified"></a>¿Cómo se clasifican las alertas?

Security Center asigna una gravedad a las alertas, que le ayudan a priorizar el orden en el que asiste a cada alerta, de modo que cuando un recurso está en peligro, puede obtenerlo inmediatamente. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

> [!NOTE]
> La gravedad de las alertas aparece de manera diferente en el portal y en las versiones de la API REST anteriores a 01-01-2019. Si usa una versión anterior de la API, actualícela para conseguir la experiencia coherente que se describe a continuación.

- **Alto:** hay una probabilidad elevada de que el recurso esté en peligro. Debe investigarse de inmediato. El grado de certeza de Security Center sobre la mala intención de la acción y los hallazgos utilizados para emitir la alerta es elevado. Una alerta de este tipo sería podría detectar la ejecución de una herramienta malintencionada conocida; por ejemplo, Mimikatz, una herramienta que se usa habitualmente para robar credenciales.
- **Medio:** es probable que sea una actividad sospechosa que puede indicar que un recurso está en peligro.
El grado de certeza de Security Center sobre el análisis o los hallazgos es medio, mientras que el grado de certeza sobre la mala intención es medio o alto. Suelen tratarse de detecciones basadas en anomalías o aprendizaje automático. Por ejemplo, un intento de inicio de sesión desde una ubicación anómala.
- **Bajo:** podría tratarse de un hallazgo benigno o de un ataque bloqueado.
   * Security Center no tiene la certeza suficiente de que la intención fuera mala y la actividad podría ser inofensiva. Por ejemplo, borrar un registro es una acción que podría producirse si un atacante intenta ocultar sus huellas, pero en muchos casos es una operación rutinaria que realizan los administradores.
   * Por lo general, Security Center no avisa cuando se bloquean ataques a menos que se considere un caso interesante que convenga examinar. 
- **Informativo:** solo verá las alertas informativas cuando explore en profundidad un incidente de seguridad, o si usa la API REST con un determinado identificador de alerta. Normalmente, las incidencias se componen de varias alertas, algunas de las cuales pueden parecer meramente informativas, aunque a tenor de otras alertas puede ser conveniente investigarlas. 

## <a name="continuous-monitoring-and-assessments"></a>Supervisión y evaluaciones continuas

Azure Security Center se beneficia de la existencia de equipos de científicos de datos e investigadores de seguridad de Microsoft que supervisan sin descanso los cambios que se registran en el terreno de las amenazas. Estos son algunas de las iniciativas que llevan a cabo:

* **Supervisión de la inteligencia sobre amenazas**: la inteligencia sobre amenazas incluye mecanismos, indicadores, implicaciones y notificaciones para las amenazas nuevas o existentes. Esta información se comparte con la comunidad de seguridad, y Microsoft supervisa sin descanso las fuentes de orígenes internos y externos.
* **Uso compartido de señales**: la información que recopilan los equipos de seguridad en la amplia cartera de servicios tanto locales como en la nube, de servidores y de dispositivos cliente de punto de conexión de Microsoft se comparte y se analiza.
* **Especialistas en seguridad de Microsoft** colaboración continua con equipos de Microsoft que trabajan en campos de seguridad especializados, como análisis forense y detección de ataques web.
* **Ajuste de la detección**: los algoritmos se ejecutan en conjuntos de datos de clientes reales y los investigadores de seguridad trabajan en conjunción con los clientes para validar los resultados. Los falsos positivos y los positivos verdaderos se utilizan para perfeccionar los algoritmos de aprendizaje automático.

Toda esta combinación de esfuerzos culmina en nuevas y mejoradas técnicas de detección, de las que puede beneficiarse al instante sin ninguna acción por su parte.


## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido obtenido información sobre los distintos tipos de alertas disponibles en Security Center. Para más información, consulte:

* [Protección contra amenazas en Azure Security Center](threat-protection.md): para una breve descripción de los orígenes de las alertas de seguridad mostradas en Azure Security Center 
* **Alertas de seguridad en el registro de actividad de Azure**: además de estar disponibles en Azure Portal o mediante programación, las alertas y los incidentes de seguridad se auditan como eventos en el [registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view). Para obtener más información sobre el esquema de eventos, consulte el artículo sobre [alertas de seguridad en el registro de actividad de Azure](https://go.microsoft.com/fwlink/?linkid=2114113).

