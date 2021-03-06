---
title: "Azure Machine Learning でモデル管理の Docker イメージを作成する | Microsoft Docs"
description: "この記事では、Web サービスの Docker イメージを作成する手順について説明します。"
services: machine-learning
author: chhavib
ms.author: chhavib
manager: neerajkh
editor: jasonwhowell
ms.service: machine-learning
ms.workload: data-services
ms.devlang: na
ms.topic: article
ms.date: 09/20/2017
ms.openlocfilehash: 03e51ab298a08386f0094d6d0290aa1ec85d337f
ms.sourcegitcommit: 80eb8523913fc7c5f876ab9afde506f39d17b5a1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2017
---
# <a name="azure-machine-learning-model-management-account-api-reference"></a>Azure Machine Learning モデル管理アカウント API リファレンス

デプロイ環境のセットアップに関する情報は、[モデル管理アカウントのセットアップ](deployment-setup-configuration.md)に関する記事を参照してください。

Azure Machine Learning モデル管理アカウント API には、次の操作が実装されています。

- モデルの登録
- マニフェストの作成
- Docker イメージの作成
- Web サービスの作成

このイメージを使用して、ローカルで、リモートの Azure Container Service クラスターで、または任意の Docker サポート環境で、Web サービスを作成できます。

## <a name="prerequisites"></a>前提条件
[クイックスタートのインストールと作成](quickstart-installation.md)に関するドキュメントで、インストール手順を確認してください。

手順を進める前に、以下が必要です。
1. モデル管理アカウントのプロビジョニング
2. モデルを展開して管理するための環境の作成
3. 機械学習モデル

### <a name="azure-ad-token"></a>Azure AD トークン
Azure CLI を使用する場合は、`az login` を使ってログインします。 CLI では、.azure ファイルの Azure Active Directory (Azure AD) トークンが使用されます。 API を使用する必要がある場合は、次の選択肢があります。

##### <a name="acquire-the-azure-ad-token-manually"></a>Azure AD トークンを手動で取得する
`az login` を使用して、ホーム ディレクトリの .azure ファイルから最新のトークンを取得します。

##### <a name="acquire-the-azure-ad-token-programmatically"></a>プログラムで Azure AD トークンを取得する
```
az ad sp create-for-rbac --scopes /subscriptions/<SubscriptionId>/resourcegroups/<ResourceGroupName> --role Contributor --years <length of time> --name <MyservicePrincipalContributor>
```
サービス プリンシパルを作成した後、出力を保存します。 これを使用して、Azure AD からトークンを取得できます。

```cs
 private static async Task<string> AcquireTokenAsync(string clientId, string password, string authority, string resource)
{
        var creds = new ClientCredential(clientId, password);
        var context = new AuthenticationContext(authority);
        var token = await context.AcquireTokenAsync(resource, creds).ConfigureAwait(false);
        return token.AccessToken;
}
```
トークンは、API 呼び出しの Authorization ヘッダー内に置かれます。


## <a name="register-a-model"></a>モデルを登録する

モデルの登録手順では、作成した Azure モデル管理アカウントを使用して、Machine Learning モデルを登録します。 この登録によって、登録時に割り当てられたモデルとバージョンを追跡できるようになります。 ユーザーがモデルの名前を付けます。 同じ名前でモデルを連続して登録すると、新しいバージョンと ID が生成されます。

### <a name="request"></a>要求

| メソッド | 要求 URI |
|------------|------------|
| POST |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/models 
 |
### <a name="description"></a>説明
モデルを登録します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |
| モデル | body | モデルの登録に使用されるペイロード | あり | [モデル](#model) |


### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | OK です。 モデルの登録が成功しました。 | [モデル](#model) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-models-in-an-account"></a>アカウントにあるモデルの一覧の照会
### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/models 
 |
### <a name="description"></a>説明
アカウントにあるモデルの一覧を照会します。 結果の一覧は、タグおよび名前でフィルター処理できます。 フィルターが渡されない場合、クエリを実行するとアカウントにあるすべてのモデルが一覧表示されます。 返された一覧は改ページ調整され、各ページの項目数は省略可能なパラメーターになっています。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |
| name | クエリ | オブジェクト名 | いいえ | string |
| tag | クエリ | モデルのタグ | いいえ | string |
| count | クエリ | 1 ページに取得する項目の数 | いいえ | string |
| $skipToken | クエリ | 次のページを取得する継続トークン | いいえ | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [PaginatedModelList](#paginatedmodellist) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse) |

## <a name="get-model-details"></a>モデルの詳細の取得
### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/models/{id}  
 |

### <a name="description"></a>説明
ID でモデルを取得します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| id | path | オブジェクト ID | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [モデル](#model) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse) |

## <a name="register-a-manifest-with-the-registered-model-and-all-dependencies"></a>登録されたモデルおよびすべての依存関係と共に、マニフェストを登録する

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| POST |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/manifests | 

### <a name="description"></a>説明
登録されたモデルおよびすべての依存関係と共にマニフェストを登録します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |
| manifestRequest | body | マニフェストの登録に使用されるペイロード | あり | [Manifest](#manifest) |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | マニフェストの登録に成功しました。 | [Manifest](#manifest) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-manifests-in-an-account"></a>アカウントにあるマニフェストの一覧を照会する

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/manifests | 

### <a name="description"></a>説明
アカウントにあるマニフェストの一覧を照会します。 結果の一覧は、モデル ID およびマニフェスト名でフィルター処理できます。 フィルターが渡されない場合、クエリを実行するとアカウントにあるすべてのマニフェストが一覧表示されます。 返された一覧は改ページ調整され、各ページの項目数は省略可能なパラメーターになっています。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |
| modelId | クエリ | モデル ID | いいえ | string |
| manifestName | クエリ | マニフェスト名 | いいえ | string |
| count | クエリ | 1 ページに取得する項目の数 | いいえ | string |
| $skipToken | クエリ | 次のページを取得する継続トークン | いいえ | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [PaginatedManifestList](#paginatedmanifestlist) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse) |

## <a name="get-manifest-details"></a>マニフェストの詳細の取得

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/manifests/{id} | 

### <a name="description"></a>説明
ID でマニフェストを取得します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| id | path | オブジェクト ID | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [Manifest](#manifest) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse) |

## <a name="create-an-image"></a>イメージを作成する

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| POST |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/images | 

### <a name="description"></a>説明
Azure Container Registry に Docker イメージとしてイメージを作成します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |
| imageRequest | body | イメージの作成に使用されるペイロード | あり | [ImageRequest](#imagerequest) |

### <a name="responses"></a>応答
| コード | 説明 | headers | スキーマ |
|--------------------|--------------------|--------------------|--------------------|
| 202 | 非同期操作の場所の URL。 GET 呼び出しは、イメージの作成タスクの状態を示します。 | Operation-Location |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-images-in-an-account"></a>アカウントにあるイメージの一覧の照会

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/images | 

### <a name="description"></a>説明
アカウントにあるイメージの一覧を照会します。 結果の一覧は、マニフェスト ID および名前でフィルター処理できます。 フィルターが渡されない場合、クエリを実行するとアカウントにあるすべてのイメージが一覧表示されます。 返された一覧は改ページ調整され、各ページの項目数は省略可能なパラメーターになっています。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |
| manifestId | クエリ | マニフェスト ID | いいえ | string |
| manifestName | クエリ | マニフェスト名 | いいえ | string |
| count | クエリ | 1 ページに取得する項目の数 | いいえ | string |
| $skipToken | クエリ | 次のページを取得する継続トークン | いいえ | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [PaginatedImageList](#paginatedimagelist) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse) |

## <a name="get-image-details"></a>イメージの詳細の取得

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/images/{id} | 

### <a name="description"></a>説明
ID でイメージを取得します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| id | path | イメージ ID | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [イメージ](#image) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse) |


## <a name="create-a-service"></a>サービスを作成する

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| POST |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services | 

### <a name="description"></a>説明
イメージからサービスを作成します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |
| serviceRequest | body | サービスの作成に使用されるペイロード | あり | [ServiceCreateRequest](#servicecreaterequest) |

### <a name="responses"></a>応答
| コード | 説明 | headers | スキーマ |
|--------------------|--------------------|--------------------|--------------------|
| 202 | 非同期操作の場所の URL。 GET 呼び出しは、サービスの作成タスクの状態を示します。 | Operation-Location |
| 409 | 指定された名前のサービスが既に存在しています。 |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-services-in-an-account"></a>アカウントにあるサービスの一覧を照会する

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services | 

### <a name="description"></a>説明
アカウントにあるサービスの一覧を照会します。 結果の一覧は、モデルの名前と ID、マニフェストの名前と ID、イメージ ID、サービス名、または Machine Learning のコンピューティング リソース ID でフィルター処理できます。 フィルターが渡されない場合、クエリを実行するとアカウントにあるすべてのサービスが一覧表示されます。 返された一覧は改ページ調整され、各ページの項目数は省略可能なパラメーターになっています。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |
| serviceName | クエリ | サービス名 | いいえ | string |
| modelId | クエリ | モデル名 | いいえ | string |
| modelName | クエリ | モデル ID | いいえ | string |
| manifestId | クエリ | マニフェスト ID | いいえ | string |
| manifestName | クエリ | マニフェスト名 | いいえ | string |
| imageId | クエリ | イメージ ID | いいえ | string |
| computeResourceId | クエリ | Machine Learning のコンピューティング リソース ID | いいえ | string |
| count | クエリ | 1 ページに取得する項目の数 | いいえ | string |
| $skipToken | クエリ | 次のページを取得する継続トークン | いいえ | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [PaginatedServiceList](#paginatedservicelist) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse) |

## <a name="get-service-details"></a>サービスの詳細を取得する

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{id} | 

### <a name="description"></a>説明
ID でサービスを取得します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| id | path | オブジェクト ID | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [ServiceResponse](#serviceresponse) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse)

## <a name="update-a-service"></a>サービスの更新

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| PUT |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{id} | 

### <a name="description"></a>説明
既存のサービスを更新します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| id | path | オブジェクト ID | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |
| serviceUpdateRequest | body | 既存のサービスの更新に使用されるペイロード | あり |  [ServiceUpdateRequest](#serviceupdaterequest) |

### <a name="responses"></a>応答
| コード | 説明 | headers | スキーマ |
|--------------------|--------------------|--------------------|--------------------|
| 202 | 非同期操作の場所の URL。 GET 呼び出しは、サービスの更新タスクの状態を示します。 | Operation-Location |
| 404 | 指定された ID のサービスが存在しません。 |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse)

## <a name="delete-a-service"></a>サービスの削除

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| 削除 |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{id} | 

### <a name="description"></a>説明
サービスを削除します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| id | path | オブジェクト ID | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 |  |
| 204 | 指定された ID のサービスが存在しません。 |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse)

## <a name="get-service-keys"></a>サービス キーの取得

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{id}/keys | 

### <a name="description"></a>説明
サービス キーを取得します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| id | path | サービス ID | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [AuthKeys](#authkeys)
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse)

## <a name="regenerate-service-keys"></a>サービス キーの再生成

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| POST |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{id}/keys | 

### <a name="description"></a>説明
サービス キーを再生成して返します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| id | path | サービス ID | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |
| regenerateKeyRequest | body | 既存のサービスの更新に使用されるペイロード | あり | [ServiceRegenerateKeyRequest](#serviceregeneratekeyrequest) |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [AuthKeys](#authkeys)
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse)

## <a name="query-the-list-of-deployments-in-an-account"></a>アカウントにあるデプロイ一覧を照会する

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/deployments | 

### <a name="description"></a>説明
アカウントにあるデプロイ一覧を照会します。 結果の一覧は、サービス ID でフィルター処理できます。特定のサービスに対して作成されたデプロイのみが返されます。 フィルターが渡されない場合、クエリを実行するとアカウントにあるすべてのデプロイが一覧表示されます。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |
| serviceId | クエリ | サービス ID | いいえ | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [DeploymentList](#deploymentlist) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse)

## <a name="get-deployment-details"></a>デプロイの詳細の取得

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/deployments/{id} | 

### <a name="description"></a>説明
ID でデプロイを取得します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| id | path | デプロイ ID | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [デプロイ](#deployment) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse)

## <a name="get-operation-details"></a>取得操作の詳細

### <a name="request"></a>要求
| メソッド | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/operations/{id} | 

### <a name="description"></a>説明
操作 ID で非同期操作の状態を取得します。

### <a name="parameters"></a>パラメーター
| 名前 | 場所 | 説明 | 必須 | スキーマ
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure サブスクリプション ID。 | あり | string |
| resourceGroupName | path | モデル管理アカウントが配置されているリソース グループの名前 | あり | string |
| accountName | path | モデル管理アカウントの名前 | あり | string |
| id | path | 操作 ID | あり | string |
| api-version | クエリ | 使用する Microsoft.Machine.Learning リソース プロバイダー API のバージョン | あり | string |
| 承認 | ヘッダー | 承認トークン。 "Bearer XXXXXX." のような形式にする必要があります。 | あり | string |

### <a name="responses"></a>応答
| コード | 説明 | スキーマ |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [OperationStatus](#asyncoperationstatus) |
| default | 操作に失敗した理由を説明するエラー応答 | [ErrorResponse](#errorresponse)



<a name="definitions"></a>
## <a name="definitions"></a>定義

<a name="asset"></a>
### <a name="asset"></a>資産
Docker イメージの作成中に必要となる資産オブジェクトです。


|名前|説明|スキーマ|
|---|---|---|
|**id**  <br>*省略可能*|資産 ID|string|
|**mimeType**  <br>*省略可能*|モデル コンテンツの MIME の種類。 MIME の種類の詳細については、「[IANA メディアの種類の一覧](https://www.iana.org/assignments/media-types/media-types.xhtml)」を参照してください。|string|
|**unpack**  <br>*省略可能*|Docker イメージの作成中に、コンテンツを解凍する必要がある場所を示します。|boolean|
|**URL**  <br>*省略可能*|資産の場所の URL|string|


<a name="asyncoperationstate"></a>
### <a name="asyncoperationstate"></a>AsyncOperationState
非同期操作の状態です。

"*型*": 列挙型 (未開始、実行中、キャンセル、成功、失敗)


<a name="asyncoperationstatus"></a>
### <a name="asyncoperationstatus"></a>AsyncOperationStatus
操作の状態です。


|名前|説明|スキーマ|
|---|---|---|
|**createdTime**  <br>*省略可能*  <br>*読み取り専用*|非同期操作の作成時刻 (UTC)|String (日時)|
|**endTime**  <br>*省略可能*  <br>*読み取り専用*|非同期操作の終了時刻 (UTC)|String (日時)|
|**error**  <br>*省略可能*||[ErrorResponse](#errorresponse)|
|**id**  <br>*省略可能*|非同期操作 ID|string|
|**operationType**  <br>*省略可能*|非同期操作の種類|列挙型 (イメージ、サービス)|
|**resourceLocation**  <br>*省略可能*|非同期操作によって作成または更新されたリソース|string|
|**state**  <br>*省略可能*||[AsyncOperationState](#asyncoperationstate)|


<a name="authkeys"></a>
### <a name="authkeys"></a>AuthKey
サービスの認証キーです。


|名前|説明|スキーマ|
|---|---|---|
|**primaryKey**  <br>*省略可能*|主キー|string|
|**secondaryKey**  <br>*省略可能*|セカンダリ キー|string|


<a name="autoscaler"></a>
### <a name="autoscaler"></a>AutoScaler
AutoScaler の設定です。


|名前|説明|スキーマ|
|---|---|---|
|**autoscaleEnabled**  <br>*省略可能*|AutoScaler を有効または無効にします。|boolean|
|**maxReplicas**  <br>*省略可能*|ポッド レプリカの最大スケールアップ数。  <br>**最小値**: `1`|integer|
|**minReplicas**  <br>*省略可能*|ポッド レプリカの最小スケールダウン数。  <br>**最小値**: `0`|integer|
|**refreshPeriodInSeconds**  <br>*省略可能*|自動スケール トリガーの更新時刻。  <br>**最小値**: `1`|integer|
|**targetUtilization**  <br>*省略可能*|自動スケールをトリガーする使用率。  <br>**最小値**: `0`  <br>**最大値** : `100`|integer|


<a name="computeresource"></a>
### <a name="computeresource"></a>ComputeResource
Machine Learning のコンピューティング リソースです。


|名前|説明|スキーマ|
|---|---|---|
|**id**  <br>*省略可能*|リソースの ID|string|
|**type**  <br>*省略可能*|リソースの種類。|列挙型 (クラスター)|


<a name="containerresourcereservation"></a>
### <a name="containerresourcereservation"></a>ContainerResourceReservation
クラスターのコンテナーに対してリソースを予約するための構成です。


|名前|説明|スキーマ|
|---|---|---|
|**cpu**  <br>*省略可能*|CPU 予約を指定します。 Kubernetes の形式: 「[Meaning of CPU](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-cpu)」(CPU の意味) を参照してください。|string|
|**memory**  <br>*省略可能*|メモリ予約を指定します。 Kubernetes の形式: 「[Meaning of memory](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-memory)」(メモリの意味) を参照してください。|string|


<a name="deployment"></a>
### <a name="deployment"></a>デプロイ
Azure Machine Learning デプロイのインスタンスです。


|名前|説明|スキーマ|
|---|---|---|
|**createdAt**  <br>*省略可能*  <br>*読み取り専用*|デプロイの作成時刻 (UTC)|String (日時)|
|**expiredAt**  <br>*省略可能*  <br>*読み取り専用*|デプロイの期限切れ時刻 (UTC)|String (日時)|
|**id**  <br>*省略可能*|デプロイ ID|string|
|**imageId**  <br>*省略可能*|このデプロイに関連付けられているイメージ ID|string|
|**serviceName**  <br>*省略可能*|サービス名|string|
|**status**  <br>*省略可能*|現在のデプロイの状態|string|


<a name="deploymentlist"></a>
### <a name="deploymentlist"></a>DeploymentList
デプロイ オブジェクトの配列です。

"*型*": <[デプロイ](#deployment)> 配列


<a name="errordetail"></a>
### <a name="errordetail"></a>ErrorDetail
モデル管理サービス エラーの詳細です。


|名前|説明|スキーマ|
|---|---|---|
|**code**  <br>*必須*|エラー コード。|string|
|**message**  <br>*必須*|エラー メッセージ。|string|


<a name="errorresponse"></a>
### <a name="errorresponse"></a>ErrorResponse
モデル管理サービス エラー オブジェクトです。


|名前|説明|スキーマ|
|---|---|---|
|**code**  <br>*必須*|エラー コード。|string|
|**details**  <br>*省略可能*|エラー詳細オブジェクトの配列|<[ErrorDetail](#errordetail)> 配列|
|**message**  <br>*必須*|エラー メッセージ。|string|
|**statusCode**  <br>*省略可能*|HTTP 状態コード。|integer|


<a name="image"></a>
### <a name="image"></a>イメージ
Auzre Machine Learning のイメージです。


|名前|説明|スキーマ|
|---|---|---|
|**computeResourceId**  <br>*省略可能*|Machine Learning のコンピューティング リソースで作成された環境の ID|string|
|**createdTime**  <br>*省略可能*|イメージの作成時刻 (UTC)|String (日時)|
|**creationState**  <br>*省略可能*||[AsyncOperationState](#asyncoperationstate)|
|**description**  <br>*省略可能*|イメージの説明のテキスト|string|
|**error**  <br>*省略可能*||[ErrorResponse](#errorresponse)|
|**id**  <br>*省略可能*|イメージ ID|string|
|**imageBuildLogUri**  <br>*省略可能*|イメージのビルドからアップロードされたログの URI|string|
|**imageLocation**  <br>*省略可能*|作成されたイメージの Azure Container Registry の場所を示す文字列|string|
|**imageType**  <br>*省略可能*||[imageType](#imagetype)|
|**manifest**  <br>*省略可能*||[Manifest](#manifest)|
|**name**  <br>*省略可能*|イメージ名|string|
|**version**  <br>*省略可能*|モデル管理サービスによって設定されたイメージのバージョン|integer|


<a name="imagerequest"></a>
### <a name="imagerequest"></a>ImageRequest
Auzre Machine Learning イメージを作成するための要求です。


|名前|説明|スキーマ|
|---|---|---|
|**computeResourceId**  <br>*必須*|Machine Learning のコンピューティング リソースで作成された環境の ID|string|
|**description**  <br>*省略可能*|イメージの説明のテキスト|string|
|**imageType**  <br>*必須*||[imageType](#imagetype)|
|**manifestId**  <br>*必須*|イメージの作成元となるマニフェストの ID|string|
|**name**  <br>*必須*|イメージ名|string|


<a name="imagetype"></a>
### <a name="imagetype"></a>ImageType
イメージの種類を指定します。

"*型*": 列挙型 (Docker)


<a name="manifest"></a>
### <a name="manifest"></a>Manifest
Azure Machine Learning のマニフェストです。


|名前|説明|スキーマ|
|---|---|---|
|**assets**  <br>*必須*|資産の一覧|<[Asset](#asset)> 配列|
|**createdTime**  <br>*省略可能*  <br>*読み取り専用*|マニフェストの作成時刻 (UTC)|String (日時)|
|**description**  <br>*省略可能*|マニフェストの説明のテキスト|string|
|**driverProgram**  <br>*必須*|マニフェストのドライバー プログラム|string|
|**id**  <br>*省略可能*|マニフェスト ID|string|
|**modelIds**  <br>*省略可能*|登録済みのモデルのモデル ID の一覧。 含まれているモデルのいずれかが登録されていない場合、要求は失敗します。|<string> 配列|
|**modelType**  <br>*省略可能*|モデル管理サービスにモデルが既に登録されていることを示します。|列挙型 (登録済み)|
|**name**  <br>*必須*|マニフェスト名|string|
|**targetRuntime**  <br>*必須*||[TargetRuntime](#targetruntime)|
|**version**  <br>*省略可能*  <br>*読み取り専用*|モデル管理サービスによって割り当てられているマニフェストのバージョン|integer|
|**webserviceType**  <br>*省略可能*|マニフェストから作成される Web サービスの目的の種類を指定します。|列挙型 (リアルタイム)|


<a name="model"></a>
### <a name="model"></a>モデル
Azure Machine Learning モデルのインスタンスです。


|名前|説明|スキーマ|
|---|---|---|
|**createdAt**  <br>*省略可能*  <br>*読み取り専用*|モデルの作成時刻 (UTC)|String (日時)|
|**description**  <br>*省略可能*|モデルの説明のテキスト|string|
|**id**  <br>*省略可能*  <br>*読み取り専用*|モデル ID|string|
|**mimeType**  <br>*必須*|モデル コンテンツの MIME の種類 MIME の種類の詳細については、「[IANA メディアの種類の一覧](https://www.iana.org/assignments/media-types/media-types.xhtml)」を参照してください。|string|
|**name**  <br>*必須*|モデル名|string|
|**タグ**  <br>*省略可能*|モデルのタグ一覧|<string> 配列|
|**unpack**  <br>*省略可能*|Docker イメージの作成中に、モデルを解凍する必要があるかどうかを示します。|boolean|
|**URL**  <br>*必須*|モデルの URL。 通常、Shared Access Signature URL をここに置きます。|string|
|**version**  <br>*省略可能*  <br>*読み取り専用*|モデル管理サービスによって割り当てられているモデルのバージョン|integer|


<a name="modeldatacollection"></a>
### <a name="modeldatacollection"></a>ModelDataCollection
モデル データ コレクションの情報です。


|名前|説明|スキーマ|
|---|---|---|
|**eventHubEnabled**  <br>*省略可能*|サービスのイベント ハブを有効にします。|boolean|
|**storageEnabled**  <br>*省略可能*|サービスのストレージを有効にします。|boolean|


<a name="paginatedimagelist"></a>
### <a name="paginatedimagelist"></a>PaginatedImageList
改ページ調整されたイメージ一覧です。


|名前|説明|スキーマ|
|---|---|---|
|**nextLink**  <br>*省略可能*|一覧にある結果の次のページへの継続リンク (絶対 URI)|string|
|**値**  <br>*省略可能*|モデル オブジェクトの配列|<[Image](#image)> 配列|


<a name="paginatedmanifestlist"></a>
### <a name="paginatedmanifestlist"></a>PaginatedManifestList
改ページ調整されたマニフェスト一覧です。


|名前|説明|スキーマ|
|---|---|---|
|**nextLink**  <br>*省略可能*|一覧にある結果の次のページへの継続リンク (絶対 URI)|string|
|**値**  <br>*省略可能*|マニフェスト オブジェクトの配列|<[Manifest](#manifest)> 配列|


<a name="paginatedmodellist"></a>
### <a name="paginatedmodellist"></a>PaginatedModelList
改ページ調整されたモデル一覧です。


|名前|説明|スキーマ|
|---|---|---|
|**nextLink**  <br>*省略可能*|一覧にある結果の次のページへの継続リンク (絶対 URI)|string|
|**値**  <br>*省略可能*|モデル オブジェクトの配列|<[Model](#model)> 配列|


<a name="paginatedservicelist"></a>
### <a name="paginatedservicelist"></a>PaginatedServiceList
改ページ調整されたサービスの一覧です。


|名前|説明|スキーマ|
|---|---|---|
|**nextLink**  <br>*省略可能*|一覧にある結果の次のページへの継続リンク (絶対 URI)|string|
|**値**  <br>*省略可能*|サービス オブジェクトの配列|<[ServiceResponse](#serviceresponse)> 配列|


<a name="servicecreaterequest"></a>
### <a name="servicecreaterequest"></a>ServiceCreateRequest
サービスを作成するための要求です。


|名前|説明|スキーマ|
|---|---|---|
|**appInsightsEnabled**  <br>*省略可能*|サービスの Application Insights を有効にします。|boolean|
|**autoScaler**  <br>*省略可能*||[AutoScaler](#autoscaler)|
|**computeResource**  <br>*必須*||[ComputeResource](#computeresource)|
|**containerResourceReservation**  <br>*省略可能*||[ContainerResourceReservation](#containerresourcereservation)|
|**dataCollection**  <br>*省略可能*||[ModelDataCollection](#modeldatacollection)|
|**imageId**  <br>*必須*|サービスを作成するためのイメージ|string|
|**maxConcurrentRequestsPerContainer**  <br>*省略可能*|同時要求の最大数。  <br>**最小値**: `1`|integer|
|**name**  <br>*必須*|サービス名|string|
|**numReplicas**  <br>*省略可能*|任意の時点で実行されているポッド レプリカの数。 autoscaler が有効になっている場合は指定できません。  <br>**最小値**: `0`|integer|


<a name="serviceregeneratekeyrequest"></a>
### <a name="serviceregeneratekeyrequest"></a>ServiceRegenerateKeyRequest
サービスのキーを再生成する要求です。


|名前|説明|スキーマ|
|---|---|---|
|**keyType**  <br>*省略可能*|再生成するキーを指定します。|列挙型 (プライマリ、セカンダリ)|


<a name="serviceresponse"></a>
### <a name="serviceresponse"></a>ServiceResponse
サービスの詳細な状態です。


|名前|説明|スキーマ|
|---|---|---|
|**createdAt**  <br>*省略可能*|サービスの作成時刻 (UTC)|String (日時)|
|**ID**  <br>*省略可能*|サービス ID|string|
|**画像**  <br>*省略可能*||[イメージ](#image)|
|**manifest**  <br>*省略可能*||[Manifest](#manifest)|
|**models**  <br>*省略可能*|モデルの一覧|<[Model](#model)> 配列|
|**name**  <br>*省略可能*|サービス名|string|
|**scoringUri**  <br>*省略可能*|サービスのスコア付けの URI|string|
|**state**  <br>*省略可能*||[AsyncOperationState](#asyncoperationstate)|
|**updatedAt**  <br>*省略可能*|最終更新時刻 (UTC)|String (日時)|
|**appInsightsEnabled**  <br>*省略可能*|サービスの Application Insights を有効にします。|boolean|
|**autoScaler**  <br>*省略可能*||[AutoScaler](#autoscaler)|
|**computeResource**  <br>*必須*||[ComputeResource](#computeresource)|
|**containerResourceReservation**  <br>*省略可能*||[ContainerResourceReservation](#containerresourcereservation)|
|**dataCollection**  <br>*省略可能*||[ModelDataCollection](#modeldatacollection)|
|**maxConcurrentRequestsPerContainer**  <br>*省略可能*|同時要求の最大数。  <br>**最小値**: `1`|integer|
|**numReplicas**  <br>*省略可能*|任意の時点で実行されているポッド レプリカの数。 autoscaler が有効になっている場合は指定できません。  <br>**最小値**: `0`|integer|
|**error**  <br>*省略可能*||[ErrorResponse](#errorresponse)|


<a name="serviceupdaterequest"></a>
### <a name="serviceupdaterequest"></a>ServiceUpdateRequest
サービスを更新するための要求です。


|名前|説明|スキーマ|
|---|---|---|
|**appInsightsEnabled**  <br>*省略可能*|サービスの Application Insights を有効にします。|boolean|
|**autoScaler**  <br>*省略可能*||[AutoScaler](#autoscaler)|
|**containerResourceReservation**  <br>*省略可能*||[ContainerResourceReservation](#containerresourcereservation)|
|**dataCollection**  <br>*省略可能*||[ModelDataCollection](#modeldatacollection)|
|**imageId**  <br>*省略可能*|サービスを作成するためのイメージ|string|
|**maxConcurrentRequestsPerContainer**  <br>*省略可能*|同時要求の最大数。  <br>**最小値**: `1`|integer|
|**numReplicas**  <br>*省略可能*|任意の時点で実行されているポッド レプリカの数。 autoscaler が有効になっている場合は指定できません。  <br>**最小値**: `0`|integer|


<a name="targetruntime"></a>
### <a name="targetruntime"></a>TargetRuntime
ターゲット ランタイムの型です。


|名前|説明|スキーマ|
|---|---|---|
|**properties**  <br>*必須*||<string, string> マップ|
|**runtimeType**  <br>*必須*|ランタイムを指定します。|列挙型 (SparkPython、Python)|

