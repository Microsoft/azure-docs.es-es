---
title: 'Seguridad: Migración de Apache Hadoop local a Azure HDInsight'
description: Obtenga información acerca de las prácticas recomendadas de DevOps y seguridad para migrar clústeres locales de Apache Hadoop a Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: ashishth
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/19/2019
ms.openlocfilehash: 99c5ff39820b652a33ab670e63ca709f8747c4cf
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88751278"
---
# <a name="migrate-on-premises-apache-hadoop-clusters-to-azure-hdinsight---security-and-devops-best-practices"></a>Prácticas recomendadas de DevOps y seguridad para migrar clústeres locales de Apache Hadoop a Azure HDInsight

En este artículo se proporcionan recomendaciones de DevOps y seguridad en sistemas de Azure HDInsight. Forma parte de una serie de artículos que proporcionan prácticas recomendadas para ayudar a migrar sistemas locales de Apache Hadoop a Azure HDInsight.

## <a name="secure-and-govern-cluster-with-enterprise-security-package"></a>Protección y control del clúster con Enterprise Security Package

Enterprise Security Package (ESP) admite la autenticación basada en Active Directory multiusuario y control de acceso basado en roles. Con la opción de ESP elegida, el clúster de HDInsight se une al dominio de Active Directory y el administrador de empresa puede configurar el control de acceso basado en roles (RBAC) para la seguridad de Apache Hive mediante el uso de Apache Ranger. El administrador también puede auditar el acceso a los datos de los empleados y todos los cambios que se realizan en las directivas de control de acceso.

ESP está disponible en los siguientes tipos de clúster: Apache Hadoop, Apache Spark, Apache HBase, Apache Kafka e Interactive Query (Hive LLAP).

Para implementar el clúster de HDInsight unido a un dominio, siga estos pasos:

- Implemente Azure Active Directory (AAD) pasando el nombre de dominio.
- Implemente Azure Active Directory Domain Services (AAD DS).
- Cree la red virtual y la subred necesarias.
- Implemente una máquina virtual en la red virtual para administrar AAD DS.
- Una la máquina virtual al dominio.
- Instale las herramientas de AD y DNS.
- Solicite que el administrador de AAD DS cree una unidad organizativa (UO).
- Habilite LDAPS para AAD DS.
- Cree una cuenta de servicio en Azure Active Directory con permisos de administrador de lectura y escritura delegados a la unidad organizativa. Esta cuenta de servicio podrá unir las máquinas al dominio y colocar las entidades de seguridad de la máquina en la unidad organizativa. También puede crear entidades de servicio dentro de la unidad organizativa que especifique durante la creación del clúster.

    > [!Note]
    > No es necesario que la cuenta de servicio sea una cuenta de administrador de dominio de AD.

- Implemente el clúster de HDInsight ESP estableciendo los parámetros siguientes:

    |Parámetro |Descripción |
    |---|---|
    |Nombre de dominio|nombre de dominio asociado con Azure AD DS.|
    |Nombre de usuario de dominio|cuenta de servicio del dominio administrado del controlador de dominio de Azure AD DS que creó en la sección anterior, por ejemplo: `hdiadmin@contoso.onmicrosoft.com`. Este usuario de dominio será el administrador de este clúster de HDInsight.|
    |Contraseña de dominio|contraseña de la cuenta de servicio.|
    |Unidad organizativa|nombre distintivo de la unidad organizativa que quiere utilizar con el clúster de HDInsight; por ejemplo, `OU=HDInsightOU,DC=contoso,DC=onmicrosoft,DC=com`. Si no existe esta unidad organizativa, el clúster de HDInsight intenta crearla mediante los privilegios que tiene la cuenta de servicio.|
    |Dirección URL de LDAPS|Por ejemplo: `ldaps://contoso.onmicrosoft.com:636`.|
    |Grupo de usuarios de acceso|grupos de seguridad cuyos usuarios quiere que se sincronicen con el clúster; por ejemplo, `HiveUsers`. Si desea especificar varios grupos de usuarios, sepárelos con un punto y coma ";". Los grupos deben existir en el directorio antes de crear el clúster de ESP.|

Para más información, consulte los siguientes artículos.

- [Introducción a la seguridad de Apache Hadoop con clústeres de HDInsight unidos a un dominio](../domain-joined/hdinsight-security-overview.md)
- [Planeamiento de clústeres de Apache Hadoop unidos a un dominio de Azure en HDInsight](../domain-joined/apache-domain-joined-architecture.md)
- [Configuración de un clúster de HDInsight unido a un dominio mediante Azure Active Directory Domain Services](../domain-joined/apache-domain-joined-configure-using-azure-adds.md)
- [Sincronización de los usuarios de Azure Active Directory en un clúster de HDInsight](../hdinsight-sync-aad-users-to-cluster.md)
- [Configuración de directivas de Apache Hive en HDInsight unido a un dominio](../domain-joined/apache-domain-joined-run-hive.md)
- [Ejecución de Apache Oozie en clústeres de HDInsight Hadoop unidos a un dominio](../domain-joined/hdinsight-use-oozie-domain-joined-clusters.md)

## <a name="implement-end-to-end-enterprise-security"></a>Implementación de la seguridad empresarial de un extremo a otro

La seguridad empresarial de extremo a extremo puede lograrse con los controles siguientes:

**Canalización de datos privados y protegidos (seguridad a nivel de perímetro)**
    - La seguridad a nivel de perímetro se puede lograr a través de redes virtuales de Azure, grupos de seguridad de red y el servicio de puerta de enlace

**Autenticación y autorización para acceso a datos**
    - Cree clústeres de HDInsight unidos a un dominio con Azure Active Directory Domain Services. (Enterprise Security Package).
    - Use Ambari para proporcionar acceso basado en roles a los recursos de clúster para los usuarios de AD.
    - Use Apache Ranger para establecer las directivas de control de acceso para Hive a nivel de tabla, columna y fila.
    - El acceso SSH al clúster puede estar restringido a solo el administrador.

**Auditoría**
    - Vea y notifique todos los accesos a los recursos y los datos del clúster de HDInsight.
    - Vea y notifique todos los cambios en las directivas de control de acceso.

**Cifrado**
    - Cifrado del lado servidor transparente con claves administradas por Microsoft o claves administradas por el cliente.
    - Cifrado en tránsito mediante el cifrado en el cliente, HTTPS y TLS.

Para más información, consulte los siguientes artículos.

- [Información general de Azure Virtual Network](../../virtual-network/virtual-networks-overview.md)
- [Información general de Grupos de seguridad de red de Azure](../../virtual-network/security-overview.md)
- [Emparejamiento de Azure Virtual Network](../../virtual-network/virtual-network-peering-overview.md)
- [Guía de seguridad de Azure Storage](../../storage/blobs/security-recommendations.md)
- [Cifrado en reposo del servicio de Azure Storage](../../storage/common/storage-service-encryption.md)

## <a name="use-monitoring--alerting"></a>Uso de supervisión y alertas

Para obtener más información, consulte el artículo:

[Información general de Azure Monitor](../../azure-monitor/overview.md)

## <a name="upgrade-clusters"></a>Actualización de clústeres

Actualice regularmente a la última versión de HDInsight para aprovechar las últimas características. Los siguientes pasos pueden utilizarse para actualizar el clúster a la versión más reciente:

1. Cree un nuevo clúster de HDInsight de prueba mediante la última versión de HDInsight.
1. Realice pruebas en el nuevo clúster para asegurarse de que los trabajos y las cargas de trabajo funcionan según lo previsto.
1. Modifique los trabajos, las aplicaciones o las cargas de trabajo según sea necesario.
1. Haga copia de seguridad de todos los datos transitorios almacenados localmente en los nodos del clúster.
1. Elimine el clúster existente.
1. Cree un clúster en la versión más reciente de HDInsight en la misma subred de red virtual, con los mismos datos y almacén predeterminados que usaba el clúster anterior.
1. Importe los datos transitorios cuya copia de seguridad realizó.
1. Inicie trabajos o continúe el procesamiento con el nuevo clúster.

Para obtener más información, consulte el artículo: [Actualización del clúster de HDInsight a una versión más reciente](../hdinsight-upgrade-cluster.md).

## <a name="patch-cluster-operating-systems"></a>Revise los sistemas operativos del clúster

Para obtener más información, consulte el artículo: [Aplicación de revisión del SO para HDInsight](../hdinsight-os-patching.md).

## <a name="post-migration"></a>Después de la migración

1. **Corregir aplicaciones**: realice de forma iterativa los cambios necesarios en los trabajos, procesos y scripts.
2. **Realizar pruebas** : realice pruebas funcionales y de rendimiento de forma iterativa.
3. **Optimizar**: aborde cualquier problema de rendimiento basándose en los resultados de las pruebas anteriores y luego vuelva a realizar la prueba para confirmar las mejoras de rendimiento.

## <a name="next-steps"></a>Pasos siguientes

Más información acerca de [HDInsight 4.0](https://docs.microsoft.com/azure/hdinsight/hadoop/apache-hadoop-introduction)
