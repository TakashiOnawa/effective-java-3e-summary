# はじめに
　Effective Java 第 3 版の個人的なまとめ。

# 第 2 章 オブジェクトの生成と消滅
## 項目1 コンストラクタの代わりに static ファクトリメソッドを適用する
* 名前を持たせることができる。コンストラクタよりもどのような状態のインスタンスを生成するかが分かりやすい。
* 呼び出しごとに新たなオブジェクトを生成する必要がない。（分からない）
* メソッドの戻り値にサブクラス型を使用できる。
* パラメータによって返すクラスを変えられる。

## 項目2 多くのコンストラクタパラメータに直面したときにはビルダーを検討する
* Immutable なクラスで、コンストラクタに大量のパラメータが必要な場合、いくつめになんのパラメータ渡せばよいか分かりづらくなるよね。
* そんなときには Builder パターンってやつを使うとよい。

## 項目3 privae のコンストラクタか enum 型でシングルトン特性を強制する
* シングルトンには従来 2 つの方法がある。
  * public static final のフィールドによるシングルトン
  * satic ファクトリメソッドによるシングルトン
* でも結局のところ、<b>単一要素の enum を使用するのがシングルトンを実装する最善の方法</b>らしい。

## 項目4 private のコンストラクタでインスタンス化不可能を強制する
* static メソッドしかないようなユーティリティクラスを作るばあは、private コンストラクタを用意して明示的にインスタンス生成できないようにすべきという感じ。コンストラクタ内で return new AssertionError(); を実装するとクラス内からもインスタンス生成できなくなるからなお良い。
* そもそも、C# なら static クラスにすればよいが、<b>Java って static クラス作れないらしい！！！</b>

## 項目5 資源を直接結びつけるよりも依存関係注入を選ぶ
* クラス内で別クラスを new して使うとテストしづらいから DI しようよ、という話。
* コンストラクタでインスタンスを直接渡したり、ファクトリクラス渡したりする。後者の場合、Supplier クラスが有効。

## 項目6 不必要なオブジェクトの生成を避ける
* インスタンス生成はなるべく少なくしよう、という話。
* String s = new String("bikini"); これはダメ。"bikini" 自体がそもそも String のインスタンスですし。
* String s = "bikini"; とする。
* new Boolean(String) 使うとインスタンス生成してしまうが、Boolean.valueOf(String) 使うとインスタンス生成されない。 True, False は Immutable だから Boolean の内部に 2 つのインスタンスを持っているんだと思う。
* あとは、Integer とか Long を使うより int、long といいたプリミティブ型を使う。
* ループ内でのインスタンス生成には要注意。

## 項目7 使われなくなったオブジェクト参照を取り除く
* ちゃんと GC されるように参照を消さないとメモリリークするよという話。
* 自前でキャッシュしていたり、リスナーやコールバックを登録している場合があるある。前者は WeakHashMap、後者は弱い参照（weak reference）使うとよい。これ、難しい。。。

## 項目8 ファイナライザとクリーナーを避ける
* AutoClosable インターフェースで明示的にクローズが必要であることを示す。（C# の IDisposable 的なやつ。）
* 利用側は try-with-resouces を使用する。（C# の using 的なやつ。）

## 項目9 try-finally よりも try-with-resouces を選ぶ
* そのまま。
* 簡単に書けるしコードが短く明瞭になる。

# 第 3 章 すべてのオブジェクトに共通するメソッド
## 項目10 equals をオーバーライドするときは一般契約に従う
* 間違った方法でオーバーライドするぐらいならオーバーライドしない。
* 値を表現するクラスの場合にオーバーライドする。（ValueObject な。）
* スーパークラスに equals が実装されていて、サブクラスに 1 つフィールドを追加して equals の比較対象を増やす場合、equals の契約を破ってしまうことになる。（詳しくは書籍参照。）例えば、スーパークラス.equals(サブクラス) と サブクラス.equals(スーパークラス) が異なる結果になってしまう。よって、継承ではなくコンポジションを選んだ方が良い。
* スーパークラスが abstract であれば問題ない。（インスタンス作れないから。）
* float, double の比較には、Float.compare(float float), Double.compare(double, double) を利用する必要がある。Float.NaN, -0.0f, および同様の意 double の値があるため。
* equals の引数は必ず Object 型にすること。
* 結局は、AutoValue などのフレームワークを使ったほうがいいとのこと。

以下、一般的な equals
```java
@Override
public boolean equals(Object o) {
    if (o == this)
        return true;
    if (!(o instanceof MyClass))
        return false;
    Myclass mc = (MyClass)o;
    return mc.value1 == value1 &&
        mc.value2 == value2;
}
```
## 項目11 equals をオーバーライドするときは、常に hashCode をオーバーライドする
* HashMap の Key にオブジェクトを指定した場合、hashCode が Key として使われる。
* 結局は、AutoValue などのフレームワークを使ったほうがいいとのこと。

以下、一般的な hashCode
```java
// こっちは早い
@Override
public int hashCode() {
    int result = Integer.hashCode(id);
    result = 31 * String.hashCode(value);
    ・・・
}

// こっちは遅い
@Override
public int hashCode() {
    Objects.hash(id, value, ・・・)
}
```

## 項目12 toString を常にオーバーライドする
* デバッガで分かりやすくする。
* 伝わる形式で返却するべき。（電話番号なら単純なカンマ区切りではなくて、「市外局番-市内局番-回線番号」みたいな形式が良い。）
* ValueObject に対してはオーバーライドすると良い。
* toString の結果をデバッグ以外で使用してもよいのであれば、コメントに形式を明示的に書く。使用してほしくない場合もコメントに形式が変わるかもしない旨を書く。
* すべてのクラスでオーバーライドすると良いと書いてあるけど、基本は ValueObject だけかな。

## 項目13 clone を注意してオーバーライドする
* Cloneable 自体はからのインターフェースである。
* Object.clone() は protected である。
* Cloneable を実装しない場合、clone() は CloneNotSupportedException がスローされるが、実装した場合はそのフィールドをコピーしたオブジェクトが返却される。
* 結局、コピーしたい場合は、clone() よりもコピーコンストラクタやコピーファクトリを使ったほうが良いとのこと。正し、配列は clone() でコピーするのが望ましい。なぜかは分からない。配列に Immutable なオブジェクト使っている時だけな気がする。
* ValueObject などの Immutable なクラスは clone を用意する必要ない。だって状態変えられないから。

以下、実装方法
```java
// Public でオーバーライドする。
// 戻り値は Object 型ではなく、対象クラスの型にする。
@Override
public PhoneNumber clone() {
    try {
        // super.clone() でクローンする。
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        // CloneNotSupportedException は発生しないため、
        // 呼び出しもとで throws を書く必要がないようにする。
        throw new AssertionError();
    }
}

// クローン対象が可変なフィールドを持つ場合は以下のようにする必要がある
@Override
public Stack clone() {
    try {
        // super.clone() でクローンする。
        Stack result = (Stack)super.clone();
        // 可変なフィールドをクローンする。
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}

// super.clone() でクローンするのはお作法であり、
// 上記の例だと、elements が final だと使えない。
// よって、以下のようにコピーコンストラクタかコピーファクトリの方が良い。
public Stack(Stack stack) {・・・}
public static Stack newInstance(Stack stack) {・・・}
```

## 項目14 Comparable の実装を検討する
* インスタンスのリストをソートする可能性がある場合は Comparable を実装する。
* Comparable を実装しておくとソートが簡単。
```java
// こんな感じ
Arrays.sort(a);
```
* 大小比較はダメ。代わりにボクシングされた基本データクラスの compare メソッドか コンパレータ構築メソッドを使う
```java
// これはダメ
private Integer value;
@Override
public int compareTo(MyClass myClass) {
    return value - myClass.value;
}

// これか（基本データクラスの compare メソッド）
private Integer value;
@Override
public int compareTo(MyClass myClass) {
    return Integer.compare(value, myClass.value)
}

// これにする（コンパレータ構築メソッド）
private Integer value;
private static final Comparator<MyClass> COMPARATOR =
    comparingInt((MyClass o) -> o.value);
@Override
public int compareTo(MyClass o) {
    return COMPARATOR.compare(this, o);
}

// 複数の要素で比較する場合はこんな感じ（コンパレータ構築メソッド）
private Integer value1;
private Integer value2;
private static final Comparator<MyClass> COMPARATOR =
        comparingInt((MyClass o) -> o.value1).
                thenComparingInt(o -> o.value2);
@Override
public int compareTo(MyClass o) {
    return COMPARATOR.compare(this, o);
}
```
# 第 4 章 クラスとインターフェース
## 項目15 クラスとメンバーへのアクセス可能性を最小限にする
* とにかくアクセス可能性はできる限りアクセスできないようにせよ、という話。
* うまく設計されているコンポーネントは、実装と API（インターフェース）が分離されていて、コンポーネント同士の通信は API を通してのみ可能にして、実装は隠ぺいする。（DIP 使って抽象に依存せよ、ということと思う。）

## 項目16 Public のクラスでは、Public のフィールドではなく、アクセサーメソッドを使う
* メンバ変数は Public にしない。そりゃそうと思います。
* public final ついたフィールドならあり。正し、公開する型には注意。配列などは代入はできなくとも要素の追加や削除はできてしまうため public にするのはよろしくない。アクセサメソッドで
* パッケージプライベートか private なネストしたクラスなら公開することが望ましい場合があると書いてあるが、とにかく、<b>いつどこで誰が書き換えるかが分からなくなってしまうような公開の方法はしないこと</b>。

# 参考
* [jbloch/effective-java-3e-source-code](https://github.com/jbloch/effective-java-3e-source-code)
* [【Effective Java】各項目のまとめ](https://www.thekingsmuseum.info/entry/2016/12/17/174236)
