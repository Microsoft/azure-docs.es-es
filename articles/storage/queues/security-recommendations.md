---
title: Recomendaciones de seguridad para Queue Storage
titleSuffix: Azure Storage
description: Obtenga información sobre recomendaciones para Queue Storage. La implementación de esta guía le ayudará a cumplir sus obligaciones de seguridad, tal y como se describe en nuestro modelo de responsabilidad compartida.
services: storage
author: tamram
ms.service: storage
ms.subservice: queues
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: tamram
ms.custom: security-recommendations
ms.openlocfilehash: 70bb96da858c94fbd2c75d56cda4e705f2ffa3ba
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90986631"
---
# <a name="security-recommendations-for-queue-storage"></a>Recomendaciones de seguridad para Queue Storage

Este artículo contiene recomendaciones de seguridad para Queue Storage. La implementación de estas recomendaciones le ayudará a cumplir sus obligaciones de seguridad, tal y como se describe en nuestro modelo de responsabilidad compartida. Para más información sobre lo que Microsoft hace para cumplir las responsabilidades del proveedor de servicios, consulte [Responsabilidades compartidas de la informática en la nube](https://gallery.technet.microsoft.com/Shared-Responsibilities-81d0ff91/file/225366/1/Shared%20Responsibility%20for%20Cloud%20Computing-2019-10-25.pdf).

Algunas de las recomendaciones incluidas en este artículo se pueden supervisar automáticamente mediante Azure Security Center. Azure Security Center es la primera línea de defensa en la protección de los recursos de Azure. Para más información sobre Azure Security Center, consulte [¿Qué es Azure Security Center?](../../security-center/security-center-intro.md)

Azure Security Center analiza periódicamente el estado de seguridad de los recursos de Azure para identificar posibles puntos vulnerables de la seguridad. Después, proporciona recomendaciones sobre cómo abordarlas. Para más información sobre las recomendaciones de seguridad de Azure Security Center, consulte [Recomendaciones de seguridad en Azure Security Center](../../security-center/security-center-recommendations.md).

## <a name="data-protection"></a>Protección de los datos

| Recomendación | Comentarios | Security Center |
|-|----|--|
| Usar el modelo de implementación de Azure Resource Manager | Cree cuentas de almacenamiento mediante el modelo de implementación de Azure Resource Manager, ya que así logrará importantes mejoras en la seguridad, entre las que se incluyen un control de acceso (RBAC) y una auditoría superiores, gobernanza e implementación basados en Resource Manager, acceso a identidades administradas, acceso a los secretos de Azure Key Vault y autorización y autenticación basadas en Azure AD para acceder a datos y recursos de Azure Storage. Si es posible, migre las cuentas de almacenamiento existentes que usan el modelo de implementación clásica para que usen Azure Resource Manager. Para más información sobre Azure Resource Manager, consulte [Introducción a Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview). | - |
| Habilitar Advanced Threat Protection en todas las cuentas de almacenamiento | Advanced Threat Protection para Azure Storage proporciona un nivel adicional de inteligencia de seguridad que detecta intentos poco habituales y potencialmente peligrosos de acceder a las cuentas de almacenamiento o vulnerarlas. En Azure Security Center se desencadenan alertas de seguridad cuando se producen anomalías en alguna actividad y también se envían por correo electrónico a los administradores de las suscripciones con detalles de la actividad sospechosa y recomendaciones sobre cómo investigar y solucionar las amenazas. Para más información, consulte [Advanced Threat Protection para Azure Storage](../common/azure-defender-storage-configure.md). | [Sí](../../security-center/security-center-sql-service-recommendations.md) |
| Limitar los tokens de firma de acceso compartido (SAS) solo a conexiones HTTPS | Requerir HTTPS cuando un cliente usa un token de SAS para acceder a los datos de la cola ayuda a minimizar el riesgo de espionaje. Para obtener más información, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../common/storage-sas-overview.md). | - |

## <a name="identity-and-access-management"></a>Administración de identidades y acceso

| Recomendación | Comentarios | Security Center |
|-|----|--|
| Usar Azure Active Directory (Azure AD) para autorizar el acceso a los datos de la cola | Azure AD proporciona seguridad superior y facilidad de uso sobre la clave compartida para autorizar solicitudes en Queue Storage. Para más información, consulte [Autenticación del acceso a blobs y colas de Azure con Azure Active Directory](../common/storage-auth-aad.md). | - |
| Tener en cuenta la entidad de seguridad de menor privilegio al asignar permisos a una entidad de seguridad de Azure AD a través de RBAC | Al asignar un rol a un usuario, grupo o aplicación, conceda a esa entidad de seguridad exclusivamente los permisos necesarios para que pueda realizar sus tareas. Limitar el acceso a los recursos ayuda a prevenir el mal uso involuntario y malintencionado de los datos. | - |
| Proteger las claves de acceso de su cuenta con Azure Key Vault | Microsoft recomienda usar Azure AD para autorizar las solicitudes que se realicen a Azure Storage. Sin embargo, si debe usar la autorización de clave compartida, proteja sus claves de cuenta con Azure Key Vault. Puede recuperar las claves del almacén de claves en tiempo de ejecución, en lugar de guardarlas con la aplicación. | - |
| Regenerar la claves de cuenta periódicamente | El cambio periódico de las claves de una cuenta reduce el riesgo de exponer los datos a actores malintencionados. | - |
| Tener en cuenta la entidad de seguridad de menor privilegio al asignar permisos a una SAS | Al crear una SAS, especifique solo aquellos permisos que el cliente requiera para realizar su función. La limitación del acceso a los recursos ayuda a prevenir el mal uso involuntario y malintencionado de los datos. | - |
| Tener en vigor un plan de revocación para cualquier SAS que emita a los clientes | Si alguna SAS corre peligro, seguro que deseará poder revocarla lo antes posible. Para revocar una SAS de delegación de usuario, revoque la clave de delegación de usuario para invalidar rápidamente todas las firmas asociadas con ella. Para revocar una SAS de servicio asociada a una directiva de acceso almacenado, puede eliminar esta, cambiar el nombre de la directiva, o bien cambiar su tiempo de vencimiento a un tiempo pasado. Para obtener más información, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../common/storage-sas-overview.md).  | - |
| Si una SAS de servicio no está asociada a una directiva de acceso almacenado, establezca el tiempo de vencimiento en una hora, o menos | Las SAS de servicio que no estén asociadas con alguna directiva de acceso almacenada no se pueden revocar. Por esta razón, se recomienda limitar el tiempo de expiración para que la SAS sea válida durante una hora o menos. | - |

## <a name="networking"></a>Redes

| Recomendación | Comentarios | Security Center |
|-|----|--|
| Configurar la versión mínima necesaria de Seguridad de la capa de transporte (TLS) para una cuenta de almacenamiento  | Exija a los clientes que usen una versión más segura de TLS para realizar solicitudes en una cuenta de Azure Storage configurando la versión mínima de TLS para esa cuenta. Para más información, consulte [Configuración de la versión mínima necesaria de Seguridad de la capa de transporte (TLS) para una cuenta de almacenamiento](../common/transport-layer-security-configure-minimum-version.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json).| - |
| Habilitar la opción **Se requiere transferencia segura** en todas las cuentas de almacenamiento | Cuando habilite la opción **Se requiere transferencia segura**, deben usarse conexiones seguras para realizar todas las solicitudes realizadas en la cuenta de almacenamiento. Las solicitudes realizadas a través de HTTP producirán un error. Para más información, consulte [Requerir transferencia segura en Azure Storage](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json). | [Sí](../../security-center/security-center-sql-service-recommendations.md) |
| Habilitar reglas de firewall | Configure las reglas de firewall para limitar el acceso a su cuenta de almacenamiento a las solicitudes que partan de direcciones IP o intervalos especificados, o de una lista de subredes de una red virtual de Azure (VNet). Para más información acerca de la configuración de reglas de firewall, consulte [Configuración de redes virtuales y firewalls de Azure Storage](../common/storage-network-security.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json). | - |
| Permitir que los servicios de Microsoft de confianza accedan a la cuenta de almacenamiento | La activación de las reglas de firewall para la cuenta de almacenamiento bloquea las solicitudes entrantes para los datos de forma predeterminada, a menos que las solicitudes procedan de un servicio que funcione en una instancia de Azure Virtual Network (VNet) o desde direcciones IP públicas permitidas. Las solicitudes que bloquean incluyen aquellas de otros servicios de Azure, desde Azure Portal, desde los servicios de registro y de métricas, etc. Para permitir solicitudes de otros servicios de Azure, agregue una excepción que permita que los servicios de Microsoft de confianza accedan a la cuenta de almacenamiento. Para más información acerca de la adición de una excepción para los servicios de Microsoft de confianza, consulte [Configuración de redes virtuales y firewalls de Azure Storage](../common/storage-network-security.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json).| - |
| Usar puntos de conexión privados | Un punto de conexión privado asigna una dirección IP privada de Azure Virtual Network (red virtual) a la cuenta de almacenamiento. Protege todo el tráfico que circule entre su red virtual y la cuenta de almacenamiento mediante un enlace privado. Para más información sobre los puntos de conexión privados, consulte [Conexión privada a una cuenta de almacenamiento mediante el punto de conexión privado de Azure](../../private-link/create-private-endpoint-storage-portal.md). | - |
| Uso de etiquetas de servicio de red virtual | Una etiqueta de servicio representa un grupo de prefijos de direcciones IP de un servicio de Azure determinado. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian. Para más información sobre las etiquetas de servicio compatibles con Azure Storage, consulte [Introducción a las etiquetas de servicio de Azure](../../virtual-network/service-tags-overview.md). Para ver un tutorial que muestra cómo usar las etiquetas de servicio para crear reglas de red de salida, consulte [Restricción del acceso a los recursos de PaaS](../../virtual-network/tutorial-restrict-network-access-to-resources.md). | - |
| Limitar el acceso a la red a determinadas redes | La limitación del acceso a la red a redes que hospeden clientes que requieran acceso reduce la exposición de sus recursos a ataques en la red. | [Sí](../../security-center/security-center-sql-service-recommendations.md) |

## <a name="loggingmonitoring"></a>Registro y supervisión

| Recomendación | Comentarios | Security Center |
|-|----|--|
| Hacer un seguimiento de cómo se autorizan las solicitudes | Habilite el registro de Azure Storage para realizar un seguimiento de cómo se autorizó cada solicitud realizada en Azure Storage. Los registros indican si una solicitud se realizó de forma anónima o mediante un token OAuth 2.0, una clave compartida o una firma de acceso compartido (SAS). Para más información, consulte [Registro de Azure Storage Analytics](../common/storage-analytics-logging.md). | - |

## <a name="next-steps"></a>Pasos siguientes

- [Documentación de Azure Security Center](https://docs.microsoft.com//azure/security/)
- [Documentación sobre desarrollo seguro](https://docs.microsoft.com/azure/security/develop/)
