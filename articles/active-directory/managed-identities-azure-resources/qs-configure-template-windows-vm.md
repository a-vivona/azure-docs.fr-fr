---
title: Configurer des identités managées sur une machine virtuelle Azure à l’aide d’un modèle - Azure AD
description: Instructions détaillées pour configurer des identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant un modèle Azure Resource Manager.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/13/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1412b0ff7703d5bcc9950c80d5ab59b4c33cc7ae
ms.sourcegitcommit: ee8ce2c752d45968a822acc0866ff8111d0d4c7f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/14/2021
ms.locfileid: "113732221"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-templates"></a>Configurer des identités managées pour les ressources Azure sur une machine virtuelle Azure en utilisant des modèles

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Les identités managées pour ressources Azure fournissent automatiquement aux services Azure une identité managée dans Azure Active Directory. Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, sans avoir d’informations d’identification dans votre code.

Dans cet article, en utilisant le modèle de déploiement Azure Resource Manager, vous allez apprendre à effectuer les opérations suivantes d’identités managées pour ressources Azure sur une machine virtuelle Azure :

## <a name="prerequisites"></a>Prérequis

- Si vous n’êtes pas familiarisé avec l’utilisation d’un modèle de déploiement Azure Resource Manager, voir la [section Vue d’ensemble](overview.md). **Veillez à consulter la [différence entre les identités managées affectées par le système et celles affectées par l’utilisateur](overview.md#managed-identity-types)** .
- Si vous n’avez pas encore de compte Azure, [inscrivez-vous à un essai gratuit](https://azure.microsoft.com/free/) avant de continuer.

## <a name="azure-resource-manager-templates"></a>Modèles Microsoft Azure Resource Manager

Comme pour le portail Azure et le script, les modèles [Azure Resource Manager](../../azure-resource-manager/management/overview.md) offrent la possibilité de déployer des ressources nouvelles ou modifiées définies par un groupe de ressources Azure. Plusieurs options sont disponibles pour la modification du modèle et le déploiement, à la fois localement et sur le portail, y compris :

   - Utiliser un [modèle personnalisé à partir de Place de marché Azure](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template), lequel vous permet de créer un modèle à partir de zéro, ou à partir d’un modèle commun existant ou d’un [modèle de démarrage rapide](https://azure.microsoft.com/resources/templates/).
   - Dériver à partir d’un groupe de ressources existant, en exportant un modèle à partir du [déploiement d’origine](../../azure-resource-manager/templates/export-template-portal.md), ou à partir de l’[état actuel du déploiement](../../azure-resource-manager/templates/export-template-portal.md).
   - Utilisation d’un [éditeur local JSON (VS Code, par exemple)](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md), puis téléchargement/déploiement à l’aide de PowerShell ou Azure CLI.
   - Utilisez le [projet de groupe de ressources Azure](../../azure-resource-manager/templates/create-visual-studio-deployment-project.md) de Visual Studio pour créer et déployer un modèle.

Quelle que soit l’option choisie, la syntaxe de modèle est identique lors du déploiement initial et lors du redéploiement. L’activation d’une identité managée affectée par le système ou par l’utilisateur sur une machine virtuelle s’effectue de la même manière, que la machine soit nouvelle ou existante. De plus, par défaut, Azure Resource Manager effectue une [mise à jour incrémentielle](../../azure-resource-manager/templates/deployment-modes.md) au niveau des déploiements.

## <a name="system-assigned-managed-identity"></a>Identité managée affectée par le système

Dans cette section, vous allez activer et désactiver une identité managée affectée par le système en utilisant un modèle Azure Resource Manager.

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm-or-on-an-existing-vm"></a>Activer une identité managée affectée par le système pendant la création d’une machine virtuelle Azure ou sur une machine virtuelle existante

Pour activer l’identité managée affectée par le système sur une machine virtuelle, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Si vous vous connectez à Azure localement ou via le portail Azure, utilisez un compte associé à l’abonnement Azure qui contient l’ordinateur virtuel.

2. Pour activer l’identité managée affectée par le système, chargez le modèle dans un éditeur, recherchez la ressource `Microsoft.Compute/virtualMachines` qui vous intéresse dans la section `resources`, puis ajoutez la propriété `"identity"` au même niveau que la propriété `"type": "Microsoft.Compute/virtualMachines"`. Utilisez la syntaxe suivante :

   ```json
   "identity": {
       "type": "SystemAssigned"
   },
   ```

3. Lorsque vous avez terminé, les sections ci-après doivent être ajoutées à la section `resource` de votre modèle et doivent ressembler à ce qui suit :

   ```json
    "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2018-06-01",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "[variables('vmName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "SystemAssigned",
                }                        
        }
    ]
   ```

### <a name="assign-a-role-the-vms-system-assigned-managed-identity"></a>Attribuer un rôle à l’identité managée affectée par le système de la machine virtuelle

Après avoir activé une identité managée affectée par le système sur votre machine virtuelle, vous pouvez lui attribuer un rôle, comme l’accès **Lecteur** au groupe de ressources dans lequel elle a été créée. Vous trouverez des informations détaillées pour vous aider dans cette étape dans l’article [Attribuer des rôles Azure avec des modèles Azure Resource Manager](../../role-based-access-control/role-assignments-template.md).


### <a name="disable-a-system-assigned-managed-identity-from-an-azure-vm"></a>Désactiver une identité managée affectée par le système d’une machine virtuelle Azure

Pour supprimer l’identité managée affectée par le système d’une machine virtuelle, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Si vous vous connectez à Azure localement ou via le portail Azure, utilisez un compte associé à l’abonnement Azure qui contient l’ordinateur virtuel.

2. Chargez le modèle dans un [éditeur](#azure-resource-manager-templates) et localisez la ressource `Microsoft.Compute/virtualMachines` qui vous intéresse dans la section `resources`. Si votre machine virtuelle dispose uniquement d’une identité managée affectée par le système, vous pouvez la désactiver en remplaçant le type d’identité par `None`.

   **API Microsoft.Compute/virtualMachines version 2018-06-01**

   Si votre machine virtuelle comporte des identités managées affectées par le système et par l’utilisateur, supprimez `SystemAssigned` du type d’identité, et conservez `UserAssigned` avec les valeurs du dictionnaire `userAssignedIdentities`.

   **API Microsoft.Compute/virtualMachines version 2018-06-01**

   Si votre `apiVersion` présente la valeur `2017-12-01` et que votre machine virtuelle comporte des identités managées affectées par le système et par l’utilisateur, supprimez `SystemAssigned` du type d’identité, et conservez `UserAssigned` avec le tableau `identityIds` des identités managées affectées par l’utilisateur.

L’exemple suivant montre comment supprimer une identité managée affectée par le système sur une machine virtuelle sans identité affectée par l’utilisateur :

```json
{
    "apiVersion": "2018-06-01",
    "type": "Microsoft.Compute/virtualMachines",
    "name": "[parameters('vmName')]",
    "location": "[resourceGroup().location]",
    "identity": {
        "type": "None"
    }
}
```

## <a name="user-assigned-managed-identity"></a>Identité managée affectée par l’utilisateur

Dans cette section, vous allez attribuer une identité managée affectée par l’utilisateur à une machine virtuelle Azure en utilisant un modèle Azure Resource Manager.

> [!NOTE]
> Pour créer une identité managée affectée par l’utilisateur en utilisant un modèle Azure Resource Manager, consultez [Créer une identité managée affectée par l’utilisateur](how-to-manage-ua-identity-arm.md#create-a-user-assigned-managed-identity).

### <a name="assign-a-user-assigned-managed-identity-to-an-azure-vm"></a>Attribuer une identité managée affectée par l’utilisateur à une machine virtuelle Azure

Pour affecter une identité managée affectée par l’utilisateur à une machine virtuelle, votre compte a besoin de l’affectation de rôle [Opérateur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) et [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Sous l’élément `resources`, ajoutez l’entrée suivante pour attribuer une identité managée affectée par l’utilisateur à votre machine virtuelle.  Veillez à remplacer `<USERASSIGNEDIDENTITY>` par le nom de l’identité managée affectée par l’utilisateur que vous avez créée.

   **API Microsoft.Compute/virtualMachines version 2018-06-01**

   Si votre `apiVersion` est `2018-06-01`, vos identités managées affectées par l’utilisateur sont stockées au format de dictionnaire `userAssignedIdentities`, et la valeur `<USERASSIGNEDIDENTITYNAME>` doit être stockée dans une variable définie dans la section `variables` de votre modèle.

   ```json
    {
        "apiVersion": "2018-06-01",
        "type": "Microsoft.Compute/virtualMachines",
        "name": "[variables('vmName')]",
        "location": "[resourceGroup().location]",
        "identity": {
            "type": "userAssigned",
            "userAssignedIdentities": {
                "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]": {}
            }
        }
    }
   ```

   **API Microsoft.Compute/virtualMachines version 2017-12-01**

   Si votre `apiVersion` est `2017-12-01`, vos identités managées affectées par l’utilisateur sont stockées dans le tableau `identityIds`, et la valeur `<USERASSIGNEDIDENTITYNAME>` doit être stockée dans une variable définie dans la section `variables` de votre modèle.

   ```json
   {
       "apiVersion": "2017-12-01",
       "type": "Microsoft.Compute/virtualMachines",
       "name": "[variables('vmName')]",
       "location": "[resourceGroup().location]",
       "identity": {
           "type": "userAssigned",
           "identityIds": [
               "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]"
           ]
       }
   }
   ```

3. Lorsque vous avez terminé, les sections ci-après doivent être ajoutées à la section `resource` de votre modèle et doivent ressembler à ce qui suit :

   **API Microsoft.Compute/virtualMachines version 2018-06-01**

   ```json
     "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2018-06-01",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "[variables('vmName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "userAssigned",
                "userAssignedIdentities": {
                   "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]": {}
                }
            }
        }
    ] 
   ```

   **API Microsoft.Compute/virtualMachines version 2017-12-01**

   ```json
   "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2017-12-01",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "[variables('vmName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "userAssigned",
                "identityIds": [
                   "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]"
                ]
            }
        }
   ]
   ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>Supprimer une identité managée affectée par l’utilisateur d’une machine virtuelle Azure

Pour supprimer une identité affectée par l’utilisateur d’une machine virtuelle, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor). Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Si vous vous connectez à Azure localement ou via le portail Azure, utilisez un compte associé à l’abonnement Azure qui contient l’ordinateur virtuel.

2. Chargez le modèle dans un [éditeur](#azure-resource-manager-templates) et localisez la ressource `Microsoft.Compute/virtualMachines` qui vous intéresse dans la section `resources`. Si votre machine virtuelle dispose uniquement d’une identité managée affectée par l’utilisateur, vous pouvez la désactiver en remplaçant le type d’identité par `None`.

   L’exemple suivant montre comment supprimer toutes les identités managées affectées par l’utilisateur d’une machine virtuelle sans identité managée affectée par le système :

   ```json
    {
      "apiVersion": "2018-06-01",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "identity": {
          "type": "None"
          },
    }
   ```

   **API Microsoft.Compute/virtualMachines version 2018-06-01**

   Pour supprimer d’une machine virtuelle une seule identité managée affectée par l’utilisateur, supprimez-la du dictionnaire `useraAssignedIdentities`.

   Si vous disposez d’une identité managée affectée par le système, conservez-la dans la valeur `type`, sous la valeur `identity`.

   **API Microsoft.Compute/virtualMachines version 2017-12-01**

   Pour supprimer d’une machine virtuelle une seule identité managée affectée par l’utilisateur, supprimez-la du tableau `identityIds`.

   Si vous disposez d’une identité managée affectée par le système, conservez-la dans la valeur `type`, sous la valeur `identity`.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des identités managées pour ressources Azure](overview.md).
