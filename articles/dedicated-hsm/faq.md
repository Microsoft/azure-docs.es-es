---
title: Preguntas frecuentes sobre Azure Dedicated HSM | Microsoft Docs
description: Encuentre respuestas a preguntas habituales sobre el módulo de seguridad de hardware dedicado de Azure, como la información básica, la interoperabilidad, la alta disponibilidad y el soporte técnico.
services: dedicated-hsm
author: johncdawson
manager: rkarlin
tags: azure-resource-manager
ms.custom: mvc, seodec18
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 02/05/2020
ms.author: mbaldwin
ms.openlocfilehash: 0d3309283279c887c00a475ccedfe1edaf311ae5
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88189866"
---
# <a name="frequently-asked-questions-faq"></a>Preguntas más frecuentes

Encuentre respuestas a preguntas habituales sobre Azure Dedicated HSM.

## <a name="the-basics"></a>Conceptos básicos

### <a name="q-what-is-a-hardware-security-module-hsm"></a>P: ¿Qué es un módulo de seguridad de hardware (HSM)?

Un módulo de seguridad de hardware (HSM) es un dispositivo informático físico utilizado para proteger y administrar claves criptográficas. Las claves almacenadas en los módulos HSM se pueden usar para operaciones criptográficas. El material de la clave permanece de forma segura en módulos de hardware resistentes a manipulaciones y con evidencia de alteración. El módulo HSM solo permite que utilicen las claves las aplicaciones autenticadas y autorizadas. El material de la clave no sale nunca de los límites de protección del HSM.

### <a name="q-what-is-the-azure-dedicated-hsm-offering"></a>P: ¿Cuál es la oferta de Azure Dedicated HSM?

Azure Dedicated HSM es un servicio basado en la nube que proporciona módulos HSM hospedados en centros de datos de Azure que se conectan directamente a la red virtual de un cliente. Estos módulos HSM son dispositivos de red dedicados (SafeNet Network HSM 7 modelo A790 de Gemalto). Se implementan directamente en el espacio de direcciones IP privadas de un cliente y Microsoft no tiene acceso a la funcionalidad criptográfica de los HSM. Solo el cliente tiene control administrativo y criptográfico completo de estos dispositivos. Los clientes son responsables de la administración del dispositivo y pueden obtener registros de actividad completos directamente desde los dispositivos. Los módulos HSM dedicados ayudan a los clientes a cumplir normas como FIPS 140-2 nivel 3, HIPAA, PCI-DSS, eIDAS y muchas otras.

### <a name="q-what-hardware-is-used-for-dedicated-hsm"></a>P: ¿Qué hardware se usa para Dedicated HSM?

Microsoft ha colaborado con Gemalto para proporcionar el servicio Azure Dedicated HSM. El dispositivo específico utilizado es el [SafeNet Luna Network HSM 7 modelo A790](https://safenet.gemalto.com/data-encryption/hardware-security-modules-hsms/safenet-network-hsm/). Este dispositivo no solo proporciona firmware con certificación FIPS 140-2 nivel 3, sino que también ofrece baja latencia, alto rendimiento y alta capacidad con 10 particiones. 

### <a name="q-what-is-an-hsm-used-for"></a>P: ¿Para qué se utiliza un módulo HSM?

Los módulos HSM se utilizan para almacenar claves que se utilizan para funcionalidad criptográfica como TLS (seguridad de la capa de transporte), cifrado de datos, PKI (infraestructura de clave pública), DRM (administración de derechos digitales) y firma de documentos.

### <a name="q-how-does-dedicated-hsm-work"></a>P: ¿Cómo funciona Dedicated HSM?

Los clientes pueden aprovisionar módulos HSM en regiones específicas con PowerShell o la interfaz de la línea de comandos. El cliente especifica a qué red virtual se conectarán los módulos HSM y, una vez aprovisionados, estarán disponibles en la subred designada, en las direcciones IP asignadas del espacio de direcciones IP privadas del cliente. Después, los clientes se pueden conectar a los módulos HSM mediante SSH para administrar los dispositivos, configurar conexiones de cliente HSM, inicializar módulos HSM, crear particiones y definir y asignar roles, como responsable de partición, responsable de criptografía y usuario de criptografía. A continuación, los clientes usarán las herramientas, el SDK y el software cliente de HSM proporcionados por Gemalto para llevar a cabo operaciones criptográficas desde las aplicaciones.

### <a name="q-what-software-is-provided-with-the-dedicated-hsm-service"></a>P: ¿Qué software se suministra con el servicio Dedicated HSM?

Gemalto proporciona todo el software para el dispositivo HSM una vez aprovisionado por Microsoft. El software está disponible en el [portal de soporte técnico para clientes de Gemalto](https://supportportal.gemalto.com/csm/). Los clientes que usan el servicio Dedicated HSM deben registrarse en el soporte técnico de Gemalto y disponer de un identificador de cliente que permite el acceso y descarga del software pertinente. El software cliente admitido es la versión 7.2, que es compatible con la versión 7.0.3 del firmware con certificación FIPS 140-2 nivel 3. 

### <a name="q-does-azure-dedicated-hsm-offer-password-based-and-ped-based-authentication"></a>P: ¿Ofrece Azure Dedicated HSM autenticación basada en contraseña y PED?

En este momento, Azure Dedicated HSM solo proporciona módulos HSM con autenticación basada en contraseña.

### <a name="q-will-azure-dedicated-hsm-host-my-hsms-for-me"></a>P: ¿Podrá hospedar Azure Dedicated HSM mis módulos HSM?

Microsoft solo ofrece el módulo SafeNet Luna Network HSM de Gemalto mediante el servicio Dedicated HSM y no puede hospedar ningún servicio proporcionado por el cliente.

### <a name="q-does-azure-dedicated-hsm-support-payment-pineft-features"></a>P: ¿Admite Azure Dedicated HSM características de pago (PIN o ETF)?

El servicio Azure Dedicated HSM usa dispositivos SafeNet Luna Network HSM 7 (modelo A790). Estos dispositivos no admiten la funcionalidad específica de HSM de pago (por ejemplo, PIN o ETF) ni certificaciones. Si quiere que el servicio Azure Dedicated HSM admita módulos HSM de pago en el futuro, envíe sus comentarios a su representante de cuenta Microsoft.

### <a name="q-which-azure-regions-is-dedicated-hsm-available-in"></a>P: ¿En qué regiones de Azure está disponible Dedicated HSM?

Desde finales de marzo de 2019, Dedicated HSM está disponible en las 14 regiones que se indican a continuación. Además, está prevista la inclusión de más regiones. Si le interesan otras, coménteselo a su representante de cuenta Microsoft.

* Este de EE. UU.
* Este de EE. UU. 2
* Oeste de EE. UU.
* Oeste de EE. UU. 2
* Centro-sur de EE. UU.
* Sudeste de Asia
* Este de Asia
* India central
* Sur de India
* Japón Oriental
* Japón Occidental
* Norte de Europa
* Oeste de Europa
* Sur de Reino Unido
* Oeste de Reino Unido
* Centro de Canadá
* Este de Canadá
* Este de Australia
* Sudeste de Australia
* Norte de Suiza
* Oeste de Suiza
* US Gov - Virginia
* US Gov Texas

## <a name="interoperability"></a>Interoperabilidad

### <a name="q-how-does-my-application-connect-to-a-dedicated-hsm"></a>P: ¿Cómo se conecta la aplicación a una instancia de Dedicated HSM?

Se utilizan las herramientas, el SDK y el software cliente de HSM proporcionados por Gemalto para llevar a cabo operaciones criptográficas desde las aplicaciones. El software está disponible en el [portal de soporte técnico para clientes de Gemalto](https://supportportal.gemalto.com/csm/). Los clientes que usan el servicio Dedicated HSM deben registrarse en el soporte técnico de Gemalto y disponer de un identificador de cliente que permite el acceso y descarga del software pertinente.

### <a name="q-can-an-application-connect-to-dedicated-hsm-from-a-different-vnet-in-or-across-regions"></a>P: ¿Se puede conectar una aplicación a Dedicated HSM desde otra red virtual o entre regiones?

Sí, deberá usar el [emparejamiento de redes virtuales](../virtual-network/virtual-network-peering-overview.md) dentro de una región para establecer la conectividad entre redes virtuales. Para la conectividad entre regiones, debe usar [VPN Gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md).

### <a name="q-can-i-synchronize-dedicated-hsm-with-on-premises-hsms"></a>P: ¿Puedo sincronizar Dedicated HSM con módulos HSM locales?

Sí, puede sincronizar los módulos HSM locales con Dedicated HSM. Puede usar conectividad [VPN de punto a punto o de punto a sitio](../vpn-gateway/vpn-gateway-about-vpngateways.md) para establecer la conectividad con la red local.

### <a name="q-can-i-encrypt-data-used-by-other-azure-services-using-keys-stored-in-dedicated-hsm"></a>P: ¿Puedo cifrar los datos utilizados por otros servicios de Azure con claves almacenadas en Dedicated HSM?

No. Los dispositivos de Azure Dedicated HSM solo son accesibles desde dentro de la red virtual.

### <a name="q-can-i-import-keys-from-an-existing-on-premises-hsm-to-dedicated-hsm"></a>P: ¿Puedo importar claves desde un módulo HSM local existente a Dedicated HSM?

Sí, si dispone de los módulos HSM SafeNet de Gemalto en el entorno local. Existen varios métodos. Consulte la documentación de HSM de Gemalto.

### <a name="q-what-operating-systems-are-supported-by-dedicated-hsm-client-software"></a>P: ¿Qué sistemas operativos son compatibles con el software cliente de Dedicated HSM?

* Windows, Linux, Solaris, AIX, HP-UX, FreeBSD
* Virtual: VMware, Hyper-V, Xen, KVM

### <a name="q-how-do-i-configure-my-client-application-to-create-a-high-availability-configuration-with-multiple-partitions-from-multiple-hsms"></a>P: ¿Cómo se configura la aplicación cliente para crear una configuración de alta disponibilidad con varias particiones de varios HSM?

Para tener alta disponibilidad, debe establecer la configuración de la aplicación cliente de HSM para utilizar particiones de cada HSM. Consulte la documentación del software cliente de HSM de Gemalto.

### <a name="q-what-authentication-mechanisms-are-supported-by-dedicated-hsm"></a>P: ¿Qué mecanismos de autenticación son compatibles con Dedicated HSM?

Azure Dedicated HSM usa dispositivos SafeNet Network HSM 7 (modelo A790) y estos admiten autenticación basada en contraseña.

### <a name="q-what-sdks-apis-client-software-is-available-to-use-with-dedicated-hsm"></a>P: ¿Qué software cliente, SDK y API están disponibles para su uso con Dedicated HSM?

PKCS#11, Java (JCA/JCE), Microsoft CAPI y CNG, OpenSSL

### <a name="q-can-i-importmigrate-keys-from-luna-56-hsms-to-azure-dedicated-hsms"></a>P: ¿Puedo importar o migrar claves desde módulos HSM Luna 5/6 a Azure Dedicated HSM?

Sí. Consulte la guía de migración de Gemalto. 

## <a name="using-your-hsm"></a>Uso de HSM

### <a name="q-how-do-i-decide-whether-to-use-azure-key-vault-or-azure-dedicated-hsm"></a>P: ¿Cómo decido si usar Azure Key Vault o Azure Dedicated HSM?

Azure Dedicated HSM es la opción adecuada para las empresas que migran a Azure aplicaciones locales que usan módulos HSM. Los módulos HSM dedicados son una opción para migrar una aplicación con cambios mínimos. Si las operaciones criptográficas se realizan en el código de la aplicación que se ejecuta en una máquina virtual de Azure o una aplicación web, pueden usar Azure Dedicated HSM. En general, el software empaquetado que se ejecuta en modelos de IaaS (infraestructura como servicio) y que admite los módulos HSM como un almacén de claves puede usar HSM dedicado, como una puerta de enlace de aplicación o un administrador de tráfico para TLS sin clave, ADCS (servicios de certificados de Active Directory) o herramientas similares de PKI, herramientas o aplicaciones utilizadas para la firma de documentos, firma de código o un servidor SQL Server (IaaS) configurado con TDE (cifrado de base de datos transparente) con la clave maestra en un módulo HSM mediante un proveedor de EKM (administración extensible de claves). Azure Key Vault es adecuado para aplicaciones "nacidas en la nube" o para el cifrado en escenarios en reposo en los que el software PaaS (plataforma como servicio) procesa los datos del cliente, o escenarios SaaS (software como servicio) como la clave de cliente de Office 365, Azure Information Protection, Azure Disk Encryption, el cifrado de Azure Data Lake Store con clave administrada por el cliente, el cifrado de Azure Storage con clave administrada por el cliente y Azure SQL con clave administrada por el cliente.

### <a name="q-what-usage-scenarios-best-suit-azure-dedicated-hsm"></a>P: ¿Qué escenarios de uso se adaptan mejor a Azure Dedicated HSM?

Azure Dedicated HSM es más adecuado para escenarios de migración. Por ejemplo, si va a migrar a Azure aplicaciones locales que ya usan módulos HSM. Esto proporciona una opción de baja fricción para la migración a Azure con cambios mínimos en la aplicación. Si las operaciones criptográficas se realizan en el código de la aplicación que se ejecuta en una máquina virtual de Azure o una aplicación web, se puede usar Dedicated HSM. En general, el software empaquetado que se ejecuta en modelos de IaaS (infraestructura como servicio) y que admite los módulos HSM como un almacén de claves puede usar HSM dedicado, por ejemplo:

* Puerta de enlace de aplicación o administrador de tráfico para TLS sin claves
* ADCS (servicios de certificados de Active Directory)
* Herramientas similares de PKI
* Herramientas y aplicaciones usadas para firmar documentos
* Firma de código
* SQL Server (IaaS) configurado con TDE (cifrado de base de datos transparente) con la clave maestra en un módulo HSM mediante un proveedor de EKM (administración extensible de claves)

### <a name="q-can-dedicated-hsm-be-used-with-office-365-customer-key-azure-information-protection-azure-data-lake-store-disk-encryption-azure-storage-encryption-azure-sql-tde"></a>P: ¿Se puede usar Dedicated HSM con la clave de cliente de Office 365, Azure Information Protection, Azure Data Lake Store, Disk Encryption, el cifrado de Azure Storage o TDE de Azure SQL?

No. Dedicated HSM se aprovisiona directamente en el espacio de direcciones IP privadas de un cliente y, por tanto, no es accesible por otros servicios de Azure o Microsoft.

## <a name="administration-access-and-control"></a>Administración, acceso y control

### <a name="q-does-the-customer-get-full-exclusive-control-over-the-hsms-with-dedicated-hsms"></a>P: ¿El cliente tiene control exclusivo completo sobre los módulos HSM con Dedicated HSM?

Sí. Cada dispositivo HSM está totalmente dedicado a un solo cliente y nadie más tiene control administrativo una vez se ha aprovisionado y se ha cambiado la contraseña del administrador.

### <a name="q-what-level-of-access-does-microsoft-have-to-my-hsm"></a>P: ¿Qué nivel de acceso tiene Microsoft al módulo HSM?

Microsoft no tiene ningún control administrativo ni criptográfico sobre el módulo HSM. Microsoft tiene acceso de nivel de supervisión a través de la conexión de puerto serie para recuperar datos de telemetría básicos, como el mantenimiento de los componentes y la temperatura. Esto permite a Microsoft proporcionar una notificación proactiva de los problemas de mantenimiento. Si es necesario, el cliente puede deshabilitar esta cuenta.

### <a name="q-what-is-the-tenantadmin-account-microsoft-uses-i-am-used-to-the-admin-user-being-admin-on-safenet-hsms"></a>P: ¿Qué es la cuenta "tenantadmin" que usa Microsoft? Estoy acostumbrado a que el usuario administrador sea "admin" en módulos HSM SafeNet.

El dispositivo HSM se suministra con un usuario administrador predeterminado con su contraseña predeterminada habitual. Microsoft no quería tener contraseñas predeterminadas en uso mientras los dispositivos se encontraban en un grupo a la espera de que los aprovisionaran los clientes, ya que no se cumplirían los estrictos requisitos de seguridad. Por este motivo, hemos establecido una contraseña segura que se descarta en el momento del aprovisionamiento. Además, cuando se realiza el aprovisionamiento, se cree un usuario con un rol de administrador denominado "tenantadmin". Este usuario tiene la contraseña predeterminada, que los clientes deben cambiar como primera acción al iniciar sesión en el dispositivo recién aprovisionado. Este proceso garantiza altos niveles de seguridad y nos ayuda a mantener nuestra promesa de ofrecer un control administrativo exclusivo a los clientes. Tenga en cuenta que el usuario "tenantadmin" puede emplearse para restablecer la contraseña de usuario administrador si un cliente prefiere usar esa cuenta. 

### <a name="q-can-microsoft-or-anyone-at-microsoft-access-keys-in-my-dedicated-hsm"></a>P: ¿Puede Microsoft o alguien de Microsoft acceder a las claves del módulo Dedicated HSM?

No. Microsoft no tiene acceso a las claves almacenadas en un módulo HSM dedicado asignado a un cliente.

### <a name="q-can-i-upgrade-softwarefirmware-on-hsms-allocated-to-me"></a>P: ¿Puedo actualizar el software o el firmware de los módulos HSM que tengo asignados?

Para obtener el mejor soporte técnico, Microsoft recomienda no actualizar el software ni el firmware del módulo HSM. Sin embargo, el cliente tiene control administrativo total, incluida la actualización de software o firmware si se requieren características específicas de otras versiones del firmware. Antes de realizar cambios se deben entender las implicaciones, ya que esto podría, por ejemplo, afectar al estado de validación FIPS. 

### <a name="q-how-do-i-manage-dedicated-hsm"></a>P: ¿Cómo puedo administrar Dedicated HSM?

Puede administrar los módulos HSM dedicados mediante el acceso a ellos con SSH.

### <a name="q-how-do-i-manage-partitions-on-the-dedicated-hsm"></a>P: ¿Cómo puedo administrar las particiones en Dedicated HSM?

El software cliente de HSM de Gemalto se utiliza para administrar las particiones y los HSM.

### <a name="q-how-do-i-monitor-my-hsm"></a>P: ¿Cómo puedo supervisar el módulo HSM?

El cliente tiene acceso completo a los registros de actividad del HSM mediante syslog y SNMP. El cliente deberá configurar un servidor syslog o SNMP para recibir los eventos o los registros de los módulos HSM.

### <a name="q-can-i-get-full-access-log-of-all-hsm-operations-from-dedicated-hsm"></a>P: ¿Puedo obtener el registro de acceso completo de todas las operaciones de HSM de Dedicated HSM?

Sí. Puede enviar registros desde el dispositivo HSM a un servidor syslog

## <a name="high-availability"></a>Alta disponibilidad

### <a name="q-is-it-possible-to-configure-high-availability-in-the-same-region-or-across-multiple-regions"></a>P: ¿Es posible configurar la alta disponibilidad en la misma región o en varias regiones?

Sí. La configuración de alta disponibilidad se realiza en el software cliente de HSM proporcionado por Gemalto. Los módulos HSM de la misma red virtual o de otras redes virtuales en la misma región o entre regiones y los módulos HSM locales conectados a una red virtual mediante una VPN de sitio a sitio o de punto a punto se pueden agregar a la misma configuración de alta disponibilidad. Tenga en cuenta que esto sincroniza solo material clave y no elementos de configuración específicos como los roles.

### <a name="q-can-i-add-hsms-from-my-on-premises-network-to-a-high-availability-group-with-azure-dedicated-hsm"></a>P: ¿Puedo agregar módulos HSM de la red local a un grupo de alta disponibilidad con Azure Dedicated HSM?

Sí. Deben cumplir los requisitos de alta disponibilidad de SafeNet Luna Network HSM 7.

### <a name="q-can-i-add-luna-56-hsms-from-on-premises-networks-to-a-high-availability-group-with-azure-dedicated-hsm"></a>P: ¿Puedo agregar módulos HSM Luna 5/6 de la red local a un grupo de alta disponibilidad con Azure Dedicated HSM?

No.

### <a name="q-how-many-hsms-can-i-add-to-the-same-high-availability-configuration-from-one-single-application"></a>P: ¿Cuántos módulos HSM puedo agregar a la misma configuración de alta disponibilidad desde una sola aplicación?

Se han sometido a prueba 16 miembros de un grupo de alta disponibilidad a pleno rendimiento y se han obtenido excelentes resultados.

## <a name="support"></a>Soporte técnico

### <a name="q-what-is-the-sla-for-dedicated-hsm-service"></a>P: ¿Cuál es el Acuerdo de Nivel de Servicio de Dedicated HSM?

No existe ninguna garantía de tiempo de actividad específico para el servicio Dedicated HSM. Microsoft garantizará el acceso de nivel de red al dispositivo y, por lo tanto, se aplican los SLA estándar de redes de Azure.

### <a name="q-how-are-the-hsms-used-in-azure-dedicated-hsm-protected"></a>P: ¿Cómo se protegen los módulos HSM utilizados en Azure Dedicated HSM?

Los centros de datos de Azure tienen controles de seguridad física y de procedimiento exhaustivos. Además, los módulos HSM dedicados se hospedan en un área de acceso más restringido del centro de datos. Estas áreas tienen controles de acceso físico adicionales y cámaras de vídeo de vigilancia para mayor seguridad.

### <a name="q-what-happens-if-there-is-a-security-breach-or-hardware-tampering-event"></a>P: ¿Qué ocurre si se produce una infracción de seguridad o un evento de manipulación del hardware?

El servicio Dedicated HSM utiliza dispositivos SafeNet Network HSM 7. Estos dispositivos admiten detección de alteraciones físicas y lógicas. Si alguna vez se diese un evento de alteración, los módulos HSM se graban con ceros automáticamente.

### <a name="q-how-do-i-ensure-that-keys-in-my-dedicated-hsms-are-not-lost-due-to-error-or-a-malicious-insider-attack"></a>P: ¿Cómo me aseguro de que las claves de los módulos de Dedicated HSM no se pierden debido a un error o un ataque malintencionado interno?

Se recomienda usar un dispositivo HSM local de respaldo para realizar la copia de seguridad periódica de los HSM para la recuperación ante desastres. Deberá usar una conexión VPN de sitio a sitio o de punto a punto a una estación de trabajo local conectada a un dispositivo de copia de seguridad de HSM.

### <a name="q-how-do-i-get-support-for-dedicated-hsm"></a>P: ¿Cómo puedo obtener soporte técnico para Dedicated HSM?

Microsoft y Gemalto ofrecen soporte técnico.  Si tiene algún problema con el hardware o el acceso de red, presente una solicitud de soporte técnico a Microsoft. Si tiene algún problema con el desarrollo de aplicaciones, el software y la configuración de HSM, presente una solicitud de soporte técnico a Gemalto. Si tiene un problema indeterminado, presente una solicitud de soporte técnico a Microsoft. Si es necesario, se recurrirá a Gemalto. 

### <a name="q-how-do-i-get-the-client-software-documentation-and-access-to-integration-guidance-for-the-safenet-luna-7-hsm"></a>P: ¿Cómo se obtiene el software cliente, la documentación y el acceso a la guía de integración para SafeNet Luna 7 HSM?

Después de registrarse en el servicio, se le proporcionará un identificador de cliente de Gemalto que le permitirá registrarse en el portal de soporte técnico de cliente de Gemalto. De este modo, tendrá acceso a todo el software y la documentación y podrá realizar solicitudes de soporte técnico directamente a Gemalto.

### <a name="q-if-there-is-a-security-vulnerability-found-and-a-patch-is-released-by-gemalto-who-is-responsible-for-upgradingpatching-osfirmware"></a>P: Si se encuentra una vulnerabilidad de seguridad y Gemalto publica una revisión, ¿quién es responsable de la actualización o revisión del sistema operativo o firmware?

Microsoft no puede conectarse a los módulos HSM asignados a los clientes. Los clientes deben actualizar y aplicar las revisiones en los módulos HSM.

### <a name="q-what-if-i-need-to-reboot-my-hsm"></a>P: ¿Qué ocurre si necesito reiniciar mi HSM?

HSM tiene una opción de reinicio de línea de comandos, pero se están experimentando problemas de falta de respuesta de forma intermitente. Por este motivo, para un reinicio seguro, se recomienda que presente una solicitud de soporte técnico a Microsoft para que el dispositivo se reinicie físicamente. 

## <a name="cryptography-and-standards"></a>Criptografía y estándares

### <a name="q-is-it-safe-to-store-encryption-keys-for-my-most-important-data-in-dedicated-hsm"></a>P: ¿Es seguro almacenar claves de cifrado para mis datos más importantes en Dedicated HSM?

Sí, Dedicated HSM aprovisiona dispositivos SafeNet Network HSM 7 que usan módulos HSM con certificación FIPS 140-2 nivel 3. 

### <a name="q-what-cryptographic-keys-and-algorithms-are-supported-by-dedicated-hsm"></a>P: ¿Qué claves criptográficas y algoritmos son compatibles con Dedicated HSM?

El servicio Dedicated HSM aprovisiona dispositivos SafeNet Network HSM 7. Estos dispositivos admiten una amplia gama de tipos de claves criptográficas y algoritmos, lo que incluye: compatibilidad total con Suite B

* Asimétricos:
  * RSA
  * DSA
  * Diffie-Hellman
  * Curva elíptica
  * Criptografía (ECDSA, ECDH, Ed25519, ECIES) con curvas con nombre, definidas por el usuario, Brainpool, KCDSA
* Simétricos:
  * AES-GCM
  * Triple DES
  * DES
  * ARIA, SEED
  * RC2
  * RC4
  * RC5
  * CAST
  * Hash/Message Digest/HMAC: SHA-1, SHA-2, SM3
  * Derivación de claves: Modo contador SP800-108
  * Ajuste de clave: SP800-38F
  * Generación de números aleatorios: DRBG aprobado por FIPS 140-2 (modo SP 800-90 CTR), compatible con BSI DRG.4

### <a name="q-is-dedicated-hsm-fips-140-2-level-3-validated"></a>P: ¿Dedicated HSM tiene certificación FIPS 140-2 nivel 3?

Sí. El servicio Dedicated HSM aprovisiona dispositivos SafeNet Network HSM 7 que usan módulos HSM con certificación FIPS 140-2 nivel 3.

### <a name="q-what-do-i-need-to-do-to-make-sure-i-operate-dedicated-hsm-in-fips-140-2-level-3-validated-mode"></a>P: ¿Qué hay que hacer para asegurarse de que Dedicated HSM funciona en modo de certificación FIPS 140-2 nivel 3?

El servicio Dedicated HSM aprovisiona dispositivos SafeNet Luna Network HSM 7. Estos dispositivos usan módulos HSM con certificación FIPS 140-2 nivel 3. La configuración predeterminada que se implementa, el sistema operativo y el firmware también tienen certificación FIPS. No es necesario realizar ninguna acción para el cumplimiento de FIPS 140-2 nivel 3.

### <a name="q-how-does-a-customer-ensure-that-when-an-hsm-is-deprovisioned-all-the-key-material-is-wiped-out"></a>P: ¿Cómo se asegura un cliente de que cuando se desaprovisiona un módulo HSM se borra el material de las claves?

Antes de solicitar el desaprovisionamiento, el cliente debe haber grabado con ceros el dispositivo HSM con las herramientas de cliente de HSM proporcionadas por Gemalto.

## <a name="performance-and-scale"></a>Rendimiento y escala

### <a name="q-how-many-cryptographic-operations-are-supported-per-second-with-dedicated-hsm"></a>P: ¿Cuántas operaciones criptográficas por segundo se admiten con Dedicated HSM?

Dedicated HSM aprovisiona dispositivos SafeNet Network HSM 7 (modelo A790). Este es un resumen de rendimiento máximo para algunas operaciones: 

* RSA-2048: 10 000 transacciones por segundo
* ECC P256: 20 000 transacciones por segundo
* AES-GCM: 17 000 transacciones por segundo

### <a name="q-how-many-partitions-can-be-created-in-dedicated-hsm"></a>P: ¿Cuántas particiones se pueden crear en Dedicated HSM?

El dispositivo SafeNet Luna HSM 7 modelo A790 usado incluye una licencia para 10 particiones en el coste del servicio. Tiene un límite de 100 particiones y, si se alcanza este número, es posible que se incurra en costes de licencia adicionales y que se requiera la instalación de un nuevo archivo de licencia en el dispositivo.

### <a name="q-how-many-keys-can-be-supported-in-dedicated-hsm"></a>P: ¿Cuántas claves se admiten en Dedicated HSM?

El número máximo de claves es una función de la memoria disponible. El dispositivo SafeNet Luna 7 modelo A790 en uso tiene 32 MB de memoria. Las cifras siguientes también son aplicables a los pares de claves si se usan claves asimétricas.

* RSA-2048: 19 000
* ECC-P256: 91 000

La capacidad variará según los atributos de clave específicos establecidos en la plantilla de generación de claves y el número de particiones.
