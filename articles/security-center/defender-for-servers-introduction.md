---
title: 'Azure Defender para servidores: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para servidores.
author: memildin
ms.author: memildin
ms.date: 9/12/2020
ms.topic: conceptual
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 29f15aa7d1cbcd260ce44f6083138681ceb9fd79
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932445"
---
# <a name="introduction-to-azure-defender-for-servers"></a>Introducción a Azure Defender para servidores

Azure Defender para servidores agrega la detección de amenazas y defensas avanzadas para las máquinas Windows y Linux.

Para Windows, Azure Defender se integra con servicios de Azure para supervisar y proteger las máquinas Windows. Security Center presenta las alertas y las sugerencias de corrección de todos estos servicios en un formato fácil de usar.

Para Linux, Azure Defender recopila registros de auditoría de máquinas Linux mediante **auditd**, uno de los marcos de trabajo de Linux más comunes. auditd se encuentra en el kernel de línea principal. 


## <a name="what-are-the-benefits-of-azure-defender-for-servers"></a>¿Cuáles son las ventajas de Azure Defender para servidores?

Las funcionalidades de detección de amenazas y protección que se proporcionan con Azure Defender para servidores incluyen:

- **Análisis de la evaluación de vulnerabilidades en máquinas virtuales**: el detector de vulnerabilidades incluido con Azure Security Center cuenta con la tecnología de Qualys. 

    El detector de Qualys es una de las herramientas líderes para identificar en tiempo real las vulnerabilidades en Azure Virtual Machines. No se necesita ninguna licencia ni cuenta de Qualys, ya que todo se administra sin problemas en Security Center. [Más información](deploy-vulnerability-assessment-vm.md).

- **Acceso Just-In-Time (JIT) a máquinas virtuales**: los agentes de amenazas buscan activamente máquinas accesibles con puertos de administración abiertos, como RDP o SSH. Todas las máquinas virtuales son objetivos potenciales para un ataque. Cuando se consigue poner en peligro a una máquina virtual, se usa como punto de entrada para atacar más recursos dentro de su entorno.

    Una vez habilitado Azure Defender para servidores, puede usar el acceso Just-In-Time a máquinas virtuales ara bloquear el tráfico entrante a las máquinas virtuales, lo que reduce la exposición a ataques al mismo tiempo que proporciona un acceso sencillo para conectarse a las máquinas virtuales cuando sea necesario. [Más información](just-in-time-explained.md).

- **Supervisión de la integridad de los archivos (FIM)** : la supervisión de la integridad de los archivos (FIM), conocida también como supervisión de los cambios, examina los archivos y los registros del sistema operativo, el software de aplicación y demás para comprobar la existencia de cambios que podrían indicar un ataque. Para determinar si el estado actual del archivo es diferente del último examen del archivo, se usa un método de comparación. Puede aprovechar esta comparación para determinar si se han realizado modificaciones sospechosas o válidas en los archivos.

    Una vez habilitado Azure Defender para servidores, puede usar FIM para validar la integridad de los archivos de Windows, los registros de Windows y los archivos de Linux. [Más información](security-center-file-integrity-monitoring.md).

- **Controles de aplicaciones adaptables (ACC)** : los controles de aplicaciones adaptables son una solución inteligente y automatizada que permite definir listas de aplicaciones permitidas seguras conocidas para las máquinas.

    Cuando haya habilitado y configurado controles de aplicaciones adaptables, recibirá alertas de seguridad si alguna aplicación ejecuta otros distintos a los definidos como seguros. [Más información](security-center-adaptive-network-hardening.md).

- **Protección de red adaptable (ANH)** : la aplicación de grupos de seguridad de red (NSG) para filtrar el tráfico hacia y desde los recursos mejora la posición de seguridad de red. Sin embargo, aún puede haber algunos casos en los que el tráfico real que fluye a través del NSG es un subconjunto de las reglas de NSG definidas. En estos casos, puede mejorar la postura de seguridad al proteger aún más las reglas de NSG, según los patrones de tráfico real.

    La protección de red adaptable proporciona recomendaciones para proteger mejor las reglas de NSG. Usa un algoritmo de aprendizaje automático que tiene en cuenta el tráfico real, la configuración de confianza conocida, la inteligencia de amenazas y otros indicadores de riesgo, y luego proporciona recomendaciones para permitir el tráfico solo desde tuplas IP y puerto específicas. [Más información](security-center-adaptive-network-hardening.md).

- **Integración con la Protección de amenazas avanzada de Microsoft Defender (ATP) (solo Windows)** : Azure Defender se integra con la Protección contra amenazas avanzada de Microsoft Defender (ATP). Esta integración ofrece funcionalidades completas de detección y respuesta (EDR) de puntos de conexión. [Más información](security-center-wdatp.md).

    > [!IMPORTANT]
    > El sensor de Microsoft Defender ATP se habilita automáticamente en los servidores Windows que usan Security Center.

    Cuando Microsoft Defender ATP detecta una amenaza, desencadena una alerta. La alerta se muestra en Security Center. En Security Center, también puede dinamizar hasta la consola de Microsoft Defender ATP para realizar una investigación detallada y descubrir el alcance del ataque. Para más información sobre Microsoft Defender ATP, consulte [Incorporación de servidores al servicio Microsoft Defender ATP](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/configure-server-endpoints).

- **Protección de host de Docker**: Azure Security Center identifica contenedores no administrados y que están hospedados en VM de IaaS Linux u otras máquinas Linux que ejecutan contenedores de Docker. Security Center evalúa continuamente las configuraciones de estos contenedores. A continuación, las compara con el Banco de prueba para Docker del Centro de seguridad de Internet (CIS). Security Center incluye todo el conjunto de reglas del banco de prueba de Docker de CIS y le avisa si los contenedores no cumplen ninguno de los controles. [Más información](harden-docker-hosts.md).

- **Detección de ataques sin archivos (solo Windows)** : los ataques sin archivos inyectan cargas malintencionadas en la memoria para evitar la detección mediante técnicas de detección basadas en disco. Luego, la carga del atacante se conserva dentro de la memoria de los procesos en peligro y realiza una amplia variedad de actividades malintencionadas.

  Con la detección de ataques sin archivos, las técnicas forense de memoria automatizadas identifican kits de herramientas, técnicas y comportamientos de los ataques sin archivos. Esta solución examina periódicamente la máquina en tiempo de ejecución y extrae conclusiones directamente de la memoria de los procesos. Las conclusiones específicas para Linux incluyen la identificación de: 

  - Kits de herramientas conocidas y software de minería de datos de cifrado. 

  - Shellcode, que es un pequeño fragmento de código que se usa normalmente como carga útil para aprovechar una vulnerabilidad de software.

  - Archivo ejecutable malintencionado insertado en la memoria de proceso.

  La detección de ataques sin archivos genera alertas de seguridad detalladas que contienen las descripciones con metadatos de proceso adicionales, como la actividad de red. Esto acelera la evaluación de prioridades de alertas, la correlación y el tiempo de respuesta descendente. Este enfoque complementa a las soluciones EDR basadas en eventos y proporciona mayor cobertura de detección.

  Para obtener detalles de las alertas de detección de ataques sin archivo, consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-windows).

- **Integración de las alertas de auditd de Linux y el agente de Log Analytics (solo Linux)** : el sistema de auditd consta de un subsistema de nivel de kernel, que es responsable de supervisar las llamadas del sistema. Las filtra según un conjunto de reglas especificado y escribe mensajes para ellas en un socket. Security Center integra funcionalidades del paquete auditd dentro del agente de Log Analytics. Esta integración permite una colección de eventos de auditd en todas las distribuciones de Linux admitidas sin requisitos previos.

    Los registros de auditd se recopilan, enriquecen y agregan en eventos mediante el agente de Log Analytics para Linux. Security Center agrega continuamente análisis nuevos que usan señales de Linux para detectar comportamientos malintencionados en máquinas Linux locales y en la nube. De manera similar a las funcionalidades de Windows, estos análisis abarcan varios procesos sospechosos, intentos de inicio de sesión dudosos, carga de módulos de kernel y otras actividades. Estas actividades pueden indicar que una máquina está sufriendo un ataque o se ha vulnerado.  

    Para obtener una lista de las alertas de Linux, consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-linux).


## <a name="simulating-alerts"></a>Simulación de alertas

Puede simular alertas mediante la descarga de alguno de los cuadernos de estrategias:

- Para Windows: [Cuaderno de estrategias de Azure Security Center: alertas de seguridad](https://gallery.technet.microsoft.com/Azure-Security-Center-f621a046)

- Para Linux: [Cuaderno de estrategias de Azure Security Center: Detecciones de Linux](https://gallery.technet.microsoft.com/Azure-Security-Center-0ac8a5ef).




## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre Azure Defender para servidores. 

Para obtener material relacionado, consulte los siguientes artículos: 

- Tanto si Security Center genera una alerta, como si la recibe de un producto de seguridad diferente, puede exportarla. Para exportar las alertas a Azure Sentinel, a cualquier SIEM de terceros o a cualquier otra herramienta externa, siga las instrucciones de [Exportación de alertas a un SIEM](continuous-export.md).

- > [!div class="nextstepaction"]
    > [Habilitación de Azure Defender](security-center-pricing.md)