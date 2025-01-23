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

👎 Bad

```php
/**
 * int는 타입 카테고리
 * pair의 첫 번째 float은 'score'
 * pair의 두 번째 float은 'weight'
 * @var array<int, Pair<float, float>> $scoreMap
 */
$scoreMap = [];
```

위와 같은 3줄의 주석으로 적을 필요가 없다. 다음과 같이 간결하게 주석을 적어 준다.

👍 Good

```php
/**
 * CategoryType -> (score, weight)
 * @var array<int, Pair<float, float>> $scoreMap
 */
$scoreMap = [];
```

#### 참고

php에서 무리하게 docblock generic을 사용하는 코드 보다, assert를 이용해서 IDE에 타입을 알려주는 코딩 스타일을 사용하는 편이 좋을 수 있다.

```php
$total = 0;

foreach($scoreMap as $pair) {
    assert($pair instanceof Pair);
    $result = $pair->evaluate();
    assert(is_float($result));
    $total += $result;
}
```

## 대명사는 피한다.

이것, 저것 등의 표현을 쓰지 않고, 가리키는 대상을 정확한 단어로 표기한다.

## 단어의 의미를 명확히

👎 Bad

```php
// 지정한 파일 데이터의 총 행 수를 반환한다.
$countLines = function(string filename): int { ... };
```

이 때 주석의 '행'이란 정확하게 어떤 의미인지 모호할 수 있다.

- 파일에 `hello`라는 데이터가 쓰여 있을 때 이 행은 1번 행인가? 0번 행인가?
- `hello\n`은 1개 행으로 구성되어 있는가? 2개 행으로 구성되어 있는가?
- `"hellol\nworld"`는 1개 행으로 구성되어 있는가? 2개 행으로 구성되어 있는가?
- `"hellol\n\rcruel\nworld\r"`은 2행 구성인가? 3행 구성인가?

행이라는 단어가 모호하기 때문에 정확하게 표현하려면 줄 바꿈 문자의 갯수를 세는 것이 좋다. 주석을 고쳐쓰면 다음과 같다.

👍 Good

```php
// 지정한 파일에 포함된 줄 바꿈 문자('\n')으로 행을 구분했을 때의 총 행 수를 반환한다.
$countLines = function(string filename): int { ... };
```

## 적절한 예를 주석으로 적는다.

👎 Bad

```php
$strip = function(string $sc, string $chars): string {
};
```

👍 Good

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
        echo $products[$i]->price.PHP_EOL;
    }
}
```

👎 Bad

```php
// 배열을 역순으로 순회하기
for ($i = count($products) - 1; $i >= 0; $i--) {
    echo $products[$i]->price.PHP_EOL;
}
```

👍 Good

```php
// 가격을 높은 순서대로 표시함
for ($i = count($products) - 1; $i >= 0; $i--) {
    echo $products[$i]->price.PHP_EOL;
}
```

## 명명된 인수를 사용

👎 Bad

```php
$connect(10, false);
```

👍 Good

```php
$connect(timeout: 10, use_encryption: false)
```

파라메터 명을 사용하는 것으로 어떤 값이 전달되는지 코드를 읽는 사람이 명확히 알 수 있다.

---

👎 Bad

```php
$user->getUserById(userId: 50);
```

👍 Good

```php
$user->getUserById(50);
```

굳이 명명된 인자를 사용하지 않아도 어떤 값이 전달될지 쉽게 파악할 수 있는 경우에는 인자명은 의미 중복이므로 굳이 쓸 필요가 없다.

## 정보의 밀도가 높은 언어를 사용

👎 Bad

```
// 데이터베이스에서 가져온 대량의 맴버를 저장하는 클래스. 속도 향상을 위해 이 클래스에 저장.
// 주의: 로드할 때는 멤버가 존재하는지 먼저 확인한다.
// 멤버가 존재하면 클래스에 저장된 값 그대로 반환한다.
// 멤버가 존재하지 않는다면, 데이터베이스에서 가져와서 다음 번을 위해 데이터를 필드에 저장한다.
```

👍 Good

```
// 이 클래스는 데이터베이스의 캐시 계층의 역할을 한다.
```

정보의 밀도가 높은 '캐시'라는 용어를 사용해서 간결하게 표현하면서도 의미를 잘 드러냄

---

👎 Bad

```
// 소재지에서 불필요한 공백을 제거 후, "Avenue"를 "Ave." 형식으로 변경한다.
// 이렇게 하면 표기가 약간 다른 소재지라도 동일한 것으로 식별할 수 있다.
```

👍 Good

```
// 소재지를 정규화 （예："  Avenue   " -> "Ave."）
```

정보의 밀도가 높은 '정규화'라는 용어와 정규화를 어떻게 하는지 예를 드는 것으로 간결하게 표현하면서도 의미를 잘 드러냄
