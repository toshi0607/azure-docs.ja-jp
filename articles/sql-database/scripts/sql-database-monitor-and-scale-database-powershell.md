---
title: "PowerShell サンプル - モニター - スケーリング - Azure SQL データベース | Microsoft Docs"
description: "単一 Azure SQL データベースを監視し、スケーリングするための Azure PowerShell サンプル スクリプト"
services: sql-database
documentationcenter: sql-database
author: janeng
manager: jstrauss
editor: carlrab
tags: azure-service-management
ms.assetid: 
ms.service: sql-database
ms.custom: monitor & tune, mvc
ms.devlang: PowerShell
ms.topic: sample
ms.tgt_pltfrm: sql-database
ms.workload: database
ms.date: 06/23/2017
ms.author: janeng
ms.openlocfilehash: 123e6c4f0f4fdde46199d80fdbc2db9c78f69f5f
ms.sourcegitcommit: 4ac89872f4c86c612a71eb7ec30b755e7df89722
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/07/2017
---
# <a name="use-powershell-to-monitor-and-scale-a-single-sql-database"></a>PowerShell を使用して単一の SQL データベースを監視およびスケーリングする

この PowerShell のサンプル スクリプトは、データベースのパフォーマンス メトリックを監視し、そのデータベースを上位のパフォーマンス レベルにスケーリングして、パフォーマンス メトリックの 1 つに警告ルールを作成します。 

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>サンプル スクリプト

[!code-powershell[main](../../../powershell_scripts/sql-database/monitor-and-scale-database/monitor-and-scale-database.ps1?highlight=13-14 "Monitor and scale single SQL Database")]

## <a name="clean-up-deployment"></a>デプロイのクリーンアップ

スクリプト サンプルの実行後は、次のコマンドを使用してリソース グループとすべての関連リソースを削除することができます。

```powershell
Remove-AzureRmResourceGroup -ResourceGroupName $resourcegroupname
```

## <a name="script-explanation"></a>スクリプトの説明

このスクリプトでは、次のコマンドを使用します。 表内の各コマンドは、それぞれのドキュメントにリンクされています。

| コマンド | メモ |
|---|---|
 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | すべてのリソースを格納するリソース グループを作成します。 |
| [New-AzureRmSqlServer](/powershell/module/azurerm.sql/new-azurermsqlserver) | データベースまたはエラスティック プールをホストする論理サーバーを作成します。 |
| [Get-AzureRmMetric](/powershell/module/azurerm.insights/get-azurermmetric) | データベース サイズの使用量に関する情報を表示します。|
| [Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase) | データベースのプロパティを更新するか、エラスティック プールに対して、エラスティック プールから、またはエラスティック プール間でデータベースを移動します。 |
| [Add-AzureRMMetricAlertRule](/powershell/module/azurerm.insights/add-azurermmetricalertrule) | DTU を今後自動的に監視するアラート ルールを設定します。 |
| [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 入れ子になったリソースすべてを含むリソース グループを削除します。 |
|||

## <a name="next-steps"></a>次のステップ

Azure PowerShell の詳細については、[Azure PowerShell のドキュメント](/powershell/azure/overview)を参照してください。

その他の SQL Database 用の PowerShell サンプル スクリプトは、[Azure SQL Database 用の PowerShell スクリプト](../sql-database-powershell-samples.md)のページにあります。
