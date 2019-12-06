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
* AutoClosable インタフェースで明示的にクローズが必要であることを示す。（C# の IDisposable 的なやつ。）
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
* Cloneable 自体はからのインタフェースである。
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
# 第 4 章 クラスとインタフェース
## 項目15 クラスとメンバーへのアクセス可能性を最小限にする
* とにかくアクセス可能性はできる限りアクセスできないようにせよ、という話。
* うまく設計されているコンポーネントは、実装と API（インタフェース）が分離されていて、コンポーネント同士の通信は API を通してのみ可能にして、実装は隠ぺいする。（DIP 使って抽象に依存せよ、ということと思う。）

## 項目16 Public のクラスでは、Public のフィールドではなく、アクセサーメソッドを使う
* メンバ変数は Public にしない。そりゃそうと思います。
* public final ついたフィールドならあり。正し、公開する型には注意。配列などは代入はできなくとも要素の追加や削除はできてしまうため public にするのはよろしくない。アクセサメソッドで
* パッケージプライベートか private なネストしたクラスなら公開することが望ましい場合があると書いてあるが、とにかく、<b>いつどこで誰が書き換えるかが分からなくなってしまうような公開の方法はしないこと</b>。

## 項目17 可変性を最小限にする
* 不変オブジェクトを利用する。作られた時から状態が変わらない。安全。
* ValueObject のことですね。
* 状態を変更するメソッドを持たせず、クラスを拡張できないようにし（サブクラスを作らせないように final クラスにする）、すべてのフィールドを private final にし、変更可能なオブジェクトを内部に持つ場合はそのオブジェクトを外部からを取得できないようにする。（コピーして返すなら可。）
* サブクラスをっ作らせないようにするためには、final クラスにする他 private コンストラクタと static なファクトリメソッドを作ることでも実現できる。筆者はどちらかというと後者を進めている感じがする。
* すべてのフィールドを final にするとあるが必ずそうでなくてもよい。**要は、外部からの操作により状態が変更できなければよい。**例えば、getHashCode などは初めて呼ばれたときにフィールドにキャッシュするようにしてもよい。

## 項目18 継承よりもコンポジションを選ぶ
* サブクラスはスーパークラスの詳細を知らなければ作れないし、スーパークラスの変更されるとサブクラスに影響する。サブクラスとスーパークラスは一緒に成長して行かなければならない。カプセル化を破っていることになる。
* ラッパークラスを作るとラップされたクラスにメソッドが追加されたとしてもラッパークラスには影響しない。
```java
// こうじゃない
public class MyHashSet<E> extends HashSet<E> {
    @Override
    public void add(E e) {
        // サブクラス独自処理
        ・・・
        super.add(e);
    }

    @Override
    public void clear() {
        // サブクラス独自処理
        ・・・
        super.clear();
    }
    ・・・
}

// こうする
// HasSet のインタフェースである Set を実装した転送クラスを作る。
public class ForwardingSet<E> implements Set<E> {
    // Set をラップする。
    private final Set<E> innerSet;
    public ForwardingSet(Set<E> s) {
        innerSet = s;
    }

    // 内部に保持する Set に処理を委譲する。
    public void add(E e) { innerSet.add(e); }
    public void clear() {innerSet.clear(); }
    ・・・
}
```
## 項目19 継承のために設計および文書化する、でなければ継承を禁止する
* タイトルがすごいな
* ぶっちゃけ余りよくわからなかった。。。
* オーバーライドできるメソッドは、スーパークラスでどのように自己利用されているかをメソッドコメントにしっかり記載する必要がある。
* スーパークラスはコンストラクタでオーバーライド可能なメソッドを呼んではいけない。
* スーパークラスを Cloneable, Serializable にすべきではない。
* サブクラスが作れるようにするにはそれなりの覚悟が必要なので、サブクラスの必要性がない場合はクラスを final にするか、private コンストラクタで static ファクトリメソッドを作ることで継承できないようにするのが吉。
* とにかく継承は危ないよ、という感じ。項目18 で言っている通り、カプセル化を破ることになる。

## 項目20 抽象クラスよりもインタフェースを選ぶ
* なんや難しくなってきたぞ。
* 単一継承であるため、既に抽象を持つクラスに対して抽象クラスによる機能拡張をしたい場合、階層の一番上に新たな抽象クラスを配置しなければならない。これにより、機能拡張不要なサブクラスに対しても抽象クラスの機能拡張が加わってしまう。
* インタフェースは必要なクラスにのみ implements を追加することができる。
* インタフェースはミックスインに最適。（本来の型に新たな振る舞いを追加すること。）
```java
public interface Singer {
    AudioClip sing(Song s);
}

public interface Songwriter {
    Song compose(int chartPosition);
}

public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip sing(Song s);
    Song compose(int chartPosition);
}
```
### デフォルトメソッド
* Java 8 からデフォルトメソッドといって、インタフェースにメソッドの実装が書けるようになった。（C# は 8 からっぽい。）
* equals と hashCode はデフォルトメソッドの提供が許されていない。
### 抽象骨格実装
* インタフェースで型を定義して、骨格実装（抽象クラス）ではそのデフォルト実装を行う。
* 骨格実装には AbstractXXX という名前をつけるのが一般的。AbstractCollection, AbstractSet, AbstractList, AbstractMap とか。
* 骨格実装のいいところは、骨格実装を拡張できないのであれば、インタフェースを独自に実装すればよい。
* 可能な限りインタフェース上のデフォルトメソッドとして骨格実装を提供する。
* **インタフェースで振る舞いを定義して、その際になるべくデフォルトの振る舞いをデフォルトメソッドで実装して、デフォルト振る舞いを変えた基本機能を用意したい場合は抽象骨格実装を定義する。** といった感じだろうか。
```java
// インタフェースで振る舞いを定義
public interface MyInterface {
    // なるべくデフォルトメソッドで実装
    default void MyMethod() {
        // 何かしらのデフォルト処理
    }
}

// 基本機能を用意したい場合は抽象骨格実装
public abstract AbstractXXXClass implements MyInterface {
    @Override
    public void MyMethod() {
        // 何かしらの基本処理
    }
}

// 別の基本機能を用意した抽象骨格実装
public abstract AbstractYYYClass implements MyInterface {
    @Override
    public void MyMethod() {
        // 何かしらの基本処理
    }
}
```

## 項目21 将来のためにインタフェースを設計する
* Java 8 までは、インタフェースに機能が追加されると既存の実装が動作しなくなる（既存のサブクラスはその実装を持たないためコンパイルエラーになる。よって、すべてのサブクラスに実装を追加しなければなかった）
* デフォルトメソッドによる機能追加を行うとサブクラスの修正が不要になる。
* コンパイルエラーは回避できるかもしれないが、サブクラスの実装によっては実行時に正しく動かなくなる恐れがある。
* **既存のインタフェースを拡張する際にデフォルトメソッドを使うことは避けるべきである。**
* **もしデフォルトメソッドを使用するならば、そのサブクラスたちが正しく動くかどうかをしっかり調べる必要がある。**
* **インタフェース作成時にはなるべくデフォルトメソッドを実装するとよい。**
* **このように、リリースしてからインタフェースを変えることはそれなりにリスクがあるので、リリース前にテストが必要。最低でも 3 つのサブクラスを作ってみるとよい。**

## 項目22 型を定義するためだけにインタフェースを使う
* インタフェースで定数を提供してはならない。
* 定数を提供するなら static クラスとか、enum を使いなさい。

## 項目23 タグ付きクラスよりもクラス階層を選ぶ
* ここでいうタグとは、特性を示すフィールドである。例えば、あるクラスのフィールドに円か長方形化を示す値を持っていて、どちらかによって動きが変わる、てきな感じ。
* 冗長になるし、switch 文多発するし、良いことなんて何もない。
* そうじゃなくて、Figure という抽象クラスを継承した Circle、Rectangle を作るべき。
* 項目16 で継承よりコンポジションと言っているけど、どっちなんだろう。。。ものによる、ということか。Figure という抽象クラスはインタフェースでもいいのかも。

## 項目24 非 static のメンバークラスよりも static のメンバークラスを選ぶ
* メンバークラスというのはクラス内に記述されたクラスのこと。（ネストしたクラスという）
* ネストしたクラスには、**static のメンバークラス**、**非 static のメンバークラス**、**無名クラス**、**ローカルクラス**、がある。
* static のメンバークラスと非 static のメンバークラスならば、static のメンバークラスを使う。（非 static のメンバークラスのインスタンスは、エンクロージングクラスの参照を持つため、使い方間違えると GC されない恐れがある。）
* ローカルクラスは、メソッド内にクラスを定義できる。
* 無名クラスは、

```java
// エンクロージングクラス
public class MyClass {
    private int myClassField;
    public MyClass(int field) {
        myClassField = field;
    }

    // 非 static のメンバークラス
    public class NonStaticClass {
        private int nonStaticClasField;
        public NonStaticClass(int field) {
            nonStaticClasField = field;
        }
        int getMyClassField() {
            // エンクロージングクラスのフィールドにアクセスできる。
            return MyClass.this.myClassField;
        }
    }

    // static のメンバークラス
    // 基本的にはこっちを使うこと
    public static class StaticClass {
        private int staticClassField;
        public StaticClass(int field) {
            staticClassField = field;
        }
    }
}

/** 使い方 **/
// 非 static なメンバークラスはエンクロージングクラスのインスタンスを生成しなければ利用できない。
MyClass myClass = new MyClass(1);
MyClass.NonStaticClass nonStaticClass = myClass.new NonStaticClass(2);
// ↑ nonStaticClass インスタンスは myClass インスタンスへの参照を持つ。

// static なメンバークラスはエンクロージングクラスのインスタンスがなくても生成できる。
MyClass.StaticClass staticClass = new MyClass.StaticClass(3);
```

```java
// ローカルクラスの使い方
public void MyMethod() {
    // ローカルクラス
    class LocalClass {
        // 何かしらのフィールドやメソッド
    }

    LocalClass localClass = new LocalClass();
}
```

```java
// 無名クラスの使い方
public class MyClass {
    public NoNameClass myMethod() {
        int field = 1;
        return new NoNameClass() {
            @Override
            public int getField() {
                return field;
            }
        };
    }
}

public class NoNameClass {
    public int getField() {
        return -1;
    }
}

/** 使い方 **/
MyClass myClass = new MyClass();
NoNameClass noNameClass = myClass.myMethod();
int field = noNameClass.getField(); // オーバーライドされたメソッドが利用される。
```


# 参考
* [jbloch/effective-java-3e-source-code](https://github.com/jbloch/effective-java-3e-source-code)
* [【Effective Java】各項目のまとめ](https://www.thekingsmuseum.info/entry/2016/12/17/174236)
