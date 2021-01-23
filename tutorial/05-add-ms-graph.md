---
ms.openlocfilehash: c954903f38d48bcca4c534f4d0cfbe1605cbf7f6
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942149"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ff647-101">このセクションでは、Microsoft Graph をアプリケーションに組み込む必要があります。</span><span class="sxs-lookup"><span data-stu-id="ff647-101">In this section you will incorporate Microsoft Graph into the application.</span></span> <span data-ttu-id="ff647-102">このアプリケーションでは [、.NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) 用 Microsoft Graph クライアント ライブラリを使用して Microsoft Graph を呼び出します。</span><span class="sxs-lookup"><span data-stu-id="ff647-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="ff647-103">Outlook からカレンダー イベントを取得する</span><span class="sxs-lookup"><span data-stu-id="ff647-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="ff647-104">まず、予定表ビュー用の新しいコントローラーを作成します。</span><span class="sxs-lookup"><span data-stu-id="ff647-104">Start by creating a new controller for calendar views.</span></span>

1. <span data-ttu-id="ff647-105">**./Controllers** ディレクトリに **CalendarController.cs** という名前の新しいファイルを追加し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ff647-105">Add a new file named **CalendarController.cs** in the **./Controllers** directory and add the following code.</span></span>

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using Microsoft.Graph;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using TimeZoneConverter;

    namespace GraphTutorial.Controllers
    {
        public class CalendarController : Controller
        {
            private readonly GraphServiceClient _graphClient;
            private readonly ILogger<HomeController> _logger;

            public CalendarController(
                GraphServiceClient graphClient,
                ILogger<HomeController> logger)
            {
                _graphClient = graphClient;
                _logger = logger;
            }
        }
    }
    ```

1. <span data-ttu-id="ff647-106">次の関数をクラスに `CalendarController` 追加して、ユーザーの予定表ビューを取得します。</span><span class="sxs-lookup"><span data-stu-id="ff647-106">Add the following functions to the `CalendarController` class to get the user's calendar view.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    <span data-ttu-id="ff647-107">コードの動作を検討 `GetUserWeekCalendar` します。</span><span class="sxs-lookup"><span data-stu-id="ff647-107">Consider what the code in `GetUserWeekCalendar` does.</span></span>

    - <span data-ttu-id="ff647-108">ユーザーのタイム ゾーンを使用して、週の UTC 開始日と終了日時の値を取得します。</span><span class="sxs-lookup"><span data-stu-id="ff647-108">It uses the user's time zone to get UTC start and end date/time values for the week.</span></span>
    - <span data-ttu-id="ff647-109">ユーザーの予定表ビューに対して [クエリを実行](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) し、開始日時と終了日付/時刻の間のすべてのイベントを取得します。</span><span class="sxs-lookup"><span data-stu-id="ff647-109">It queries the user's [calendar view](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) to get all events that fall between the start and end date/times.</span></span> <span data-ttu-id="ff647-110">イベントを一覧に表示する[](/graph/api/user-list-events?view=graph-rest-1.0)代わりにカレンダー ビューを使用すると、定期的なイベントが展開され、指定したタイム ウィンドウで発生したイベントが返されます。</span><span class="sxs-lookup"><span data-stu-id="ff647-110">Using a calendar view instead of [listing events](/graph/api/user-list-events?view=graph-rest-1.0) expands recurring events, returning any occurrences that happen in the specified time window.</span></span>
    - <span data-ttu-id="ff647-111">ヘッダーを `Prefer: outlook.timezone` 使用して、ユーザーのタイムゾーンで結果を返します。</span><span class="sxs-lookup"><span data-stu-id="ff647-111">It uses the `Prefer: outlook.timezone` header to get results back in the user's timezone.</span></span>
    - <span data-ttu-id="ff647-112">アプリで `Select` 使用されるフィールドにだけ戻るフィールドを制限するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="ff647-112">It uses `Select` to limit the fields that come back to just those used by the app.</span></span>
    - <span data-ttu-id="ff647-113">この例では `OrderBy` 、結果を時系列的に並べ替えるのに使用されます。</span><span class="sxs-lookup"><span data-stu-id="ff647-113">It uses `OrderBy` to sort the results chronologically.</span></span>
    - <span data-ttu-id="ff647-114">これは、a `PageIterator` を使用 [してイベント コレクションをページ移動します](/graph/sdks/paging)。</span><span class="sxs-lookup"><span data-stu-id="ff647-114">It uses a `PageIterator` to [page through the events collection](/graph/sdks/paging).</span></span> <span data-ttu-id="ff647-115">これにより、ユーザーの予定表に要求されたページ サイズよりも多くのイベントがある場合に処理されます。</span><span class="sxs-lookup"><span data-stu-id="ff647-115">This handles the case where the user has more events on their calendar than the requested page size.</span></span>

1. <span data-ttu-id="ff647-116">次の関数をクラスに追加 `CalendarController` して、返されるデータの一時的なビューを実装します。</span><span class="sxs-lookup"><span data-stu-id="ff647-116">Add the following function to the `CalendarController` class to implement a temporary view of the returned data.</span></span>

    ```csharp
    // Minimum permission scope needed for this view
    [AuthorizeForScopes(Scopes = new[] { "Calendars.Read" })]
    public async Task<IActionResult> Index()
    {
        try
        {
            var userTimeZone = TZConvert.GetTimeZoneInfo(
                User.GetUserGraphTimeZone());
            var startOfWeek = CalendarController.GetUtcStartOfWeekInTimeZone(
                DateTime.Today, userTimeZone);

            var events = await GetUserWeekCalendar(startOfWeek);

            // Return a JSON dump of events
            return new ContentResult {
                Content = _graphClient.HttpProvider.Serializer.SerializeObject(events),
                ContentType = "application/json"
            };
        }
        catch (ServiceException ex)
        {
            if (ex.InnerException is MicrosoftIdentityWebChallengeUserException)
            {
                throw;
            }

            return new ContentResult {
                Content = $"Error getting calendar view: {ex.Message}",
                ContentType = "text/plain"
            };
        }
    }
    ```

1. <span data-ttu-id="ff647-117">アプリを起動し、サインインして、ナビゲーション バーの **[カレンダー]** リンクをクリックします。</span><span class="sxs-lookup"><span data-stu-id="ff647-117">Start the app, sign in, and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="ff647-118">すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。</span><span class="sxs-lookup"><span data-stu-id="ff647-118">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="ff647-119">結果の表示</span><span class="sxs-lookup"><span data-stu-id="ff647-119">Display the results</span></span>

<span data-ttu-id="ff647-120">結果を表示するとき、よりユーザー フレンドリなビューを追加できます。</span><span class="sxs-lookup"><span data-stu-id="ff647-120">Now you can add a view to display the results in a more user-friendly manner.</span></span>

### <a name="create-view-models"></a><span data-ttu-id="ff647-121">ビュー モデルを作成する</span><span class="sxs-lookup"><span data-stu-id="ff647-121">Create view models</span></span>

1. <span data-ttu-id="ff647-122">**./Models** ディレクトリに **CalendarViewEvent.cs** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ff647-122">Create a new file named **CalendarViewEvent.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. <span data-ttu-id="ff647-123">**./Models** ディレクトリに **DailyViewModel.cs** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ff647-123">Create a new file named **DailyViewModel.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. <span data-ttu-id="ff647-124">**./Models** ディレクトリに **CalendarViewModel.cs** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ff647-124">Create a new file named **CalendarViewModel.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a><span data-ttu-id="ff647-125">ビューを作成する</span><span class="sxs-lookup"><span data-stu-id="ff647-125">Create views</span></span>

1. <span data-ttu-id="ff647-126">**./Views** ディレクトリに Calendar という名前の **新しいディレクトリを作成** します。</span><span class="sxs-lookup"><span data-stu-id="ff647-126">Create a new directory named **Calendar** in the **./Views** directory.</span></span>

1. <span data-ttu-id="ff647-127">**./Views/Calendar** ディレクトリに **_DailyEventsPartial.cshtml** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ff647-127">Create a new file named **_DailyEventsPartial.cshtml** in the **./Views/Calendar** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. <span data-ttu-id="ff647-128">**./Views/Calendar** ディレクトリに **Index.cshtml** という名前の新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="ff647-128">Create a new file named **Index.cshtml** in the **./Views/Calendar** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a><span data-ttu-id="ff647-129">予定表コントローラーを更新する</span><span class="sxs-lookup"><span data-stu-id="ff647-129">Update calendar controller</span></span>

1. <span data-ttu-id="ff647-130">**./Controllers/CalendarController.cs** を開き、既存の関数を次の `Index` 関数に置き換える。</span><span class="sxs-lookup"><span data-stu-id="ff647-130">Open **./Controllers/CalendarController.cs** and replace the existing `Index` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. <span data-ttu-id="ff647-131">アプリを起動し、サインインして、**[カレンダー]** リンクをクリックします。</span><span class="sxs-lookup"><span data-stu-id="ff647-131">Start the app, sign in, and click the **Calendar** link.</span></span> <span data-ttu-id="ff647-132">アプリにイベントの表が表示されます。</span><span class="sxs-lookup"><span data-stu-id="ff647-132">The app should now render a table of events.</span></span>

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
