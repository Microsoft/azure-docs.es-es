---
title: Protección contra amenazas en Azure Security Center
description: En este tema se describen los recursos protegidos por las características de protección contra amenazas de Azure Security Center
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 33c45447-3181-4b75-aa8e-c517e76cd50d
ms.service: security-center
ms.topic: conceptual
ms.date: 06/30/2020
ms.author: memildin
ms.openlocfilehash: 69f439e102edc53207e44d63cb29396f64f59e0e
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88272508"
---
# <a name="threat-protection-in-azure-security-center"></a>Protección contra amenazas en Azure Security Center

Cuando Security Center detecta una amenaza en cualquiera de las áreas del entorno, genera una alerta. Estas alertas describen los detalles de los recursos afectados, los pasos de corrección sugeridos y, en algunos casos, una opción para desencadenar una aplicación lógica como respuesta.

La protección contra amenazas de Azure Security Center proporciona defensas completas para el entorno:

* **Protección contra amenazas para los recursos de proceso de Azure**: máquinas Windows, máquinas Linux, Azure App Service y contenedores de Azure

* **Protección contra amenazas para los recursos de datos de Azure**: SQL Database y SQL Data Warehouse, Azure Storage y Azure Cosmos DB

* **Protección contra amenazas para capas de servicios de Azure**: capa de red de Azure, capa de administración de Azure (Azure Resource Manager) (versión preliminar) y Azure Key Vault (versión preliminar)

Tanto si Security Center genera una alerta, como si la recibe de un producto de seguridad diferente, puede exportarla. Para exportar las alertas a Azure Sentinel, a cualquier SIEM de terceros o a cualquier otra herramienta externa, siga las instrucciones de [Exportación de alertas a un SIEM](continuous-export.md). 

> [!NOTE]
> Las alertas de orígenes diferentes pueden tardar un tiempo distinto en aparecer. Por ejemplo, las alertas que requieren un análisis del tráfico de red pueden tardar más en aparecer que las alertas relacionadas con procesos sospechosos que se ejecutan en máquinas virtuales.

> [!TIP]
> Para habilitar las funcionalidades de protección contra amenazas de Security Center, debe aplicar el plan de tarifa estándar a la suscripción que contiene las cargas de trabajo aplicables.
>
> Puede habilitar la protección contra amenazas para **cuentas de Azure Storage** en el nivel de suscripción o de recursos.
> Puede habilitar la protección contra amenazas para **servidores SQL de Azure SQL Database** en el nivel de suscripción o de recursos.
> Puede habilitar la protección contra amenazas para **Azure Database for MariaDB/MySQL/PostgreSQL** solo en el nivel de recursos.



## <a name="threat-protection-for-windows-machines"></a>Protección contra amenazas para máquinas Windows <a name="windows-machines"></a>

Azure Security Center se integra con servicios de Azure para supervisar y proteger las máquinas Windows. Security Center presenta las alertas y las sugerencias de corrección de todos estos servicios en un formato fácil de usar.

* **Advanced Threat Protection (ATP) de Microsoft Defender** <a name="windows-atp"></a>: Security Center amplía sus plataformas de protección de carga de trabajo en la nube mediante la integración con Advanced Threat Protection (ATP) de Microsoft Defender. Esta integración ofrece funcionalidades completas de detección y respuesta (EDR) de puntos de conexión.

    > [!IMPORTANT]
    > El sensor de Microsoft Defender ATP se habilita automáticamente en los servidores Windows que usan Security Center.

    Cuando Microsoft Defender ATP detecta una amenaza, desencadena una alerta. La alerta se muestra en el panel de Security Center. En el panel, puede dinamizar hasta la consola de Microsoft Defender ATP para realizar una investigación detallada y descubrir el alcance del ataque. Para más información sobre Microsoft Defender ATP, consulte [Incorporación de servidores al servicio Microsoft Defender ATP](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/configure-server-endpoints).

* **Detección de ataques sin archivos** <a name="windows-fileless"></a>: Los ataques sin archivos inyectan cargas malintencionadas en la memoria para evitar la detección mediante técnicas de detección basadas en disco. Luego, la carga del atacante se conserva dentro de la memoria de los procesos en peligro y realiza una amplia variedad de actividades malintencionadas.

    Con la detección de ataques sin archivos, las técnicas forense de memoria automatizadas identifican kits de herramientas, técnicas y comportamientos de los ataques sin archivos. Esta solución examina periódicamente la máquina en tiempo de ejecución y extrae conclusiones directamente de la memoria de los procesos. Las conclusiones específicas para Linux incluyen la identificación de: 

    - Kits de herramientas conocidas y software de minería de datos de cifrado. 
    - Shellcode, que es un pequeño fragmento de código que se usa normalmente como carga útil para aprovechar una vulnerabilidad de software.
    - Archivo ejecutable malintencionado insertado en la memoria de proceso.

    La detección de ataques sin archivos genera alertas de seguridad detalladas que contienen las descripciones con metadatos de proceso adicionales, como la actividad de red. Esto acelera la evaluación de prioridades de alertas, la correlación y el tiempo de respuesta descendente. Este enfoque complementa a las soluciones EDR basadas en eventos y proporciona mayor cobertura de detección.

    Para obtener detalles de las alertas de detección de ataques sin archivo, consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-windows).

> [!TIP]
> Para simular alertas de Windows, descargue [Azure Security Center Playbook: Alertas de seguridad](https://gallery.technet.microsoft.com/Azure-Security-Center-f621a046).






## <a name="threat-protection-for-linux-machines"></a>Protección contra amenazas para máquinas Linux <a name="linux-machines"></a>

Security Center recopila registros de auditoría de máquinas Linux mediante **auditd**, uno de los marcos de trabajo de Linux más comunes. auditd se encuentra en el kernel de línea principal. 

* **Integración de las alertas de auditd de Linux y el agente de Log Analytics** <a name="linux-auditd"></a>: el sistema de auditd consta de un subsistema de nivel de kernel, que es responsable de supervisar las llamadas del sistema. Las filtra según un conjunto de reglas especificado y escribe mensajes para ellas en un socket. Security Center integra funcionalidades del paquete auditd dentro del agente de Log Analytics. Esta integración permite una colección de eventos de auditd en todas las distribuciones de Linux admitidas sin requisitos previos.

    Los registros de auditd se recopilan, enriquecen y agregan en eventos mediante el agente de Log Analytics para Linux. Security Center agrega continuamente análisis nuevos que usan señales de Linux para detectar comportamientos malintencionados en máquinas Linux locales y en la nube. De manera similar a las funcionalidades de Windows, estos análisis abarcan varios procesos sospechosos, intentos de inicio de sesión dudosos, carga de módulos de kernel y otras actividades. Estas actividades pueden indicar que una máquina está sufriendo un ataque o se ha vulnerado.  

    Para obtener una lista de las alertas de Linux, consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-linux).

> [!TIP]
> Para simular alertas de Linux descargue [Guía de Azure Security Center: Detecciones de Linux](https://gallery.technet.microsoft.com/Azure-Security-Center-0ac8a5ef).





## <a name="threat-protection-for-azure-app-service"></a>Protección contra amenazas para Azure App Service <a name="app-services"></a>

> [!NOTE]
> Este servicio no está disponible actualmente en Azure Government ni en las regiones de nubes soberanas.

Security Center usa la escala de la nube para identificar ataques dirigidos a aplicaciones que se ejecutan mediante App Service. Los atacantes sondean las aplicaciones web para buscar y aprovechar los puntos débiles. Antes de enrutarse a entornos específicos, las solicitudes para las aplicaciones que se ejecutan en Azure atraviesan varias puertas de enlace donde se las inspecciona y registra. A continuación, estos datos se usan para identificar vulnerabilidades y atacantes, así como para aprender nuevos patrones que se usarán más adelante.

Al usar la visibilidad que Azure tiene como proveedor de nube, Security Center analiza los registros internos de App Service para metodología de ataques en múltiples destinos, como, por ejemplo, la metodología incluye el análisis generalizado y los ataques distribuidos. Este tipo de ataque normalmente procede de un pequeño subconjunto de direcciones IP y muestra patrones de rastreo a puntos de conexión similares en varios hosts. Los ataques buscan una página o complemento vulnerables y no se pueden identificar desde el punto de vista de un solo host.

Si está usando un plan de App Service basado en Windows, Security Center también tiene acceso a las máquinas virtuales y los espacios aislados subyacentes. Junto con los datos de registro mencionados anteriormente, la infraestructura puede contar lo que ocurre, desde un nuevo ataque que circula por la red hasta riesgos concretos para las máquinas de los clientes. Por lo tanto, incluso si se implementa Security Center una vez que se han aprovechado las vulnerabilidades de una aplicación web, es posible que pueda detectar los ataques en curso.

Para obtener una lista de las alertas de Azure App Service, consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-azureappserv).

Para más información sobre los planes de App Service, consulte [Planes de App Service](https://azure.microsoft.com/pricing/details/app-service/plans/).





## <a name="threat-protection-for-containers"></a>Protección contra amenazas para contenedores <a name="azure-containers"></a>

### <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponibilidad general|
|Precios:|Nivel Standard|
|Roles y permisos necesarios:|**Administración de seguridad** puede descartar las alertas.<br>**El lector de seguridad** puede ver los resultados.|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||

[!INCLUDE [AKS in ASC threat protection](../../includes/security-center-azure-kubernetes-threat-protection.md)]








## <a name="threat-protection-for-sql-database-and-sql-data-warehouse"></a>Protección contra amenazas para SQL Database y SQL Data Warehouse <a name="data-sql"></a>

Advanced Threat Protection para Azure SQL Database detecta actividades anómalas que indican intentos inusuales y potencialmente dañinos de acceso o ataque a las bases de datos.

Verá alertas cuando haya actividades sospechosas en las bases de datos, posibles vulnerabilidades o ataques por inyección de código SQL, y patrones anómalos de acceso y consulta a las bases de datos.

Advanced Threat Protection para Azure SQL Database y SQL forma parte del paquete unificado [Advanced Data Security (ADS)](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security) para funcionalidades de seguridad SQL avanzadas, que abarca instancias de Azure SQL Database, Azure SQL Managed Instance, bases de datos de Azure SQL Data Warehouse y servidores SQL Server en Azure Virtual Machines.

Para más información, consulte:

* [Cómo habilitar Advanced Threat Protection para Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview)
* [Cómo habilitar Advanced Threat Protection para servidores SQL Server en Azure Virtual Machines](security-center-iaas-advanced-data.md)
* [La lista de alertas de protección contra amenazas para SQL Database y SQL Data Warehouse](alerts-reference.md#alerts-sql-db-and-warehouse)



## <a name="threat-protection-for-azure-storage"></a>Protección contra amenazas para Azure Storage <a name="azure-storage"></a>

### <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|[Blob Storage](https://azure.microsoft.com/services/storage/blobs/) (disponibilidad general)<br>[Azure Files](https://docs.microsoft.com/azure/storage/files/storage-files-introduction) (versión preliminar)<br>[Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-introduction) (versión preliminar)|
|Precios:|Nivel Standard|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) US Gov<br>![No](./media/icons/no-icon.png) China Gov, otros gobiernos|
|||


### <a name="whats-protected"></a>¿Qué se protege?

La protección contra amenazas para Azure Storage detecta actividades potencialmente dañinas en las cuentas de Azure Storage. Los datos se pueden proteger tanto si se almacenan como contenedores de blobs, recursos compartidos de archivos o lagos de datos.

Esta capa de protección le permite afrontar las amenazas *sin* necesidad de ser un experto en seguridad y le ayuda a administrar sus sistemas de supervisión de la seguridad.

Sus cuentas de almacenamiento están protegidas 

### <a name="what-kind-of-alerts-does-threat-protection-for-azure-storage-provide"></a>¿Qué tipo de alertas proporciona la protección contra amenazas de Azure Storage?

Las alertas de seguridad se desencadenan en los escenarios siguientes:

- **Actividad sospechosa**: por ejemplo, se ha accedido correctamente a la cuenta correctamente desde una dirección IP conocida como nodo de salida activo de Tor.
- **Comportamiento anómalo**: por ejemplo, se producen cambios en el patrón de acceso a una cuenta de almacenamiento.
- **Malware potencial cargado**: el análisis de reputación de hash indica que un archivo cargado contiene malware.

Las alertas incluyen detalles acerca del incidente que las desencadenó, así como recomendaciones acerca de cómo investigar y corregir las amenazas.

### <a name="what-is-hash-reputation-analysis-for-malware"></a>¿Qué es el análisis de reputación de hash para malware?

Para determinar si un archivo cargado es sospechoso, la protección contra amenazas de Azure Storage usa el análisis de reputación de hash compatible con [Inteligencia sobre amenazas de Microsoft](https://go.microsoft.com/fwlink/?linkid=2128684). Las herramientas de protección contra amenazas no examinan los archivos cargados, sino que examinan los registros de almacenamiento y comparan los valores hash de los archivos recién cargados con los de virus conocidos, troyanos, spyware y ransomware. 

Cuando se sospecha que un archivo contiene malware, Security Center muestra una alerta y, opcionalmente, puede enviar un correo electrónico al propietario del almacenamiento para que elimine el archivo sospechoso. Para configurar esta eliminación automática de archivos que el análisis de reputación de hash indica que contienen malware, implemente una [automatización del flujo de trabajo para desencadenar alertas que contengan el "malware potencial cargado en una cuenta de almacenamiento"](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-respond-to-potential-malware-uploaded-to-azure-storage/ba-p/1452005).



### <a name="next-steps"></a>Pasos siguientes 

Para más información, incluida una evaluación gratuita de 30 días, consulte la [página de precios de Azure Security Center](https://azure.microsoft.com/pricing/details/security-center/).

Para más información, consulte:

* [Cómo habilitar Advanced Threat Protection para Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-advanced-threat-protection)
* [La lista de alertas de protección contra amenazas para Azure Storage](alerts-reference.md#alerts-azurestorage)
* [Funcionalidades de inteligencia sobre amenazas de Microsoft](https://go.microsoft.com/fwlink/?linkid=2128684)

> [!TIP]
> Para simular alertas de almacenamiento, siga las instrucciones de [esta entrada de blog](https://techcommunity.microsoft.com/t5/azure-security-center/validating-atp-for-azure-storage-detections-in-azure-security/ba-p/1068131).







## <a name="threat-protection-for-azure-cosmos-db"></a>Protección contra amenazas para Azure Cosmos DB <a name="cosmos-db"></a>

Las alertas de Azure Cosmos DB las generan los intentos inusuales y potencialmente dañinos de acceso o aprovechamiento de cuentas de Azure Cosmos DB.

Para más información, consulte:

* [Advanced Threat Protection para Azure Cosmos DB (versión preliminar)](../cosmos-db/cosmos-db-advanced-threat-protection.md)
* [La lista de alertas de protección contra amenazas para Azure Cosmos DB (versión preliminar)](alerts-reference.md#alerts-azurecosmos)




## <a name="threat-protection-for-azure-network-layer"></a>Protección contra amenazas para la capa de red de Azure <a name="network-layer"></a>

El análisis de la capa de red de Security Center se basa en [datos IPFIX](https://en.wikipedia.org/wiki/IP_Flow_Information_Export) de ejemplo, que son encabezados de paquete recopilados por los enrutadores principales de Azure. En función de esta fuente de distribución de datos, Security Center utiliza modelos de Machine Learning para identificar y marcar actividades de tráfico malintencionado. Security Center también utiliza la base de datos de Microsoft Threat Intelligence para enriquecer las direcciones IP.

Algunas configuraciones de red pueden restringir la Security Center de la generación de alertas en una actividad de red sospechosa. Para que Security Center genere alertas de red, asegúrese de que:

- La máquina virtual tenga una dirección IP pública (o se encuentre en un equilibrador de carga con una dirección IP pública).

- El tráfico de salida de red de la máquina virtual no esté bloqueado por una solución de IDS externa.

- A la máquina virtual se le haya asignado la misma dirección IP durante toda la hora en la que se produjo la comunicación sospechosa. Esto también se aplica a las máquinas virtuales creadas como parte de un servicio administrado (por ejemplo, AKS, Databricks).

Para obtener una lista de las alertas de nivel de red de Azure, consulte la [tabla de referencia de alertas](alerts-reference.md#alerts-azurenetlayer).




## <a name="threat-protection-for-azure-management-layer-azure-resource-manager-preview"></a>Protección contra amenazas para la capa de administración de Azure (Azure Resource Manager) (versión preliminar)<a name ="management-layer"></a>

La capa de protección de Security Center basada en Azure Resource Manager está actualmente en versión preliminar.

Security Center ofrece una capa adicional de protección al usar eventos de Azure Resource Manager, lo que se considera el plano de control de Azure. Al analizar los registros de Azure Resource Manager, Security Center detecta operaciones inusuales o potencialmente peligrosas en el entorno de suscripción de Azure.

Para obtener una lista de las alertas de Azure Resource Manager (versión preliminar), consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-azureresourceman).



>[!NOTE]
> Varios de los análisis anteriores se realizan mediante Microsoft Cloud App Security. Para beneficiarse de estos análisis, debe activar una licencia de Cloud App Security. Si tiene una licencia de Cloud App Security, estas alertas están habilitadas de forma predeterminada. Para deshabilitar las alertas:
>
> 1. En el menú de Security Center, seleccione **Precios y configuración**.
> 1. Seleccione la suscripción que desea cambiar.
> 1. Seleccione **Detección de amenazas**.
> 1. Desactive la opción **Permita que Microsoft Cloud App Security acceda a sus datos** y seleccione **Guardar**.

>[!NOTE]
>Security Center almacena datos de clientes relacionados con la seguridad en la misma zona geográfica que su recurso. Si Microsoft aún no ha implementado Security Center en la zona geográfica del recurso, almacenará los datos en Estados Unidos. Cuando Cloud App Security esté habilitado, esta información se almacenará con arreglo a las reglas de ubicación geográfica de Cloud App Security. Para obtener más información, consulte [Almacenamiento de datos para servicios no regionales](https://azuredatacentermap.azurewebsites.net/).

1. Establezca el área de trabajo en la que va a instalar el agente. Asegúrese de que el área de trabajo está en la misma suscripción que se usa en Security Center y que tiene permisos de lectura/escritura en el área de trabajo.

1. Establezca el plan de tarifa estándar y seleccione **Guardar**.






## <a name="threat-protection-for-azure-key-vault-preview"></a>Protección contra amenazas para Azure Key Vault (versión preliminar)<a name="azure-keyvault"></a>

> [!NOTE]
> Este servicio no está disponible actualmente en Azure Government ni en las regiones de nubes soberanas.

Azure Key Vault es un servicio en la nube que protege las claves de cifrado y los secretos, como certificados, cadenas de conexión y contraseñas. 

En Azure Security Center se incluye protección frente a amenazas avanzada, nativa de Azure, para Azure Key Vault que proporciona una capa adicional de inteligencia de seguridad. Security Center detecta intentos inusuales y potencialmente perjudiciales de acceder o explotar las cuentas de Key Vault. Esta capa de protección le permite afrontar las amenazas sin ser un experto en seguridad y sin la necesidad de administrar sistemas de supervisión de la seguridad de terceros.  

Cuando se producen actividades anómalas, Security Center muestra alertas y, opcionalmente, las envía por correo electrónico a los administradores de suscripciones. Dichas alertas incluyen detalles acerca de cualquier actividad sospechosa y recomendaciones acerca de cómo investigar amenazas y mitigarlas. 

Para obtener una lista de las alertas de Azure Key Vault, consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-azurekv).





## <a name="threat-protection-for-other-microsoft-services"></a>Protección contra amenazas para otros servicios de Microsoft <a name="alerts-other"></a>

### <a name="threat-protection-for-azure-waf"></a>Protección contra amenazas para el firewall de aplicaciones web de Azure <a name="azure-waf"></a>

Firewall de aplicaciones web (WAF) es una característica de Azure Application Gateway que proporciona a las aplicaciones web una protección centralizada contra vulnerabilidades de seguridad comunes.

Las aplicaciones web son cada vez más el objetivo de ataques malintencionados que aprovechan vulnerabilidades habitualmente conocidas. El firewall de aplicaciones web de Application Gateway se basa en el conjunto de reglas básicas 3.0 o 2.2.9 de Open Web Application Security Project. El firewall de aplicaciones web se actualiza automáticamente para incluir protección frente a nuevas vulnerabilidades. 

Si tiene una licencia para el firewall de aplicaciones web de Azure, las alertas de este se transmitirán a Security Center sin necesidad de configuración adicional. Para más información sobre las alertas generadas por WAF, consulte [Reglas y grupos de reglas de CRS del firewall de aplicaciones web](../web-application-firewall/ag/application-gateway-crs-rulegroups-rules.md?tabs=owasp31#crs911-31).


### <a name="threat-protection-for-azure-ddos-protection"></a>Protección contra amenazas para Azure DDoS Protection <a name="azure-ddos"></a>

Los ataques de denegación de servicio distribuido (DDoS) son conocidos por lo fáciles que son de ejecutar. Se han convertido en un problema de seguridad muy importante, especialmente si va a trasladar sus aplicaciones a la nube. 

Un ataque DDoS intenta agotar los recursos de una aplicación haciendo que esta no esté disponible para los usuarios legítimos. Los ataques DDoS pueden dirigirse a cualquier punto de conexión accesible a través de Internet.

Para defenderse contra ataques DDoS, adquiera una licencia de Azure DDoS Protection y asegúrese de que está siguiendo los procedimientos recomendados de diseño de aplicaciones. DDoS Protection proporciona distintos niveles de servicio. Para más información, consulte [Introducción a Azure DDoS Protection](https://docs.microsoft.com/azure/virtual-network/ddos-protection-overview).

Para obtener una lista de las alertas de Azure DDoS Protection, consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-azureddos).


## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de las alertas de seguridad de estas características de protección contra amenazas, consulte los siguientes artículos:

* [Tabla de referencia para todas las alertas de Azure Security Center](alerts-reference.md)
* [Alertas de seguridad en el Centro de seguridad de Azure](security-center-alerts-overview.md)
* [Administración y respuesta a alertas de seguridad en Azure Security Center](security-center-managing-and-responding-alerts.md)
* [Exportar alertas y recomendaciones de seguridad (versión preliminar)](continuous-export.md)