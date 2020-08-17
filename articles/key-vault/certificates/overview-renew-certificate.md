---
title: Acerca de la renovación de certificados de Azure Key Vault
description: Acerca de la renovación de certificados de Azure Key Vault
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: conceptual
ms.date: 07/20/2020
ms.author: sebansal
ms.openlocfilehash: c6999b67a5c0a0f4ca7cb943ae8de3afd8b6a11e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095229"
---
# <a name="about-azure-key-vault-certificate-renewal"></a>Acerca de la renovación de certificados de Azure Key Vault

Azure Key Vault permite aprovisionar, administrar e implementar fácilmente certificados digitales para una red y habilitar las comunicaciones seguras para las aplicaciones. Para más información general acerca de los certificados, consulte [Certificados de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/certificates/about-certificates).

Tener certificados de corta duración o aumentar la frecuencia de la rotación de certificados limita el ámbito de los daños de roles no admitidos.

Hay tres categorías de creación de certificados en el almacén de claves. Esta guía le ayudará a entender cómo se puede realizar la renovación de certificados.
-   Certificados creados con CA integradas (DigiCert o GlobalSign)
-   Certificados creados con una CA no integrada
-   Certificados autofirmados

## <a name="renewal-of-integrated-ca-certificate"></a>Renovación del certificado con CA integrada 
¡Buenas noticias! Azure Key Vault se encarga del mantenimiento integral de los certificados que emiten las entidades de certificación de confianza de Microsoft; es decir, DigiCert y GlobalSign. Obtenga información sobre cómo [integrar una CA de confianza con Key Vault](https://docs.microsoft.com/azure/key-vault/certificates/how-to-integrate-certificate-authority).

## <a name="renewal-of-non-integrated-ca-certificate"></a>Renovación del certificado con CA no integrada 
Azure Key Vault proporciona a sus usuarios la ventaja de importar certificados de cualquier CA para que los usuarios puedan integrarlos con varios recursos de Azure y facilitar su implementación. Si le preocupa no saber exactamente cuándo expirará su certificado o ha descubierto que el certificado ya ha expirado, Key Vault puede ayudarle a mantenerse al día. En el caso de los certificados de CA no integrada, Key Vault permite que los usuarios configuren notificaciones por correo electrónico acerca de un vencimiento próximo. Estas notificaciones también se pueden configurar para varios usuarios.

Para renovar un certificado es importante comprender que un certificado de Azure Key Vault es un objeto con versiones. Si la versión actual expira, deberá crear una nueva versión. En términos conceptuales, cada nueva versión sería un certificado completamente nuevo compuesto por una clave y un blob que une esa clave a una identidad. Cuando use una CA no asociada, Key Vault generará un par clave-valor y devolverá la solicitud de firma de certificado.

**Pasos que se deben seguir en Azure Portal:**
1.  Abra el certificado que quiere renovar.
2.  Seleccione el botón **+ Nueva versión** en la pantalla Certificado.
3.  Seleccione **Operación de certificados**.
4.  Seleccionar **Descargar CSR**. Se descargará un archivo .csr en la unidad local.
5.  Lleve la CSR a la CA de su elección para firmar la solicitud.
6.  Recupere la solicitud firmada y seleccione **Merge CSR** (Fusionar CSR) en la misma pantalla Operación de certificados.

> [!NOTE]
> Es importante que fusione la CSR firmada con la misma solicitud CSR que se creó; de lo contrario, la clave no coincidiría.

Los pasos son similares a la creación de un certificado nuevo y se documentan de forma más detallada [aquí]( https://docs.microsoft.com/azure/key-vault/certificates/create-certificate-signing-request#azure-portal).

## <a name="renewal-of-self-signed-certificate"></a>Renovación de un certificado autofirmado

¡Más buenas noticias! Las instancias de Azure Key Vault también se encargarán de la renovación automática de los certificados autofirmados para sus usuarios. Para obtener más información sobre cómo cambiar la directiva de emisión y cómo actualizar los atributos de acción de la duración del certificado, consulte [este artículo](https://docs.microsoft.com/azure/key-vault/certificates/tutorial-rotate-certificates#update-lifecycle-attributes-of-a-stored-certificate).

### <a name="troubleshoot"></a>Solución de problemas
Si el certificado emitido se encuentra en el estado "deshabilitado" en Azure Portal, consulte la pantalla Operación de certificados y revise el mensaje de error sobre ese certificado.

### <a name="see-also"></a>Consulte también
*   [Integración de Key Vault con la entidad de certificación DigiCert](how-to-integrate-certificate-authority.md)
*   [Tutorial: Configuración de la rotación automática de certificados en Key Vault](tutorial-rotate-certificates.md)