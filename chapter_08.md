# 第8章　特性の移動
## 関数の移動
```
class Account {
  get overdraftCharge() {...}
```
↓
```
class AccountType {
  get overdraftCharge() {...}
```

### 動機
- 優れたソフトウェア設計の核心はモジュール性であり、それを実現するには関連するソフトウェア要素をグループにまとめて、それらの結び付きを簡単に特定して把握できるようにする必要がある。自分が行なっていることの理解が深まるにつれて、どのようにグループ化するのが最善なのかが分かってくる。深まった理解を反映するには、要素を移動する必要がある。
- 全ての関数は何らかのコンテキストに置かれる。オブジェクト指向のプログラムでは、中核となるモジュールのコンテキストはクラスである。
- 関数を移動する最も直接的な理由は、その関数が存在するコンテキストの要素よりも他のコンテキストの要素を多く参照している場合である。
- また、呼び出し元が存在する場所や、次の拡張時に呼び出したい場所に関数を移動することがある。
- 関数の移動の判断を下すには、その関数の現在のコンテキストと移動先候補のコンテキストを調べる。

### 手順
- 移動対象の関数が現在のコンテキストで使用しているプログラム要素をすべて調べる。それらも移動すべきかどうかを検討する。
- 選択した関数がポリモーフィックなメソッドがどうかを確認する。
- 関数を移動先のコンテキストにコピーする。新居となる移動先に関数が適合するように調整する。
- 静的解析を実行する。
- 元のコンテキストから移動後の関数を参照できるようにする。
- 元の関数を委譲関数（移動後の関数を内部で呼び出す関数）に変更する。
- テストする。
- 元の関数に対して「関数のインライン化（p.121）」の適用を検討する。

- 注意：一般論として、入れ子関数には注意が必要。隠れたデータの相互関係が非常に簡単にできてしまい、追跡を難しくするため。（本節の最初の例では、入れ子となっていた関数をトップレベルに移動していた。）

## フィールドの移動
```
class Customer {
  get plan() {return this._plan;}
  get discountRate() {return this._discountRate;}
```
↓
```
class Customer {
  get plan() {return this._plan;}
  get discountRate() {return this.plan._discountRate;}
```

### 動機
- プログラムにおいてデータ構造は重要であり、データ構造が優れていれば振る舞いの記述は単純で明瞭なものになる。しかし、適切なデータ構造を定義するのは難しい。開発の過程でデータ構造が正しくないとわかったら、すぐに変更することが重要である。
- あるレコードを関数に渡すときに、必ず別レコードのフィールドも渡す必要がある場合、フィールドを移動したくなる。まとめて関数に渡すデータを同じレコードにまとめた方がデータの関係が明確になるためである。
- ここまで「レコード」について書いたことは、クラスやオブジェクトにも当てはまる。

### 手順
- 移動元のフィールドをカプセル化する。
- テストする。
- 移動先にフィールド（およびアクセサ）を作成する。
- 静的解析を実行する。
- 移動元のオブジェクトから移動先のオブジェクトを参照できるようにする。
    - 移動先への参照として、既存のフィールドやメソッドの使用を検討する。
- 移動先のフィールドを使うようにアクセサを調整する。
- テストする。
- 移動元のフィールドを削除する。
- テストする。

## ステートメントの関数内への移動
逆：ステートメントの呼び出し側への移動（p.225）
```
result.push('<p>title: ${person.photo.title}</p>');
result.concat(photoData(person.photo));

function photoData(aPhoto) {
  return [
    '<p>location: ${aPhoto.location}</p>',
    '<p>date: ${aPhoto.date.toDateString()}</p>',
  ];
}
```
↓
```
result.concat(photoData(person.photo));

function photoData(aPhoto) {
  return [
    '<p>title: ${aPhoto.title}</p>',
    '<p>location: ${aPhoto.location}</p>',
    '<p>date: ${aPhoto.date.toDateString()}</p>',
  ];
}
```

### 動機
- 重複の除去は、健全なコードを書くための最も優れた経験則の一つである。特定の関数を呼び出すたびに同じコードが実行されていたら、反復コードを関数自体に組み込むことを検討する。
- ステートメントを関数に移動するのは、それを呼び出し先の関数の一部とみなす方が理解しやすい場合である。呼び出し先の関数の一部としては意味をなさないもの、一緒に呼び出す必要がある場合は、そのステートメントと呼び出し先の関数を対象にして「関数の抽出」を行う。

### 手順
- 反復コードが移動先の関数呼び出しに隣接していない場合は、「ステートメントのスライド（p.231）」を行なって隣接させる。
- 移動元の関数の呼び出し元が移動先の関数だけだった場合は、移動元の関数からコードをカットし、移動先の関数にペーストして、テストを行い、残りの手順を無視する。
- 複数の呼び出し元がある場合は、いずれかの呼び出しに対して「関数の抽出（p.112）」を行い、移動先の関数呼び出しと移動したいステートメントを抽出する。名前は一時的なものにするが、簡単にgrepできる名前にする。
- 他のすべての呼び出しを変更して、新しい関数を使うようにする。変更のたびにテストする。
- 元の呼び出しのすべてで新しい関数を使うようになったら、「関数のインライン化（p.121）」を行なって元の関数を新しい関数に完全にインライン化し、元の関数を削除する。
- 「関数名の変更（p.130）」を行なって、新しい関数の名前を元の関数と同じ名前にする。

## ステートメントの呼び出し側への移動

### 動機
- コードベースの機能は変化するものだが、その際に抽象化の境界線がいつの間にか変わっていることがある。
- 境界線の変化に気付くのは、複数箇所で利用していた共通の振る舞いを、一部の呼び出しに対してだけ変更する必要が出てきた場合である。
- そうしたときには、変更の対象となる振る舞いを、関数から呼び出し側に移動する必要がある。
- 呼び出し側と呼び出される側の境界線を改めて定義し直す必要がある場合には、まず「関数のインライン化」を行い、次にステートメントをスライドした後で、より良い境界線となる新しい関数を抽出する方法が最善である。

## 関数呼び出しによるインラインコードの置き換え

### 動機
- 関数を利用することで複数の振る舞いをまとめることができる。
- 名前の付いた関数は、コードの仕組みではなく目的を説明できるため、その理解にも役立つ。また、重複の除去の観点でも有意義である。
- 既存の関数と同じ処理をしているインラインコードがある場合、通常はそのコードを関数呼び出しに置き換える。
- 置き換える際の一つの指針は関数名である。置き換えても意味をなさない場合、名前が不適切であるか、あるいは関数の目的がその状況にマッチしていないか、である。前者の場合は「関数宣言の変更（第6章）」を行い、後者の場合はその関数に置き換えるべきではない。
- ライブラリ関数群の呼び出しを使ってこのリファクタリングを行えた場合、関数本体を書く必要がないため、満足のいくものになるだろう。

## ステートメントのスライド

### 動機
- 互いに関係する処理が並んでいると、コードは理解しやすくなる。そのため、同じデータ構造にアクセスするコードが複数行ある場合、他のデータ構造にアクセスするコードと混在させずに、それらだけをまとめるべきである。そのための最も素朴な方法は、「ステートメントのスライド」を行って、該当するコードを集めることである。
- 「関数の抽出（第6章）」の準備段階で、関係するコードをまとめる作業を行うことがよくある。

### 手順、注意点
- コード断片をスライドする際に、判断すべきことが二つある。それは、どんなスライドをしたいのか、および実際にスライドできるのか、である。スライドによってコードが互いに干渉して、プログラムの外部使用を変えてしまわないかどうかを確認する必要がある。
- 副作用のあるコードをスライドする時、あるいは副作用のあるコードを超えてスライドする時は、最新の注意を払う必要がある。（具体的には、対象コードの関数が呼び出す関数を全て調べる必要がある。）

## ループの分離

### 動機
- 同じループの中で二つの異なる処理をしていると、ループを修正する際には、必ず両方の処理内容を理解しなければならなくなる。ループを分離することで、変更すべき処理だけを理解すればよくなる。
- 結果としてループを2回実行することになるため、多くのプログラマはこのリファクタリングを不快に思うだろう。しかし、著者が心掛けていることは、リファクタリングと際的かの分離である。最適化はコードをきれいにした後で行えば良い。ループの走査がボトルネックになっているなら、簡単に戻せる。

### 手順
- ループをコピーする。
- 重複による副作用を特定して排除する。
- テストする。

- 上記で「ループの分離」は終わりだが、「ループの分離」の目的は、それ自体ではなく、次の一手のための準備である。通常は、分離したループをそれぞれ独立した関数に抽出できないかを調べる。

## パイプラインによるループの置き換え

### 動機
- 多くの言語は、ループより優れた仕組みとしてコレクションのパイプラインを提供するようになった。一般的なものはmapやfilterなど。
- パイプライン構造で表現すると、ロジックを追うのがはるかに簡単になる。パイプラインを通過するオブジェクト群の流れを上から下に向かって読むことができるからである。

### 手順
- ループ処理のコレクションように新しい変数を作成する。
- ループ内の各処理を上から一つずつ取り出し、コレクションのパイプライン操作に置き換えて、ループ処理のコレクション用変数を加工するパイプラインに付け加える。変更のたびにテストする。
- すべての処理をループから削除できたら、ループを取り除く。

## デッドコードの削除

### 動機
- 未使用コードはソフトウェアの動作を把握しようとする際の大きな足かせとなる。
- そのため、コードが使用されなくなったら削除すべきである。将来的に必要になったら、その際にバージョン管理システムから再び掘り起こせるので問題ない。

### 手順
- デッドコードが外部から参照できる場合、たとえばそれが独立した関数の場合は、呼び出しが残っていないことを確認するために検索する。
- デッドコードを削除する。
- テストする。