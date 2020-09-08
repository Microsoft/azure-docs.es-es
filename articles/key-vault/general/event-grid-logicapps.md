---
title: Envío de un correo electrónico cuando cambia el estado del secreto en Key Vault
description: Guía para usar Logic Apps en la respuesta a los cambios de secretos en Key Vault
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 11/11/2019
ms.author: mbaldwin
ms.openlocfilehash: c0121c42c5d498aa79109c874981b9de0f8f4b7c
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88588881"
---
# <a name="use-logic-apps-to-receive-email-about-status-changes-of-key-vault-secrets"></a>Uso de Logic Apps para recibir correo electrónico sobre los cambios de estado de los secretos en el almacén de claves

En esta guía aprenderá a responder a los eventos de Azure Key Vault que se reciben a través de [Azure Event Grid](../../event-grid/index.yml) mediante [Azure Logic Apps](../../logic-apps/index.yml). Al terminar, tendrá una aplicación lógica en Azure Logic Apps configurada para enviar una notificación por correo electrónico cada vez que se cree un secreto en Azure Key Vault.

Para información general sobre la integración de Azure Key Vault y Azure Event Grid, consulte [Supervisión de Key Vault con Azure Event Grid (versión preliminar)](event-grid-overview.md).

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de correo electrónico de cualquier proveedor de correo electrónico que sea compatible con Azure Logic Apps (como, por ejemplo, Office 365 Outlook). Esta cuenta de correo electrónico se usa para enviar las notificaciones de eventos. Para obtener una lista completa de los conectores compatibles de Logic Apps, consulte la [información general sobre los conectores](/connectors).
- Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Un almacén de claves en la suscripción de Azure. Puede crear rápidamente un almacén de claves si sigue los pasos descritos en [Establecimiento y recuperación de un secreto desde Azure Key Vault mediante la CLI de Azure](../secrets/quick-create-cli.md).
- Event Grid registrado como proveedor de recursos; consulte los [registros de los proveedores de recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-providers-and-types)

## <a name="create-a-logic-app-via-event-grid"></a>Creación de una aplicación lógica mediante Event Grid

En primer lugar, cree una aplicación lógica con el controlador de Event Grid y suscríbase a los eventos "SecretNewVersionCreated" de Azure Key Vault.

Para crear una suscripción de Azure Event Grid, siga estos pasos:

1. En Azure Portal, vaya a su almacén de claves, seleccione **Eventos > Tareas iniciales** y haga clic en **Aplicaciones lógicas**.

    
    ![Key Vault: página de eventos](../media/eventgrid-logicapps-kvsubs.png)

1. En **Diseñador de aplicaciones lógicas**, valide la conexión y haga clic en **Continuar**. 
 
    ![Diseñador de aplicaciones lógicas: conexión](../media/eventgrid-logicappdesigner1.png)

1. En la pantalla **Cuando se produce un evento de recursos**, lleve a cabo las siguientes acciones:
    - Deje **Suscripción** y **Nombre de recurso** con los valores predeterminados.
    - Seleccione **Microsoft.KeyVault.vaults** en **Tipo de recurso**.
    - Seleccione **Microsoft.KeyVault.SecretNewVersionCreated** en **Event Type Item - 1** (Elemento de tipo de evento - 1).

    ![Diseñador de aplicaciones lógicas: controlador de eventos](../media/eventgrid-logicappdesigner2.png)

1. Seleccione **+ Nuevo paso**. Se abrirá una ventana para elegir una acción.
1. Busque **Correo electrónico**. En función de su proveedor de correo electrónico, busque y seleccione el conector correspondiente. Este tutorial usa **Office 365 Outlook**. Los pasos para otros proveedores de correo electrónico son similares.
1. Seleccione la acción **Enviar correo electrónico (V2)**.

   ![Diseñador de aplicaciones lógicas: agregar correo electrónico](../media/eventgrid-logicappdesigner3.png)

1. Cree la plantilla de correo electrónico:
    - **Para**: escriba la dirección de correo electrónico en la que desea recibir los correos electrónicos de notificación. En este tutorial, use una cuenta de correo electrónico a la que pueda acceder para realizar pruebas.
    - **Asunto** y **Cuerpo**: escriba el texto del correo electrónico. Seleccione propiedades JSON en la herramienta de selección para incluir contenido dinámico en función de los datos del evento. Puede recuperar los datos del evento mediante `@{triggerBody()?['Data']}`.

    La plantilla de correo electrónico puede tener un aspecto similar al de este ejemplo.

    ![Diseñador de aplicaciones lógicas: agregar correo electrónico](../media/eventgrid-logicappdesigner4.png)

8. Haga clic en **Guardar como**.
9. Escriba un **nombre** para la nueva aplicación lógica y haga clic en **Crear**.
    
    ![Diseñador de aplicaciones lógicas: agregar correo electrónico](../media/eventgrid-logicappdesigner5.png)

## <a name="test-and-verify"></a>Prueba y comprobación

1.  Vaya al almacén de claves en Azure Portal y seleccione **Eventos > Suscripciones a eventos**.  Compruebe que se crea una nueva suscripción.
    
    ![Diseñador de aplicaciones lógicas: agregar correo electrónico](../media/eventgrid-logicapps-kvnewsubs.png)

1.  Vaya al almacén de claves, seleccione **Secretos** y, después, **+ Generate/Import** (+ Generar/importar). Cree un secreto nuevo para las pruebas, asigne un nombre a la clave y mantenga el resto de los parámetros con su configuración predeterminada.

    ![Key Vault: crear secreto](../media/eventgrid-logicapps-kv-create-secret.png)

1. En la pantalla **Crear un secreto**, indique cualquier nombre, cualquier valor y seleccione **Crear**.

Cuando se cree el secreto, se recibirá un correo electrónico en las direcciones configuradas.

## <a name="next-steps"></a>Pasos siguientes

- Introducción: [Supervisión de Key Vault con Azure Event Grid (versión preliminar)](event-grid-overview.md)
- Procedimientos: [Enrutar las notificaciones del almacén de claves a Azure Automation](event-grid-tutorial.md).
- [Esquema de eventos de Azure Event Grid para Azure Key Vault (versión preliminar)](../../event-grid/event-schema-key-vault.md)
- Más información sobre [Azure Event Grid](../../event-grid/index.yml).
- Más información acerca de la [característica Logic Apps de Azure App Service](../../logic-apps/index.yml).
