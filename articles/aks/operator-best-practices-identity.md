---
title: Procedimientos recomendados para la administración de identidad
titleSuffix: Azure Kubernetes Service
description: Obtenga información sobre los procedimientos recomendados del operador de clústeres para saber cómo administrar la autenticación y autorización de los clústeres en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: jpalma
author: palma21
ms.openlocfilehash: 0e11f345bfed287be3170df38a909ed24149b754
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88010266"
---
# <a name="best-practices-for-authentication-and-authorization-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para la autenticación y autorización en Azure Kubernetes Service (AKS)

A medida que implementa y mantiene clústeres en Azure Kubernetes Service (AKS), deberá implementar maneras de administrar el acceso a los recursos y servicios. Sin estos controles, las cuentas pueden tener acceso a recursos y servicios que no necesitan. También puede ser difícil realizar un seguimiento del conjunto de credenciales que se usaron para realizar cambios.

Este artículo de procedimientos recomendados se centra en cómo un operador de clústeres puede administrar el acceso y la identidad de los clústeres de AKS. En este artículo aprenderá a:

> [!div class="checklist"]
>
> * Autenticar a los usuarios de clústeres de AKS con Azure Active Directory.
> * Controlar el acceso a los recursos con el control de acceso basado en roles (RBAC) de Kubernetes.
> * Use Azure RBAC para controlar de forma granular el acceso al recurso AKS y a la API Kubernetes a escala, así como al kubeconfig.
> * Use una identidad administrada para autenticar los pods con otros servicios.

## <a name="use-azure-active-directory"></a>Uso de Azure Active Directory

**Guía de procedimientos recomendados**:implementación de clústeres de AKS con integración de Azure AD. Mediante Azure AD se centraliza el componente de administración de identidades. Cualquier cambio en el estado del grupo o cuenta del usuario se actualiza automáticamente al acceder al clúster de AKS. Use roles o ClusterRoles y enlaces, como se expone en la siguiente sección, para definir el ámbito de usuarios o grupos con la mínima cantidad de permisos necesarios.

Los desarrolladores y propietarios de aplicaciones del clúster de Kubernetes necesitan obtener acceso a diferentes recursos. Kubernetes no proporciona una solución de administración de identidades para controlar qué usuarios pueden interactuar con qué recursos. En su lugar, el usuario suele integrar el clúster con una solución de identidades existente. Azure Active Directory (AD) proporciona una solución de administración de identidades para el ámbito empresarial que se puede integrar con los clústeres de AKS.

Con los clústeres integrados de Azure AD en AKS, el usuario crea *Roles* o *ClusterRoles* que definen los permisos de acceso a los recursos. A continuación, *enlaza* los roles a usuarios o grupos de Azure AD. Este control de acceso basado en roles (RBAC) de Kubernetes se describe en la sección siguiente. En el diagrama siguiente se pueden ver la integración de Azure AD y la forma en que controla el acceso a los recursos:

![Autenticación a nivel de clúster para la integración de Azure Active Directory con AKS](media/operator-best-practices-identity/cluster-level-authentication-flow.png)

1. El desarrollador se autentica con Azure AD.
1. El punto de conexión de emisión de tokens de Azure AD emite el token de acceso.
1. El desarrollador realiza una acción con el token de Azure AD, como `kubectl create pod`.
1. Kubernetes valida el token con Azure Active Directory y recupera las pertenencias a grupos del desarrollador.
1. Se aplican las directivas del clúster y el control de acceso basado en rol (RBAC) de Kubernetes.
1. La solicitud del desarrollador es correcta o no en función de la validación anterior de pertenencia a grupos de Azure AD y las directivas y RBAC de Kubernetes.

Para crear un clúster de AKS que use Azure AD, consulte [Integración de Azure Active Directory con AKS][aks-aad].

## <a name="use-kubernetes-role-based-access-control-rbac"></a>Uso del control de acceso basado en roles (RBAC) de Kubernetes

**Guía de procedimientos recomendados**: uso de RBAC de Kubernetes para definir los permisos que los usuarios o grupos tienen en relación con los recursos del clúster. Cree roles y enlaces que asignen la mínima cantidad de permisos necesarios. Realice la integración con Azure AD, para que cualquier cambio en el estado del usuario o la pertenencia al grupo se actualice automáticamente y que el acceso a los recursos del clúster sea como el actual.

En Kubernetes, puede proporcionar un control granular del acceso a los recursos del clúster. Los permisos se definen en el nivel de clúster, así como en espacios de nombres específicos. Puede definir qué recursos se pueden administrar y con qué permisos. Estos roles se aplican a usuarios o grupos con un enlace. Para obtener más información sobre los *roles*, *ClusterRoles* y *enlaces*, consulte [Opciones de acceso e identidad en Azure Kubernetes Service (AKS)][aks-concepts-identity].

Por ejemplo, puede crear un rol que conceda acceso completo a los recursos en el espacio de nombres denominado *finance-app*, tal y como se muestra en el siguiente ejemplo de manifiesto de YAML:

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: finance-app-full-access-role
  namespace: finance-app
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
```

A continuación, se crea un elemento RoleBinding que enlaza el usuario de Azure AD *developer1\@contoso.com* con RoleBinding, como se muestra en el siguiente manifiesto de YAML:

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: finance-app-full-access-role-binding
  namespace: finance-app
subjects:
- kind: User
  name: developer1@contoso.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: finance-app-full-access-role
  apiGroup: rbac.authorization.k8s.io
```

Cuando *developer1\@contoso.com* se autentica en el clúster de AKS, se tienen los permisos completos sobre los recursos en el espacio de nombres *finance-app*. De esta forma, separa y controla el acceso a los recursos de forma lógica. RBAC de Kubernetes debe usarse junto con la integración de Azure AD, como se describe en la sección anterior.

Para ver cómo usar los grupos de Azure AD para controlar el acceso a los recursos de Kubernetes mediante RBAC, consulte [Controlar el acceso a recursos de clúster mediante el control de acceso basado en roles y las identidades de Azure Active Directory en AKS][azure-ad-rbac].

## <a name="use-azure-rbac"></a>Uso de Azure RBAC 
**Guía de procedimientos recomendados**: use Azure RBAC para definir los permisos mínimos necesarios que los usuarios o grupos tienen para AKS recursos en una o más suscripciones.

Hay dos niveles de acceso necesarios para operar completamente un clúster de AKS: 
1. Acceda al recurso de AKS en la suscripción de Azure. Este nivel de acceso permite controlar las cosas que escalan o actualizan el clúster mediante las API de AKS, así como la extracción de kubeconfig.
Para ver cómo controlar el acceso al recurso AKS y kubeconfig, consulte [Limitar el acceso al archivo de configuración del clúster](control-kubeconfig-access.md).

2. Acceso al API de Kubernetes. Este nivel de acceso se controla mediante [Kubernetes RBAC](#use-kubernetes-role-based-access-control-rbac) (originalmente) o mediante la integración de Azure RBAC con AKS para la autorización de Kubernetes.
Para ver cómo conceder permisos de forma granular a la API de Kubernetes con Azure RBAC, consulte [Uso de Azure RBAC para la autorización de Kubernetes](manage-azure-rbac.md).

## <a name="use-pod-identities"></a>Uso de identidades de pod

**Guía de procedimientos recomendados**: no use credenciales fijas dentro de pods o imágenes de contenedor, ya que corren riesgo de exposición o abuso. En su lugar, use identidades de pod para solicitar acceso automáticamente mediante una solución central de identidades de Azure AD. Las identidades de pod están pensadas para su uso únicamente con pods de Linux y las imágenes de contenedor.

Cuando los pods necesitan obtener acceso a otros servicios de Azure, como Cosmos DB, Key Vault o Blob Storage, el pod necesita credenciales de acceso. Estas credenciales de acceso pueden definirse con la imagen de contenedor o insertadas como un secreto de Kubernetes, pero es necesario crearlas y asignarlas manualmente. A menudo, las credenciales se reutilizan en los pods y no se rotan de forma periódica.

Las identidades administradas para los recursos de Azure (actualmente se implementan como un proyecto de código abierto AKS asociado) permiten automáticamente solicitar acceso a los servicios a través de Azure AD. No defina manualmente las credenciales de los pods; en su lugar, solicite un token de acceso en tiempo real y podrá usarlo para acceder solo a los servicios asignados. En AKS, se implementan dos componentes mediante el operador de clústeres para permitir que los pods usen identidades administradas:

* **El servidor de identidad de administración de nodos (NMI)** es un pod que se ejecuta como DaemonSet en cada nodo del clúster de AKS. El servidor de NMI escucha las solicitudes de pods para servicios de Azure.
* **El controlador de identidades administradas (MIC)** es un pod central con permisos para consultar el servidor de API de Kubernetes y comprueba si hay una asignación de identidades de Azure que corresponde a un pod.

Cuando los pods solicitan acceso a un servicio de Azure, las reglas de red redirigen el tráfico al servidor de identidad de administración de nodos (NMI). El servidor de NMI identifica los pods que solicitan acceso a servicios de Azure en función de su dirección remota y consulta el controlador de identidades administradas (MIC). MIC comprueba si hay asignaciones de identidades de Azure en el clúster de AKS, y el servidor de NMI, a continuación, solicita un token de acceso de Azure Active Directory (AD) en función de la asignación de identidades del pod. Azure AD proporciona acceso al servidor de NMI, que se devuelve al pod. El pod puede usar este token de acceso para, después, solicitar acceso a los servicios de Azure.

En el ejemplo siguiente, un desarrollador crea un pod que usa una identidad administrada para solicitar acceso a Azure SQL Database:

![Las identidades de pod permiten que un pod solicite acceso a otros servicios automáticamente.](media/operator-best-practices-identity/pod-identities.png)

1. Primero, el operador de clústeres crea una cuenta de servicio que se puede usar para asignar identidades cuando los pods solicitan acceso a los servicios.
1. El servidor de NMI y MIC se implementan para retransmitir las solicitudes de pod de tokens de acceso a Azure AD.
1. Un desarrollador implementa un pod con una identidad administrada que solicita un token de acceso a través del servidor de NMI.
1. El token se devuelve al pod y se utiliza para obtener acceso a Azure SQL Database.

> [!NOTE]
> Las identidades administradas del pod conforman un proyecto de código abierto y no es compatible con el soporte técnico de Azure.

Para usar las identidades de pod, consulte [Identidades de Azure Active Directory para aplicaciones de Kubernetes][aad-pod-identity].

## <a name="next-steps"></a>Pasos siguientes

Este artículo de procedimientos recomendados se centra en la autenticación y autorización para el clúster y los recursos. Para implementar algunos de estos procedimientos recomendados, consulte los artículos siguientes:

* [Integración de Azure Active Directory con AKS][aks-aad]
* [Uso de identidades administradas para recursos de Azure con AKS][aad-pod-identity]

Para obtener más información acerca de las operaciones de clúster en AKS, consulte los siguientes procedimientos recomendados:

* [Aislamiento de clúster y de multiinquilinato][aks-best-practices-cluster-isolation]
* [Características básicas del programador de Kubernetes][aks-best-practices-scheduler]
* [Características avanzadas del programador de Kubernetes][aks-best-practices-advanced-scheduler]

<!-- EXTERNAL LINKS -->
[aad-pod-identity]: https://github.com/Azure/aad-pod-identity

<!-- INTERNAL LINKS -->
[aks-concepts-identity]: concepts-identity.md
[aks-aad]: azure-ad-integration-cli.md
[managed-identities:]: ../active-directory/managed-identities-azure-resources/overview.md
[aks-best-practices-scheduler]: operator-best-practices-scheduler.md
[aks-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[aks-best-practices-cluster-isolation]: operator-best-practices-cluster-isolation.md
[azure-ad-rbac]: azure-ad-rbac.md
