---
title: Referencia de atributos de SuccessFactors de SAP
description: Obtenga información sobre qué atributos de SuccessFactors se admiten con el aprovisionamiento controlado por RR. HH. de SuccessFactors.
services: active-directory
author: cmmdesai
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: reference
ms.workload: identity
ms.date: 08/05/2020
ms.author: chmutali
ms.reviewer: celested
ms.openlocfilehash: ef2da377c7720cfb7b431d1ce0fed56656a2b8c7
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87808514"
---
# <a name="sap-successfactors-attribute-reference"></a>Referencia de atributos de SuccessFactors de SAP

En este artículo se ofrece información sobre:

- [Entidades y atributos admitidos de SuccessFactors](#supported-successfactors-entities-and-attributes)
- [Asignación predeterminada de atributos](#default-attribute-mapping)

## <a name="supported-successfactors-entities-and-attributes"></a>Entidades y atributos admitidos de SuccessFactors

En la tabla siguiente se captura la lista de atributos de SuccessFactors incluidos de manera predeterminada en las dos aplicaciones de aprovisionamiento siguientes:

- [Aprovisionamiento de usuarios de SuccessFactors a Active Directory](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md)
- [Aprovisionamiento de usuarios de SuccessFactors a Azure AD](../saas-apps/sap-successfactors-inbound-provisioning-cloud-only-tutorial.md)

Consulte la [referencia de integración de SAP SuccessFactors](./sap-successfactors-integration-reference.md#retrieving-additional-attributes) para extender el esquema de atributos adicionales. 

| \# | Entidad de SuccessFactors                  | Atributo de SuccessFactors     | Tipo de operación |
|----|----------------------------------------|------------------------------|----------------|
| 1  | PerPerson                              | personIdExternal             | Lectura           |
| 2  | PerPerson                              | personId                     | Lectura           |
| 3  | PerPerson                              | perPersonUuid                | Lectura           |
| 4  | PerPersonal                            | DisplayName                  | Lectura           |
| 5  | PerPersonal                            | firstName                    | Lectura           |
| 6  | PerPersonal                            | gender                       | Lectura           |
| 7  | PerPersonal                            | lastName                     | Lectura           |
| 8  | PerPersonal                            | middleName                   | Lectura           |
| 9  | PerPersonal                            | preferredName                | Lectura           |
| 10 | Usuario                                   | addressLine1                 | Lectura           |
| 11 | Usuario                                   | addressLine2                 | Lectura           |
| 12 | Usuario                                   | addressLine3                 | Lectura           |
| 13 | Usuario                                   | businessPhone                | Lectura           |
| 14 | Usuario                                   | cellPhone                    | Lectura           |
| 15 | Usuario                                   | city                         | Lectura           |
| 16 | Usuario                                   | country                      | Lectura           |
| 17 | Usuario                                   | custom01                     | Lectura           |
| 18 | Usuario                                   | custom02                     | Lectura           |
| 19 | Usuario                                   | custom03                     | Lectura           |
| 20 | Usuario                                   | custom04                     | Lectura           |
| 21 | Usuario                                   | custom05                     | Lectura           |
| 22 | Usuario                                   | custom06                     | Lectura           |
| 23 | Usuario                                   | custom07                     | Lectura           |
| 24 | Usuario                                   | custom08                     | Lectura           |
| 25 | Usuario                                   | custom09                     | Lectura           |
| 26 | Usuario                                   | custom10                     | Lectura           |
| 27 | Usuario                                   | custom11                     | Lectura           |
| 28 | Usuario                                   | custom12                     | Lectura           |
| 29 | Usuario                                   | custom13                     | Lectura           |
| 30 | Usuario                                   | custom14                     | Lectura           |
| 31 | Usuario                                   | empId                        | Lectura           |
| 32 | Usuario                                   | homePhone                    | Lectura           |
| 33 | Usuario                                   | jobFamily                    | Lectura           |
| 34 | Usuario                                   | nickname                     | Lectura           |
| 35 | Usuario                                   | state                        | Lectura           |
| 36 | Usuario                                   | timeZone                     | Lectura           |
| 37 | Usuario                                   | username                     | Lectura           |
| 38 | Usuario                                   | zipCode                      | Lectura           |
| 39 | PerPhone                               | areaCode                     | Lectura           |
| 40 | PerPhone                               | countryCode                  | Lectura           |
| 41 | PerPhone                               | extensión                    | Lectura           |
| 42 | PerPhone                               | phoneNumber                  | Lectura           |
| 43 | PerPhone                               | phoneType                    | Lectura           |
| 44 | PerEmail                               | emailAddress                 | Lectura, escritura    |
| 45 | PerEmail                               | emailType                    | Lectura           |
| 46 | EmpEmployment                          | firstDateWorked              | Lectura           |
| 47 | EmpEmployment                          | lastDateWorked               | Lectura           |
| 48 | EmpEmployment                          | userId                       | Lectura           |
| 49 | EmpEmployment                          | isContingentWorker           | Lectura           |
| 50 | EmpJob                                 | countryOfCompany             | Lectura           |
| 51 | EmpJob                                 | emplStatus                   | Lectura           |
| 52 | EmpJob                                 | endDate                      | Lectura           |
| 53 | EmpJob                                 | startDate                    | Lectura           |
| 54 | EmpJob                                 | jobTitle                     | Lectura           |
| 55 | EmpJob                                 | position                     | Lectura           |
| 65 | EmpJob                                 | customString13               | Lectura           |
| 56 | EmpJob                                 | managerId                    | Lectura           |
| 57 | EmpJob\.BusinessUnit                   | businessUnit                 | Lectura           |
| 58 | EmpJob\.BusinessUnit                   | businessUnitId               | Lectura           |
| 59 | EmpJob\.Company                        | company                      | Lectura           |
| 60 | EmpJob\.Company                        | companyId                    | Lectura           |
| 61 | EmpJob\.Company\.CountryOfRegistration | twoCharCountryCode           | Lectura           |
| 62 | EmpJob\.CostCenter                     | costCenter                   | Lectura           |
| 63 | EmpJob\.CostCenter                     | costCenterId                 | Lectura           |
| 64 | EmpJob\.CostCenter                     | costCenterDescription        | Lectura           |
| 65 | EmpJob\.Department                     | department                   | Lectura           |
| 66 | EmpJob\.Department                     | departmentId                 | Lectura           |
| 67 | EmpJob\.Division                       | division                     | Lectura           |
| 68 | EmpJob\.Division                       | divisionId                   | Lectura           |
| 69 | EmpJob\.JobCode                        | jobCode                      | Lectura           |
| 70 | EmpJob\.JobCode                        | jobCodeId                    | Lectura           |
| 71 | EmpJob\.Location                       | LocationName                 | Lectura           |
| 72 | EmpJob\.Location                       | officeLocationAddress        | Lectura           |
| 73 | EmpJob\.Location                       | officeLocationCity           | Lectura           |
| 74 | EmpJob\.Location                       | officeLocationCustomString4  | Lectura           |
| 75 | EmpJob\.Location                       | officeLocationZipCode        | Lectura           |
| 76 | EmpJob\.PayGrade                       | payGrade                     | Lectura           |
| 77 | EmpEmploymentTermination               | activeEmploymentsCount       | Lectura           |
| 78 | EmpEmploymentTermination               | latestTerminationDate        | Lectura           |

## <a name="default-attribute-mapping"></a>Asignación predeterminada de atributos

En la siguiente tabla se indica la asignación predeterminada de atributos entre los atributos de SuccessFactors enumerados anteriormente y los atributos de AD/Azure AD. En la hoja "Asignación" de la aplicación de aprovisionamiento de Azure AD, puede modificar esta asignación predeterminada para incluir atributos de la lista anterior. 

| \# | Entidad de SuccessFactors                  | Atributo de SuccessFactors | Asignación predeterminada de atributos de AD/Azure AD   | Comentario de procesamiento                                                                            |
|----|----------------------------------------|--------------------------|-----------------------------------------|----------------------------------------------------------------------------------------------|
| 1  | PerPerson                              | personIdExternal         | employeeId                              | Se usa como atributo coincidente                                                                   |
| 2  | PerPerson                              | perPersonUuid            | \[No se asignado \- se usa como delimitador de origen\] | Durante la sincronización inicial, el servicio de aprovisionamiento vincula el atributo personUuid con el atributo objectGuid\ existente.  |
| 3  | PerPersonal                            | DisplayName              | DisplayName                             | N/D                                                                                           |
| 4  | PerPersonal                            | firstName                | givenName                               | N/D                                                                                           |
| 5  | PerPersonal                            | lastName                 | sn                                      | N/D                                                                                           |
| 6  | Usuario                                   | addressLine1             | streetAddress                           | N/D                                                                                           |
| 7  | Usuario                                   | city                     | l                                       | N/D                                                                                           |
| 8  | Usuario                                   | country                  | co                                      | N/D                                                                                           |
| 9  | Usuario                                   | state                    | st                                      | N/D                                                                                           |
| 10 | Usuario                                   | username                 | samAccountName                          | N/D                                                                                           |
| 11 | Usuario                                   | zipCode                  | postalCode                              | N/D                                                                                           |
| 12 | PerEmail                               | emailAddress             | mail                                    | N/D                                                                                           |
| 13 | EmpJob                                 | jobTitle                 | title                                   | N/D                                                                                           |
| 14 | EmpJob                                 | managerId                | manager                                 | N/D                                                                                           |
| 15 | EmpJob\.Company\.CountryOfRegistration | twoCharCountryCode       | c                                       | N/D                                                                                           |
| 16 | EmpJob\.Department                     | department               | department                              | N/D                                                                                           |
| 17 | EmpJob\.Division                       | division                 | company                                 | N/D                                                                                           |
| 18 | EmpJob\.Location                       | officeLocationAddress    | streetAddress                           | N/D                                                                                           |
| 19 | EmpJob\.Location                       | officeLocationZipCode    | postalCode                              | N/D                                                                                           |
| 20 | EmpEmploymentTermination               | activeEmploymentsCount   | accountEnabled                          | Si activeEmploymentsCount = 0, deshabilita la cuenta.                                           |
