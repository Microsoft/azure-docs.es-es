---
title: Detección de amenazas avanzada de Azure | Microsoft Docs
description: Más información sobre la funcionalidad de detección de amenazas avanzada para Azure como, por ejemplo, el servicio Azure AD Identity Protection.
services: security
documentationcenter: na
author: UnifyCloud
manager: barbkess
editor: TomSh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2017
ms.author: TomSh
ms.openlocfilehash: 1522d143f2837a57481a4d678911e7a3c44d084d
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87541029"
---
# <a name="azure-advanced-threat-detection"></a>Detección de amenazas avanzada de Azure

Azure ofrece funciones de detección de amenazas avanzada integradas en servicios como Azure Active Directory (Azure AD), registros de Azure Monitor y Azure Security Center. Esta colección de servicios de seguridad y funcionalidades ofrece una manera sencilla y rápida de comprender lo que ocurre en las implementaciones de Azure.

Azure proporciona una amplia gama de opciones para configurar y personalizar la seguridad para que cumpla los requisitos de las implementaciones de la aplicación. En este artículo se explica cómo cumplir estos requisitos.

## <a name="azure-active-directory-identity-protection"></a>Azure Active Directory Identity Protection

[Azure AD Identity Protection](../../active-directory/identity-protection/overview.md) es una característica de la edición [Azure Active Directory Premium P2](../../active-directory/active-directory-whatis.md) que proporciona información general de las detecciones de riesgo y las posibles vulnerabilidades que pueden afectar a las identidades de la organización. Identity Protection usa las funcionalidades de detección de anomalías existentes de Azure AD que están disponibles mediante [informes de actividad anómala de Azure AD](../../active-directory/active-directory-reporting-azure-portal.md) e introduce nuevos tipos de detección de riesgo que pueden detectar anomalías en tiempo real.

![Diagrama de Azure AD Identity Protection](./media/threat-detection/azure-threat-detection-fig1.png)

En Identity Protection se usan algoritmos y heurística de aprendizaje automático adaptable para detectar anomalías y detecciones de riesgo que es posible que indiquen que una identidad se ha puesto en peligro. Con estos datos, Identity Protection genera informes y alertas que permiten investigar estas detecciones de riesgo y tomar las acciones de corrección o mitigación adecuadas.

Azure Active Directory Identity Protection es más que una herramienta de supervisión e informes. En función de las detecciones de riesgo, Identity Protection calcula un nivel de riesgo del usuario para cada usuario, de modo que pueda configurar las directivas basadas en riesgos con el fin de proteger de forma automática las identidades de la organización.

Estas directivas basadas en riesgos, además de otros [controles de acceso condicional](../../active-directory/active-directory-conditional-access-azure-portal.md) proporcionados por Azure Active Directory y [EMS](../../active-directory/active-directory-conditional-access-azure-portal.md), pueden bloquear automáticamente u ofrecer acciones de corrección adaptables que incluyen el restablecimiento de contraseñas y el cumplimiento de la autenticación multifactor.

### <a name="identity-protection-capabilities"></a>Funcionalidades de Identity Protection

Azure Active Directory Identity Protection es más que una herramienta de supervisión e informes. Para proteger las identidades de la organización, puede configurar directivas basadas en riesgos que respondan automáticamente a problemas detectados si se alcanza un nivel de riesgo específico. Estas directivas, además de otros controles de acceso condicional proporcionados por Azure Active Directory y EMS, pueden bloquear automáticamente o iniciar acciones de corrección adaptables que incluyen el restablecimiento de contraseñas y el cumplimiento de la autenticación multifactor.

Ejemplos de algunas de las maneras en que Azure Identity Protection puede ayudar a proteger las cuentas e identidades:

[Detecciones de riesgo y cuentas peligrosas](../../active-directory/identity-protection/overview.md)
-   Detecte seis tipos de detecciones de riesgo mediante el aprendizaje automático y las reglas heurísticas.
-   Calcular los niveles de riesgo del usuario.
-   Proporcionar recomendaciones personalizadas para mejorar la posición de seguridad general al resaltar los puntos vulnerables.

[Investigación de las detecciones de riesgo](../../active-directory/identity-protection/overview.md)
-   Enviar notificaciones de las detecciones de riesgo.
-   Investigar las detecciones de riesgo con información pertinente y contextual.
-   Proporcionar los flujos de trabajo básicos para realizar un seguimiento de las investigaciones.
-   Proporcionar un acceso fácil a las acciones de corrección, como el restablecimiento de contraseñas.

[Directivas de acceso condicional basadas en riesgos](../../active-directory/identity-protection/overview.md)
-   Mitigar los inicios de sesión peligrosos mediante el bloqueo de los inicios de sesión o requerir desafíos de la autenticación multifactor.
-   Proteger o bloquear cuentas de usuario peligrosas.
-   Exigir a los usuarios que se registren en la autenticación multifactor.

### <a name="azure-ad-privileged-identity-management"></a>Administración de identidades con privilegios de Azure AD

Con [Azure Active Directory Privileged Identity Management (PIM)](../../active-directory/privileged-identity-management/pim-configure.md), puede administrar, controlar y supervisar el acceso dentro de la organización. Esta característica incluye el acceso a los recursos de Azure AD y otros servicios en línea de Microsoft, como Office 365 o Microsoft Intune.

![Diagrama de Azure AD Privileged Identity Management](./media/threat-detection/azure-threat-detection-fig2.png)

PIM ayuda a:

-   Obtener alertas e informes sobre los administradores de Azure AD y el acceso administrativo Just-In-Time (JIT) a los servicios en línea de Microsoft, como Office 365 e Intune.

-   Obtener informes sobre el historial de acceso de administrador y los cambios en las asignaciones de administrador.

-   Obtener alertas sobre el acceso a un rol con privilegios.

## <a name="azure-monitor-logs"></a>Registros de Azure Monitor

Los [registros de Azure Monitor](../../azure-monitor/index.yml) son una solución de administración de TI basada en la nube de Microsoft que permiten administrar y proteger la infraestructura local y en la nube. Como los registros de Azure Monitor se implementan como un servicio basado en la nube, puede ponerlos en funcionamiento rápidamente con una inversión mínima en servicios de infraestructura. Las características nuevas de seguridad se entregan de forma automática, lo que supone un ahorro en costos permanentes de mantenimiento y actualización.

Además de proporcionar servicios útiles, los registros de Azure Monitor se pueden integrar con componentes de System Center, como [System Center Operations Manager](https://blogs.technet.microsoft.com/cbernier/2013/10/23/monitoring-windows-azure-with-system-center-operations-manager-2012-get-me-started/), para ampliar a la nube las inversiones de administración de seguridad existentes. System Center y los registros de Azure Monitor pueden funcionar de forma conjunta para ofrecer una experiencia de administración híbrida completa.

### <a name="holistic-security-and-compliance-posture"></a>Seguridad integral y postura de cumplimiento

En el [panel Seguridad y auditoría de Log Analytics](../../security-center/security-center-intro.md), se muestra una vista completa de la posición de seguridad de TI de la organización, con consultas de búsqueda integradas para problemas relevantes que necesiten su atención. El panel Seguridad y auditoría es la pantalla principal para todo lo relacionado con la seguridad en los registros de Azure Monitor. Proporciona información detallada sobre el estado de seguridad de los equipos. También se pueden ver todos los eventos de las últimas 24 horas, siete días o cualquier otro intervalo personalizado.

Los registros de Azure Monitor permiten comprender de forma rápida y sencilla la posición de seguridad global de cualquier entorno, todo ello en el contexto de las operaciones de TI, como la evaluación de actualizaciones de software, la evaluación de antimalware y las líneas base de configuración. Los datos del registro de seguridad son accesibles en todo momento para simplificar los procesos de auditoría de seguridad y cumplimiento.

![Panel Seguridad y auditoría de Log Analytics](./media/threat-detection/azure-threat-detection-fig3.jpg)

El panel Seguridad y auditoría de Log Analytics se divide en cuatro categorías principales:

-   **Dominios de seguridad**: permite explorar más los registros de seguridad a lo largo del tiempo, acceder a evaluaciones de malware, actualizar las evaluaciones, ver información de seguridad de la red, identidad y acceso, ver equipos con eventos de seguridad y acceder rápidamente al panel de Azure Security Center.

-   **Problemas importantes**: permite identificar de forma rápida la cantidad de problemas activos y su gravedad.

-   **Detecciones (versión preliminar)** : permite identificar los patrones de ataque mediante la visualización de alertas de seguridad a medida que se producen contra sus recursos.

-   **Información sobre amenazas**: permite identificar los patrones de ataque mediante la visualización del número total de servidores con tráfico IP malintencionado de salida, el tipo de amenaza malintencionada y un mapa de las ubicaciones IP.

-   **Consultas comunes de seguridad**: enumera las consultas de seguridad más comunes que se pueden usar para supervisar el entorno. Al seleccionar cualquier consulta, se abre el panel de búsqueda y se muestran los resultados para esa consulta.

### <a name="insight-and-analytics"></a>Insight y Analytics
En el centro de los [registros de Azure Monitor](../../log-analytics/log-analytics-queries.md) se encuentra el repositorio de OMS, que está hospedado en Azure.

![Diagrama de Insight y Analytics](./media/threat-detection/azure-threat-detection-fig4.png)

Los datos se recopilan en el repositorio desde los orígenes conectados mediante la configuración de orígenes de datos y la incorporación de soluciones a la suscripción.

![El panel de registros de Azure Monitor](./media/threat-detection/azure-threat-detection-fig5.png)

Cada uno de los orígenes de datos y soluciones crea tipos de registros distintos con su propio conjunto de propiedades, pero se pueden seguir analizando de forma conjunta en consultas al repositorio. Se pueden usar las mismas herramientas y métodos para trabajar con una variedad de datos que se recopilan mediante diversos orígenes.


La mayor parte de la interacción con los registros de Azure Monitor se realiza mediante Azure Portal, que se ejecuta en cualquier explorador y proporciona acceso a opciones de configuración y a herramientas para analizar los datos recopilados y realizar acciones en estos. Desde el portal, puede usar:
* [Búsquedas de registros](../../log-analytics/log-analytics-queries.md) donde se crean consultas para analizar los datos recopilados.
* [Paneles](../../azure-monitor/learn/tutorial-logs-dashboards.md), que se pueden personalizar con vistas gráficas de las búsquedas más valiosas.
* [Soluciones](../../monitoring/monitoring-solutions.md), que proporcionan herramientas de análisis y funcionalidad adicionales.

![Herramientas de análisis](./media/threat-detection/azure-threat-detection-fig6.png)

Las soluciones agregan funcionalidad a los registros de Azure Monitor. Se ejecutan principalmente en la nube y proporcionan análisis de los datos recopilados en el repositorio de Log Analytics. Puede que las soluciones también definan nuevos tipos de registro para recopilar que se pueden analizar con las búsquedas de registros o mediante una interfaz de usuario adicional que la solución proporciona en el panel de Log Analytics.

El panel Seguridad y auditoría es un ejemplo de estos tipos de solución.

### <a name="automation-and-control-alert-on-security-configuration-drifts"></a>Automatización y control: desviación de la alerta de configuración de seguridad

Azure Automation automatiza los procesos administrativos con runbooks que se basan en PowerShell y se ejecutan en la nube. Los runbooks pueden ejecutarse en un servidor en el centro de datos local para administrar recursos locales. Azure Automation proporciona administración de configuración con Desired State Configuration (DSC) de PowerShell.

![Diagrama de Azure Automation](./media/threat-detection/azure-threat-detection-fig7.png)

Puede crear y administrar recursos de DSC hospedados en Azure y aplicarlos a los sistemas de nube y locales. Al hacerlo, puede definir y aplicar de forma automática su configuración, o bien obtener informes de la desviación para ayudar a garantizar que las configuraciones de seguridad se ajusten siempre a las directivas.

## <a name="azure-security-center"></a>Azure Security Center

Azure Security Center ayuda a proteger los recursos de Azure. Proporciona una supervisión de la seguridad y una administración de directivas integradas en suscripciones de Azure. En el servicio, puede definir directivas para las suscripciones de Azure y los [grupos de recursos](../../azure-resource-manager/management/manage-resources-portal.md) para obtener una mayor granularidad.

![Diagrama de Azure Security Center](./media/threat-detection/azure-threat-detection-fig8.png)

Los investigadores de seguridad de Microsoft trabajan sin descanso para localizar amenazas. Tienen acceso al amplio conjunto de recursos de telemetría que les proporciona la presencia global de Microsoft en la nube y en sistemas locales. La amplitud y diversidad de estos conjuntos de datos permite a Microsoft detectar nuevos patrones y tendencias de ataque tanto en sus productos locales, destinados a particulares y empresas, como en sus servicios en línea.

Por lo tanto, Security Center es capaz de actualizar rápidamente los algoritmos de detección a medida que los atacantes idean nuevos y más sofisticados ataques de seguridad. Este enfoque le ayuda a mantenerse al día en entornos con amenazas que cambian continuamente.

![Detección de amenazas de Security Center](./media/threat-detection/azure-threat-detection-fig9.jpg)

La detección de amenazas de Security Center recopila automáticamente información de seguridad de sus recursos de Azure, de la red y de soluciones de asociados relacionadas. Analiza estos datos (a menudo, al relacionar la información de diferentes orígenes) para identificar las amenazas.

En Security Center, las alertas de seguridad están clasificadas por prioridad y se incluyen recomendaciones para solucionar la amenaza.

Security Center utiliza análisis avanzados que superan con creces los enfoques basados en firmas. Se usan innovaciones en tecnologías de macrodatos y [aprendizaje automático](https://azure.microsoft.com/blog/machine-learning-in-azure-security-center/) para evaluar eventos en todo el tejido en la nube. El análisis avanzado puede detectar amenazas que sería imposible identificar mediante enfoques manuales y predice la evolución de los ataques. Estos tipos de análisis de seguridad se tratan en las secciones siguientes.

### <a name="threat-intelligence"></a>Información sobre amenazas

Microsoft tiene acceso a una cantidad ingente de información sobre amenazas globales.

Los recursos telemétricos proceden de diferentes fuentes, como Azure, Office 365, Microsoft CRM Online, Microsoft Dynamics AX, outlook.com, MSN.com, la Unidad de crímenes digitales de Microsoft (DCU) y Microsoft Security Response Center (MSRC).

![Conclusiones de inteligencia sobre amenazas](./media/threat-detection/azure-threat-detection-fig10.jpg)

Los investigadores también cuentan con la información de inteligencia sobre amenazas que comparten los principales proveedores de servicios en la nube y que procede de fuentes de terceros. Azure Security Center puede usar todos estos datos para avisarle de las amenazas procedentes de actores malintencionados conocidos. Estos son algunos ejemplos:

-   **Aprovechamiento de la capacidad del aprendizaje automático**: Azure Security Center tiene acceso a una gran cantidad de datos sobre la actividad de red en la nube, que se puede usar para detectar amenazas dirigidas a las implementaciones de Azure.

-   **Detección de fuerza bruta**: el aprendizaje automático se usa para crear un patrón histórico de los intentos de acceso remoto, lo que permite detectar los ataques de fuerza bruta contra los puertos Secure Shell (SSH), Protocolo de escritorio remoto (RDP) y SQL.

-   **Salida DDoS y detección de botnet**: un objetivo común de los ataques dirigidos a los recursos de nube consiste en usar la capacidad de proceso de estos recursos para ejecutar otros ataques.

-   **Nuevos servidores de análisis de comportamiento y máquinas virtuales**: después de poner en peligro un servidor o una máquina virtual, los atacantes emplean una gran variedad de técnicas para ejecutar código malintencionado en el sistema sin ser detectados, lo que garantiza la persistencia y consigue evitar los controles de seguridad.

-   **Detección de amenazas de Azure SQL Database**: la detección de amenazas de Azure SQL Database identifica actividades anómalas de la base de datos que indican intentos inusuales o posiblemente dañinos de acceso o ataque a las bases de datos.

### <a name="behavioral-analytics"></a>Análisis del comportamiento

El análisis del comportamiento es una técnica que analiza datos y los compara con una serie de patrones conocidos. No obstante, estos patrones no son simples firmas, sino que están determinados por unos complejos algoritmos de aprendizaje automático que se aplican a conjuntos de datos masivos.

![Conclusiones del análisis de comportamiento](./media/threat-detection/azure-threat-detection-fig11.jpg)

Los patrones también se determinan por medio de un análisis cuidadoso, llevado a cabo por analistas expertos, de los comportamientos malintencionados. Azure Security Center puede utilizar el análisis del comportamiento para identificar recursos en peligro a partir del análisis de registros de las máquinas virtuales, registros de los dispositivos de redes virtuales, registros de los tejidos, volcados de memoria y otros orígenes.

Además, los patrones se ponen en correlación con otras señales que se comprueban para fundamentar las pruebas de que se trata una campaña de gran difusión. Dicha correlación permite identificar eventos que se ajustan a unos indicadores de peligro establecidos.

Estos son algunos ejemplos:
-   **Ejecución de procesos sospechosos**: los atacantes utilizan varias técnicas para ejecutar software malintencionado sin que se detecte. Por ejemplo, un atacante podría asignar a un malware los mismos nombres que se usan en los archivos legítimos del sistema, pero colocarlos en otras ubicaciones, usar un nombre muy parecido al de un archivo legítimo o enmascarar la verdadera extensión del archivo. Security Center adapta los comportamientos de los procesos y supervisa su ejecución para detectar valores atípicos como estos.

-   **Malware oculto e intentos de aprovecharse de vulnerabilidades de seguridad**: el malware sofisticado puede eludir los productos antimalware tradicionales, para lo que evita escribir en el disco o cifra los componentes de software almacenados en el disco. Pero este malware se puede detectar mediante un análisis de memoria, ya que, para funcionar, deja rastros en la memoria. En el momento en que el software se bloquea, un volcado de memoria captura una porción de la memoria. Con un análisis de la memoria durante el volcado, Azure Security Center puede detectar las técnicas usadas para aprovechar las vulnerabilidades del software, acceder a información confidencial y permanecer en secreto en un equipo afectado sin que esto afecte a su rendimiento.

-   **Movimiento lateral y reconocimiento interno**: para poder permanecer en una red afectada y localizar y recopilar información valiosa, los atacantes suelen tratar de desplazar lateralmente el contenido de la máquina afectada a otras de la misma red. Security Center supervisa las actividades de los procesos e inicios de sesión para detectar cualquier intento de expandir el punto de apoyo del atacante en la red, como sondeos de redes de ejecución remota de comandos y enumeración de cuentas.

-   **Scripts de PowerShell malintencionados**: los atacantes usan PowerShell para ejecutar código malintencionado en las máquinas virtuales objetivo con diferentes propósitos. Security Center analiza la actividad de PowerShell en busca de evidencias de actividad sospechosa.

-   **Ataques de salida**: a menudo, el objetivo de los atacantes son recursos en la nube, que utilizan con el propósito de perpetrar otros ataques. Por ejemplo, es posible que las máquinas virtuales afectadas se usen para iniciar ataques por fuerza bruta contra otras máquinas virtuales, enviar correo no deseado o buscar puertos abiertos y otros dispositivos en Internet. Gracias a la aplicación del aprendizaje automático en el tráfico de red, Security Center puede detectar cuándo las comunicaciones de red salientes superan la norma establecida. Cuando se detecta correo no deseado, Security Center relaciona el tráfico inusual de correo electrónico con la información de Office 365 para determinar si es probable que el mensaje sea fraudulento o si es el resultado de una campaña de correo electrónico legítima.

### <a name="anomaly-detection"></a>Detección de anomalías

Azure Security Center también utiliza la detección de anomalías para identificar amenazas. A diferencia del análisis del comportamiento, que depende de patrones conocidos obtenidos a partir de grandes conjuntos de datos, la detección de anomalías es una técnica más “personalizada” y se basa en referencias que son específicas de las implementaciones. El aprendizaje automático se aplica para determinar la actividad normal de las implementaciones y, después, se generan reglas para definir condiciones de valores atípicos que podrían constituir un evento de seguridad. Veamos un ejemplo:

-   **Ataques ataque por fuerza bruta de RDP/SSH de entrada**: es posible que las implementaciones integren máquinas virtuales con mucha actividad que tengan multitud de inicios de sesión al día y otras máquinas virtuales que tengan pocos inicios de sesión o ninguno. Azure Security Center puede determinar la actividad de referencia de los inicios de sesión de estas máquinas virtuales y utilizar el aprendizaje automático para definir las actividades relacionadas con el inicio de sesión normal. Si hay alguna discrepancia con la línea de base definida para las características relacionadas con el inicio de sesión, es posible que se genere una alerta. Una vez más, es el aprendizaje automático el que se encarga de determinar qué es significativo.

### <a name="continuous-threat-intelligence-monitoring"></a>Supervisión continuada de la información sobre amenazas

Azure Security Center cuenta con equipos de científicos de datos e investigadores de seguridad de todo el mundo que supervisan sin descanso los cambios que se registran en el terreno de las amenazas. Estos son algunas de las iniciativas que llevan a cabo:

-   **Supervisión de la inteligencia sobre amenazas**: la inteligencia sobre amenazas incluye mecanismos, indicadores, implicaciones y notificaciones para las amenazas nuevas o existentes. Esta información se comparte con la comunidad de seguridad, y Microsoft supervisa sin descanso las fuentes de inteligencia sobre amenazas de orígenes internos y externos.

-   **Uso compartido de señales**: se comparte y analiza la información que recopilan los equipos de seguridad a partir de la amplia cartera de servicios de Microsoft en la nube y locales, de servidores y de dispositivos cliente de punto de conexión.

-   **Especialistas en seguridad de Microsoft** colaboración continua con equipos de Microsoft que trabajan en campos de seguridad especializados, como análisis forense y detección de ataques web.

-   **Ajuste de la detección**: los algoritmos se ejecutan en conjuntos de datos de clientes reales y los investigadores de seguridad trabajan en conjunción con los clientes para validar los resultados. Los falsos positivos y los positivos verdaderos se utilizan para perfeccionar los algoritmos de aprendizaje automático.

Toda esta combinación de esfuerzos culmina en técnicas de detección nuevas y mejoradas, de las que puede beneficiarse al instante. No es necesaria ninguna acción por su parte.

## <a name="advanced-threat-detection-features-other-azure-services"></a>Funciones de detección de amenazas avanzada: Otros servicios de Azure

### <a name="virtual-machines-microsoft-antimalware"></a>Máquinas virtuales: Microsoft Antimalware

[Microsoft Antimalware](antimalware.md) para Azure es una solución de un único agente dirigida a entornos de aplicaciones e inquilinos, diseñada para la ejecución en segundo plano sin intervención humana. Puede implementar la protección en función de las necesidades de sus cargas de trabajo de aplicaciones, con configuración de protección básica o personalizada avanzada que incluye supervisión antimalware. Azure Antimalware es una opción de seguridad para máquinas virtuales de Azure que se instala automáticamente en todas las máquinas virtuales de PaaS de Azure.

#### <a name="microsoft-antimalware-core-features"></a>Características principales de Microsoft Antimalware

Estas son las características de Azure que implementan y habilitan Microsoft Antimalware para las aplicaciones:

-   **Protección en tiempo real**: supervisa la actividad en los servicios en la nube y las máquinas virtuales para detectar y bloquear la ejecución de malware.

-   **Análisis programado**: realiza periódicamente exámenes dedicados a detectar malware, lo que incluye programas que se ejecutan activamente.

-   **Corrección de malware**: actúa automáticamente sobre el malware detectado y elimina o pone en cuarentena los archivos malintencionados y limpia las entradas del Registro malintencionadas.

-   **Actualizaciones de firmas**: instala de forma automática las firmas de protección (definiciones de virus) más recientes para garantizar que la protección está actualizada con una frecuencia determinada previamente.

-   **Actualizaciones de Antimalware Engine**: actualiza automáticamente Microsoft Antimalware Engine.

-   **Actualizaciones de la plataforma antimalware**: actualiza automáticamente la plataforma de Microsoft Antimalware.

-   **Protección activa**: envía a Microsoft Azure informes de metadatos de telemetría sobre las amenazas detectadas y los recursos sospechosos para garantizar una respuesta rápida a las amenazas en constante evolución, lo que permite la entrega sincrónica de firmas en tiempo real a través del sistema de protección activa de Microsoft.

-   **Informes de ejemplos:** proporciona informes de ejemplos al servicio Microsoft Antimalware que ayudan a mejorar el servicio y permiten la solución de problemas.

-   **Exclusiones**: permite a los administradores de aplicaciones y servicios configurar determinados archivos, procesos y unidades para que se excluyan de la protección y el examen por motivos de rendimiento o de otro tipo.

-   **Recopilación de eventos antimalware**: registra el estado del servicio de antimalware, las actividades sospechosas y las acciones de corrección adoptadas en el registro de eventos del sistema operativo y los recopila en la cuenta de Azure Storage del cliente.

### <a name="azure-sql-database-threat-detection"></a>Detección de amenazas de Azure SQL Database

[Detección de amenazas de Azure SQL Database](https://azure.microsoft.com/blog/azure-sql-database-threat-detection-your-built-in-security-expert/) es una nueva característica de inteligencia de seguridad integrada en el servicio de Azure SQL Database. Estamos trabajando permanentemente para conocer y detectar actividades anómalas de la base de datos y generar perfiles; Detección de amenazas de Azure SQL Database identifica las posibles amenazas a la base de datos.

Los responsables de seguridad u otros administradores designados pueden recibir una notificación inmediata sobre las actividades sospechosas en las bases de datos cuando se producen. Cada notificación proporciona detalles de la actividad sospechosa y recomienda cómo investigar más y mitigar la amenaza.

Actualmente, Detección de amenazas de Azure SQL Database detecta posibles vulnerabilidades y ataques de inyección de SQL, así como patrones de acceso anómalos a las bases de datos.

Al recibir una notificación por correo electrónico de detección de amenazas, los usuarios pueden navegar y ver los registros de auditoría pertinentes a través de un vínculo profundo en el correo electrónico. El vínculo abre un visor de auditoría o una plantilla de Excel de auditoría preconfigurada en la que se muestran los registros de auditoría pertinentes en torno a la hora del evento sospechoso, según lo siguiente:

-   El almacenamiento de información de auditoría para el servidor o la base de datos con las actividades anómalas de la base de datos.

-   La tabla de almacenamiento de información de auditoría correspondiente que se usó en el momento del evento para escribir el registro de auditoría.

-   Los registros de auditoría de la hora inmediatamente posterior a la aparición del evento.

-   Los registros de auditoría con identificadores de evento similares en el momento del evento (opcional para algunos detectores).

Los detectores de amenazas de SQL Database usan una de las metodologías de detección siguientes:

-   **Detección determinista**: detecta patrones sospechosos (en función de reglas) en las consultas de cliente SQL que coinciden con ataques conocidos. Esta metodología tiene un grado alto de detección y bajo de falsos positivos; pero la cobertura es limitada, ya que se encuentra dentro de la categoría de "detecciones atómicas".

-   **Detección de comportamiento**: detecta la actividad anómala, que es el comportamiento anómalo de la base de datos que no se haya observado durante los últimos 30 días. Ejemplos de actividad anómala del cliente SQL pueden ser un pico de consultas o inicios de sesión fallidos, un gran volumen de datos extraídos, consultas canónicas inusuales, o bien direcciones IP no familiares que se hayan usado para acceder a la base de datos.

### <a name="application-gateway-web-application-firewall"></a>Firewall de aplicaciones web de Application Gateway

El [firewall de aplicaciones web (WAF)](../../app-service/environment/app-service-app-service-environment-web-application-firewall.md) es una característica de [Azure Application Gateway](../../application-gateway/application-gateway-web-application-firewall-overview.md) que protege las aplicaciones web en las que se usa una puerta de enlace de aplicaciones para las funciones estándar de [control de entrega de aplicaciones](https://kemptechnologies.com/in/application-delivery-controllers). Para hacerlo, el firewall de aplicaciones web ofrece protección contra las [10 vulnerabilidades web más comunes identificadas por Proyecto de seguridad de aplicación web abierta (OWASP)](https://owasp.org/www-project-top-ten/).

![Diagrama del firewall de aplicaciones web de Application Gateway](./media/threat-detection/azure-threat-detection-fig13.png)

Las protecciones incluyen lo siguiente:

-   Protección contra la inyección de código SQL.

-   Protección contra scripts entre sitios.

-   Protección contra ataques web comunes, como inyección de comandos, contrabando de solicitudes HTTP, división de respuestas HTTP y ataque remoto de inclusión de archivos.

-   Protección contra infracciones del protocolo HTTP.

-   Protección contra anomalías del protocolo HTTP, como la falta de agentes de usuario de host y encabezados de aceptación.

-   Prevención contra bots, rastreadores y escáneres.

-   Detección de errores de configuración comunes en aplicaciones (es decir, Apache, IIS, etc.).

La configuración de WAF en Application Gateway proporciona las siguientes ventajas:

-   Protección de la aplicación web contra las vulnerabilidades y los ataques web sin modificación del código de back-end.

-   Protección de varias aplicaciones web al mismo tiempo detrás de una instancia de Application Gateway. Una instancia de Application Gateway admite el hospedaje de hasta 20 sitios web.

-   Supervisión de las aplicaciones web de cara a los ataques mediante informes en tiempo real generados por los registros WAF de la puerta de enlace de aplicaciones.

-   Ayuda a cumplir los requisitos de cumplimiento. Algunos controles de cumplimiento requieren que todos los puntos de conexión accesibles desde Internet estén protegidos por una solución WAF.

### <a name="anomaly-detection-api-built-with-azure-machine-learning"></a>Anomaly Detection API: integrada con Azure Machine Learning

La API de detección de anomalías es una API útil para detectar una variedad de patrones anómalos en datos de series temporales. La API asigna una puntuación de anomalía a cada punto de datos de la serie temporal, que se puede usar para generar alertas, supervisar a través de paneles o conectar con los sistemas de vales.

La [API de detección de anomalías](../../machine-learning/team-data-science-process/apps-anomaly-detection-api.md) detecta los siguientes tipos de anomalías en datos de series temporales:

-   **Cambios abruptos e interrupciones**: al supervisar el número de errores de inicio de sesión en un servicio o el número de finalizaciones de compras en un sitio de comercio electrónico, los cambios abruptos y las interrupciones inusuales podrían indicar ataques a la seguridad o interrupciones en el servicio.

-   **Tendencias positivas y negativas**: cuando se está supervisando el uso de memoria en informática, la reducción del tamaño de memoria libre indica una posible pérdida de memoria. Para supervisar la longitud de la cola del servicio, una tendencia al alza persistente puede indicar un problema de software subyacente.

-   **Cambios de nivel y cambios en el intervalo dinámico de valores**: puede ser interesante supervisar los cambios de nivel en las latencias de un servicio después de una actualización del servicio o los niveles menores de excepciones después de una actualización.

La API basada en aprendizaje automático permite lo siguiente:

-   **Detección fiable y flexible**: los modelos de detección de anomalías permiten a los usuarios configurar las opciones de confidencialidad y detectar anomalías en conjuntos de datos de temporada y no estacionales. Los usuarios pueden ajustar el modelo de detección de anomalías para que la API de detección tenga más o menos sensibilidad en función de sus necesidades. Esto significaría detectar las anomalías más o menos visibles en los datos con y sin patrones estacionales.

-   **Detección escalable y a tiempo**: la forma tradicional de supervisión con umbrales presentes definidos por el conocimiento de dominios de los expertos es costosa y no escalable a millones de conjuntos de datos que cambian de forma dinámica. Los modelos de detección de anomalías de esta API se aprenden y los modelos se optimizan automáticamente a partir de datos tanto históricos como en tiempo real.

-   **Detección proactiva y factible**: la detección de cambios lentos en las tendencias y los niveles se puede aplicar a la detección de anomalías tempranas. Las señales anómalas tempranas que se detectan sirven para dirigir a las personas para que investiguen y tomen medidas en las zonas problemáticas. Por otra parte, además de este servicio de API de detección de anomalías se pueden desarrollar modelos de análisis de causa principal y herramientas de alerta.

La API de detección de anomalías es una solución eficaz y eficiente para una amplia gama de escenarios, como el estado del servicio y la supervisión de KPI, IoT, la supervisión del rendimiento y la del tráfico de red. Estos son algunos escenarios populares donde esta API puede resultar útil:

- Los departamentos de TI necesitan herramientas de seguimiento de eventos, códigos de error, registros de uso y rendimiento (CPU, memoria, etc.) de manera puntual.

-   Los sitios de comercio en línea desean realizar el seguimiento de las actividades de los clientes, las vistas a la página, los clics y mucho más.

-   Las empresas de servicios públicos desean realizar el seguimiento del consumo de agua, gas, electricidad y otros recursos.

-   A los servicios de administración de instalaciones o edificios les interesa supervisar la temperatura, la humedad, el tráfico y mucho más.

-   Los fabricantes e IoT desean utilizar datos de detección de series temporales para supervisar el flujo de trabajo, la calidad y mucho más.

-   Los proveedores de servicios, como los centros de llamadas, necesitan supervisar las tendencias de demanda del servicio, el volumen de incidentes, la longitud de la cola de espera y mucho más.

-   A los grupos de análisis de negocios les interesa supervisar los movimientos anómalos en los KPI (como el volumen de ventas, las opiniones de los clientes o los precios) en tiempo real.

### <a name="cloud-app-security"></a>Cloud App Security

[Cloud App Security](https://docs.microsoft.com/cloud-app-security/what-is-cloud-app-security) es un componente esencial de la pila de seguridad de Microsoft Cloud. Es una solución integral que ayuda a la organización a medida que avanza para aprovechar al máximo la promesa de las aplicaciones en la nube. Permite mantener el control gracias a la mejor visibilidad de la actividad. También ayuda a aumentar la protección de los datos críticos en todas las aplicaciones de la nube.

Con las herramientas que ayudan a descubrir zonas oscuras de TI, evaluar los riesgos, aplicar directivas, investigar actividades y detener amenazas, la organización puede mover datos a la nube con mayor seguridad y sin perder el control sobre los datos críticos.

| Category | Descripción |
| -------- | ----------- |
| Descubra | Descubra las zonas oscuras de TI con Cloud App Security. Gane visibilidad al descubrir aplicaciones, actividades, usuarios, datos y archivos del entorno de la nube. Descubra las aplicaciones de terceros conectadas a su nube.|
|Investigación | Investigue las aplicaciones de nube mediante herramientas forenses de nube en profundidad en las aplicaciones de riesgo o archivos y usuarios específicos de la red. Identifique patrones en los datos recopilados de la nube. Genere informes para supervisar su nube. |
| Control | Mitigue el riesgo al establecer directivas y alertas para lograr el máximo control sobre el tráfico de red en la nube. Use Cloud App Security para migrar los usuarios a alternativas de aplicaciones de nube seguras y autorizadas. |
| Protección | Use Cloud App Security para autorizar o prohibir aplicaciones, prevenir la pérdida de datos, otorgar permisos de control, compartir y generar alertas e informes personalizados. |
| Control | Mitigue el riesgo al establecer directivas y alertas para lograr el máximo control sobre el tráfico de red en la nube. Use Cloud App Security para migrar los usuarios a alternativas de aplicaciones de nube seguras y autorizadas. |


![Diagrama de Cloud App Security](./media/threat-detection/azure-threat-detection-fig14.png)

Cloud App Security integra visibilidad con la nube mediante lo siguiente:

-   El uso de Cloud Discovery para asignar e identificar el entorno de la nube y las aplicaciones de nube que usa la organización.

-   La prohibición y autorización de aplicaciones en la nube.

-   El uso de conectores de aplicaciones fáciles de implementar que aprovechan las ventajas de las API de proveedor, para ganar visibilidad y gobernanza de las aplicaciones a las que se conecte.

-   La ayuda para mantener el control constante mediante la configuración y posterior ajuste permanente de las directivas.

En la recopilación de datos de estos orígenes, Cloud App Security ejecuta un sofisticado análisis sobre los datos. Le avisa inmediatamente en caso de actividades anómalas y proporciona visibilidad profunda en el entorno de nube. Puede configurar una directiva en Cloud App Security y usarla para proteger todo el contenido del entorno de nube.

## <a name="third-party-advanced-threat-detection-capabilities-through-the-azure-marketplace"></a>Funciones de detección de amenazas avanzada de terceros a través de Azure Marketplace

### <a name="web-application-firewall"></a>Firewall de aplicaciones web

El firewall de aplicaciones web inspecciona el tráfico web entrante y bloquea las inyecciones de SQL, el scripting entre sitios, las cargas de malware, los ataques DDoS de aplicación, así como otros ataques dirigidos a las aplicaciones web. También inspecciona las respuestas de los servidores web back-end para la prevención de pérdida de datos (DLP). El motor de control de acceso integrado permite a los administradores crear directivas de control de acceso pormenorizadas para la autenticación, la autorización y la contabilidad (AAA), lo que proporciona autenticación fiable a las organizaciones así como control sobre los usuarios.

Firewall de aplicaciones web proporciona las ventajas siguientes:

-   Detecta y bloquea las inyecciones de SQL, el scripting entre sitios, las cargas de malware, el DDoS de aplicación y cualquier otro ataque contra la aplicación.

-   Autenticación y control de acceso.

-   Analiza el tráfico saliente para detectar los datos confidenciales y oculta o bloquea la información para que no se filtre.

-   Acelera la entrega de contenido de aplicación web, con funcionalidades como el almacenamiento en caché, la compresión y otras optimizaciones del tráfico.

Para obtener ejemplos de los firewalls de aplicaciones web que están disponibles en Azure Marketplace, vea [Barracuda WAF, firewall de aplicaciones web virtual Brocade (vWAF), Imperva SecureSphere y el firewall ThreatSTOP IP](https://azuremarketplace.microsoft.com/marketplace/apps/barracudanetworks.waf).

## <a name="next-steps"></a>Pasos siguientes

- [Respuesta a las amenazas actuales](../../security-center/security-center-alerts-overview.md#respond-threats): ayuda a identificar las amenazas activas dirigidas a los recursos de Azure y proporciona la información necesaria para responder a estas amenazas con rapidez.

- [Detección de amenazas de Azure SQL Database](https://azure.microsoft.com/blog/azure-sql-database-threat-detection-your-built-in-security-expert/): ayuda a eliminar las preocupaciones sobre las amenazas contra las bases de datos.
