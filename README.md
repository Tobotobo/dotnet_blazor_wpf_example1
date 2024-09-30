# dotnet_blazor_wpf_example1

## 概要
* まだ途中　※エラーになる

## 環境
```
$ dotnet --info
.NET SDK:
 Version:           8.0.400
 Commit:            36fe6dda56
 Workload version:  8.0.400-manifests.f51a3a6b
 MSBuild version:   17.11.3+0c8610977

ランタイム環境:
 OS Name:     ubuntu
 OS Version:  24.04
 OS Platform: Linux
 RID:         linux-x64
 Base Path:   /usr/lib/dotnet/sdk/8.0.400/
```

## 詳細

```
dotnet new sln -n ExampleApp
dotnet new gitignore
dotnet new editorconfig
```

```
dotnet new blazor --all-interactive true --no-https -n ExampleApp.Web
dotnet sln add ExampleApp.Web
```

```
dotnet run --project ExampleApp.Web
dotnet watch --project ExampleApp.Web
```

[Linux 環境に WPF のテンプレートをインストールする](https://github.com/Tobotobo/my_knowledge_base/issues/30)
  
```
dotnet new wpf -n ExampleApp.WPF
dotnet sln add ExampleApp.WPF
dotnet add ExampleApp.WPF reference ExampleApp.Web
dotnet add ExampleApp.Web package Microsoft.AspNetCore.Components.WebView.Wpf
```
※以下のエラーが発生する（この後対応するので一旦無視でOK）
```
Error NETSDK1100: このオペレーティング システムで Windows を対象とするプロジェクトをビルドするには、EnableWindowsTargeting プロパティを true に設定します。 
```

ExampleApp.WPF/ExampleApp.WPF.csproj
```xml
<Project Sdk="Microsoft.NET.Sdk.Razor">　※Microsoft.NET.Sdk から変更
<PropertyGroup>
    〜〜〜〜〜〜〜〜〜〜
    <EnableWindowsTargeting>true</EnableWindowsTargeting>　※追加
    〜〜〜〜〜〜〜〜〜〜
</PropertyGroup>
```

ExampleApp.WPF/MainWindow.xaml
```xml
<Window x:Class="ExampleApp.WPF.MainWindow"
〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜
        xmlns:blazor="clr-namespace:Microsoft.AspNetCore.Components.WebView.Wpf;assembly=Microsoft.AspNetCore.Components.WebView.Wpf"
        xmlns:components="clr-namespace:ExampleApp.Web.Components;assembly=ExampleApp.Web"
〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜>
    <Grid>
        <blazor:BlazorWebView HostPage="wwwroot/index.html" Services="{DynamicResource services}">
            <blazor:BlazorWebView.RootComponents>
                <blazor:RootComponent Selector="#app" ComponentType="{x:Type components:Routes}" />
            </blazor:BlazorWebView.RootComponents>
        </blazor:BlazorWebView>
    </Grid>
</Window>
```

ExampleApp.WPF/MainWindow.xaml.cs
```cs
〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜
using Microsoft.Extensions.DependencyInjection;
〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        var serviceCollection = new ServiceCollection();
        serviceCollection.AddWpfBlazorWebView();
        var services = serviceCollection.BuildServiceProvider();
        Resources.Add("services", services);
    }
}
```

wwwroot フォルダを作成し、中に以下の index.html を作成
```html
<!DOCTYPE html>
<html lang="jp">

<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
    <base href="/" />
    <link href="_content/ExampleApp.Web/app.css" rel="stylesheet" />
    <link rel="stylesheet" href="ExampleApp.WPF.styles.css" />
</head>

<body>
    <div id="app">Loading...</div>

    <div id="blazor-error-ui" data-nosnippet>
        An unhandled error has occurred.
        <a href="" class="reload">Reload</a>
        <a class="dismiss">🗙</a>
    </div>
    <script src="_framework/blazor.webview.js"></script>
</body>

</html>
``` 



exe 作成
```
dotnet publish -r win-x64 --self-contained ExampleApp.WPF -o publish/ExampleApp.WPF
```



