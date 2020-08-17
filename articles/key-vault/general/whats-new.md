---
title: Novedades de Azure Key Vault | Microsoft Docs
description: Actualizaciones recientes de Azure Key Vault
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 07/27/2020
ms.author: mbaldwin
ms.openlocfilehash: 6df9d5de88a3317c509279a7667c1a60eaf26a79
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87563656"
---
# <a name="whats-new-for-azure-key-vault"></a>Novedades de Azure Key Vault

> [!WARNING]
> **Julio de 2020**: hay dos actualizaciones para Key Vault que pueden afectar a las implementaciones del servicio: [activación predeterminada de la eliminación temporal de almacenes de claves](#soft-delete-on-by-default) y [cambios en los certificados TLS de Azure](#azure-tls-certificate-changes). Consulte a continuación para más información.

Estas son las novedades de Azure Key Vault. También se anuncian nuevas características y mejoras en el [Canal de Key Vault de actualizaciones de Azure](https://azure.microsoft.com/updates/?category=security&query=Key%20vault).

## <a name="soft-delete-on-by-default"></a>Activación predeterminada de la eliminación temporal

A finales de 2020, la **eliminación temporal estará activada de forma predeterminada en todos los almacenes de claves**, tanto nuevos como existentes. Para obtener toda la información no solo sobre este posible cambio importante, sino también sobre los pasos necesarios para buscar los almacenes de claves afectados y actualizarlos de antemano, consulte el artículo [La eliminación temporal se habilitará en todos los almacenes de claves](soft-delete-change.md). 

## <a name="azure-tls-certificate-changes"></a>Cambios en los certificados TLS de Azure  

Microsoft está actualizando los servicios de Azure para que usen los certificados TLS de un conjunto diferente de entidades de certificación (CA) raíz. Este cambio se realiza porque los certificados de entidad de certificación actuales no cumplen uno de los requisitos de la base de referencia del foro CA/Browser.

### <a name="when-will-this-change-happen"></a>¿Cuándo se producirá este cambio?

- Los servicios de Azure Active Directory (Azure AD) comenzaron esta transición el 7 de julio de 2020.
- Todos los puntos de conexión de TLS/SSL de Azure recién creados contienen certificados actualizados que se encadenan a las nuevas entidades de certificación raíz. 
- Los puntos de conexión de Azure existentes realizarán la transición de forma escalonada a partir del 13 de agosto de 2020 y finalizarán el 26 de octubre de 2020.

> [!IMPORTANT]
> Es posible que los clientes necesiten actualizar sus aplicaciones después de este cambio, con el fin de evitar errores de conectividad al intentar conectarse a los servicios de Azure. 

### <a name="what-is-changing"></a>¿Qué está cambiando?

En la actualidad, la mayoría de los certificados TLS usados por los servicios de Azure se encadenan a la siguiente CA raíz:

| Nombre común de la CA | Huella digital (SHA1) |
|--|--|
| [Baltimore CyberTrust Root](https://cacerts.digicert.com/BaltimoreCyberTrustRoot.crt) | d4de20d05e66fc53fe1a50882c78db2852cae474 |

Los certificados TLS usados por los servicios de Azure se encadenarán a una de las siguientes CA raíz:

| Nombre común de la CA | Huella digital (SHA1) |
|--|--|
| [DigiCert Global Root G2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt) | df3c24f9bfd666761b268073fe06d1cc8d4f82a4 |
| [DigiCert Global Root CA](https://cacerts.digicert.com/DigiCertGlobalRootCA.crt) | a8985d3a65e5e5c4b2d7d66d40c6dd2fb19c5436 |
| [Baltimore CyberTrust Root](https://cacerts.digicert.com/BaltimoreCyberTrustRoot.crt) | d4de20d05e66fc53fe1a50882c78db2852cae474 |
| [D-TRUST Root Class 3 CA 2 2009](https://www.d-trust.net/cgi-bin/D-TRUST_Root_Class_3_CA_2_2009.crt) | 58e8abb0361533fb80f79b1b6d29d3ff8d5f00f0 |
| [Microsoft RSA Root Certificate Authority 2017](https://www.microsoft.com/pkiops/certs/Microsoft%20RSA%20Root%20Certificate%20Authority%202017.crt) | 73a5e64a3bff8316ff0edccc618a906e4eae4d74 | 
| [Microsoft EV ECC Root Certificate Authority 2017](https://www.microsoft.com/pkiops/certs/Microsoft%20EV%20ECC%20Root%20Certificate%20Authority%202017.crt) | 6b1937abfd64e1e40daf2262a27857c015d6228d |

### <a name="when-can-i-retire-the-old-intermediate-thumbprint"></a>¿Cuándo puedo retirar la huella digital intermedia anterior?

Los certificados de CA actuales *no se* revocarán hasta el 15 de febrero de 2021. Después de esa fecha, puede quitar las huellas digitales anteriores del código.

Si esta fecha cambia, se le notificará la nueva fecha de revocación.

### <a name="will-this-affect-me"></a>¿Me afectará esto?

Esperamos que la **mayoría de los clientes de Azure no** se vean afectados.  Sin embargo, la aplicación puede resultar afectada si especifica explícitamente una lista de entidades de certificación aceptables. Esta práctica se conoce como anclaje de certificados.

Estas son algunas maneras de detectar si una aplicación se ve afectada:

- Busque en el código fuente la huella digital, el nombre común y otras propiedades de certificado de cualquiera de las entidades de certificación de Microsoft IT TLS que se encuentran [aquí](https://www.microsoft.com/pki/mscorp/cps/default.htm). Si hay alguna coincidencia, la aplicación se verá afectada. Para resolver este problema, actualice el código fuente para incluir las nuevas entidades de certificación. Como procedimiento recomendado, asegúrese de que las entidades de certificación se pueden agregar o editar rápidamente. Las regulaciones del sector requieren que los certificados de entidad de certificación se reemplacen en un plazo máximo de siete días, por lo que es preciso que los clientes que usan el anclaje reaccionen con rapidez.

- Si tiene alguna aplicación que se integra con las API de Azure u otros servicios de Azure y no está seguro de si usa el anclaje de certificados, póngase en contacto con el proveedor de la aplicación.

- Los distintos sistemas operativos y entornos de ejecución de lenguaje que se comunican con los servicios de Azure pueden requerir pasos adicionales para compilar correctamente la cadena de certificados con estas nuevas raíces: 
    - **Linux**: muchas distribuciones requieren que se agreguen las entidades de certificación anteriores a /etc/SSL/certs. Para obtener instrucciones específicas, consulte la documentación de la distribución.
    - **Java**: asegúrese de que el almacén de claves de Java contenga las entidades de certificación indicadas anteriormente.
    - **Windows se ejecuta en entornos desconectados**: los sistemas que se ejecuten en entornos desconectados deberán tener agregadas las CA raíz indicadas anteriormente en el almacén de entidades de certificación raíz de confianza y las CA intermedias agregadas en el almacén de entidades de certificación intermedias.
    - **Android**: consulte la documentación del dispositivo y la versión de Android.
    - **Otros dispositivos de hardware, especialmente IoT**: póngase en contacto con el fabricante del dispositivo. 

- Si en su entorno se ha establecido que las reglas de firewall permitan llamadas salientes solo a determinadas ubicaciones de descarga de la lista de revocación de certificados (CRL) o de comprobación del Protocolo de estado de certificados en línea (OCSP), tendrá que permitir las siguientes direcciones URL de CRL y OCSP:

    - http://crl3&#46;digicert&#46;com
    - http://crl4&#46;digicert&#46;com
    - http://ocsp&#46;digicert&#46;com
    - http://www&#46;d-trust&#46;net
    - http://root-c3-ca2-2009&#46;ocsp&#46;d-trust&#46;net
    - http://crl&#46;microsoft&#46;com
    - http://oneocsp&#46;microsoft&#46;com
    - http://ocsp&#46;msocsp&#46;com

## <a name="june-2020"></a>Junio de 2020

Ahora, Azure Monitor para Key Vault está en versión preliminar.  Azure Monitor proporciona una supervisión completa de los almacenes de claves proporcionando una vista unificada del rendimiento, los errores, la latencia y las solicitudes de Key Vault. Para obtener más información, consulte [Azure Monitor para Key Vault (versión preliminar).](../../azure-monitor/insights/key-vault-insights-overview.md).

## <a name="may-2020"></a>Mayo de 2020

El servicio Bring "Your Own Key" (BYOK) de Key Vault ya está disponible con carácter general. Consulte la [especificación BYOK de Azure Key Vault](../keys/byok-specification.md) y aprenda a [Importar claves protegidas con HSM a Key Vault (BYOK)](../keys/hsm-protected-keys-byok.md).

## <a name="march-2020"></a>Marzo de 2020

Ahora, los puntos de conexión privados están disponibles en versión preliminar. El servicio Azure Private Link permite acceder a Azure Key Vault y a los servicios de asociados o clientes hospedados de Azure mediante un punto de conexión privado de la red virtual.  Aprenda a [integrar Key Vault con Azure Private Link](private-link-service.md).

## <a name="2019"></a>2019

- Versión de los SDK de Azure Key Vault de última generación. Para ver ejemplos de su uso, consulte las guías de inicio rápido de Azure Key Vault para [Python](../secrets/quick-create-python.md), [.NET](../secrets/quick-create-net.md), [Java](../secrets/quick-create-java.md) y [Node.js.](../secrets/quick-create-node.md)
- Nuevas directivas de Azure para administrar certificados de Key Vault. Consulte las [definiciones integradas de Azure Policy para Key Vault](../policy-samples.md).
- La extensión de máquina virtual de Azure Key Vault ya está disponible con carácter general.  Consulte [extensión de máquina virtual de Key Vault para Linux](../../virtual-machines/extensions/key-vault-linux.md) y [extensión de máquina virtual de Key Vault para Windows](../../virtual-machines/extensions/key-vault-windows.md).
- Administración de secretos basada en eventos para Azure Key Vault ya disponible en Azure Event Grid. Para obtener más información, consulte [el esquema de Event Grid para eventos en Azure Key Vault](../../event-grid/event-schema-key-vault.md) y aprenda a [recibir y responder notificaciones del almacén de claves con Azure Event Grid](event-grid-tutorial.md).

## <a name="2018"></a>2018

Nuevas características e integraciones lanzadas este año:

- Integración con Azure Functions. Para ver un escenario de ejemplo en el que se aprovecha [Azure Functions](../../azure-functions/index.yml) para las operaciones del almacén de claves, consulte [Automatización de la rotación de secretos](../secrets/tutorial-rotation.md). 
- [Integración con Azure Databricks](/azure/databricks/scenarios/store-secrets-azure-key-vault). Con esto, ahora, Azure Databricks admite dos tipos de ámbitos secretos: basado en Azure Key Vault y en Databricks. Para obtener más información, consulte [Creación de un ámbito de secreto basado en Azure Key Vault](/azure/databricks/security/secrets/secret-scopes#--create-an-azure-key-vault-backed-secret-scope)
- [Puntos de conexión de servicio de red virtual para Azure Key Vault](overview-vnet-service-endpoints.md).

## <a name="2016"></a>2016

Nuevas características publicadas este año:

- Claves de cuenta de almacenamiento administrada. La característica Claves de cuenta de almacenamiento supuso una integración más sencilla con Azure Storage. Para más información, consulte el tema de introducción, [Claves de cuenta de almacenamiento de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-ovw-storage-keys).
- Eliminación temporal. La característica de eliminación temporal mejora la protección de los datos de los almacenes de claves y de los objetos de estos. Para más información, consulte el tema de introducción [Información general sobre la eliminación temporal de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-ovw-soft-delete).

## <a name="2015"></a>2015

Nuevas características publicadas este año:
- Administración de certificados. Se agregó como característica a la versión 2015-06-01 de disponibilidad general el 26 de septiembre de 2016.

La versión 2015-06-01 de disponibilidad general se anunció el 24 de junio de 2015. Se han realizado los siguientes cambios en esta versión: 
- Eliminación de una clave: se ha quitado el campo "uso".
- Obtención de información acerca de una clave: se ha quitado el campo "uso".
- Importación de una clave en un almacén: se ha quitado el campo "uso".
- Restauración de una clave: se ha quitado el campo "uso".     
- Se ha cambiado "RSA_OAEP" a "RSA-OAEP" para los algoritmos RSA. Consulte [Información acerca de claves, secretos y certificados](about-keys-secrets-certificates.md).    
 
La segunda versión preliminar 2015-02-01-preview se anunció el 20 de abril de 2015. Para más información, consulte la entrada de blog [REST API Update](https://docs.microsoft.com/archive/blogs/kv/rest-api-update) (Actualización de API REST). Se actualizaron las siguientes tareas:
 
- Enumeración de las claves en un almacén: se ha agregado compatibilidad con la paginación a la operación.
- Enumeración de las versiones de una clave: se ha agregado una operación para enumerar las versiones de una clave.  
- Enumeración de los secretos de un almacén: se ha agregado compatibilidad con la paginación.
- Enumeración de las versiones de un secreto: se ha agregado una operación para enumerar las versiones de un secreto.  
- Todas las operaciones: se han agregado marcas de tiempo de creación o actualización a los atributos.  
- Creación de un secreto: se ha agregado Content-Type a los secretos.
- Creación de una clave: se han agregado etiquetas como información opcional.
- Creación de un secreto: se han agregado etiquetas como información opcional.
- Actualización de una clave: se han agregado etiquetas como información opcional.
- Actualización de un secreto: se han agregado etiquetas como información opcional.
- Se ha cambiado el tamaño máximo de los secretos de 10 KB a 25 KB. Consulte [Información acerca de claves, secretos y certificados](about-keys-secrets-certificates.md).    

## <a name="2014"></a>2014
 
La primera versión preliminar 2014-12-08-preview se anunció el 8 de enero de 2015.  

## <a name="next-steps"></a>Pasos siguientes

Si tiene más preguntas, póngase en contacto con nosotros a través del departamento de [soporte técnico](https://azure.microsoft.com/support/options/).  
