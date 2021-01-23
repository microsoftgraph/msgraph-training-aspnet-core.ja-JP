---
ms.openlocfilehash: 73ba9c271c9c6675ffbb22ce4f800ffb652c715d
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942135"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="9abf5-101">このチュートリアルでは、Microsoft Graph API を使用してユーザーの予定表情報ASP.NETコア Web アプリを構築する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="9abf5-101">This tutorial teaches you how to build an ASP.NET Core web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="9abf5-102">完成したチュートリアルをダウンロードする場合は [、GitHub](https://github.com/microsoftgraph/msgraph-training-aspnet-core)リポジトリをダウンロードまたは複製できます。</span><span class="sxs-lookup"><span data-stu-id="9abf5-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnet-core).</span></span> <span data-ttu-id="9abf5-103">アプリ ID とシークレットを使ってアプリを構成する手順については、デモ フォルダーの README ファイルを参照してください。</span><span class="sxs-lookup"><span data-stu-id="9abf5-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="9abf5-104">前提条件</span><span class="sxs-lookup"><span data-stu-id="9abf5-104">Prerequisites</span></span>

<span data-ttu-id="9abf5-105">このチュートリアルを開始する前に、開発用コンピューターに [.NET Core SDK](https://dotnet.microsoft.com/download) がインストールされている必要があります。</span><span class="sxs-lookup"><span data-stu-id="9abf5-105">Before you start this tutorial, you should have the [.NET Core SDK](https://dotnet.microsoft.com/download) installed on your development machine.</span></span> <span data-ttu-id="9abf5-106">SDK がない場合は、ダウンロード オプションの前のリンクを参照してください。</span><span class="sxs-lookup"><span data-stu-id="9abf5-106">If you do not have the SDK, visit the previous link for download options.</span></span>

<span data-ttu-id="9abf5-107">また、メールボックスを持つ個人用の Microsoft アカウントが Outlook.com Microsoft の仕事用アカウントまたは学校アカウントである必要があります。</span><span class="sxs-lookup"><span data-stu-id="9abf5-107">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="9abf5-108">Microsoft アカウントをお持ちない場合は、無料アカウントを取得するためのオプションが 2 つ提供されています。</span><span class="sxs-lookup"><span data-stu-id="9abf5-108">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="9abf5-109">新しい [個人用 Microsoft アカウントにサインアップできます](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。</span><span class="sxs-lookup"><span data-stu-id="9abf5-109">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="9abf5-110">Office [365 開発者プログラムにサインアップして、365](https://developer.microsoft.com/office/dev-program) サブスクリプションを無料Office取得できます。</span><span class="sxs-lookup"><span data-stu-id="9abf5-110">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="9abf5-111">このチュートリアルは、.NET Core SDK バージョン 5.0.102 で記述されています。</span><span class="sxs-lookup"><span data-stu-id="9abf5-111">This tutorial was written with .NET Core SDK version 5.0.102.</span></span> <span data-ttu-id="9abf5-112">このガイドの手順は他のバージョンでも動作する可能性がありますが、テストは行ってはいではありません。</span><span class="sxs-lookup"><span data-stu-id="9abf5-112">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="9abf5-113">フィードバック</span><span class="sxs-lookup"><span data-stu-id="9abf5-113">Feedback</span></span>

<span data-ttu-id="9abf5-114">このチュートリアルに関するフィードバックは [、GitHub リポジトリで提供してください](https://github.com/microsoftgraph/msgraph-training-aspnet-core)。</span><span class="sxs-lookup"><span data-stu-id="9abf5-114">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnet-core).</span></span>
