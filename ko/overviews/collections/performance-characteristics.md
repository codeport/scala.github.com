---
layout: overview-large
title: 성능 특성

disqus: true

partof: collections
num: 12
language: ko
---

앞의 설명에서 다른 종류의 콜렉션은 다른 성능 특성(performance characteristics)을 지니는 것을 알 수 있다. 이러한 성능 특성은 여러 콜렉션 가운데 하나를 선택하는 주요 기준이 된다. 아래의 두 개의 표에서 콜렉션들의 공통된 연산들의 성능 특성을 요약하였다.

시퀀스 타입의 성능 특성:

|               | head | tail | apply | update| prepend | append | insert |
| --------      | ---- | ---- | ----  | ----  | ----    | ----   | ----   |
| **immutable** |      |      |       |       |         |        |        |
| `List`        | C    | C    | L     | L     |  C      | L      |  -     |
| `Stream`      | C    | C    | L     | L     |  C      | L      |  -     |
| `Vector`      | eC   | eC   | eC    | eC    |  eC     | eC     |  -     |
| `Stack`       | C    | C    | L     | L     |  C      | C      |  L     |
| `Queue`       | aC   | aC   | L     | L     |  L      | C      |  -     |
| `Range`       | C    | C    | C     | -     |  -      | -      |  -     |
| `String`      | C    | L    | C     | L     |  L      | L      |  -     |
| **mutable**   |      |      |       |       |         |        |        |
| `ArrayBuffer` | C    | L    | C     | C     |  L      | aC     |  L     |
| `ListBuffer`  | C    | L    | L     | L     |  C      | C      |  L     |
|`StringBuilder`| C    | L    | C     | C     |  L      | aC     |  L     |
| `MutableList` | C    | L    | L     | L     |  C      | C      |  L     |
| `Queue`       | C    | L    | L     | L     |  C      | C      |  L     |
| `ArraySeq`    | C    | L    | C     | C     |  -      | -      |  -     |
| `Stack`       | C    | L    | L     | L     |  C      | L      |  L     |
| `ArrayStack`  | C    | L    | C     | C     |  aC     | L      |  L     |
| `Array`       | C    | L    | C     | C     |  -      | -      |  -     |

집합과 맵의 성능 특성:

|                    | lookup | add | remove | min           |
| --------           | ----   | ---- | ----  | ----          |
| **immutable**      |        |      |       |               |
| `HashSet`/`HashMap`| eC     | eC   | eC    | L             |
| `TreeSet`/`TreeMap`| Log    | Log  | Log   | Log           |
| `BitSet`           | C      | L    | L     | eC<sup>1</sup>|
| `ListMap`          | L      | L    | L     | L             |
| **mutable**        |        |      |       |               |
| `HashSet`/`HashMap`| eC     | eC   | eC    | L             |
| `WeakHashMap`      | eC     | eC   | eC    | L             |
| `BitSet`           | C      | aC   | C     | eC<sup>1</sup>|
| `TreeSet`          | Log    | Log  | Log   | Log           |

각주: <sup>1</sup> 비트들이 밀집해서 담겨있다고 전제한다.

두 표의 각 요소는 아래의 의미를 가진다:

|     |                                           |
| --- | ----                                      |
| **C**   | 연산에 (빠른) 상수 시간이 소요된다 |
| **eC**  | 연산에 실질적으로 상수 시간이 소요되지만, 벡터의 최대 길이나 해시 키의 분포등의 전제에 의존한다.|
| **aC**  | 평균적으로 상수 시간이 소요된다. 연산의 호출이 떄때로 더 긴 시간이 걸릴 때도 있지만, 여러 번의 연산 소요 시간을 평균내면 한 연산 당 상수 시간이 소요된다.|
| **Log** | 연산 소요 시간이 콜렉션 크기의 로그 연산에 비례한다.|
| **L**   | 연산에 선형 시간이 소요되며, 콜렉션 크기에 비례한다.|
| **-**   | 연산이 지원되지 않는다.|

첫 번째 표는 시퀀스 타입(가변과 불변 모두)을 아래의 연산으로 사용하였다:

|     |                                                     |
| --- | ----                                                |
| **head**   | 시퀀스의 첫 번째 원소를 선택한다. |
| **tail**   | 첫 번째 원소를 제외한 모든 원소를 포함하는 새로운 시퀀스를 생성한다.|
| **apply**  | 인덱스 접근. |
| **update** | 불변 시퀀스에서는 함수형 갱신(`updated`), 가변 시퀀스에는 사이드 이펙트 갱신 (`update`).|
| **prepend**| 원소를 시퀀스 제일 앞에 추가한다. 불변 시퀀스에서는 새로운 시퀀스를 생성한다. 가변 시퀀스에서는 존재하는 시퀀스를 수정한다.|
| **append** | 원소를 시퀀스 제일 뒤에 추가한다. 불변 시퀀스에서는 새로운 시퀀스를 생성한다. 가변 시퀀스에서는 존재하는 시퀀스를 수정한다.|
| **insert** | 원소를 시퀀스의 임의 위치에 삽입한다. 가변 시퀀스에서만 지원된다.|

두 번째 표는 가변, 불변 집합과 맵을 아래의 연산으로 사용하였다:

|     |                                                     |
| --- | ----                                                |
| **lookup** | 원소가 집합에 포함되어있는지를 검사하거나, 키에 관련된 값을 선택한다.|
| **add**    | 집합에 새로운 원소를 추가하거나, 맵에 키/값 쌍을 추가한다.|
| **remove** | 집합에서 원소를 제거하거나, 맵에서 키를 삭제한다.|
| **min**    | 집합의 가장 작은 원소나 맵의 가장 작은 키를 반환한다.|

