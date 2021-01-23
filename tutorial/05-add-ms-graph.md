---
ms.openlocfilehash: c954903f38d48bcca4c534f4d0cfbe1605cbf7f6
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942149"
---
<!-- markdownlint-disable MD002 MD041 -->

このセクションでは、Microsoft Graph をアプリケーションに組み込む必要があります。 このアプリケーションでは [、.NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) 用 Microsoft Graph クライアント ライブラリを使用して Microsoft Graph を呼び出します。

## <a name="get-calendar-events-from-outlook"></a>Outlook からカレンダー イベントを取得する

まず、予定表ビュー用の新しいコントローラーを作成します。

1. **./Controllers** ディレクトリに **CalendarController.cs** という名前の新しいファイルを追加し、次のコードを追加します。

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

1. 次の関数をクラスに `CalendarController` 追加して、ユーザーの予定表ビューを取得します。

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    コードの動作を検討 `GetUserWeekCalendar` します。

    - ユーザーのタイム ゾーンを使用して、週の UTC 開始日と終了日時の値を取得します。
    - ユーザーの予定表ビューに対して [クエリを実行](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) し、開始日時と終了日付/時刻の間のすべてのイベントを取得します。 イベントを一覧に表示する[](/graph/api/user-list-events?view=graph-rest-1.0)代わりにカレンダー ビューを使用すると、定期的なイベントが展開され、指定したタイム ウィンドウで発生したイベントが返されます。
    - ヘッダーを `Prefer: outlook.timezone` 使用して、ユーザーのタイムゾーンで結果を返します。
    - アプリで `Select` 使用されるフィールドにだけ戻るフィールドを制限するために使用されます。
    - この例では `OrderBy` 、結果を時系列的に並べ替えるのに使用されます。
    - これは、a `PageIterator` を使用 [してイベント コレクションをページ移動します](/graph/sdks/paging)。 これにより、ユーザーの予定表に要求されたページ サイズよりも多くのイベントがある場合に処理されます。

1. 次の関数をクラスに追加 `CalendarController` して、返されるデータの一時的なビューを実装します。

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

1. アプリを起動し、サインインして、ナビゲーション バーの **[カレンダー]** リンクをクリックします。 すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。

## <a name="display-the-results"></a>結果の表示

結果を表示するとき、よりユーザー フレンドリなビューを追加できます。

### <a name="create-view-models"></a>ビュー モデルを作成する

1. **./Models** ディレクトリに **CalendarViewEvent.cs** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. **./Models** ディレクトリに **DailyViewModel.cs** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. **./Models** ディレクトリに **CalendarViewModel.cs** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a>ビューを作成する

1. **./Views** ディレクトリに Calendar という名前の **新しいディレクトリを作成** します。

1. **./Views/Calendar** ディレクトリに **_DailyEventsPartial.cshtml** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. **./Views/Calendar** ディレクトリに **Index.cshtml** という名前の新しいファイルを作成し、次のコードを追加します。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a>予定表コントローラーを更新する

1. **./Controllers/CalendarController.cs** を開き、既存の関数を次の `Index` 関数に置き換える。

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. アプリを起動し、サインインして、**[カレンダー]** リンクをクリックします。 アプリにイベントの表が表示されます。

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
