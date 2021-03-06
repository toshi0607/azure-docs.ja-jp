---
title: "StorSimple ジョブの表示と管理 | Microsoft Docs"
description: "StorSimple Manager サービスの [ジョブ] ページと、最近のバックアップ ジョブ、現在のバックアップ ジョブ、スケジュールされたバックアップ ジョブを追跡する方法について説明します。"
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: 
ms.assetid: 55922cd0-d490-48eb-938a-012a67c1c09e
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/03/2017
ms.author: alkohli
ms.openlocfilehash: 426fd3ec76157670c9d192f007adacc4284abb42
ms.sourcegitcommit: 0930aabc3ede63240f60c2c61baa88ac6576c508
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/07/2017
---
# <a name="use-the-storsimple-manager-service-to-view-and-manage-storsimple-jobs"></a>StorSimple Manager サービスを使用して StorSimple ジョブを表示および管理する
> [!NOTE]
> StorSimple のクラシック ポータルは廃止される予定です。 ご使用の StorSimple デバイス マネージャーは、廃止スケジュールに従い、自動的に新しい Azure Portal に移行されます。 この移行に関しては、メールとポータル通知でお知らせします。 このドキュメントも間もなく廃止されます。 この移行についてご質問があれば、[Azure Portal への移行に関する FAQ](storsimple-8000-move-azure-portal-faq.md) に関するページを参照してください。

[!INCLUDE [storsimple-version-selector-manage-jobs](../../includes/storsimple-version-selector-manage-jobs.md)]

## <a name="overview"></a>概要
**[ジョブ]** ページには、StorSimple Manager サービスに接続されているデバイスで開始されたジョブを表示および管理するための一元的なポータルがあります。 複数のデバイスについて、スケジュールされたジョブ、実行中のジョブ、完了したジョブ、および失敗したジョブを確認できます。 結果は表形式で表示されます。

![[ジョブ] ページ](./media/storsimple-manage-jobs/HCS_JobsPage.png)

以下のフィールドにフィルター処理を行うことで、関心のあるジョブを素早く見つけることができます。

* **状態** - ジョブの状態は、実行中、スケジュール済み、失敗、完了、取り消し中、取り消し済みのいずれかです。
* **タイプ** - ジョブは、スケジュール済みまたはオンデマンドのバックアップ (**バックアップの作成**)、複製、デバイスの復元、または更新操作の結果として作成されます。
* **デバイス** - ジョブは、サービスに接続されている特定のデバイスで開始されます。
* **[開始日時] と [終了日時]** - ジョブには、日時の範囲に基づいてフィルターをかけることができます。

フィルター選択されたジョブは、次の属性に基づいて表形式で表示されます。

* **タイプ** - バックアップ、クローン、復元、フェールオーバー、または更新。
* **状態** - 実行中、スケジュール済み、失敗、完了、取り消し中、または取り消し済み。
* **エンティティ** - ジョブは、ボリューム、バックアップ ポリシー、デバイスに関連付けられます。 クローン ジョブはボリュームに関連付けられますが、スケジュール済みのバックアップ ジョブはバックアップ ポリシーに関連付けられます。 デバイス ジョブは、障害復旧 (DR) または復元操作の結果として作成されます。
* **デバイス** - ジョブが開始されたデバイスの名前。
* **開始日** - ジョブが開始された日時。
* **進行状況** – 実行中のジョブの完了率。 完了したジョブの場合、これは常に 100% です。

ジョブの一覧は 30 秒ごとに更新されます。

このページでは、以下のジョブ関連の操作を実行できます。

* ジョブの詳細を表示する
* ジョブを取り消す

## <a name="view-job-details"></a>ジョブの詳細を表示する
任意のジョブの詳細を表示するには、以下の手順を実行します。

#### <a name="to-view-job-details"></a>ジョブの詳細を表示するには
1. **[ジョブ]** ページで適切なフィルターを使用してクエリを実行し、関心のあるジョブを表示します。 完了済み、実行中、または取り消し済みのジョブを検索できます。
2. ジョブを選択します。
3. ページの下部にある **[詳細]**をクリックします。
4. **[ジョブの詳細]** ダイアログ ボックスで、状態、詳細、時間統計、データ統計を確認できます。

## <a name="cancel-a-job"></a>ジョブを取り消す
実行中のジョブを取り消すには、以下の手順を実行します。

### <a name="to-cancel-a-job"></a>ジョブを取り消すには
1. **[ジョブ]** ページで適切なフィルターを使用してクエリを実行し、取り消しを行う実行中のジョブを表示します。
2. 該当するジョブを選択します。
3. ページの下部にある **[キャンセル]**をクリックします。
4. 確認を求められたら、 **[はい]**をクリックします。

このジョブが取り消されました。

## <a name="next-steps"></a>次のステップ
* [StorSimple バックアップ ポリシーの管理方法](storsimple-manage-backup-policies.md)。
* [StorSimple Manager サービスを使用した StorSimple デバイスの管理方法](storsimple-manager-service-administration.md)

