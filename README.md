# RxSwift 전체 요약

**목차**  
- [기본개념](#기본개념)
- [Observable](#observable)
- [Subject](#subject)  
- [Filtering Operator](#filtering-operator)
- [Transforming Operator](#transforming-operator)
- [Combinging Operator](#combinging-operator)
- [TimeBased Operator](#timebased-operator)
- [Error Handling](#error-handling)
- [RxCocoa](#rxcocoa)

## **기본개념**

#### RxSwift

-   우리가 작성하는 코드의 대부분은 외부이벤트에 대한 응답과 관련
-   사용자가 컨트롤을 조작할때 응답할 IBAction handler, 키보드 위치 변경을 감지하기 위해 notification을 관찰해야함, urlsession이 데이터로 응답할 때 실행할 클로저, KVO를 사용해서 변수의 변경사항을 감지해야함
-   이러한 다양한 시스템은 모두 코드를 복잡하게 만듬
-   또한 일반적으로 대부분의 클래스들은 비동기적으로 작업을 수행하고 모든 UI 구성요소들은 본질적으로 비동기적
-   따라서 내가 어떤 앱 코드를 작성했을 때 정확히 매번 어떤 순서로 작동하는지 가정하는 것이 불가능
-   결국 앱의 코드는 사용자 입력, 네트워크 활동 또는 기타 OS 이벤트와 같은 다양한 외부 요인에 따라 완전히 다른 순서로 실행될 수 있음
-   모든 호출 응답코드를 처리하는 일관된 시스템이 하나 있다면, 더좋지 않을까 한 질문에서 시작한것이 Rxswift
-   전체적인 과정에서 Observable과 Observer만 있을뿐 delegate패턴, 클래스간의 통신을 위해 closure를 삽입할 필요가 없음
-   RxSwift는 '본질적'으로 코드가 '새로운 데이터에 반응'하고 '순차적으로 분리 된' 방식으로 처리함으로써 '비동기식' 프로그램 개발을 간소화함

#### RxSwift를 사용하면 좋은점

-   Swift에 반응형 프로그래밍을 더해주어, 일관성이 없는 비동기 코드를 하나의 비동기 코드로 개발 가능
-   확장이 불가능한 아키택처 패턴을 해결 가능
-   Thread 처리가 쉬워짐
-   서로 다르게 구현한 로직을 조합하기 쉬워지기 때문에, 콜백 지옥에서 탈출 가능
-   UI 이벤트, 네트워크 처리 등의 데이터를 갱신했을 때의 처리가 쉬워짐, 코드도 깔끔해짐

#### RxSwift 단점

-   러닝커브가 매우 높음
-   클로저 사용이 많기에, 순환 참조 사이클이 일어날 수 있기에 주의 해야함. (\[weak self\] ^^)

#### Observable<T>

-   Rx 코드의 기반이자 심장
-   T형태의 데이터 snapshot을 '전달'할 수 있는 일련의 이벤트를 비동기적으로 생성하는 기능
-   다른 클래스에서 만든 값을 시간에 따라서 읽을 수가 있다
-   하나 이상의 observer가 실시간으로 어떤 이벤트에 반응, 세 가지 유형의 이벤트만 방출
-   Observable = Observable Sequence = Sequence
-   비동기적(asynchronous)
-   Observable들은 일정 기간 동안 계속해서 이벤트를 생성(emit)
-   이벤트는 next, error, completed가 있음

![image-20220329165823558](https://user-images.githubusercontent.com/35060252/177490301-c3b089ae-6172-41fe-8187-5ea9cd762812.png)

-   next라는 이벤트는 T에 해당하는 Element를 전달함
-   completed는 성공적으로 일련의 이벤트들을 종료시키는 것
-   error는 Swift에러를 감싸서 내뱉게 되는데, Observable이 에러를 발생시켜 추가적으로 이벤트를 생성하지 않을 것을 의미, 에러와 함께 Observable은 종료됨
-   tap과 같은 제스처도 element로 사용 가능
-   Observable 생명주기
    -   Observable은 어떤 구성요소(elemnet)를 가지는 next이벤트를 계속해서 방출할 수 있음
    -   Observable은 error이벤트를 방출하여 완전 종료될 수 있음
    -   Observable은 complete 이벤트를 방출하여 완전 종료 될 수 있음

#### Finite Observable

-   **Element을 방출한 뒤, 성공 또는 에러를 통해 종료되는 Observable**
-   파일을 다운로드하는 코드
-   시간에 흐름에 따라서 다운로드 시작

#### Infinite Observable

-   무한한 시퀀스 즉 Observable
-   UI 이벤트는 무한하게 관찰할수 있는 시퀀스

#### Operator

-   Observable의 이벤트를 입력받아 결과로 출력해 내는 연산자
-   다양한 형태로 값을 걸러내거나, 변환하거나, 조합하거나 자기들끼리합치는 그러한 연산자들이 있음
-   주로 비동기 입력을 받아 부수작용 없이 출력만 생성하므로 퍼즐 조각과 같이 쉽게 결합할 수 있음
-   표현식이 최종값으로 배출될때 까지 Observable의 방출한 값에 rx의 연산자를 적용하는것

#### Scheduler

-   우리가 직접 스케줄러를 생성하거나 커스텀할 일은 거의 없음. Rx의 dispatch queue라고 생각하면됨, 하지만 훨씬 강력하고 쓰기 쉬움
-   Dispatch Queue와 동일함 하지만 훨씬 강력하고 쓰기 쉬움
-   자신만의 스케줄러를 생성할 일은 거의 없을 것

## **Observable**

#### just

-   오직 하나의 요소를 포함하는 Observable 시퀀스를 생성

```swift
Observable<Int>.just(1)
```

#### of

-   타입 추론을 통한 Observable 생성

```swift
Observable<Int>.of(1, 2, 3, 4, 5) // 5개의 Int 타입의 element의 이벤트를 생성
​
Observable.of([1, 2, 3, 4, 5])
```

#### from

-   오직 array 형태의 element만 받음

```swift
Observable.from([1, 2, 3, 4, 5])
```

#### subscribe

-   Observable이 이벤트들을 방출하도록 해줄 방아쇠 역할
-   Observable은 실제로는 시퀀스 정의일뿐, 즉 Subscribe(구독) 되기 전에는 아무런 이벤트도 내보내지 않음

```swift
Observable<Int>.just(1)
.subscribe(onNext: {
         print($0)
       })
```

#### empty

-   아무런 element를 방출하지 않음, completed 이벤트만 방출

```swift
Observable.empty() 
.subscribe {
     print($0)
   }
```

#### never

-   아무런 이벤트를 방출하지 않음. Completed 이벤트 조차 방출하지 않음

```swift
Observable.never() 
.subscribe(
     onNext: {
       print($0)
     },
     onCompleted: {
       print("Completed")
     }
   )
```

#### range

-   start 부터 count크기 만큼의 값을 갖는 Observable을 생성

```swift
Observable.range(start: 1, count: 9) // 1부터 9 까지 값을 요소를 이벤트로 방출
.subscribe( onNext: {
     print("2*\($0)= \(2*$)")
   })
```

#### dispose

-   구독(Subscribe)을 처리, 메모리 누수를 막기위해!

```swift
Observable.of(1, 2, 3) 
.subscribe(onNext: {
     print($0)
   })
.dispose() // 구독을 dispose
```

#### disposeBag

-   구독에 대해서 일일히 관리하는 것은 효율적이지 못하기 때문에, RxSwift에서 제공하는 disposedBag 타입을 이용
-   disposeBag에는 disposables를 가지고 있음, disposable은 dispose bag이 할당 해제 하려고 할 때마다 dispose()를 호출

```swift
let disposeBag = DisposeBag()
​
Observable.of(1, 2, 3) 
.subscribe(onNext: {
     print($0)
   })
.disposed(by: disposeBag)
```

#### create

-   Obseravble을 만드는 방법 중 하나
-   create는 escaping 클로저로, escaping에서는 AnyObserver를 취한 뒤 Disposable을 리턴한다.
-   여기서 AnyObserver란 generic 타입으로 Observable sequence에 값을 쉽게 추가할 수 있다

예시1)

```swift
Observable.create { observer -> Disposable in
                  observer.onNext(1)
                  observer.on(.next(1))
                  observer.onCompleted()
                  onberver.onNext(2)
                  return Disposables.create()
}
.subscribe {
 print($0)
}
.disposed(by: disposeBag)
```

예시2)

```swift
enum MyError: Error {
 case anError
}
​
Observable.create { observer -> Disposable in 
                  observer.onNext(1)
                  observer.onError(MyError.anError)
                  observer.onCompleted()
                  observer.onNext(2)
                  return Disposables.create()
}
.subscribe (
 onNext: {
   print($0)
 },
 onError: {
   print($0.localizedDescription)
 },
 onCompleted: {
   print("completed")
 },
 onDisposed: {
   print("disposed")
 }
)
```

#### deferred

-   각 Subscriber에게 새롭게 Observable를 생성해 제공하는 Observable factory (Observable를 감싸는 Observable)

```swift
var 뒤집기: Bool = false
​
let fatory: Observable<String> = Observable.deferred {
뒤집기 = !뒤집기
 
 if 뒤집기 {
   return Observable.of("🤟")
 } else {
   return Observable.of("👌")
 }
}
​
for _ in 0...3 {
 factory.subscribe(onNext: {
   print($0)
 })
 .disposed(by: disposeBag)
}
```

#### Trait

-   Single, Maybe, completable
-   이전의 Observable 보다는 좁은 범위의 Observable, 선택적으로 사용할 수 있음
-   좁은 범위의 Observable를 사용하는 이유는 가독성을 높이는 데 있음

#### Single

-   .success(value) 또는 .error 이벤트를 방출
-   .success(value) = .next + .completed
-   성공 또는 실패로 확인될 수 있는 1회성 프로세스 (예. 데이터 다운로드, 디스크에서 데이터 로딩)
-   정확히 한가지 요소만을 방출하는 Observable에 적합, asSingle로 변경가능

#### Completable

-   .completed 또는 .error 만을 방출하며, 이 외 어떠한 값도 방출하지 않는다.
-   연산이 제대로 완료되었는지만 확인하고 싶을 때 (예. 파일 쓰기)
-   asCompleted는 없다.
-   Observable이 값요소를 방출한 이상 completable로 바꿀수 없다.
-   create를 활용해 만들수 밖에 없음, 어떠한 값도 방출하지 않는다.

#### Maybe

-   Single과 Completable을 섞어놓은 것
-   success(value), .completed, .error를 모두 방출할 수 있다.
-   사용: 프로세스가 성공, 실패 여부와 더불어 출력된 값도 내뱉을 수 있을 때

## **Subject**

**하지만, 보통의 앱개발에서 필요한 것은 실시간으로 Observable에 새로운 값을 수동으로 추가하고, subscriber에 방출하도록 하는것**

#### Subject

-   Observable이자 Observer, 실시간으로 이벤트를 생성하고 구독함

1.  PublishSubject
    -   빈 상태로 시작하여, subscribe 이후의 이벤트만을 subscriber를 통해 방출한다.
2.  BehaviorSubject
    -   subscribe 직전의 하나의 이벤트를 포함한채 subscribe 이후 이벤트들을 subscriber를 통해 방출한다.
3.  ReplaySubject
    -   버퍼를 두고 초기화하며, 버퍼 사이즈 만큼의 직전의 이벤트들을 포함한채 subscribe 이후 이벤트들을 subscriber를 통해 방출한다.
4.  Varaible
    -   BehaviorSubject 를 래핑하고, 현재의 값을 상태로 보존. 가장 최신/초기 값만을 새로운 subscriber에게 방출

- Subject와 Relay의 차이점
	-   Subject는 .completed, .error의 이벤트가 발생하면 subscribe가 종료됨
	-   Relay는 .completed, .error를 발생하지 않고 Dispose되기 전까지 계속 작동하기 때문에 UI Event에서 사용하기 적합
## **Filtering Operator**

#### ignoreElements

-   next 이벤트를 무시함, completed, error 같은 정지이벤트는 허용

```swift
let disposeBage = DisposeBag()
​
let 취침모드 = PublishSubject<String>()
​
취침모드
.ignoreElements()
.subscribe { _ in
             print("햇빛")
   
 }
.diposed(by: disposeBag)
​
취침모드.onNext("알람")
취침모드.onNext("알람")
취침모드.onNext("알람")
```

#### elementAt

-   특정 인덱스에 해당하는 요소만 방출함, 나머지는 무시함

```swift
let 두면울면깨는사람 = PublishSubject<String>()
​
두면울면깨는사람
.element(at: 2)
.subscribe(onNext: {
   print($0)
 })
.disposed(by: diposeBag)
​
두번울면깨는사람("알람")
두번울면깨는사람("알람")
두번울면깨는사람("방긋")
두번울면깨는사람("알람")
​
방긋만 출력됨
```

#### filter

-   Bool 데이터 타입의 파라미터(Bool값을 리턴하는 클로저)에 따라 true일 이벤트 방출

```swift
Observable.of(1, 2, 3, 4, 5, 6, 7, 8)
.filter { $0 % 2 == 0 }
.subscribe(onNext: {
   print($0)     // 2 4 6 8 만 로그 찍힘
 })
.diposed(by: diposeBag)
```

#### skip

-   첫번째 요소를 기준으로 몇개의 요소를 스킵할건지에 대한 연산자

```swift
Observable.of(1, 2, 3, 4, 5, 6, 7, 8)
.skip(5)
.subscribe(onNext: {
   print($0)
 })
.diseposed(by: diposeBag)
​
로그는 다음과 같이 찍힘
6
7
8
```

#### skipWhile

-   while 클로저 안의 로직이 true일때 까지 무시하게됨

```swift
Observable.of(1, 2, 3, 4, 5, 6, 7, 8)
.skip(while: {
   $0 != 6
 })
.subscribe(onNext: {
   print($0)
 })
.diseposed(by: diposeBag)
​
로그는 다음과 같이 찍힘
6
7
8
```

#### skipUntil

-   이전의 로직은 고정 조건에서 이루어 졌지만, 다른 Observable에 기반한 요소들을 다이나믹하게 필터하고 싶으면 skipUntil 사용
-   기준이 되는 Observable이 이벤트를 나타내기 전까지 요소들을 무시함

```swift
let 손님 = PublishSubject<String>()
let 문여는시간 = PublishSubject<String>()
​
손님 // 현재 Observable
.skip(until: 문여는 시간) // 다른 Observable
.subscribe(onNext: {
   print($0)
 })
​
손님.onNext("1")
손님.onNext("1")
​
문여는시간.onNext("떙!")
손님.onNext("2")
​
로그는 다음과 같이 찍힘
2
```

#### take

-   첫번째 요소를 기준으로 몇개의 요소를 나타날건지에 대한 연산자 (skip 연산자와 반대)

```swift
Observable.of("1", "2", "3", "4", "5")
.take(3)
.subscribe(onNext: {
   print($0)
 })
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘
1
2
3
```

#### takeWhile

-   while 구문 내에서 true일 때까지 방출하게됨 (skipWhile 연산자와 반대)

```swift
Observable.of("1", "2", "3", "4", "5")
.take(while: {
$0 != "3"
})
.subscribe(onNext: {
   print($0)
 })
.disposed(by: disposeBag)

로그는 다음과 같이 찍힘
1
2
```

#### enumerated

-   방출된 요소의 index를 참고하고 싶을때 사용

```swift
Observable.of("1", "2", "3", "4", "5")
.enumerated()
.takeWhile {
   $0.index < 3
 }
.subscribe(onNext: {
   print($0)
 })
.disposed(by: disposeBag)
```

#### takeUntil

-   이전의 로직은 고정 조건에서 이루어 졌지만, 다른 Observable에 기반한 요소들을 다이나믹하게 필터하고 싶으면 takeUntil 사용
-   기준이 되는 Observable이 이벤트를 나타내기 전까지 요소들을 나타냄

```swift
let 수강신청 = PublishSubject<String>()
let 신청마감 = PublishSubject<String>()
​
수강신청 // 현재 Observable
.take(until: 신청마감) // 다른 Observable
.subscribe(onNext: {
   print($0)
 })
​
수강신청.onNext("1")
수강신청.onNext("2") // 여기 까지만 방출함
​
신청마감.onNext("끝!")
수강신청.onNext("3") // 여기 부터는 무시됨
```

#### distincUntilChanged

-   연달아 같은 요소가 이어질때 중복된 방출을 막아주는 역할

```swift
Observable.of("저는", "저는", "앵무새", "앵무새", "앵무새", "앵무새", "입니다", "입니다", "입니다", "입니다", "저는", "앵무새", "일까요?", "일까요?")
.distinctUntilChanged()
.subscribe(onNext: {
   print($0) // 저는 앵무새 입니다 저는 앵무새 일까요 (\n생략)
 })
```

## **Transforming Operator**

#### toArray

-   Observable의 독립적 요소들을 array로 만드는 연산자 (Singe<\[T\]> 형태로 변환됨)

```swift
Observable.of("A", "B", "C")
.toArray()
.subscribe(onNext: {
   print($0) 
 })
.disposed(by: disposeBag)
```

#### map

-   요소를 원하는 타입의 데이터로 변환해 주는 연산자

```swift
Observable.of(Date())
.map { date -> String in
   let dateFormatter = DateFormatter()
   dateFormatter.dateFormate = "yyyy-MM-dd"
   dateFormatter.local = Locale(identifier: "ko_KR")
   return dateFormatter.string(from: date)
 }
.subscribe(onNext: {
   print($0)
 })
.disposed(by: disposeBag)
```

#### flatMap

-   Observable 내부의 Observable를 모두 같은 위상으로 평평하게 펼쳐주는 것
-   반환과정은 Observable<Observable<T>> -> Observable<T>

```swift
protocol 선수 {
  var 점수: BehaviorSubject<Int> { get }
}

struct 양궁선수: 선수 {
  var 점수: BehaviorSubject<Int>
}

let 한국국가대표 = 양궁선수(점수: BehaviorSubject<Int>(value: 10))
let 미국국가대표 = 양궁선수(점수: BehaviorSubject<Int>(value: 8))

let 올림픽경기 = PublishSubject<선수>()

올림픽경기
	.flatMap { 선수 in
    선수.점수
  }
	.subscribe(onNext: {
    print($0)
  })
	.disposed(by: disposeBag)

올림픽경기.onNext(한국국가대표)
한국국가대표.점수.onNext(10)

올림픽경기.onNext(미국국가대표)
한국국가대표.점수.onNext(10)
미국국가대표.점수.onNext(9)

로그는 다음과 같이 찍힘
10
10
8
10
9
```

#### flatMapLatest

-   시퀀스 내부의 시퀀스 중 가장 최근에 전환된 시퀀스에서 나온 값만 반영.
-   Target observable의 결과값으로는 오직 가장 최근의 observable에서 나온 값만 받게 된다

```swift
protocol 선수 {
 var 점수: BehaviorSubject<Int> { get }
}
​
struct 높이뛰기선수: 선수 {
 var 점수: BehaviorSubject<Int>
}
​
let 서울 = 높이뛰기선수(점수: BehaviorSubject<Int>(value: 7))
let 제주 = 높이뛰기선수(점수: BehaviorSubject<Int>(value: 6))
​
let 전국체전 = publishSubject<선수>()
​
전국체전
.flatMapLatest { 선수 in   // 가장 최신의 시퀀스만 반영함
  선수.점수
 }
.subscribe(onNext: {
   print($0)
 })
.disposed(by: disposeBag)
​
전국체전.onNext(서울) // 이 시점 최신 시퀀스
서울.점수.onNext(9)
​
전국체전.onNext(제주) // 이 시점 최신 시퀀스
서울.점수.onNext(10) // 서울 시퀀스는 무시됨
제줄.점수.onNext(8)
```

#### meterialize

-   단순히 요소만이 아니라 요소를 포함한 이벤트로 받음

#### dematerialize

-   요소를 포함한 이벤트를 다시 요소로 받음

```swift
enum 반칙: Error {
 case 부정출발
}
​
struct 달리기선수: 선수 {
 var 점수: BehaviorSubject<Int>
}
​
let 김토끼 = 달리기선수(점수: BehaviorSubject<Int>(value: 0))
let 박치타 = 달리기선수(점수: BehaviorSubject<Int>(value: 1))
​
let 달리기100M = BehaviorSubject<선수>(value: 김토끼) // 시퀀스 내부 첫 시퀀스는 김토끼
​
달리기100M
.flatMapLatest { 선수 in   
  선수.점수
.materialize()
 }
.filter {
   guard let error = $0.error else {
     return true // 에러가 없을 때만 통과
   }
   print(error)  // 에러 로그 찍어주고
   return false  // 에러가 없을 때는 패스
 } 
.dematerialize()
.subscribe(onNext: {
   print($0)
 })
.disposed(by: disposeBag)
​
김토끼.점수.onNext(1)
김토끼.점수.onError(반칙.부정출발)
김토끼.점수.onNext(2)
​
달리기100M.onNext(박치타)
```

#### 전화번호 11자리 연습

```swift
let input = PublishSubject<Int?>()
​
let list: [Int] = [1]
​
imput
.flatMap {
   $0 == nil 
   ? Observable.empty()
   ? Observable.just($0)
 }
.map { $0! }
.skip(while: { $0 != 0 })
.take(11)
.toArray()
.asObservable()
.map {
   $0.map { "\($0)" }
 }
.map { numbers in
   var numberList = numbers
       numberList.inert("-", at: 3) // 010-
       numberList.inert("-", at: 8) // 010-1234-
       let number = numberList.reduce(" ", +)
       return number
 }
.subscribe(onNext: {
   print($0)
 })
.disposed(by: disposeBag)
​
input.onNext(10)
input.onNext(0)
input.onNext(nil)
input.onNext(1)
input.onNext(0)
input.onNext(4)
input.onNext(3)
input.onNext(nil)
input.onNext(1)
input.onNext(8)
input.onNext(9)
input.onNext(4)
input.onNext(9)
input.onNext(1)
```

원래의 값은 변화시키지 않으면서 연산자에 따른 결과값만 변화시키기 때문에 좋다

## **Combinging Operator**

#### startWith

-   Observable 시퀀스에 초기값을 앞에 붙임

```swift
let 노랑반 = Observable.of("학생1","학생2","학생3")
​
노랑반
.starWith("선생님")
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘
선생님
학생1
학생2
학생3
```

#### concat

-   같은 데이터 타입의 요소를 갖는 두개의 Observable들을 묶을 때 사용

```swift
let 모바일팀원들 = Observable<String>.of("팀원1","팀원2","팀원3")
let 팀장님 = Observable<String>.of("팀장님")
​
let 줄서서걷기 = Observable
.concat([팀장님, 모바일팀원들])
​
줄서서걷기
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘
팀장님
팀원1
팀원2
팀원3
```

```swift
팀장님
.concat(모바일팀원들)
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘
팀장님
팀원1
팀원2
팀원3
```

#### concatMap

-   각각의 시퀀스가 다음 스퀀스가 구독되기 전에 합쳐짐을 보증

```swift
let 학교: [String: Observable<String>] = [
 "1반": Observable.of("학생1","학생2","학생3"),
 "2반": Observable.of("학생4","학생5")
]
​
Observable.of("1반","2반")
.concatMap { 반 in
  학교[반] ?? .empty()
 }
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘
학생1
학생2
학생3
학생4
학생5
```

#### merge

-   sequence들을 합치는 방법 중 하나

```swift
let 강북 = Observable.from(["강북구", "성북구", "동대문구", "종로구"])
let 강남 = Observable.from(["강남구", "강동구", "영등포구", "양천구"])
​
Observable.of(강북, 강남)
.merge()
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘  // 순서를 보장하지 않고 로그가 찍힘
강북구
성북구
강남구
동대문구
강동구
종로구
영등포구
양천구
```

```swift
Observable.of(강북, 강남)
.merge(maxConcurrent: 1) // maxConcurrent: 한번에 받아낼 Observable의 수, 네트워크 요청이 많아질때 리소스나 연결수를 제한할때 사용할 가능성있음.
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘
// 강북을 먼저 받아냈으면 강북 먼저 찍히고, 강남을 먼저 받아냈으면 강남 먼저 찍힘
```

#### combineLatest

-   combine(결합)된 Observable들은 값을 방출할 때마다, 제공한 클로저를 호출하며 우리는 각각의 내부 Observable들의 최종값을 받음
-   여러 TextField를 한번에 관찰하고 값을 결합하거나 여러 소스들의 상태들을 보는 것과 같은 app이 있음

```swift
let 성 = PublishSubject<String>()
let 이름 = PublishSubject<String>()
​
let 성명 = Observable
.combineLatest(성, 이름) { 성, 이름 in
    성 + 이름
   }
​
성명
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
성.onNext("김")
이름.onNext("똘똘")
이름.onNext("영수")
이름.oNext("은영")
성.onNext("박")
성.onNext("이")
성.onNext("조")
```

```swift
let 날짜표시형식 = Observable<DateFormatter.Style>.of(.short, .long)
let 현재날짜 = Observable<Date>.of(Date())
​
let 현재날짜표시 = Observable
.combineLatest(
  날짜표시형식,
  현재날짜,
   resultSelector: { 형식, 날짜 -> String in
     let dateFormatter = DateFormatter()
     dateFromatter.dateStyle = 형식
return dateFormatter.string(from: 날짜)
   }
 )
​
현재날짜표시
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘
9/12/21
September 12, 2021
```

```swift
let lastName = PublishSubject<String>() // 성
let firstName = PublishSubject<String>() // 이름
​
let fullName = Observable
.combineLatest([firstName, lastName]) { name in // array 형태의 combineLast 존재 
     name.joined(separator: " ")
   }
​
fullName
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
lastName.onNext("Kim")
fistName.onNext("Paul")
fistName.onNext("Stella")
fistName.onNext("Lily")
​
로그는 다음과 같이 찍힘
Paul Kim
Stella Kim
Lilly Kim
```

#### zip

-   결합을 원하는 각각의 시퀀스들의 요소들을 순차적으로 결합함
-   둘중 하나의 Observable이 완료되면 zip에대한 Observable은 종료함

```swift
enum 승패 {
 case 승
 case 패
}
​
let 승부 = Observable<승패>.of(.승, .승, .패, .승, .패)
let 선수 = Observable<String>.of("🇰🇷", "🇩🇪", "🇪🇸", "🇺🇸", "🇳🇴", "🇬🇧")
​
let 시합결과 = Observable
.zip(승부, 선수) { 결과, 대표선수 in
   return 대표선수 + "선수" + "\(결과)"
 }
​
시합결과
.subscribe(onNext: {
   print($0)
 })
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘
🇰🇷선수 승
🇩🇪선수 승
🇪🇸선수 패
🇺🇸선수 승
🇳🇴선수 패
​
!🇬🇧는 안찍힘!
```

#### withLatestFrom

-   withLatestFrom을 호출한 Observable은 onNext하면 withLatestFrom의 파라미터인 Observable의 최신값을 trigger함

```swift
let 🔫 = PublishSubject<Void>()
let 달리기선수 = PublishSubject<String>()
​
🔫
.withLatestFrom(달리기선수)
.subscribe(onNext: {
   print($0)
 })
.disposed(by: disposeBag)
​
달리기선수.onNext("🏃")
달리기선수.onNext("🏃", "🏃")
달리기선수.onNext("🏃", "🏃", "🏃🏻")
​
🔫.onNext(Void())
🔫.onNext(Void())
​
로그는 다음과 같이 찍힘
"🏃", "🏃", "🏃🏻"
"🏃", "🏃", "🏃🏻"
```

#### sample

-   withLatestFrom 처럼 trigger 역할을 하지만 중복된 항목의 경우 방출하지 않음

```swift
let 출발 = PublishSubject<Void>()
let F1선수 = PublishSubject<String>()
​
F1선수
.sample(출발)
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
F1선수.onNext("🏎")
F1선수.onNext("🏎 🚗")
F1선수.onNext("🏎     🚗 🚙")
​
출발.onNext(Void())
출발.onNext(Void())
출발.onNext(Void())
​
로그는 다음과 같이 찍힘
🏎     🚗 🚙
​
/*
withLatestFrom로 sample처럼 한번만 trigger하게 하려면 distinctUntilChanged을 withLatestFrom연산자 뒤에 적어주면된다.
ex)
.withLatestFrom(F1선수)
.distinctUntilChanged()
*/
```

#### amb

-   두가지 시퀀스를 받을 때, 두가지 시퀀스 중 어떤것을 구독할 지 애매모호 할 때 사용하는 방식이라는데, amb에 대한 두가지 Observable중 먼저 element를 방출하는 Observable만 구독하고 나머지 ObserVable은 무시됨.

```swift
let 버스1 = PublishSubject<String>()
let 버스2 = PublishSubject<String>()
​
let 버스정류장 = 버스1.amb(버스2)
​
버스정류장
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
버스2.onNext("버스2-승객0: 사람1")
버스1.onNext("버스1-승객0: 사람2")
버스1.onNext("버스1-승객1: 사람3")
버스2.onNext("버스2-승객1: 사람4")
버스1.onNext("버스1-승객1: 사람5")
버스2.onNext("버스2-승객2: 사람6")
​
로그는 다음과 같이 찍힘
버스2-승객0: 사람1
버스2-승객1: 사람4
버스2-승객2: 사람6
```

#### switchLatest

-   SourceObservable로 들어온 마지막 시퀀스만 구독하는 방식

```swift
let 학생1 = PublishSubject<String>()
let 학생2 = PublishSubject<String>()
let 학생3 = PublishSubject<String>()
​
let 손들기 = PublishSubject<Observable<String>>() // SourceObservable
​
let 손든사람만말할수있는교실 = 손들기.switchLatest()
​
손든사람만말할수있는교실
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
손들기.onNext(학생1)
학생1.onNext("학생1: 저는 1번 학생입니다.")
학생2.onNext("학생2: 저요 저요!!!")
​
손들기.onNext(학생2)
학생2.onNext("학생2: 저는 2번이에요!")
학생1.onNext("학생1: 아.. 나 아직 할말 있는데")
​
손들기.onNext(학생3)
학생2.onNext("학생2: 아니 잠깐만! 내가!")
학생1.onNext("학생1: 언제 말할 수 있죠")
학생3.onNext("학생3: 저는 3번 입니다~ 아무래도 제가 이긴 것 같네요.")
​
손들기.onNext(학생1)
학생1.onNext("학생1: 아니, 틀렸어, 승자는 나야.")
학생2.onNext("학생2: ㅠㅠ")
학생3.onNext("학생3: 이긴 줄 알았는데")
학생2.onNext("학생2: 이거 이기고 지는 손들기였나요?")
​
로그는 다음과 같이 찍힘
학생1: 저는 1번 학생입니다.
학생2: 저는 2번이에요!
학생3: 저는 3번 입니다~ 아무래도 제가 이긴 것 같네요.
학생1: 아니, 틀렸어, 승자는 나야.
```

#### reduce

-   제공된 초기값(예제에서는 0)부터 시작해서 source observable이 값을 방출할 때마다 그 값을 가공함 (swift 기본 문법 reduce와 동일)

```swift
Observable.from([1...10])
.reduce(0, accumlator: { summary, newValue in
     return summary + newValue
   })
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘
55
```

#### scan

-   reduce의 경우, 결과값만을 방출하지만, scan은 매번 값이 들어올때 마다 결과값을 방출하게 됨

```swift
Observable.from([1...10])
.scan(0, accumlator: { summary, newValue in
     return summary + newValue
   })
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘
1
3
6
10
15
21
28
36
45
55
```

## **TimeBased Operator** 

#### replay

-   구독자가 과거의 요소들을 자신이 구독하기 전에 나왔던 이벤트들을 버퍼의 갯수만큼 최신 순서대로 받게 한다.

```swift
let 인사말 = PublishSubject<String>()
let 반복하는앵무새 = 인사말.replay(1)
반복하는앵무새.connect()
​
인사말.onNext("1. hello")
인사말.onNext("2. hi")
​
반복하는앵무새
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
인사말.onNext("3. 안녕하세요.")
​
로그는 다음과 같이 찍힘
2. hi
3. 안녕하세요
```

#### replayAll

-   구독자가 과거의 요소들을 자신이 구독하기 전에 나왔던 이벤트들을 무제한으로 받게 한다.

```swift
let 닥터스트레인지 = PublishSubject<String>()
let 타임스톤 = 닥터스트레인지.replayAll()
타임스톤.connect()
​
닥터스트레인지.onNext("도르마무")
닥터스트레인지.onNext("거래를 하러 왔다.")
​
타임스톤
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘
도르마무
거래를 하러 왔다.
```

#### buffer

-   이벤트를 번들로 한번에 묶어서 묶음(Array)으로 방출
-   **timeSpan**은 항목을 수집하는 시간, **count**는 최대 몇개까지의 요소를 담을지, **scheduler**는 해당 연사자가 실행될 쓰레드를 결정

```swift
let source = PublishSubject<String>()
​
var count = 0
let timer = DispatchSource.makeTimerSource()
​
timer.schedule(deadline: .now() + 2, repeating: .seconds(1))
timer.setEventHandler {
 count += 1
 source.onNext("\(count)")
}
timer.resume()
​
source
.buffer(
     timeSpan: .seconds(2),
     count: 2,
     scheduler: MainScheduler.instance
   )
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
​
로그는 다음과 같이 찍힘 // 타이밍에 따라 바뀔 수도 있음
["1"]
["2","3"]
["4","5"]
```

#### window

-   Buffer와 달리 **묶음(Array)이 아닌 Observable 하나씩 방출해줌**

```swift
let 만들어낼최대Observable수 = 1
let 만들시간 = RxTimerInterval.seconds(2)
​
let window = PublishSubject<String>()
​
let windowCount = 0 
let windowTimerSource = DispatchSource.makeTimerSource()
windowTimerSource.schedule(deadline: now() + 2, repeating: .seconds(1))
windowTimerSource.setEventHandler {
 windowCount += 1
 window.onNext("\(windowCount)")
}
windowTimerSource.resume()
​
window
.window(
     timeSpan: 만들시간,
     count: 만들어낼최대Observable수,
     schedule: MainScheduler.instance
   )
.flatMap { windowObservable -> Observable<(index: Int, element: String)> in
             retrun windowObservable.enumerated()
   }
.subscribe(onNext: {
   print("\($0.index)번째 Observable의 요소 \($0.element)")
})
.disposed(by: disposeBag)
​
```

#### delaySubscription

-   구독을 지연하는 연산자

```swift
let delaySource = PublishSubject<String>()
​
var delayCount = 0
let delayTImeSource = DispatchSource.makeTimerSource()
delayTimeSource.schedule(deadline: .now() + 2, repeating: .seconds(1))
delayTimeSource.setEventHandler {
 delayCount += 1
 delayCount.onNext("\(delayCount)")
}
delayTimeSource.resume()
​
delaySource
.delaySubscription(.second(2), scheduler: MainScheduler.instance)
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
```

#### delay

-   시퀀스를 지연하는 연산자

```swift
let delaySubject = PublishSubject<Int>()
​
var delayCount = 0
let delayTimerSource = DispatchSourec.makeTimerSource()
delayTimerSource.schedule(deadline: .now(), repeating: .seconds(1))
delayTimerSource.setEventHandler {
 delayCount += 1
 delaysubject.onNext(delayCount)
}
delayTimerSource.resume()
​
delaySubject
.delay(.seconds(3), scheduler: MainScheduler.instance)
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
```

#### cold observable

-   요소를 등록할때, 방출이 시작
-   구독할 때만 이벤트 방출
-   구독을 지연시켰을 때, 지연에 따른 차이가 없음

#### hot observable

-   어떤 시점에서부터 영구적으로 작동하는 것
-   구독과 관계없이 이벤트를 방출
-   구독을 지연시켰을 때, 일정 요소를 건너뛰게 됨

#### interval

-   지정한 시간에 따라 이벤트를 방출 시켜주는 연산자

```swift
Observable<Int>
.interval(.seconds(3), scheduler: MainScheduler.instance)
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
```

#### timer

-   **dueTime**을 통해 구독을 시작하기까지의 딜레이값, **period**는 이벤트가 방출되는 간격

```swift
Observable<Int>
.timer(
     .seconds(5),
     period: .seconds(2),
     scheduler: MainScheduler.instance
   )
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
```

#### timeout

-   **dueTime** 시간내에 어떠한 이벤트도 방출하지 않았을때, 에러를 방출함

```swift
let 누르지않으면에러 = UIButton(type: .system)
누르지않으면에러.setTile("눌러주세요!", for: .normal)
누르지않으면에러.sizeToFit()
​
PlaygroundPage.current.liveView = 누르지않으면에러
​
누르지않으면에러.rx.tap
.do(onNext: {
     print("tap")
   })
.timeout(.seconds(5), scheduler: MainScheduler.instance)
.subscribe(onNext: {
   print($0)
})
.disposed(by: disposeBag)
```

## **Error Handling**

![스크린샷 2022-07-05 오후 4 39 28](https://user-images.githubusercontent.com/35060252/177490616-24af21c8-e065-4dad-b613-ed266a07c641.png)

#### catch

-   에러가 발생했을 때, Error 이벤트로 종료되지 않게 한다
-   Error 이벤트 대신 특정 값의 이벤트를 발생시키고 complete 시킨다
-   관련메소드
    1.  **catchError**
        
        -   Error를 다른 타입의 Observable 로 반환하는 클로저를 parameter로 받음
        -   Error가 발생했을 때 Error를 무시하고 클로저의 반환값(Observable<E>)을 반환
        
        ```swift
        let observable = Observable<Int>
           .create { observer -> Disposable in
               observer.onNext(1)
               observer.onNext(2)
               observer.onNext(3)
               observer.onError(NSError(domain: "", code: 100, userInfo: nil))
               observer.onError(NSError(domain: "", code: 200, userInfo: nil))
               return Disposables.create { }
        }
        ​
        observable
           .catchError { .just(($0 as NSError).code) }
           .subscribe { print($0) }
           .disposed(by: disposeBag)
        ​
        로그는 다음과 같이 찍힘
        next(1)
        next(2)
        next(3)
        next(100)
        completed
        ```
        
    2.  **catchErrorJustReturn**
        
        -   Error 가 발생했을 때 Error 를 무시하고 element를 반환
        -   모든 에러에 동일한 값이 반환되기 때문에 catchError 에 비해 제한적
        
        ```swift
        let observable = Observable<Int>
           .create { observer -> Disposable in
               observer.onNext(1)
               observer.onNext(2)
               observer.onNext(3)
               observer.onError(NSError(domain: "", code: 100, userInfo: nil))
               observer.onError(NSError(domain: "", code: 200, userInfo: nil))
               return Disposables.create { }
        }
        ​
        observable
           .catchErrorJustReturn(999)
           .subscribe { print($0) }
           .disposed(by: disposeBag)
        ​
        로그는 다음과 같이 찍힘
        next(1)
        next(2)
        next(3)
        next(999)
        completed
        ```
        

#### retry

-   에러가 발생 했을 때 다시 시도할 수 있게 해줌
-   에러가 발생했을 때 Observable 을 다시 시도
-   관련 메소드
    1.  **retry()**
        
        -   에러가 발생했을 때 성공할 때까지 Observable을 다시 시도
        
        ```swift
        let reloadPublisher = PublishSubject<Void>()
        ​
        reloadPublisher
         .flatMap {
           Api.getRepositories()
             .retry()
         }
        ```
        
    2.  **retry(\_ maxAttemptCount: Int)**
        
        -   몇 번에 걸쳐서 재시도 할지 지정할 수 있는 연산자
        -   maxAttemptCount 가 3 이라면 총 3번의 요청을 보냄 (재시도는 2번)
        -   재시도 횟수가 넘어가면 그대로 Error를 이벤트로 전달
        
        ```swift
        let reloadPublisher = PublishSubject<Void>()
        ​
        reloadPublisher
         .flatMap {
           Api.getRepositories()
             .retry(3)
         }
        ```
        
    3.  **retryWhen**
        
        -   재시도 하는 시점을 지정할 수 있고, 한번만 수행함
        -   retry 와 다르게 마지막 Error를 이벤트로 전달하지 않음
        
        ```swift
        let observable = Observable<Int>
           .create { observer -> Disposable in
               observer.onNext(1)
               observer.onNext(2)
               observer.onNext(3)
               observer.onError(NSError(domain: "", code: 100, userInfo: nil))
               observer.onError(NSError(domain: "", code: 200, userInfo: nil))
               return Disposables.create { }
        }
        ​
        observable
           .retryWhen { err -> Observable<Int> in
               return .timer(3, scheduler: MainScheduler.instance)
           }
           .subscribe { print($0) }
           .disposed(by: disposeBag)
        ​
        로그는 다음과 같이 찍힘
        next(1)
        next(2)
        next(3)
        (3 seconds)
        next(1)
        next(2)
        next(3)
        completed
        ```

## **RxCocoa**

-   iOS의 Cocoa Framework를 Rx스럽게 사용할 수 있도록 Rx로 감싼 프레임워크 (Cocoa Framework를 wrapping했음)

#### ObserverType

-   해당 타입에 값을 주입시킬 수 있음

#### ObservableType

-   해당 타입의 값을 관찰할 수 있음

#### ControlProperty

-   Subject와 같이 프로퍼티에 새 값을 주입시킬 수 있음 (ObserverType)
-   값의 변화도 관찰할 수 있음(ObservableType)
-   ControlPropertyType을 준수함 (ControlPropertyType은 ObserverType과 ObservableType을 준수함)
-   ex) UITextField+Rx.Swift의 text(ControlPropery 프로퍼티)는 프로퍼티에 새값을 주입시킬 수 있고 값의 변화도 관찰할 수 있음

#### Binder

![스크린샷 2022-07-06 오후 1 24 03](https://user-images.githubusercontent.com/35060252/177490723-c0c52e8c-1704-4378-b58c-8944687f12a8.png)

-   ObserverType을 준수함, 따라서 값을 생성해내고 주입할 수는 있으나 관찰할 수는 없음
-   error 이벤트를 방출할 수 없음
-   RxCocoa에서 binding은 Publisher에서 Subscriber로 향하는 단방향 binding임
-   bind(to:)메소드는 메인스레드 실행을 보장함
-   bind(to: observer)를 호출하게 되면 subscribe(observer)가 실행됨
-   binding 작업을 언제나 메인 스레드에서 실행해주기에, 쓰레드에 대한 관리를 해 줄 필요가 없음
-   ex) UILabel+Rx.Swift에서 text Binder 프로퍼티는 값을 주입만 시킬 수 있음

이 코드를

```swift
textField.rx.text
	.observe(on: MainScheduler.instance)
    .subscribe(onNext: {
    	label.text = $0
    })
    .disposed(by: disposeBag)
```

이렇게 변경가능

```swift
textField.rx.text
	.bind(to: label.rx.text)
    .disposed(by: disposeBag)
```

#### Traits

-   UI처리에 특화된 Observable(UI작업시 코드를 쉽고 직관적으로 작성해 사용할 수 있도록 도와주는 특별한 Observable클래스 모음)
-   error를 방출하지 않음
-   메인 스케줄러에서 observe or subscribe됨
-   Signal을 제외한 나머지 Traits들은 모든 구독자에 대해 동일한 시퀀스를 공유(share연산자가 내부적으로 사용된 상태)
-   종류
    1.  **ControlProperty**
    	- 컨트롤에 data를 binding하기 위해 사용
		- Subject와 같이 프로퍼티에 새 값을 주입시킬 수 있음 (ObserverType), 값의 변화도 관찰할 수 있음(ObservableType)
		- ControlPropertyType을 준수함 (ControlPropertyType은 ObserverType과 ObservableType을 준수함)
        - ex.UITextField+Rx.Swift의 text(ControlPropery 프로퍼티)는 프로퍼티에 새값을 주입시킬 수 있고 값의 변화도 관찰할 수 있음
		
	
    2.  **ControlEvent**
        -   event(버튼 tap같은)를 Observable로 래핑한 속성
        -   Observable의 역할은 수행하지만, ControlProperty와는 다르게 Observer의 역할은 수행하지 못함
        -   control이 해제될 경우 Complete이벤트 방출
        -   **컨트롤의 event를 수신하기 위해 사용**

```swift
// UIButton+Rx.swift
extension Reactive where Base: UIButton {
    
    /// Reactive wrapper for `TouchUpInside` control event.
    public var tap: ControlEvent<Void> {
        return controlEvent(.touchUpInside)
    }
}
```

-   3.  **Driver**
        -   Observable을 Driver로 바꿔서 사용가능 
            -   asDriver(onErrorDriverWith:)
                -   error를 수동적으로 리턴하여, error에 이벤트를 handle할 수 있음
            -   asDriver(onErrorRecover:)
                -   driver에 사용되며 error에 대한 이벤트를 handle할 수 있음
            -   asDriver(onErrorJustReturn:) 
                -   Observable에서 error가 방출됐을때 Driver에서 error 대신 지정한 기본 값을 리턴하도록 만들어 Driver에서 error가 방출 되는 것을 막음
    4.  **Signal**
        -   Driver와 거의 동일하나 자원을 공유하지 않음 (Signal은 event모델링에 유용, Driver는 state모델링에 더 적합)

**Driver와 ControlPropery의 사용**

```swift
let search = myTextField.rx.text.orEmpty
		.filter { !$0.imEmpty }
        .flatMapLatest { text in
        return ApiController.shared.currentWeather(city: text)
        .catchErrorJustReture(ApiController.Weather.empty)
        }
        .asDriver(onErrorJustReturn: ApiController.weather.empty)
```
<br/><br/><br/>

- 출처 및 참고 - [https://ios-development.tistory.com/143](https://ios-development.tistory.com/143)
- 출처 및 참고 - [https://okanghoon.medium.com/rxswift-5-error-handling-example-9f15176d11f](https://okanghoon.medium.com/rxswift-5-error-handling-example-9f15176d11f)
- 출처 및 참고 - [https://github.com/fimuxd/RxSwift#Contributors](https://github.com/fimuxd/RxSwift#Contributors)
- 출처 및 참고 - [https://fastcampus.co.kr/dev\_online\_iosappfinal](https://fastcampus.co.kr/dev_online_iosappfinal)
- 출처 및 참고 - [https://duwjdtn11.tistory.com/628](https://duwjdtn11.tistory.com/628)
