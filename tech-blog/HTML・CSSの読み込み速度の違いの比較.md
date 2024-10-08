# HTML・CSSの読み込み速度の違いの比較

## はじめに

最近フロントエンドを開発することが多くなり、HTML・CSSの読み込み速度について気になりました。
プロダクトとしても、読み込み速度をあげることでユーザー体験を向上させることを目指しているため、調査してみました。
計測は手軽に行える、今回はlighthouseを使用しました。
また最近はモバイルファーストインデックスが重要視されているため、デバイスの設定はモバイルで行いました。

## HTMLの読み込み速度調査
### めっちゃネスト深くするとどうなるか
まずは単純に1つの要素をdivで1000ネストしてみました。

検証してみたら、エラーが出ました。

![スクリーンショット 2024-09-29 21.06.37.png](../images/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202024-09-29%2021.06.37.png)

ネストしすぎるとlighthouseが正しく計測できないようです。

どのくらいだと計測できなくなるのか調べてみましたが大体150ネストくらいで計測できなくなりました。
ネストしすぎるとよくないっぽいですね。当たり前か。

### ネストが浅いものと深いもので計測結果を比較

それではネストが浅いものと深いもので計測結果を比較してみます。
1ネスト増えるごとに、計測結果がどのように変わるのか調査してみます。
同じ要素数（100）でネストが浅いものと深いものを作成し、計測してみました。

以下が計測結果です。Total Blocking TimeとCumulative Layout Shiftはどの場合も0sで結果が変わらなかったため省略します。

| ネスト数 | First Contentful Pain | Largest Contentful Paint | Speed Index |
|-----|-----------------------|---------------------------|------------|
| 1   | 0.6s                  | 0.6s                     | 0.6s        |
| 2   | 0.6s                  | 0.6s                     | 0.6s        |
| 4   | 0.6s                  | 0.6s                     | 0.6s        |
| 8   | 0.6s                  | 0.6s                     | 0.6s        |
| 16  | 0.8s                  | 0.8s                     | 0.8s        |
| 32  | 0.8s                  | 0.8s                     | 0.8s        |
| 64  | 1.1s                  | 1.1s                     | 1.1s        |
| 128 | 1.8s                  | 1.8s                     | 1.8s        |

大体16ネストくらいまでは計測結果に差が出ませんでしたが、それ以上になると計測結果に差が出るようです。 
1ネストとの差を単純計算で、
- 0.2s / 15ネスト / 100要素 = 0.0001333s
- 0.5s / 63ネスト / 100要素 = 0.0000794s
- 1.2s / 127ネスト / 100要素 = 0.0000945s
- (0.0001333s + 0.0000794s + 0.0000945s) / 3 = 0.0001024s
- なので、1ネスト増えるごとに0.0001024sの誤差が生じる
誤差と言えば誤差ですが、for文で同じようなUIを大量に生成する場合はなるべくネストを浅くしたほうが良さそうです。

## CSSの読み込み速度調査
先ほどの100要素全てにdiv-nのクラス名を付与して、CSSでスタイルを適用してみました。
CSSを適応する要素数を増やしてみてどうなるのかを検証します。

### 1つのスタイルを適応した場合
```css
.div-n {
  display: grid;
}
```
| CSS適応要素数 | First Contentful Pain | Largest Contentful Paint | Speed Index |
|--------|-----------------------|--------------------------|-------------|
| 1      | 0.6s                  | 0.6s                     | 0.6s        |
| 10     | 0.6s                  | 0.6s                     | 0.6s        |
| 20    | 0.6s                  | 0.6s                     | 0.6s        |
| 50    | 0.7s                  | 0.7s                     | 0.7s        |
| 100   | 0.6s                  | 0.6s                     | 0.6s        |

ほとんど変化なし

### 複数のスタイルを適応した場合
ここからはスタイルのフィールドを増やしていきます。
適応数は変わらなかったので全てのクラスに同じフィールド数を適応しています。
上から順に増やしていきます。
```css
.div-n {
  display: grid;
  justify-items: center;
  align-items: center;
  width: 100px; 
  height: 100px;
  background-color: #333;
  font-size: 16px;
  font-weight: bold;
  color: #333;
  border-width: 1px;
  border-style: solid;
  border-color: red;
  border-radius: 10px;
  padding: 10px;
  margin: 10px;
  text-align: center;
  
}
```

| 適応フィールド数 | First Contentful Pain | Largest Contentful Paint | Speed Index |
|----------|-----------------------|--------------------------|-------------|
| 1        | 0.6s                  | 0.6s                     | 0.6s        |
| 2        | 0.7s                  | 0.7s                     | 0.7s        |
| 3        | 0.9s                  | 0.9s                     | 0.9s        |
| 4        | 0.9s                  | 0.9s                     | 0.9s        |
| 5        | 0.8s                  | 0.8s                     | 0.8s        |
| 6        | 0.8s                  | 0.8s                     | 0.8s        |
| 7        | 0.8s                  | 0.8s                     | 0.8s        |
| 8        | 0.8s                  | 0.8s                     | 0.8s        |
| 9        | 0.8s                  | 0.8s                     | 0.8s        |
| 10       | 0.8s                  | 0.8s                     | 0.8s        |
| 11       | 0.8s                  | 0.8s                     | 0.8s        |
| 12       | 0.8s                  | 0.8s                     | 0.8s        |
| 13       | 0.8s                  | 0.8s                     | 0.8s        |
| 14       | 0.8s                  | 0.8s                     | 0.8s        |
| 15       | 0.8s                  | 0.8s                     | 0.8s        |

適応フィールド数が増えても、計測結果に変化は見られませんでした。

### HTMLのネストとCSSのネストを増やしてみる
適応フィールド数は15のまま、HTMLのネストを増やしそれに応じてCSSのネストも増やしてみます。

| ネスト数 | First Contentful Pain | Largest Contentful Paint | Speed Index | Total Blocking Time |
|------|-----------------------|--------------------------|-------------|--------------------|
| 1    | 0.8s                  | 0.8s                     | 0.8s        | 190ms              |
| 2    | 0.9s                  | 0.9s                     | 0.9s        | 190ms              |
| 4    | 0.9s                  | 0.9s                     | 0.9s        | 190ms              |
| 8    | 0.9s                  | 0.9s                     | 0.9s        | 190ms              |
| 16   | 1.0s                  | 1.0s                     | 1.0s        | 190ms              |
| 32   | 1.2s                  | 1.2s                     | 1.2s        | 190ms              |
| 64   | 1.4s                  | 1.5s                     | 1.4s        | 300ms              |
| 120  | 1.8s                  | 1.9s                     | 1.8s        | 1000ms             |
| 128  | エラーで計測不可              | エラーで計測不可                 | エラーで計測不可    | エラーで計測不可           |