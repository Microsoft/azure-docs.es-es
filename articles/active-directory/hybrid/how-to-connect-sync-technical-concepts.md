---
title: 'Sincronización de Azure AD Connect: Conceptos técnicos | Microsoft Docs'
description: Explica los conceptos técnicos de la sincronización de Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 731cfeb3-beaf-4d02-aef4-b02a8f99fd11
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/15/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 01e53b30a4c27296e30e031ffb771697afa8e1e9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87019682"
---
# <a name="azure-ad-connect-sync-technical-concepts"></a>Sincronización de Azure AD Connect: Conceptos técnicos
Este artículo es un resumen del tema [Descripción de la arquitectura](how-to-connect-sync-technical-concepts.md).

Sincronización de Azure AD Connect se basa en una sólida plataforma de sincronización de metadirectorios.
Las secciones siguientes presentan los conceptos de sincronización de metadirectorio.
Al compilar sobre MIIS (Microsoft Identity Integration Server), ILM (Identity Lifecycle Manager) y FIM (Forefront Identity Manager), los servicios de Azure Active Directory Sync proporcionan la plataforma de próxima generación para conectarse a orígenes de datos, sincronizar datos entre orígenes de datos, y aprovisionar y desaprovisionar identidades.

![Conceptos técnicos](./media/how-to-connect-sync-technical-concepts/scenario.png)

Las secciones siguientes proporcionan más detalles sobre los siguientes aspectos del Servicio de sincronización de FIM:

* Conector
* Flujo del atributo
* Espacio del conector
* Metaverso
* Aprovisionamiento

## <a name="connector"></a>Conector
Los módulos de código que se usan para comunicarse con un directorio conectado se denominan conectores (conocidos anteriormente como agentes de administración o MA).

Se instalan en el equipo donde se ejecuta la sincronización de Azure AD Connect. Los conectores proporcionan la capacidad sin agente de inversión mediante protocolos de sistema remoto en lugar de depender de la implementación de agentes especializados. Esto significa menor riesgo y el tiempo de implementación, especialmente al tratar con sistemas y aplicaciones críticas.

En la figura anterior, el conector es sinónimo del espacio del conector, pero abarca toda la comunicación con el sistema externo.

El conector es responsable de la funcionalidad de importación y exportación en el sistema y libera a los desarrolladores de la necesidad de entender cómo conectarse a cada sistema de forma nativa al usar el aprovisionamiento declarativo para personalizar las transformaciones de datos.

Las importaciones y exportaciones solo se producen cuando se programan, lo que permite un mejor aislamiento de los cambios que ocurren en el sistema, dado que estos no se propagan automáticamente al origen de datos conectado. Además, los desarrolladores pueden crear también sus propios conectores para conectarse a prácticamente cualquier origen de datos.

## <a name="attribute-flow"></a>Flujo del atributo
El metaverso es la vista consolidada de todas las identidades combinadas de otros espacios de conector. En la ilustración anterior, el flujo de atributos se representa mediante líneas con puntas de flecha para el flujo entrante y saliente. El flujo de atributos es el proceso de copiar o transformar los datos entre un sistema y otro y todos los flujos de atributos (entrantes o salientes).

El flujo de atributos se produce entre el espacio del conector y el metaverso bidireccionalmente cuando las operaciones de sincronización (completa o delta) están programadas para ejecutarse.

El flujo de atributos solo se produce cuando se ejecutan estas sincronizaciones. Los flujos de atributos se definen en las reglas de sincronización. Pueden ser de entrada (ISR en la imagen anterior) o salida (OSR en la imagen anterior).

## <a name="connected-system"></a>Sistema conectado
Un sistema conectado (conocido también como directorio conectado) se refiere al sistema remoto al que se ha conectado Azure AD Connect Sync y en el que se leen y escriben datos de identidad.

## <a name="connector-space"></a>Espacio del conector
Cada origen de datos conectado se representa como un subconjunto filtrado de los objetos y atributos en el espacio del conector.
Esto permite que el servicio de sincronización funcione localmente sin necesidad de ponerse en contacto con el sistema remoto al sincronizar los objetos y restringe la interacción solo a las importaciones y exportaciones.

Cuando el origen de datos y el conector tienen la capacidad de proporcionar una lista de cambios (una importación delta), la eficiencia operativa aumenta significativamente porque solo se intercambian los cambios desde el último ciclo de sondeo. El espacio del conector aísla el origen de datos conectado de los cambios que se propagan automáticamente solicitando la importación y exportación de la programación del conector. Esta seguridad adicional le proporciona tranquilidad durante las pruebas, la obtención de vista previa o la confirmación de la próxima actualización.

## <a name="metaverse"></a>Metaverso
El metaverso es la vista consolidada de todas las identidades combinadas de otros espacios de conector.

Puesto que las identidades están vinculadas entra sí y la autoridad está asignada para varios atributos a través de asignaciones de flujo de importación, el objeto de metaverso central comienza a agregar información desde varios sistemas. A partir de este flujo de atributos de objetos, las asignaciones proporcionan información a los sistemas de salida.

Los objetos se crean cuando un sistema autorizado los proyecta en el metaverso. En cuanto se quitan todas las conexiones, se elimina el objeto de metaverso.

Los objetos del metaverso no se pueden editar directamente. Todos los datos del objeto se deben aportar mediante el flujo de atributos. El metaverso mantiene conectores persistentes con cada espacio de conector. Estos conectores no requieren una nueva evaluación en cada ejecución de la sincronización. Esto significa que Azure AD Connect Sync no tiene que buscar sistemáticamente el objeto remoto coincidente. De esta forma se evita la necesidad de costosos agentes para impedir cambios en los atributos que normalmente serían responsable de la correlación de los objetos.

Durante la detección de nuevos orígenes de datos que pueden tener objetos ya existentes que deban administrarse, la sincronización de Azure AD Connect usa un proceso llamado regla de combinación para evaluar los posibles candidatos con los que se va a establecer un vínculo.
Una vez establecido el vínculo, esta evaluación no se repite y el flujo de atributos normal puede producirse entre el origen de datos conectado remoto y el metaverso.

## <a name="provisioning"></a>Aprovisionamiento
Cuando un origen autoritativo proyecta un nuevo objeto en el metaverso, puede crearse un nuevo objeto de espacio de conector en otro conector que representa un origen de datos conectado de nivel inferior.

Esto establece intrínsecamente un vínculo y el flujo de atributos puede continuar de manera bidireccional.

Siempre que una regla se determina que tiene que crearse un nuevo objeto de espacio de conector, se le denomina aprovisionamiento. Sin embargo, puesto que esta operación solo se produce dentro del espacio del conector, no se mantienen en el origen de datos conectado hasta que se realiza una exportación.

## <a name="additional-resources"></a>Recursos adicionales
* [Sincronización de Azure AD Connect: personalización de las opciones de sincronización](how-to-connect-sync-whatis.md)
* [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md)

<!--Image references-->
[1]: ./media/active-directory-aadsync-technical-concepts/ic750598.png
