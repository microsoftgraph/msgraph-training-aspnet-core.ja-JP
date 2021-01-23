---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942163"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f997e-101">この演習では、前の演習のアプリケーションを拡張して、Azure AD での認証をサポートします。</span><span class="sxs-lookup"><span data-stu-id="f997e-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="f997e-102">これは、Microsoft Graph API を呼び出すのに必要な OAuth アクセス トークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="f997e-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="f997e-103">この手順では [、Microsoft.Identity.Web ライブラリを構成](https://www.nuget.org/packages/Microsoft.Identity.Web/) します。</span><span class="sxs-lookup"><span data-stu-id="f997e-103">In this step you will configure the [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) library.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="f997e-104">アプリケーション ID とシークレットをソースに格納しないようにするには、.NET [Secret Manager](/aspnet/core/security/app-secrets) を使用してこれらの値を格納します。</span><span class="sxs-lookup"><span data-stu-id="f997e-104">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="f997e-105">シークレット マネージャーは開発のみを目的とします。実稼働アプリでは、シークレットの保存に信頼できるシークレット マネージャーを使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f997e-105">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

1. <span data-ttu-id="f997e-106">**./appsettings.jsを開き**、その内容を次の内容に置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="f997e-106">Open **./appsettings.json** and replace its contents with the following.</span></span>

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. <span data-ttu-id="f997e-107">**GraphTu clil.csproj** があるディレクトリで CLI を開き、Azure portal のアプリケーション ID とアプリケーション シークレットに置き換え、次のコマンドを実行します。 `YOUR_APP_ID` `YOUR_APP_SECRET`</span><span class="sxs-lookup"><span data-stu-id="f997e-107">Open your CLI in the directory where **GraphTutorial.csproj** is located, and run the following commands, substituting `YOUR_APP_ID` with your application ID from the Azure portal, and `YOUR_APP_SECRET` with your application secret.</span></span>

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="f997e-108">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="f997e-108">Implement sign-in</span></span>

<span data-ttu-id="f997e-109">まず、Microsoft Identity プラットフォーム サービスをアプリケーションに追加します。</span><span class="sxs-lookup"><span data-stu-id="f997e-109">Start by adding the Microsoft Identity platform services to the application.</span></span>

1. <span data-ttu-id="f997e-110">**./Graph** ディレクトリに **GraphConstants.cs** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="f997e-110">Create a new file named **GraphConstants.cs** in the **./Graph** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. <span data-ttu-id="f997e-111">**./Startup.cs** ファイルを開き、ファイルの一番上に次 `using` のステートメントを追加します。</span><span class="sxs-lookup"><span data-stu-id="f997e-111">Open the **./Startup.cs** file and add the following `using` statements to the top of the file.</span></span>

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

1. <span data-ttu-id="f997e-112">既存の `ConfigureServices` 関数を、以下の関数で置き換えます。</span><span class="sxs-lookup"><span data-stu-id="f997e-112">Replace the existing `ConfigureServices` function with the following.</span></span>

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

1. <span data-ttu-id="f997e-113">関数で `Configure` 、行の上に次の行を追加 `app.UseAuthorization();` します。</span><span class="sxs-lookup"><span data-stu-id="f997e-113">In the `Configure` function, add the following line above the `app.UseAuthorization();` line.</span></span>

    ```csharp
    app.UseAuthentication();
    ```

1. <span data-ttu-id="f997e-114">**./Controllers/HomeController.cs を** 開き、その内容を次の内容に置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="f997e-114">Open **./Controllers/HomeController.cs** and replace its contents with the following.</span></span>

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

1. <span data-ttu-id="f997e-115">変更を保存してプロジェクトを開始します。</span><span class="sxs-lookup"><span data-stu-id="f997e-115">Save your changes and start the project.</span></span> <span data-ttu-id="f997e-116">Microsoft アカウントでログインします。</span><span class="sxs-lookup"><span data-stu-id="f997e-116">Login with your Microsoft account.</span></span>

1. <span data-ttu-id="f997e-117">同意プロンプトを確認します。</span><span class="sxs-lookup"><span data-stu-id="f997e-117">Examine the consent prompt.</span></span> <span data-ttu-id="f997e-118">アクセス許可の一覧は **、./Graph/GraphConstants.cs** で構成されたアクセス許可スコープの一覧に対応しています。</span><span class="sxs-lookup"><span data-stu-id="f997e-118">The list of permissions correspond to list of permissions scopes configured in **./Graph/GraphConstants.cs**.</span></span>

    - <span data-ttu-id="f997e-119">**アクセス権を付与** したデータへのアクセスを維持する: ( ) このアクセス許可は、更新トークンを取得するために `offline_access` MSAL によって要求されます。</span><span class="sxs-lookup"><span data-stu-id="f997e-119">**Maintain access to data you have given it access to:** (`offline_access`) This permission is requested by MSAL in order to retrieve refresh tokens.</span></span>
    - <span data-ttu-id="f997e-120">**サインインしてプロファイルを** 読み取る: ( ) このアクセス許可により、アプリはログインしているユーザーのプロファイルとプロファイル写真 `User.Read` を取得できます。</span><span class="sxs-lookup"><span data-stu-id="f997e-120">**Sign you in and read your profile:** (`User.Read`) This permission allows the app to get the logged-in user's profile and profile photo.</span></span>
    - <span data-ttu-id="f997e-121">**メールボックスの設定を読み取る:** ( ) このアクセス許可により、アプリはタイム ゾーンや時刻形式を含むユーザーのメールボックス設定 `MailboxSettings.Read` を読み取りできます。</span><span class="sxs-lookup"><span data-stu-id="f997e-121">**Read your mailbox settings:** (`MailboxSettings.Read`) This permission allows the app to read the user's mailbox settings, including time zone and time format.</span></span>
    - <span data-ttu-id="f997e-122">**予定表へのフル** アクセス権: ( ) このアクセス許可により、アプリはユーザーの予定表のイベントの読み取り、新しいイベントの追加、既存のイベントの変更を `Calendars.ReadWrite` 行います。</span><span class="sxs-lookup"><span data-stu-id="f997e-122">**Have full access to your calendars:** (`Calendars.ReadWrite`) This permission allows the app to read events on the user's calendar, add new events, and modify existing ones.</span></span>

    ![Microsoft ID プラットフォームの同意プロンプトのスクリーンショット](./images/add-aad-auth-03.png)

    <span data-ttu-id="f997e-124">同意の詳細については、「Azure アプリケーションの同意エクスペリエンスについて [AD参照してください](/azure/active-directory/develop/application-consent-experience)。</span><span class="sxs-lookup"><span data-stu-id="f997e-124">For more information regarding consent, see [Understanding Azure AD application consent experiences](/azure/active-directory/develop/application-consent-experience).</span></span>

1. <span data-ttu-id="f997e-125">要求されたアクセス許可に同意します。</span><span class="sxs-lookup"><span data-stu-id="f997e-125">Consent to the requested permissions.</span></span> <span data-ttu-id="f997e-126">ブラウザーがアプリにリダイレクトし、トークンが表示されます。</span><span class="sxs-lookup"><span data-stu-id="f997e-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="f997e-127">ユーザーの詳細情報を取得する</span><span class="sxs-lookup"><span data-stu-id="f997e-127">Get user details</span></span>

<span data-ttu-id="f997e-128">ユーザーがログインすると、Microsoft Graph からそのユーザーの情報を入手できます。</span><span class="sxs-lookup"><span data-stu-id="f997e-128">Once the user is logged in, you can get their information from Microsoft Graph.</span></span>

1. <span data-ttu-id="f997e-129">**./Graph/GraphClaimsPrincipalExtensions.cs** を開き、その内容全体を次の内容に置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="f997e-129">Open **./Graph/GraphClaimsPrincipalExtensions.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. <span data-ttu-id="f997e-130">**./Startup.cs** 開き、既存の行を次 `.AddMicrosoftIdentityWebApp(Configuration)` のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="f997e-130">Open **./Startup.cs** and replace the existing `.AddMicrosoftIdentityWebApp(Configuration)` line with the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    <span data-ttu-id="f997e-131">このコードの動作を検討します。</span><span class="sxs-lookup"><span data-stu-id="f997e-131">Consider what this code does.</span></span>

    - <span data-ttu-id="f997e-132">イベントのイベント ハンドラーを追加 `OnTokenValidated` します。</span><span class="sxs-lookup"><span data-stu-id="f997e-132">It adds an event handler for the `OnTokenValidated` event.</span></span>
        - <span data-ttu-id="f997e-133">インターフェイスを `ITokenAcquisition` 使用してアクセス トークンを取得します。</span><span class="sxs-lookup"><span data-stu-id="f997e-133">It uses the `ITokenAcquisition` interface to get an access token.</span></span>
        - <span data-ttu-id="f997e-134">Microsoft Graph を呼び出して、ユーザーのプロファイルと写真を取得します。</span><span class="sxs-lookup"><span data-stu-id="f997e-134">It calls Microsoft Graph to get the user's profile and photo.</span></span>
        - <span data-ttu-id="f997e-135">Graph 情報をユーザーの ID に追加します。</span><span class="sxs-lookup"><span data-stu-id="f997e-135">It adds the Graph information to the user's identity.</span></span>

1. <span data-ttu-id="f997e-136">呼び出しの後と呼び出しの `EnableTokenAcquisitionToCallDownstreamApi` 前に、次の関数呼び出しを追加 `AddInMemoryTokenCaches` します。</span><span class="sxs-lookup"><span data-stu-id="f997e-136">Add the following function call after the `EnableTokenAcquisitionToCallDownstreamApi` call and before the `AddInMemoryTokenCaches` call.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    <span data-ttu-id="f997e-137">これにより、依存関係挿入によって、 **認証された GraphServiceClient** をコントローラーで使用できます。</span><span class="sxs-lookup"><span data-stu-id="f997e-137">This will make an authenticated **GraphServiceClient** available to controllers via dependency injection.</span></span>

1. <span data-ttu-id="f997e-138">**./Controllers/HomeController.cs を** 開き、関数を `Index` 次の関数に置き換える。</span><span class="sxs-lookup"><span data-stu-id="f997e-138">Open **./Controllers/HomeController.cs** and replace the `Index` function with the following.</span></span>

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. <span data-ttu-id="f997e-139">`ITokenAcquisition` **HomeController** クラス内のすべての参照を削除します。</span><span class="sxs-lookup"><span data-stu-id="f997e-139">Remove all references to `ITokenAcquisition` in the **HomeController** class.</span></span>

1. <span data-ttu-id="f997e-140">変更内容を保存し、アプリを起動して、サインイン プロセスを実行します。</span><span class="sxs-lookup"><span data-stu-id="f997e-140">Save your changes, start the app, and go through the sign-in process.</span></span> <span data-ttu-id="f997e-141">ホーム ページに戻る必要がありますが、サインイン中を示すために UI が変更される必要があります。</span><span class="sxs-lookup"><span data-stu-id="f997e-141">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![サインイン後のホーム ページのスクリーンショット](./images/add-aad-auth-01.png)

1. <span data-ttu-id="f997e-143">右上隅にあるユーザー アバターをクリックして、[サインアウト **] リンクにアクセス** します。</span><span class="sxs-lookup"><span data-stu-id="f997e-143">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="f997e-144">**[サインアウト]** をクリックすると、セッションがリセットされ、ホーム ページに戻ります。</span><span class="sxs-lookup"><span data-stu-id="f997e-144">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![[サインアウト] リンクのドロップダウン メニューのスクリーンショット](./images/add-aad-auth-02.png)

> [!TIP]
> <span data-ttu-id="f997e-146">ホーム ページにユーザー名が表示されない場合に、これらの変更を行った後に[アバターの使用] ドロップダウンに名前とメールが表示されない場合は、サインアウトしてサインインし戻します。</span><span class="sxs-lookup"><span data-stu-id="f997e-146">If you do not see your user name on the home page and the use avatar dropdown is missing name and email after making these changes, sign out and sign back in.</span></span>

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="f997e-147">トークンの保存と更新</span><span class="sxs-lookup"><span data-stu-id="f997e-147">Storing and refreshing tokens</span></span>

<span data-ttu-id="f997e-148">この時点で、アプリケーションはアクセス トークンを持ち、API 呼び出しのヘッダー `Authorization` で送信されます。</span><span class="sxs-lookup"><span data-stu-id="f997e-148">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="f997e-149">これは、アプリが Microsoft Graph にユーザーの代わりにアクセスできるようにするトークンです。</span><span class="sxs-lookup"><span data-stu-id="f997e-149">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="f997e-150">ただし、このトークンは一時的なものです。</span><span class="sxs-lookup"><span data-stu-id="f997e-150">However, this token is short-lived.</span></span> <span data-ttu-id="f997e-151">トークンは発行後 1 時間で期限切れになります。</span><span class="sxs-lookup"><span data-stu-id="f997e-151">The token expires an hour after it is issued.</span></span> <span data-ttu-id="f997e-152">ここで、更新トークンが役に立ちます。</span><span class="sxs-lookup"><span data-stu-id="f997e-152">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="f997e-153">更新トークンを使用すると、ユーザーが再度サインインしなくても、アプリは新しいアクセス トークンを要求できます。</span><span class="sxs-lookup"><span data-stu-id="f997e-153">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="f997e-154">アプリは Microsoft.Identity.Web ライブラリを使用しているので、トークンストレージまたは更新ロジックを実装する必要は一切ない。</span><span class="sxs-lookup"><span data-stu-id="f997e-154">Because the app is using the Microsoft.Identity.Web library, you do not have to implement any token storage or refresh logic.</span></span>

<span data-ttu-id="f997e-155">アプリはメモリ内トークン キャッシュを使用しますが、アプリの再起動時にトークンを保持する必要がないアプリでは十分です。</span><span class="sxs-lookup"><span data-stu-id="f997e-155">The app uses the in-memory token cache, which is sufficient for apps that do not need to persist tokens when the app restarts.</span></span> <span data-ttu-id="f997e-156">実稼働アプリでは、代わりに[](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization)Microsoft.Identity.Web ライブラリの分散キャッシュ オプションを使用できます。</span><span class="sxs-lookup"><span data-stu-id="f997e-156">Production apps may instead use the [distributed cache options](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in the Microsoft.Identity.Web library.</span></span>

<span data-ttu-id="f997e-157">この `GetAccessTokenForUserAsync` メソッドは、トークンの有効期限と更新を処理します。</span><span class="sxs-lookup"><span data-stu-id="f997e-157">The `GetAccessTokenForUserAsync` method handles token expiration and refresh for you.</span></span> <span data-ttu-id="f997e-158">最初にキャッシュされたトークンをチェックし、有効期限が切れていない場合はトークンを返します。</span><span class="sxs-lookup"><span data-stu-id="f997e-158">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="f997e-159">有効期限が切れている場合は、キャッシュされた更新トークンを使用して新しい更新トークンを取得します。</span><span class="sxs-lookup"><span data-stu-id="f997e-159">If it is expired, it uses the cached refresh token to obtain a new one.</span></span>

<span data-ttu-id="f997e-160">コントローラー **が依存関係挿入** を介して取得する GraphServiceClient は、自動的に使用する認証プロバイダーで事前 `GetAccessTokenForUserAsync` に構成されます。</span><span class="sxs-lookup"><span data-stu-id="f997e-160">The **GraphServiceClient** that controllers get via dependency injection will be pre-configured with an authentication provider that uses `GetAccessTokenForUserAsync` for you.</span></span>
