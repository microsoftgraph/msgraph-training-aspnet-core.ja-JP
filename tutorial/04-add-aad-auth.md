---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942163"
---
<!-- markdownlint-disable MD002 MD041 -->

この演習では、前の演習のアプリケーションを拡張して、Azure AD での認証をサポートします。 これは、Microsoft Graph API を呼び出すのに必要な OAuth アクセス トークンを取得するために必要です。 この手順では [、Microsoft.Identity.Web ライブラリを構成](https://www.nuget.org/packages/Microsoft.Identity.Web/) します。

> [!IMPORTANT]
> アプリケーション ID とシークレットをソースに格納しないようにするには、.NET [Secret Manager](/aspnet/core/security/app-secrets) を使用してこれらの値を格納します。 シークレット マネージャーは開発のみを目的とします。実稼働アプリでは、シークレットの保存に信頼できるシークレット マネージャーを使用する必要があります。

1. **./appsettings.jsを開き**、その内容を次の内容に置き換えてください。

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. **GraphTu clil.csproj** があるディレクトリで CLI を開き、Azure portal のアプリケーション ID とアプリケーション シークレットに置き換え、次のコマンドを実行します。 `YOUR_APP_ID` `YOUR_APP_SECRET`

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a>サインインの実装

まず、Microsoft Identity プラットフォーム サービスをアプリケーションに追加します。

1. **./Graph** ディレクトリに **GraphConstants.cs** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. **./Startup.cs** ファイルを開き、ファイルの一番上に次 `using` のステートメントを追加します。

    ```csharp
    using Microsoft.AspNetCore.Authentication.OpenIdConnect;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc.Authorization;
    using Microsoft.Identity.Web;
    using Microsoft.Identity.Web.UI;
    using Microsoft.IdentityModel.Protocols.OpenIdConnect;
    using Microsoft.Graph;
    using System.Net;
    using System.Net.Http.Headers;
    ```

1. 既存の `ConfigureServices` 関数を、以下の関数で置き換えます。

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services
            // Use OpenId authentication
            .AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
            // Specify this is a web app and needs auth code flow
            .AddMicrosoftIdentityWebApp(Configuration)
            // Add ability to call web API (Graph)
            // and get access tokens
            .EnableTokenAcquisitionToCallDownstreamApi(options => {
                Configuration.Bind("AzureAd", options);
            }, GraphConstants.Scopes)
            // Use in-memory token cache
            // See https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization
            .AddInMemoryTokenCaches();

        // Require authentication
        services.AddControllersWithViews(options =>
        {
            var policy = new AuthorizationPolicyBuilder()
                .RequireAuthenticatedUser()
                .Build();
            options.Filters.Add(new AuthorizeFilter(policy));
        })
        // Add the Microsoft Identity UI pages for signin/out
        .AddMicrosoftIdentityUI();
    }
    ```

1. 関数で `Configure` 、行の上に次の行を追加 `app.UseAuthorization();` します。

    ```csharp
    app.UseAuthentication();
    ```

1. **./Controllers/HomeController.cs を** 開き、その内容を次の内容に置き換えてください。

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using System.Diagnostics;
    using System.Threading.Tasks;

    namespace GraphTutorial.Controllers
    {
        public class HomeController : Controller
        {
            ITokenAcquisition _tokenAcquisition;
            private readonly ILogger<HomeController> _logger;

            // Get the ITokenAcquisition interface via
            // dependency injection
            public HomeController(
                ITokenAcquisition tokenAcquisition,
                ILogger<HomeController> logger)
            {
                _tokenAcquisition = tokenAcquisition;
                _logger = logger;
            }

            public async Task<IActionResult> Index()
            {
                // TEMPORARY
                // Get the token and display it
                try
                {
                    string token = await _tokenAcquisition
                        .GetAccessTokenForUserAsync(GraphConstants.Scopes);
                    return View().WithInfo("Token acquired", token);
                }
                catch (MicrosoftIdentityWebChallengeUserException)
                {
                    return Challenge();
                }
            }

            public IActionResult Privacy()
            {
                return View();
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            public IActionResult Error()
            {
                return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            [AllowAnonymous]
            public IActionResult ErrorWithMessage(string message, string debug)
            {
                return View("Index").WithError(message, debug);
            }
        }
    }
    ```

1. 変更を保存してプロジェクトを開始します。 Microsoft アカウントでログインします。

1. 同意プロンプトを確認します。 アクセス許可の一覧は **、./Graph/GraphConstants.cs** で構成されたアクセス許可スコープの一覧に対応しています。

    - **アクセス権を付与** したデータへのアクセスを維持する: ( ) このアクセス許可は、更新トークンを取得するために `offline_access` MSAL によって要求されます。
    - **サインインしてプロファイルを** 読み取る: ( ) このアクセス許可により、アプリはログインしているユーザーのプロファイルとプロファイル写真 `User.Read` を取得できます。
    - **メールボックスの設定を読み取る:** ( ) このアクセス許可により、アプリはタイム ゾーンや時刻形式を含むユーザーのメールボックス設定 `MailboxSettings.Read` を読み取りできます。
    - **予定表へのフル** アクセス権: ( ) このアクセス許可により、アプリはユーザーの予定表のイベントの読み取り、新しいイベントの追加、既存のイベントの変更を `Calendars.ReadWrite` 行います。

    ![Microsoft ID プラットフォームの同意プロンプトのスクリーンショット](./images/add-aad-auth-03.png)

    同意の詳細については、「Azure アプリケーションの同意エクスペリエンスについて [AD参照してください](/azure/active-directory/develop/application-consent-experience)。

1. 要求されたアクセス許可に同意します。 ブラウザーがアプリにリダイレクトし、トークンが表示されます。

### <a name="get-user-details"></a>ユーザーの詳細情報を取得する

ユーザーがログインすると、Microsoft Graph からそのユーザーの情報を入手できます。

1. **./Graph/GraphClaimsPrincipalExtensions.cs** を開き、その内容全体を次の内容に置き換えてください。

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. **./Startup.cs** 開き、既存の行を次 `.AddMicrosoftIdentityWebApp(Configuration)` のコードに置き換えます。

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    このコードの動作を検討します。

    - イベントのイベント ハンドラーを追加 `OnTokenValidated` します。
        - インターフェイスを `ITokenAcquisition` 使用してアクセス トークンを取得します。
        - Microsoft Graph を呼び出して、ユーザーのプロファイルと写真を取得します。
        - Graph 情報をユーザーの ID に追加します。

1. 呼び出しの後と呼び出しの `EnableTokenAcquisitionToCallDownstreamApi` 前に、次の関数呼び出しを追加 `AddInMemoryTokenCaches` します。

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    これにより、依存関係挿入によって、 **認証された GraphServiceClient** をコントローラーで使用できます。

1. **./Controllers/HomeController.cs を** 開き、関数を `Index` 次の関数に置き換える。

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. `ITokenAcquisition` **HomeController** クラス内のすべての参照を削除します。

1. 変更内容を保存し、アプリを起動して、サインイン プロセスを実行します。 ホーム ページに戻る必要がありますが、サインイン中を示すために UI が変更される必要があります。

    ![サインイン後のホーム ページのスクリーンショット](./images/add-aad-auth-01.png)

1. 右上隅にあるユーザー アバターをクリックして、[サインアウト **] リンクにアクセス** します。 **[サインアウト]** をクリックすると、セッションがリセットされ、ホーム ページに戻ります。

    ![[サインアウト] リンクのドロップダウン メニューのスクリーンショット](./images/add-aad-auth-02.png)

> [!TIP]
> ホーム ページにユーザー名が表示されない場合に、これらの変更を行った後に[アバターの使用] ドロップダウンに名前とメールが表示されない場合は、サインアウトしてサインインし戻します。

## <a name="storing-and-refreshing-tokens"></a>トークンの保存と更新

この時点で、アプリケーションはアクセス トークンを持ち、API 呼び出しのヘッダー `Authorization` で送信されます。 これは、アプリが Microsoft Graph にユーザーの代わりにアクセスできるようにするトークンです。

ただし、このトークンは一時的なものです。 トークンは発行後 1 時間で期限切れになります。 ここで、更新トークンが役に立ちます。 更新トークンを使用すると、ユーザーが再度サインインしなくても、アプリは新しいアクセス トークンを要求できます。

アプリは Microsoft.Identity.Web ライブラリを使用しているので、トークンストレージまたは更新ロジックを実装する必要は一切ない。

アプリはメモリ内トークン キャッシュを使用しますが、アプリの再起動時にトークンを保持する必要がないアプリでは十分です。 実稼働アプリでは、代わりに[](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization)Microsoft.Identity.Web ライブラリの分散キャッシュ オプションを使用できます。

この `GetAccessTokenForUserAsync` メソッドは、トークンの有効期限と更新を処理します。 最初にキャッシュされたトークンをチェックし、有効期限が切れていない場合はトークンを返します。 有効期限が切れている場合は、キャッシュされた更新トークンを使用して新しい更新トークンを取得します。

コントローラー **が依存関係挿入** を介して取得する GraphServiceClient は、自動的に使用する認証プロバイダーで事前 `GetAccessTokenForUserAsync` に構成されます。
