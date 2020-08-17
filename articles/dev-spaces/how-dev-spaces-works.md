---
title: Funcionamiento de Azure Dev Spaces
services: azure-dev-spaces
ms.date: 06/02/2020
ms.topic: conceptual
description: Describe los procesos que posibilitan el funcionamiento de Azure Dev Spaces
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores
ms.openlocfilehash: 40125b07029c78b4e8858d54058aabe038109057
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87837421"
---
# <a name="how-azure-dev-spaces-works"></a>Funcionamiento de Azure Dev Spaces

El desarrollo de una aplicación de Kubernetes puede ser complicado. Para ello, necesita archivos de configuración de Docker y Kubernetes. Además, debe averiguar cómo probar la aplicación localmente e interactuar con otros servicios dependientes. Es posible que también deba controlar el desarrollo y las pruebas en varios servicios a la vez y con un equipo de desarrolladores.

Azure Dev Spaces le ofrece varias formas de iterar y depurar rápidamente las aplicaciones de Kubernetes, así como de colaborar con su equipo. En este artículo se describe lo que puede hacer Azure Dev Spaces y cómo funciona.

## <a name="rapidly-iterate-and-debug-your-kubernetes-application"></a>Iteración y depuración rápidas de la aplicación de Kubernetes

Azure Dev Spaces reduce el trabajo de tener que desarrollar, probar e iterar la aplicación de Kubernetes en el contexto del clúster de AKS. Esta reducción permite a los desarrolladores centrarse en la lógica de negocios de sus aplicaciones en lugar de tener que configurar los servicios para que se ejecuten en Kubernetes.

### <a name="local-process-with-kubernetes"></a>Proceso local con Kubernetes

Con Proceso local con Kubernetes, puede conectar el equipo de desarrollo al clúster de Kubernetes, lo que le permite ejecutar y depurar código en el equipo de desarrollo como si se estuviera ejecutando en el clúster. Azure Dev Spaces redirige el tráfico entre el clúster conectado al ejecutar un pod en el clúster que actúa como agente remoto para redirigir el tráfico entre el equipo de desarrollo y el clúster. Este redireccionamiento del tráfico permite que el código del equipo de desarrollo y los servicios que se ejecutan en el clúster se comuniquen como si estuvieran en el mismo clúster. Para obtener más información sobre cómo conectar el equipo de desarrollo a un clúster de Kubernetes, vea [Funcionamiento de Proceso local con Kubernetes][how-it-works-local-process-kubernetes].

### <a name="run-your-code-in-aks"></a>Ejecución del código en AKS

Además de redirigir el tráfico entre su equipo de desarrollo y su clúster de AKS, con Azure Dev Spaces puede configurar y ejecutar su código rápida y directamente en AKS. Con Visual Studio, Visual Studio Code o la CLI de Azure Dev Spaces, Azure Dev Spaces cargará el código en el clúster y, luego, lo compilará y ejecutará. Azure Dev Spaces también puede sincronizar de forma inteligente los cambios en el código y reiniciar el servicio a fin de reflejar los cambios según sea necesario. Al ejecutar el código, los registros de compilación y los seguimientos HTTP se transmiten de vuelta al cliente para que pueda supervisar el progreso y diagnosticar cualquier problema. También puede usar Azure Dev Spaces para asociar el depurador en Visual Studio y Visual Studio Code a los servicios Java, Node.js y .NET Core. Para obtener más información, consulte [Funcionamiento de la preparación de un proyecto para Azure Dev Spaces][how-it-works-prep], [Funcionamiento de la ejecución del código con Azure Dev Spaces][how-it-works-up]y [Funcionamiento de la depuración remota del código con Azure Dev Spaces][how-it-works-remote-debugging].

## <a name="team-development"></a>Desarrollo en equipo

Azure Dev Spaces permite a los equipos trabajar de forma productiva en su aplicación en el mismo clúster de AKS sin interrupciones.

### <a name="intelligent-routing-between-dev-spaces"></a>Enrutamiento inteligente entre Dev Spaces

Con Azure Dev Spaces, un equipo puede compartir un único clúster de AKS que ejecute una aplicación nativa en la nube y crear espacios de desarrollo aislados en los que el equipo pueda realizar desarrollos, pruebas y depuraciones sin que ello interfiera en los demás espacios de desarrollo. Una versión de línea de base de la aplicación se ejecuta en un espacio de desarrollo raíz. A continuación, los miembros del equipo crean espacios de desarrollo secundarios independientes basados en el espacio raíz para el desarrollo, las pruebas y la depuración de los cambios en la aplicación. Mediante las funcionalidades de enrutamiento de Dev Spaces, los espacios de desarrollo secundarios pueden enrutar las solicitudes entre los servicios que se ejecutan en el espacio de desarrollo secundario y el primario. Este enrutamiento permite a los desarrolladores ejecutar su propia versión de un servicio mientras reutilizan los servicios dependientes del espacio primario. Cada espacio secundario tiene su propia dirección URL única, que se puede compartir con otros usuarios y a la que estos pueden acceder con finalidades de colaboración. Para obtener más información sobre cómo funciona el enrutamiento en Azure Dev Spaces, consulte [Funcionamiento del enrutamiento con Azure Dev Spaces][how-it-works-routing].

### <a name="live-testing-an-open-pull-request"></a>Pruebas dinámicas de una solicitud de incorporación de cambios abierta

También puede usar las Acciones de GitHub con Azure Dev Spaces para probar los cambios de la aplicación en una solicitud de incorporación de cambios directamente en el clúster antes de la combinación. Azure Dev Spaces puede implementar automáticamente una versión de revisión de la aplicación en el clúster, lo que permite al autor y a otros miembros del equipo revisar los cambios en el contexto de toda la aplicación. Con las funcionalidades de enrutamiento de Azure Dev Spaces, esta versión de revisión de la aplicación también se implementa en el clúster sin que ello afecte a otros espacios de desarrollo. Todas estas funcionalidades permiten aprobar y combinar solicitudes de incorporación de cambios de forma confiable. Para ver un ejemplo de Acciones de GitHub y Azure Dev Spaces, consulte [Acciones de GitHub y Azure Kubernetes Service][pr-flow].

## <a name="next-steps"></a>Pasos siguientes

Para empezar a conectar el equipo de desarrollo local al clúster de AKS, consulte [Conexión del equipo de desarrollo a un clúster de AKS][connect].

Para empezar a usar Azure Dev Spaces para el desarrollo en equipo, consulte la guía de inicio rápido [Desarrollo en equipo en Azure Dev Spaces][quickstart-team].

[connect]: https://code.visualstudio.com/docs/containers/local-process-kubernetes
[how-it-works-local-process-kubernetes]: /visualstudio/containers/overview-local-process-kubernetes
[how-it-works-prep]: how-dev-spaces-works-prep.md
[how-it-works-remote-debugging]: how-dev-spaces-works-remote-debugging.md
[how-it-works-routing]: how-dev-spaces-works-routing.md
[how-it-works-up]: how-dev-spaces-works-up.md
[pr-flow]: how-to/github-actions.md
[quickstart-team]: quickstart-team-development.md
[routing]: #team-development