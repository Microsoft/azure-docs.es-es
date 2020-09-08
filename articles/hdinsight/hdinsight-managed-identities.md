---
title: Identidades administradas en Azure HDInsight
description: Proporciona información general de la implementación de las identidades administradas en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 04/15/2020
ms.openlocfilehash: 07a8c26f7fc314680c51270ebafe03d4e3a84757
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88749862"
---
# <a name="managed-identities-in-azure-hdinsight"></a>Identidades administradas en Azure HDInsight

Una identidad administrada es una identidad registrada en Azure Active Directory (Azure AD) cuyas credenciales son administradas por Azure. Con identidades administradas, no es necesario registrar entidades de servicio en Azure AD. Tampoco mantener credenciales como certificados.

Se usan identidades administradas en Azure HDInsight para acceder a servicios de dominio de Azure AD o a archivos de Azure Data Lake Storage Gen2 cuando sea necesario.

Hay dos tipos de identidades administradas: asignadas por el sistema y asignadas por el usuario. Azure HDInsight solo admite identidades administradas asignadas por el usuario; no admite identidades administradas asignadas por el sistema. Una identidad administrada asignada por el usuario se crea como un recurso independiente de Azure que, a continuación, se puede asignar a una o varias instancias de servicio de Azure. En cambio, una identidad administrada asignada por el sistema se crea en Azure AD y, a continuación, se habilita directamente en una instancia en particular de un servicio de Azure de forma automática. La vida útil de esa identidad administrada asignada por el sistema se asocia entonces a la vida de la instancia de servicio en la que se ha habilitado.

## <a name="hdinsight-managed-identity-implementation"></a>Implementación de la identidad administrada de HDInsight

En Azure HDInsight, las identidades administradas solo se pueden usar en el servicio HDInsight para los componentes internos. Actualmente, no hay ningún método admitido para generar tokens de acceso con las identidades administradas instaladas en los nodos de clúster de HDInsight para acceder a los servicios externos. En algunos servicios de Azure, como máquinas virtuales de proceso, las identidades administradas se implementan con un punto de conexión que puede usar para adquirir tokens de acceso. Actualmente, este punto de conexión no está disponible en los nodos de HDInsight.

Si necesita arrancar las aplicaciones para evitar colocar secretos y contraseñas en los trabajos de análisis (por ejemplo, trabajos de SCALA), puede distribuir sus propios certificados a los nodos de clúster mediante acciones de script y, a continuación, usar ese certificado para adquirir un token de acceso (por ejemplo, para acceder a Azure KeyVault).

## <a name="create-a-managed-identity"></a>Creación de una entidad administrada

Las identidades administradas se pueden crear con cualquiera de los métodos siguientes:

* [Azure Portal](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md)
* [Azure PowerShell](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md)
* [Azure Resource Manager](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-arm.md)
* [CLI de Azure](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md)

Los pasos restantes para configurar la identidad administrada dependen del escenario donde se vaya a usar.

## <a name="managed-identity-scenarios-in-azure-hdinsight"></a>Escenarios de identidades administradas en Azure HDInsight

Las identidades administradas se usan en Azure HDInsight en varios escenarios. Consulte en los documentos relacionados las instrucciones detalladas para la instalación y la configuración:

* [Azure Data Lake Storage Gen2](hdinsight-hadoop-use-data-lake-storage-gen2.md#create-a-user-assigned-managed-identity)
* [Paquete de seguridad de la empresa](domain-joined/apache-domain-joined-configure-using-azure-adds.md#create-and-authorize-a-managed-identity)
* [Cifrado de disco mediante claves administradas por el cliente](disk-encryption.md)

## <a name="faq"></a>Preguntas más frecuentes

### <a name="what-happens-if-i-delete-the-managed-identity-after-the-cluster-creation"></a>¿Qué ocurre si elimino la identidad administrada después de la creación del clúster?

El clúster tendrá problemas cuando se necesite la identidad administrada. Actualmente no hay ninguna manera de actualizar o cambiar la identidad administrada una vez creado el clúster. Por lo tanto, nuestra recomendación es asegurarse de que la identidad administrada no se elimina durante el tiempo de ejecución del clúster. También puede volver a crear el clúster y asignar una nueva identidad administrada.

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué son las identidades administradas de los recursos de Azure?](../active-directory/managed-identities-azure-resources/overview.md)
