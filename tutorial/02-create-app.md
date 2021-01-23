---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942156"
---
<!-- markdownlint-disable MD002 MD041 -->

まず、コア Web アプリASP.NET作成します。

1. プロジェクトを作成するディレクトリでコマンド ライン インターフェイス (CLI) を開きます。 次のコマンドを実行します。

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. プロジェクトが作成された後、現在のディレクトリを **GraphTu clil** ディレクトリに変更し、CLI で次のコマンドを実行して、プロジェクトが動作を確認します。

    ```Shell
    dotnet run
    ```

1. ブラウザーを開き、参照します `https://localhost:5001` 。 すべてが動作している場合は、既定の [コア] ページASP.NET表示されます。

> [!IMPORTANT]
> **localhost** の証明書が信頼されていないという警告を受け取った場合は、.NET Core CLI を使用して開発証明書をインストールして信頼できます。 特定 [のオペレーティング システムの手順については、「ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) での HTTPS の適用」を参照してください。

## <a name="add-nuget-packages"></a>NuGet パッケージを追加する

次に進む前に、後で使用する追加の NuGet パッケージをインストールします。

- アクセス トークンを要求および管理するための[Microsoft.Identity.Web。](https://www.nuget.org/packages/Microsoft.Identity.Web/)
- 依存関係挿入を使用して Microsoft Graph SDK を追加する[Microsoft.Identity.Web.MicrosoftGraph。](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/)
- サインインおよびサインアウト UI 用の[Microsoft.Identity.Web.UI。](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/)
- タイム ゾーン ID をプラットフォーム間で処理する[TimeZoneConverter。](https://github.com/mj1856/TimeZoneConverter)

1. CLI で次のコマンドを実行して、依存関係をインストールします。

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a>アプリを設計する

このセクションでは、アプリケーションの基本的な UI 構造を作成します。

### <a name="implement-alert-extension-methods"></a>アラート拡張メソッドを実装する

このセクションでは、コントローラー ビューによって返される型の `IActionResult` 拡張メソッドを作成します。 この拡張機能により、一時的なエラーまたは成功メッセージをビューに渡す機能が有効になります。

> [!TIP]
> このチュートリアルでは、任意のテキスト エディターを使用してソース ファイルを編集できます。 ただし [、Visual Studioコードには](https://code.visualstudio.com/) 、デバッグや Intellisense などの追加機能があります。

1. Alerts という名前の **GraphTu読み込みディレクトリに** 新しいディレクトリを **作成します**。

1. **./Alerts** ディレクトリに **WithAlertResult.cs** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. **./Alerts** ディレクトリに **AlertExtensions.cs** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a>ユーザー データ拡張メソッドを実装する

このセクションでは、Microsoft Identity プラットフォームによって生成されるオブジェクト `ClaimsPrincipal` の拡張メソッドを作成します。 これにより、既存のユーザー ID を Microsoft Graph のデータで拡張できます。

> [!NOTE]
> このコードは、今のところプレースホルダーにすがっています。後のセクションで完成します。

1. Graph という名前の **GraphTu graphl ディレクトリに新** しいディレクトリを作成 **します**。

1. 新しいファイルを GraphClaimsPrincipalExtensions.csし **、** 次のコードを追加します。

    ```csharp
    using System.Security.Claims;

    namespace GraphTutorial
    {
        public static class GraphClaimTypes {
            public const string DisplayName ="graph_name";
            public const string Email = "graph_email";
            public const string Photo = "graph_photo";
            public const string TimeZone = "graph_timezone";
            public const string DateTimeFormat = "graph_datetimeformat";
        }

        // Helper methods to access Graph user data stored in
        // the claims principal
        public static class GraphClaimsPrincipalExtensions
        {
            public static string GetUserGraphDisplayName(this ClaimsPrincipal claimsPrincipal)
            {
                return "Adele Vance";
            }

            public static string GetUserGraphEmail(this ClaimsPrincipal claimsPrincipal)
            {
                return "adelev@contoso.com";
            }

            public static string GetUserGraphPhoto(this ClaimsPrincipal claimsPrincipal)
            {
                return "/img/no-profile-photo.png";
            }
        }
    }
    ```

### <a name="create-views"></a>ビューを作成する

このセクションでは、アプリケーションの "多" ビューを実装します。

1. **_LoginPartial.cshtml という** 名前の新しいファイルを **./Views/Shared** ディレクトリに追加し、次のコードを追加します。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. **_AlertPartial.cshtml という名前** の新しいファイルを **./Views/Shared** ディレクトリに追加し、次のコードを追加します。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. アプリのグローバル レイアウトを更新するため、**./Views/Shared/_Layout.cshtml** ファイルを開き、内容全体を次のコードで置き換えます。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. **./wwwroot/css/site.css** を開き、ファイルの下部に次のコードを追加します。

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. **./Views/Home/index.cshtml** ファイルを開き、その内容を次の内容に置き換えます。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. img という名前の **./wwwroot ディレクトリに新しい** ディレクトリを作成 **します**。 このディレクトリに、選択した名前 **のno-profile-photo.pngファイル** を追加します。 この画像は、ユーザーが Microsoft Graph に写真を持たない場合に、ユーザーの写真として使用されます。

    > [!TIP]
    > これらのスクリーンショットで使用されている画像は [、GitHub からダウンロードできます](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)。

1. すべての変更を保存し、サーバー () を再起動します `dotnet run` 。 これで、アプリは非常に異なって表示されます。

    ![デザインが変更されたホーム ページのスクリーンショット](./images/create-app-01.png)
