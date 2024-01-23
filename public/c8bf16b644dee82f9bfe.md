---
title: VBAをリボンUIに追加する（インストーラー付き）
tags:
  - Excel
  - VBScript
  - VBA
private: false
updated_at: '2018-08-05T23:22:33+09:00'
id: c8bf16b644dee82f9bfe
organization_url_name: null
slide: false
ignorePublish: false
---
# 最初に
そもそもこの記事を書こうと思ったのは，業務に使うVBAアプリを叩くときに

 - マクロをショートカットキーじゃなくてOfficeのリボンUIから叩けるようにしたい．
 - 引き継ぎとかを考えてVBAとかマクロとかよく分からない人でも簡単にアプリを使えるようにしたい．

と思ったことがきっかけ．

## 開発環境と実行環境
- Windows 10
- Excel2016

## 配布作業の簡単化（インストーラー）
作ったVBAとかをExcelの機能としてリボンUIに追加する場合は，
1. Excelアドイン作成
2. C:\Users\\{ユーザ名}\AppData\Roaming\Microsoft\AddIns に配置
3. Excelで有効化
4. リボンUIに追加
という作業をしなければならないのですが，細かいところを知らない人に毎回これを教えるのは大変なので一連の作業をスクリプトでやってくれるようにインストーラー的なのも書くことにしました．

## 完成形
- リボンUIからボタンを押して，"Hello World!"を表示するメッセージボックスを出せるようにします．
- インストーラーも作成

# 作業手順
1. VBAの作成
2. リボンUIをXMLで定義（デザイン）
3. アドインとして保存
4. インストーラーの作成

## 1. VBAの作成
Excelでマクロ付きブックを新規作成し，リボンメニューの「開発」から「Visual Basic」を起動します．
メッセージボックスを表示するSubはhello()としました．

```basic:Book1.xlsm
Sub hello(ByVal control As IRibbonControl)
    MsgBox "Hello World!"
End Sub
```

引数の `ByVal control As IRibbonControl` はリボン（Excel本体）から呼び出す際コントローラを渡すらしいので，これを書かないとリボンから起動するときに引数エラーが表示されます．（※意外とこれ忘れがち）

## 2. リボンUIをXMLで定義
リボンUIをカスタマイズしたいときはxlsm内のXMLファイルで定義します．
ご存じの方もいるかと思いますが，xlsmやxlsxはZIP書庫なので拡張子を変えれば解凍圧縮できます．

で実際にXMLを書くときそれぞれ決まったファイル名や宣言が必要なのですが，いちいちそれを書くのも面倒なので「Custom UI Editor Tool（以下，CustomUI）」を使用しました．
（DL先：http://openxmldeveloper.org/blog/b/openxmldeveloper/archive/2006/05/26/customuieditor.aspx ）

1. CustomUIをインストールしたら，さきほど保存したマクロ付きブックを直接開きます．
2. メニューバーの「Insert」→「Sample XML」→「Custom Tab」
3. カスタムタブ用のサンプルXMLが自動生成されます．

サンプルXML自動生成後のスクリーンショット
![CustomUI.jpg](https://qiita-image-store.s3.amazonaws.com/0/145838/730d01c7-dce7-aa38-c506-ae069a012dac.jpeg)

ちなみに自動生成されたXMLファイル名が`customUI14.xml`とっていますが，この「14」がつくとOffice2010以降のXMLらしく，つかないとOffice2007以前のXMLになるらしいです．（ファイル文頭の宣言部も若干異なります）

このツールのお陰でXMLが自動生成されましたが，サンプルの状態だとニッコリ顔 :smiley: のボタンが追加されているだけなので，適宜XMLファイルを変更します．（今回は例なのでこのデザインで進めます）
カスタムUIの定義については下の記事で詳しく説明されているので，そちらをお読みください．

Excel のリボンUIを業務アプリとして使うhttp://qiita.com/tomochan154/items/3614b6f3ebc9ef947719#%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0-ui-%E3%82%92%E5%AE%9A%E7%BE%A9%E3%81%99%E3%82%8B

### 2.1 CustomUIで日本語を使用する
今回使用しているCustomUI，実は日本語（というか多バイト文字？）に対応しておらず，いくら入力しても`，`に変換されてしまいます．
ですので日本語のメニューを使用したい人はxlsmファイルを直接解凍して，自動生成されたcustomUI14.xmlを編集したほうが早いと思います．
cutomUI14.xmlは解凍したディレクトリの中の \\customUI\ の下にあります．
編集後は再度ZIPで圧縮して拡張子を.xlsmに戻せば大丈夫です．

### 2.2 ボタン押下時のアクション
ボタンを押されたときにどのアクションを呼ぶかを `onAction` に書きます．
サンプルだと`Callback`になっているので，今回は先ほど作成したSub（hello）にします．
その場合のButton定義はこんな感じ．

```xml:cutomUI14.xml
	<button id="customButton" label="Custom Button" imageMso="HappyFace" size="large" onAction="hello" />
```

### 2.3 自分の画像を使いたい（画像の読み込み）
ボタンのアイコンを自分が用意した画像にしたいときがあります．
そんなときはCustomUIでメニューバー「Insert」→「Icons...」で指定してあげればファイル内に組み込んでくれます．
読み込んだ画像は自動的にIDが振られるので，そのIDを`image=`に書きます．
振られたIDは左側のファイルツリーで確認できます．
※標準で組み込まれている画像を使用するときは`imageMso=`ですが，自分が用意した画像を使用する際は`image=`になりますのでご注意．
（標準で組み込まれている画像一覧はこちらの記事をご覧ください．https://www.ka-net.org/blog/?p=5201 ）

## 3. アドインとして保存
ここまでの段階で，VBAとカスタマイズしたリボンUIが一緒になったマクロ付きブックができている状態です．
なのでこのブックを名前をつけて保存で「Excelアドイン(.xlam)」として保存してあげます．

※このときデフォルトの保存先が C:\Users\\{ユーザ名}\AppData\Roaming\Microsoft\AddIns になっていると思うので，デスクトップ等好きな場所に一旦保存します．
デフォルトの保存先にしてしまうとそのまますぐExcelアドインとして登録されてしまうので，あとでインストーラーを作るときに混乱するのでオススメしません．

## 4. インストーラーの作成
基本的にこちらの記事を参考にしてます．http://qiita.com/fuku2014/items/9c72fc04265bfc7f7f40
今回はインストール確認ウィンドウなどを追加しました．

スクリプトの基本的な流れは
1. ファイルをAddInsにコピー
2. Excelでアドイン登録
3. アドイン有効化

下のスクリプトをテキストエディタで書いて，`Install.vbs`として保存します．
※ファイルのエンコードは **Shift-JIS** でおこなってください．

```basic:Install.vbs
Const FILLE_NAME="Book1.xlam"

Call Exec

Sub Exec()
    Dim objExcel
    Dim strAdPath
    Dim strMyPath
    Dim strAdCp
    Dim strMyCp
    Dim objFileSys
    Dim oAdd

    ' イントール確認ウィンドウ
    IF MsgBox("アドインをイントールしますか？", vbYesNo + vbQuestion) = vbNo Then
        WScript.Quit
    End IF
 
    ' Excelインタンス化
    Set objExcel   = CreateObject("Excel.Application")
    Set objFileSys = CreateObject("Scripting.FileSystemObject")

    ' パス設定
    strAdPath = objExcel.Application.UserLibraryPath
    strMyPath = Replace(WScript.ScriptFullName, WScript.ScriptName, "")
    strAdCp   = objFileSys.BuildPath(strAdPath, FILLE_NAME)
    strMyCp   = objFileSys.BuildPath(strMyPath, FILLE_NAME)

    ' ファイルコピー
    objFileSys.CopyFile strMyCp, strAdCp

    ' アドイン登録
    objExcel.Workbooks.Add
    Set oAdd = objExcel.AddIns.Add(strAdCp,True)
    ' アドイン有効化
    oAdd.Installed = True
    objExcel.Quit

    Set objExcel   = Nothing
    Set objFileSys = Nothing

    MsgBox "イントールが完了しました"
End Sub
```

アンインストーラーはインストーラーの逆の手順を行えば大丈夫です．
（アドイン無効化→登録解除→ファイル削除）

### 4.1 配布方法とイントール方法
作成したExcelアドインとインストーラーを同じディレクトリに入れて配布してください．
イントールする際はInstall.vbsを叩けば終わりです．

というわけで完成！
![キャプチャ.PNG](https://qiita-image-store.s3.amazonaws.com/0/145838/1451bb51-82ec-6528-66e0-6720080ee075.png)

### 4.1.1 ネット経由での配布時の注意事項
（2018/08/05追記）
インターネット経由でこのアドインを配布する場合，「ブロックの解除」をする必要があります．
もしインストールをしたのに，リボンUIに出てこない方はブロックの解除をお試しください．
ブロックの解除方法は下記URL参照のこと．
http://www.atmarkit.co.jp/ait/articles/1603/11/news050.html

実際に配布する際は
1. インストーラーとアドイン本体を配る
2. 落としてきたアドイン本体のブロックを解除してもらう
3. インストーラー起動
になりますかね．

# 最後に
VBAとリボンUIに関する記事はネット上に落ちてはいるのですが，インストーラーも含めて一つの記事としてまとまっているのが見つからなかったので，書いてみました．
たったこれだけの完成物に対しては結構なボリュームの記事になってしまいました．
VBA＋リボンUIのカスタマイズというところにどのくらいの需要があるのか分かりませんが，少しでもお役に立てれば幸いです．

（2018/08/05追記）
こんな誰得記事でも「いいね！」を付けてくださる方が居て有難い限りです．
今どきVBAなんて誰が使うんだよ．とお思いの方がいるでしょう．まさにその通り！
でも社内の業務効率化や内製でなんとかするしかない人のために少しでも力になれれば幸いです．
