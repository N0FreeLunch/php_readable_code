# 주석

## 주석은 간결하고 명확하게

```php
/**
 * @var array<int, Pair<float, float>> $scoreMap
 */
$scoreMap = [];

/**
 * @template TScore of int|float
 * @template TWeight of int|float
 */
class Pair
{
    /**
     * @param TScore $score
     * @param TWeight $weight
     */
    public function __construct(
        private readonly mixed $score,
        private readonly mixed $weight
    ) {
        // ...
    }

    /**
     * @return TScore&TWeight
     */
    public function evaluate(): mixed
    {
        return $this->score * $this->weight;
    }
}
```

### 주석을 어떻게 달까?

// int는 타입 카테고리
// pair의 첫 번째 float은 'score'
// pair의 두 번째 float은 'weight'

위와 같은 3줄의 주석으로 적을 필요가 없다. 다음과 같이 간결하게 주석을 적어 준다.

// CategoryType -> (score, weight)

## 대명사는 피한다.

이것, 저것 등의 표현을 쓰지 않고, 가리키는 대상을 정확한 단어로 표기한다.

## 단어의 의미를 명확히

```
$countLines = function(string filename): int { ... };
```

> // 이 파일에 포함된 행 수를 반환한다.

이 때 주석의 '행'이란 정확하게 어떤 의미인지 모호할 수 있다.

- 파일에 `hello`라는 데이터가 쓰여 있을 때 이 행은 1번 행인가? 0번 행인가?
- `hello\n`은 1개 행으로 구성되어 있는가? 2개 행으로 구성되어 있는가?
- `"hellol\nworld"`는 1개 행으로 구성되어 있는가? 2개 행으로 구성되어 있는가?
- `"hellol\n\rcruel\nworld\r"`은 2행 구성인가? 3행 구성인가?

행이라는 단어가 모호하기 때문에 정확하게 표현하려면 줄 바꿈 문자의 갯수를 세는 것이 좋다. 주석을 고쳐쓰면 다음과 같다.

> '// 이 파일에 포함된 줄 바꿈 문자 ('\n')행 수를 반환한다.'

## 적절한 예를 주석으로 적는다.

```php
$strip = function(string $sc, string $chars): string {
};
```

```php
/**
 * @example $strip("abba/a/ba", "ab") => "/a/"
 */
$strip = function(string $sc, string $chars): string {
};
```

## 코드의 의도를 쓰기

```php
$compareProductByPrice = function(Product $a, Product $b) {
    return $a->price - $b->price;
}); 

/**
 * @param array<int, Product> $products
 */
function displayProducts(array $products) use ($compareProductByPrice): void 
{
    usort($products, $compareProductByPrice);
    
    for ($i = count($products) - 1; $i >= 0; $i--) {
        displayPrice($products[$i]->price);
    }
}
```

```php
    for ($i = count($products) - 1; $i >= 0; $i--) {
        displayPrice($products[$i]->price);
    }
```

위 부분의 주석으로 적절한 것은?

1. 배열을 역순으로 순회하기
2. 가격을 높은 순서대로 표시함

## 명명된 인수를 사용

```php
$connect(10, false);
```

```php
$connect(timeout: 10, use_encryption: false)
```

파라메터 명을 사용하는 것으로 어떤 값이 전달되는지 코드를 읽는 사람이 명확히 알 수 있다.

## 정보의 밀도가 높은 언어를 사용

// このクラスには大量のメンバがある。同じ情報はデータベースにも保管されている。ただし、
// 速度の面からここにも保管しておく。このクラスを読み込むときには、メンバが存在してい
// るかどうかを先に確認する。もし存在していれば、そのまま返す。存在しなければ、データベー
// スから読み込んで、次回のためにデータをフィールドに保管する。

위의 주석 보다는

// このクラスの役割は、データベースのキャッシュ層である。

---

// 所在地から余分な空白を除去する。それから「Avenue」を「Ave.」にするなどの整形を施す。
// こうすれば、表記がわずかに違う所在地でも同じものであると判別できる。

위의 주석 보다는

// 所在地を正規化する（例："Avenue" -> "Ave."）。
