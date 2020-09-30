---
title: Preguntas más frecuentes
description: Respuestas a las preguntas más frecuentes sobre Microsoft Azure Attestation
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: reference
ms.date: 07/20/2020
ms.author: mbaldwin
ms.openlocfilehash: 39f628845bdc9d54b48b1c8037f4a506a9d5c00a
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89236550"
---
# <a name="frequently-asked-questions-for-microsoft-azure-attestation"></a>Preguntas más frecuentes sobre Microsoft Azure Attestation

En este artículo se proporcionan respuestas a algunas de las preguntas más comunes sobre [Azure Attestation](overview.md).

Si tiene un problema relacionado con Azure que no se trata en este artículo, también puede enviar una solicitud de soporte técnico de Azure en la [página de soporte técnico de Azure](https://azure.microsoft.com/support/options/).

## <a name="what-is-azure-pck-caching-service-and-its-role-in-enclave-attestation"></a>Qué es el servicio de almacenamiento en caché de PCK de Azure y su rol en la atestación de enclaves

El servicio de almacenamiento en caché de PCK de Azure define la línea de base de seguridad de Azure para los nodos de [computación confidencial de Azure (ACC)](../confidential-computing/overview.md) de Intel y almacena en caché los datos. La información almacenada en caché la usará Azure Attestation para validar entornos de ejecución de confianza (TEE).  

El servicio de almacenamiento en caché de PCK de Azure tiene las siguientes características:
   - Ofrece alta disponibilidad. 
   - Reduce las dependencias en servicios hospedados externamente y en la conectividad a Internet.
   - Obtiene las versiones más recientes de los certificados Intel, las CRL, la información de la base de computación de confianza (TCB) y la identidad de enclaves de citas de los nodos de ACC de Intel. Por lo tanto, el servicio confirma la línea de base de seguridad de Azure a la que va a hacer referencia Azure Attestation durante la validación de los TEE, lo que reduce considerablemente los errores de atestación debido a la invalidación o la revocación de los certificados de Intel.  

## <a name="is-sgx-attestation-supported-by-azure-attestation-in-non-azure-environments"></a>¿La atestación de SGX es compatible con Azure Attestation en entornos que no son de Azure?

Azure Attestation depende de la línea de base de seguridad que indique el servicio de almacenamiento en caché de PCK de Azure para validar los TEE. El servicio de almacenamiento en caché de PCK de Azure está diseñado actualmente para admitir solamente los nodos de computación confidencial de Azure. 

## <a name="what-validations-does-azure-attestation-perform-for-attesting-sgx-enclaves"></a>¿Qué validaciones realiza Azure Attestation para atestiguar enclaves de SGX?

Azure Attestation es un marco unificado para atestiguar de forma remota distintos tipos de TEE. Azure Attestation se encarga de lo siguiente:

   - Valida si la raíz de confianza de una cita del enclave firmado pertenece a Intel.
   - Valida si la cita del enclave cumple la línea de base de seguridad de Azure, tal como se define en el servicio de almacenamiento en caché de PCK de Azure.
   - Valida si el hash SHA256 de los datos contenidos en el enclave (EHD) del objeto de solicitud de atestación coincide con los primeros 32 bytes del campo reportData de la cita del enclave.
   - Permite a los clientes crear un proveedor de atestación y configurar una directiva personalizada. Además de las validaciones anteriores, Azure Attestation evalúa la cita del enclave con respecto a la directiva. Las directivas definen las reglas de autorización para el enclave y también dictan reglas de emisión para generar el token de atestación. Para confirmar si el software previsto se está ejecutando en un enclave, los clientes pueden agregar reglas de autorización para comprobar si los campos **mrsigner** y **mrenclave** de la cita del enclave coinciden con los valores de los archivos binarios del cliente.

## <a name="how-can-a-verifier-obtain-the-collateral-for-sgx-attestation-supported-by-azure-attestation"></a>¿Cómo se puede obtener el colateral de la atestación de SGX compatible con Azure Attestation?

En general, en los modelos de atestación con Intel como raíz de confianza, el cliente de atestación se comunica con las API de enclave para capturar la evidencia del enclave. Las API de enclave llaman internamente al servicio de almacenamiento en caché de PCK de Intel para capturar los certificados de Intel del nodo que se va a atestiguar. Los certificados se usan para firmar la evidencia del enclave, con lo que se genera un colateral de forma remota.  

El mismo proceso se puede implementar para Azure Attestation. Sin embargo, para aprovechar las ventajas que ofrece el servicio de almacenamiento en caché de PCK de Azure, después de instalar la máquina virtual de ACC, se recomienda instalar la [biblioteca de Azure DCAP](https://www.nuget.org/packages/Microsoft.Azure.DCAP). En función del contrato con Intel, cuando se instala la biblioteca de Azure DCAP, las solicitudes de generación de evidencias de enclave se redirigen desde el servicio de almacenamiento en caché de PCK de Intel al servicio de almacenamiento en caché de PCK de Azure. La biblioteca de Azure DCAP es compatible con entornos basados en Windows y Linux.

## <a name="how-to-shift-to-azure-attestation-from-other-attestation-models"></a>Migración a Azure Attestation desde otros modelos de atestación

- Después de instalar la máquina virtual de ACC, instale la biblioteca de Azure DCAP ([Windows/](https://www.nuget.org/packages/Microsoft.Azure.DCAP/)[Linux](https://packages.microsoft.com/ubuntu/18.04/prod/pool/main/a/az-dcap-client/)) para aprovechar las ventajas que ofrece el servicio de almacenamiento en caché de PCK de Azure.
- Es necesario crear el cliente de atestación remoto que pueda recuperar la evidencia del enclave y enviar solicitudes a Azure Attestation. Consulte [estos códigos de ejemplos](/samples/browse/?expanded=azure&terms=attestation) como referencia. 
- Las solicitudes de atestación se pueden enviar al punto de conexión de la API REST de proveedores predeterminados o proveedores de atestación personalizados. 
- Las API de Azure Attestation están protegidas por la autenticación de Azure AD. Por lo tanto, el cliente que invoca las API de atestación debe poder obtener y pasar un token de acceso de Azure AD válido en la solicitud de atestación. 

## <a name="how-can-the-relying-party-verify-the-integrity-of-attestation-token"></a>¿Cómo comprueba el usuario de confianza la integridad del token de atestación?

El token de atestación que genera Azure Attestation se firma con un certificado autofirmado. Los certificados se exponen a través de un [punto de conexión de metadatos de OpenID](/rest/api/attestation/metadataconfiguration/get). El usuario de confianza puede recuperar los certificados de firma de este punto de conexión y realizar la comprobación de la firma del token de atestación. El tiempo de validez del token de atestación es de 8 horas. 

## <a name="how-to-identify-the-certificate-to-be-used-for-signature-verification-from-the-openid-metadata-endpoint"></a>¿Cómo se puede identificar el certificado que se va a usar para la comprobación de firmas desde el punto de conexión de metadatos de OpenID?

Hay varios certificados expuestos en el punto de conexión de metadatos de OpenID que se corresponden con distintos casos de uso (por ejemplo, la atestación de SGX) compatibles con Azure Attestation. Según los estándares especificados por [RFC 7515](https://tools.ietf.org/html/rfc7515), el certificado con el identificador de clave (kid) que coincida con el parámetro *kid* del encabezado del token de atestación se usará para la comprobación de la firma. Si no se encuentra ninguna clave **kid** coincidente, pruebe todos los certificados expuestos por el punto de conexión de metadatos de OpenID.

## <a name="is-it-possible-for-the-relying-party-to-share-secrets-with-the-validated-trusted-execution-environments-tees"></a>¿Es posible que el usuario de confianza comparta secretos con los entornos de ejecución de confianza validados (TEE)?

La clave pública generada en un enclave se puede expresar en la propiedad de los datos contenidos en el enclave (EHD) del objeto de solicitud de atestación enviado por el cliente a Azure Attestation. Después de confirmar si el hash SHA256 de EHD se expresa en el campo reportData de la cita, Azure Attestation incluye EHD en el token de atestación. El usuario de confianza puede usar los datos EHD de la respuesta de atestación comprobada para cifrar los secretos y compartirlos con el enclave. Para obtener más información, consulte el artículo sobre [conceptos básicos de Azure Attestation](basic-concepts.md).
