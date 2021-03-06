ビュー
======

ビューは [MVC](http://ja.wikipedia.org/wiki/Model_View_Controller) アーキテクチャの一部を成すものです。
ビューはエンドユーザにデータを表示することに責任を持つコードです。
ウェブアプリケーションにおいては、ビューは、通常、主として HTML コードと表示目的の PHP コードを含む PHP スクリプトファイルである、
*ビューテンプレート* の形式で作成されます。
そして、ビューテンプレートを管理する [[yii\web\View|ビュー]] [アプリケーションコンポーネント](structure-application-components.md) は、
ビューの構築とレンダリングを助けるためによく使われるメソッドを提供します。
なお、簡潔さを重視して、ビューテンプレートまたはビューテンプレートファイルを単にビューと呼ぶことがよくあります。


## ビューを作成する <a name="creating-views"></a>

前述のように、ビューは HTML と PHP コードが混ざった単なる PHP スクリプトです。
次に示すのは、ログインフォームを表示するビューです。
見ると分るように、PHP コードがタイトルやフォームなど動的なコンテンツを生成するのに使われ、HTML コードがそれらを編成して表示可能な HTML ページを作っています。

```php
<?php
use yii\helpers\Html;
use yii\widgets\ActiveForm;

/* @var $this yii\web\View */
/* @var $form yii\widgets\ActiveForm */
/* @var $model app\models\LoginForm */

$this->title = 'ログイン';
?>
<h1><?= Html::encode($this->title) ?></h1>

<p>次の項目を入力してログインしてください:</p>

<?php $form = ActiveForm::begin(); ?>
    <?= $form->field($model, 'username') ?>
    <?= $form->field($model, 'password')->passwordInput() ?>
    <?= Html::submitButton('ログイン') ?>
<?php ActiveForm::end(); ?>
```

ビューの中でアクセスできる `$this` は、このビューテンプレートを管理しレンダリングしている
[[yii\web\View|ビューコンポーネント]] を参照します。

`$this` 以外に、上記の例の `$model` のように、前もって定義された変数がビューの中にあることがあります。
このような変数は、[コントローラ](structure-controllers.md) または [ビューのレンダリング](#rendering-views) をトリガするオブジェクトによってビューに *プッシュ* されたデータを表します。

> Tip|ヒント: 上の例では、事前に定義された変数は、IDE に認識されるように、
  ビューの先頭のコメントブロックの中にリストされています。これは、ビューに
  ドキュメントを付けるのにも良い方法です。


### セキュリティ <a name="security"></a>

HTML ページを生成するビューを作成するときは、エンドユーザから受け取るデータを
表示する前に エンコード および/または フィルター することが重要です。
そうしなければ、あなたのアプリケーションは [クロスサイトスクリプティング](http://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AD%E3%82%B9%E3%82%B5%E3%82%A4%E3%83%88%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%97%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0) 攻撃をこうむるおそれがあります。

平文テキストを表示するためには、まず [[yii\helpers\Html::encode()]] を呼んでエンコードします。
例えば、次のコードはユーザの名前を表示する前にエンコードしています:

```php
<?php
use yii\helpers\Html;
?>

<div class="username">
    <?= Html::encode($user->name) ?>
</div>
```

HTML コンテンツを表示するためには、[[yii\helpers\HtmlPurifier]] を使って、最初にコンテンツをフィルターします。
例えば、次のコードは、投稿のコンテンツを表示する前にフィルターしています:

```php
<?php
use yii\helpers\HtmlPurifier;
?>

<div class="post">
    <?= HtmlPurifier::process($post->text) ?>
</div>
```

> Tip|ヒント: HTMLPurifier は、出力を安全なものにすることにおいては素晴らしい仕事をしますが、
  速くはありません。アプリケーションが高いパフォーマンスを要求する場合は、
  フィルター結果を [キャッシュ](caching-overview.md) することを考慮すべきです。


### ビューを整理する <a name="organizing-views"></a>

[コントローラ](structure-controllers.md) や [モデル](structure-models.md) と同じように、
ビューを整理するための規約があります。.

* コントローラによって表示されるビューは、既定では、ディレクトリ
  `@app/views/ControllerID` の下に置かれるべきものです。
  ここで、`ControllerID` は [コントローラ ID](structure-controllers.md#routes) を指します。
  例えば、コントローラクラスが `PostController` である場合、ディレクトリは `@app/views/post`
  となります。`PostCommentController` の場合は、ディレクトリは `@app/views/post-comment` です。
  また、コントローラがモジュールに属する場合は、ディレクトリは [[yii\base\Module::basePath|モジュールディレクトリ]]
  の下の `views/ControllerID` です。
* [ウィジェット](structure-widgets.md) で表示されるビューは、既定では、`WidgetPath/views`
  ディレクトリの下に置かれるべきものです。ここで、`WidgetPath` は、ウィジェットのクラスファイル
  を含んでいるディレクトリを指します。
* 他のオブジェクトによって表示されるビューについても、ウィジェットの場合と同じ規約に従うことが
  推奨されます。

これらの既定のビューディレクトリは、コントローラやウィジェットの [[yii\base\ViewContextInterface::getViewPath()]]
メソッドをオーバーライドすることでカスタマイズすることが可能です。


## ビューをレンダリングする <a name="rendering-views"></a>

[コントローラ](structure-controllers.md) の中でも、[ウィジェット](structure-widgets.md) の中でも、
または、その他のどんな場所でも、ビューをレンダリングするメソッドを呼ぶことによって
ビューをレンダリングすることが出来ます。
これらのメソッドは、下記に示されるような類似のシグニチャを共有します。

```
/**
 * @param string $view ビュー名またはファイルパス、実際のレンダリングメソッドに依存する
 * @param array $params ビューに引き渡されるデータ
 * @return string レンダリングの結果
 */
methodName($view, $params = [])
```


### コントローラでのレンダリング <a name="rendering-in-controllers"></a>

[コントローラ](structure-controllers.md) の中では、ビューをレンダリングするために
次のコントローラメソッドを呼ぶことが出来ます:

* [[yii\base\Controller::render()|render()]]: [名前付きビュー](#named-views) をレンダリングし、
  その結果に [レイアウト](#layouts) を適用する。
* [[yii\base\Controller::renderPartial()|renderPartial()]]: [名前付きビュー](#named-views) を
  レイアウトなしでレンダリングする。
* [[yii\web\Controller::renderAjax()|renderAjax()]]: [名前付きビュー](#named-views) を
  レイアウトなしでレンダリングし、登録されている全ての JS/CSS スクリプトおよびファイルを注入する。
  通常、AJAX ウェブリクエストに対するレスポンスにおいて使用される。
* [[yii\base\Controller::renderFile()|renderFile()]]: ビューファイルのパスまたは [エイリアス](concept-aliases.md)
  の形式で指定されたビューをレンダリングする。
* [[yii\base\Controller::renderContent()|renderContent()]]: 現在適用可能な [レイアウト](#layouts) に埋め込むことによって、
  静的な文字列をレンダリングする。このメソッドは バージョン 2.0.1 以降で使用可能。

例えば、

```php
namespace app\controllers;

use Yii;
use app\models\Post;
use yii\web\Controller;
use yii\web\NotFoundHttpException;

class PostController extends Controller
{
    public function actionView($id)
    {
        $model = Post::findOne($id);
        if ($model === null) {
            throw new NotFoundHttpException;
        }

        // "view" という名前のビューをレンダリングし、レイアウトを適用する
        return $this->render('view', [
            'model' => $model,
        ]);
    }
}
```


### ウィジェットでのレンダリング <a name="rendering-in-widgets"></a>

[ウィジェット](structure-widgets.md) の中では、ビューをレンダリングするために、
次のウィジェットメソッドを使用することが出来ます。

* [[yii\base\Widget::render()|render()]]: [名前付きのビュー](#named-views) をレンダリングする。
* [[yii\base\Widget::renderFile()|renderFile()]]: ビューファイルのパスまたは [エイリアス](concept-aliases.md)
  の形式で指定されたビューをレンダリングする。

例えば、

```php
namespace app\components;

use yii\base\Widget;
use yii\helpers\Html;

class ListWidget extends Widget
{
    public $items = [];

    public function run()
    {
        // "list" という名前のビューをレンダリングする
        return $this->render('list', [
            'items' => $this->items,
        ]);
    }
}
```


### ビューでのレンダリング <a name="rendering-in-views"></a>

[[yii\base\View|ビューコンポーネント]] によって提供される下記のメソッドのどれかを使うと、
ビューの中で、別のビューをレンダリングすることが出来ます:

* [[yii\base\View::render()|render()]]: [名前付きのビュー](#named-views) をレンダリングする。
* [[yii\web\View::renderAjax()|renderAjax()]]: [名前付きビュー](#named-views) をレンダリングし、
  登録されている全ての JS/CSS スクリプトおよびファイルを注入する。
  通常、AJAX ウェブリクエストに対するレスポンスにおいて使用される。
* [[yii\base\View::renderFile()|renderFile()]]: ビューファイルのパスまたは [エイリアス](concept-aliases.md)
  の形式で指定されたビューをレンダリングする。

例えば、ビューの中の次のコードは、現在レンダリングされているビューと同じディレクトリにある
`_overview.php` というビューファイルをレンダリングします。
ビューでは `$this` が [[yii\base\View|ビュー]] コンポーネントを参照することを思い出してください:

```php
<?= $this->render('_overview') ?>
```


### 他の場所でのレンダリング <a name="rendering-in-other-places"></a>

場所がどこであれ、`Yii::$app->view` という式によって [[yii\base\View|ビュー]] アプリケーションコンポーネントにアクセスすることが出来ますから、
前述の [[yii\base\View|ビュー]] コンポーネントメソッドを使ってビューをレンダリングすることが出来ます。
例えば、

```php
// ビューファイル "@app/views/site/license.php" を表示
echo \Yii::$app->view->renderFile('@app/views/site/license.php');
```


### 名前付きビュー <a name="named-views"></a>

ビューをレンダリングするとき、ビューを指定するのには、ビューの名前か、
ビューファイルのパス/エイリアスか、どちらかを使うことが出来ます。
たいていの場合は、より簡潔で柔軟な前者を使います。
名前を使って指定されるビューを *名前付きビュー* と呼びます。

ビューの名前は、以下の規則に従って、対応するビューファイルのパスに解決されます。

* ビュー名はファイル拡張子を省略することが出来ます。その場合、`.php` が拡張子として使われます。
  例えば、`about` というビュー名は `about.php` というファイル名に対応します。
* ビュー名が二つのスラッシュ (`//`) で始まる場合は、対応するビューファイルのパスは `@app/views/ViewName`
  となります。つまり、ビューファイルは [[yii\base\Application::viewPath|アプリケーションのビューパス]]
  の下で探されます。例えば、`//site/about` は `@app/views/site/about.php` へと解決されます。
* ビュー名が一つのスラッシュ (`/`) で始まる場合は、ビューファイルのパスは、ビュー名の前に、現在
  アクティブな [モジュール](structure-modules.md) の [[yii\base\Module::viewPath|ビューパス]]
  を置くことによって形成されます。アクティブなモジュールが無い場合は、`@app/views/ViewName`
  が使用されます。例えば、`/user/create` は、現在アクティブなモジュールが `user` である場合は、
  `@app/modules/user/views/user/create.php` へと解決されます。アクティブなモジュールが無い場合は、
  ビューファイルのパスは `@app/views/user/create.php` となります。
* ビューが [[yii\base\View::context|コンテキスト]] を伴ってレンダリングされ、そのコンテキストが
  [[yii\base\ViewContextInterface]] を実装している場合は、ビューファイルのパスは、コンテキストの
  [[yii\base\ViewContextInterface::getViewPath()|ビューパス]] をビュー名の前に置くことによって
  形成されます。これは、主として、コントローラとウィジェットの中でレンダリングされるビューに当てはまります。
  例えば、コンテキストが `SiteController` コントローラである場合、`site/about` は `@app/views/site/about.php`
  へと解決されます。
* あるビューが別のビューの中でレンダリングされる場合は、後者のビューファイルを含んでいるディレクトリが
  前者のビュー名の前に置かれて、実際のビューファイルのパスが形成されます。例えば、`item` は、
  `@app/views/post/index.php` というビューの中でレンダリングされる場合、`@app/views/post/item`
  へと解決されます。

上記の規則によると、コントローラ `app\controllers\PostController` の中で `$this->render('view')` を呼ぶと、
実際には、ビューファイル `@app/views/post/view.php` がレンダリングされ、一方、そのビューの中で
`$this->render('_overview')` を呼ぶと、ビューファイル `@app/views/post/_overview.php`
がレンダリングされることになります。


### ビューの中でデータにアクセスする <a name="accessing-data-in-views"></a>

ビューの中でデータにアクセスするためのアプローチが二つあります: 「プッシュ」と「プル」です。

ビューをレンダリングするメソッドに二番目のパラメータとしてデータを渡すのが「プッシュ」のアプローチです。
データは、「名前-値」のペアの配列として表されなければなりません。
ビューがレンダリングされるときに、PHP の `extract()` 関数がこの配列に対して呼び出され、
ビューの中でこの配列から変数が抽出されます。
例えば、次のコードはコントローラの中でビューをレンダリングしていますが、`report` ビューに
二つの変数、すなわち、`$foo = 1` と `$bar = 2` をプッシュしています。

```php
echo $this->render('report', [
    'foo' => 1,
    'bar' => 2,
]);
```

「プル」のアプローチは、[[yii\base\View|ビューコンポーネント]] またはビューからアクセス出来るその他のオブジェクト (例えば `Yii::$app`) から
積極的にデータを読み出すものです。
下記のコードを例として使って、ビューの中で `$this->context` という式でコントローラオブジェクト
を取得することが出来ます。その結果、`report` ビューの中でコントローラの全てのプロパティや
メソッドにアクセスすることが出来ます。次の例ではコントローラ ID にアクセスしています:

```php
The controller ID is: <?= $this->context->id ?>
?>
```

通常は「プッシュ」アプローチが、ビューでデータにアクセスする方法として推奨されます。
なぜなら、ビューのコンテキストオブジェクトに対する依存がより少ないからです。
その短所は、常にデータ配列を手作業で作成する必要がある、ということです。
ビューが共有されてさまざまな場所でレンダリングされる場合、その作業が面倒くさくなり、また、
間違いも生じやすくなります。


### ビューの間でデータを共有する <a name="sharing-data-among-views"></a>

[[yii\base\View|ビューコンポーネント]] が提供する [[yii\base\View::params|params]] プロパティを使うと
ビューの間でデータを共有することが出来ます。

例えば、`about` というビューで、次のようなコードを使って、
パン屑リストの現在の区分を指定することが出来ます。

```php
$this->params['breadcrumbs'][] = 'About Us';
```

そして、[レイアウト](#layouts) ファイル (これも一つのビューです) の中で、[[yii\base\View::params|params]]
によって渡されたデータを使って、パン屑リストを表示することが出来ます:

```php
<?= yii\widgets\Breadcrumbs::widget([
    'links' => isset($this->params['breadcrumbs']) ? $this->params['breadcrumbs'] : [],
]) ?>
```


## レイアウト <a name="layouts"></a>

レイアウトは、複数のビューの共通部分をあらわす特殊なタイプのビューです。
例えば、たいていのウェブアプリケーションでは、ページは共通のヘッダとフッタを持っています。
すべてのビューで同じヘッダとフッタを繰り返すことも出来ますが、もっと良い方法は、
そういうことはレイアウトの中で一度だけして、コンテンツビューのレンダリング結果を
レイアウトの中の適切な場所に埋め込むことです。


### レイアウトを作成する <a name="creating-layouts"></a>

レイアウトもまたビューですので、通常のビューと同様な方法で作成することが出来ます。既定では、
レイアウトは `@app/views/layouts` ディレクトリに保存されます。[モジュール](structure-modules.md)
の中で使用されるレイアウトについては、[[yii\base\Module::basePath|モジュールディレクトリ]] の下の
`views/layouts` ディレクトリに保存されるべきものとなります。既定のレイアウトディレクトリは、
アプリケーションまたはモジュールの [[yii\base\Module::layoutPath]] プロパティを構成することで
カスタマイズすることが出来ます。

次の例は、レイアウトがどのようなものであるかを示すものです。説明のために、レイアウトの中のコードを
大幅に単純化していることに注意してください。実際には、ヘッドのタグやメインメニューなど、もっと
多くのコンテンツを追加する必要があるでしょう。

```php
<?php
use yii\helpers\Html;

/* @var $this yii\web\View */
/* @var $content string */
?>
<?php $this->beginPage() ?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <?= Html::csrfMetaTags() ?>
    <title><?= Html::encode($this->title) ?></title>
    <?php $this->head() ?>
</head>
<body>
<?php $this->beginBody() ?>
    <header>My Company</header>
    <?= $content ?>
    <footer>&copy; 2014 by My Company</footer>
<?php $this->endBody() ?>
</body>
</html>
<?php $this->endPage() ?>
```

見ると分かるように、レイアウトはすべてのページに共通な HTML タグを生成しています。`<body>`
セクションの中でレイアウトが `$content` という変数をエコーしていますが、これは、
コンテンツビューのレンダリング結果を表すものであり、[[yii\base\Controller::render()]] が呼ばれるときに、レイアウトにプッシュされるものです。

上記のコードに示されているように、たいていのレイアウトは次に挙げるメソッドを呼び出すべきです。
これらのメソッドは主としてレンダリングの過程に関するイベントをトリガして、他の場所で登録された
スクリプトやタグが、メソッドが呼ばれた場所に正しく注入されるようにするためのものです。

- [[yii\base\View::beginPage()|beginPage()]]: このメソッドがレイアウトの一番初めに呼ばれるべきです。
  これは、ページの開始を示す [[yii\base\View::EVENT_BEGIN_PAGE|EVENT_BEGIN_PAGE]] イベントをトリガします。
- [[yii\base\View::endPage()|endPage()]]: このメソッドがレイアウトの最後で呼ばれるべきです。
  これは、ページの終了を示す [[yii\base\View::EVENT_END_PAGE|EVENT_END_PAGE]] イベントをトリガします。
- [[yii\web\View::head()|head()]]: このメソッドが HTML ページの `<head>` セクションの中で呼ばれるべきです。
  このメソッドは、ページのレンダリングが完了したときに、登録された head の HTML コード (リンクタグ、メタタグなど) に置き換えられるプレースホルダを生成します。
- [[yii\web\View::beginBody()|beginBody()]]: このメソッドが `<body>` セクションの最初で呼ばれるべきです。
  このメソッドは [[yii\web\View::EVENT_BEGIN_BODY|EVENT_BEGIN_BODY]] イベントをトリガし、
  body の開始位置を目的とする登録された HTML コード (JavaScript など) によって置き換えられる
  プレースホルダを生成します。
- [[yii\web\View::endBody()|endBody()]]: このメソッドが `<body`> セクションの最後で呼ばれるべきです。
  このメソッドは  [[yii\web\View::EVENT_END_BODY|EVENT_END_BODY]] イベントをトリガし、
  body の終了位置を目的とする登録された HTML コード (JavaScript など) によって置き換えられる
  プレースホルダを生成します。


### レイアウトでデータにアクセスする <a name="accessing-data-in-layouts"></a>

レイアウトの中では、事前定義された二つの変数にアクセス出来ます: `$this` と `$content` です。前者は、
通常のビューにおいてと同じく、[[yii\base\View|ビュー]] コンポーネントを参照します。一方、後者は、
コントローラの中で [[yii\base\Controller::render()|render()]] メソッドを呼ぶことによってレンダリングされる、
コンテンツビューのレンダリング結果を含むものです。

レイアウトの中で他のデータにアクセスする必要があるときは、[ビューの中でデータにアクセスする](#accessing-data-in-views)
の項で説明されている「プル」の方法を使う必要があります。コンテンツビューからレイアウトにデータを渡す必要があるときは、
[ビューの間でデータを共有する](#sharing-data-among-views) の項で説明されている方法を使うことが出来ます。


### レイアウトを使う <a name="using-layouts"></a>

[コントローラでのレンダリング](#rendering-in-controllers) の項で説明されているように、コントローラの中で
[[yii\base\Controller::render()|render()]] メソッドを呼んでビューをレンダリングすると、レンダリング結果に
レイアウトが適用されます。既定では、`@app/views/layouts/main.php` というレイアウトが使用されます。

[[yii\base\Application::layout]] または [[yii\base\Controller::layout]] のどちらかを構成することによって、異なるレイアウトを
使うことが出来ます。前者は全てのコントローラによって使用されるレイアウトを決定するものですが、後者は個々のコントローラについて
前者をオーバーライドするものです。例えば、次のコードは、`post` コントローラがビューをレンダリングするときに
`@app/views/layouts/post.php` をレイアウトとして使うようにするものです。その他のコントローラは、`layout` プロパティに
触れられていないと仮定すると、引き続き既定の `@app/views/layouts/main.php` をレイアウトとして使います。
 
```php
namespace app\controllers;

use yii\web\Controller;

class PostController extends Controller
{
    public $layout = 'post';
    
    // ...
}
```

モジュールに属するコントローラについては、モジュールの [[yii\base\Module::layout|layout]] プロパティを構成して、モジュール内の
コントローラに特定のレイアウトを使用することも出来ます。

`layout` プロパティは異なるレベル (コントローラ、モジュール、アプリケーション) で構成されうるものですので、
Yii は舞台裏で二つのステップを践んで、特定のコントローラで実際に使われるレイアウトファイルが何であるかを決定します。

最初のステップで、Yii はレイアウトの値とコンテキストモジュールを決定します:

- コントローラの [[yii\base\Controller::layout]] プロパティが null でないときは、それをレイアウトの値として使い、
  コントローラの [[yii\base\Controller::module|モジュール]] をコンテキストモジュールとして使う。
- [[yii\base\Controller::layout|layout]] が null のときは、コントローラの祖先となっている全てのモジュール
  (アプリケーション自体も含む) を探して、[[yii\base\Module::layout|layout]] プロパティが null でない最初のモジュールを見つける。
  見つかったモジュールとその [[yii\base\Module::layout|layout]] の値をコンテキストモジュールと選ばれたレイアウトの値とする。
  そのようなモジュールが見つからなかったときは、レイアウトは適用されないということを意味する。

第二のステップでは、最初のステップで決定されたレイアウトの値とコンテキストモジュールに従って、実際のレイアウトファイルを決定します。
レイアウトの値は下記のいずれかであり得ます:

- パスエイリアス (例えば、`@app/views/layouts/main`)。
- 絶対パス (例えば、`/main`): すなわち、スラッシュで始まるレイアウトの値の場合。
  実際のレイアウトファイルはアプリケーションの [[yii\base\Application::layoutPath|レイアウトパス]]
  (デフォルトでは `@app/views/layouts`) の下で探される。
- 相対パス (例えば、`main`): 実際のレイアウトファイルはコンテキストモジュールの [[yii\base\Module::layoutPath|レイアウトパス]]
  (デフォルトでは [[yii\base\Module::basePath|モジュールディレクトリ]] の下の `views/layouts` ディレクトリ) の下で探される。
- 真偽値 `false`: レイアウトは適用されない。

レイアウトの値がファイル拡張子を含んでいない場合は、デフォルト値である `.php` を使います。


### 入れ子のレイアウト <a name="nested-layouts"></a>

ときとして、あるレイアウトの中に別のレイアウトを入れたい場合があるでしょう。例えば、
ウェブサイトの別々のセクションにおいて、違うレイアウトを使いたいけれども、
それらのレイアウトは全て、全体としての HTML5 ページ構造を生成する同一の基本レイアウトを
共有している、という場合です。この目的を達することは、次のように、子レイアウトの中で
[[yii\base\View::beginContent()|beginContent()]] と [[yii\base\View::endContent()|endContent()]]
を呼ぶことで可能になります:

```php
<?php $this->beginContent('@app/views/layouts/base.php'); ?>

... 子レイアウトのコンテンツをここに ...

<?php $this->endContent(); ?>
```

上のコードが示すように、子レイアウトのコンテンツは [[yii\base\View::beginContent()|beginContent()]] と
[[yii\base\View::endContent()|endContent()]] によって囲まれなければなりません。
[[yii\base\View::beginContent()|beginContent()]] に渡されるパラメータは、
親レイアウトで何であるかを指定するものです。レイアウトのファイルまたはエイリアスのどちらかを使うことが出来ます。

上記のアプローチを使って、2レベル以上のレイアウトを入れ子にすることも出来ます。


### ブロックを使う <a name="using-blocks"></a>

ブロックを使うと、ある場所でビューコンテンツを規定して、別の場所でそれを表示することが可能になります。
ブロックはたいていはレイアウトと一緒に使われます。例えば、ブロックをコンテンツビューで定義して、
それをレイアウトで表示する、ということが出来ます。

[[yii\base\View::beginBlock()|beginBlock()]] と [[yii\base\View::endBlock()|endBlock()]]
を呼んでブロックを定義します。
すると、そのブロックを `$view->blocks[$blockID]` によってアクセス出来るようになります。
ここで `$blockID` は、定義したときにブロックに割り当てたユニークな ID を指します。

次の例は、どのようにブロックを使えば、レイアウトの特定の部分をコンテンツビューで
カスタマイズすることが出来るかを示すものです。

最初に、コンテンツビューで、一つまたは複数のブロックを定義します。

```php
...

<?php $this->beginBlock('block1'); ?>

... block1 のコンテンツ ...

<?php $this->endBlock(); ?>

...

<?php $this->beginBlock('block3'); ?>

... block3 のコンテンツ ...

<?php $this->endBlock(); ?>
```

次に、レイアウトビューで、得ることが出来ればブロックをレンダリングし、ブロックが定義されていないときは
何らかの既定のコンテンツを表示します。

```php
...
<?php if (isset($this->blocks['block1'])): ?>
    <?= $this->blocks['block1'] ?>
<?php else: ?>
    ... block1 の既定のコンテンツ ...
<?php endif; ?>

...

<?php if (isset($this->blocks['block2'])): ?>
    <?= $this->blocks['block2'] ?>
<?php else: ?>
    ... block2 の既定のコンテンツ ...
<?php endif; ?>

...

<?php if (isset($this->blocks['block3'])): ?>
    <?= $this->blocks['block3'] ?>
<?php else: ?>
    ... block3 の既定のコンテンツ ...
<?php endif; ?>
...
```


## ビューコンポーネントを使う <a name="using-view-components"></a>

[[yii\base\View|ビューコンポーネント]] はビューに関連する多くの機能を提供します。
ビューコンポーネントは、[[yii\base\View]] またはその子クラスの個別のインスタンスを作成することによっても取得できますが、
たいていの場合は、`view` アプリケーションコンポーネントを主として使うことになるでしょう。
このコンポーネントは [アプリケーションのコンフィギュレーション](structure-applications.md#application-configurations) の中で、次のようにして構成することが出来ます:

```php
[
    // ...
    'components' => [
        'view' => [
            'class' => 'app\components\View',
        ],
        // ...
    ],
]
```

ビューコンポーネントは、次に挙げるビュー関連の有用な機能を提供します。それぞれについては、
独立の節で更に詳細に説明されます。

* [テーマ](output-theming.md): ウェブサイトのテーマを開発し変更することを可能にします。
* [フラグメントキャッシュ](caching-fragment.md): ウェブページの中の断片をキャッシュすることを可能にします。
* [クライアントスクリプトの取り扱い](output-client-scripts.md): CSS と JavaScript の登録とレンダリングをサポートします。
* [アセットバンドルの取り扱い](structure-assets.md): [アセットバンドル](structure-assets.md) の登録とレンダリングをサポートします。
* [代替のテンプレートエンジン](tutorial-template-engines.md): [Twig](http://twig.sensiolabs.org/)、[Smarty](http://www.smarty.net/) など、他のテンプレートエンジンを使用することを可能にします。

次に挙げるマイナーではあっても有用な諸機能は、ウェブページを開発するときに頻繁に使用するでしょう:


### ページタイトルを設定する <a name="setting-page-titles"></a>

どんなウェブページにもタイトルが無ければなりません。通常、タイトルタグは [layout](#layouts) の中で表示されます。しかし、
実際においては、多くの場合、タイトルはレイアウトではなくコンテンツビューで決められます。この問題を解決するために、
[[yii\web\View]] は、タイトル情報をコンテンツビューからレイアウトに渡すための [[yii\web\View::title|title]] プロパティを
提供しています。

この機能を利用するためには、全てのコンテンツビューにおいて、次のようにタイトルを設定します:

```php
<?php
$this->title = 'My page title';
?>
```

そして、レイアウトビューで、`<head>` セクションに次のコードを忘れずに書くようにします:

```php
<title><?= Html::encode($this->title) ?></title>
```


### メタタグを登録する <a name="registering-meta-tags"></a>

ウェブページは、通常、いろいろな関係者によって必要とされるさまざまなメタタグを生成する必要があります。ページタイトルと同じように、
メタタグは `<head>` セクションに出現して、通常はレイアウトの中で生成されます。

どのようなメタタグを生成するかをコンテンツビューの中で指定したい場合は、下記のように、
[[yii\web\View::registerMetaTag()]] をコンテンツビューの呼ぶことが出来ます:

```php
<?php
$this->registerMetaTag(['name' => 'keywords', 'content' => 'yii, framework, php']);
?>
```

上記のコードは、ビューコンポーネントによって "keywords" メタタグを登録するものです。登録されたメタタグは、
レイアウトがレンダリングを完了した後でレンダリングされます。すなわち、レイアウトの中で [[yii\web\View::head()]]
を呼び出した場所に、次の HTML コードが生成されて挿入されます:

```php
<meta name="keywords" content="yii, framework, php">
```

[[yii\web\View::registerMetaTag()]] を複数回呼び出した場合は、メタタグが同じものか否かに関係なく、
複数のメタタグが登録されることに注意してください。

ある型のメタタグのインスタンスが一つだけになることを保証したい場合は、このメソッドを呼ぶときに第二のパラメータとして
キーを指定することが出来ます。例えば、次のコードでは、二つの "description" メタタグを登録していますが、
二番目のものだけがレンダリングされることになります。

```html
$this->registerMetaTag(['name' => 'description', 'content' => '俺が Yii で作ったクールなウェブサイトだぜぃ!!'], 'description');
$this->registerMetaTag(['name' => 'description', 'content' => '面白いアライグマに関するウェブサイトです。'], 'description');
```


### リンクタグを登録する <a name="registering-link-tags"></a>

[メタタグ](#registering-meta-tags) と同じように、リンクタグも多くの場合において有用なものです。例えば、favicon をカスタマイズしたり、
RSS フィードを指し示したり、OpenID を別のサーバに委任したり、等々。リンクタグも、[[yii\web\View::registerLinkTag()]] を使って、
メタタグと同じような方法で取り扱うことが出来ます。例えば、コンテンツビューにおいて、次のようにしてリンクタグを登録することが出来ます。

```php
$this->registerLinkTag([
    'title' => 'Yii ライブニューズ',
    'rel' => 'alternate',
    'type' => 'application/rss+xml',
    'href' => 'http://www.yiiframework.com/rss.xml/',
]);
```

上記のコードは、次の結果になります。

```html
<link title="Yii ライブニューズ" rel="alternate" type="application/rss+xml" href="http://www.yiiframework.com/rss.xml/">
```

[[yii\web\View::registerMetaTag()|registerMetaTags()]] と同じように、[[yii\web\View::registerLinkTag()|registerLinkTag()]]
を呼ぶときにキーを指定すると、同じリンクタグを繰り返して生成するのを避けることが出来ます。


## ビューのイベント <a name="view-events"></a>

[[yii\base\View|ビューコンポーネント]] はビューをレンダリングする過程においていくつかのイベントをトリガします。
これらのイベントに反応することによって、ビューにコンテンツを注入したり、
エンドユーザに送信される前にレンダリング結果を加工したりすることが出来ます。

- [[yii\base\View::EVENT_BEFORE_RENDER|EVENT_BEFORE_RENDER]]: コントローラでファイルをレンダリングする前にトリガされます。
  このイベントのハンドラは、[[yii\base\ViewEvent::isValid]] を false にセットして、レンダリングのプロセスをキャンセルすることが出来ます。
- [[yii\base\View::EVENT_AFTER_RENDER|EVENT_AFTER_RENDER]]: ファイルのレンダリングの後、[[yii\base\View::afterRender()]] を呼ぶことによってトリガされます。
  このイベントのハンドラは、レンダリング結果を [[yii\base\ViewEvent::output]] によって取得することが出来、
  このプロパティを修正してレンダリング結果を変更することが出来ます。
- [[yii\base\View::EVENT_BEGIN_PAGE|EVENT_BEGIN_PAGE]]: レイアウトの中で [[yii\base\View::beginPage()]] を呼ぶことによってトリガされます。
- [[yii\base\View::EVENT_END_PAGE|EVENT_END_PAGE]]: レイアウトの中で [[yii\base\View::endPage()]] を呼ぶことによってトリガされます。
- [[yii\web\View::EVENT_BEGIN_BODY|EVENT_BEGIN_BODY]]: レイアウトの中で [[yii\web\View::beginBody()]] を呼ぶことによってトリガされます。
- [[yii\web\View::EVENT_END_BODY|EVENT_END_BODY]]: レイアウトの中で [[yii\web\View::endBody()]] を呼ぶことによってトリガされます。

例えば、次のコードはページの body の最後に現在の日付を注入するものです:

```php
\Yii::$app->view->on(View::EVENT_END_BODY, function () {
    echo date('Y-m-d');
});
```


## 静的なページをレンダリングする <a name="rendering-static-pages"></a>

静的なページというのは、主たるコンテンツのほとんどが静的なもので、コントローラからプッシュされる動的なデータに
アクセスする必要がないページを指します。

静的なページは、そのコードをビューに置き、そして、コントローラで次のようなコードを使うと表示することが出来ます:

```php
public function actionAbout()
{
    return $this->render('about');
}
```

ウェブサイトが多くの静的なページを含んでいる場合、同じようなコードを何度も繰り返すのは非常に面倒くさいでしょう。
この問題を解決するために、[[yii\web\ViewAction]] という [スタンドアロンアクション](structure-controllers.md#standalone-actions)
をコントローラに導入することが出来ます。例えば、

```php
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
    public function actions()
    {
        return [
            'page' => [
                'class' => 'yii\web\ViewAction',
            ],
        ];
    }
}
```

このようにすると、ディレクトリ `@app/views/site/pages` の下に `about` という名前のビューを作成したときに、
次の URL によってこのビューを表示することが出来るようになります:

```
http://localhost/index.php?r=site/page&view=about
```

`view` という `GET` パラメータが、どのビューがリクエストされているかを [[yii\web\ViewAction]] に教えます。
そこで、アクションはこのビューをディレクトリ `@app/views/site/pages` の下で探します。
[[yii\web\ViewAction::viewPrefix]] を構成して、ビューを探すディレクトリを変更することが出来ます。


## 最善の慣行 <a name="best-practices"></a>

ビューはエンドユーザが望む形式でモデルを表現することに対して責任を持ちます。一般的に、ビューは

* 主として表示目的のコードを含むべきです。例えば、HTML、そしてデータをたどり、書式化してレンダリングする簡単な PHP コードなど。
* DB クエリを実行するコードは含むべきではありません。そのようなコードはモデルの中で実行されるべきです。
* `$_GET` や `$_POST` のようなリクエストデータに直接アクセスするべきではありません。それはコントローラの仕事です。
  リクエストデータが必要な場合は、コントローラからビューにプッシュされるべきです。
* モデルのプロパティを読み出すことが出来ます。しかし、それを修正するべきではありません。

ビューを管理しやすいものにするために、複雑すぎるビューや、冗長なコードをあまりに多く含むビューを作ることは避けましょう。
この目的を達するために、次のテクニックを使うことが出来ます:

* 共通の表示セクション (ページのヘッダやフッタなど) を表すために [レイアウト](#layouts) を使う。
* 複雑なビューはいくつかの小さなビューに分割する。既に説明したレンダリングのメソッドを使えば、
  小さなビューをレンダリングして大きなビューを組み上げることが出来る。
* ビューの構成要素として [ウィジェット](structure-widgets.md) を使う。
* ビューでデータを変換し書式化するためのヘルパークラスを作成して使う。

