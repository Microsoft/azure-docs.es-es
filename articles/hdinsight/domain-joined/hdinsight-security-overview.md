---
title: Introducción a la seguridad de la empresa en Azure HDInsight
description: Conozca los diferentes métodos para garantizar la seguridad de la empresa en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: overview
ms.custom: seoapr2020
ms.date: 08/24/2020
ms.openlocfilehash: 9cfda93cb7f99851109ab7c4a4590517f785c8a1
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89292986"
---
# <a name="overview-of-enterprise-security-in-azure-hdinsight"></a>Introducción a la seguridad de la empresa en Azure HDInsight

Azure HDInsight ofrece una serie de métodos para abordar las necesidades de seguridad de la empresa. De forma predeterminada, la mayoría de estas soluciones no están activadas. Esta flexibilidad le permite elegir las características de seguridad que considere más importantes y le ayuda a evitar pagar por características que no desea. Esta flexibilidad también significa que es responsabilidad suya asegurarse de que las soluciones correctas estén habilitadas para su configuración y entorno.

En este artículo se analizan las soluciones de seguridad; para ello, se dividen en cuatro pilares de seguridad tradicionales: seguridad perimetral, autenticación, autorización y cifrado.

En este artículo también se presenta **Enterprise Security Package (ESP) de Azure HDInsight** , que proporciona la autenticación basada en Active Directory, compatibilidad multiusuario y control de acceso basado en rol para clústeres de HDInsight.

## <a name="enterprise-security-pillars"></a>Pilares de seguridad de la empresa

Una forma de ver la seguridad empresarial divide las soluciones de seguridad en cuatro grupos principales basados en el tipo de control. Estos grupos también se denominan pilares de seguridad y son los siguientes: seguridad perimetral, autenticación, autorización y cifrado.

### <a name="perimeter-security"></a>Seguridad del perímetro

La seguridad del perímetro en HDInsight se logra a través de [redes virtuales](../hdinsight-plan-virtual-network-deployment.md). Un administrador de empresa puede crear un clúster dentro de una red virtual y usar grupos de seguridad de red (NSG) para restringir el acceso a esta. Solo las direcciones IP permitidas en las reglas del grupo de seguridad de red de entrada pueden comunicarse con el clúster de HDInsight. Esta configuración proporciona la seguridad del perímetro.

Todos los clústeres implementados en una red virtual también tendrán un punto de conexión privado. El punto de conexión se resuelve como una dirección IP privada dentro de la red virtual para el acceso HTTP privado a las puertas de enlace del clúster.

### <a name="authentication"></a>Authentication

[Enterprise Security Package](apache-domain-joined-architecture.md) de HDInsight proporciona autenticación basada en Active Directory, compatibilidad multiusuario y control de acceso basado en rol. La integración con Active Directory se logra mediante el uso de [Azure Active Directory Domain Services](../../active-directory-domain-services/overview.md). Con estas funcionalidades, puede crear un clúster de HDInsight unido a un dominio de Active Directory. Luego, configure una lista de empleados de la empresa que pueden autenticarse e iniciar sesión en el clúster.

Con esta configuración, los empleados de la empresa pueden iniciar sesión en los nodos del clúster con sus credenciales de dominio. También pueden usar sus credenciales de dominio para autenticarse con otros puntos de conexión aprobados, como por ejemplo, las vistas de Apache Ambari, ODBC, JDBC, PowerShell y las API REST a fin de interactuar con el clúster.

### <a name="authorization"></a>Authorization

Un procedimiento recomendado que la mayoría de las empresas siguen es que no todos los empleados tengan acceso completo a todos los recursos de la empresa. Asimismo, el administrador puede definir directivas de control de acceso basado en rol para los recursos del clúster. Esta acción solo está disponible en los clústeres de ESP.

El administrador de Hadoop puede configurar el control de acceso basado en rol (RBAC). Las configuraciones protegen Apache [Hive](apache-domain-joined-run-hive.md), [HBase](apache-domain-joined-run-hbase.md) y [Kafka](apache-domain-joined-run-kafka.md) con complementos de Apache Ranger. La configuración de las directivas RBAC permite asociar permisos a un rol de la organización. Esta capa de abstracción permite garantizar fácilmente que los usuarios solo tengan los permisos necesarios para realizar sus responsabilidades laborales. Ranger también puede auditar el acceso a los datos de los empleados y todos los cambios que se realizan en las directivas de control de acceso.

Por ejemplo, puede configurar el administrador [Apache Ranger](https://ranger.apache.org/) para establecer las directivas de control de acceso para Hive. Esta funcionalidad garantiza el filtrado de nivel de fila y de columna (enmascaramiento de datos). Además, filtra los datos confidenciales para usuarios no autorizados.

### <a name="auditing"></a>Auditoría

Es necesario auditar el acceso a los recursos del clúster para realizar un seguimiento del acceso no autorizado o no intencionado de los recursos. Es tan importante como proteger los recursos del clúster frente al acceso no autorizado.

El administrador puede ver y notificar todos los accesos a los recursos y los datos del clúster de HDInsight. El administrador puede ver y notificar los cambios en las directivas de control de acceso.

Para acceder a los registros de auditoría de Apache Ranger y Ambari, así como a los registros de acceso mediante SSH, [habilite Azure Monitor](../hdinsight-hadoop-oms-log-analytics-tutorial.md#cluster-auditing) y consulte las tablas que proporcionan registros de auditoría.

### <a name="encryption"></a>Cifrado

Proteger los datos es importante para cumplir con los requisitos de cumplimiento y de seguridad de la organización. Además de restringir el acceso de empleados no autorizados a los datos, es necesario cifrarlos.

HDInsight admite el cifrado de datos en reposo tanto con las claves administradas por la plataforma como con [y las administradas por el cliente](../disk-encryption.md). El cifrado de datos en tránsito se controla con TLS e IPSec. Para más información, consulte el artículo sobre el [cifrado en tránsito para Azure HDInsight](encryption-in-transit.md) para obtener más información.

### <a name="compliance"></a>Cumplimiento normativo

Las ofertas de cumplimiento de Azure se basan en diversos tipos de garantías, incluidas las certificaciones formales, pero también las atestaciones, las validaciones y las autorizaciones, las evaluaciones producidas por empresas de auditoría de terceros independientes, las modificaciones contractuales, las autoevaluaciones y los documentos de orientación para los clientes producidos por Microsoft. Para obtener información sobre el cumplimiento de HDInsight, acceda al [Centro de confianza de Microsoft](https://www.microsoft.com/trust-center) y la [Información general sobre el cumplimiento de Microsoft Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942).

## <a name="shared-responsibility-model"></a>Modelo de responsabilidad compartida

En la siguiente imagen se resumen las principales áreas de seguridad del sistema y las soluciones de seguridad que tiene disponibles en cada una de ellas. También se destacan las áreas de seguridad que son responsabilidad suya como cliente y las áreas que son responsabilidad de HDInsight como proveedor de servicios.

![Diagrama de responsabilidades compartidas de HDInsight](./media/hdinsight-security-overview/hdinsight-shared-responsibility.png)

En la tabla siguiente se proporcionan vínculos a recursos para cada tipo de solución de seguridad.

| Área de seguridad | Soluciones disponibles | Parte responsable |
|---|---|---|
| Seguridad de acceso a datos | Configurar las [listas de control de acceso](../../storage/blobs/data-lake-storage-access-control.md) para Azure Data Lake Storage Gen1 y Gen2  | Customer |
|  | Habilitar la propiedad ["Se requiere transferencia segura"](../../storage/common/storage-require-secure-transfer.md) en las cuenta de almacenamiento | Customer |
|  | Configurar las redes virtuales y los [firewalls de Azure Storage](../../storage/common/storage-network-security.md) | Customer |
|  | Configurar los [puntos de conexión de servicio de red virtual de Azure](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview) para Cosmos DB y [Azure SQL DB](https://docs.microsoft.com/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview) | Customer |
|  | Asegúrese de que la característica [Cifrado en tránsito](./encryption-in-transit.md) está habilitada para usar TLS e IPSec para la comunicación interna de los clústeres. | Customer |
|  | Configuración de las [claves que administra el cliente](../../storage/common/storage-encryption-keys-portal.md) para el cifrado de Azure Storage | Customer |
|  | Controlar el acceso a los datos por parte de soporte técnico de Azure mediante la [Caja de seguridad del cliente](https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview) | Customer |
| Seguridad de aplicaciones y middleware | Integración con AAD-DS y [configuración de la autenticación](apache-domain-joined-configure-using-azure-adds.md) | Customer |
|  | Configurar las directivas de [autorización de Apache Ranger](apache-domain-joined-run-hive.md) | Customer |
|  | Usar [registros de Azure Monitor](../hdinsight-hadoop-oms-log-analytics-tutorial.md) | Customer |
| Seguridad del sistema operativo | Crear clústeres con la imagen base segura más reciente | Customer |
|  | Garantizar la [revisión del sistema operativo](../hdinsight-os-patching.md) en intervalos regulares | Customer |
| Seguridad de las redes | Configurar una [red virtual](../hdinsight-plan-virtual-network-deployment.md) |
|  | Configurar las [reglas del grupo de seguridad de red (NSG) de entrada](../control-network-traffic.md) | Customer |
|  | Configurar la [restricción del tráfico de salida](../hdinsight-restrict-outbound-traffic.md) con el firewall | Customer |
| Infraestructura virtualizada | N/D | HDInsight (proveedor de servicios en la nube) |
| Seguridad de la infraestructura física | N/D | HDInsight (proveedor de servicios en la nube) |

## <a name="next-steps"></a>Pasos siguientes

* [Planeamiento de clústeres de HDInsight con Enterprise Security Package](apache-domain-joined-architecture.md)
* [Configuración de clústeres de HDInsight con Enterprise Security Package](apache-domain-joined-configure.md)
* [Administración de clústeres de HDInsight con Enterprise Security Package](apache-domain-joined-manage.md)
