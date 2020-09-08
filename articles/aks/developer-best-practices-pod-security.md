---
title: 'Procedimientos recomendados para desarrolladores: seguridad de pods en Azure Kubernetes Service (AKS)'
description: Obtenga información acerca de los procedimientos recomendados para desarrolladores para la protección de pods en Azure Kubernetes Services (AKS)
services: container-service
ms.topic: conceptual
ms.date: 07/28/2020
ms.author: zarhoads
ms.openlocfilehash: fab4943cad1a87bda70a4c4332ab6135ed99bf1b
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022282"
---
# <a name="best-practices-for-pod-security-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para la seguridad de pods en Azure Kubernetes Service (AKS)

A la hora de desarrollar y ejecutar aplicaciones en Azure Kubernetes Service (AKS), es importante tener en cuenta la seguridad de los pods. Las aplicaciones se deben diseñar con el principio del número mínimo de privilegios necesarios. Mantener seguros los datos privados es prioritario para los clientes. No deseará que credenciales como las cadenas de conexión de base de datos, las claves o los secretos y los certificados se vean expuestas al mundo exterior, donde un atacante podría aprovechar esos secretos con propósitos malintencionados. No debe incorporarlas al código ni insertarlas en las imágenes de contenedor. Este enfoque crearía un riesgo de exposición y limitaría la posibilidad de rotar esas credenciales, ya que las imágenes de contenedor se deberían volver a generar.

Este artículo de procedimientos recomendados se centra en cómo proteger los pods en AKS. Aprenderá a:

> [!div class="checklist"]
> * Usar el contexto de seguridad del pod para limitar el acceso a los procesos y los servicios o una elevación de privilegios
> * Autenticarse con otros recursos de Azure mediante identidades administradas del pod
> * Solicitar y recuperar credenciales de un almacén digital, como Azure Key Vault

También puede consultar los procedimientos recomendados sobre la [seguridad del clúster][best-practices-cluster-security] y la [administración de imágenes de contenedor][best-practices-container-image-management].

## <a name="secure-pod-access-to-resources"></a>Protección del acceso del pod a los recursos

**Guía de procedimiento recomendado**: para la ejecución como un usuario o grupo diferentes y limitar el acceso a los procesos y servicios del nodo subyacente, defina la configuración del contexto de seguridad del pod. Asigne el menor número de privilegios necesarios.

Para que las aplicaciones se ejecuten correctamente, los pods se deben ejecutar como un usuario o grupo definidos y no como *root*. El elemento `securityContext` de un pod o un contenedor permite definir valores de configuración como *runAsUser* o *fsGroup* para asumir los permisos adecuados. Asigne solo los permisos de usuario o grupo necesarios y no utilice el contexto de seguridad como un medio para asumir permisos adicionales. El *runAsUser*, elevación de privilegios y otros valores de configuración de las capacidades de Linux solo están disponibles en los pods y los nodos de Linux.

Cuando se ejecuta como un usuario que no es root, los contenedores no se pueden enlazar a los puertos con privilegios inferiores al 1024. En este escenario, los servicios de Kubernetes se pueden utilizar para ocultar el hecho de que una aplicación se ejecuta en un puerto determinado.

Un contexto de seguridad del pod también puede definir funcionalidades o permisos adicionales para el acceso a los procesos y servicios. Se pueden establecer las siguientes definiciones de contexto de seguridad comunes:

* **allowPrivilegeEscalation** define si el pod puede asumir privilegios de usuario *root*. Diseñe las aplicaciones de modo que este valor siempre esté establecido en *false*.
* **Funcionalidades de Linux**: permite el acceso del pod a los procesos de nodo subyacente. Tenga cuidado con la asignación de estas funcionalidades. Asigne el menor número de privilegios necesarios. Para más información, consulte las [funcionalidades de Linux][linux-capabilities].
* **Etiquetas SELinux** es un módulo de seguridad del kernel de Linux que le permite definir directivas para el acceso a los servicios, los procesos y el sistema de archivos. De nuevo, asigne el menor número de privilegios necesarios. Para más información, consulte las [opciones de SELinux disponibles en Kubernetes][selinux-labels].

El siguiente ejemplo de manifiesto YAML de pod establece la configuración del contexto de seguridad para definir:

* El pod se ejecuta con el identificador de usuario *1000* y es parte del grupo con el identificador *2000*
* No se pueden elevar los privilegios para usar `root`
* Permite a las funcionalidades de Linux acceder a las interfaces de red y al reloj en tiempo real del host (hardware)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    fsGroup: 2000
  containers:
    - name: security-context-demo
      image: nginx:1.15.5
      securityContext:
        runAsUser: 1000
        allowPrivilegeEscalation: false
        capabilities:
          add: ["NET_ADMIN", "SYS_TIME"]
```

Colabore con el operador del clúster para determinar la configuración de contexto de seguridad que necesita. Intente diseñar las aplicaciones para minimizar los permisos adicionales y el acceso que requiere el pod. Hay características de seguridad adicionales para limitar el acceso mediante AppArmor y seccomp (informática segura) que los operadores del clúster pueden implementar. Para más información, consulte [Protección del acceso del contenedor a los recursos][apparmor-seccomp].

## <a name="limit-credential-exposure"></a>Limitación de la exposición de credenciales

**Guía de procedimiento recomendado**: no defina las credenciales en el código de aplicación. Use identidades administradas para los recursos de Azure para permitir que el pod solicite acceso a otros recursos. Se debería usar también un almacén digital, como Azure Key Vault, para almacenar y recuperar las credenciales y claves digitales. Las identidades administradas de pod está pensadas para usar únicamente con pods de Linux y las imágenes de contenedor.

Para limitar el riesgo de exposición de las credenciales en el código de la aplicación, evite el uso de credenciales compartidas o fijas. Las credenciales y las claves no se deberían incluir directamente en el código. Si se exponen estas credenciales, la aplicación se debe actualizar y volver a implementar. Un enfoque mejor es dar a los pods sus propias identidades y formas para autenticarse a sí mismos o recuperar automáticamente las credenciales de un almacén digital.

### <a name="use-azure-container-compute-upstream-projects"></a>Uso de proyectos ascendentes de proceso de contenedor de Azure

> [!IMPORTANT]
> El soporte técnico de Azure no admite los proyectos de código abierto de AKS asociados. Se proporcionan para que los usuarios se ocupen por sí mismos de su instalación en los clústeres y recopilen comentarios de nuestra comunidad.

Los siguientes [proyectos de código abierto de AKS asociados][aks-associated-projects] le permiten autenticar automáticamente los pods o las credenciales de la solicitud y las claves desde un almacén digital. Estos proyectos los mantiene el equipo de flujo ascendente de proceso de contenedor de Azure y forman parte de [una lista más amplia de proyectos disponibles para su uso](https://github.com/Azure/container-compute-upstream/blob/master/README.md#support).

 * [Azure Active Directory Pod Identity][aad-pod-identity]
 * [Proveedor de Azure Key Vault para el controlador de CSI del almacén de secretos](https://github.com/Azure/secrets-store-csi-driver-provider-azure#usage)

#### <a name="use-pod-managed-identities"></a>Uso de identidades administradas del pod

Las identidades administradas de los recursos de Azure permiten a un pod autenticarse a sí mismo en cualquier servicio de Azure que lo permita, como Storage o SQL. Se asigna al pod una identidad de Azure que le permite autenticarse en Azure Active Directory y recibir un token digital. Este token digital se puede presentar a otros servicios de Azure que comprueban si el pod está autorizado para acceder al servicio y realizar las acciones necesarias. Este enfoque significa que no es necesario ningún secreto para las cadenas de conexión de base de datos, por ejemplo. El flujo de trabajo simplificado de la identidad administrada del pod se muestra en el diagrama siguiente:

:::image type="content" source="media/developer-best-practices-pod-security/basic-pod-identity.svg" alt-text="Flujo de trabajo simplificado de la identidad administrada del pod en Azure":::

Con una identidad administrada, el código de la aplicación no necesita incluir credenciales para acceder a un servicio como Azure Storage. Como cada pod se autentica con su propia identidad, puede auditar y revisar el acceso. Si la aplicación se conecta con otros servicios de Azure, utilice identidades administradas para limitar la reutilización de credenciales y el riesgo de exposición.

Para más información acerca de las identidades de los pods, consulte [Configuración de un clúster de AKS para usar identidades administradas de pods con las aplicaciones][aad-pod-identity].

#### <a name="use-azure-key-vault-with-secrets-store-csi-driver"></a>Uso de Azure Key Vault con el controlador de CSI del almacén de secretos

El uso del proyecto de identidades de pods permite la autenticación en los servicios compatibles de Azure. Para sus propios servicios o aplicaciones sin identidades administradas para recursos de Azure, puede seguir autenticándose mediante credenciales o claves. Se puede utilizar un almacén digital para almacenar el contenido de estos secretos.

Cuando las aplicaciones necesitan una credencial, se comunican con el almacén digital, recuperan el contenido de los secretos más reciente y luego se conectan al servicio solicitado. Azure Key Vault puede ser este almacén digital. El flujo de trabajo simplificado para recuperar una credencial de Azure Key Vault mediante identidades administradas de pods se muestra en el diagrama siguiente:

:::image type="content" source="media/developer-best-practices-pod-security/basic-key-vault.svg" alt-text="Flujo de trabajo simplificado para recuperar una credencial de Key Vault mediante una identidad administrada de pods":::

Con Key Vault, puede almacenar y rotar periódicamente secretos como credenciales, claves de cuenta de almacenamiento o certificados. Puede integrar Azure Key Vault con un clúster de AKS mediante el [proveedor de Azure Key Vault para el controlador de CSI del almacén de secretos](https://github.com/Azure/secrets-store-csi-driver-provider-azure#usage). El controlador de CSI del almacén de secretos permite que el clúster de AKS pueda recuperar de forma nativa el contenido de los secretos de Key Vault y proporcionárselo de forma segura solo al pod solicitante. Trabaje con el operador de clúster para implementar el controlador de CSI del almacén de secretos en los nodos de trabajo de AKS. Puede usar una identidad administrada de pods para solicitar acceso a Key Vault y recuperar el contenido de los secretos que necesita mediante el controlador de CSI del almacén de secretos.

Azure Key Vault con el controlador de CSI del almacén de secretos se puede usar para pods y nodos de Linux que requieren una versión la versión 1.16, o superior, de Kubernetes. Para pods y nodos de Windows, se requiere la versión 1.18, o superior, de Kubernetes 1.18.

## <a name="next-steps"></a>Pasos siguientes

Este artículo se centra en cómo proteger los pods. Para implementar algunas de estas áreas, consulte los artículos siguientes:

* [Uso de identidades administradas para recursos de Azure con AKS][aad-pod-identity]
* [Integración de Azure Key Vault con AKS][aks-keyvault-csi-driver]

<!-- EXTERNAL LINKS -->
[aad-pod-identity]: https://github.com/Azure/aad-pod-identity#demo
[aks-keyvault-csi-driver]: https://github.com/Azure/secrets-store-csi-driver-provider-azure#usage
[linux-capabilities]: http://man7.org/linux/man-pages/man7/capabilities.7.html
[selinux-labels]: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#selinuxoptions-v1-core
[aks-associated-projects]: https://awesomeopensource.com/projects/aks?categoryPage=11

<!-- INTERNAL LINKS -->
[best-practices-cluster-security]: operator-best-practices-cluster-security.md
[best-practices-container-image-management]: operator-best-practices-container-image-management.md
[aks-pod-identities]: operator-best-practices-identity.md#use-pod-identities
[apparmor-seccomp]: operator-best-practices-cluster-security.md#secure-container-access-to-resources
