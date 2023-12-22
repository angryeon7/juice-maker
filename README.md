# ios-juice-maker
쥬스메이커 프로젝트 저장소입니다. 

## 1. 팀원

|Tommy|Jin|
|:---:|:---:|
|![IMG_0362](https://github.com/angryeon7/Algorithm/assets/87401351/c67b0ae0-af15-4675-838d-77f0da720769)|![파덕](https://github.com/angryeon7/Algorithm/assets/87401351/d46fb85b-a567-4c79-899f-eb57ecfc0aac)|
|@angryeon7|@wlsdud-dev|

## 2. 소개

**과일저장소 객체의 과일 재고를 관리하여 생과일 쥬스를 만드는 프로그램**

### Model

| Type | 기능 |
| --- | --- |
| Fruits | 쥬스를 만드는데 사용되는 과일들을 담은 Enum |
| Juice | - 만들어지는 쥬스들을 관리하는 Enum - Recipe 라는 computed property 를 사용하여, 쥬스를 선택하면, 레시피 별로 필요한 과일의 재고를 return 해주는 Dictionary 형태로 구현 |
| FruitStore | - 현재 프로젝트에서는 과일재고 객체가 하나만 존재한다는 것을 보장하기 위해 싱글톤 적용 | - 각 과일의 초기 재고: 10개각 과일의 수량 n개를 변경하는 메서드각 과일의 재고를 확인하는 메서드 |
| JuiceMaker | - FruitStore을 가져와서 쥬스를 만드는 기능만 담은 구조체 |
| JuiceMakerError | - 재고 부족가 부족할때, 에러문구를 띄어주는 Error 타입 |

---

### ViewController

| Type | 기능 |
| --- | --- |
| JuiceMakerViewController | - 화면간 데이터 전달을 위한 NotificationCenter 를 화면 초기 실행시 호출되는 함수 viewDidLoad에 addObserver - 각 쥬스 버튼을 클릭할 때 재고가 충분하다면, 재고가 실시간으로 수정되고 성공 Alert 실행 - 재고가 충분하지 않다면, 재고를 수정하는 Modal 이동 |
| FruitStockViewController | - 각 Stepper에 값이 변경될때마다 Label이 변경되게 하는 함수 구현  - 닫기 버튼을 클릭하였을때, Model의 FruitStore()로 NotificationCenter post 를 보내게 하여 JuiceOrderViewController()의 값이 변하게 하는 함수 구현 |

## 3. 프로젝트 구조(UML)
### [싱글톤]
![싱글톤](https://github.com/angryeon7/Algorithm/assets/87401351/2fd235db-f292-4de1-978f-0b5120d15b69)
### [Delegate 패턴]
![델리게이트](https://github.com/angryeon7/Algorithm/assets/87401351/5a37bc37-a7cb-4456-8ee0-a3ea78a78c7b)
## 4. 설계 과정

## STEP1. ****쥬스 메이커 타입 정의****

> PR **#77** Step1 ****쥬스 메이커 타입 정의****
> 
- Fruits - 과일의 종류에 대한 열거형 생성
- Juice - 쥬스의 종류에 대한 열거형 생성
    - 각 쥬스 제조에 필요한 정해진 개수의 과일 목록을 제시(recipe)
- FruitStore 과일별 재고를 저장, 확인 , 변경
- JuiceMaker - 현재 주문한 쥬스의 제조가 가능한지 판별
    - 레시피에 따라 재고 감소
- 쥬스 자판기에서 발생하는 에러 처리

### 개선한 내용

- 에러메세지의 가독성 향상
    - 연산 프로퍼티를 사용
- 접근제어자 추가
    - private(set)을 통해 수정이 불가능하게 막아 외부에서의 재고 수정을 방지
- final을 사용하여 static dispatch가 가능해 메모리 효율이 좋아짐
- 클래스 당 하나의 파일에 작성되도록 수정

**실행 결과**

## STEP2. ****초기화면 기능구현****

> PR **#97** Step2 ****초기화면 기능구현****
> 
- '재고 수정' 버튼을 터치하면 '재고 추가' 화면으로 이동
- 각 주문 버튼 터치 시
    - 쥬스 재료의 재고가 있는 경우 : 쥬스 제조 후 `“*** 쥬스 나왔습니다! 맛있게 드세요!”` 얼럿 표시
    - 쥬스 재료의 재고가 없는 경우 : `“재료가 모자라요. 재고를 수정할까요?”` 얼럿 표시
        - ‘예' 선택시 재고수정 화면으로 이동
        - ‘아니오' 선택시 얼럿 닫기

**개선한 내용**

- 버튼 식별
    
    여러개의 버튼을 어떻게 식별하여 명세서 대로의 기능으로 구현할 수 있을지 고민하였다.
    
    1. 버튼의 tag를 부여해 구별을 하였지만 가독성이 떨어지는것 같아, 버튼의 TitleText로 식별하여 버튼의 처리를 수행했다.
    
    > 하나의 메서드가 버튼을 식별하고 식별한 버튼에 따라 기능을 처리 한다면, SRP를 위배하고 추후 특정 버튼에서만 로직이 추가되어야 하는 경우 문제가 발생 할 수 있다.
    > 
    
    <aside>
    💡 버튼 하나당 하나의 메소드로 관리하고 하나의 메소드는 하나의 기능만 하도록 수정하였다.
    
    </aside>
    
    ```swift
    @IBAction private func touchUpStrawberryBananaJuiceOrderButton(_ sender: Any) {
            orderJuice(juice: .strawberryBanana)
        }
    ```
    
- 화면 전환
    
    쥬스메이커 뷰에서 재고수정 뷰로 화면전환을 어떤 방식으로 할지 고민하였다.
    
    1. 네비게이션 뷰, 모달 뷰 어떤 것으로 만들지 고민 하였고 논의 끝에 재고수정은 깊이와 흐름을 가지기 보다는 쥬스메이커와는 다른 흐름이라는 결론으로 모달뷰로 만들었다.
    2. 화면전환 구현에 세그, 프레젠트 호출 중 고민하였다. 스토리보드를 최대한 활용하고 코드의 깔끔함을 위해 세그를 사용해 구현하였다. (이름 짓기가 어려웠다.)
    
    ```swift
    @IBAction private func addButton(_ sender: UIButton) {
            guard let fruitStockViewController = storyboard?.instantiateViewController(identifier: "fruitStockViewController") as? FruitStockViewController else { return }
            self.present(fruitStockViewController, animated: true, completion: nil)
        }
    ```
    

## STEP3. 데이터 전달

> PR #**110** Step3 ****재고 수정 기능구현****
> 
- 스토리보드 AuotoLayout 적용
- IBOutlet, IBAction등을 이용하여 스토리보드와 코드 연결
- 화면 제목 '재고 추가' 및 '닫기' 버튼 구현
    - 닫기를 터치하면 이전화면으로 복귀
- 화면 진입시 과일의 현재 재고 수량 표시
- stepper를 통한 재고 수정

### STEP3-1 Delegate 패턴 적용

**화면간의 데이터 이동**

```swift
final class JuiceMakerViewController: UIViewController, FruitStockDelegate {

	func didUpdateFruitStock(fruitStock: [Fruits : Int]) {
		       fruitStock.forEach { (fruit,value) in
		           let currentFruit = juiceMaker.fruitStore.quantity(of: fruit)
		           let difference = value - currentFruit
		           juiceMaker.fruitStore.changeStock(fruitName: fruit, amount: difference)
		       }
		       configureView()
		   }
//...

	override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
	        guard let navigationViewController = segue.destination as? UINavigationController else { return }
	        if let fruitStockViewController = navigationViewController.visibleViewController as? FruitStockViewController {
	            fruitStockViewController.fruitStock = juiceMaker.fruitStore.fruitStock
	            fruitStockViewController.delegate = self
	        }
	    }
}
```

```swift
protocol FruitStockDelegate: AnyObject {
    func didUpdateFruitStock(fruitStock: [Fruits: Int])
}
//...
@IBAction func closedStockViewButton(_ sender: UIBarButtonItem) {
        delegate?.didUpdateFruitStock(fruitStock: fruitStock)
        self.dismiss(animated: true, completion: nil)
    }
```

### STEP3-2 싱글톤 패턴 적용 + NotificationCenter

### 싱글톤을 선택한 이유

> 현재 프로젝트에서는 과일재고객체가 하나만 존재한다는 것을 보장하기 위해 싱글톤이 적절한 패턴이라고 생각함 (서버와 클라이언트의 관계)
> 
> 
> 과일의 재고를 소모하고 충전하므로 과일 창고의 또 다른 객체가 생성될 필요가 없다고 판단.
> 
- 싱글톤 패턴 적용
- Notification을 이용하여 뷰 업데이트 구현

**화면간의 데이터 이동**

```swift
final class JuiceMakerViewController: UIViewController {
    
    private let juiceMaker = JuiceMaker()
    
    @IBOutlet private weak var strawberryLabel: UILabel!
    @IBOutlet private weak var bananaLabel: UILabel!
    @IBOutlet private weak var kiwiLabel: UILabel!
    @IBOutlet private weak var pineappleLabel: UILabel!
    @IBOutlet private weak var mangoLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        configureView()
        NotificationCenter.default.addObserver(self,
                                               selector: #selector(configureView),
                                               name: .update,
                                               object: nil)
    }
		//...
}
extension Notification.Name {
    static let update = Notification.Name("update")
}
```

```swift
override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        NotificationCenter.default.post(name: .update, object: nil)
    }
```

### 개선한 내용

- 약한 참조
    - 클로저는 외부에 프로퍼티, 메서드의 값을 캡처 하기 때문에 **`self` 에 강한 참조가 일어남.** 때문에 강한 순환참조가 일어나 `[weak self]`로 강한 순환참조를 끊어줌.
- 기능분리을 통해 객체지향적 코드 작성
    - 메소드가 한가지 일만 할 수 있도록 수정.
