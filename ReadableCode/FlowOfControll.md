# 제어의 흐름

## 대상을 기준 보다 먼저 적기

'어떻게 무엇을'의 순서 보다는 '무엇을 어떻게'의 순서가 자연스럽다.

무엇에 해당하는 개념을 먼저 쓰고, 어떻게 할 것인지 나타내는 코드를 만드는 편이 좋다.

👎 Bad

```php
if ($length >= 10) {
}
```
- 길이가 10 보다 크다.
- 무엇(길이가), 어떻게(10 보다 크다)의 순이다.

👍 Good

```php
if (10 <= $length) {
}
```
- 10 보다 큰 길이.
- 어떻게(10 보다 큰), 무엇(길이)의 순이다.

코드를 적는 순서에 따라 생각되는 언어의 표현이 달라진다. 무엇을 어떻게로 읽히도록 코드를 적는 편이 좋다.

---

👎 Bad

```php
while ($bytes_received < $bytes_expected) {
    ...
}
```

- 전달된 값(`$bytes_received`)은 확인하고자 하는 대상(`$bytes_expected`) 보다 작다.

👍 Good

```php
while ($bytes_expected > $bytes_received) {
    ...
}
```

- 확인하고자 하는 대상(`$bytes_expected`)은 전달된 값(`$bytes_received`) 보다 크다.

확인하고자 하는 대상을 먼저 쓰는 편이 더 자연스럽다.

## Yoda기법

요다 기법은 문장의 단어의 순서을 이상하게 바꾸어 말하는 특징에서 나온 용어다.

대입을 하는 `=` 문법과 비교를 하는 `==` 또는 `===` 문법을 사용할 때 코딩의 실수를 방지하기 위해서 값을 왼쪽에 변수를 오른쪽에 배치하는 코딩 스타일이다.

👌 SOSO

```
if ($var === null) {
    ...
}
```

- `===`를 `=`으로 적어 코딩의 실수가 있다면 `$var`의 값은 `null`이 되어 버린다.

👍 Good

```
if (null === $var) {
    ...
}
```

- `===`를 `=`으로 적어 코딩의 실수가 있더라도 값에는 대입이 불가능하기 때문에 실행시 에러가 발생하게 된다.

요다 기법은 '무엇을 어떻게'의 순의 변수 값의 순서의 역순의 나열 방향이다.

## if-else문

### if문은 긍정형으로

기본적으로 긍정적인 조건문을 먼저 사용하도록 하자.

👎 Bad

```php
if ($a !== $b) {
   // ...
} else {
   // ...
}
```

👍 Good

```php
if ($a === $b) {
    // ...
} else {
    // ...
}
```

### 관심끄는 주제를 먼저 처리하자.

else 문의 분기 조건을 확인하기 위해서는 if 문의 조건을 확인할 수 밖에 없고, 조건문을 읽는 순간 조건문을 머릿속에 떠올릴 수 밖에 없다. 생각한 내용을 뒤로 미루기 보다는 생각한 내용을 먼저 처리하도록 만드는 코드가 자연스럽다.

다음 예제는 쿼리 파라메터 "with_detail"의 값이 있는지 확인하는 코드로 관심을 끄는 주제이다.

👎 Bad

```php
if (!$request->query('with_detail')) {
    $users = User::query()->limit(100)->get();
} else {
    $users = User::query()->withUserDetail()->limit(100)->get();
}
return view('users', ['users' => $users]);
```

쿼리 파라메터가 없을 때 처리하는 코드가 위에 있으므로 먼저 읽게 된다. 조건문의 주제와 연관성이 떨어진 처리를 하기 때문에 주의가 분산된다.

👍 Good

```php
if ($request->query('with_detail')) {
    $users = User::query()->withUserDetail()->limit(100)->get();
} else {
    $users = User::query()->limit(100)->get();
}
return view('users', ['users' => $users]);
```

쿼리 파라메터가 있을 때 처리하는 코드가 위에 있으므로 먼저 읽게 된다. 조건문의 주제와 연관성이 있기 때문에 집중할 수 있고, 조건문의 주제와 떨어진 것은 else로 거리가 떨어진 곳에 위치하므로 연관성이 떨어져 있다고 자연스럽게 생각할 수 있다.

조건이 부정형이라도 관심을 끄는 조건이라면 if문의 조건에 넣는 편이 좋다.

## 삼항연산자

삼항 연산자를 사용하면 장황한 코드를 간결하게 만들 수 있다.

👎 Bad

```php
if (hour => 12) {
    time_str += "pm";
} else {
    time_str += "am";
}
```

👍 Good

```php
$time_str += (hour >= 12) ? "p" : "am";
```

### 간결한 경우에만 삼항 연산자를 쓰자.

코드를 짧게 만드는 것이 중요한 것이 아니라, 쉽게 파악할 수 있는 코드를 만드는 것이 중요하다.

👎 Bad

```php
return $exponent >= 0 ? $mantissa * pow(2, $exponent) : $mantissa / pow(2, -$exponent);
```

👍 Good

```php
if ($exponent >= 0) {
    return $mantissa * pow(2, $exponent);
} else {
    return $mantissa / pow(2, -$exponent);
}
```

if 블록과 else 블록에서 처리하는 동작이 어떤 차이를 갖는지 쉽게 비교할 수 있다.

## do while

do ... while은 한 번 실행을 하고, while 조건에 맞으면 또 실행하는 구문이며, while 구문은 조건에 맞을 때 또 실행하는 구문이다.

do ... while은 조건에 관계 없이 반드시 한 번은 실행되지만 while은 조건에 맞지 않으면 한 번도 실행되지 않는 문법이다.

문제는 do ... while은 do 블록을 읽고 while의 조건을 보고 do 블록을 한 번 더 읽게 되는 코드를 두 번 읽게 유도하는 코드이므로 가능하면 간결하게 while문 하나로 쓰는 편이 좋다.

👎 Bad

```php
public function ListHasNode(Node $node, string $name, int $max_length) {
   do {
       if ($node->name() === $name) return true;
       $node = $node->next();
   } while ($node !== null && $max_length-- > 0);
   return false;
}
```

👍 Good

```php
public function ListHasNode(Node $node, string $name, int $max_length) {
   while ($node !== null && $max_length-- > 0) {
       if ($node->name() === $name) return true;
       $node = $node->next();
   }
   return false;
}
```

## 얼리 리턴 사용하기

특정 조건에 맞지 않으면 다음 코드가 실행되지 않는다는 것을 통해서, 코드가 실행되는 조건을 하나씩 제외할 수 있다.

👎 Bad

```php
function Contains(string $str, string $substr) {
    $result = false;
    
    if ($str !== null && $substr !== null) {
        if ($substr === '') {
            $result = true;
        } else {
            ...
            $result = (strpos($str, $substr) !== false);
        }
    }
    
    return $result;
}
```

👍 Good

```php
function Contains(string $str, string $substr) {
    if ($str === null || $substr === null) return false;
    if ($substr === '') return true;
    ...
}
```

## goto

goto 문의 문제점은 코드가 어디로 이동할지 파악하기 어렵다는 문제점이 있다. 코드를 읽기 위해 이리 저리 왔다갔다 해야 해서 코드를 순차적으로 읽어나가는 흐름을 방해할 수 있다.

하지만, 건너 뛰어야 하는 코드가 많은 경우, 다음 코드를 순차적으로 읽는 것 보다는 다음 코드 모두를 무시하고 지정한 부분까지 코드를 읽을 필요가 없다는 것을 알려주기 위해 goto문을 사용할 수 있다.

👎 Bad

```php
$sentence = '';

str_replace('str', 'string', $inputValue);

str_replace('arr', 'array', $inputValue);

str_replace('int', 'integer', $inputValue);

// ...

$sentence .= $inputValue:
```

위에서 아래로 순차적으로 읽으면 데이터 변경을 하지 않는 경우도 데이터 변경 코드를 읽어야 한다.

👎 Good

```php
$sentence = '';

if (
    is_numeric($inputValue)
    || $inputValue === ''
    // ...
) goto not_chnage;

str_replace('str', 'string', $inputValue);

str_replace('arr', 'array', $inputValue);

str_replace('int', 'integer', $inputValue);

// ...

not_chnage:

$sentence .= $inputValue:
```

특정한 조건에 대해서는 다음 코드를 읽을 필요가 없다는 것을 알려주기 위해서 goto문을 사용할 수도 있다.

## 얕은 중첩

👎 Bad

```php
f ($user_result === SUCCESS) {
    if ($permission_result !== SUCCESS) {
        $reply->writeErrors("error reading permissions");
        $reply->done();
        return;
    }
    $reply->writeErrors("");
} else {
    $reply->writeErrors($user_result);
}
$reply->done();
```

👍 Good

```php
if ($user_result !== SUCCESS) {
    $reply->writeErrors($user_result);
    $reply->done();
    return;
}

if ($permission_result !== SUCCESS) {
    $reply->writeErrors($permission_result);
    $reply->done();
    return;
}

$reply->writeErrors();
$reply->done();
```

## loop의 중첩

👎 Bad

```php
$null_count = 0;
foreach ($results as $result) {
    if ($result !== null) {
        $null_count++;
        if ($result->name !== "") {
            echo "Considering candidate...\n";
        }
    }
}
```

👍 Good

```php
$non_null_count = 0;
foreach ($results as $result) { 
    if ($result === null) continue;
    
    $non_null_count++;
    
    if (empty($result->name)) continue;
    
    echo "Considering candidate...\n"; 
}
```

## 흐름

일반적으로 코드는 위에서 아래로 읽을 수 있도록 작성된다.

exec를 통한 외부 프로세스 실행, goto 문, 스레드, 예외, (통신 등의) 비동기 등의 코드는 위에서 아래로의 흐름이 아닌 흐름의 코드를 만들 수 있기 때문에 작성 시 주의해야 한다.
