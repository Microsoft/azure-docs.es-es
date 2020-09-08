---
title: ¿Cómo administrar la configuración? - Custom Translator
titleSuffix: Azure Cognitive Services
description: Cómo administrar la configuración, crear áreas de trabajo, compartir áreas de trabajo y administrar la clave de suscripción en Custom Translator.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 488ca52abdd5f8d6cfd1f92a5f91861a17a77d19
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510513"
---
# <a name="how-to-manage-settings"></a>Cómo administrar la configuración

En la página de configuración de Traductor personalizado, puede compartir el área de trabajo, modificar la clave de suscripción al traductor y eliminar el área de trabajo.

Para acceder a la página de configuración, realice lo siguiente:

1. Inicie sesión en el portal de [Custom Translator](https://portal.customtranslator.azure.ai/).
2. En el portal de Custom Translator, haga clic en el icono de engranaje de la barra lateral.

    ![Vínculo de configuración](media/how-to/how-to-settings.png)

## <a name="associating-translator-subscription"></a>Asociación de la suscripción de Traductor

Debe tener una clave de suscripción de Traductor asociada con el área de trabajo para entrenar o implementar modelos.

Si no tiene una suscripción, siga los pasos a continuación:

1. Suscríbase para crear un recurso de Translator. Siga [Cómo suscribirse a Translator](https://docs.microsoft.com/azure/cognitive-services/translator/translator-how-to-signup) para suscribirse y adquirir una clave de traductor.
2. Apunte la clave para la suscripción a Translator. Tanto la clave Key1 como la clave Key2 son aceptables.
3. Navegue de vuelta al portal de Custom Translator.

## <a name="create-a-new-workspace"></a>Crear un área de trabajo

1. Haga clic en el botón "+ Crear área de trabajo" en la barra lateral "Traductor personalizado".

    ![Crear área de trabajo nueva](media/how-to/create-new-workspace.png)

2. En el cuadro de diálogo, escriba el nombre de la nueva área de trabajo.
3. Haga clic en "Siguiente".
4. Elija el tipo de suscripción.
5. Seleccione la región de la suscripción. La región debe coincidir con la región seleccionada cuando se creó la clave de recurso de Translator.
6. Escriba la clave de la suscripción a Translator y, a continuación, haga clic en el botón "Guardar".

    ![Cuadro de diálogo Crear área de trabajo nueva](media/how-to/create-new-workspace-dialog.png)


### <a name="modify-existing-key"></a>Modificación de la clave existente

1. Vaya a la página "Configuración" de su área de trabajo.
2. Haga clic en Cambiar clave.

    ![Cómo incorporar la clave de suscripción](media/how-to/how-to-add-subscription-key.png)

3. En el cuadro de diálogo, escriba la clave de la suscripción a Translator y, a continuación, haga clic en el botón "Guardar".

    ![Cómo incorporar la clave de suscripción](media/how-to/how-to-add-subscription-key-dialog.png)

## <a name="manage-your-workspace"></a>Administración del área de trabajo

Un área de trabajo es un área para crear y compilar el sistema de traducción personalizada. Un área de trabajo puede contener varios proyectos, modelos y documentos.

Si una parte distinta del trabajo debe compartirse con personas distintas, puede resultar útil crear varias áreas de trabajo.

## <a name="share-your-workspace"></a>Compartir el área de trabajo

En Custom Translator, puede compartir el área de trabajo con otros usuarios si una parte distinta del trabajo debe compartirse con personas diferentes.

1. Vaya a la página "Configuración" del área de trabajo.
2. Haga clic en el botón "Agregar contactos" de la sección "Configuración de uso compartido".

    ![Compartir área de trabajo](media/how-to/share-workspace.png)

3. En el cuadro de diálogo, escriba una lista separada por comas de direcciones de correo electrónico con las que quiera compartir esta área de trabajo. Asegúrese de compartirla con la dirección de correo electrónico que la persona usa para iniciar sesión en Custom Translator. A continuación, seleccione el nivel de permisos de uso compartido adecuado y haga clic en el botón "Guardar".

    ![Cuadro de diálogo Compartir área de trabajo](media/how-to/share-workspace-dialog.png)

4. Si el área de trabajo todavía tiene el nombre predeterminado "Mi área de trabajo", será necesario cambiarlo antes de compartir el área de trabajo.
5. Haga clic en "Guardar".

## <a name="sharing-permissions"></a>Permisos de uso compartido

1. **Lector:** un lector en el área de trabajo podrá ver toda la información del área de trabajo.

2. **Editor:** un editor en el área de trabajo podrá agregar documentos, entrenar modelos y eliminar documentos y proyectos. Puede agregar una clave de suscripción, pero no puede modificar con quién se comparte el área de trabajo, no puede eliminar el área de trabajo ni cambiarle el nombre.

3. **Propietario:** un propietario tiene permisos totales en el área de trabajo.

## <a name="change-sharing-permission"></a>Cambiar los permisos de uso compartido

Cuando se comparte un área de trabajo, en la sección "Configuración de uso compartido" se muestran todas las direcciones de correo electrónico con las que se comparte esta área de trabajo. Puede cambiar los permisos de uso compartido existentes para cada dirección de correo electrónico si tiene acceso de propietario al área de trabajo.

1. En la sección "Configuración de uso compartido", se muestra un menú desplegable para cada correo electrónico donde se muestra el nivel de permisos actual.

2. Haga clic en el menú desplegable y seleccione el nuevo nivel de permisos que quiere asignarle a dicho correo electrónico.

    ![Configuración de los permisos de uso compartido](media/how-to/sharing-permission-settings.png)

## <a name="pin-your-workspace"></a>Anclado del área de trabajo

La primera área de trabajo creada está anclada de forma predeterminada. Cada vez que inicie sesión, el área de trabajo anclada se muestra en la carga del sitio. Si ha creado muchas áreas de trabajo y desea configurar alguna de ellas como predeterminada al iniciar sesión, debe anclarla.

1. En la barra lateral, haga clic en el nombre del área de trabajo que desea anclar.
2. Vaya a la página "Configuración" de su área de trabajo.
3. Haga clic en el icono Anclar.

    ![Anclar área de trabajo](media/how-to/how-to-pin-workspace.png)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre [cómo administrar el área de trabajo y los proyectos](workspace-and-project.md).
