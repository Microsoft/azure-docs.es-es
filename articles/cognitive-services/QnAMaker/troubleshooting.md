---
title: 'Solución de problemas: QnA Maker'
description: La lista de las preguntas frecuentes seleccionadas sobre QnA Maker le ayudará a adoptar este servicio de forma más rápida y a obtener mejores resultados.
ms.topic: troubleshooting
ms.date: 03/25/2020
ms.openlocfilehash: 50823e29eb5c276c00a1c8f0e3227ec67999a703
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502586"
---
# <a name="troubleshooting-for-qna-maker"></a>Solución de problemas para QnA Maker

La lista de las preguntas frecuentes seleccionadas sobre QnA Maker le ayudará a adoptar este servicio de forma más rápida y a obtener mejores resultados.

<a name="how-to-get-the-qnamaker-service-hostname"></a>

## <a name="manage-predictions"></a>Administración de predicciones

<details>
<summary><b>¿Cómo puedo mejorar el rendimiento de las predicciones de consulta?</b></summary>

**Respuesta**: Los problemas de rendimiento indican que es necesario realizar un escalado vertical de sus instancias de App Service y Cognitive Search. Considere la posibilidad de agregar una réplica a su instancia de Cognitive Search para mejorar el rendimiento.

Obtenga más información sobre los [planes de tarifa](Concepts/azure-resources.md).
</details>

<details>
<summary><b>Cómo obtener el punto de conexión de servicio de QnAMaker</b></summary>

**Respuesta**: El punto de conexión de servicio de QnAMaker es útil para fines de depuración cuando se ponga en contacto con el soporte técnico de QnAMaker o UserVoice. El punto de conexión es una dirección URL en este formato: `https://your-resource-name.azurewebsites.net`.

1. Vaya a su servicio QnAMaker (grupo de recursos) en [Azure Portal](https://portal.azure.com)

    ![Grupo de recursos de Azure para QnAMaker en Azure Portal](./media/qnamaker-how-to-troubleshoot/qnamaker-azure-resourcegroup.png)

1. Seleccione la instancia de App Service asociada al recurso de QnA Maker. Normalmente, los nombres son los mismos.

     ![Selección del App Service QnAMaker](./media/qnamaker-how-to-troubleshoot/qnamaker-azure-appservice.png)

1. La dirección URL del punto de conexión está disponible en la sección Información general

    ![Punto de conexión de QnAMaker](./media/qnamaker-how-to-troubleshoot/qnamaker-azure-gethostname.png)

</details>

## <a name="manage-the-knowledge-base"></a>Administrar la base de conocimiento

<details>
<summary><b>Eliminé por accidente una parte de QnA Maker, ¿qué debo hacer?</b></summary>

**Respuesta**: No elimine ninguno de los servicios de Azure creados junto con el recurso de QnA Maker, como Search o Web App. Estos son necesarios para que QnA Maker funcione; si elimina uno, QnA Maker dejará de funcionar correctamente.

Todas las eliminaciones son permanentes, incluidos los pares de preguntas y respuestas, archivos, direcciones URL, preguntas y respuestas personalizadas, bases de conocimiento o recursos de Azure. Asegúrese de exportar la base de conocimiento desde la página **Settings** (Configuración) antes de eliminar cualquier de sus partes.

</details>

<details>
<summary><b>¿Por qué mis direcciones URL o archivos no extraen pares de preguntas y respuestas?</b></summary>

**Respuesta**: Es posible que QnA Maker no pueda extraer automáticamente algún contenido de preguntas y respuestas (QnA) de las direcciones URL de P+F válidas. En tales casos, puede pegar el contenido de QnA en un archivo .txt y ver si la herramienta puede ingerirlo. Como alternativa, puede redactar contenido y agregarlo a la knowledge base a través del [portal de QnA Maker](https://qnamaker.ai).

</details>

<details>
<summary><b>¿Cuál es el tamaño máximo para crear una base de conocimiento?</b></summary>

**Respuesta**: El tamaño de la knowledge base depende de la SKU de Azure Search que elija al crear el servicio QnA Maker. Obtenga más detalles [aquí](./Tutorials/choosing-capacity-qnamaker-deployment.md).

</details>

<details>
<summary><b>¿Por qué no se ve nada en el menú desplegable cuando intento crear una nueva base de conocimiento?</b></summary>

**Respuesta**: Todavía no ha creado ningún servicio QnA Maker en Azure. Lea [aquí](./How-To/set-up-qnamaker-service-azure.md) para saber cómo hacerlo.

</details>

<details>
<summary><b>¿Cómo se puede compartir una base de conocimiento con otros usuarios?</b></summary>

**Respuesta**: El uso compartido funciona en el nivel de un servicio QnA Maker, es decir, todas las knowledge bases de los servicios se compartirán. Obtenga información [aquí](./How-To/collaborate-knowledge-base.md) para colaborar en una knowledge base.

</details>

<details>
<summary><b>¿Puede compartir una base de conocimiento con un colaborador que no esté en el mismo inquilino de AAD, para modificarla?</b></summary>

**Respuesta**: El uso compartido se basa en el control de acceso basado en rol (RBAC) de Azure. Si puede compartir _cualquier_ recursos en Azure con otro usuario, también puede compartir QnA Maker.

</details>

<details>
<summary><b>Si tiene un plan de App Service con cinco bases de conocimiento de QnAMaker. ¿Puede asignar derechos de lectura o escritura a cinco usuarios distintos para que cada uno de ellos pueda acceder solo a una base de conocimiento de QnAMaker?</b></summary>

**Respuesta**: Puede compartir un servicio completo de QnAMaker, no bases de conocimiento individuales.

</details>

<details>
<summary><b>¿Cómo se puede cambiar el mensaje predeterminado cuando no se encuentra ninguna buena coincidencia?</b></summary>

**Respuesta**: El mensaje predeterminado es parte de la configuración de App Service.
- Vaya al recurso de App Service en Azure Portal.

![App Service QnA Maker](./media/qnamaker-faq/qnamaker-resource-list-appservice.png)
- Haga clic en la opción **Configuración**.

![Configuración de App Service QnA Maker](./media/qnamaker-faq/qnamaker-appservice-settings.png)
- Cambie el valor del parámetro **DefaultAnswer**.
- Reiniciar App Service

![Reiniciar App Service QnA Maker](./media/qnamaker-faq/qnamaker-appservice-restart.png)


</details>

<details>
<summary><b>¿Por qué mi vínculo de SharePoint no se extrae?</b></summary>

**Respuesta**: Para más información, consulte [Ubicaciones de orígenes de datos](./Concepts/knowledge-base.md#data-source-locations).

</details>

<details>
<summary><b>Las actualizaciones que realizo en mi base de conocimiento no se publican. ¿Por qué no?</b></summary>

**Respuesta**: Cada edición, ya sea para actualizar la tabla, hacer pruebas o cambiar la configuración, debe guardarse para poder publicarla. Asegúrese de hacer clic en el botón **Save and train** (Guardar y entrenar) después de cada edición que realice.

</details>

<details>
<summary><b>¿La base de conocimiento admite datos enriquecidos o multimedia?</b></summary>

**Respuesta**:

#### <a name="multimedia-auto-extraction-for-files-and-urls"></a>Extracción automática multimedia para archivos y direcciones URL

* Direcciones URL: capacidad limitada de conversión de HTML a Markdown.
* Archivos: no compatibles.

#### <a name="answer-text-in-markdown"></a>Texto de respuesta en formato Markdown
Una vez que los pares de QnA estén en knowledge base, puede editar el texto de Markdown de la respuesta para incluir vínculos a los elementos multimedia disponibles desde direcciones URL públicas.


</details>

<details>
<summary><b>¿Admite QnA Maker otros idiomas aparte de inglés?</b></summary>

**Respuesta**: Obtenga más detalles acerca de los [idiomas admitidos](./Overview/languages-supported.md).

Si tiene contenido en varios idiomas, asegúrese de crear un servicio independiente para cada idioma.

</details>

## <a name="manage-service"></a>Administración de servicios

<details>
<summary><b>¿Cuándo debo reiniciar mi instancia de App Service?</b></summary>

**Respuesta**: Actualice su instancia de App Service cuando el icono de precaución aparezca junto al valor de versión de la base de conocimiento en la tabla **Claves de punto de conexión** de la [página](https://www.qnamaker.ai/UserSettings) **Configuración de usuario**.

</details>

<details>
<summary><b>Eliminé mi servicio Search existente. ¿Cómo lo puedo corregir?</b></summary>

**Respuesta**: Si elimina un índice de Azure Cognitive Search, la operación es definitiva y no es posible recuperar el índice.

</details>

<details>
<summary><b>Eliminé mi índice `testkb` en el servicio Search. ¿Cómo lo puedo corregir?</b></summary>

**Respuesta**: No se pueden recuperar los datos antiguos. Cree un nuevo recurso de QnA Maker y cree su base de conocimiento nuevamente.

</details>

<details>
<summary><b>¿Cuándo debo actualizar mis claves de punto de conexión?</b></summary>

**Respuesta**: Actualice las claves de punto de conexión si sospecha que han sido objeto de alguna acción fraudulenta.

</details>

<details>
<summary><b>¿Se puede usar el mismo recurso de Azure Cognitive Search para bases de conocimiento que emplean varios idiomas?</b></summary>

**Respuesta**: Para usar varios idiomas y varias bases de conocimiento, el usuario tiene que crear un recurso de QnA Maker para cada idioma. De esta manera, se creará un servicio de Azure Search independiente por idioma. La combinación de bases de datos de distintos idiomas en un único servicio de Azure Search dará lugar a una importancia degradada de los resultados.

</details>

<details>
<summary><b>¿Cómo se puede cambiar el nombre del recurso de Azure Cognitive Search usado por QnA Maker?</b></summary>

**Respuesta**: El nombre del recurso de Azure Cognitive Search es el nombre del recurso de QnA Maker con algunas letras aleatorias colocadas al final. Esto hace que sea más difícil distinguir entre varios recursos de Search en QnA Maker. Cree un servicio de búsqueda independiente (asígnele el nombre que desee) y conéctelo a su servicio de QnA. Los pasos son similares a los que debe hacer para [actualizar una instancia de Azure Search](How-To/set-up-qnamaker-service-azure.md#upgrade-the-azure-cognitive-search-service).

</details>

<details>
<summary><b>Cuando QnA Maker devuelve `Runtime core is not initialized,`, ¿cómo puedo corregirlo?</b></summary>

**Respuesta**: Es posible que el espacio en disco para el servicio de aplicaciones esté lleno. Pasos para corregir el espacio en disco:

1. En [Azure Portal](https://portal.azure.com), seleccione el servicio de aplicaciones de QnA Maker y, a continuación, detenga el servicio.
1. En el servicio de aplicaciones, seleccione **Herramientas de desarrollo**, **Herramientas avanzadas** y, a continuación, **Ir**. Se abre una nueva ventana del explorador.
1. Seleccione **Depurar consola** y, a continuación, **CMD** para abrir una herramienta de línea de comandos.
1. Vaya al directorio _site/wwwroot/Data/QnAMaker/_ .
1. Quite todas las carpetas cuyo nombre comience por `rd`.

    **No elimine** lo siguiente:

    * Archivo KbIdToRankerMappings.txt
    * Archivo EndpointSettings.json
    * Carpeta EndpointKeys

1. Inicie el servicio de aplicaciones.
1. Obtenga acceso a la base de conocimiento para comprobar que funcione ahora.

</details>

## <a name="integrate-with-other-services-including-bots"></a>Integración con otros servicios, como bots

<details>
<summary><b>¿Es necesario utilizar Bot Framework para usar QnA Maker?</b></summary>

**Respuesta**: No, no es necesario usar [Bot Framework](https://github.com/Microsoft/botbuilder-dotnet) con QnA Maker. Sin embargo, QnA Maker se ofrece como una de las diversas plantillas de  [Azure Bot Service](https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0). Bot Service permite el desarrollo rápido de bots inteligentes mediante Microsoft Bot Framework y se ejecuta en un entorno sin servidor.

</details>

<details>
<summary><b>¿Cómo se puede crear un bot con QnA Maker?</b></summary>

**Respuesta**: Siga las instrucciones de [esta](./Quickstarts/create-publish-knowledge-base.md) documentación para crear su Bot con Azure Bot Service.

</details>

<details>
<summary><b>¿Cómo se usa una base de conocimiento diferente con un servicio de bot existente de Azure?</b></summary>

**Respuesta**: Debe tener la siguiente información sobre la base de conocimiento:

* Id. de base de conocimiento.
* Nombre de subdominio personalizado del punto de conexión publicado de la base de conocimiento, conocido como `host`, que se encuentra en la página **Configuración** después de la publicación.
* Clave del punto de conexión publicado de la base de conocimiento; se encuentra en **Settings** (Configuración) después de publicarlo.

Con esta información, vaya al servicio de aplicaciones del bot en Azure Portal. En **Configuración -> Configuración -> Configuración de la aplicación**, cambie esos valores.

La clave del punto de conexión de la base de conocimiento se llama `QnAAuthkey` en el servicio ABS.

</details>

<details>
<summary><b>¿Pueden dos o más aplicaciones cliente compartir una base de conocimiento?</b></summary>

**Respuesta**: Sí, la base de conocimiento se puede consultar desde cualquier número de clientes. Si la respuesta de la base de conocimiento parece lenta o se agota el tiempo de espera, considere la posibilidad de actualizar el nivel de servicio del servicio de aplicaciones asociado a la base de conocimiento.

</details>

<details>
<summary><b>¿Cómo puedo insertar el servicio QnA Maker en mi sitio web?</b></summary>

**Respuesta**: Siga estos pasos para insertar el servicio QnA Maker como control de chat en web en su sitio web:

1. Cree su bot de P+F siguiendo las instrucciones que encontrará [aquí](./Quickstarts/create-publish-knowledge-base.md).
2. Habilite el chat en web mediante los pasos que se indican [aquí](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-webchat).

</details>

## <a name="data-storage"></a>Almacenamiento de datos

<details>
<summary><b>¿Qué datos se almacenan y dónde?</b></summary>

**Respuesta**:

Cuando se crea el servicio de QnA Maker, se selecciona una región de Azure. Sus bases de conocimiento y los archivos de registro se almacenan en esta región.

</details>