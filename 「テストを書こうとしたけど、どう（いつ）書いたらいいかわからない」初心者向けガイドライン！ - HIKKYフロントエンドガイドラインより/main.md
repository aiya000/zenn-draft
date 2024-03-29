本稿では、自動テストについての「ちょうど一歩くらい」進んだ内容について、述べていきます。

「**テストを実施したいけど、どう書けばいいかわからない**」という全人類は、これを読んでください。
損はさせませんよ！

これを読めば、「一歩目の知識」「**初めてのテスト作成**」「**テストの初歩概念**」を得ることができ、「**次になにをすればよいか**」がわかるようになります！！

![](https://spice-factory.co.jp/wp-content/uploads/2022/01/e2d79507cf929987aa4068446da76d51.png)

対象読者:
- 実際にテストを実装してみようと思ったけど、**とっかかりがつかめなかった**
- **テストの概要はわかった**（[単体テストと結合テストガイドライン](https://zenn.dev/aiya000/articles/978fa504b1da3f)を読んだ）
- テストのTipsに興味がある

なお本稿は株式会社HIKKYフロントエンドチーム向けのガイドラインであるものの、内容については筆者に一任されており、**株式会社HIKKY及びその意向等とは無関係です。**

# テストってどういうフローで開発していくの？ どのタイミングでテストコードを書くの？

おおまかには以下の、2通りのテスト開発フローがあります。
この2通りをプロジェクトごとに規定するか、また規定がなければ、そのときの作業者の裁量で採用するとよいでしょう。

- テスト駆動開発（TDD）
    - **テストを書いてから**、実装を書く
- 実装駆動開発
    - **実装を書いてから**、テストを書く

これについて、少し詳しく解説していきます。[^both-tdd-and-implement-driven]

## テスト駆動開発（TDD）

TDDとは、おおまかに言って「**最初にテスト**を書いて、そのあとに実装を書く」フローのことです。

実際の流れは下記の記事でとてもよく述べられているので、下記の記事の「1. テスト駆動開発(TDD)」を参照してください。
（BDDについては読む必要はありません。ただし興味深いので、読んでも問題ありません。）

- [アジャイル開発における「TDD」と「BDD」 - SHIFT ASIA -ソフトウェア品質保証のプロフェッショナル-](https://shiftasia.com/ja/column/%E3%82%A2%E3%82%B8%E3%83%A3%E3%82%A4%E3%83%AB%E9%96%8B%E7%99%BA%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8Btdd%E3%81%A8bdd/)

TDDの利点は以下です。
- 一度テストを書いてしまえば、テストが通る状態を目指すように、実装を行える
- 実装への先入観なしに、テストを書くことができる

TDDの欠点は以下が挙げられます。
- 実装なしにテストを書くのが重い。実装を元にテストを書いた方が楽
- そもそもテストを書くコストが高い。テストを書く暇がない

## 実装駆動開発

実装駆動開発とは、おおまかに言って「**最初に実装**を書いて、そのあとにテストを書く」フローのことです。

あまり一般的に「実装駆動開発」という用語が用いられないので、ここではこの用語を下記のフローのこと、と定義します。

1. 実装を書く
1. テストを書く
1. テストを実行する
    - テストが通らなかったら、実装を修正する
1. 必要があれば、実装のリファクタリングをする
1. テストを実行する
    - テストが通ることを確認する

TDDと比べて、実装とテストが真逆の順番になっていることがわかります。
実装駆動開発は、テストよりも実装を先に行います。

実装駆動開発の利点は以下です。
- 最悪の場合、テストを書かないでも、納品することができる
- 実装を見ながら、それが満たすであろうテストを書ける

実装駆動開発の欠点は以下が挙げられます。
- 実装をするときに、それが満たすべき性質（を表すテスト）が手元にない
- 実装への先入観があるので、実装と独立したテストが書きにくい。実装が当然満たすであろうテストしか書けなくなる [^by-PBT]

## おまけ: リファクタリング時テスト化について

おまけ程度ですが、上記2つに当てはまらない、テスト化のタイミングもあります。
その一例として「**リファクタリング時**テスト化」（リファクタリングするときに、テスト化する）を説明します。

業務をしていると「既存のリファクタリングをしたいけど、現在の挙動を変更してしまわないか心配」という場合が多くあります。
この場合はテスト化を介することで、挙動を担保することができます。

その際のフローは以下になります。

1. 既存のコードに対する、保障したいふるまいをテスト化する
1. テストを実行する
    - テストが通ることを確認する
1. リファクタリングする
1. テストを実行する
    - テストが通ることを確認する

# テストってどういう観点でどういう内容を書くべきなの？
## 単体テスト

単体テストでは、下記の点について着目すべきです。

- 「下記の項目」に対して、所望の性質を満たすこと [^normal-testing]
    - **ある特定の値**に対して
    - **ある境界**に対して
    - **全ての値**に対して

また「**不正な値**」に対して、所望の性質を**満たさない**ことを確認するのも、重要です。[^abnormal-testing]

具体的なコードを見ていきましょう。

- - - - -

#### UIコンポーネントへの単体テスト

ここで単体テストを関数に対して書いていますが、UIコンポーネントにも同等な考えを使えます。
例えばVueであれば、引数はpropsとして、戻り値はViewの結果・状態として考えられます。

また弊社Webフロントエンドチームでは、UIコンポーネントにおいて、Atomicデザインを採用しています。

![](https://spice-factory.co.jp/wp-content/uploads/2022/01/e2d79507cf929987aa4068446da76d51.png)

- [アトミックデザインとは？メリットや気を付けるポイントを徹底解説！｜スパイスファクトリー株式会社](https://spice-factory.co.jp/web/about-atmicdesign/)

UIコンポーネントへの単体テストはこのうち、Atoms・Moleculesのコンポーネントのことになるでしょう。

実際は[Vue Test Utils](https://v1.test-utils.vuejs.org/ja/)を用いるのがよいかもしれません。

- - - - -

### ある特定の値に対して、性質を満たす

これはテストにおいて、最も単純な考え方です。

例えば次の関数`div`が、単に`x=10, y=2`の場合に`5`を確認できれば十分の場合、この観点が使えます。

```typescript
function div(x: number, y: number): number {
  if (y === 0) {
    throw new Error('Div By Zero')
  }
  return x / y
}
```

```typescript
test('div(10, 2) should be 5', () => {
  expect(div(10, 2)).toBe(5)
})
```

どの観点でテストを書くか迷った場合は、これに立ち戻るのがよいでしょう。

ただし、この観点でテストを書くと、**テストケースは不足しがち**です。
もし不足しがちだと感じた場合は、他の観点を試みてください。

**テストケースが単純でよい場合**のみ
（うまくいくケースが1つ以上示せればいい場合のみ）、これを採用してください。

### ある境界に対して、性質を満たす

これはテストにおいて、実用的な考え方です。
テスト対象が「**境界**」を持つ場合に、この観点が使えます。

では、境界とはなんでしょうか。
境界とは、テスト対象がある値（**境界値**）の前後によって挙動が変わる場合の、その境界値の前後のことを指します。

例えば関数`f`が

- 41以下の場合
- 42以上の場合

で処理を分岐する場合、`41, 42`が境界です。
そして`41`と`42`が境界値です。

その場合に

- 41以下の場合
- 42以上の場合

に対してテストケースを作成するのが、本観点になります。

- - - - -

余談ですが、この文脈では
「x**より大きい**」
「x**以上**」、そして
「y**より小さい**」
「y**以下**」
の違いが重要なので、混同しないで考えるようにしましょう。

- - - - -

具体例を見てみましょう。

```typescript
function range(start: number, end: number): Array<number> {
  if (start < 0) {
    const result: Array<number> = []
    for (let i = start; i >= end; i--) {
      result.push(i)
    }
    return result
  }

  // start >= 0 の場合
  const result: Array<number> = []
  for (let i = start; i <= end; i++) {
    result.push(i)
  }
  return result
}
```

[^range]

`if (start < 0)`でわかる通り、この関数`range`の境界は（`start`に対して）`-1, 0`です。

まとめると、境界は以下にあることになります。

- -1以下の場合
- 0以上の場合

境界値は`-1`, `0`です。

確認してみましょう。

```typescript
console.log(range(0, 10))
// [ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ]

console.log(range(-1, -10))
// [ -1, -2, -3, -4,  -5, -6, -7, -8, -9, -10 ]
```

startが**0以上**であるか、**-1以下**であるかで動作が変わっていることが、確かにわかりますね。

この場合のテストケースも単純で、次のように記述します。

```typescript
test('makes lists by positive steps', () => {
  expect(range(0, 10)).toEqual([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
})

test('makes lists by positive steps', () => {
  expect(range(-1, -10)).toEqual([-1, -2, -3, -4, -5, -6, -7, -8, -9, -10])
})
```

以上が「ある境界に対して、性質を満たす」の観点でした。

この観点は、非常に応用が効きます。
テストケースを考えた際に

- 境界が分かれている
- 後述の「全ての値に対して、性質を満たす」を使うほどでもない

といった場合は、境界値の観点を使うとよいでしょう。

ちなみにこの観点を使ったテストを、特別に
「**境界値分析**」
または
「**境界値テスト**」
と言います。

#### 応用: 同値分割

境界値でテストケースを作成する際に、境界値のみのテストだとケースが不足していると感じることがあります。
例えば上述の`'makes lists by positive steps'`や`'makes lists by positive steps'`のテストでは、ケースが1つずつしかありません。

その場合は「**同値分割**」を使うとよいです。

同値分割では、境界値を区切りにした区間の値群（同値の値）をテストケースに用います。

`'makes lists by positive steps'`と`'makes lists by positive steps'`のテストで同値分割を使ってみると、次のようになります。

```typescript
test('makes lists by positive steps', () => {
  // start >= 0 のケース
  expect(range(0, 10)).toEqual([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
  expect(range(1, 10)).toEqual([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
  expect(range(5, 10)).toEqual([5, 6, 7, 8, 9, 10])
})

test('makes lists by positive steps', () => {
  // start < 0 のケース
  expect(range(-1, -10)).toEqual([-1, -2, -3, -4, -5, -6, -7, -8, -9, -10])
  expect(range(-2, -10)).toEqual([-2, -3, -4, -5, -6, -7, -8, -9, -10])
  expect(range(-5, -10)).toEqual([-5, -6, -7, -8, -9, -10])
})
```

「`start >= 0`のケース」や「`start < 0` のケース」へのテストなのが、わかりやすくなりましたね！
これが同値分割です。

ただし**テストケースは多ければいいというものではない**ので、可能であれば増やしすぎず、クリティカルなテストケースだけを記述するようにしましょう。

- 参考
    - [単体テストのテスト項目の観点  |  ソフトウェア雑記](https://softwarenote.info/p738/)

### 全ての値に対して、性質を満たす

これはテストにおいて、実用的かつ発展的な考え方です。
テストケースが少なからず必要な**全ての場合**に、この観点が使えます。

この観点では**Property Based Testing**（PBT）を用います。
雑に言えば、PBTは「**入力をランダムにしたテスト**」です。

今までの観点とPBTが大きく違うのは「テストケースを自動生成する」こと、つまりテスト設計の大きな部分をコンピューターに任せられることです。
今まではテストの対象をプログラマーがよく見て、テストケースの値をプログラマーが決定しなければいけませんでしたから。

ここでは例のために、PBTライブラリとして、[fast-check](https://github.com/dubzzz/fast-check)を使います。
簡単な例を見てみましょう。

```typescript
// 配列を逆向きにして返す関数
export function reversed<T>(array: T[]): T[] {
  return [...array].reverse()
}
```

下記は、実際にfast-checkを使ったテストです。

```typescript
// reversed関数を2回適用すると、元の配列に戻る
test('forall value, reversed ○ reversed = identity', () => {
  fc.assert(
    fc.property(fc.array(fc.anything()), (xs) => {
      expect(reversed(reversed(xs))).toEqual(xs)
    })
  )
})
```

このテストは
「任意の型の、任意の配列`xs`について」
「`reverse(reverse(xs)) = xs`が成り立つ」
を表すものです。[^pseudo-forall]
（そりゃ配列を2回逆向きにすれば、正向きになりますよね！）

これまでのテスト観点では、こちらで用意した値についてのテストしかできなかったので、
このような性質（関数自体の性質）は述べることができませんでした。

PBTはそれを可能にします。
今見た通りね！

訓練のために、これまでの観点で書いてきたテストケースを、PBTで書き直して、PBTの直腸をつかんでみましょう。

前述の`div`関数のテストでは、`div(10, 2) = 5`であることを確認しました。

```typescript
test('div(10, 2) should be 5', () => {
  expect(div(10, 2)).toBe(5)
})
```

これをPBTにしてみましょう。
「任意の整数`x`に対して、`div(x, 2) = x / 2`」をテストすることにします。

```typescript
test('forall x, div(x, 2) = x / 2', () => {
  fc.assert(
    fc.property(fc.integer(), (x) => {
      expect(div(x, 2)).toBe(x / 2)
    })
  )
})
```

次は`range`関数のテストをPBTにしてみましょう。

```typescript
test('makes lists by positive steps', () => {
  expect(range(0, 10)).toEqual([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
})
```

今回のケースでは（`range`をテスト内で再実装しない限り）
PBTでは具体的な比較はできません。
上記の`range(0, 10)`へのテストの`toEqual()`の引数`[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]`のように、具体的な既知の値が定まらないからです。

ですので、性質（Property）の検査に置き換えます。
具体的には、下記の`expect()`を使用している3行です。

```typescript
test('makes lists by positive steps', () => {
  fc.assert(
    fc.property(
      fc.integer({ min: 0, max: 100 }),
      fc.integer({ min: 0, max: 100 }),
      (start, end) => {
        fc.pre(start <= end)

        const result = range(start, end)
        expect(result.at(0)).toBe(start)
        expect(result.at(-1)).toBe(end)
        expect(result.length).toBe(end + 1 - start)
      }
    )
  )
})
```

※自然数を表すために`fc.integer({ min: 0, max: 100 })`を使っています。
　fast-checkには`fc.nat()`という自然数を表す関数もありますが、今回の場合`range(x, 非常に大きい自然数)`などとすると、大きな配列を作ろうとしすぎてJavaScriptランタイム（JestやVitest）が壊れるので（実際に壊れたので！！）、`100`に制限しています。

PBTが何であって、何でないのかがつかめてきたでしょうか。

というところで、次に進もうと思います。

### 不正な値に対して、性質を満たさないこと

前述の3つのテスト観点は「どのように所望の性質を満たすか」を確認するものでした。

それらも大切なテストでしたが、それとは別に「どのように所望の性質を**満たさないか**」をテスト化することも、同様に大切です。

この観点は、前述の3つの観点に横断します。
（つまり、前述の3つを用いて、本テスト観点を確かめることができます。）

例えば、下記の関数`div`で、第二引数`0`の場合に例外をthrowすること
（この場合に、`div`関数が「割り算をする」という役割を満たさないこと）
を確認するとき、この観点が使えます。

```typescript
function div(x: number, y: number): number {
  if (y === 0) {
    throw new Error('Div By Zero')
  }
  return x / y
}
```

```typescript
test('throws if div by zero', () => {
  const randomValue = 42 // 適当な値
  expect(() => div(randomValue, 0)).toThrow()
})
```

この観点は、前述の3つの観点に横断するのでした。
つまりここでPBTや、境界値テストを使うこともできます。

```typescript
test('throws if div by zero', () => {
  fc.assert(
    // 任意の整数に対して、0で割ると、例外が送出される
    fc.property(fc.integer(), (randomValue) => {
      expect(() => div(randomValue, 0)).toThrow()
    })
  )
})
```

必要であれば、同様に境界値テストを用いることもできます。

## スナップショットテスト

ここからはスナップショットテストを行うときの観点について、説明します。

スナップショットテストは難しいことはなく、「**UIが予期せず変更されていないこと**」を確認するために行われます。

ですので、導入する機会は

- 理由はないけど、とりあえずスナップショットテストはしておく
- UIが完成したので、スナップショットテストをしておく

あたりかなと思います。
**UIを勝手に変更されたくない場合に**、導入しておきましょう。

## ビジュアルリグレッションテスト

ビジュアルリグレッションテスト（VRT）についてです。

これはスナップショットテストとかなり似た用途で用いられますが、それとはどちらかというと逆向きで
「**UIが期待した通りに変更されていること**」
を確認するために行われやすいと思います。

ただしこちらは、GitHubなどと連携させて、PRが出されたときにそのPRが変更している箇所を
**視覚的に表示する**
という運用がしやすいです。

やはり導入する機会はスナップショットテストと同じでしょう。

## 結合テストとE2Eテスト
### 結合テストへの姿勢

まずはじめに、Testing Trophy[^testing-trophy]にて、E2Eテストでない**結合テストは**各テストレイヤーの中で「**最も比重を高くするべきテスト**」だと主張されています。
そして**E2Eテスト**は「**最も比重を低くすべきテスト**」です。

![](https://res.cloudinary.com/kentcdodds-com/image/upload/f_auto,q_auto,dpr_2.0,w_1600/v1622744540/kentcdodds.com/blog/the-testing-trophy-and-testing-classifications/trophy_wx9aen.png)

- [The Testing Trophy and Testing Classifications](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications)

（この章にて、以降「結合テスト」は「（E2Eテストでない）結合テスト」と読み替えてください。）

それを踏まえ「結合テスト・E2Eテストはどのような観点で実施すべきか」を考えていきます。
まずは具体的なテストコードを見てみましょう。

結合テストライブラリとして、ここでは[Testing Library](https://testing-library.com/docs/dom-testing-library/intro/)を用います。

[UIコンポーネントへの単体テスト](#UIコンポーネントへの単体テスト)で述べた、Atomicデザインにおける単体テストの区分と対比して、Organisms・Templatesへのテストが結合テストになるでしょう。
下記では`Component.vue`がOrganismsとします。

```typescript
import { render, fireEvent, screen } from '@testing-library/vue'
import Component from './Component.vue'

test('increments value on click', async () => {
  render(Component)

  screen.getByText('Times clicked: 0')

  const button = screen.getByText('increment')

  await fireEvent.click(button)
  await fireEvent.click(button)

  screen.getByText('Times clicked: 2')
})
```

[Examples | Testing Library](https://testing-library.com/docs/vue-testing-library/examples)より引用。

UIコンポーネントの結合テストの話ですが、
テストトロフィーの作者でもあるTesting Libraryの作者は、Testing Libraryにて

> テストがソフトウェアの使用方法に似ているほど、より信頼できるようになります

と述べられています。

- [Introduction | Testing Library](https://testing-library.com/docs/dom-testing-library/intro/)

上述で例示したコードのように、「実装に着目する」よりも「ふるまいに着目する」方がよいでしょう。
（実装に着目したい場合、単体テストが最適な場合があるかもしれません。）

### 結合テストの観点

観点についてですが、結合テストは次の観点で作成されるべきです。

- 気になったケースがあったが、**単体テストではカバーできなかったので**、結合テスト化する
- **設計が適切なことを確認するため**に、設計を結合テスト化する[^user-acceptance-test]
- バグが見つかった際に、**それを再現させる操作**を、結合テスト化する

またこの際に、**結合テストで再現させられなかった場合のみ、E2Eテストにします**。
前述の通り、できるだけE2Eの比重は軽くすべきなので、この扱いは納得ができるかと思います。

# まとめ

おつかれさまでした！
ここでは多くのテストを「どのように（どんなときに）書くか」という観点について、説明してきました。

テスト作成のフローは2つありました。

- テスト駆動開発（TDD）
- 実装駆動開発

単体テストでは、次のような観点がありました。

- 「下記の項目」に対して、所望の性質を満たすこと（正常系テスト）
    - ある特定の値に対して
    - ある境界に対して（境界値テスト・境界値分析・同値分割）
    - 全ての値に対して（Property Based Testing）
- 「不正な値」に対して、所望の性質を満たさないこと（異常系テスト）

UIの変更に対するテストは、2つあります。

- スナップショットテスト
- ビジュアルリグレッションテスト

結合テストは「単体テストでは再現できないテスト」を書くべきで、
E2Eテストは「E2Eテストでない結合テストでは再現できないテスト」を書くべきでした。

また結合テストは、各テストレイヤーの中でも、最も比重を多くすべきテストでした。

これからあなたがテストを書くときに、どうすべきか迷わないことを祈っております。
読んでくださり、ありがとうございました！

- - - - -
- - - - -

# 付録
## rangeのより正確な定義と、ふるまいについて

[^range] より。

```typescript
function range(start: number, end: number): Array<number> {
  if (end === start) {
    return [end]
  }

  if (end < start) {
    const result: Array<number> = []
    for (let i = start; i >= end; i--) {
      result.push(i)
    }
    return result
  }

  // end > start の場合
  const result: Array<number> = []
  for (let i = start; i <= end; i++) {
    result.push(i)
  }
  return result
}
```

```typescript
const end = 5

test('makes singleton lists', () => {
  expect(range(5, end)).toEqual([5])
})

test('makes lists by positive steps', () => {
  expect(range(0, end)).toEqual([0, 1, 2, 3, 4, 5])
})

test('makes lists by positive steps', () => {
  expect(range(10, end)).toEqual([10, 9, 8, 7, 6, 5])
})
```

- - - - -
- - - - -

[^both-tdd-and-implement-driven]: ちなみにですが、実際は双方のいいとこ取りをした（上記2つの両方を採用した）「テストを書いてから、実装を書いて、そのあとにまたテストを書く」という方法も、現実的にはあります。
[^range]: 目ざとい方は気づかれたかもしれませんが、通常の`range`関数は`start < 0`で分岐するよりも、`end < start`で分岐した方が有用です。例えば前者であれば`range(-5, 0)`は`[]`になりますが、後者であれば`[-5, -4, -3, -2, -1, 0]`になります。多くの方は後者のふるまいを期待するかもしれません。今回は簡単のために前者の定義を採用し、後者の定義は[こちら（付録）](#付録)に載せるにとどめておきます。
[^by-PBT]: Property Based Testingでテストケースをランダムに作成することにより、ある程度は未知に対するテストを書くことができます。Property Based Testingについては、[「全ての値に対して、性質を満たす」](#forall-values-satisfies)および[単体テストと結合テストガイドライン](https://zenn.dev/aiya000/articles/978fa504b1da3f)の「さいごに」を参照してください。」
[^normal-testing]: これらは「これらの値を使えば、正常な動作をする」ということを確認することから「**正常系テスト**」と呼ばれます。
[^abnormal-testing]: これらは正常系テストと対比し、「これらの値を使えば、正常でない動作をする（例えば例外を送出するなど。）」ということから「**異常系テスト**」と呼ばれます。
[^pseudo-forall]: ここでわかりやすさのために「**任意の**」と言っていますが、PBTを使っていても、実際に全ての値でテストができる**わけではありません**。全ての値をテストすると、処理に大きな時間がかかってしまうからです。ここでは「任意の値」を「多くのランダムな値」に読み替えてください。
[^testing-trophy]: 「各テストレイヤーのあるべき比率」を表した、トロフィー状の概念。
[^user-acceptance-test]: この観点は「受け入れテスト」にて顕著です。
