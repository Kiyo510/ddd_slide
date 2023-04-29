-
## ドメイン駆動設計とは
### ドメインの知識に焦点をあてた設計手法
#### ドメインとは？
-  ソフトウェア開発において、 「プログラムを適用する対象となる領域」。
-  重要なのはドメインが何かではなく「ドメインに何が含まれるか」ということ。
	- 例 会計システムのドメイン
		- 金銭、帳票
	- 例 物流システムのドメイン
		- 貨物、倉庫、輸送手段
### 「焦点を当てる」とは？
-  ソフトウェア利用者の視点や考え、取り巻く環境などを真に理解する必要がある
		- ドメインと向き合う必要がある。
	- ドメインの概念や事象を理解し、その中から問題解決に役立つものを抽出して得られた知識をソフトウェアに反映する。
		- これは当たり前だが技術指向の開発者であればあるほど疎かにしやすい。
		- ピカピカのハンマーは開発者の目を曇らせ、見るものすべてを釘に変えてしまう
- ソフトウェアを適用する領域(ドメイン)と向き合い、ここに渦巻く知識に焦点をあてるということ。
	- よく観察し、よく表現すること。これは当たり前だが難しい。ドメイン駆動設計の指針はこれを補佐するものである。ドメイン駆動設計は当たり前を当たり前に実践するための開発手法である。

- 厳選された知識が集約され、適切に表現されたコードは有用な知識が詰めこまれたドキュメントの様相を呈してくる。(究極、非エンジニアでも何をやっているのかなんとなくわかる)

### ドメインモデル
#### モデルとは
- 現実の事象や概念を抽象化したもの。
	- 例 ペン
		-  小説家 : 文字が書けることが大事
		- 文房具店: 文字が書けることよりも値段が重要視される(利益を出す必要がある)
	- 対象が同じものであっても、何に重きを置くかは異なる。
	- 例: トラック
		- 物流システム: 「荷運びができること」が表現できればよい。
			- エンジンキーを回すとエンジンがかかるといったことまで表現する必要はない
- 上記のような事象や概念を抽象化する作業をモデリングと言い、その結果得られるのがモデル。
- ドメイン駆動設計ではこのモデルのことをドメインモデルと呼ぶ。
- ドメインの世界の住人は、ドメインの知識はあるが、ソフトウェアにとって重要な知識がどれかは不明。逆に開発者はソフトウェアにとって重要な知識はわかるが、ドメインについての知識がない。
	- 両者は協力する必要がある。
### 知識をコードで表現するドメインオブジェクト
- ドメインモデルだけでは抽象化されただけでなんの問題解決にもならないので、ソフトウェアで動作するモジュールとして表現する必要がある。これがドメインオブジェクトである。
- 当然ソフトウェアは変化していく。
	- ドメインの概念→ドメインモデル→ドメインオブジェクト
		- 上記3つは互いに影響し合う

## ドメイン駆動設計のパターン
- 知識を表現する
	- 値オブジェクト
	- エンティティ
	- ドメインサービス
- アプリケーションを実現するためのパターン
	- リポジトリ
	- アプリケーションサービス
	- ファクトリ
- 知識を表現する、より発展的なパターン
	- 集約
	- 仕様
    
## 知識を表現する
### 値オブジェクト(Value Object)
- システム固有の値を表現するために定義されたもの。

例えば、下記のような工業製品番号の定義をみてみる。
工業製品は、ロットやシリアル、製品番号など製品を識別するためにさまざまな意味を持たせた番号が存在している。

これをプリミティブな値のみで表現してみる。

```php
$modelNumber = 'a20421-100-1';  
```

- プリミティブな値だけだと表現力に乏しい。ソースコード上からドメイン知識が欠落している。

#### 値オブジェクトを使ってみる
```php
class ModelNumber
{
    public function __construct(
        private readonly string $productCode,
        private readonly string $branch,
        private readonly string $lot
    ) {
        if (!$this->isValidBranch()) {
            throw new \InvalidArgumentException("branchは3桁である必要があります");
        }
    }

    public function productCode(): string
    {
        return $this->productCode;
    }

    public function branch(): string
    {
        return $this->branch;
    }

    public function lot(): string
    {
        return $this->lot;
    }

    public function __toString(): string
    {
        return "{$this->productCode}-{$this->branch}-{$this->lot}";
    }

    private function isValidBranch(): bool
    {
        return strlen($this->branch) === 3;
    }
}

$number = new ModelNumber("a20421", "100", "1");
echo $number->productCode() . PHP_EOL;// a20421
echo $number->branch() . PHP_EOL;// 100
echo $number->lot() . PHP_EOL;// 1
echo $number . PHP_EOL;// a20421-100-1
```

- 値をオブジェクトとして表現してあげることで、表現力が増す。
- 仕様がひとめでわかる。
- 不正な値を存在させない。

#### 値に振る舞いを持たせられる
下記のような値オブジェクトがあるとする
```php
<?php

class Money
{
    public function __construct(private readonly float $amount, private readonly string $currency)
    {
    }

    public function amount(): float
    {
        return $this->amount;
    }

    public function currency(): string
    {
        return $this->currency;
    }
}
```

値オブジェクトはデータを保持するコンテナではない。
ふるまいをもつことができる。
お金は加算することがある↓

```php
class Money
{
    public function __construct(private readonly float $amount, private readonly string $currency)
    {
    }
    
    (略)

    // お金を加算する
    public function add(Money $money): Money
    {
        if ($this->currency !== $money->currency) {
            throw new \InvalidArgumentException("通貨単位が異なります");
        }

        return new Money($this->amount + $money->amount, $this->currency);
    }
}

$myMoney = new Money(100, "JPY");  
$yourMoney = new Money(200, "JPY");  
$result = $myMoney->add($yourMoney);  
echo $result->amount(); // 300

$myMoney = new Money(100, "JPY");  
$yourUSDMoney = new Money(2.5, "USD");  
// Fatal error: Uncaught InvalidArgumentException: 通貨単位が異なります 
$result = $myMoney->add($yourUSDMoney);
```

-  計算処理にルールを記述し、それにそぐわない操作を弾くようにしてシステマチックにバグを防止できる。
- 値オブジェクトはただのデータの構造体ではなく、オブジェクトに対する操作をふるまいとしてまとめることで、自身に関するを語るドメインオブジェクトらしさを帯びてくる。
- 逆に上記に定義されないことで、それができないということも示している。

### エンティティ
- ドメインモデルを実装したドメインオブジェクト。
- 値オブジェクトを真逆にしたもの

#### エンティティの性質
##### 可変である
```php
<?php

class User
{
    public function __construct(private string $name)
    {
    }

    //getter
    public function name(): string
    {
        return $this->name;
    }

    public function changeName(string $name): void
    {
        if (mb_strlen($name < 3)) {
            throw new InvalidArgumentException("名前は3文字以上です");
        }

        $this->name = $name;
    }
}

$user = new User("John");
$user->changeName("Elisa");
echo $user->name(); // Elisa
```

ただ、基本的にオブジェクトは不変にすべき

- 同じ属性であっても区別される。
- 例: 氏名が同じだったら人間は同一人物である？→ 違う。
	- 人間は属性では区別されない。もっと別のところで評価される。まさにエンティティとしてふさわしい
	- 何で区別するか？→ 識別子。等価性ではなく、同一性によって識別される必要がある。
```php
<?php

class UserId
{
    public function __construct(private readonly int $value)
    {
    }

    public function value(): int
    {
        return $this->value;
    }
}


class User
{
    public function __construct(private readonly UserId $id,private string $name)
    {
    }

    public function name(): string
    {
        return $this->name;
    }

    public function changeName(string $name): void
    {
        if (mb_strlen($name < 3)) {
            throw new InvalidArgumentException("名前は3文字以上です");
        }

        $this->name = $name;
    }
}
```

- 識別子は同一性の実体なので、readonlyをつける → 読み取り専用にして値が不変であることを担保。
- エンティティでは属性を表す識別子だけが比較の対象となる。属性が変化してもエンティティの同一性は変わらない。

```php
class User
{
    public function __construct(private readonly UserId $id, private string $name)
    {
    }

    //getter
    public function name(): string
    {
        return $this->name;
    }

    public function changeName(string $name): void
    {
        if (mb_strlen($name < 3)) {
            throw new InvalidArgumentException("名前は3文字以上です");
        }

        $this->name = $name;
    }

    public function equals(object $other): bool
    {
        //同一オブジェクトであればtrueを返す
        if ($this === $other) {
            return true;
        }

		if (gettype($obj) !== gettype($this)) {
			return false; 
		}

        return $this->id->value() === $other->id->value();
    }
}

$user = new User(new UserId(1), "test");
echo $user->name(). PHP_EOL;
$user->changeName("test2");
echo $user->equals(new User(new UserId(1), $user->name())) ? "true" : "false" . PHP_EOL;
```

#### エンティティにする判断基準
- ライフサイクルが存在するか
	- ユーザーは登録があり、途中で名前が変わったり、退会することがある

##### ドメインオブジェクトを定義するメリット
- コードのドキュメント性
	- 無口なコード
	```php
	class User
	{
		public string $name;
	}
	```

- 饒舌なコード
```php
class UserName
{
    public function __construct(private readonly string $value)
    {
        if (mb_strlen($value) < 3) {
            throw new InvalidArgumentException("名前は3文字以上です");
        }
    }

    public function value(): string
    {
        return $this->value;
    }
}
```

- コードを饒舌にする努力を怠らなければ、開発者はコードを手掛かりにしてそこに存在するルールを確認できる。
	- 本来はドメインモデルをつくり、これからドメインオブジェクトとして実装する
	- P61の図を書く
- 無口なコードだと、そこにあるルールがすべて守られているかはすべてのコードを洗い出し調べる必要がある。これはバグのもとになる。


####  ドメインにおける変更をコードに伝えやすくする
- ユーザー名の最少を3文字→6文字にしたいという依頼がくる
	- Userクラスの↑のリストのようなただの構造体であると、ロジックがちらばっていて変更箇所を抜き出すのは至難のわざ
	- コードを饒舌にする努力を怠るな！

### ドメインサービス
#### サービスとは
- クライアントのために何かを行うオブジェクト
	- ドメインのためのサービスとアプリケーションのためのサービスがある。
		- 前者をドメインサービス、後者をアプリケーションサービスと呼ぶ
        
#### ドメインサービスとは
- 値オブジェクトやエンティティに記述すると不自然なものはドメインサービスに書く。

#### 不自然なふるまいとは
- 例
```php
class User
{
    public function __construct(private readonly UserId $id, private string $name)
    {
    }

	(省略)

    public function exists(User $user): bool
    {
        //重複確認処理
        return true;
    }
}
$userId = new UserId(1);
$userName = new UserName("Taro");
$user = new User($userId, $userName);

$duplicateCheckResult = $user->exists($user);//自身に重複の問い合わせ？
```

- 重複確認のメソッドはExistsにあるので、自身に対して重複を問い合わせるのは不自然。これはメソッドではtrueを返すかfalseを返すのが正解かわかりにくい。
- このことから、この重複判定メソッドをUserエンティティに記述するのは不自然なふるまいっぽい

#### 不自然さを解決するオブジェクト
ドメインサービス

```php
class UserService
{
	(省略)

    public function exists(User $user): bool
    {
        //重複確認処理
    }
}

$userService = new UserService();

$userId = new UserId(1);
$userName = new UserName("Taro");
$user = new User($userId, $userName);

$duplicateCheckResult = $userService->exists($user);
```

#### ドメインサービスの濫用に注意
- 大事なのは不自然なふるまいに限定すること

```php
class User
{
    public function __construct(private readonly UserId $id, private string $name)
    {
    }

    public function name(): string
    {
        return $this->name;
    }
```

- サービスになんでも定義してしまうと、結局エンティティにはゲッターとセッターしかのこらない。
- これだけでは、どんな熟練の開発者でもこのUserクラスからどんなふるまいやルールが存在するのはを知ることは不可能。
- この状況をドメインモデル貧血症に落ちいっているという
- **まずエンティティや値オブジェクトに定義できないか考えて、それが不自然なときにドメインサービスの利用はとどめる。**極力利用しないようにする。

#### エンティティ、値オブジェクトとユースケースの組み立て
- ユースケースというのはいわばクライアントがどんな動作をするかということ。

上記をつかって例えばユーザーが登録をするという処理を考える
```php
class Program
{
    // ユーザー登録
    public function createUser(UserName $userName): void
    {
        $userService = new UserService();

        $userId = new UserId(1);
        $userName = new UserName("Taro");
        $user = new User($userId, $userName);

        if ($userService->exists($user)) {
            throw new Exception("ユーザーは既に存在しています");
        };

        // ユーザー登録処理
        try {
            $dbHost = getenv('DB_HOST');
            $dbName = getenv('DB_NAME');
            $dbUser = getenv('DB_USER');
            $dbPassword = getenv('DB_PASSWORD');

            $dsn = "mysql:host=$dbHost;dbname=$dbName;charset=utf8";
            $pdo = new PDO($dsn, $dbUser, $dbPassword);
            $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

            // ユーザー登録用のSQLクエリ
            $sql = "INSERT INTO users (username) VALUES (:username)";
            $stmt = $pdo->prepare($sql);

            $stmt->bindParam(':username', $username, PDO::PARAM_STR);

            $stmt->execute();
        } catch (PDOException $e) {
            // エラー処理
            echo "Error: " . $e->getMessage();
        }
    }
}
```

- ユーザーをつくり重複チェックしているのはわかるが、その後は何？？
	- 処理をちゃんと読めばDBに繋いで、SQLを発行しユーザーの保存を行っているのがわかる

- ユーザーサービスの実装
```php
class UserService
{
    public function exists($userName): bool
    {
        try {
            // 環境変数からデータベース接続情報を取得
            $dbHost = getenv('DB_HOST');
            $dbName = getenv('DB_NAME');
            $dbUser = getenv('DB_USER');
            $dbPassword = getenv('DB_PASSWORD');

            // データベースへの接続
            $dsn = "mysql:host=$dbHost;dbname=$dbName;charset=utf8";
            $pdo = new PDO($dsn, $dbUser, $dbPassword);
            $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

            // ユーザーが存在するかどうかを確認するSQLクエリ
            $sql = "SELECT * FROM users WHERE username = :username";
            $stmt = $pdo->prepare($sql);
            $stmt->bindParam(':username', $userName, PDO::PARAM_STR);
            $stmt->execute();

            $result = $stmt->fetch(PDO::FETCH_ASSOC);

            return $result !== false;
        } catch (PDOException $e) {
            echo "Error: " . $e->getMessage();
            return false;
        }
    }
}
```

- ユーザー名の重複を確認するのにもDBへ問い合わせが必要。
	- UserServiceはほぼほぼDB操作に終始してしまっている。
	- どちらもちゃんと動くが、柔軟性に乏しい。リレーショナルデータストアがNoSQLデータへ変更するとなったら、ユーザー作成処理の本質は変わっていなくてもコードを変更する必要がでてくる。
- データを扱う以上データの保存や読み取りは避けられない。
	- ユーザー作成処理が特定のデータストアや、操作処理について関心も持つべきではない。
	- これは後述するリポジトリというもので行う。
    
    

## アプリケーションを実現するためのパターン
### リポジトリ
### アプリケーションサービス
### ファクトリ
## 知識を表現する、より発展的なパターン
### 集約

### 仕様

