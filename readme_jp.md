# CSharpVitamins.ShortGuid
URL セーフな Base64 でエンコードされたグローバル一意識別子 (GUID) を扱うための便利なラッパーで、36 文字の GUID を 22 文字の短い文字列にします。

URL セーフな Base64 とは、よく知られた特殊文字 (/, +) を置き換え、末尾の == を取り除くことで、一貫して 22 文字でありつつ URI フレンドリな Base64 文字列にしたものです。

もともとは [@madskristensen](https://github.com/madskristensen) 氏のブログで紹介されていたアイデアを発展させたヘルパークラスで、長い間は [2007 年ごろのブログ記事](https://www.singular.co.nz/2007/12/shortguid-a-shorter-and-url-friendly-guid-in-c-sharp/) としてのみ存在していました。


このライブラリは [NuGet](https://www.nuget.org/packages/csharpvitamins.shortguid/) から入手できます。インストールするには Package Manager Console で次のコマンドを実行します。

    PM> Install-Package CSharpVitamins.ShortGuid

### 厳密なパース (Strict Parsing, v2.0.0)
バージョン 2.0.0 以降、`ShortGuid` は文字列をデコードする際にサニティチェックを行い、文字列が改ざんされていないことを検証します。具体的には、Base64 文字列の末尾を変更しても同じバイト配列になってしまうようなケースを防ぎます。

Base64 文字列には実質的に「未使用領域」があり、通常は無視されますが、その結果として同じ Guid を生成してしまうことがあります。v2.0.0 ではそのような場合に `FormatException` をスローするようになりました。

`ShortGuid` 自身は常に有効な文字列のみを生成しますが、外部から無効な文字列が渡された場合、複数の URL セーフ Base64 文字列が同じ Guid を指してしまう意図しない衝突が発生する可能性があります。この不確実性を避けるため、入力文字列との 1 対 1 の対応が取れているかを確認するためのラウンドトリップチェックが行われます。

旧来の挙動 (オプトインの厳密パース) が必要な場合は、バージョン 1.1.0 を利用してください。

---

## 概要

このライブラリは、次のような標準的な GUID を:

`c9a646d3-9c61-4cb7-bfcd-ee2522c8f633`

次のような、より短い文字列に変換します:

`00amyWGct0y_ze4lIsj2Mw`

エンコードとデコードそのものは、最終的には 1 行ずつのシンプルな関数に集約できます。`ShortGuid` が提供するのは、新しい型として次の 2 点を明示的に扱えるようにすることです。

1. 文字列と Guid の間のパースや型変換を容易にする
2. コード上で、その型のデータを期待しているという意図を表現できるようにする

---

## ShortGuid の利用方法

ShortGuid は通常の Guid や他の ShortGuid 文字列と互換性があります。まずは簡単な例です。

```csharp
Guid guid = Guid.NewGuid();
ShortGuid sguid1 = guid; // guid から shortguid への暗黙的キャスト
Console.WriteLine(sguid1);
Console.WriteLine(sguid1.Guid);
```

上記は新しい Guid を生成し、その Guid から ShortGuid を作成し、コンソールに 2 つの等価な値を表示します。結果は次のようになります。

`FEx1sZbSD0ugmgMAF_RGHw`  
`b1754c14-d296-4b0f-a09a-030017f4461f`

文字列から ShortGuid への暗黙的キャストも可能です。

```csharp
string code = "Xy0MVKupFES9NpmZ9TiHcw";
ShortGuid sguid2 = code; // 文字列から shortguid への暗黙的キャスト
Console.WriteLine(sguid2);
Console.WriteLine(sguid2.Guid);
```

出力は次のようになります。

`Xy0MVKupFES9NpmZ9TiHcw`  
`540c2d5f-a9ab-4414-bd36-9999f5388773`

### 他の型との柔軟な連携

ShortGuid はさまざまな型と簡単に組み合わせて使えるように設計されているため、コードをシンプルにできます。以下の例をご覧ください。

```csharp
// Guid.NewGuid() と同様に、新しい ShortGuid を生成
ShortGuid sguid = ShortGuid.NewGuid();

// 文字列 "myString" を ShortGuid として扱う
string myString = "Xy0MVKupFES9NpmZ9TiHcw";

// 次の 3 行は等価です
ShortGuid sguid = new ShortGuid(myString); // 従来のコンストラクタ
ShortGuid sguid = (ShortGuid)myString;     // 明示的キャスト
ShortGuid sguid = myString;                // 暗黙的キャスト

// 同様に、Guid "myGuid" を ShortGuid として扱う
Guid myGuid = new Guid("540c2d5f-a9ab-4414-bd36-9999f5388773");

// 次の 3 行も等価です
ShortGuid sguid = new ShortGuid(myGuid); // 従来のコンストラクタ
ShortGuid sguid = (ShortGuid)myGuid;     // 明示的キャスト
ShortGuid sguid = myGuid;                // 暗黙的キャスト
```

ShortGuid を作成した後に主に利用するメンバーは、元の Guid 値、新しい短い文字列 (エンコードされた Guid 文字列)、および ToString() メソッドです。ToString() は短いエンコード済み Guid 文字列を返します。

```csharp
sguid.Guid;       // Guid 値を取得
sguid.Value;      // エンコードされた Guid を文字列として取得
sguid.ToString(); // sguid.Value と同じ値を返す
```

### Guid や文字列との簡単な比較

Guid、string、ShortGuid の 3 種類の型の間で等値比較を行うこともできます。以下の例を参照してください。

```csharp
Guid myGuid = new Guid("540c2d5f-a9ab-4414-bd36-9999f5388773");
ShortGuid sguid = (ShortGuid)"Xy0MVKupFES9NpmZ9TiHcw";

if (sguid == myGuid) {
  // Guid と ShortGuid が等しい場合の処理
}

if (sguid == "Xy0MVKupFES9NpmZ9TiHcw") {
  // 文字列と ShortGuid が等しい場合の処理
}
```

---

## SQL での利用

SQL Server (SQL 2005 以降) で ShortGuid を直接扱いたい場合は、プロジェクトのソースファイルに含まれている `ShortGuid.sql` を利用できます。

```SQL
SET @sguid = dbo.EncodeShortGuid(@myUniqueIdentifier)
-- 出力例 'Xy0MVKupFES9NpmZ9TiHcw'

SET @guid = dbo.DecodeShortGuid(@myShortGuidValue)
-- 出力例 540c2d5f-a9ab-4414-bd36-9999f5388773