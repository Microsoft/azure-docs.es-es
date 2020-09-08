---
title: Movimiento de un almacén a una grupo de recursos diferente en Azure Key Vault | Microsoft Docs
description: Guía para mover un almacén de claves a un grupo de recursos diferente.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 04/29/2020
ms.author: sudbalas
Customer intent: As a key vault administrator, I want to move my vault to another resource group.
ms.openlocfilehash: fe8051d551077666c06ac033f22303fd643ac602
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88585740"
---
# <a name="moving-an-azure-key-vault-across-resource-groups"></a>Movimiento de un almacén de Azure Key Vault entre grupos de recursos

## <a name="overview"></a>Información general

El movimiento de un almacén de claves entre grupos de recursos es una característica que es compatible con el almacén de claves. El movimiento de un almacén de claves entre grupos de recursos no afectará a las configuraciones de directiva de acceso o firewall del almacén de claves. Las aplicaciones conectadas y las entidades de servicio deben seguir funcionando según lo previsto.

## <a name="design-considerations"></a>Consideraciones de diseño

Su organización puede haber implementado Azure Policy con cumplimiento o exclusión en el nivel de grupo de recursos. Puede haber un conjunto diferente de asignaciones de directiva en el grupo de recursos en el que se encuentra el almacén de claves y el grupo de recursos al que se va a trasladar el almacén de claves. Un conflicto en los requisitos de la directiva tiene la posibilidad de interrumpir las aplicaciones.

### <a name="example"></a>Ejemplo

Tiene una aplicación conectada al almacén de claves que crea certificados que son válidos durante dos años. El grupo de recursos al que está intentando trasladar el almacén de claves tiene una asignación de directiva que bloquea la creación de certificados que son válidos durante más de un año. Después de trasladar el almacén de claves al nuevo grupo de recursos, una asignación de directiva de Azure bloqueará la operación que crea un certificado válido durante dos años.

### <a name="solution"></a>Solución

Asegúrese de ir a la página Azure Policy del Azure Portal y examine las asignaciones de directiva de su grupo de recursos actual, así como del grupo de recursos al que está pasando, y asegúrese de que no haya ninguna discrepancia.

## <a name="procedure"></a>Procedimiento

1. Iniciar sesión en Azure Portal
2. Vaya a almacén de claves.
3. Haga clic en la pestaña "Introducción".
4. Seleccione el botón "Mover".
5. Seleccione "Mover a otro grupo de recursos" en las opciones de lista desplegable.
6. Seleccione el grupo de recursos a donde quiere mover el almacén de claves
7. Confirme la advertencia relacionada con el traslado de recursos.
8. Seleccione Aceptar.

Key Vault evaluará ahora la validez del movimiento de recursos y le avisará de cualquier error. Si no se encuentra ningún error, se completará el traslado de recursos. 
