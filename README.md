# dotnet_blazor_wpf_example1

## 概要
* WPF + Blazor を Linux 環境（リモートを想定）で開発する
* 極力最小限の構成とする

## 開発イメージ
* Docker 等に SSH で接続可能な .NET8 開発コンテナを構築する
* VSCode + Remote-SSH で上記コンテナに接続して開発する
* 画面は ExampleApp.Web を watch で実行し、ブラウザで確認しながら開発する
* Windows 環境でないと動かない機能は、別途開発し依存性の注入で対応する
* 当該箇所を ExampleApp.Web で実行する際はモックを用いる 

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
        serviceCollection.AddBlazorWebViewDeveloperTools(); // F12 で開発者ツールが起動する
        var services = serviceCollection.BuildServiceProvider();
        Resources.Add("services", services);
    }
}
```

wwwroot フォルダを作成し、中に以下の index.html を作成
```html
<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
    <base href="/" />
    <link rel="stylesheet" href="bootstrap/bootstrap.min.css" />
    <link rel="stylesheet" href="app.css" />
    <link rel="stylesheet" href="ExampleApp.Web.styles.css" />
</head>

<body>
    <div id="app">Loading...</div>
    <script src="_framework/blazor.webview.js"></script>
</body>

</html>
``` 

exe 作成
```
dotnet publish -r win-x64 --self-contained ExampleApp.WPF -o publish/ExampleApp.WPF
```

実行
```
dotnet run --project ExampleApp.WPF
dotnet watch --project ExampleApp.WPF
```
