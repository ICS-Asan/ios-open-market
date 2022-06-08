# 🛒오픈 마켓
- 프로젝트 기간: 2022.01.03 ~ 2022.01.28

## 목차
- [구현 내용](#구현_내용)
    - [화면](#화면)
    - [주요 구현 내용](#주요구현내용)
- [STEP1](#STEP1)
    - [주요 구현 내용](#STEP1-1)
    - [Trouble Shooting](#STEP1-2)
- [STEP2](#STEP2)
    - [주요 구현 내용](#STEP2-1)
    - [Trouble Shooting](#STEP2-2)
- [STEP3](#STEP3)
    - [주요 구현 내용](#STEP3-1)
    - [Trouble Shooting](#STEP3-2)

## 구현 내용
<a name="화면"></a>
### 📱 화면

|메인화면|상품 조회|등록화면|
|:---:|:---:|:---:|
|<img src="https://i.imgur.com/R528WFG.gif" width="390">|<img src="https://i.imgur.com/XJGhq08.gif" width="390">|<img src="https://i.imgur.com/zBcyvVQ.gif" width="390">|

|등록 경고|상품 수정|상품 삭제|
|:---:|:---:|:---:|
|<img src="https://i.imgur.com/Kn2DzhA.gif" width="390">|<img src="https://i.imgur.com/wjAB279.gif" width="390">|<img src="https://i.imgur.com/vSsI91x.gif" width="390">|

<a name="주요구현내용"></a>
### 💻 주요 구현 내용
- REST API 활용(with URLSession)
    - 서버와 통신을 하기위한 로직 및 CRUD기능 구현
    - 추후 확장성 및 재사용성을 고려하여 실제 통신을 수행하는 메서드와 dataTask, requestBody 등을 생성하는 메서드를 분리
- 네트워크에 의존하지 않는 테스트 구현
    - 실제 서버와 통신하지 않고 Mock데이터를 활용하여 API관련 메서드가 정상적으로 동작하는지 테스트를 할 수 있도록 구현
    - URLSessionProtocol을 생성 및 활용하여 기존 URLSession과 테스트를 위한 MockURLSession을 사용할 수 있도록 구현
- UICollectionViewDiffableDataSource 사용
    - DiffableDataSource를 사용하여 CollectionView의 DataSource를 간단하게 세팅
    - 데이터 변화에 따른 UI업데이트가 자연스럽게 보일 수 있도록 구현

<a name="STEP1"></a>
## 1️⃣ STEP1
<a name="STEP1-1"></a>
### 💻 주요 구현 내용
#### < 모델 타입 >
- 서버 API 문서를 참고하여 모델타입 구현
- 리스트 조회 등의 특정 상황에서 사용되지 않는 프로퍼티의 타입을 옵셔널로 설정(ProductDetail-images, vendors)

#### < JSON Parsing >
- JSON타입의 데이터를 디코딩하고 에러를 처리하는 `decodeData`메서드 구현
- 외부에서 decode 할 때마다 에러를 처리하는 것 보다 decode와 동시에 에러를 처리하게 하여 호출부에서의 코드를 간결하게 함
- 제네릭 타입으로 구현하여 원하는 타입으로 decode할 수 있도록 구현

#### < 네트워킹 >
- DataTast를 관리하는 메서드와 실제 네트워킹을 수행하는 메서드를 분리
    - 네트워킹을 수행할 때마다 DataTask관련 코드가 반복되는 것을 피하고 추후에 DataTask를 추가로 생성할 시 재사용할 수 있음
    - 실제 네트워킹을 수행하는 부분의 코드가 간결해지고 가독성을 향상시킴
- escapingClosure와 Result타입을 활용하여 통신이 완료된 후에 데이터를 활용할 수 있도록 구현
- URLRequest 생성 시 httpMethod도 함께 설정해줄 수 있도록 URLRequest의 extension에 이니셜라이저를 추가로 구현
    - httpMethod를 세팅하는 과정에서 하드코딩을 하지 않기 위해 내부에 열거형을 생성하여 활용
- URLManager열거형을 통해 api통신을 위한 url생성을 간편하게 할 수 있도록 구현
    - 요청별로 네이밍을 하여 case로 나누고 연산 프로퍼티 활용
    - URLComponents와 quertyItems를 활용하여 특정 값을 넣어주면 원하는 url이 생성되도록 구현

<a name="STEP1-2"></a>
### 🔫 Trouble Shooting
#### 비동기적인 네트워크 통신 처리
- 문제
> escapingClosoure에 대한 이해도가 부족하여 비동기적인 네트워킹 작업을 처리하기가 어려웠습니다.
> dataTak 메서드 내부에서 `main.async`로 뷰를 업데이트 하면 정상적으로 작동하지만 뷰를 바로 업데이트 하기 위해서는 dataTask메서드를 뷰컨트롤러에 구현을 해야 했습니다.
> 그렇게 되면 뷰컨트롤러가 비대해지고 각 객체별 역할이 모호해진다는 단점이 있어 모델에서 해당 작업을 구현하고자 했습니다.
> 그래서 APIManager에 구현을 하기 위해 네트워킹 작업의 결과를 담을 프로퍼티를 생성하고 DispatchSemaphore를 사용하여 메인 스레드가 작업을 기다리도록 구현했습니다.
> 하지만 네트워킹 작업을 위해 메인스레드를 기다리게 하는 방법은 적절치 못하다고 판단하고 네트워크 요청이 많아질수록 더 문제가 될 것이라고 생각했습니다.
- 해결
    - escaping Clousure를 활용하여 작업이 완료되면 completion Handler를 통해 필요한 작업을 수행할 수 있도록 변경하였습니다.
    - completion Handler에 Result타입을 활용하여 성공한 경우의 데이터 처리 뿐만아니라 실패한 경우의 에러처리까지 가능하도록 구현하였습니다.

<a name="STEP2"></a>
## 2️⃣ STEP2
<a name="STEP2-1"></a>
### 💻 주요 구현 내용
#### < 메인 화면 >
- CollectionView를 list, grid 형식으로 두가지를 구현한 뒤 segmentedControl을 이용하여 뷰를 바꿔주도록 함
- 미리 두가지 뷰를 구현한 뒤 MainView의 뷰를 할당해주기 때문에 화면 전환시 빠르게 전환이 가능

#### < CollectionView >
- list, grid에 맞게 diffableDataSource를 세팅하는 메서드 구현
- OpenMarketViewLayout 내부 CollectionViewFlowLayout을 타입 프로퍼티로 구현하여 layout을 관리하고 사용하기 쉽도록 함
- defaultContentConfiguration을 사용하지 않고 cell을 커스텀하기 위해 list와 grid에서 사용할 cell을 구현한 뒤 적용
    - 각 cell타입에 xib파일을 생성하여 AutoLayout설정과 각 뷰 요소의 기본값 설정
    - cell타입 내부에 상품의 정보를 받아 cell을 세팅하는 메서드 구현

#### < 반복 로직 최소화 및 가독성 개선 >
- String, Int타입을 활용하는 과정에서 반복적으로 사용되는 로직을 extension에 메서드로 분리함
- NameSpace를 생성하여 Cell에 상품정보를 표시하는데 공통적으로 사용되는 문자열을 관리하고 명확한 네이밍을 통해 가독성을 향상


<a name="STEP2-2"></a>
### 🔫 Trouble Shooting
#### CollectionViewCell 구현 방식
- 문제
> defaultContentConfiguration을 사용하여 ListCollectionView 구현 시 원하는 형태로 Cell을 만들어낼 수 없었습니다.
- 해결
    - defaultContentConfiguration의 반환값이 UIListContentConfiguration인것을 확인하고 UIContentConfiguration을 생성 및 원하는 형태로 구현하고 CollectionView에서 사용하는 방법을 먼저 사용
    - 생성과정이 복잡하고 효율적이지 못하다고 판단하여 CustomCell을 구현한 뒤 CollectionView에 등록 및 재사용하는 방법으로 Cell을 구현
    - Cell내부 요소들의 배치와 설정을 원하는대로 세팅할 수 있다는 장점이 있음

#### List<->Grid 전환시 로딩 속도 개선
- 문제
> 기존 하나의 dataSource에서 segmentedControl의 상태에 따라 collectionView의 재사용 cell과 layout을 변경하도록 구현했습니다.
> 하지만 List<->Grid 전환 시 로딩이 오래걸리는 문제가 발생했습니다.
- 해결-1
    - dataSource를 변경하지 않고 CollectionView를 2개를 만든 후 segmentedControl의 상태에 따라 view를 변경하는 방식 선택
    - 이에 따라 dataSource 또한 2개를 생성하여 사용
- 해결-2
    - 근본적인 원인이 dataSource에 있지 않고 이미지를 불러오는 과정에서 `Data(contentsOf url: URL)`을 사용하고 있었는데 해당 이니셜라이저가 동기처리를 하기 때문에 때문에 로딩이 오래걸림
    - 해당 과정을 cell이 재사용될 때마다 반복되기 때문에 눈에 띄게 느려지게 보임
    - 서버에서 받아온 JSON 데이터를 decode할 때 `dataDecodingStrategy`를 활용하여 미리 이미지로 전환하여 저장 할 수 있도록 구현

<a name="STEP3"></a>
## 3️⃣ STEP3
<a name="STEP3-1"></a>
### 💻 주요 구현 내용
#### < 상품 등록을 위한 Model타입 >
- 상품 등록을 위한 Model타입 생성 및 서버로 전달 시 JSON 데이터로 전달하기 위해 Codable 프로토콜 채택
- API문서에 기본값이 있는 프로퍼티는 이니셜라이저 기본값 사용
- multipart-form으로 요청 시 편의성을 고려하여 Image를 위한 타입 

#### < 상품 이미지 추가 >
- UIImagePickerController를 사용하여 앨범이나 카메라에서 이미지를 가져오도록 구현
- `imagePickerController(_ : , didFinishPickingMediaWithInfo:)`메서드를 사용하여 이미지를 수정하여 추가할 수 있도록 구현
- UIAlertController를 통해 선택지를 제공
- 사용자 편의를 위해 추가 버튼을 이미지 StackView의 제일 앞에 배치
- 추가할 이미지가 5개인 경우 이미지 추가 버튼을 hidden처리하여 추가하지 못하도록 구현
- 추가된 이미지를 버튼으로 구현하고 tag를 설정하여 수정 및 삭제시 이미지를 식별할 수 있도록 구현

#### < 상품 설명 textView 설정 >
- textViewDelegate의 `textViewDidBeginEditing`, `textViewDidEndEditing` 메서드를 사용하여 placeholder기능 구현
- `textView(_ :, shouldChangeTextIn:, replacementText:)`메서드를 사용하여 입력가능 글자 수를 1000자로 제한

#### < 상품 추가 요구사항에 대한 에러처리 >
- 각 항목별 요구사항들에 대해 판단하는 로직을 통해 에러문구가 작성 되도록 구현
- 불충족된 항목에 대해서만 에러 문구가 표시되도록 하여 사용자가 어떤 부분에 대한 정보가 누락되었는지 판단할 수 있도록 구현
- <img src="https://i.imgur.com/xEdPcM4.png" width="200">

#### < 키보드 유/무에 따른 View 크기 변경 >
- 키보드의 Notification(`keyboardWillShowNotification`, `keyboardWillHideNotification`)를 받아 키보드 유/무를 판단
- 키보드가 나타나면 크기를 계산하여 scrollView의 `contentInset.bottom`에 할당하여 크기를 조절
- 키보드가 사라지면 `contentInset.bottom = .zero`로 할당하여 원래 크기로 돌려줌
- `UITapGestureRecognizer(target: Any?, action: Selector?)`를 등록하여 빈 공간을 터치하면 키보드가 내려가도록 구현



<a name="STEP3-2"></a>
### 🔫 Trouble Shooting
#### StackView를 활용한 사진 등록 방법
- 문제
> 단순히 사진을 추가하고 삭제하는 것은 어렵지 않았습니다.
> 하지만 등록한 사진을 다른 사진으로 변경하기 위해서는 해당 이미지를 알아야 했습니다.
> 그리고 버튼 - 추가된 이미지 - 이미지1 처럼 새로 추가하기 위해선 `addArrangedSubView`메서드가 아닌 다른 방법을 선택해야 했습니다.

- 해결
    - 사진변경을 위해서 각 이미지에 `tag`를 할당하여 변경할 이미지를 식별할 수 있도록 함
    - `insertArrangedSubView` 메서드를 사용하여 추가될 이미지의 위치를 계산하여 이미지가 추가되도록 함


