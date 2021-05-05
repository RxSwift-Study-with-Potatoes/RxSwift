### RXSwift

ReactiveX 라이브러리를 스위프트 언어로 구현한 것

이를 통해서 observable sequence를 사용하여 비동기 및 이벤트 기반 프로그램을 작성할 수 있음

## Observable

+ RxSwift에서 가장 핵심적인 개념

+ Observable(=Observable sequence / sequence)

  이벤트를 Observer에게 전달함

+ Observer(옵저버)

  Observable을 구독(subscribe)함.

  구독(subscribe)하고 있다가 Observable이 이벤트를 보내면 전달된 이벤트를 처리함
  
  동시에 두개 이상의 이벤트를 처리하지 않음
  
  실제로 이벤트가 전달되는 시점은 Observer이 구독을 시작하는 시점

 즉, Observable이 이벤트를 발생시키면 Observer의 관찰자가 그 순간을 감지하고 준비된     연산을 실행시켜 결과를 리턴하는 메커니즘

## observable의 생명주기

+ next 이벤트

  Observable에서 발생한 새로운 이벤트는 next이벤트를 통해서 Observer에게 전달됨(=emission이라고 함)

  이벤트에 값이 포함되어 있다면 next이벤트와 함께 전달됨

+ error 이벤트

  Observable에서 error가 발생하면 Observer에게 error이벤트가 전달됨(= Notification이라고 함)

  Observable의 life cycle에서 가장 마지막에 전달되는 이벤트임

  Observable은 error이벤트를 방출(Notification)하여 완전 종료될 수 있음

+ completed 이벤트

  Observable이 정상적인 종료를 하게 되면 Observer에게 completed이벤트가 전달됨(= Notification이라고 함)

  Observable의 life cycle에서 가장 마지막에 전달되는 이벤트임
  
  Observable은 completed이벤트를 방출(Notification)하여 완전 종료될 수 있음

## disposable - 리소스 정리에 사용되는 객체
1. dispose()를 직접 호출함

   ```swift
   let o1=Observable.from([0,1])
   .subscribe{
     print($0)
   }
   
   o1.dispose() //dispose()를 호출함으로써 메모리에서 리소스를 해제시킬 수 있음
   ```

   

2. dispose Bag을 사용함

   ```swift
   var bag = DisposeBag()
   
   Observable.from([0,1])
   .subscribe{
     print($0)
   }
   .disposed(by: bag) //이런식으로 disposed(by: )메서드를 사용함
   ```

   

## operator(연산자) 

+ 대부분의 연산자는 Observable타입을 리턴함. 따라서 두 개 이상의 연산자를 연달아서 호출하는 것도 가능함 , 보통은 subscribe메서드 앞에 추가함

## Combining operator(결합 연산자)

여러 개의 소스 Observable들을 하나의 Observable로 만드는 연산자들

1. concat 연산자
<img width="500" height="200" alt="concat" src="https://user-images.githubusercontent.com/70764912/117155825-14f49500-adf8-11eb-9205-94964bd603df.png">

   두 개의 옵저버블을 결합함

   타입 메서드와 인스턴스 메서드로 2가지로 구현할 수 있음

   타입 메서드로 구현 : 파라미터로 전달된 컬렉션에 있는 모든 Observable을 순서대로 연결한 하나의 Observable을 리턴함

   인스턴스 메서드로 구현 :

   completed이벤트는 하나로 연결된 리턴되는 Observable이 모든 요소를 방출하면 한번 옵저버에게 전달됨.

   ```swift
   let Dispose = DisposeBag()
   let one = Observable.from(["H","e","o"]) //from 연산자로 Observable 생성함
   let two = Observable.from([100,110,120]) //from 연산자로 Observable 생성함
   
   //타입 메서드로 구현(1)
   Observable.concat([one,two])
   .subscribe{print($0)}
   .disposed(by:Dispose)
   
   //인스턴스 메서드로 구현(2)
   one.concat(two)
   .subscribe{print($0)}
   .disposed(by:Dispose)
   
   ```

   ```swift
   next(H)
   next(e)
   next(o)
   next(100)
   next(110)
   next(120)
   completed
   ```

   

2. startWith 연산자
<img width="500" height="200" alt="startwith" src="https://user-images.githubusercontent.com/70764912/117161703-1d030380-adfd-11eb-8e9f-a39bb7434783.png">
   Observable이 항목을 배출하기 전에 다른 항목들을 앞에 추가하는 연산자

   주로 기본값이나 시작값을 저장할때 활용함 , last in first out 방식임

   ```swift
   let Dispose = DisposeBag()
   
   Observable.from([100,200,300])
   .startsWith(0)
   .startsWith(1)
   .startsWith(2)
   .subscribe{print($0)}
   .disposed(by:Dispose)
   ```

   ```swift
   next(2)
   next(1)
   next(0)
   next(100)
   next(200)
   next(300)
   completed
   ```

3. merge연산자
<img width="500" height="200" alt="merge" src="https://user-images.githubusercontent.com/70764912/117161853-386e0e80-adfd-11eb-8ee6-2e77df12f485.png">
   복수 개의 Observable들이 배출하는 항목들을 머지시켜 하나의 Observable이 방출하도록 병합함

   merge로 병합할 수 있는 Observable의 수는 제한이 없음

   concat 연산자와 차이점 : concat 연산자는 하나의 Observable이 모든 요소를 방출하고 completed 이벤트를 전달하면 이어지는 Observable을 연결함.

   이와 달리 merge 연산자는 두 개 이상의 Observable을 병합하고 모든 Observable에서 방출하는 요소들을 순서대로 방출하는 Observable을 리턴함.

   ```swift
   let Dispose = DisposeBag()
   
   let odd = BehaviorSubject(value:1)
   let even = BehaviorSubject(value:2)
   let negative = BehaviorSubject(value:-2)
   
   let source = Observable.of(odd,even)
   source.merge()
   .subscribe{print($0)}
   .disposed(by:Dispose)
   
   odd.onNext(3)
   even.onNext(4)
   
   odd.onNext(6)
   even.onNext(5)
   
   odd.onCompleted() //odd는 더이상 observer에게 이벤트 전달 못함
   even.onNext(8)  //even은 Observer에게 이벤트를 전달할 수 있음
   
   even.onCompleted()  //최종적으로 Observer에게 completed이벤트가 전달됨.
   
   ```

   ```swift
   next(1)
   next(2)
   next(3)
   next(4)
   next(6)
   next(5)
   next(8)
   completed
   ```

4. combineLatest 연산자
<img width="500" height="200" alt="combineLatest" src="https://user-images.githubusercontent.com/70764912/117161937-4e7bcf00-adfd-11eb-81ea-318c7933392e.png">
   Observable이 방출하는 최소 요소를 병합하는 연산자

   두개의 Observable과 클로저를 파라미터로 받음

   ```swift
   let Dispose = DisposeBag()
   
   let greetings = PublishSubject<String>()
   let languages = PublishSubject<String>()
   
   Observable.combineLatest(greetings,languages){
     greetings,languages -> String in return "\(greetings) \(languages)"
   }.subscribe{print($0)}
    .disposed(by:Dispose)
   
   greetings.onNext("Hi")
   languages.onNext("World")    //next(Hi World) 출력됨
   
   greetings.onNext("Hello")    //next(Hello World) 출력됨
   languages.onNext("RxSwift")  //next(Hello RxSwift) 출력됨
   
   greetings.onComplted() //observer에게는 completed가 전달되지 않음(languages가 completed를 이벤트를 보내지 않았으므로)
   
   languages.onNext("SwiftUI")  //next(Hello SwiftUI) 출력됨
   //결합한 Observable 중 하나가 completed되면 마지막 요소를 방출함
   
   languages.onCompleted()  //병합한 모든 Observable이 completed이벤트를 전달하면 그때 Observer에게 completed전달함.
   
   ```

   ```swift
   next(Hi World)
   next(Hello World)
   next(Hello RxSwift)
   next(Hello SwiftUI)
   completed
   ```

5. withLatestFrom 연산자
<img width="500" height="200" alt="withLatestFrom" src="https://user-images.githubusercontent.com/70764912/117162018-618e9f00-adfd-11eb-87af-2e214c93286f.png">
   ```swift
   triggerObservable.withLatestFrom(dataObservable)
   ```

   연산자를 호출하는 Observable : triggerObservable

   파라미터로 전달하는 Observable : dataObservable

   triggerObservable이 next이벤트를 방출하면 dataObservable이 가장 최근에 방출한 next이벤트를 구독자에게 전달함.

   (ex) 회원가입 버튼을 클릭하는 시점에서 textfield에 입력된 값을 가져오는 기능을 구현하고 싶을때 사용하는 것이 가능함.

   ```swift
   let Dispose = DisposeBag()
   
   let trigger = PublishSubject<Void>()
   let data = PublishSubject<String>()
   
   trigger.withLatestFrom(data)
   .subscribe{print($0)}
   .dispose(by: Dispose)
   
   data.onNext("Hello") //아직 트리거 이벤트가 next이벤트를 전달하지 않았기 때문에 data이벤트를 구독자에게 전달하지 않음
   trigger.onNext(()) //trigger이벤트가 next이벤트를 전달했으므로 next(Hello)가 출력됨.
   trigger.onNext(()) //trigger이벤트가 next이벤트를 전달하면 또 next(Hello)가 출력됨, 이미 전달된 이벤트여도 또 전달될 수 있음
   
   data.onCompleted()
   trigger.onNext(()) //또 next(Hello)가 출력됨.
   
   trigger.onCompleted() //trigger observable가 completed이벤트를 방출하면 observer에게 completed이벤트가 전달됨.
   
   ```

   ```
   next(Hello)
   next(Hello)
   next(Hello)
   completed
   ```

   

6. zip 연산자
<img width="500" height="200" alt="zip" src="https://user-images.githubusercontent.com/70764912/117162103-7408d880-adfd-11eb-9edd-da4c9500f83a.png">
   결합한다는 점에서 combineLatest와 동일하지만 순서를 맞춰서 결합함(인덱스를 기준으로 짝을 맞춰서 전달함)

   (첫번째 요소는 첫번째 요소와 결합하고 두번째 요소는 두번째 요소와 결합함)

   combineLatest 연산자와 비교하면 combineLatest는 아래 소스코드에서 number.onNext(2) 이벤트가 발생하면 next(2 : one)을 Observer에게 전달할 것임

   이와 달리 zip은 string observable이 두번째 요소를 방출하기를 기다렸다가 방출하면 그때 next(2 :two)를 Observer에게 전달함

   항상 방출된 순서로 짝을 맞춰 observer에게 전달함

   ```swift
   let Dispose = DisposeBag()
   
   enum error: Error {
     	case error
   }
   
   let number = PublishSubject<Int>()
   let string = PublishSubject<Int>()
   
   Observable.zip(number,string){"\($0) : \($1)"}
   .subscribe{print($0)}
   .disposed(by:Dispose)
   
   number.onNext(1)
   string.onNext("one")   //next(1 : one)이 출력됨
   
   number.onNext(2)
   number.onNext(3)
   number.onNext(4)
   number.onNext(5)
   string.onNext("two")   //next(2 : two)이 출력됨
   
   number.onCompleted()
   string.onCompleted()
   ```

   ```swift
   next(1 : one)
   next(2 : two)
   completed
   ```

7. sample 연산자

   ```swift
   dataObservable.sample(triggerObservable)
   ```

   dataObservable에서 연산자를 호출하고 trigggerObservable을 파라미터로 전달함

   triggerObservable이 next이벤트를 전달할 때마다 dataObservable이 next이벤트를 방출함. 동일한 next이벤트를 반복해서 방출하지는 않음 

   ```swift
   let Dispose = DisposeBag()
   
   enum error: Error {
     	case error
   }
   
   let trigger = PublishSuject<Void>()
   let data = PublishSuject<String>()
   
   data.sample(trigger)
   .subscribe{print($0)}
   .disposed(by:Dispose)
   
   trigger.onNext(())
   data.onNext("Hello")
   
   trigger.onNext(())  //next(Hello)가 출력됨 
   trigger.onNext(())  //아무것도 출력되지 않음(sample연산자는 동일한 next이벤트를 두번이상 반복하지 않음)
   ```

   ```swift
   next(Hello)
   completed
   ```

8. switchLatest 연산자

   가장 최근에 방출한 Observable을 구독하고, 이 Observable이 전달하는 이벤트를 observer에게 전달함

   어떤 Observable이 가장 최근 Observable인지 이해하는 것이 핵심

   ```swift
   let Dispose = DisposeBag()
   
   enum error: Error {
     	case error
   }
   
   let a = PublishSubject<String>()
   let a = PublishSubject<String>()
   
   let source = PublishSubject<Observable<String>>()
   
   source.switchLatest()
   			.subscribe{ print($0) }
   			.disposed(by:Dispose)
   
   a.onNext("1")
   b.onNext("a")  //아무것도 출력 안됨, 이벤트가 전달되지 않음
   
   source.onNext(a)  //a가 최신 observable이 됨
   
   a.onNext("2")  //즉시 observer에게 전달됨 : next(2)가 출력됨
   b.onNext("b")  //b는 observer가 구독하는 최신 이벤트가 아니기 때문에 여기로 전달되는 것은 observer에게 전달되지 않음
   
   source.onNext(b)  //b가 최신 observable이 됨, a observable에 대한 구독을 종료하고 b를 구독하기 시작함
   
   a.onNext("3")   //이벤트가 전달돠지 않음
   b.onNext("c")  //next(c)가 출력됨  
   
   a.onCompleted()
   b.onCompleted()  //둘다 observer에게 전달되지 못함
   
   source.onCompleted //observer에게 completed이벤트가 전달됨
   
   ```

   ```swift
   next("2")
   next("c")
   completed
   ```

9. reduce 연산자
<img width="500" height="200" alt="reduce" src="https://user-images.githubusercontent.com/70764912/117163781-f5ad3600-adfe-11eb-8551-714b4c0e652e.png">
   Seed 값과 accumulator 클로저를 parameter로 받아서 옵저버블을 통해서 옵저버에게 결과를 방출함

   최종 결과만 필요하면 reduce연산자를 사용하면 되고 중간 결과와 최종 결과 모두 필요하면 변환 연산자의 scan연산자를 사용하면 됨.

   ```swift
   let Dispose = DisposeBag()
   
   enum error: Error {
     	case error
   }
   
   let o=Observable.range(start:1,start:5)
   
   o.reduce(0,accumulator: +)
   .subscribe{print($0)}
   .disposed(by:Dispose)
   ```

   ```swift
   next(15)
   completed
   ```

   
