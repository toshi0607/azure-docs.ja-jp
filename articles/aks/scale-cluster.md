---
title: "Azure Container Service (AKS) クラスターのスケーリング"
description: "Azure Container Service (AKS) クラスターをスケーリングします。"
services: container-service
author: gabrtv
manager: timlt
ms.service: container-service
ms.topic: article
ms.date: 11/15/2017
ms.author: gamonroy
ms.custom: mvc
ms.openlocfilehash: 299eb74686f00dc6d5eb9a1c6127aa134dcd9b77
ms.sourcegitcommit: 5d3e99478a5f26e92d1e7f3cec6b0ff5fbd7cedf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/06/2017
---
# <a name="scale-an-azure-container-service-aks-cluster"></a>Azure Container Service (AKS) クラスターのスケーリング

AKS クラスターをさまざまな数のノードに容易にスケーリングできます。  目的のノードの数を選択し、`az aks scale` コマンドを実行します。  スケール ダウンの際、実行中のアプリケーションへの中断を最小限に抑えるために、ノードは慎重に[切断およびドレインされます](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)。  スケール アップの際、ノードが Kubernetes クラスターによって `Ready` としてマークされるまで、`az` コマンドは待機します。

## <a name="scale-the-cluster-nodes"></a>クラスター ノードのスケーリング

`az aks scale` コマンドを使用してクラスター ノードをスケーリングします。 次の例では、*myK8SCluster* という名前のクラスターを単一のノードにスケーリングします。

```azurecli-interactive
az aks scale --name myK8sCluster --resource-group myResourceGroup --node-count 1
```

出力:

```json
{
  "id": "/subscriptions/4f48eeae-9347-40c5-897b-46af1b8811ec/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myK8sCluster",
  "location": "eastus",
  "name": "myK8sCluster",
  "properties": {
    "accessProfiles": {
      "clusterAdmin": {
        "kubeConfig": "..."
      },
      "clusterUser": {
        "kubeConfig": "..."
      }
    },
    "agentPoolProfiles": [
      {
        "count": 1,
        "dnsPrefix": null,
        "fqdn": null,
        "name": "myK8sCluster",
        "osDiskSizeGb": null,
        "osType": "Linux",
        "ports": null,
        "storageProfile": "ManagedDisks",
        "vmSize": "Standard_D2_v2",
        "vnetSubnetId": null
      }
    ],
    "dnsPrefix": "myK8sClust-myResourceGroup-4f48ee",
    "fqdn": "myk8sclust-myresourcegroup-4f48ee-406cc140.hcp.eastus.azmk8s.io",
    "kubernetesVersion": "1.7.7",
    "linuxProfile": {
      "adminUsername": "azureuser",
      "ssh": {
        "publicKeys": [
          {
            "keyData": "..."
          }
        ]
      }
    },
    "provisioningState": "Succeeded",
    "servicePrincipalProfile": {
      "clientId": "e70c1c1c-0ca4-4e0a-be5e-aea5225af017",
      "keyVaultSecretRef": null,
      "secret": null
    }
  },
  "resourceGroup": "myResourceGroup",
  "tags": null,
  "type": "Microsoft.ContainerService/ManagedClusters"
}
```

## <a name="next-steps"></a>次のステップ

AKS のデプロイと管理の詳細を AKS チュートリアルで確認してください。

> [!div class="nextstepaction"]
> [AKS チュートリアル](./tutorial-kubernetes-prepare-app.md)
