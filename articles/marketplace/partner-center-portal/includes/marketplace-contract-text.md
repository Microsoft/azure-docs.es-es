---
title: archivo de inclusión
description: archivo de inclusión de texto para el contrato estándar de Marketplace comercial de Microsoft
documentationcenter: partner-center-commercial-marketplace
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: include
ms.date: 01/22/2020
ms.custom: include file
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: d899db8b5f6f3b7c0ccef4e44c814f1e126c626a
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326685"
---
Microsoft proporciona una plantilla de contrato estándar para Marketplace comercial.

- **¿Quiere usar el contrato estándar para Marketplace comercial de Microsoft?**

Para simplificar el proceso de adquisición para los clientes y reducir la complejidad legal para los proveedores de software, Microsoft ofrece un contrato estándar para el Marketplace comercial de Microsoft con el fin de ayudar a facilitar las transacciones en Marketplace. En lugar de crear términos y condiciones personalizados, los anunciantes del Marketplace comercial pueden ofrecer su software en virtud del Contrato estándar, que los clientes solo tienen que revisar y aceptar una vez. El contrato estándar se puede encontrar aquí: https://go.microsoft.com/fwlink/?linkid=2041178.

Puede optar por usar el contrato estándar en lugar de proporcionar sus propios términos y condiciones personalizados activando la casilla "¿Quiere usar el contrato estándar para Marketplace comercial de Microsoft?".

![Uso de la casilla Contrato estándar](./media/use-standard-contract.png)

> [!NOTE]
> Después de publicar una oferta con el contrato estándar para Marketplace comercial de Microsoft, no puede usar sus propios términos y condiciones personalizados. Solo puede elegir una de las dos opciones. Puede ofrecer su solución con el Contrato estándar **o** con sus propios términos y condiciones. Si quiere modificar los términos del contrato estándar, puede hacerlo a través de las modificaciones del contrato estándar.

**Modificaciones del Contrato estándar**

Las modificaciones del contrato estándar permiten a los anunciantes seleccionar los términos del contrato estándar por motivos de simplicidad y personalizarlos para su producto o empresa. Los clientes solo necesitan revisar las modificaciones del contrato, si ya han revisado y aceptado el Contrato estándar de Microsoft.

Hay dos tipos de modificaciones para los anunciantes de Marketplace comercial:

- Modificaciones universales: Estas modificaciones se aplican universalmente al Contrato estándar para todos los clientes. Las modificaciones universales se muestran a cada cliente de la oferta en el flujo de compra. Los clientes deben aceptar los términos del Contrato estándar y la modificación antes de que puedan usar su oferta.
- Modificaciones personalizadas: Estas son modificaciones especiales al Contrato estándar que están dirigidas a cliente únicamente a clientes específicos a través de los Id. de inquilino de Azure. Los anunciantes pueden elegir al inquilino al que desean dirigirse. Solo los clientes del inquilino recibirán los términos con modificaciones personalizadas en el flujo de compra de la oferta.  Los clientes deben aceptar los términos del Contrato estándar y las modificaciones antes de que puedan usar su oferta.

>[!NOTE]
> Estos dos tipos de modificaciones se superponen entre sí. Los clientes específicos con modificaciones personalizadas también obtendrán la modificación universal al Contrato estándar durante la compra.

**Términos de modificación universales del Contrato estándar de Marketplace comercial de Microsoft**: Escriba los términos de modificación universal en este cuadro. Puede proporcionar una única modificación universal por oferta. Puede escribir un número ilimitado de caracteres en este cuadro. Estos términos se muestran a los clientes en AppSource, Azure Marketplace o Azure Portal durante el flujo de detección y compra.

**Términos de modificación personalizados del Contrato estándar de Marketplace comercial de Microsoft**: Comience por seleccionar **Add custom amendment terms** (Agregar términos de modificación personalizados). Puede proporcionar hasta 10 términos de modificación personalizados por oferta.

- **Términos de modificación personalizados**: Escriba los términos de modificación personalizados en el cuadro Términos de modificación personalizados. Puede escribir un número ilimitado de caracteres en este cuadro. Solo los clientes de los id. de inquilino que se especifiquen para estos términos personalizados recibirán los términos de modificación personalizados en el flujo de compra de la oferta en Azure Portal.  
- **Tenant IDs** (Identificadores de inquilino) (opcional): Cada modificación personalizada puede tener como destino hasta 20 identificadores de inquilino. Si agrega una modificación personalizada, debe proporcionar al menos un identificador de inquilino. El identificador de inquilino identifica al cliente en Azure. Puede pedir a su cliente este identificador y encontrarlo si navega a portal.azure.com > Azure Active Directory > Propiedades. El valor del identificador de directorio es el identificador de inquilino (por ejemplo, 50c464d3-4930-494c-963c-1e951d15360e). También puede buscar el identificador de inquilino de la organización de su cliente mediante su dirección URL de nombre de dominio en [What is my Microsoft Azure and Office 365 tenant ID?](https://www.whatismytenantid.com) (¿Cuál es mi identificador de inquilino de Microsoft Azure y Office 365?).
- **Descripción** (opcional): Opcionalmente, proporcione una descripción detallada del identificador de inquilino que le ayude a identificar al cliente al que está destinada la modificación.

**Términos y condiciones**

Si quiere proporcionar sus propios términos y condiciones personalizados, puede especificarlos en el campo Términos y condiciones. Puede escribir hasta 10 000 caracteres de texto en este campo. Si los términos y condiciones requieren una descripción más larga, escriba en este campo un vínculo de dirección URL donde se pueden encontrar los términos y condiciones. A los clientes les aparecerá como un vínculo activo.

Los clientes deben aceptar estos términos para poder probar la oferta.

Seleccione **Guardar borrador** antes de continuar.
