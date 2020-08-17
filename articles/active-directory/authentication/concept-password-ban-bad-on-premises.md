---
title: 'Protección con contraseña de Azure AD: Azure Active Directory'
description: Prohibición de contraseñas no seguras en entornos de Active Directory Domain Services locales mediante Protección con contraseña de Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/17/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: feb7c4a4417d64e039793bd96141c965f6437414
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87050928"
---
# <a name="enforce-on-premises-azure-ad-password-protection-for-active-directory-domain-services"></a>Aplicación de Protección con contraseña de Azure AD local en Active Directory Domain Services

La característica Protección con contraseña de Azure AD detecta y bloquea las contraseñas no seguras conocidas y sus variantes, y también puede bloquear otros términos poco seguros específicos de una organización. La implementación local de Protección con contraseña de Azure AD utiliza las mismas listas de contraseñas prohibidas personalizadas y globales que se almacenan en Azure AD y realiza las mismas comprobaciones de los cambios de contraseña locales que hace Azure AD de los cambios basados en la nube. Estas comprobaciones se realizan durante los cambios de contraseña y los eventos de restablecimiento de contraseña en los controladores de dominio de Active Directory Domain Services (AD DS) local.

## <a name="design-principles"></a>Principios de diseño

Protección con contraseña de Azure AD se ha diseñado teniendo en cuenta estos principios:

* Los controladores de dominio (DC) nunca tienen que comunicarse directamente con Internet.
* No hay ningún puerto de red nuevo abierto en los controladores de dominio.
* No se requieren cambios en el esquema de AD DS. El software usa los objetos de esquema *container* y *serviceConnectionPoint* de AD DS existentes.
* No hay ningún requisito mínimo de nivel funcional del bosque (DFL/FFL) ni de dominio de AD DS.
* El software no crea ni necesita ninguna cuenta en los dominios de AD DS que protege.
* Las contraseñas de usuario no cifradas nunca dejan el controlador de dominio, ya sea durante las operaciones de validación de contraseña o en cualquier otro momento.
* El software no depende de otras características de Azure AD. Por ejemplo, la sincronización de hash de contraseñas (PHS) de Azure AD no está relacionada con Protección con contraseña de Azure AD ni es necesaria con funcionalidad.
* Se admite la implementación incremental, sin embargo, la directiva de contraseñas se aplicará solamente donde se instale el agente de controlador de dominio.

## <a name="incremental-deployment"></a>Implementación incremental

Protección con contraseña de Azure AD admite la implementación incremental en los controladores de dominio de un dominio de AD DS. Es importante comprender lo que esto significa realmente y cuáles son los inconvenientes.

El software agente de DC de Protección con contraseña de Azure AD solo puede validar las contraseñas cuando está instalado en un controlador de dominio, y solo en el caso de los cambios de contraseña que se envían a ese controlador de dominio. No es posible controlar qué controladores de dominio eligen las máquinas cliente de Windows para procesar los cambios de contraseña de los usuarios. Para garantizar el comportamiento coherente y el cumplimiento de la seguridad universal de Protección con contraseña de Azure AD, el software agente de DC debe estar instalado en todos los controladores de dominio de un dominio.

Muchas organizaciones quieren probar con precaución Protección con contraseña de Azure AD en un subconjunto de sus controladores de dominio antes de realizar una implementación completa. Este escenario es posible gracias a que Protección con contraseña de Azure AD admite la implementación parcial. El software de agente de DC de un controlador de dominio dado valida de forma activa las contraseñas incluso cuando otros controladores de dominio del dominio no tienen instalado dicho software. Las implementaciones parciales de este tipo no son seguras y solo se recomiendan con fines de prueba.

## <a name="architectural-diagram"></a>Diagrama de arquitectura

Es importante comprender el diseño y los conceptos funcionales subyacentes antes de implementar Protección con contraseña de Azure AD en un entorno de AD DS local. En el diagrama siguiente se muestra cómo los componentes de Protección con contraseña de Azure AD funcionan conjuntamente:

![Funcionamiento conjunto de los componentes de la protección con contraseña de Azure AD](./media/concept-password-ban-bad-on-premises/azure-ad-password-protection.png)

* El servicio de proxy de Protección con contraseña de Azure AD se ejecuta en cualquier máquina unida a un dominio del bosque de AD DS actual. La finalidad principal de este servicio es la de reenviar las solicitudes de descarga de la directiva de contraseñas de los controladores de dominio a Azure AD y, luego, devolver las respuestas de Azure AD al controlador de dominio.
* La DLL del agente de controlador de dominio del filtro de contraseña recibe las solicitudes de validación de contraseña de usuario desde el sistema operativo. El filtro las reenvía al servicio DC Agent que se ejecuta localmente en el controlador de dominio.
* El servicio DC Agent de Protección con contraseña de Azure AD recibe solicitudes de validación de contraseña del archivo DLL del filtro de contraseñas de DC Agent. Este servicio las procesa mediante la directiva de contraseñas (disponible localmente) actual y devuelve el resultado de *correcta* o *error*.

## <a name="how-azure-ad-password-protection-works"></a>Funcionamiento de Protección con contraseña de Azure AD

Los componentes de Protección con contraseña de Azure AD local funcionan de la siguiente manera:

1. Cada instancia del servicio de proxy de Protección con contraseña de Azure AD se anuncia ella misma a los controladores de dominio del bosque mediante la creación de un objeto *serviceConnectionPoint* en Active Directory.

    Cada servicio DC Agent de Protección con contraseña de Azure AD crea también un objeto *serviceConnectionPoint* en Active Directory. Este objeto se utiliza principalmente para los informes y diagnósticos.

1. El servicio DC Agent es el responsable de iniciar la descarga de una directiva de contraseña desde Azure AD. El primer paso es localizar un servicio Password Protection Proxy de Azure AD consultando en el bosque los objetos *serviceConnectionPoint* del proxy.

1. Cuando se encuentra un servicio de proxy disponible, el agente de controlador de dominio envía una solicitud de descarga de la directiva de contraseña al servicio de proxy. A su vez, el servicio de proxy envía la solicitud a Azure AD para, posteriormente, devolver la respuesta al servicio DC Agent.

1. Una vez que el servicio DC Agent recibe una nueva directiva de contraseña de Azure AD, la almacena en una carpeta dedicada en la raíz de su recurso compartido de carpeta *sysvol* de dominio. El servicio DC Agent también supervisa esta carpeta en caso de que las directivas más recientes se repliquen desde otros servicios DC Agent en el dominio.

1. El servicio siempre solicita una nueva directiva al inicio del servicio. Una vez iniciado el servicio DC Agent, comprueba la antigüedad de la directiva actual disponible de forma local cada hora. Si la directiva es anterior a una hora, el agente de controlador de dominio solicita una nueva directiva de Azure AD a través del servicio de proxy, como se describió anteriormente. Si la directiva actual no es anterior a una hora, el agente de controlador de dominio sigue usando esa directiva.

1. Cuando un controlador de dominio recibe los eventos de cambio de contraseña, la directiva almacenada en caché se usa para determinar si la nueva contraseña se acepta o se rechaza.

### <a name="key-considerations-and-features"></a>Principales consideraciones y características

* Cada vez que se descarga una directiva de contraseñas de Protección con contraseña de Azure AD, esa directiva es específica de un inquilino. En otras palabras, las directivas de contraseñas son siempre una combinación de la lista global de contraseñas prohibidas de Microsoft y la lista de contraseñas prohibidas por inquilino personalizada.
* El agente de controlador de dominio se comunica con el servicio de proxy por RPC a través de TCP. El servicio de proxy escucha estas llamadas en un puerto RPC dinámico o estático, según esté configurado.
* El agente de controlador de dominio nunca escucha en un puerto de red disponible.
* El servicio de proxy nunca llama al servicio DC Agent.
* El servicio de proxy es sin estado. Nunca almacena en caché las directivas ni cualquier otro estado descargado de Azure.
* El servicio DC Agent siempre usa la directiva de contraseña localmente disponible más reciente para evaluar una contraseña de usuario. Si no hay ninguna directiva de contraseña disponible en el controlador de dominio local, la contraseña se acepta automáticamente. Cuando esto ocurre, se registra un mensaje de evento para advertir al administrador.
* Protección con contraseña de Azure AD no es un motor de aplicación de directivas en tiempo real. Puede haber un retraso desde que se realiza un cambio en la configuración de la directiva de contraseñas en Azure AD hasta que el cambio llega y se aplica a todos los controladores de dominio.
* Protección con contraseña de Azure AD funciona como un complemento a las directivas de contraseña de AD DS existentes, no las sustituye. Esto incluye las demás DLL de filtro de contraseña de terceros que puedan estar instaladas. En AD DS, es necesario siempre que todos los componentes de validación de contraseñas estén de acuerdo antes de aceptar una contraseña.

## <a name="forest--tenant-binding-for-azure-ad-password-protection"></a>Enlace bosque\inquilino para Protección con contraseña de Azure AD

Para implementar Protección con contraseña de Azure AD en un bosque de AD DS, es necesario registrar dicho bosque con Azure AD. Cada servicio de proxy que se implementa también debe registrarse con Azure AD. Ambos registros de bosque y de proxy están asociados a un determinado inquilino de Azure AD que se identifica implícitamente con las credenciales utilizadas durante el registro.

El bosque de AD DS y todos los servicios de proxy implementados dentro de un bosque deben registrarse con el mismo inquilino. No se permite registrar un bosque de AD DS ni ningún servicio de proxy de ese bosque con diferentes inquilinos de Azure AD. Entre los síntomas de esa implementación mal configurada se incluye la imposibilidad de descargar las directivas de contraseña.

## <a name="download"></a>Descargar

Los dos instaladores de agente necesarios para Protección con contraseña de Azure AD se pueden descargar del [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=57071).

## <a name="next-steps"></a>Pasos siguientes

Para empezar a usar Protección con contraseña de Azure AD local, realice el procedimiento siguiente:

> [!div class="nextstepaction"]
> [Implementación de la protección con contraseña de Azure AD local](howto-password-ban-bad-on-premises-deploy.md)
