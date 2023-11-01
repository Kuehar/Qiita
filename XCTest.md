# 概要
この記事を読むことで、単体テストにおける基本的な考え方を具体例を交えて学習し、その後、Swiftプロジェクトにおける一般的な自動テストのツールである`XCTest`を用いた自動テストを実施できるようになります。

# 環境
XCode15.0.1以上をインストール済みであること
(それ以前のバージョンでもXCTestの実施は可能ですが、Code Coverageの部分が異なります。)

# 単体テストとは？
単体テストはソフトウェア開発における重要な工程の一つで、開発したソフトウェアの内、1つのコンポーネントが正しく機能することを確認するために実施します。

単体テストの実行回数を出来るだけ減らし、効率的に単体テストを実施するために、まずは単体テストの種類について見ていきます。

## 単体テストの種類

今回、以下のような四則演算を行う関数を用意しました。その上で、どのようなテストケースを実装するべきかをテストケースの種類を含めて考えていきましょう。

| 関数名  | Input| Output |
|:-----------|:------------|:------------|
| add| num1:Int,num2:Int|num1+num2の結果|
| subtract| num1:Int,num2:Int|num1-num2の結果|
| multiple| num1:Int,num2:Int|num1×num2の結果|
| divide| num1:Int,num2:Int|num1/num2の結果|


単体テストの種類には、一般的に以下の種類が挙げられます。

| 種類  | 目的| 具体例 |注意点 |
|:-----------|:------------|:------------|:------------|
| 機能テスト (Functional Tests)| 関数が正しい計算結果を返すことを確認する。|今回の関数の例では、`add`,`subtract`,`multiple`,`divide`関数に自然数を与えたときに正しく動作するかをテストする。|具体的な機能の仕様に基づいてテストケースを設計し、正しい結果を期待してテストする。| 
境界(限界)値テスト (Boundary Tests)| ある関数が正しい計算結果を返すことを確認する。|今回の関数の例では、`divide`関数で整数の最大/最小値に対する挙動をテストする。|境界値に対する振る舞いを特にテストし、境界値に対する期待結果を定義する。
| 異常系テスト (Error Cases Tests)|関数がエラーケースを正しく処理することを確認する。| 今回の関数の例では、`divide`関数のnum1に自然数、`num2`に0を与えた時に割り算にエラーがスローされることを確認する。|エラーが正しくスローされ、エラーメッセージが期待通りかどうかを確認する。


:::note info
上記の例では0除算を行った場合をエラーパターンとしていますが、境界(限界)値テストの項目として考えるケースもあります。
:::

`divide`関数を例にテストケースを考えてみます。
それぞれのケースで何の値を引数として渡し、どういった結果が出力されればテストとして正しいかを考えてみましょう。

1.機能テストの場合
- 割り切れるケース -> `10/2が5と等しいか`
- 割り切れないケース -> `10/3が3と等しいか`

2.限界値テストの場合
- `Int.max(9,223,372,036,854,775,807:実行環境が64bitの場合)`に対して割り算を行った場合 -> `Int.max / 2`が`4,611,686,018,427,387,903`と等しいか
- `Int.max-1`に対して割り算を行った場合 -> `(Int.max-1) / 2` が`4,611,686,018,427,387,904`と等しいか
- `Int.max+1`に対して割り算を行った場合 -> オーバーフローが発生するため実行不可
- `Int.min(-9223372036854775808)`に対して割り算を行った場合 -> `Int.min / 2`が`4,611,686,018,427,387,903`と等しいか
- `Int.min`-1に対して割り算を行った場合 -> オーバーフローが発生するため実行不可
- `Int.min+1`に対して割り算を行った場合 -> `Int.min / 2`が`-4611686018427387903`と等しいか

3.異常系テスト
- 0除算を行った場合 -> `5/0を行った場合、エラーが発生する`

と考えてみます。
上記の内容を踏まえて、テストコードを書いてみましょう。

# XCTestとは？
XCTestは、Appleが提供するソフトウェアテストフレームワークで、自動化されたテストスイート(テストケースの集まりのこと)を構築してアプリケーションの安定性と品質を向上させるのに役立ちます。

## 事前準備

ソースコードを用意しました。
以下のリンクからダウンロードし、XCodeで開いてください。
[XCTestSample](https://github.com/Kuehar/XCTestSample)

また、自動テストに移る前に画像の赤枠の`Code Coverage`の設定を有効にしておいてください。

![Frame 2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/412789/3da112ca-6802-8f9c-8f1b-bfc5cd7f2609.png)


## XCTestファイルの書き方

今回使用するファイルは
- `Calculator.swift`
- `CalculatorTests.swift`
です。

`Calculator.swift`には単体テストの種類の項で挙げた関数が以下のように実装されています。

```Swift:Calculator.swift
class Calculator {
    
    // 足し算
    func add(_ x: Int, _ y: Int) -> Int {
        return x + y
    }
    
    // 引き算
    func subtract(_ x: Int, _ y: Int) -> Int {
        return x - y
    }
    
    // 掛け算
    func multiple(_ x: Int, _ y: Int) -> Int {
        return x * y
    }
    
    // 割り算
    func divide(_ x: Int, _ y: Int) throws -> Int {
        if y == 0 {
            throw CalculatorError.divisionByZero
        }
        return x / y
    }
}

enum CalculatorError: Error {
    case divisionByZero
}

```

そして、具体例を挙げたテストケースは`CalculatorTests.swift`内に以下のようなテストコードの方式で記述してあります。

```Swift:CalculatorTests.swift
import XCTest
@testable import XCTestSample

class CalculatorTests:XCTestCase{
    
    var calculator:Calculator!
    
    // 各テストメソッドごとの前処理
    override func setUp(){
        super.setUp()
        self.calculator = Calculator()
    }
    
    // 各テストメソッドごとの後処理
    override func tearDown() {
        super.tearDown()
    }
    
  
    // ...コード省略...
  
    
      // 機能テスト - 割り切れるケース
    func testDivisionWithDivisibleNumbers() {
        do {
            XCTAssertEqual(try calculator.divide(10, 2), 5, "Division of 10 by 2 failed")
        } catch {
            XCTFail("Unexpected error: \(error)")
        }
    }
    
    // 機能テスト - 割り切れないケース
    func testDivisionWithNonDivisibleNumbers() {
        do {
            XCTAssertEqual(try calculator.divide(10, 3), 3, "Division of 10 by 3 failed")
        } catch {
            XCTFail("Unexpected error: \(error)")
        }
    }
    
    // 限界値テスト
    func testDivisionWithBoundaryValues() {
        do {
            XCTAssertEqual(try calculator.divide(Int.max, 2), 4_611_686_018_427_387_903, "Division of Int.max by 2 failed")
            XCTAssertEqual(try calculator.divide(Int.max - 1, 2), 4_611_686_018_427_387_903, "Division of (Int.max - 1) by 2 failed")
            XCTAssertEqual(try calculator.divide(Int.min, 2), -4_611_686_018_427_387_904, "Division of Int.min by 2 failed")
            XCTAssertEqual(try calculator.divide(Int.min + 1, 2), -4_611_686_018_427_387_903, "Division of (Int.min + 1) by 2 failed")
        } catch {
            XCTFail("Unexpected error: \(error)")
        }
    }
    
    // 異常系テスト - 0除算
    func testDivisionByZero() {
        XCTAssertThrowsError(try calculator.divide(5, 0)) { error in
            XCTAssertTrue(error is CalculatorError, "Division by zero did not throw CalculatorError")
        }
    }
    
}

```

:::note info
テストファイルを作る際の注意点
- XCTestをimportをする
- @testable importを行う
- クラスはXCTestCaseクラスを継承する
- テストメソッドの名前はtestで始める（testDivideのような形式）
- テストを実行したいファイルとテストケースを記述したファイルがそれぞれTaegetに含まれていることを確認する
:::

## Assertの種類
XCTestではコードサンプルにあるように、XCTAssert〇〇のような関数を使用して値を評価します。

主な関数は以下の通りです。
より詳しく知りたい場合には[公式ドキュメント](https://developer.apple.com/documentation/xctest)を参照してください。


| XCTest 関数名              | サンプルコードでの入力 (Input)                   | 出力 (Output)                 | 効果 (Effect)                                                  | 使い所 (Use Case)                                           |
|--------------------------|------------------------------|--------------------------|---------------------------------------------------------------|-----------------------------------------------------------|
| XCTAssertEqual         | calculator.add(3, 5)        | 8                      | XCTAssertEqual(calculator.add(3, 5), 8, "Addition failed")   | 数値の等しいか確認                                      |
| XCTAssertNotEqual      | calculator.subtract(10, 3) | 7                      | XCTAssertNotEqual(calculator.subtract(10, 3), 7, "Subtraction failed") | 数値の等しくないか確認                                |
| XCTAssertGreaterThan   | calculator.multiple(4, 7)  | 28                     | XCTAssertGreaterThan(calculator.multiple(4, 7), 20, "Multiplication failed") | 数値が大きいか確認                                     |
| XCTAssertLessThan      | calculator.divide(12, 4)    | 3                      | XCTAssertLessThan(calculator.divide(12, 4), 5, "Division failed") | 数値が小さいか確認                                     |
| XCTAssertTrue          | isPrime(11)                | true                   | XCTAssertTrue(isPrime(11), "11 should be prime")           | 条件が真であるか確認                                  |
| XCTAssertFalse         | isEven(7)                  | false                  | XCTAssertFalse(isEven(7), "7 should not be even")         | 条件が偽であるか確認                                  |
| XCTAssertNil           | calculateSquareRoot(-4)    | nil                    | XCTAssertNil(calculateSquareRoot(-4), "Square root should be nil") | 値が `nil` であるか確認                                |
| XCTAssertNotNil        | calculateSquareRoot(16)    | 4                      | XCTAssertNotNil(calculateSquareRoot(16), "Square root should not be nil") | 値が `nil` でないか確認                              |
| XCTAssertThrowsError   | try calculator.divide(5, 0)  | Error                  | XCTAssertThrowsError(try calculator.divide(5, 0), "Division by zero should throw an error") | エラーがスローされるか確認                         |
| XCTAssertNoThrow       | try calculator.divide(10, 2)  | No Error               | XCTAssertNoThrow(try calculator.divide(10, 2), "Division should not throw an error") | エラーがスローされないか確認                     |



## XCTestの実行方法

テストコードを記載後、XCode左側に存在するテストタブを選択し、実行したいテスト関数の名前にカーソルを合わせると以下のように再生ボタンになります。
再生すると、テストコード内で記載されたテストが実行されます。

![スクリーンショット 2023-10-22 19.22.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/412789/0cad291d-43a6-faa6-4ec3-419b212697b2.png)

実行後、テスト結果のタブに移動すると、実行結果やテストカバレッジなどのテストに関する情報を参照できるようになっています。

![スクリーンショット 2023-10-22 19.22.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/412789/50d72811-63ab-917f-61a1-7ae425c84de5.png)
