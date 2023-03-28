## 동기(synchronous)와 비동기(asynchronously)

> 곰튀김님 강의와 다른 분들의 포스팅을 참조하여 설명, 예시는 이미지 업로드로 함
> <br/>

## 동기(synchronous): 일반적인 방법

```swift
func syncLoadImage(from imageUrl: String) -> UIImage? {
    guard let url = URL(string: imageUrl) else { return nil }
    guard let data = try? Data(contentsOf: url) else { return nil }

    let image = UIImage(data: data)
    return image
}
```
하나의 작업만 실행, 타이머를 실행하고 이미지를 불러오면 타이머가 멈춘 뒤 syncLoadImage() 코드 실행이 완료되면 타이머가 다시 시작 됌
<br/>

## 비동기(asynchronously): 동시에 두 개 이상의 작업을 실행하는 것처럼 해줌

비동기 방식의 경우 타이머를 실행 중에 이미지를 불러와도 타이머가 멈추지 않음
<br/>
이미지를 불러오는 작업을 별도 쓰레드로 처리해주기 때문 -> GCD (Grand Central Dispatch)
<br/> 
queue에 넣은 작업을 스레드로 적절하게 분배해주는 역할
<br/>

```swift
func asyncLoadImage(from imageUrl: String, completed: @escaping (UIImage?) -> Void) {
    DispatchQueue.global().async { //
        let image = syncLoadImage(from: imageUrl)
        completed(image)
    }
}
```

실행절차
<br/>

1. 일반적으로 코드는 main thread에서 실행(특히 UIKit 관련된 코드는 메인 쓰레드에서 실행되지 않으면 오류가 발생)
<br/>

2. 그리고 이렇게 async로 들어간 코드는 'background thread'에서 실행    
<br/>

3. sync를 사용하면 해당 블록을 실행하는 동안 main thread 작업은 실행 X
<br/>

## DispatchQueue
GCD에서 사용하는 queue라고 생각하면 됌
<br/>

- global()은 queue의 한 종류 
<br/>
- sync, async를 선택해서 사용
<br/>

큐의 종류: global(), main()
<br/>

- global(): concurrent 특성, 그러니까 큐에 먼저 들어가더라도 먼저 나오지 않을 수 있는 특성, 작업량이 적으면 일찍 나옴.
<br/>

- main() 은 반드시 들어간 순서대로 나옴
<br/>
