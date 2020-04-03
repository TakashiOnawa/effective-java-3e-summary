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
    // ローカルクラス（メソッド内でクラスの定義ができる）
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
        // クラスを作ることなくインタフェース実装できる（インタフェースだけでなく通常のクラスでも良い。）
        return new NoNameInterface() {
            @Override
            public int getField() {
                return field;
            }
        };
    }
}

public interface NoNameInterface {
    int getField();
}

/** 使い方 **/
MyClass myClass = new MyClass();
NoNameInterface noNameClass = myClass.myMethod();
int field = noNameClass.getField();
```

## ソースファイルを単一のトップレベルのクラスに限定する
* 1 ファイル 1 クラスにしなさい、ということかな？
* そもそも Java では 1 ファイル複数クラスってできないと思ってた。

# ジェネリックス
## 項目26 原型を使わない
* 原型型を使わずにジェネリックス型を使う。
* 例えば、List という原型を使うのではなく、List<String> というパラメータ化された型を使う。

```java
// 原型を使うと
List list = new ArrayList();
// 文字列とか
list.add("testString");
// 数値とか
list.add(1);
// なんでも入れられてしまうため、取り出すときに適切にキャストしないとエラーになってしまう

// ジェネリックス型を使用すると
List<String> list = new ArrayList<String>();
// 宣言した型のみ追加できる
list.add("testString");
list.add(1); // ← コンパイルエラー
// よって、安全である
```

```java
// 型が指定できない場合は、
// 原型ではなく
private static int myMethod(List a, List b) {...}
// 非境界ワイルドカード型を使う
private static int myMethod(List<?> a, List<?> b) {...}
// これだとなんでも入れられてしまうから、ある程度制限したい場合は
// 境界ワイルドカード型を使う
private static int myMethod(List<? extends Number> a, List<? extends Number> b) {...}
```

|用語|例|
|---|---|
|パラメータ化された型|List\<String\>|
|実型パラメータ|String|
|ジェネリック型|List\<E\>|
|仮型パラメータ|E|
|非境界ワイルドカード型|List\<?\>|
|原型|List|
|境界型パラメータ|\<E extends Number\>|
|再起型境界|\<T extends Comparable\<T\>\>|
|境界ワイルドカード型|List\<? extends Number\>|
|ジェネリックメソッド|static \<E\> List\<E\> asLIst(E[] a)|
|型トークン|String.class|

## 項目27 無検査警告を取り除く
* 無検査警告は必ず取り除く
* 安全である場合のみアノテーションで除外する
```java
@SuppressWarnings("unchecked")
// クラスを対象にしたり、メソッドを対象にしたり、メソッド内の処理を対象にしたりできる。
// return を対象にすることはできない。
```

## 項目28 配列よりもリストを選ぶ
* 配列は共変であり、ジェネリックスは不変である。ジェネリックスの方が安全。
```java
// 共変：これはできる。（Long は Object の派生なので）
// C# ならコンパイルエラーなのに。。。
Object[] objectArray = new Long[1];
// ↓ コンパイルエラー起きないけど実行時にエラーになる。
objectArray[0] = "aaa"; 

// 不変：これはコンパイルエラー。こっちのほうが安全。
List<Object> objectList = new ArrayList<Long>();
```

* ジェネリック配列が生成できない
```java
// これができない。
E[] test = new E[1];
// こうするとよい
List<E> test = new List<E>();
```

## 項目29 ジェネリック型を使う
```java
// こうするよりも
public class Stack {
    public void push(Object e) {
        ...
    }
    public Object pop() {
        ...
    }
}

// ジェネリッククラスを使って作る。
public class Stack<E> {
    public void push(E e) {
        ...
    }
    public E pop() {
        ...
    }
}
```
* ジェネリック型にはプリミティブ型の指定はできない。（int, boolean など。）
* ジェネリック型の中身の実装はパフォーマンスを考慮したうえで配列にすることもある。（無検査警告は使う側には影響ないため @SuppressWarnings をつけて OK。）

## ジェネリックメソッドを使う
```java
// こうするよりも
public static Set union(Set s1, Set s2) {
    ...
}

// ジェネリックメソッドを使って作る。
public static <E> Set union(Set<E> s1, Set<E> s2) {
    ...
}
```
* ジェネリック・シングルトン・ファクトリ、再起型境界とかの記述もあるけど、置いておく。m(__)m

## 項目31 API の柔軟性向上のために境界ワイルドカードを使う
* ジェネリックスは不変であるため、少々使いづらい場合がある。
* そんな時は境界ワイルドカード型を利用するとよい。

```java
public class Stack<E> {
    public void push(E e);
    public void pushAll(Iterable<E> src);
    public void popAll(Iterable<E> dst);
}

Stack<Number> stack = new Stack<>();
Iterable<Integer> pushList = new ArrayList<>();
Iterable<Integer> popList = new ArrayList<>();
stack.pushAll(list);
// ↑ Integer は Number を継承しているがコンパイルエラーになってしまう。
stack.popAll(popList);
// ↑ これも同じようにコンパイルエラーとなる。

// なぜならジェネリックは不変であるため。

// 境界ワイルドカード型を使用するとよい。
public class Stack<E> {
    public void push(E e);
    public void pushAll(Iterable<? extends E> src);
    public void popAll(Iterable<? super E> dst);
}
```

### PECS（Produser-Extends, Consumer-Super）、Get&Put 原則
* extends と super の使い分けの原則。
* パラメータがプロデューサ（生産者）ならば extends を利用する。
* パラメータがコンシューマ（消費者）ならば super を利用する。
* よくわからなかったので調べてみると分かりやすい記事があった。
[境界ワイルドカード型をいつ使うか、どちらを使うか](https://relearn-java.com/generics/#i-34)
>引数から値などを取り出す操作（Producer、get）だけを行う場合は<? extends T>を、引数に値を渡して格納したり処理させる操作（Consumer、put）だけを行う場合は<? super T>を、両方行う場合は境界ワイルドカード型は使わない

>```java
>public static<T> void copy(List<? extends T> from, List<? super >T> to) {
>    to.put(from.get());
>}
>```

* extends は上限を示し、super は下限を示す。

```java
// extends は E で指定した型を継承するクラスを渡せる。（E で指定した型を含む。）
Stack<Number> stack = new Stack<>();
// Number 型を継承する Integer 型のリストを渡せる。
Iterable<Integer> pushList = new ArrayList<>();
stack.pushAll(pushList);

// super は E で指定した型の親クラスを渡せる。（E で指定した型を含む。）
// E で指定した型を継承するクラスは渡せない。
Stack<Number> stack = new Stack<>();
// Number 型を継承する Integer 型のリストを渡せない。
Iterable<Integer> popList = new ArrayList<>();
stack.popAll(popList); // ← コンパイルエラー

//これならできる。（Number は Object を継承するため。）
Iterable<Object> popList = new ArrayList<>();
stack.popAll(popList);
```

## 項目32 ジェネリクスと可変長引数を注意して組み合わせる
* 可変長パラメータを持つメソッドを作るなら、配列は使うな、ジェネリック型を使え。そのうえで可変長パラメータを持つメソッドには @SafeVarargs を付けろ。
* そして、そもそも可変長パラメータは使わず List を渡せ。
という理解でよいだろうか。。。

```Java
// こんな感じでジェネリクス型にする。
@SafeVarargs // ← 安全な可変長パラメータを持つメソッドにはこれを付ける。
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = ne ArryaList<>();
    for (List<? extends T> list : lists) {
        result.addAll(list);
    }
    return result;
}
```

## 項目33 型安全な異種コンテナを検討する
* クラスをキーにしたコンテナ（中に Map を持つクラス）を作ると、異なる型を一つのクラスに格納できる。
```Java
public class Favorites {
    privater Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}

public static void main(String[] args) {
    Favorites f = new Favorites();
    f.putFavorite(String.class, "Java");
    f.putFavorite(Integer.class, new Integer(1));
    f.putFavorite(Class.class, Favorites.class);
    String favoriteString = f.getFavorite(String.class);
    System.out.pringf("%s", favoriteString); // ← 「Java」と出力される。
}
```

# 第 6 章 enum とアノテーション
## 項目34 int 定数の代わりに enum を使う
* int や String などの定数だと、型安全ではないため、間違った値を要れたとしてもコンパイルエラーにならない。
* 定数固有メソッドの実装ができる。
* enum.values() で定数の定義順に列挙できる。
* コンパイル時に要素が分かっている定数の集合が必要な場合は enum を使う。

```Java
// 定数固定メソッドを持つ enum
public enum Operation {
    PLULS { public double apply(double x, double y) { return x + y; } },
    MINUS { public double apply(double x, double y) { return x - y; } }

    public abstract double apply(double x, double y);
}

// こんな感じにもできる。
public enum Operation {
    PLULS("+") { public double apply(double x, double y) { return x + y; } },
    MINUS("-") { public double apply(double x, double y) { return x - y; } }

    private final String symbol;

    private Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    private static final Map<String, Operation> stringToEnum = Stream.of(values())
        .collect(toMap(Object::toString, e -> e));

    // 文字列を Operation に変換するようなメソッドも検討すると良い。
    public static Optional<Operation> fromString(String symbol) {
        // 下記の様にあらかじめ Map を作っておいても良いし、
        // Values() でループして探しても良い。
        return Optional.ofNullable(stringToEnum.get(symbol));
    }

    public abstract double apply(double x, double y);
}

// values() で定数の定義順に列挙できる。
public static void main(String[] args) {
    for (Operation op : Operation.values()) {
        System.out.printf("%s", op.toString());
    }
}
```

enum の戦略パターン
```Java
enum PayrollDay {
    // PayType が計算の戦略となる。
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND), // 週末
    SUNDAY(PayType.WEEKEND); // 週末

    private final Paytype payType;

    private PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    private enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                // 平日用の残業計算
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                // 週末用の残業計算
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);

        int pay(int minutesWorked, int payRate) {
            // 基本給 ＋ 残業代
            return (minutesWorked * payRate) + overtimePay(minutesWorked, payRate);
        }
    }
}
```

## 項目35 序数の代わりにインスタンスフィールドを使う
* 序数というのは Enum.ordinal() で取得できるインデックスの事。
```Java
// これはダメ
enum Ensemble {
    SOLO, // ordinal() = 0
    DUET // ordinal() = 1

    // ordinal() から動的に計算してはいけない。
    int numberOfMusicians() { return ordinal() + 1; }
}

// これは良い
enum Ensemble {
    SOLO(1), // ordinal() = 0
    DUET(2) // ordinal() = 1

    // フィールドに持つ
    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }

    int numberOfMusicians() { return numberOfMusicians; }
}
``` 

## 項目36 ビットフィールドの代わりに EnumSet を使う
```Java
class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE }

    // EnumSet<Style> 型の日k数でもいいけど、
    // Set<Style> といった Interface にしておいた方が呼び出し側に自由度が生まれるから良い。（これは enum に限った話ではない。）
    public void applyStyles(Set<Style> styles) {
        // ・・・
    }
}

public static void main(String[] args) {
    Text text = new Text();
    // こんな感じで EmumSet で複数渡せる。
    text.applyStyles(EnumSet.of(BOLD, ITALIC));
}
```

## 項目37 序数インデックスの代わりに EnumMap を使う
* Enum の値毎の Set を作りたいなら、Set の配列でインデックスに ordinal() を利用するのではなく、EnumMap を利用してキーに Enum を利用する。

```Java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIEENNIAL }

    final LifeCycle lifeCycle;
}

// 使い方
MapM<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lc, new HashSet<>());
}
for (Plant p : garden) {
    plantsByLifeCycle.get(p.lifeCycle).add(p);
}
```

## 38 拡張可能な enum をインターフェースで模倣する
* enum を継承して enum を作ることはできないが、インターフェースを実装した enum を作ることで拡張できる。
```Java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    };

    private final String symbol;
    private BasicOperation(String symbol) {
        this.symbol = symbol;
    }
}

public enum ExtendedOperation implements Operation {
    EXA("^") {
        public double apply(double x, double y) { return Math.pow(x, y); }
    };
}
```

## 項目39 命名パターンよりもアノテーションを選ぶ
* 例えばテストなどで、testXXX といった場合は、テストメソッドとして認識させるといった、命名規則で判断させることはせずに、@Test といったアノテーションを付けようよ、という話。
```Java
// 自作のアノテーションを作ることができる。
public @interface MyTest {}

// 自作アノテーションにアノテーションを付けることができる。
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyTest {}

// こうやって使う
public class Sample {
    @MyTest
    public void testXXX() { ... }
}

// アノテーションにメンバを持たせることもできる
public @interface ExceptionTest {
    Class<? extends Throwable> values();
}

// こうやって使う
public class Sample {
    @ExceptionTest({ IndexoutOfBoundsException.class,
                     NullPointerException.class})
    public void testXXX() { ... }
}

// こんな感じで処理できる。
public class RunTests {
    public static void main(String[] args) throws Exception {
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            // アノテーションがついている場合はメソッドを実行する。
            if (m.isAnnotationPresent(ExceptiionTest.class)) {
                try {
                    m.invoke(null);
                } catch (InvocationTargetException wrappedEx) {
                    // 発生した例外の型を取ってきて、
                    Throwable exc = wrappedEx.getCause();
                    // アノテーションに指定した期待する例外の型を取ってきて
                    Class<? extends Throwable> excType =
                        m.getAnnotaion(ExceptionTest.class).value();
                    // 発生した例外が期待する例外なら
                    if (excType.isInstance(exc)) {
                        // テストをパス
                    }
                }
            }
        }
    }
}
```

## 項目40 常に Override アノテーションを使う
* オーバーライドしたいという意思が合って実装するならば、必ず @Override を付ける。そうしないとバグに気づかない恐れがある。
```Java
// equals メソッドをオーバーライドしよとしたときに、以下の様に実装してしまった場合は、
// 間違った実装にもかかわらずコンパイルエラーは発生しない。
// （equals は、パラメータが Object 型である必要がある。）
public boolean equals(Sample s) { ... }

// こうやって実装するとコンパイルエラーが発生するため間違いに気付ける。
@Override
public boolean equals(Sample s) { ... }
```

## 項目41 型を定義するためにマーカーインターフェースを使う
* インターフェースを引数の型として使うことで、コンパイルの力を借りることができる。

# 第 7 章 ラムダとストリーム

## 項目42 無名クラスよりもラムダを選ぶ
* 無名クラスよりもラムダの方が簡潔である。
* ただし、ラムダでの計算が自明ではなかったり、計算が数行を超えたりする場合は使わない。1 行が理想。3 行が妥当。
```Java
// 無名クラス
Collections.sort(words,
    new Comparator<String>() {
        public int compare(String s1, String s2) {
            return Integer.compare(s1.length(), s2.length());
        }
    });

// ラムダを使うと
Collections.sort(words, (s1, s2) => Integer.compare(s1.length(), s2.length));
```

```Java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    public abstract double apply(double x, double y);
}

// 上記をラムダで書き換えると
public enum Operation {
    PLUS  ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y);

    private final String symbol;
    private final DoubleBinaryOperator op; // 関数型インターフェース。項目 44 参照。

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    public double apply(double x, double y) { return op.applyAsDouble(x, y); }
}
```

## 項目43 ラムダよりもメソッド参照を選ぶ
* メソッド参照の方がラムダよりも短く明瞭である場合はメソッド参照を使う。そうでない場合はラムダを使う。
```Java
// ラムダ
map.merge(keym 1, (count, incr) -> count + incr);
// メソッド参照
map.merge(key, 1, Integer::sum);
```

## 項目44 標準の関数型インターフェースを使う
* 基本的に、java.util.function.Function で提供されている標準の関数型インターフェースを利用する。
* 独自の関数型インターフェースには場合は必ず @FunctionalInterface アノテーションを付ける。

|インターフェース|関数のシグニチャ|例|説明|
|---|---|---|---|
|UnaryOperator<T>|T apply(T t)|String::toLowerCase|Tを受け取ってTを返す関数|
|BinaryOperator<T>|T apply(T t1, T t2)|BigInteger::add|Tを2つ受け取ってTを返す関数|
|Predicate<T>|boolean test(T t)|Collection::isEmpty|Tを受け取って判定する関数|
|Function<T,R>|R apply(T t)|Arrays::asList|Tを受け取ってRを返す関数|
|Supplier<T>|T get()|Instant::now|Tを返す。|
|Consumer<T>|void accept(T t)|Sysmte.out::println|Tを受け取って処理する関数|
※上記の他にも沢山ある。

例）
```Java
UnaryOperator<String> op = s -> s.toUpperCase();
System.out.println(op.apply("abc")); // ABC

BinaryOperator<String> op = (s1, s2) -> s1 + s2;
System.out.println(op.apply("abc", "def")); // abcdef

Predicate<String> p = s -> s.isEmpty();
System.out.println(p.test("abc")); // false
System.out.println(p.test("")); // true

Function<String, File> f = s -> new File("/tmp", s);
File file = f.apply("test.txt");
System.out.println(file); // /temp/test.txt

Supplier<String> s = () -> "abc";
System.out.println(s.get()); // abc

Consumer<String> c = s -> System.out.println(s);
c.accept("abc"); // abc
```

参考：http://www.ne.jp/asahi/hishidama/home/tech/java/functionalinterface.html#h_Consumer

## 項目45 ストリームを注意して使う
* ストリームパイプラインは 0 個以上の中間操作と 1 つの終端操作から構成される。
* ストリームパイプラインは終端操作で初めて実行される。（遅延評価）
* **乱用すると読みづらくなるから注意すること!!**
* ラムダのパラメータに適切な名前を付けたり、中間操作で適切に命名されたメソッドを利用したりして整理する。
* ストリーム内では final な変数のみ利用できる。

## 項目46 ストリームで副作用のない関数を選ぶ
* foreach 操作は、ストリームの計算結果を報告するためだけに使い、計算を行うために使うべきではない。
```Java
// これはダメ（foreach 内で freq の値を変えているため。）
Map<String , Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.foreach(word -> {
        freq.marge(word.toLowerCase(), 1L, Long::sum);
    })
}
// これは良い
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```
* その他、toSet, toMap, groupingBy, joining などのコレクターファクトリに関する使い方が記載されているが、読んでも頭に入ってこないから使いながら覚える。

## 項目47 戻り値型として Stream よりも Collection を選ぶ
* ループでしか使われないなら Iterable を返すべきだが、stream が使えないので、Collection やそのサブタイプを返すのが良い。
* その他、メモリを節約するやり方が書いてあるけどよく分からない。。。

## 項目48 ストリームを並列化するときは注意を払う
* **計算の正しさを維持し**、パフォーマンス向上に値する十分な理由がない限り、ストリームパイプラインの並列化は試みない。
* コードの正しさが維持さ
れ、測定の結果がパフォーマンス向上を裏付けるなら利用する。
```Java
// 例：素数を数えるストリームパイプラインの並列化バージョン
static logn pi(long n) {
    return LongStream.rangeClosed(2, n)
        .parallel() // 並列化のパイプライン
        .mapToObj(BegInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}
```

# 第 8 章 メソッド
## 項目49 パラメータの正当性を検査する。
* メソッドやコンストラクタを各場合、その都度、そのパラメータにどのような制約が存在するかを考え、チェックする。
* null チェックや、範囲のチェックなど。
* public と protected のメソッドに対してはチェックした後にスローする例外を @throws タグを使って文書化すること。
* private なメソッドには assertion を利用できる。

```Java
// これで null の時に例外をスローしてくれる。
this.strategy = Objects.requiredNonNull(strategy, "strategy");

// その他に、checkFromIndexSize, checkFromToIndex, checkIndex などがある。
```
```Java
// アサーションの使用例
// java コマンドに -ea を指定しない限り assert は有効にならない。（コストも発生しない。）
private static void sort(long a[], int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    // 計算を行う
}
```

## 項目50 必要な場合、防御的にコピーする
* 外からコンストラクタなどに渡されたインスタンス、外に公開するインスタンスはコピーすることでクラスを不変にする。
```Java
// ダメな例
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }
}

Date start = new Date();
Date end = new Date();
Pefiod p = new Period(start, end);
end.setYear(78); // Period のインスタンスの内部を変えてしまう！
p.start().setYear(78); // Period のインスタンスの内部を変えてしまう！

// 良い例
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime()); // コピーする。
        this.end = new Date(end.getTime()); // コピーする。
    }

    public Date start() {
        return nre Date(start.getTime()); // コピーする。
    }
}
```

* そもそも Date は使わずに、Instant、LocalDateTime、ZonedDateTime などの不変なクラスを利用する。
* パフォーマンスを優先しなければならない場合、コピーすることが必ずしも正義ではない。値を変えないようドキュメンテーションで明確して対処する。

# 参考
* [jbloch/effective-java-3e-source-code](https://github.com/jbloch/effective-java-3e-source-code)
* [【Effective Java】各項目のまとめ](https://www.thekingsmuseum.info/entry/2016/12/17/174236)
* [境界ワイルドカード型をいつ使うか、どちらを使うか](https://relearn-java.com/generics/#i-34)
