---
title: Flujo de trabajo de Azure Attestation
description: El flujo de trabajo de Azure Attestation.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: 240b27f897d8e7a34026701cf7fdc844eb9d4086
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89237105"
---
# <a name="workflow"></a>Flujo de trabajo

Microsoft Azure Attestation recibe la evidencia de los enclaves y la evalúa con respecto a las directivas de base de referencia y configurables de seguridad de Azure. Tras una verificación correcta, Azure Attestation genera un token de atestación para confirmar la confiabilidad del enclave.

En el flujo de trabajo de Azure Attestation participan los siguientes actores:

- **Usuario de confianza**: componente que se basa en Azure Attestation para comprobar la validez del enclave. 
- **Cliente**: componente que recopila información de un enclave y envía solicitudes a Azure Attestation. 
- **Azure Attestation**: componente que acepta la evidencia de enclave del cliente, la valida y devuelve el token de atestación al cliente.


## <a name="enclave-validation-work-flow"></a>Flujo de trabajo de validación del enclave

Estos son los pasos generales de un flujo de trabajo típico de atestación de un enclave SGX (mediante Azure Attestation):

1. El cliente recopila la evidencia de un enclave. Esta evidencia consiste en información sobre el entorno del enclave y la biblioteca cliente que se ejecuta en el enclave.
1. El cliente tiene un URI que hace referencia a una instancia de Azure Attestation. El cliente se autentica en Azure AD y obtiene un token de acceso.
1. El cliente envía la evidencia a Azure Attestation, junto con el token de acceso. La información exacta que se envía al proveedor depende del tipo de enclave.
1. Azure Attestation valida la información enviada y la evalúa con respecto a una directiva configurada. Si la comprobación finaliza correctamente, Azure Attestation emite un token de atestación y lo devuelve al cliente. Si se produce un error en este paso, Azure Attestation notifica este error al cliente. 
1. El cliente envía el token de atestación al usuario de confianza. El usuario de confianza llama al punto de conexión de metadatos de clave pública de Azure Attestation para recuperar certificados de firma. Después, el usuario de confianza comprueba la firma del token de atestación y garantiza la confiabilidad del enclave. 

![Flujo de validación del enclave](./media/validation-flow.png)


## <a name="next-steps"></a>Pasos siguientes
- [Creación y firma de una directiva de atestación](author-sign-policy.md)
- [Configuración de Azure Attestation con PowerShell](quickstart-powershell.md)
