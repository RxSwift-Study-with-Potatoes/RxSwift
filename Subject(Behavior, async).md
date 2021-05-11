:star:  Subject

* 우리가 개발을 할때 실시간으로 observable에 값을 추가하고 subscriber를 할 수 있는 것이 필요함. 이때 사용하는 것이 subject!!

- Observable인 동시에 Observer임. 두가지 역할을 할 수 있음.

--------------------------------------------------------------------------------------------------------

observable이랑 비슷한거 같지만 다름,,,!!

:exclamation:Subject와 Observable의 차이점

subject는 multicast방식으로 여러개의 observer를 subscribe할 수 있음

반면, observable은 unicast방식이므로 observer 하나만 subscribe 할 수 있음

(https://sujinnaljin.medium.com/rxswift-subject-99b401e5d2e5 공유 자료에 좋은 링크를 올려주셔서 링크 첨부할께요!)  

:star:  ​BehaviorSubject

​	publishsubject이랑 유사하지만 초기 값을 가지고 생성된다는 점이 다름

​	구독자는 구독시에 값이 없다면 default값을 받고 값이 있다면 최신 값을 받음

​	언제 사용?? 보통 뷰를 가장 최신의 데이터로 미리 세팅할 때 

![D4FD318F-FEA1-4839-B921-3486FDE166AF_1_105_c](https://user-images.githubusercontent.com/70764912/117772323-054ed380-b272-11eb-8643-c71f6ce3182b.jpeg)

```swift
let disposeBag = DisposeBag()

enum MyError: Error {
   case error
}

let one = BehaviorSubject<String>(value: "Behavior")

one.subscribe{print("Subject1:",$0)}
    .disposed(by: disposeBag)

one.onNext("subject")

one.onNext("Hello")

one.subscribe{print("Subject2:",$0)}
    .disposed(by: disposeBag)

one.onNext("world")

one.onCompleted()
```

출력결과는?

```swift
Subject1: next(Behavior)
Subject1: next(subject)
Subject1: next(Hello)
Subject2: next(Hello)
Subject1: next(world)
Subject2: next(world)
Subject1: completed
Subject2: completed
```

즉 Behaviorsubject를 생성하면 내부에 next이벤트가 하나 만들어지는것 , 그리고 이것을 구독하는 구독자가 추가되면 저장된 next이벤트가 바로 전달됨.

:exclamation:새로운 옵저버를 추가하면?

```swift
let disposeBag = DisposeBag()

enum MyError: Error {
   case error
}

let one = BehaviorSubject<String>(value: "Behavior")

one.subscribe{print("Subject1:",$0)}
    .disposed(by: disposeBag)

one.onNext("subject")

one.subscribe{print("Subject2:",$0)}
    .disposed(by: disposeBag)

one.onNext("world")

one.onCompleted()
```

```swift
Subject1: next(Behavior)
Subject1: next(subject)
Subject2: next(subject)    //가장 최신의 이벤트 전달됨
Subject1: next(world)
Subject2: next(world)      //가장 최신의 이벤트 전달됨
Subject1: completed
Subject2: completed
```

이후 새로운 옵저버가 추가되면 항상 가장 최신의 이벤트를 옵저버가 받고 시작함

:exclamation:error이벤트가 전달되면? :

<img width="791" alt="B1507166-185B-4C1F-A02D-4ECD5DBFF37B" src="https://user-images.githubusercontent.com/70764912/117772605-5068e680-b272-11eb-8454-156b88d08d63.png">

Behavior subject에서 에러가 나면 subscribe하고 있는 모든 옵저버에서 error남

```swift
let disposeBag = DisposeBag()

enum MyError: Error {
   case error
}

let one = BehaviorSubject<String>(value: "Behavior")

one.subscribe{print("Subject1:",$0)}
    .disposed(by: disposeBag)

one.onNext("subject")

one.onError(MyError.error)

one.subscribe{print("Subject2:",$0)}
    .disposed(by: disposeBag)
```

출력결과는? 구독하고 있는 모든 옵저버로 error이벤트가 전달됨

```swift
Subject1: next(Behavior)
Subject1: next(subject)
Subject1: error(error)   
Subject2: error(error)
```

--------------------------------------------------------------------------------------------------------------

:star: AsyncSubject

:exclamation:다른 Subject들과 이벤트를 옵저버에게 전달하는 시점에서 차이가 있음

subject로 completed이벤트가 구독되기전까지 어떠한 이벤트도 전달되지 않음

completed이벤트가 전달되면 그 시점의 가장 최근의 next이벤트를 구독자에게 전달함

![DE4A3BC7-DA72-4647-BD0A-B348D580A6BD_1_105_c](https://user-images.githubusercontent.com/70764912/117788314-418a3000-b282-11eb-927a-5e330e91c0f6.jpeg)

```swift
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let subject = AsyncSubject<String>()

subject.subscribe{(print($0))}
    .disposed(by: bag)

subject.onNext("aaa")    //이벤트 전달되지 않음
subject.onNext("bbb")    //이벤트 전달되지 않음

subject.subscribe{(print($0))}
    .disposed(by: bag)

subject.onNext("ccc")    //이벤트 전달되지 않음

subject.onCompleted()    //가장 최근의 이벤트를 전달함
```

결과는??

```swift
next(ccc)
next(ccc)
completed
completed
```

:exclamation:error이벤트가 전달되면??

![0A8A3909-70E1-47DB-B79E-11A6AF0FBDE2_1_105_c](https://user-images.githubusercontent.com/70764912/117788395-549d0000-b282-11eb-8445-c74407109c20.jpeg)

```swift
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let subject = AsyncSubject<String>()

subject.subscribe{(print($0))}
    .disposed(by: bag)

subject.onNext("aaa")
subject.onNext("bbb")

subject.subscribe{(print($0))}
    .disposed(by: bag)

subject.onNext("ccc")

subject.onError(MyError.error)  //최근의 이벤트가 전달되지 않고 구독자들에게 에러 이벤트를 전달함
```

결과는?? 마지막 이벤트 전달 없이 구독자에게 에러를 전달함.

```swift
error(error)
error(error)
```





[참고] http://reactivex.io/documentation/subject.html

​		  https://ntomios.tistory.com/12

​		  https://gwangyonglee.tistory.com/61

​		 https://jinshine.github.io/2019/01/05/RxSwift/3.Subject%EB%9E%80/





