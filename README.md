# MVVM++

## MVVM에서의 의문점 3가지 

영상에서 안드로이드 자체가 가지고 있는 문제점
1. LifeCycle
2. Databinding으로 다 표현하기 힘든 View 이벤트 (_View의 감지)
3. Resource 등 Context를 접근해야 하는 경우
이렇게 세 가지를 말씀하셨습니다. 

제가 궁금했던 것은, 네 가지가 있습니다. 그래서 정승욱 개발자님께 직접 여쭤봤습니다.

### 1. RxBinder라는 것을 자체적으로 만드셨을 때, 1. LifeCycle에서 어떤 경우의 문제를 해결하셨나요..?
앱 전환할 때 비정상적으로 종료되는 부분, 간단히 화면 회전 시 데이터의 관리들이 있다고는 찾아봤는데, 제가 RxBinder 코드를 이용해서 어떤 문제를 해결 하신 것인지는 모르겠어서 여쭤봤습니다.

답변 : 

제가 직접 만든 Android Lifecycle 과 관련된 Rx 컴포넌트는 원하는 시점에 자동으로 Rx 코드를 실행해주고 
원하는 시점에 자동으로 Dispose 를 실행해주는 코드입니다. 
예를 들어 Resume 시점마다 특정 코드를 실행하고 싶다면 onResume() 을 상속받아 구해야 하고 
이때 실행된 코드를 CompositionDispoable 에 저장하였다가 onPause 에 종료시켜줘야 합니다. 
이런 코드들은 관리 면에서 onResume 과 onPause 를 지속적으로 상속받아 구현해야 하는 것과 명시적으로 CompositionDispoable 을 정의하고 종료를 호출해줘야 한다는 것입니다. 
제가 만든 코드는 이 부분을 @ResumeToPause fun xx() : Observable 로 명세하면 자동으로 호출 및 종료를 관리해주는 코드를 만든 것입니다.

답변요약: 모두 상속해서 라이프 싸이클을 고려하지 않고 간단히 어노테이션으로 명세해줄 수 있습니다.

### 2. GlobalLayoutChangeListener가 dataBinding으로 표현하기 힘든(2way-binding을 써야하는) 이유가 view가 그려지는 그 순간, 순간의 width와 height를 계속 리턴 받아서 그에 따른 다른 view의 변경을 진행해줘야 하기 때문인가요..?
리스너가 달려있는 view의 크기가 그때 그때 주어지고, 다른 뷰의 크기가 달라지기 때문에 다른 뷰에 영향을 주는 메서드를 구현하기 위해 2way-binding이 필요한 건가요?
[관련 링크](https://keykat7.blogspot.com/2021/02/android-ongloballayoutlistener.html)

#### 배경 지식

GlobalLayoutChangeListener :  뷰의 너비 및 높이를 알고 싶을 때 사용합니다.

getWidth와 getHeight으로 구할 수 있는 것을 왜 리스너를 쓸까요?
이것은 앱이 뷰를 완벽히 그리기 전에 너비와 높이를 가져오려고 해서,
아직 그려지지 않은 뷰의 너비와 높이, 즉 0을 반환해주는 것입니다.
생명주기로 해결하는 것도, 쉽지 않습니다.(예 파이어스토어에서 이미지 로드해 올 때 오래 걸리는 것)

답변 : 

View size, 토글된 정보, 키보드 입력 등의 이벤트는 데이터바인딩으로 구현이 가능하더라도 관리면에서 쉽지는 않으며 내부 변수로 선언하더라도 언제나 위변조 될 가능성을 가지고 있습니다. 
저는 극도로 immutable 을 지향하기 때문에 read-only 전용 컴포넌트를 구현하여 접근하는 것을 선호합니다.

read-only 전용 컴포넌트 :

  InverseBindingAdapter = (읽어오기 + 다른 곳에 또 다른 BindingAdapter)

  read-only 전용 컴포넌트 = (읽어오기) 

### 3. 바로 위의 GlobalLayoutChangeListener 뿐만 아니라 View의 Attach 여부의 listener, 단발성으로 listen해야 하는 것들 모두리스너가 달린 것들이 2way-binding이 필요한 이유가 다른 뷰에 영향을 주는 또 다른 bindingAdapter(?)를 추가 해줘야 하기 때문인가요?

이 질문의 가정이 맞다면 다른 뷰에 영향을 주는 리스너를 가진 상황은, 
전부 databinding으로 구현하기 위해서 2way-binding이 필요한 것이 맞는 걸까요?

답변 : 

네 그러한 BindingAdapter, InverseBindingAdapter 와 같은 부차적 코드들 또한 관리면에서 최소화 하고자하는 것이 목적입니다.

### 4. [activity가 아닌 곳에서 context에 접근하기 위해] applicationcontext를 이용해 주입 받아서 Resource 같은 곳에 접근하는 것인가요?

답변 : 

별도로 Activity 나 Context 를 랩핑한 컴포넌트들을 만들고 그것을 통해서 접근하도록 명세하는게 **가장 기초적인 작업**입니다.



