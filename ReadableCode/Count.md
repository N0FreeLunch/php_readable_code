## 과제

최근 1분의 서버의 데이터 전송량

최근 1시간 서버의 데이터 전송량

### 불필요한 get은 생략하자

👎 Bad

```php
$obj->getMinuteCount();
$obj->getHourCount();
```

setter와 짝을 이루지 않는다면 get은 불필요한 접미사일 수 있다.

👍 Good

```php
$obj->minuteCount();
$obj->hourCount();
```

get 접미사를 쓰지 않아도 객체에서 값을 얻는다는 것을 보통 알 수 있기 때문에 get을 쓰지 않는 편을 추천한다.

### 단어 선정이 적절한가?

👎 Bad

```php
interface MinuteHourCounterInterface
{
    public function count(int $numByte): int;
}
```

`count`라는 메소드명과 `$numByte`라는 매개 변수로 무엇을 위한 메소드인지 알 수 없다.

👍 Good

```php
interface MinuteHourCounterInterface
{
    public function add(int $count): int;
}
```

객체에 `$count` 만큼의 셀수 있는 값을 추가(`add`)한다는 의미라는 것을 알 수 있다.

### 주석 추가

주석을 명확하고 오해 없게 만들자.

👎 Bad

```php
interface MinuteHourCounterInterface {
    /**
     * 카운트를 추가한다.
     */
    public function add(int $count): void;
    /**
     * 최근 1분간의 카운트를 반환한다.
     */
    public function minuteCount(): int;
    /**
     * 최근 1시간의 카운트를 반환한다.
     */
    public function hourCount(): int;
}
```

👍 Good

```php
/**
 * 최근 1분간, 최근 1시간의 누적 카운트를 기록한다.
 * 대역폭 사용 여부를 확인하는데 사용할 수 있다.
 */
interface MinuteHourCounterInterface {
    /**
     * 새로운 데이터 추가 (count >= 0)
     * 데이터 추가 시점에서 1분간, minuteCount 반환 값이 +count 만큼 증가
     * 데이터 추가 시점에서 1시간, hourCount 반환 값이 +count 만큼 증가
     */
    public function add(): void;
    /**
     * 최근 60초간의 누적 카운트를 반환한다.
     */
    public function minuteCount(): int;
    /**
     * 최근 3600초의 누적 카운트를 반환한다.
     */
    public function hourCount(): int;
}
```

### 리펙토링

```php
class MinuteHourCounter implements MinuteHourCounterInterface
{
    private array $events = [];

    public function add(int $count): void
    {
        if ($count < 0) {
            throw new InvalidArgumentException("Count must be >= 0");
        }

        $this->events[] = ['count' => $count, 'time' => time()];
    }

    public function minuteCount(): int
    {
        $now = time();
        $threshold = $now - 60;
        $count = 0;

        for ($i = count($this->events) - 1; $i >= 0; --$i) {
            $event = $this->events[$i];
            if ($event['time'] <= $threshold) {
                break;
            }
            $count += $event['count'];
        }

        return $count;
    }

    public function hourCount(): int
    {
        $now = time();
        $threshold = $now - 3600;
        $count = 0;

        for ($i = count($this->events) - 1; $i >= 0; --$i) {
            $event = $this->events[$i];
            if ($event['time'] <= $threshold) {
                break;
            }
            $count += $event['count'];
        }

        return $count;
    }
}
```

`minuteCount`와 `hourCount`의 코드의 중복을 메소드화 할 수 있다.

```php
class MinuteHourCounter implements MinuteHourCounterInterface
{
    private array $events = [];

    public function add(int $count): void
    {
        if ($count < 0) {
            throw new InvalidArgumentException("Count must be >= 0");
        }

        $now = time();
        $this->events[] = ['count' => $count, 'time' => $now];
    }

    public function minuteCount(): int
    {
        return $this->countSince(seconds: 60);
    }

    public function hourCount(): int
    {
        return $this->countSince(seconds: 3600);
    }

    private function countSince(int $seconds): int
    {
        $now = time();
        $threshold = $now - $seconds;
        $count = 0;

        // 역순 순회
        for ($i = count($this->events) - 1; $i >= 0; --$i) {
            $event = $this->events[$i];
            if ($event['time'] <= $threshold) {
                break; // 오래된 이벤트: 더 볼 필요 없음
            }
            $count += $event['count'];
        }

        return $count;
    }
}
```

`countSince(int $seconds): int`을 `countSince(int $cutoff): int`으로 바꾸어 `countSince` 부분의 로직을 간소화 했다.

```php
$this->countSince(seconds: 60);
$this->countSince(seconds: 3600);
```

```php
$this->countSince(time() - 60);
$this->countSince(time() - 3600);
```

메소드명이 `countSince`이므로 매개변수로는 `count`으로 집계할 대상이 생성되는 타이밍을 매개변수에 넣어주는 것이 좋으므로, 현재 Unix 초에서 60초 전, 3600초 전의 시점을 지정한다.

```php
class MinuteHourCounter implements MinuteHourCounterInterface
{
    private array $events = [];

    public function add(int $count): void
    {
        if ($count < 0) {
            throw new InvalidArgumentException("Count must be >= 0");
        }

        $this->events[] = ['count' => $count, 'time' => time()];
    }

    public function minuteCount(): int
    {
        return $this->countSince(time() - 60);
    }

    public function hourCount(): int
    {
        return $this->countSince(time() - 3600);
    }

    private function countSince(int $cutoff): int
    {
        $count = 0;

        // 배열 뒤에서부터 순회 (C++의 reverse_iterator 대응)
        for ($i = count($this->events) - 1; $i >= 0; --$i) {
            $event = $this->events[$i];
            if ($event['time'] <= $cutoff) {
                break;
            }
            $count += $event['count'];
        }

        return $count;
    }
}
```

`$events`에 이벤트가 계속 쌓이면 메모리가 무한정 늘어난다. 속도를 개선하기 위해 다음과 같이 리펙토링한다.

`$this->shiftOldEvents($now)`를 통해서 오래된 이벤트는 제거한다.

```php
class MinuteHourCounter implements MinuteHourCounterInterface
{
    private array $minuteEvents = []; // list of ['count' => int, 'time' => int]
    private array $hourEvents = [];

    private int $minuteCount = 0;
    private int $hourCount = 0;

    public function add(int $count): void
    {
        if ($count < 0) {
            throw new InvalidArgumentException("Count must be >= 0");
        }

        $now = time();
        $this->shiftOldEvents($now);

        $this->minuteEvents[] = ['count' => $count, 'time' => $now];
        $this->minuteCount += $count;
        $this->hourCount += $count;
    }

    public function minuteCount(): int
    {
        $this->shiftOldEvents(time());
        return $this->minuteCount;
    }

    public function hourCount(): int
    {
        $this->shiftOldEvents(time());
        return $this->hourCount;
    }

    private function shiftOldEvents(int $now): void
    {
        $minuteAgo = $now - 60;
        $hourAgo = $now - 3600;

        // Move old events from minute to hour
        while (!empty($this->minuteEvents) && $this->minuteEvents[0]['time'] <= $minuteAgo) 
        {
            $event = array_shift($this->minuteEvents);
            $this->minuteCount -= $event['count'];
            $this->hourEvents[] = $event;
        }

        // Remove expired events from hour list
        while (!empty($this->hourEvents) && $this->hourEvents[0]['time'] <= $hourAgo) {
            $event = array_shift($this->hourEvents);
            $this->hourCount -= $event['count'];
        }
    }
}
```

그럼에도 불구하고 이벤트에 누적되는 아이템의 수가 많아질 수 있기 때문에 메모리 안전한 코드가 아니다.

메모리 사용량을 줄이기 위해서 1분 동안 추가된 이벤트의 경우 평균을 내서 저장하도록 한다.

```php
class ConveyorQueue
{
    private array $queue = [];
    private int $maxItems;
    private int $totalSum = 0;

    public function __construct(int $maxItems)
    {
        $this->maxItems = $maxItems;
    }

    public function totalSum(): int
    {
        return $this->totalSum;
    }

    public function shift(int $numShifted): void
    {
        if ($numShifted >= $this->maxItems) {
            $this->queue = [];
            $this->totalSum = 0;
            return;
        }

        while ($numShifted-- > 0) {
            array_push($this->queue, 0);
        }

        while (count($this->queue) > $this->maxItems) {
            $removed = array_shift($this->queue);
            $this->totalSum -= $removed;
        }
    }

    public function addToBack(int $count): void
    {
        if (empty($this->queue)) {
            $this->shift(1);
        }

        $lastIndex = count($this->queue) - 1;
        $this->queue[$lastIndex] += $count;
        $this->totalSum += $count;
    }
}
```

```php
class TrailingBucketCounter
{
    private ConveyorQueue $buckets;
    private int $secsPerBucket;
    private int $lastUpdateTime;

    public function __construct(int $numBuckets, int $secsPerBucket)
    {
        $this->buckets = new ConveyorQueue($numBuckets);
        $this->secsPerBucket = $secsPerBucket;
        $this->lastUpdateTime = time();
    }

    private function update(int $now): void
    {
        $currentBucket = intdiv($now, $this->secsPerBucket);
        $lastBucket = intdiv($this->lastUpdateTime, $this->secsPerBucket);
        $this->buckets->shift($currentBucket - $lastBucket);
        $this->lastUpdateTime = $now;
    }

    public function add(int $count, int $now = null): void
    {
        $now = $now ?? time();
        $this->update($now);
        $this->buckets->addToBack($count);
    }

    public function trailingCount(int $now = null): int
    {
        $now = $now ?? time();
        $this->update($now);
        return $this->buckets->totalSum();
    }
}
```

#### ConveyorQueue

정해진 최대 수(max_items)를 유지하며, 자동으로 오래된 데이터를 제거

#### TrailingBucketCounter

- 버킷 기반 시간 윈도우 카운터
- 초당, 분당 등 간격을 지정 가능
- Add 시 현재 시간에 따라 적절한 버킷으로 이동
- 총합 반환 가능
