---
title: "Azure Service Fabric コンテナー サービスのネットワーク モードを構成する | Microsoft Docs"
description: "Azure Service Fabric によってサポートされるさまざまなネットワーク モードを設定する方法について説明します。"
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: 
ms.assetid: d552c8cd-67d1-45e8-91dc-871853f44fc6
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 8/9/2017
ms.author: subramar
ms.openlocfilehash: 1dacbbef915580b0095ef588f3dafad35daf1bde
ms.sourcegitcommit: b07d06ea51a20e32fdc61980667e801cb5db7333
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2017
---
# <a name="service-fabric-container-networking-modes"></a>Service Fabric コンテナー ネットワーク モード

コンテナー サービス用の Azure Service Fabric クラスターは、既定では **nat** ネットワーク モードを使います。 複数のコンテナー サービスが同じポートでリッスンしていて、nat モードが使われていると、デプロイ エラーが発生することがあります。 複数のコンテナー サービスが同じポートでリッスンできるようにするため、Service Fabric では **Open** ネットワーク モード (バージョン 5.7 以降) が提供されています。 Open モードでは、各コンテナー サービスに、複数のサービスによる同じポートでのリッスンをサポートする内部的な IP アドレスが動的に割り当てられます。  

サービス マニフェストに静的なエンドポイントを使うコンテナー サービスが 1 つ含まれる場合は、Open モードを使って新しいサービスを作成および削除でき、デプロイ エラーは発生しません。 同じ docker-compose.yml ファイルを静的ポート マッピングで使って、複数のサービスを作成することもできます。

コンテナー サービスが再起動するか、クラスター内の別のノードに移動すると、IP アドレスが変わります。 このため、動的に割り当てられる IP アドレスを使ってコンテナー サービスを検出することはお勧めしません。 サービスの検出には、Service Fabric ネーム サービスまたは DNS サービスのみを使う必要があります。 

>[!WARNING]
>Azure では、仮想ネットワークごとに全部で 4,096 個の IP アドレスを使うことができます。 仮想ネットワーク内のノードの数とコンテナー サービス インスタンス (Open モードを使っているもの) の数の合計が、IP アドレスの数 4,096 を超えることはできません。 高密度シナリオでは、nat ネットワーク モードお勧めします。
>

## <a name="set-up-open-networking-mode"></a>Open ネットワーク モードを設定する

1. Azure Resource Manager テンプレートを設定します。 **fabricSettings** セクションで、DNS サービスと IP プロバイダーを有効にします。 

    ```json
    "fabricSettings": [
                {
                    "name": "DnsService",
                    "parameters": [
                       {
                            "name": "IsEnabled",
                            "value": "true"
                      }
                    ]
                },
                {
                    "name": "Hosting",
                    "parameters": [
                      { 
                            "name": "IPProviderEnabled",
                            "value": "true"
                      }
                    ]
                },
                {
                    "name":  "Trace/Etw", 
                    "parameters": [
                    {
                            "name": "Level",
                            "value": "5"
                    }
                    ]
                },
                {
                    "name": "Setup",
                    "parameters": [
                    {
                            "name": "ContainerNetworkSetup",
                            "value": "true"
                    }
                    ]
                }
            ],
    ```

2. 複数の IP アドレスをクラスターの各ノードに構成できるように、ネットワーク プロファイル セクションを設定します。 次の例は、Windows/Linux Service Fabric クラスター用にノードごとに 5 つの IP アドレスを設定します。 各ノードのポートで 5 つのサービス インスタンスがリッスンできます。

    ```json
    "variables": {
        "nicName": "NIC",
        "vmName": "vm",
        "virtualNetworkName": "VNet",
        "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',variables('virtualNetworkName'))]",
        "vmNodeType0Name": "[toLower(concat('NT1', variables('vmName')))]",
        "subnet0Name": "Subnet-0",
        "subnet0Prefix": "10.0.0.0/24",
        "subnet0Ref": "[concat(variables('vnetID'),'/subnets/',variables('subnet0Name'))]",
        "lbID0": "[resourceId('Microsoft.Network/loadBalancers',concat('LB','-', parameters('clusterName'),'-',variables('vmNodeType0Name')))]",
        "lbIPConfig0": "[concat(variables('lbID0'),'/frontendIPConfigurations/LoadBalancerIPConfig')]",
        "lbPoolID0": "[concat(variables('lbID0'),'/backendAddressPools/LoadBalancerBEAddressPool')]",
        "lbProbeID0": "[concat(variables('lbID0'),'/probes/FabricGatewayProbe')]",
        "lbHttpProbeID0": "[concat(variables('lbID0'),'/probes/FabricHttpGatewayProbe')]",
        "lbNatPoolID0": "[concat(variables('lbID0'),'/inboundNatPools/LoadBalancerBEAddressNatPool')]"
    }
    "networkProfile": {
                "networkInterfaceConfigurations": [
                  {
                    "name": "[concat(parameters('nicName'), '-0')]",
                    "properties": {
                      "ipConfigurations": [
                        {
                          "name": "[concat(parameters('nicName'),'-',0)]",
                          "properties": {
                            "primary": "true",
                            "loadBalancerBackendAddressPools": [
                              {
                                "id": "[variables('lbPoolID0')]"
                              }
                            ],
                            "loadBalancerInboundNatPools": [
                              {
                                "id": "[variables('lbNatPoolID0')]"
                              }
                            ],
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 1)]",
                          "properties": {
                            "primary": "false",
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 2)]",
                          "properties": {
                            "primary": "false",
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 3)]",
                          "properties": {
                            "primary": "false",
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 4)]",
                          "properties": {
                            "primary": "false",
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        },
                        {
                          "name": "[concat(parameters('nicName'),'-', 5)]",
                          "properties": {
                            "primary": "false",
                            "subnet": {
                              "id": "[variables('subnet0Ref')]"
                            }
                          }
                        }
                      ],
                      "primary": true
                    }
                  }
                ]
              }
   ```
 
3. Windows クラスターのみの場合は、次の値で仮想ネットワーク用のポート UDP/53 を開く Azure ネットワーク セキュリティ グループ (NSG) ルールを設定します。

   |設定 |値 | |
   | --- | --- | --- |
   |優先順位 |2000 | |
   |名前 |Custom_Dns  | |
   |ソース |VirtualNetwork | |
   |変換先 | VirtualNetwork | |
   |サービス | DNS (UDP/53) | |
   |[操作] | ALLOW  | |
   | | |

4. 各サービスのアプリケーション マニフェストでネットワーク モードを指定します。`<NetworkConfig NetworkType="Open">` **Open** ネットワーク モードにすると、サービスに専用 IP アドレスが割り当てられます。 モードを指定しないと、サービスは既定で **nat** モードになります。 次のマニフェストの例では、`NodeContainerServicePackage1` サービスと `NodeContainerServicePackage2` サービスはそれぞれ同じポートでリッスンできます (どちらのサービスも `Endpoint1` でリッスンしています)。 Open ネットワーク モードを指定するときは、`PortBinding` 構成を指定できません。

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <ApplicationManifest ApplicationTypeName="NodeJsApp" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <Description>Calculator Application</Description>
      <Parameters>
        <Parameter Name="ServiceInstanceCount" DefaultValue="3"></Parameter>
        <Parameter Name="MyCpuShares" DefaultValue="3"></Parameter>
      </Parameters>
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="NodeContainerServicePackage1" ServiceManifestVersion="1.0"/>
        <Policies>
          <ContainerHostPolicies CodePackageRef="NodeContainerService1.Code" Isolation="hyperv">
           <NetworkConfig NetworkType="Open"/>
          </ContainerHostPolicies>
        </Policies>
      </ServiceManifestImport>
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="NodeContainerServicePackage2" ServiceManifestVersion="1.0"/>
        <Policies>
          <ContainerHostPolicies CodePackageRef="NodeContainerService2.Code" Isolation="default">
            <NetworkConfig NetworkType="Open"/>
          </ContainerHostPolicies>
        </Policies>
      </ServiceManifestImport>
    </ApplicationManifest>
    ```

    Windows クラスターの場合、アプリケーション内のサービス全体で複数のネットワーク モードを混在させ、対応付けることができます。 一部のサービスで Open モードを使い、他のサービスで nat モードを使うことができます。 nat モードを使うようにサービスを構成するときは、サービスがリッスンするポートは一意である必要があります。

    >[!NOTE]
    >Linux クラスターでは、複数のサービスについてネットワーク モードを混在させることはサポートされていません。 
    >

## <a name="next-steps"></a>次のステップ
* [Service Fabric アプリケーション モデルを理解する](service-fabric-application-model.md)
* [Service Fabric サービス マニフェスト リソースについて詳しく学習する](https://docs.microsoft.com/azure/service-fabric/service-fabric-service-manifest-resources)
* [Windows Server 2016 上での Service Fabric への Windows コンテナーのデプロイ](service-fabric-get-started-containers.md)
* [Linux 上での Service Fabric への Docker コンテナーのデプロイ](service-fabric-get-started-containers-linux.md)
