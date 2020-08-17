---
title: 'Ética y uso responsable: Personalizer'
titleSuffix: Azure Cognitive Services
description: Estas instrucciones están pensadas para ayudarle a implementar la personalización de forma que le ayude a generar confianza en su empresa y servicio. Asegúrese de que se toma un tiempo para investigar, conocer y deliberar sobre el impacto de la personalización en la vida de las personas. En caso de duda, busque orientación.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 06/12/2019
ms.openlocfilehash: 602da28f0c235fb0e797a493bc1160631c042a9d
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132728"
---
# <a name="guidelines-for-responsible-implementation-of-personalizer"></a>Directrices para la implementación responsable de Personalizer

Para que los usuarios y la sociedad comprendan todo el potencial de la inteligencia artificial, las implementaciones se deben diseñar de modo que se ganen la confianza de aquellos que agregan la inteligencia artificial a sus aplicaciones y la de los usuarios de las aplicaciones compiladas con esta. Estas instrucciones están pensadas para ayudarle a implementar la Personalizer de forma que le ayude a generar confianza en su empresa y servicio. Asegúrese de que se toma un tiempo para investigar, conocer y deliberar sobre el impacto de la personalización en la vida de las personas. En caso de duda, busque orientación.

Estas directrices no están pensadas para proporcionar asesoramiento legal y debería asegurarse por otro medio de que la aplicación cumple los cambiantes requisitos legales de esta área y del sector.

Además, al diseñar la aplicación mediante Personalizer, debe tener en cuenta una serie de responsabilidades a la hora de desarrollar cualquier sistema de inteligencia artificial basado en datos, entre las que se incluyen temas éticos, de seguridad, inclusión, transparencia y responsabilidad. Puede leer más acerca de estos temas en la sección de [lecturas recomendadas](#recommended-reading).

Puede usar el siguiente contenido como lista de comprobación inicial y personalizarlo y mejorarlo para adaptarlo a su escenario. Este documento incluye dos secciones principales: La primera está dedicada a resaltar consideraciones de uso responsables a la hora de elegir escenarios, características y recompensas para Personalizer. La segunda toma un conjunto de valores que Microsoft cree que se deben tener en cuenta al crear sistemas de inteligencia artificial y proporciona sugerencias prácticas sobre cómo el uso de Personalizer les afecta y habla también de los posibles riesgos.


## <a name="your-responsibility"></a>Su responsabilidad

Todas las directrices de una implementación responsable se basan en que los desarrolladores y las empresas que usan Personalizer son responsables de los efectos del uso de estos algoritmos en la sociedad. Si va a desarrollar una aplicación para que la implemente la organización, debe reconocer su papel y su responsabilidad en cuanto a su funcionamiento y a cómo afecta a los usuarios. Si va a desarrollar una aplicación para que la implemente un tercero, acuerde con ellos quién es responsable en última instancia del comportamiento de la aplicación y ponga por escrito ese acuerdo.

La confianza se basa en la noción de compromisos cumplidos: tenga en cuenta a los usuarios, la sociedad y el marco legal en el que funcionarán sus aplicaciones para identificar los compromisos explícitos e implícitos que pueden tener.

Microsoft se esfuerza constantemente en mejorar sus documentos y herramientas para ayudarle a actuar en relación con estas responsabilidades. [Proporcione comentarios a Microsoft](mailto:cogsvcs-RL-feedback@microsoft.com?subject%3DPersonalizer%20Responsible%20Use%20Feedback&body%3D%5BPlease%20share%20any%20question%2C%20idea%20or%20concern%5D) si cree que hay herramientas, características de productos y documentos adicionales que le ayudarían a implementar estas directrices de uso de Personalizer.


## <a name="factors-for-responsibly-implementing-personalizer"></a>Factores para la implementación responsable de Personalizer

La implementación de Personalizer puede ser de gran valor para los usuarios y para su negocio. Para implementar Personalizer de forma responsable, empiece teniendo en cuenta las siguientes directrices:

* Al elegir los casos de uso a los que aplicar la personalización.
* Al generar [funciones de recompensa](concept-rewards.md).
* Al elegir las [características](concepts-features.md) del contexto y las posibles acciones que usará para la personalización.


## <a name="choosing-use-cases-for-personalizer"></a>Elección de los casos de uso para Personalizer

El uso de un servicio que aprende a personalizar contenido e interfaces de usuario resulta útil. También se pueden aplicar de forma incorrecta si la aplicación genera efectos secundarios negativos en el mundo real, incluyendo el hecho de que los usuarios no sean conscientes de la personalización del contenido.

Entre los ejemplos de usos de Personalizer con una posibilidad elevada de efectos secundarios negativos o una falta de transparencia se incluyen los escenarios en los que la "recompensa" depende de muchos factores complejos a largo plazo que, cuando se simplifican en una recompensa inmediata pueden tener resultados desfavorables para los usuarios. Estas opciones se suelen considerar como "consecuencias indirectas" u opciones que conllevan un riesgo de perjuicio. Por ejemplo:


* **Finanzas**: La personalización de ofertas en préstamos, productos financieros o de seguros en los que los factores de riesgo se basan en datos que los individuos no conocen, no pueden obtener ni pueden impugnar.
* **Educación**: La personalización de la clasificación de cursos escolares e instituciones educativas en los que las recomendaciones pueden contribuir a propagar prejuicios o a reducir el conocimiento de los usuarios del resto de opciones.
* **Democracia y participación cívica**: La personalización de contenido para usuarios con el objetivo de influenciar opiniones es consecuente y manipuladora.
* **Evaluación de recompensas por parte de terceros**: La personalización de elementos en los que la recompensa se basa en una evaluación del usuario por parte de terceros en lugar de tener una recompensa generada por el propio comportamiento del usuario.
* **Intolerancia a la exploración**: Cualquier situación en la que el comportamiento de exploración de Personalizer sea perjudicial.

Al elegir los casos de uso para Personalizer:

* Inicie el proceso de diseño analizando cómo la personalización ayuda a los usuarios.
* Tenga en cuenta las consecuencias negativas en el mundo real en caso de que algunos elementos no estén correctamente ordenados para los usuarios debido a patrones de personalización o exploración específicos.
* Considere si su caso de uso constituye un procesamiento automatizado que afecta de forma significativa a los asuntos de datos regulados en el Artículo 22 de [GDPR](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32016R0679) u otras leyes.
* Considere la posibilidad de crear bucles de profecías que se cumplen automáticamente. Se puede dar este caso si una recompensa de personalización entrena un modelo de forma que este pueda excluir posteriormente a un grupo demográfico del acceso a contenido pertinente. Por ejemplo, la mayoría de las personas de un barrio pobre no reciben ofertas de seguros premium por lo que, poco a poco, las personas del barrio dejan de mirar dichas ofertas por completo, si no hay suficiente exploración.
* Guarde copias de los modelos y de las directivas de aprendizaje en caso de que sean necesarias para reproducir Personalizer en el futuro. Puede hacerlo periódicamente o en cada período de actualización del modelo.
* Considere el nivel de exploración adecuado para el espacio y cómo usarlo como herramienta para mitigar los efectos tipo "cámara de eco".


## <a name="selecting-features-for-personalizer"></a>Selección de características para Personalizer

La personalización del contenido depende del hecho de tener información útil sobre el contenido y el usuario. Tenga en cuenta que, para algunas aplicaciones y sectores, algunas características de usuario se pueden considerar directa o indirectamente como discriminatorias y potencialmente ilegales.

Analice el efecto de estas características:

* **Datos demográficos de los usuarios**: Características relacionadas con el género, edad, raza, religión: Puede que estas características no se permitan en determinadas aplicaciones por motivos legales y que no sea ético personalizar el contenido mediante su uso ya que la personalización podría propagar generalizaciones y prejuicios. Un ejemplo de esta propagación de prejuicios sería una oferta de trabajo de ingeniería que no se les muestra a personas mayores ni a audiencias con determinado género.
* **Información regional**: En muchos lugares del mundo, la información de ubicación (por ejemplo, un código postal o el nombre del barrio) puede estar muy relacionada con el nivel de ingresos, la raza o la religión.
* **Percepción de la imparcialidad por parte del usuario**: Incluso en aquellos casos en los que la aplicación está tomando decisiones firmes, considere el efecto en usuarios que perciben que el contenido mostrado en la aplicación cambia de una forma que parece estar relacionada con características que podrían considerarse como discriminatorias.
* **Prejuicios no intencionados en las características**:       Hay tipos de prejuicios que se pueden introducir con el uso de características que solo afectan a un subconjunto de la población. Esto requiere una atención adicional si las características se generan de forma algorítmica, como cuando se usa el análisis de imágenes para extraer los elementos de una imagen o el análisis de texto para detectar las entidades de este. Sea consciente de las características de los servicios que usa para crear estas características.

Aplique los siguientes procedimientos al elegir las características que se van a enviar en contextos y acciones a Personalizer:

* Tenga en cuenta la legalidad y la ética que implica el uso de determinadas características en algunas aplicaciones y si características aparentemente inocentes podrían constituir indicadores discriminatorios para otros usuarios, indicadores que puede que quiera usar o evitar.
* Sea transparente con los usuarios a la hora de explicarles los algoritmos y el análisis de datos que se usa para personalizar las opciones que ellos ven.
* Pregúntese lo siguiente: ¿Les importaría a mis usuarios si uso esta información para personalizar el contenido para ellos? ¿Serían más felices si lo hago? ¿Me sentiría cómodo explicándoles cómo se tomó la decisión de mostrar u ocultar determinados elementos?
* Use datos de comportamiento en lugar de datos de clasificaciones o segmentaciones basados en otras características. Los distribuidores solían usar tradicionalmente la información demográfica por motivos históricos: los atributos demográficos eran sencillos de recopilar y de trabajar con ellos antes de la era digital, pero pregúntese cómo de pertinente es la información demográfica cuando dispone de datos de interacciones reales, contextuales e históricos más relacionados con las preferencias e identidades de los usuarios.
* Analice cómo impedir que las características sean "falsificadas" por usuarios malintencionados, ya que si se aprovechan de ellas en gran número podrían provocar que el entrenamiento de Personalizer fuera incorrecto para interrumpir, molestar e incluso acosar a propósito a determinadas clases de usuario.
* Si es apropiado y posible, diseñe la aplicación para permitir que los usuarios decidan si se deben usar o no determinadas características personales. Estas se podrían agrupar en epígrafes como, por ejemplo: "Información de ubicación", "Información de dispositivo", "Historial de compra anterior" etc.


## <a name="computing-rewards-for-personalizer"></a>Procesamiento de recompensas para Personalizer

Personalizer se esfuerza por mejorar la elección de qué acción va a recompensar en función de la puntuación de recompensa que proporciona la lógica de negocios de la aplicación.

Una puntuación de recompensa bien desarrollada puede convertirse en un obstáculo para un objetivo empresarial ligado a una misión de la organización.

Por ejemplo, si se recompensa por el número de clics esto hará que el servicio Personalizer analice el número de clics a costa de todo lo demás, incluso si aquello en lo que se hace clic puede distraer o no está ligado a un objetivo empresarial.

En el extremo opuesto, por ejemplo, un sitio web de noticias puede que desee establecer las recompensas ligadas a algo más significativo que el número de clics como, por ejemplo, "¿Ha pasado el usuario el tiempo suficiente como para leer el contenido?". "¿Han hecho clic en los artículos o referencias pertinentes?". Con Personalizer es fácil asociar las métricas estrechamente a las recompensas. Pero tenga cuidado de no confundir la involucración del usuario a corto plazo con buenos resultados.

### <a name="unintended-consequences-from-reward-scores"></a>Consecuencias no intencionadas de las puntuaciones de recompensa
Las puntuaciones de recompensa se pueden diseñar con la mejor de las intenciones pero generar, aún así, consecuencias imprevistas o resultados no intencionados en la manera en que Personalizer clasifica el contenido.

Considere los ejemplos siguientes:

* Una recompensa de personalización de contenido de vídeo basada en el porcentaje de minutos de vídeo vistos tenderá, probablemente, a clasificar en primer lugar los vídeos más cortos.
* Recompensar los recursos compartidos a través de las redes sociales sin un análisis de opinión de cómo se comparte o del contenido en sí, podría provocar que se clasificara en primer lugar contenido ofensivo, sin moderador o incluso provocador, lo cual suele incitar a mucha "involucración", pero agrega poco valor.
* Recompensar según los elementos de la interfaz de usuario que los usuarios no esperan que cambien puede interferir con la usabilidad y predictibilidad de una interfaz de usuario en la que los botones cambian sorprendentemente de ubicación o de propósito sin previo aviso, lo cual dificulta a determinados grupos de usuarios que sean productivos.

Implemente estos procedimientos recomendados:

* Realice varios experimentos sin conexión con el sistema usando diferentes enfoques de recompensas para comprender el impacto y los posibles efectos secundarios.
* Evalúe las funciones de recompensa y pregúntese a sí mismo si una persona extremadamente inocente podría malinterpretar el contenido y alcanzar resultados no deseados.


## <a name="responsible-design-considerations"></a>Consideraciones de diseño responsable

Las siguientes son áreas de diseño para implementaciones responsables de inteligencia artificial. Para más información sobre este marco de trabajo, consulte [The Future Computed](https://news.microsoft.com/futurecomputed/) (El futuro calculado).

![Valores de inteligencia artificial extraídos de "The Future Computed" (El futuro calculado)](media/ethics-and-responsible-use/ai-values-future-computed.png)

### <a name="accountability"></a>Rendición de cuentas
*Las personas que diseñan e implementan sistemas de inteligencia artificial deben ser responsables de su funcionamiento*.

* Cree directrices internas sobre cómo implementar Personalizer, documéntelas y comuníquelas a su equipo, directivos y proveedores.
* Realice revisiones periódicas sobre cómo se calculan las puntuaciones de recompensa, realice evaluaciones sin conexión para ver qué características están afectando a Personalizer y utilice los resultados para eliminar las características innecesarias.
* Comunique claramente a los usuarios cómo se utiliza Personalizer, con qué finalidad y con qué datos.
* Archive la información y los recursos como, por ejemplo, los modelos, las directivas de aprendizaje y el resto de la información que usa Personalizer, para que se puedan reproducir los resultados.

### <a name="transparency"></a>Transparencia
*Los sistemas de inteligencia artificial debe ser comprensibles*. Con Personalizer:

* *Proporcione a los usuarios información acerca de cómo se ha personalizado el contenido.* Por ejemplo, puede mostrar a los usuarios un botón con la etiqueta `Why These Suggestions?` que muestre indique qué características principales del usuario y qué acciones han desempeñado un rol importante en los resultados de Personalizer.
* Asegúrese de que las condiciones de uso mencionen que usará la información de los usuarios y su comportamiento para personalizar la experiencia.

### <a name="fairness"></a>Imparcialidad
*Los sistemas de inteligencia artificial deben tratar a todas las personas de forma imparcial*.

* No use Personalizer para casos de uso en que los resultados sean a largo plazo, indirectos o impliquen daños reales.
* No utilice características que no sean adecuadas para personalizar el contenido o que ayuden a propagar prejuicios no deseados. Por ejemplo, cualquier persona con una situación financiera similar debería ver las mismas recomendaciones personalizadas de productos financieros.
* Comprenda que los prejuicios pueden existir en características que proceden de editores, herramientas algorítmicas o de los propios usuarios.

### <a name="reliability-and-safety"></a>Confiabilidad y seguridad
*Los sistemas de inteligencia artificial deben funcionar de forma confiable y segura*. Para Personalizer:

* *No proporcione acciones a Personalizer que no deban elegirse*. Por ejemplo, se deben eliminar las películas no apropiadas de las acciones de personalización si va a hacer una recomendación para un usuario anónimo o menor de edad.
* *Administre el modelo de Personalizer como un recurso empresarial*.  Considere la frecuencia con la que guarda y realiza copias de seguridad del modelo y de las directivas de aprendizaje del bucle Personalizer y trate esta información como un importante recurso empresarial. Reproducir resultados pasados es importante para un autoservicio de auditoría y para medir las mejoras.
* *Proporcione canales para obtener comentarios directos de los usuarios*. Además de codificar las comprobaciones de seguridad para asegurarse de que solo el público correcto ve el contenido adecuado, proporcione un mecanismo de comentarios para que los usuarios notifiquen contenidos que puedan resultar extraños o desagradables. Más concretamente, si el contenido procede de otros usuarios o de terceros, considere la posibilidad de usar Microsoft Content Moderator para revisar y validar el contenido.
* *Realice evaluaciones sin conexión frecuentes*. Esto le ayudará a supervisar las tendencias y a garantizar la eficacia.
* *Establezca un proceso para detectar y actuar sobre manipulaciones malintencionadas*. Hay actores que se aprovecharán de la capacidad de los sistemas de aprendizaje automático e inteligencia artificial de aprender a partir del entorno para acercar los resultados hacia sus propios objetivos. Si el uso de Personalizer puede influir en decisiones importantes, asegúrese de tener los medios adecuados para detectar y mitigar este tipo de ataques incluyendo la revisión humana en las circunstancias adecuadas.

### <a name="security-and-privacy"></a>Seguridad y privacidad
*Los sistemas de inteligencia artificial deben ser seguros y respetar la privacidad*. Cuando use Personalizer:

* *Informe por adelantado a los usuarios sobre los datos que se recopilarán y cómo se usará y se obtendrá su consentimiento*, siguiendo la normativa local y del sector.
* *Proporcione controles de usuario de protección de la privacidad.* Para las aplicaciones que almacenan información personal, considere la posibilidad de incluir un botón que sea muy visible para funciones como:
   * `Show me all you know about me`
   * `Forget my last interaction`
   * `Delete all you know about me`

En algunos casos, esto puede constituir una exigencia legal. Considere los inconvenientes de tener que volver a entrenar los modelos periódicamente para que no contengan trazas de datos eliminados.

### <a name="inclusiveness"></a>Inclusión
*Dé respuesta a una amplia gama de necesidades humanas y experiencias*.
* *Proporcione experiencias personalizadas para interfaces habilitadas para accesibilidad.* La eficacia procedente de una buena personalización (aplicada para reducir la cantidad de esfuerzo, el movimiento y la repetición innecesaria en las interacciones) puede resultar especialmente beneficiosa en el caso de personas con discapacidades.
* *Ajuste el comportamiento de la aplicación al contexto*. Puede usar Personalizer para eliminar la ambigüedad entre las intenciones de un bot de chat, por ejemplo, la interpretación correcta puede ser contextual y una sola opción podría no ser adecuada para todos los casos.


## <a name="proactive-readiness-for-increased-data-protection-and-governance"></a>Preparación proactiva para una mayor protección y gobernanza

Es difícil predecir cambios específicos en contextos legales pero, en general, sería conveniente ir más allá del marco legal mínimo a la hora de garantizar un uso respetuoso de la información personal y de proporcionar transparencia y opciones relacionadas con la toma de decisiones algorítmica.


* Considere la posibilidad de planear por anticipado una situación en la que pueda haber nuevas restricciones en los datos que se recopilan de los usuarios y en la que haya una necesidad de explicar cómo se usaron dichos datos para tomar decisiones.
* Considere una disponibilidad adicional en la que los usuarios puedan incluir poblaciones vulnerables marginales, niños, usuarios económicamente vulnerables o cualquier otro usuario susceptible de ser influido por una manipulación algorítmica.
* Considere la posibilidad de una insatisfacción muy extendida con cómo los programas y algoritmos de recopilación de datos destinados a un público concreto (o para influir en este) se han llevado a cabo y en cómo evitar errores estratégicos ya probados.


## <a name="proactive-assessments-during-your-project-lifecycle"></a>Evaluaciones proactivas durante el ciclo de vida del proyecto

Analice la posibilidad de crear métodos para que los miembros del equipo, los usuarios y los propietarios de la empresa informen sobre sus preocupaciones acerca de un uso responsable, y de crear un proceso que priorice la resolución de estas e impida las represalias.

Toda persona que reflexione sobre los efectos del uso de cualquier tecnología está limitada por su perspectiva y su experiencia de vida. Aumente el número de opiniones disponibles incluyendo a voces más diversas en los equipos, grupos de usuarios o consejos consultivos y anime a los usuarios a compartir sus opiniones. Considere la posibilidad de usar materiales de formación y de aprendizaje para expandir aún más el conocimiento del equipo sobre esta materia y para agregar una funcionalidad para discutir temas complejos o confidenciales.

Considere la posibilidad de tratar las tareas relacionadas con un uso responsable de la misma forma que otras tareas transversales del ciclo de vida de la aplicación como, por ejemplo, la tareas relacionadas con la experiencia, la seguridad o las operaciones de desarrollo. Estas tareas y sus requisitos no pueden ser una idea tardía. Se debe analizar y comprobar el uso responsable a lo largo de todo el ciclo de vida de la aplicación.

## <a name="questions-and-feedback"></a>Preguntas y comentarios

Microsoft se esfuerza constantemente en mejorar sus documentos y herramientas para ayudarle a actuar en relación con estas responsabilidades. Nuestro equipo le anima a que [proporcione comentarios a Microsoft](mailto:cogsvcs-RL-feedback@microsoft.com?subject%3DPersonalizer%20Responsible%20Use%20Feedback&body%3D%5BPlease%20share%20any%20question%2C%20idea%20or%20concern%5D) si cree que hay herramientas, características de productos y documentos adicionales que le ayudarían a implementar estas directrices de uso de Personalizer.

## <a name="recommended-reading"></a>Lecturas recomendadas

* Consulte los seis principios de Microsoft para el desarrollo responsable de la inteligencia artificial publicados en enero de 2018 en el libro titulado [The Future Computed](https://news.microsoft.com/futurecomputed/)
* [Who Owns the Future?](https://www.goodreads.com/book/show/15802693-who-owns-the-future) (El futuro calculado. ¿Quién será el dueño del futuro?) de Jaron Lanier.
* [Weapons of Math Destruction](https://www.goodreads.com/book/show/28186015-weapons-of-math-destruction) (Armas de destrucción matemática) de Cathy O'Neil
* [Ethics and Data Science](https://www.oreilly.com/library/view/ethics-and-data/9781492043898/) (Ética y ciencia de datos) de DJ Patil, Hilary Mason, Mike Loukides.
* [ACM Code of Ethics](https://www.acm.org/code-of-ethics) (Código ACM de ética)
* [Ley de no discriminación por información genética (GINA)](https://en.wikipedia.org/wiki/Genetic_Information_Nondiscrimination_Act)
* [FATML Principles for Accountable Algorithms](https://www.fatml.org/resources/principles-for-accountable-algorithms) (Principios FATML de algoritmos de responsabilidad)


## <a name="next-steps"></a>Pasos siguientes

[Características: acción y contexto](concepts-features.md).
