---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942156"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="02d63-101">まず、コア Web アプリASP.NET作成します。</span><span class="sxs-lookup"><span data-stu-id="02d63-101">Start by creating an ASP.NET Core web app.</span></span>

1. <span data-ttu-id="02d63-102">プロジェクトを作成するディレクトリでコマンド ライン インターフェイス (CLI) を開きます。</span><span class="sxs-lookup"><span data-stu-id="02d63-102">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="02d63-103">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="02d63-103">Run the following command.</span></span>

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. <span data-ttu-id="02d63-104">プロジェクトが作成された後、現在のディレクトリを **GraphTu clil** ディレクトリに変更し、CLI で次のコマンドを実行して、プロジェクトが動作を確認します。</span><span class="sxs-lookup"><span data-stu-id="02d63-104">Once the project is created, verify that it works by changing the current directory to the **GraphTutorial** directory and running the following command in your CLI.</span></span>

    ```Shell
    dotnet run
    ```

1. <span data-ttu-id="02d63-105">ブラウザーを開き、参照します `https://localhost:5001` 。</span><span class="sxs-lookup"><span data-stu-id="02d63-105">Open your browser and browse to `https://localhost:5001`.</span></span> <span data-ttu-id="02d63-106">すべてが動作している場合は、既定の [コア] ページASP.NET表示されます。</span><span class="sxs-lookup"><span data-stu-id="02d63-106">If everything is working, you should see a default ASP.NET Core page.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="02d63-107">**localhost** の証明書が信頼されていないという警告を受け取った場合は、.NET Core CLI を使用して開発証明書をインストールして信頼できます。</span><span class="sxs-lookup"><span data-stu-id="02d63-107">If you receive a warning that the certificate for **localhost** is un-trusted you can use the .NET Core CLI to install and trust the development certificate.</span></span> <span data-ttu-id="02d63-108">特定 [のオペレーティング システムの手順については、「ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) での HTTPS の適用」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="02d63-108">See [Enforce HTTPS in ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) for instructions for specific operating systems.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="02d63-109">NuGet パッケージを追加する</span><span class="sxs-lookup"><span data-stu-id="02d63-109">Add NuGet packages</span></span>

<span data-ttu-id="02d63-110">次に進む前に、後で使用する追加の NuGet パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="02d63-110">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="02d63-111">アクセス トークンを要求および管理するための[Microsoft.Identity.Web。](https://www.nuget.org/packages/Microsoft.Identity.Web/)</span><span class="sxs-lookup"><span data-stu-id="02d63-111">[Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) for requesting and managing access tokens.</span></span>
- <span data-ttu-id="02d63-112">依存関係挿入を使用して Microsoft Graph SDK を追加する[Microsoft.Identity.Web.MicrosoftGraph。](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/)</span><span class="sxs-lookup"><span data-stu-id="02d63-112">[Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) for adding the Microsoft Graph SDK via dependency injection.</span></span>
- <span data-ttu-id="02d63-113">サインインおよびサインアウト UI 用の[Microsoft.Identity.Web.UI。](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/)</span><span class="sxs-lookup"><span data-stu-id="02d63-113">[Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) for sign-in and sign-out UI.</span></span>
- <span data-ttu-id="02d63-114">タイム ゾーン ID をプラットフォーム間で処理する[TimeZoneConverter。](https://github.com/mj1856/TimeZoneConverter)</span><span class="sxs-lookup"><span data-stu-id="02d63-114">[TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) for handling time zoned identifiers cross-platform.</span></span>

1. <span data-ttu-id="02d63-115">CLI で次のコマンドを実行して、依存関係をインストールします。</span><span class="sxs-lookup"><span data-stu-id="02d63-115">Run the following commands in your CLI to install the dependencies.</span></span>

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a><span data-ttu-id="02d63-116">アプリを設計する</span><span class="sxs-lookup"><span data-stu-id="02d63-116">Design the app</span></span>

<span data-ttu-id="02d63-117">このセクションでは、アプリケーションの基本的な UI 構造を作成します。</span><span class="sxs-lookup"><span data-stu-id="02d63-117">In this section you will create the basic UI structure of the application.</span></span>

### <a name="implement-alert-extension-methods"></a><span data-ttu-id="02d63-118">アラート拡張メソッドを実装する</span><span class="sxs-lookup"><span data-stu-id="02d63-118">Implement alert extension methods</span></span>

<span data-ttu-id="02d63-119">このセクションでは、コントローラー ビューによって返される型の `IActionResult` 拡張メソッドを作成します。</span><span class="sxs-lookup"><span data-stu-id="02d63-119">In this section you will create extension methods for the `IActionResult` type returned by controller views.</span></span> <span data-ttu-id="02d63-120">この拡張機能により、一時的なエラーまたは成功メッセージをビューに渡す機能が有効になります。</span><span class="sxs-lookup"><span data-stu-id="02d63-120">This extension will enable passing temporary error or success messages to the view.</span></span>

> [!TIP]
> <span data-ttu-id="02d63-121">このチュートリアルでは、任意のテキスト エディターを使用してソース ファイルを編集できます。</span><span class="sxs-lookup"><span data-stu-id="02d63-121">You can use any text editor to edit the source files for this tutorial.</span></span> <span data-ttu-id="02d63-122">ただし [、Visual Studioコードには](https://code.visualstudio.com/) 、デバッグや Intellisense などの追加機能があります。</span><span class="sxs-lookup"><span data-stu-id="02d63-122">However, [Visual Studio Code](https://code.visualstudio.com/) provides additional features, such as debugging and Intellisense.</span></span>

1. <span data-ttu-id="02d63-123">Alerts という名前の **GraphTu読み込みディレクトリに** 新しいディレクトリを **作成します**。</span><span class="sxs-lookup"><span data-stu-id="02d63-123">Create a new directory in the **GraphTutorial** directory named **Alerts**.</span></span>

1. <span data-ttu-id="02d63-124">**./Alerts** ディレクトリに **WithAlertResult.cs** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="02d63-124">Create a new file named **WithAlertResult.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. <span data-ttu-id="02d63-125">**./Alerts** ディレクトリに **AlertExtensions.cs** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="02d63-125">Create a new file named **AlertExtensions.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a><span data-ttu-id="02d63-126">ユーザー データ拡張メソッドを実装する</span><span class="sxs-lookup"><span data-stu-id="02d63-126">Implement user data extension methods</span></span>

<span data-ttu-id="02d63-127">このセクションでは、Microsoft Identity プラットフォームによって生成されるオブジェクト `ClaimsPrincipal` の拡張メソッドを作成します。</span><span class="sxs-lookup"><span data-stu-id="02d63-127">In this section you will create extension methods for the `ClaimsPrincipal` object generated by the Microsoft Identity platform.</span></span> <span data-ttu-id="02d63-128">これにより、既存のユーザー ID を Microsoft Graph のデータで拡張できます。</span><span class="sxs-lookup"><span data-stu-id="02d63-128">This will allow you to extend the existing user identity with data from Microsoft Graph.</span></span>

> [!NOTE]
> <span data-ttu-id="02d63-129">このコードは、今のところプレースホルダーにすがっています。後のセクションで完成します。</span><span class="sxs-lookup"><span data-stu-id="02d63-129">This code is just a placeholder for now, you will complete it in a later section.</span></span>

1. <span data-ttu-id="02d63-130">Graph という名前の **GraphTu graphl ディレクトリに新** しいディレクトリを作成 **します**。</span><span class="sxs-lookup"><span data-stu-id="02d63-130">Create a new directory in the **GraphTutorial** directory named **Graph**.</span></span>

1. <span data-ttu-id="02d63-131">新しいファイルを GraphClaimsPrincipalExtensions.csし **、** 次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="02d63-131">Create a new file named **GraphClaimsPrincipalExtensions.cs** and add the following code.</span></span>

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

### <a name="create-views"></a><span data-ttu-id="02d63-132">ビューを作成する</span><span class="sxs-lookup"><span data-stu-id="02d63-132">Create views</span></span>

<span data-ttu-id="02d63-133">このセクションでは、アプリケーションの "多" ビューを実装します。</span><span class="sxs-lookup"><span data-stu-id="02d63-133">In this section you will implement the Razor views for the application.</span></span>

1. <span data-ttu-id="02d63-134">**_LoginPartial.cshtml という** 名前の新しいファイルを **./Views/Shared** ディレクトリに追加し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="02d63-134">Add a new file named **_LoginPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. <span data-ttu-id="02d63-135">**_AlertPartial.cshtml という名前** の新しいファイルを **./Views/Shared** ディレクトリに追加し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="02d63-135">Add a new file named **_AlertPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. <span data-ttu-id="02d63-136">アプリのグローバル レイアウトを更新するため、**./Views/Shared/_Layout.cshtml** ファイルを開き、内容全体を次のコードで置き換えます。</span><span class="sxs-lookup"><span data-stu-id="02d63-136">Open the **./Views/Shared/_Layout.cshtml** file, and replace its entire contents with the following code to update the global layout of the app.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. <span data-ttu-id="02d63-137">**./wwwroot/css/site.css** を開き、ファイルの下部に次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="02d63-137">Open **./wwwroot/css/site.css** and add the following code at the bottom of the file.</span></span>

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. <span data-ttu-id="02d63-138">**./Views/Home/index.cshtml** ファイルを開き、その内容を次の内容に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="02d63-138">Open the **./Views/Home/index.cshtml** file and replace its contents with the following.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. <span data-ttu-id="02d63-139">img という名前の **./wwwroot ディレクトリに新しい** ディレクトリを作成 **します**。</span><span class="sxs-lookup"><span data-stu-id="02d63-139">Create a new directory in the **./wwwroot** directory named **img**.</span></span> <span data-ttu-id="02d63-140">このディレクトリに、選択した名前 **のno-profile-photo.pngファイル** を追加します。</span><span class="sxs-lookup"><span data-stu-id="02d63-140">Add an image file of your choosing named **no-profile-photo.png** in this directory.</span></span> <span data-ttu-id="02d63-141">この画像は、ユーザーが Microsoft Graph に写真を持たない場合に、ユーザーの写真として使用されます。</span><span class="sxs-lookup"><span data-stu-id="02d63-141">This image will be used as the user's photo when the user has no photo in Microsoft Graph.</span></span>

    > [!TIP]
    > <span data-ttu-id="02d63-142">これらのスクリーンショットで使用されている画像は [、GitHub からダウンロードできます](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)。</span><span class="sxs-lookup"><span data-stu-id="02d63-142">You can download the image used in these screenshots from [GitHub](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png).</span></span>

1. <span data-ttu-id="02d63-143">すべての変更を保存し、サーバー () を再起動します `dotnet run` 。</span><span class="sxs-lookup"><span data-stu-id="02d63-143">Save all of your changes and restart the server (`dotnet run`).</span></span> <span data-ttu-id="02d63-144">これで、アプリは非常に異なって表示されます。</span><span class="sxs-lookup"><span data-stu-id="02d63-144">Now, the app should look very different.</span></span>

    ![デザインが変更されたホーム ページのスクリーンショット](./images/create-app-01.png)
