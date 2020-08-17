---
title: Creación de plan de migración con Azure Migrate | Microsoft Docs
description: Proporciona instrucciones sobre cómo crear un plan de migración con Azure Migrate.
author: rashi-ms
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 06/21/2020
ms.author: rajosh
ms.openlocfilehash: db1de363856fd560fea97f8f9cdf542717c4cca3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090100"
---
# <a name="build-migration-plan-with-azure-migrate"></a>Creación de plan de migración con Azure Migrate

En este artículo se proporciona una guía rápida para ayudar a crear el plan de migración en Azure con [Azure Migrate](migrate-services-overview.md). Si tiene otras preguntas, repase estos recursos:

- [Preguntas generales](resources-faq.md) sobre Azure Migrate
- Preguntas sobre el [dispositivo de Azure Migrate](common-questions-appliance.md)
- Preguntas sobre la [migración de servidores](common-questions-server-migration.md)
- Obtenga respuestas a sus preguntas en el [foro de Azure Migrate](https://aka.ms/AzureMigrateForum)

## <a name="define-the-goals-of-cloud-migration"></a>Definir los objetivos de la migración en la nube

Antes de crear un plan de migración, es importante reconocer y evaluar la [motivación](/azure/cloud-adoption-framework/strategy/motivations) para pasar a la nube que puede lograr resultados empresariales más satisfactorios. Como se explica en el [Marco de adopción de la nube de Azure](/azure/cloud-adoption-framework), puede haber diferentes desencadenadores y enfoques de migración adecuados para su empresa:  

**Eventos empresariales críticos** | **Resultado de migración**
--- | ---
Salida del centro de datos | Ahorros en costos
Fusión, adquisición o desinversión | Reducción de la complejidad técnica o de proveedores
Reducción de gastos de capital | Optimización de las operaciones internas
Finalización del soporte técnico de tecnologías críticas | Aumento de agilidad empresarial
Respuesta a los cambios de cumplimiento normativo | Preparación para nuevas funcionalidades técnicas
Nuevos requisitos de soberanía de datos | Escalado para satisfacer demandas del mercado
Reducción de interrupciones y mejora de la estabilidad de TI | Escalado para satisfacer demandas geográficas

La motivación de la migración también puede ayudarle a reflejar los objetivos estratégicos y los resultados que le gustaría lograr mediante la migración a Azure. El siguiente paso es identificar y planear una ruta de migración a Azure que esté adaptada a sus cargas de trabajo. Azure Migrate: La herramienta de Server Assessment evalúa las cargas de trabajo locales y proporciona orientación y herramientas que le ayudarán a hacer la migración.

## <a name="understand-your-digital-estate"></a>Reconocimiento de su patrimonio digital

Comience por reconocer la infraestructura, las aplicaciones y las dependencias locales para ayudar a identificar las cargas de trabajo que quiere migrar a Azure y obtener proyecciones de costos optimizados. La herramienta de Server Assessment le ayudará a responder las siguientes preguntas:

### <a name="what-workloads-are-in-use"></a>¿Qué cargas de trabajo están en uso?

Use el dispositivo de Azure Migrate ligero para realizar una detección sin agente de las máquinas virtuales de VMware locales, las máquinas virtuales de Hyper-V y los servidores físicos. La detección continua recopila metadatos de configuración y rendimiento del equipo y también se puede usar para obtener el inventario de las aplicaciones instaladas y los roles o características que se ejecutan en las máquinas locales. El dispositivo de Azure Migrate recopila:

- Detalles de los metadatos de los equipos, discos y NIC.

- Aplicaciones instaladas, incluidas aplicaciones y roles/características.  

- Datos de rendimiento, incluido el uso de CPU y memoria, IOPS de disco y rendimiento.

A continuación, exporte la lista de inventario de aplicaciones para averiguar todas las instancias de SQL Server que se ejecutan en las cargas de trabajo y use el Azure Migrate: Herramienta de evaluación de bases de datos para reconocer su preparación.

 ![Inventario de aplicaciones en el portal](./media/concepts-migration-planning/application-inventory-portal.png)

 ![Exportación del inventario de aplicaciones](./media/concepts-migration-planning/application-inventory-export.png)

Junto con los datos de detección de la herramienta de evaluación del servidor, use los datos de CMDB existentes para crear nuestra vista de su servidor y base de datos, y reconocer la distribución del servidor entre unidades de negocio, propietarios de aplicaciones, geografías, etc., que puede ayudar a su vez a priorizar las cargas de trabajo que se van a migrar.

### <a name="what-dependencies-exist-between-workloads"></a>¿Qué dependencias existen entre las cargas de trabajo?

Una vez que haya detectado los servidores, use la asignación de dependencias sin agente para visualizar e identificar las dependencias entre servidores y las estrategias de optimización para mover servidores interdependientes a Azure. La visualización le ayuda a reconocer si ciertas máquinas están en uso o si se pueden retirar en lugar de migrarse.  Asegúrese de que se analicen las dependencias para garantizar que no se olvida de nada y evitar interrupciones inesperadas durante la migración. Una vez que haya realizado el inventario de aplicaciones y la asignación de dependencias, puede crear grupos de confianza alta e iniciar la evaluación de los servidores.

 ![Mapa de dependencia](./media/concepts-migration-planning/expand-client-group.png)

### <a name="are-they-optimized-and-sized-properly"></a>¿Están optimizados y dimensionados correctamente?

Dado que Azure proporciona flexibilidad para cambiar el tamaño de la capacidad de la nube con el tiempo, la migración es una oportunidad para optimizar los recursos de memoria y CPU asignados a los servidores. Cree una evaluación en el grupo identificado antes de reconocer el historial de rendimiento de las cargas de trabajo, lo que resultará muy importante en las recomendaciones de discos y SKU de máquinas virtuales con privilegios en Azure.

## <a name="assess-your-readiness-for-migration"></a>Evaluación de la preparación para la migración

### <a name="readiness-and-suitability-analysis-for-azure"></a>Preparación y análisis de idoneidad para Azure
Exporte el informe de evaluación de máquinas virtuales de Azure y fíltrelo por las siguientes categorías de preparación para reconocer la preparación de la máquina virtual de Azure:

- **Preparada para Azure**: Puede migrar estas máquinas tal como está en Azure sin necesidad de realizar ningún cambio.  

- **Condicionalmente preparada para Azure**: Puede migrar estas máquinas a Azure, pero necesitará cambios menores en estos servidores según las instrucciones de corrección que se proporcionan en la evaluación.

- **No está preparado para Azure**: No puede migrar estas máquinas a Azure tal cual y se necesitan corregir los problemas según las instrucciones de corrección antes de la migración.

- **Preparación desconocida**: Azure Migrate no puede determinar la preparación de la máquina debido a que no hay metadatos suficientes.

Mediante el uso de las evaluaciones de base de datos, puede evaluar la preparación para migrar los datos de SQL Server a Azure SQL Database o a Azure SQL Managed Instances. Puede ver el porcentaje de estado de preparación de la migración para cada una de las instancias de SQL Server. Además, para cada una de las instancias, puede ver el destino recomendado en Azure, los bloqueadores de migración potenciales, el recuento de cambios importantes, la preparación para Azure SQL DB o la máquina virtual de Azure SQL y el nivel de compatibilidad. Puede profundizar más para reconocer el impacto de los bloqueadores de migración y las recomendaciones para corregirlos.

 ![Valoraciones de la base de datos:](./media/concepts-migration-planning/database-assessment-portal.png)

### <a name="sizing-recommendations"></a>Recomendaciones de dimensionamiento

Una vez que la máquina se marca como preparada para Azure, Server Assessment realiza recomendaciones de dimensionamiento para identificar la máquina virtual de Azure y la SKU de las máquinas virtuales. Puede ver la recomendación de ajuste de tamaño en función del historial de rendimiento (para optimizar los recursos durante la migración) o según su configuración local sin tener en cuenta el historial de rendimiento. En el caso de las bases de datos, puede ver las recomendaciones sobre la SKU de la base de datos, el plan de tarifa y el nivel de proceso en la evaluación de la base de datos.  

### <a name="compute-assessments-to-get-estimated-costs-for-running-the-workloads-in-azure"></a>Calcular evaluaciones para obtener los costos estimados de ejecución de las cargas de trabajo en Azure

La opción de ajuste de dimensionamiento a la derecha *basada en el rendimiento* en las evaluaciones permite optimizar las cargas de trabajo de Azure. Además de la aplicación de los derechos, hay algunas otras formas de ahorrar costos:

- **Instancias reservadas**: Con las instancias reservadas, puede reducir significativamente los costos en comparación con los precios de pago por uso con términos de 1 o 3 años en Windows y Linux (máquinas virtuales).

- **Ventajas híbridas de Azure**: Puede llevar a Azure licencias locales de Windows Server con software Assurance y combinarlas con las opciones de instancias reservadas.

- **Oferta del contrato Enterprise (EA)** : El Contrato Enterprise ofrece ahorros integrados aplicables a su suscripción.

- **Ofertas**: Hay varias ofertas de Azure, por ejemplo, Desarrollo/pruebas - Pago por uso y Desarrollo/pruebas - Enterprise que proporcionan tarifas reducidas para las máquinas virtuales de desarrollo o pruebas.

- **Tiempo de actividad de la máquina virtual**: puede mencionar la duración en días por mes y horas al día en que se ejecutarán las máquinas virtuales de Azure para reducir los costos (no aplicable a RI).

- **Target Region** (Región de destino): Puede crear varias evaluaciones en distintas regiones para compararlas si la migración a una región determinada en una geografía puede ser más rentable.

- **Recomendaciones basadas en el rendimiento**: Como práctica recomendada, intente usar recomendaciones de máquinas virtuales de Azure con derechos que le ayuden a ahorrar costos en la nube.

### <a name="visualize-data"></a>Visualización de datos

Puede ver el informe de evaluación del servidor con la preparación de Azure y la distribución del costo mensual en el portal, así como exportar la evaluación para aplicar más visualizaciones en los datos de detección y evaluación para que el plan de migración sea más completo. Puede crear varias evaluaciones para diferentes combinaciones de propiedades y elegir el conjunto de propiedades que mejor se adapten a su negocio.  

 ![Introducción a las evaluaciones](./media/concepts-migration-planning/assessment-summary.png)

### <a name="evaluate-gaps-and-potential-blockers"></a>Evaluación de brechas y posibles bloqueos

A la hora de determinar las aplicaciones y la infraestructura subyacente que se va a migrar, identifique las restricciones de tiempo de inactividad para estas aplicaciones y busque las dependencias operativas entre las aplicaciones y la infraestructura subyacente. Este análisis puede ayudarle a planear las migraciones que cumplen su objetivo de tiempo de recuperación (RTO) y garantizar una pérdida de datos mínima o nula. Antes de realizar la migración, se recomienda revisar y mitigar los problemas de compatibilidad o las características no admitidas que podrían bloquear la migración de los servidores y las bases de datos SQL con la ayuda del informe de evaluación del servidor y Azure Migrate: Recomendaciones de evaluación de la base de datos.

### <a name="first-workloads-to-target-and-approach"></a>Primeras cargas de trabajo para destino y enfoque

Ahora que tiene toda la información fundamental para tomar la decisión de la migración, debe priorizar qué aplicaciones y cargas de trabajo se deben migrar primero. Desarrolle un enfoque de "aplicación y aprendizaje" para migrar las aplicaciones preparadas de forma sistemática y controlable, de modo que pueda aumentar los defectos de esta estrategia antes de embarcarse en una migración a gran escala. También puede usar factores estratégicos como la complejidad y el tiempo para migrar, la urgencia de la empresa, el entorno de producción/no producción, el cumplimiento y los requisitos de seguridad, el conocimiento de la aplicación, etc. para priorizar los grupos de aplicaciones que se van a migrar.

Algunas de las estrategias de migración recomendadas son:

- **Dé prioridad a sus logros rápidos**: Puede usar los informes de evaluación para identificar frutas de bajo nivel, incluidos los servidores y las bases de datos que están totalmente preparados y requieren un esfuerzo mínimo para migrar a Azure:
    - Preparado para Azure: Exporte el informe de evaluación y filtre todas las máquinas que estén "Preparadas para Azure". Este puede ser el primer grupo de máquinas que puede levantar y desplazar con el Azure Migrate: Herramienta de migración del servidor.
    - Finalización del soporte técnico del sistema operativo: Exporte el informe de evaluación y filtre todas las máquinas que ejecutan los sistemas operativos Windows Server 2008 y Windows Server 2008 R2. Estas SKU son el fin del soporte técnico y solo Azure proporciona actualizaciones de seguridad gratuitas durante 3 años al migrarlas a Azure. Al combinar, Ventaja híbrida de Azure y usar instancias reservadas, el almacenamiento podría ser mucho más alto.
    - Migración de SQL Server: Use las recomendaciones de evaluación de bases de datos para migrar las bases de datos preparadas para las bases de datos de Azure SQL mediante Azure Migrate: La migración de bases de datos y las bases de datos están preparadas para la máquina virtual de Azure SQL mediante Azure Migrate: Server Migration.
    - Finalización del soporte técnico del software: Exporte el inventario de la aplicación y filtre el software o las extensiones que puedan estar llegando al final del soporte técnico. Debe priorizar estas aplicaciones.
    - Máquinas virtuales con aprovisionamiento excesivo: Exporte el informe de evaluación y filtre las máquinas con un uso de CPU bajo (%) y el uso de memoria (%).  Puede usar esta oportunidad para migrar a una máquina virtual con derechos en Azure y ahorrar lo que ha pagado por recursos infrautilizados.
    - Restricciones de capacidad: Exporte el informe de evaluación y filtre las máquinas con un uso de CPU alto (%) y el uso de memoria (%).  Puede evitar que las máquinas virtuales sobrecargadas interrumpan y aumenten el rendimiento mediante la migración a Azure y el uso de la capacidad de escalado automático para satisfacer la demanda. También puede consultar el informe de evaluación para reconocer las restricciones de almacenamiento mediante el análisis de la IOPS y el rendimiento del disco y encontrar el tipo de disco recomendado que mejor se adapte a sus necesidades.

- **Empiece poco a poco y, después, podrá ir por más**: Comience con el traslado de aplicaciones y cargas de trabajo con un riesgo mínimo y menos complejo, para generar confianza en su estrategia de migración. También puede intersectar las recomendaciones de evaluación de Azure Migrate con el repositorio de CMDB de su organización para buscar y migrar cargas de trabajo de entorno de desarrollo/pruebas en las migraciones piloto. Los aprendizajes de estos pilotos se pueden usar al migrar cargas de trabajo de producción.  

- **Cumplir los requisitos reglamentarios y del sector**: Azure mantiene la cartera de cumplimiento más grande en todo el sector, en términos de amplitud y profundidad de las ofertas. Úselo como una oportunidad para priorizar sus migraciones a Azure y cumplir con sus estándares y leyes nacionales, regionales y específicos del sector. Esto es especialmente cierto en el caso de las organizaciones que se ocupan de la información confidencial de la empresa o que están en industrias muy reguladas, en las que los estándares y las regulaciones abundan y en algunos casos pueden cambiar con frecuencia, lo que dificulta la tarea de mantener el ritmo.  

## <a name="finalize-the-migration-planandprepare-formigration"></a>Finalización del plan de migración y preparación para la migración

Antes de finalizar el plan de migración, asegúrese de que estas consideraciones de migración clave no juegan ningún obstáculo en el planeamiento de la migración:

- Evalúe las restricciones de ancho de banda de red y la latencia, lo que puede provocar retrasos imprevistos e interrumpir la velocidad de replicación de la migración.

- Se almacena en un búfer a tiempo para realizar pruebas de rendimiento y aceptación de usuario en las aplicaciones migradas o ejecutar cualquier ajuste de aplicación posterior a la migración, como actualizar las cadenas de conexión de base de datos y las configuraciones de servidor Web, realizar el traslado y la limpieza, etc.

- Revise los permisos de Azure recomendados y los roles de acceso de servidor/base de datos y el modelo de permisos necesarios para la migración.

- Prepare su organización y asegúrese de la alineación de los empleados con la transformación digital. Una base de entrenamiento sólida es importante para el cambio correcto de la organización. Consulte el entrenamiento gratuito disponible en [Microsoft Learn](/learn/azure/?ocid=CM_Discovery_Checklist_PDF), incluidos los cursos sobre aspectos básicos de Azure, arquitectura de la solución y seguridad. También anime a su equipo a explorar la  [certificación de Azure](https://www.microsoft.com/learning/certification-overview.aspx?ocid=CM_Discovery_Checklist_PDF) .  

- Obtenga soporte técnico para su implementación si es necesario. Muchas organizaciones optan por la ayuda externa para respaldar la migración en la nube. Para pasar a Azure de forma rápida y confiable con asistencia personalizada, considere la posibilidad de un  [proveedor de servicios administrados por expertos de Azure](https://www.microsoft.com/solution-providers/search?cacheId=9c2fed4f-f9e2-42fb-8966-4c565f08f11e&ocid=CM_Discovery_Checklist_PDF) o [FastTrack for Azure](https://azure.microsoft.com/programs/azure-fasttrack/?ocid=CM_Discovery_Checklist_PDF).  

Cree un plan de migración efectivo en la nube que incluya información detallada sobre la lista o los grupos de aplicaciones que desea migrar, la disponibilidad de la aplicación y la base de datos y las restricciones de tiempo de inactividad, así como los hitos de migración deseados. El plan de migración también debe tener en cuenta cuánto tiempo tardará la copia de datos e incluir un búfer para las pruebas posteriores a la migración y las actividades de transferencia. Las pruebas posteriores a la migración deben incluir casos de uso funcional, integración, seguridad y pruebas de rendimiento para garantizar que las aplicaciones migradas funcionen según lo previsto y que todos los objetos de base de datos y las relaciones de datos se hayan transferido correctamente a la nube.  

Use este análisis para crear un mapa de ruta de migración y declarar una ventana de mantenimiento para migrar las aplicaciones y bases de datos con un tiempo de inactividad mínimo de cero y limitar el posible impacto operativo o empresarial durante la migración.  

Se recomienda siempre probar y continuar el uso de la capacidad de  *Migración de prueba* de Azure Migrate antes de iniciar migraciones a gran escala en Azure. Estos datos reales le ayudarán a calcular el tiempo real implicado y a realizar los ajustes necesarios en el plan de migración. La migración de prueba también supone una oportunidad para detectar cualquier posible problema y corregirlo antes de pasar a la migración real.  

Una vez que esté listo para migrar, use la herramienta de migración del servidor de *Azure Migrate* y de *servicio de migración de datos* de Azure Migrate para obtener una experiencia de migración sin problemas e integrada con seguimiento de un extremo a otro. La herramienta de migración del servidor es compatible con la migración de máquinas virtuales y servidores hospedados localmente en el centro de centros de usuarios de los clientes o en cualquier otra nube privada o pública, como AWS, GCP, etc., con un tiempo de inactividad nulo. Azure Database Migration Service es un servicio totalmente administrado diseñado para permitir migraciones completas desde varios orígenes de base de datos hasta las plataformas de datos de Azure con un tiempo de inactividad mínimo.  

> [!NOTE]
> En el caso de las máquinas virtuales de VMware, Server Assessment utiliza el sistema operativo especificado para la máquina virtual en vCenter Server para realizar el análisis del sistema operativo invitado. En el caso de las máquinas virtuales de Linux que se ejecutan en VMware, actualmente no se identifica la versión exacta del kernel del sistema operativo invitado.

## <a name="next-steps"></a>Pasos siguientes

- Investigue el  [recorrido de la migración en la nube](/azure/architecture/cloud-adoption/getting-started/migrate)  en el marco de Cloud Adoption Framework para Azure.
- [Introducción](https://youtu.be/wFfq3YPxYHE) a Azure Migrate.
- Creación de una evaluación para [máquinas virtuales de VMware](tutorial-assess-vmware.md) o [máquinas virtuales de Hyper-V](tutorial-assess-hyper-v.md).
