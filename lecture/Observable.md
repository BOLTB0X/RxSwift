## Observables<Data>

RxSwift에서 가장 핵심
<br/>

### Observable
RxSwift에서 가장 핵심, RxSwift의 Observable 객체를 생성하고, 그 객체에서 이벤트가 생성되는 이 과정들이 비동기로 구성, 실행 됨
<br/>

### Subscribe
Observable 객체를 Subscribe(구독)하고, 객체에서 생성된 값에 Observer는 반응
<br/>
Observable 객체는 정의만 되어있음,  Subscribe(구독) 되기 전에는 아무런 이벤트 발생 X
<br/>

## Observable의 3가지 행동 규칙
- next event: Observable은 next를 통해 행동(이벤트)를 발행
<br/>

- completed event: next를 통해 모든 값이 발행되면 complete 가 호출되면 subscribe가 종료, 중간에 error가 발생하면 complete는 호출 X
<br/>

- error event: 에러가 발생하면 subscribe가 종료(dispose)
<br/>

```swift
func rxswiftLoadImage(from imageUrl: String) -> Observable<UIImage?> {
    return Observable.create { seal in
        asyncLoadImage(from: imageUrl) { image in
            seal.onNext(image) // 2.1
            seal.onCompleted() // 2.2
        }
        return Disposables.create()
    }
}
```

 1. create를 통해 Observable 객체를 생성
 <br/>

 2. Observable이 발행한 이벤트를 클로져의 result에 받아서 코드 블로 내부의 행동 규칙을 실행
 <br/>

메소드들이 연결되어 흘러가는 것을 "stream"이라 정의
<br/>

## Disposable: Subscribe 종료
위 방식처럼 모든 이벤트를 발행한 Observable의 stream은 Subscribe을 종료해야함

 1. disposable

```swift
// Disposable 객체를 변수로 이용
var dispose: Disposable?

@IBAction func onLoadImage(_ sender: Any) {
    imageView.image = nil
    
    dispose = rxswiftLoadImage(from: LARGER_IMAGE_URL)
    .observeOn(MainScheduler.instance)
    .subscribe({ result in
        switch result {
        // 3가지 규칙
        // Observable은 next를 통해 행동(이벤트)를 발행
        case let .next(image):
            self.imageView.image = image

        case let .error(err):
            print(err.localizedDescription)

        // completed event: next를 통해 모든 값이 발행되면 complete 가 호출되면 subscribe가 종료, 중간에 error가 발생하면 complete는 호출 X
        case .completed:
            break
        }
    })
}

@IBAction func onCancel(_ sender: Any) {
	dispose?.dispose() // dispose() 메소드
}​
```

 Disposable 객체를 변수로 받아서 dispose() 메소드로 취소하기
 <br/>

 2. disposeBag

```swift
var dispose: Disposable?
// 객체를 생성
var disposeBag: DisposeBag = DisposeBag()

@IBAction func onLoadImage(_ sender: Any) {
    imageView.image = nil

    dispose = rxswiftLoadImage(from: LARGER_IMAGE_URL)
        .observeOn(MainScheduler.instance)
        .subscribe({ result in
            switch result {
            case let .next(image):
                self.imageView.image = image

            case let .error(err):
                print(err.localizedDescription)

            case .completed:
                break
            }
        })
    disposeBag.insert(dispose)
}

@IBAction func onCancel(_ sender: Any) {
    disposeBag = DisposeBag()
}​
```
 DisposeBag() 객체를 이용해 여러 개의 Subscribe에 대한 구독 취소 가능
 <br/>

 ```

