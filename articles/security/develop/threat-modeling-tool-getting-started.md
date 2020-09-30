---
title: 'Introducción: Microsoft Azure Threat Modeling Tool | Microsoft Docs'
description: Obtenga información sobre cómo empezar a usar Threat Modeling Tool. Cree un diagrama, identifique amenazas, mitíguelas y valide cada mitigación.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/17/2017
ms.author: jegeib
ms.openlocfilehash: 6ddb2ecef3a70ed6a56b404e64cef5b78b7abd13
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90980464"
---
# <a name="getting-started-with-the-threat-modeling-tool"></a>Introducción a Threat Modeling Tool

Microsoft Threat Modeling Tool 2018 2018 se lanzó para que tuviese disponibilidad general en septiembre de 2018, como una opción gratuita de **[hacer clic para descargar](https://aka.ms/threatmodelingtool)** . El cambio en el mecanismo de entrega nos permite insertar las mejoras y correcciones de errores más recientes para los clientes cada vez que abran la herramienta, facilitando el mantenimiento y el uso.
Este artículo le guía por el proceso de empezar a usar la solución de modelado de amenazas SDL de Microsoft y le muestra cómo utilizar la herramienta para desarrollar modelos de amenazas excepcionales como columna vertebral del proceso de seguridad.

Este artículo se basa en los conocimientos actuales de la solución de modelado de amenazas SDL. Para una revisión rápida, consulte **[Aplicaciones web de modelado de amenazas](https://msdn.microsoft.com/library/ms978516.aspx)** y una versión archivada del artículo de MSDN sobre el **[descubrimiento de defectos de seguridad con el enfoque STRIDE](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGVmYXVsdGRvbWFpbnxzZWN1cmVwcm9ncmFtbWluZ3xneDo0MTY1MmM0ZDI0ZjQ4ZDMy)** publicado en 2006.

En resumen, la solución implica la creación de un diagrama, la identificación de las amenazas, su mitigación y la validación de cada una. Este es un diagrama que esquematiza este proceso:

![Proceso SDL](./media/threat-modeling-tool-getting-started/sdlapproach.png)

## <a name="starting-the-threat-modeling-process"></a>Inicio del proceso de modelado de amenazas

Cuando inicie Threat Modeling Tool, observará algunas cosas, como se muestra en la imagen:

![Página de inicio en blanco](./media/threat-modeling-tool-feature-overview/tmtstart.png)

### <a name="threat-model-section"></a>Sección del modelo de amenaza

| Componente                                   | Detalles                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Botón de comentarios, sugerencias y problemas** | Le lleva al **[Foro de MSDN](https://social.msdn.microsoft.com/Forums/en-US/home?forum=sdlprocess)** para todo lo relacionado con SDL. Proporciona una oportunidad para leer lo que otros usuarios hacen, junto con soluciones alternativas y recomendaciones. Si sigue sin encontrar lo que está buscando, envíe un mensaje de correo electrónico a tmtextsupport@microsoft.com para que nuestro equipo de soporte técnico le ayude                                                                                                                            |
| **Creación de un modelo**                          | Abre un lienzo en blanco para que dibuje el diagrama. Asegúrese de seleccionar qué plantilla le gustaría usar para el modelo                                                                                                                                                                                                                                                                                                                                                                       |
| **Plantilla para los modelos nuevos**                 | Debe seleccionar qué plantilla utilizar antes de crear un modelo. La plantilla principal es la del modelo de amenaza de Azure, que contiene galerías de símbolos específicas de Azure, amenazas y las formas de mitigarlas. Si desea obtener modelos genéricos, seleccione la base de conocimiento de modelos de amenazas SDL en el menú desplegable. ¿Desea crear su propia plantilla o enviar una nueva para todos los usuarios? Visite nuestra página de GitHub de **[repositorio de plantillas](https://github.com/Microsoft/threat-modeling-templates)** para obtener más información                              |
| **Abrir un modelo**                            | <p>Se abren los modelos de amenazas guardados previamente. La característica Recently Opened Models (modelos abiertos recientemente) es útil si tiene que abrir los archivos más recientes. Cuando mantenga el mouse sobre la selección, encontrará dos formas de abrir los modelos:</p><p><ul><li>Abrir desde este equipo: modo clásico de abrir un archivo usando el almacenamiento local</li><li>Abrir desde OneDrive: los equipos pueden usar las carpetas de OneDrive para guardar y compartir todos sus modelos de amenaza en una única ubicación para ayudar a aumentar la productividad y la colaboración</li></ul></p> |
| **Guía de introducción**                   | Abre la página principal de **[Microsoft Threat Modeling Tool](threat-modeling-tool.md)**                                                                                                                                                                                                                                                                                                                                                                                            |

### <a name="template-section"></a>Sección de plantilla

| Componente               | Detalles                                                                                                                                                          |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Crear nueva plantilla** | Se abre una plantilla en blanco para que la cree. A menos que tenga muchos conocimientos sobre la creación de plantillas desde el principio, le recomendamos que empiece a partir de las existentes |
| **Abrir plantilla**       | Abre una plantilla para que pueda realizar cambios en ella                                                                                                              |

El equipo de Threat Modeling Tool trabaja constantemente para mejorar la funcionalidad y la experiencia de la herramienta. Pueden producirse algunos cambios poco importantes a lo largo del año, pero todos los principales requieren escribir de nuevo en la guía. Remítase a ella con frecuencia para garantizar que conoce los anuncios más recientes.

## <a name="building-a-model"></a>Generación de un modelo

En esta sección, seguimos a:

- Cristina (programadora)
- Ricardo (administrador de programas) y
- Ashish (evaluador)

Van a realizar el proceso de desarrollo de su primer modelo de amenaza.

> Ricardo: Hola Cristina, trabajé en el diagrama del modelo de amenaza y quería asegurarme de que los detalles son correctos. ¿Podrías ayudarme a comprobarlo?
> Cristina: Totalmente. Vamos a echar un vistazo.
> Ricardo abre la herramienta y comparte la pantalla con Cristina.

![Modelo de amenaza básica](./media/threat-modeling-tool-feature-overview/basictmt.png)

> Cristina: Bien, parece sencillo, pero ¿puedes guiarme para comprobarlo?
> Ricardo: Por supuesto. Este es el análisis:
> - El usuario se dibuja como una entidad externa: un cuadrado
> - Van a enviar comandos a nuestro servidor web: el círculo
> - El servidor web va a consultar una base de datos: dos líneas paralelas

Lo que Ricardo ha mostrado a Cristina es un DFD, abreviatura de **[diagrama de flujo de datos](https://en.wikipedia.org/wiki/Data_flow_diagram)** . Threat Modeling Tool permite a los usuarios especificar los límites de confianza, indicados por las líneas de puntos rojos, que muestran dónde están las distintas entidades en el control. Por ejemplo, los administradores de TI requieren un sistema de Active Directory para la autenticación, por lo que Active Directory está fuera de su control.

> Cristina: Me parece correcto. ¿Qué ocurre con las amenazas?
> Ricardo: Permíteme que te lo muestre.

## <a name="analyzing-threats"></a>Análisis de las amenazas

Una vez que el usuario hace clic en la vista de análisis desde la selección del menú de icono (el archivo con la lupa), se le muestra una lista de amenazas generadas que Threat Modeling Tool encontró en función de la plantilla predeterminada, que usa el método SDL denominado **[STRIDE (suplantación de identidad, manipulación, revelación de información, rechazo, denegación de servicio y elevación de privilegios)](https://en.wikipedia.org/wiki/STRIDE_(security))** . La idea es que el software viene con un conjunto predecible de amenazas, que se puede encontrar usando estas seis categorías.

Este enfoque es como proteger su casa asegurándose de que cada puerta y cada ventana disponen de un mecanismo de bloqueo antes de agregar un sistema de alarma o de perseguir al ladrón.

![Amenazas básicas](./media/threat-modeling-tool-getting-started/basicthreats.png)

Ricardo comienza seleccionando el primer elemento de la lista. Esto es lo que sucede:

En primer lugar, se ha mejorado la interacción entre las dos galerías de símbolos

![Captura de pantalla que muestra dos galerías de símbolos y la flecha curva que las conecta con un mayor grosor de línea.](./media/threat-modeling-tool-getting-started/interaction.png)

En segundo lugar, aparece información adicional sobre la amenaza en la ventana Threat Properties (Propiedades de amenaza)

![Información de interacción](./media/threat-modeling-tool-getting-started/interactioninfo.png)

La amenaza generada le ayuda a comprender los posibles errores de diseño. La categorización STRIDE le da una idea de los posibles vectores de ataque, mientras que la descripción adicional le indica exactamente cuál es el problema, junto con las formas posibles de mitigarlo. Puede utilizar los campos modificables para escribir notas en los detalles de justificación o cambiar las clasificaciones de prioridad según la barra de error de su organización.

Las plantillas de Azure tienen detalles adicionales para ayudar a los usuarios a entender no solo cuál es el problema, sino también cómo corregirlo agregando descripciones, ejemplos e hipervínculos a la documentación específica de Azure.

La descripción le hizo darse cuenta de la importancia de agregar un mecanismo de autenticación para impedir que los usuarios sean suplantados, revelando la primera amenaza en la que se trabajará. Solo algunos minutos de conversación con Cristina bastaron para entender la importancia de implementar el control de acceso y los roles. Ricardo escribió algunas notas rápidas para asegurarse de que se implementaban.

A medida que Ricardo se ocupaba de las amenazas relativas a la divulgación de información, se dio cuenta de que el plan de control de acceso requería algunas cuentas de solo lectura para la auditoría y la generación de informes. Se preguntó si debería tratarse como una nueva amenaza, aunque las mitigaciones fueran las mismas, por lo que anotó la amenaza como correspondía.
También pensó más sobre la divulgación de información y se percató de que las cintas de copia de seguridad iban a necesitar cifrado, una tarea para el equipo de operaciones.

Las amenazas no aplicables al diseño debido a las mitigaciones existentes o a garantías de seguridad pueden cambiarse a "No aplicables" en la lista desplegable del estado. Hay otras tres opciones: No iniciado, la selección predeterminada; Necesita investigación, que se usa para hacer un seguimiento de los elementos; y Mitigado, que indica que está en pleno funcionamiento.

## <a name="reports--sharing"></a>Informes y uso compartido

Una vez que Ricardo examina la lista con Cristina y agrega algunas notas importantes, mitigaciones o justificaciones, los cambios de estado y prioridad, selecciona Informes -> Crear informe completo -> Guardar informe, con lo que se imprime un informe muy útil para repasarlo con sus compañeros y asegurarse de implementar el trabajo de seguridad apropiado.

![Información de interacción](./media/threat-modeling-tool-feature-overview/report.png)

Si Ricardo desea compartir el archivo en su lugar, puede hacerlo fácilmente guardándolo en la cuenta de OneDrive de su organización. Cuando lo haga, podrá copiar el vínculo de documento y compartirlo con sus compañeros. 

## <a name="threat-modeling-meetings"></a>Herramienta de modelado de amenazas

Cuando Ricardo envió su modelo de amenazas a su compañero con OneDrive, Ashish, el evaluador, no se dejó impresionar. Parecía que Ricardo y Cristina habían olvidado algunos casos importantes más complejos, que podían verse comprometidos fácilmente. Su escepticismo es un complemento de los modelos de amenaza.

En este escenario, una vez que Ashish se encargó del modelo de amenaza, convocó dos reuniones: una para sincronizar el proceso y examinar los diagramas y, a continuación, otra para revisar las amenazas y concluir el proceso.

En la primera reunión, Ashish empleó diez minutos en mostrar a los demás el proceso de modelado de amenazas SDL. A continuación, se detuvo en el diagrama del modelo de amenazas y comenzó a explicarlo en detalle. Tras cinco minutos, se había identificado un importante componente que faltaba.

Unos minutos después, Ashish y Ricardo comenzaron una larga discusión sobre cómo se había creado el servidor web. No era la forma ideal de proceder en una reunión, pero todos los usuarios acordaron finalmente que detectar la discrepancia al principio iba a ahorrarles tiempo en el futuro.

En la segunda reunión, el equipo examinó las amenazas, debatieron algunas maneras de abordarlas y concluyó el modelo de amenazas. Registraron el documento en el control de origen y continuaron con el desarrollo.

## <a name="thinking-about-assets"></a>Evaluación de los recursos

Algunos lectores que hayan modelado una amenaza pueden darse cuenta de que no hablaron sobre todos los recursos. Hemos descubierto que muchos ingenieros de software conocen mejor su software de lo que entiendan el concepto de proceso y de en qué procesos podría estar interesado un atacante.

Si va a modelar las amenazas de una casa, puede comenzar por pensar en la familia, en las fotos que son irreemplazables o en las obras de arte valiosas. Quizás pueda comenzar por pensar quién podría entrar en la casa y en el sistema de seguridad actual. O bien podría empezar teniendo en cuenta las características físicas, como la piscina o el porche delantero. Esto es igual que pensar en los recursos, los atacantes o el diseño del software. Cualquiera de estos tres enfoques funcionan.

El enfoque del modelado de amenazas que hemos presentado en esta guía es mucho más sencillo de lo que Microsoft hacía antes. Encontramos que el enfoque de diseño de software funciona bien para muchos equipos. Esperamos que sea el caso del suyo.

## <a name="next-steps"></a>Pasos siguientes

Envíe sus preguntas, comentarios y preocupaciones a tmtextsupport@microsoft.com. **[Descargue](https://aka.ms/threatmodelingtool)** Threat Modeling Tool para empezar.
