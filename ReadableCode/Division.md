# 거대한 코드를 분할하라

## 설명 변수

👎 Bad

```php
$wordCount = count(explode(' ', "Hello World"));

var_dump($wordCount);
```

👍 Good

```php
$message = "Hello World";

$words = explode(' ', $message);

$wordCount = count($words);

var_dump($words);
var_dump($wordCount);
```

변수의 중간 단계에 설명을 두면, 변수의 명칭을 통해서 좀 더 이해하기 쉬워진다.

## 요약 변수

```php
class Controller
{
    public function edit(int $id)
    {
        if (Document::find($id) === null)
        {
            return abort(404);
        }

        return viewEditPage();
    }
}

(new Controller)->edit(30, new Document);
(new Controller)->edit(31, new Document);

class Document
{
	public static function find(int $id)
	{
		if ($id === 30) return new self;
		return null;
	}
}

function abort(int $statusCode) {
	if ($statusCode === 404) {
	    echo 'page not found'.PHP_EOL;	
	}
}

function viewEditPage()
{
    echo "edit page".PHP_EOL;
}
```

👎 Bad

```php
class Controller
{
    public function edit(int $id)
    {
        if (Document::find($id) === null)
        {
            return abort(404);
        }

        return viewEditPage();
    }
}
```

👍 Good

```php
class Controller
{
    public function edit(int $id)
    {
        $hasDocument = Document::find($id) instanceof Document;
        if (!$hasDocument)
        {
            return abort(404);
        }

        return viewEditPage();
    }
}
```

사용자가 소유한 문서에 대해서 edit를 할 수 있어야 한다. 따라서 사용자가 문서를 소유하고 있는가?의 의미를 나타내는 변수 `$hasDocument`를 쓰면 좀 더 이해하기 쉬운 코드를 만들 수 있다.


