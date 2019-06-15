---
title: "API 없이 순수 C++로 만든 나만의 레이 트레이서!"
date: 2019-06-15 23:44:00 +0900
categories: C++
---
얼마 전 페이스북 생활코딩 페이지에서 '[Ray Tracing in a Weekend](http://www.realtimerendering.com/raytracing/Ray%20Tracing%20in%20a%20Weekend.pdf)' 라는 짧은 pdf 책을 발견했다. 제목 그대로 주말동안에 나만의 레이 트레이서를 만들어보는 내용이었다. 흥미로운건 DirectX 같은 그래픽 라이브러리를 일체 사용하지 않고 순수 C++로만 구현하여 이런 이미지를 생성한다는 것이다:

![where-next](/assets/images/posts/2019-06-15-ray-tracer/where-next.jpg)

내 호기심을 자극했다. 어떻게 아무 라이브러리도 사용하지 않고 이렇게 아름다운 씬을 렌더링한다는 말인가! 해보지 아니할 수 없었다. 바로 착수했다.

주말만에 만들 수 있다고 했으나, 주말엔 휴식을 취하느라 평일에 틈틈이 하여 1주일에 걸쳐 완성했다: https://github.com/Othereum/RayTracer

아래 사진은 렌더링 결과물이다:

![image](/assets/images/posts/2019-06-15-ray-tracer/image.png)

내가 뭔가를 잘못한건지 DoF가 조금 부자연스러운듯 하다. 그림자도 약하고... 왠지 화면도 좌우반전되었다. 책에서는 유니티처럼 y축을 위아래로 하는 오른손 좌표계를 사용하는데, 나는 언리얼 엔진 프로그래머니까 좌표계도 언리얼 엔진의 z축을 위아래로 하는 왼손 좌표계를 사용해야겠다고 마음먹었다. 혹시 그래서 문제가 생긴건가 싶다. 조만간 디버깅을 해봐야겠다.

여튼, 구현은 생각보다 쉬웠다. 아니, 놀라울 정도로 간단했다. 오히려 그래픽 라이브러리를 활용하여 프로그래밍 하는 것보다 훨씬 간단했다. 이렇게 간단한 코드에서 저렇게 디테일한 scene이 렌더링되다니, 내가 코드를 작성하면서도 믿기지 않았다. 그리고 무엇보다 광선을 추적하고 직접 픽셀을 그리고 AA를 직접 구현해보는 것은 상당히 의미있는 작업이었다. 실제 그래픽 라이브러리 아래에서 어떤 일들이 일어나고 있는건지 조금은 알 수 있는 기회였다. 1주일동안 매우 즐거웠다.

수학 공식이 자주 등장하는데 그중 절반 정도는 이해하지 못해서 결국 그냥 코드를 따라 쳤다. 수학 공부를 열심히 해야겠다는 생각이 들었다.. 그래도 구를 그리기 위해 고등학교에서 배운 수학이 사용되었는데 상당히 흥미로웠다. 원의 방정식을 확장한 구의 방정식과 이차 방정식의 판별식을 응용하여, 카메라의 원점에서 해당 픽셀 방향을 향해 발사한 광선이 구를 지나는지 판별해, 지난다면 구를 그리는 메커니즘이다. 고등학교 수학 시간에서 배웠던 지식을 실제로 써먹게 되다니 참으로 재미있다.

GPU가 아닌 CPU를 사용하여 렌더링하기 때문에 상당한 시간이 걸린다. 위 이미지는 해상도 1920x1080, 픽셀당 1000개(!!)의 AA 샘플을 사용하여 렌더링 하는 데 i7-8750h CPU에서 약 1시간 10분 정도 걸렸다. 원래 싱글 스레드라서 훨씬 오래걸리는데, 병렬로 처리하도록 하여 6코어 12스레드의 CPU를 활용하도록 했다.

병렬화 작업은 `<future>` 헤더의 `async` 함수를 사용했다. `thread` 객체를 사용할 수도 있지만 각각의 스레드를 사용자가 직접 제어해야 하기 때문에 살짝 까다롭다. 반면 `async`는 병렬화 단위가 스레드가 아닌 **task**, 즉 병렬로 실행될 수 있는 하나의 최소 작업 단위이다. 사용자는 `async` 함수를 통해 **task**를 할당하면 라이브러리단에서 스레드 관리를 알아서 해준다. 그저 비동기로 실행할 함수를 인자로 넘겨주기만 하면 되는 것이다. 덕분에 어려움 없이 병렬화를 실현할 수 있었다.

task의 분할은 화면을 가로로 똑같이 잘라서 하나의 task가 각 부분을 담당하도록 했다.

![task1234](/assets/images/posts/2019-06-15-ray-tracer/task1234.png)

처음에는 `thread::hardware_concurrency` 함수를 통해 실행 환경의 CPU 코어 (엄밀히는 논리 프로세서) 개수만큼의 task를 만들었다. 그러나 분할된 화면들의 복잡도가 천차만별이다. 어떤 task는 간단해서 계산이 금방 끝나지만 어떤 task는 복잡해서 오래걸린다. 문제는 처음에 task 수를 코어 수만큼만 만들었기 때문에 task가 종료되면 해당 코어는 놀게 된다. 하나의 task를 여러 코어에서 작업할 수는 없기 때문이다. (위에서 task를 병렬로 실행될 수 있는 하나의 최소 작업 단위라고 했다.) 이러한 현상은 진행률 80~90%부터 두드러지게 나타난다.

![few-tasks](/assets/images/posts/2019-06-15-ray-tracer/few-tasks.png)

가장 간단한 해결책은 그냥 task를 많이 만드는 것이다. 화면을 더 잘게 분할하면 하나의 코어가 하나의 task를 완료했어도 task가 많이 남아있기 때문에 더 가져와서 하면 된다. 덕분에 진행률 95~99% 까지도 항상 CPU 사용률이 90% 아래로 떨어지지 않는다.

![many-tasks](/assets/images/posts/2019-06-15-ray-tracer/many-tasks.png)

멀티스레딩 작업 외에도 소소하게 남은 시간 계산 기능과 간단한 .ini 설정 파일 parser도 직접 구현해봤다. 토큰을 활용하지 않아서 파싱이 완벽하지는 않지만 지금 목적으로는 부족함 없이 잘 작동한다. Scene에 object를 배치하는 것도 하드코딩하지 않고 json을 사용하여 유연하게 처리하고 싶었으나, 이는 배보다 배꼽이 더 커질 것 같아 하지 않기로 했다.

남은 시간 계산을 위해서는 현재까지 렌더링한 픽셀 수를 알아야 하는데, 생각해낸 가장 간단한 방법은 공용 변수를 두고 각각의 task가 픽셀 하나를 렌더링 할 때마다 1씩 증가시키도록 하는 것이었다. 그러나 여러 스레드가 한 번에 공유 자원에 접근하면 race condition이 발생할 수 있기 때문에 이를 해결하기 위하여 `atomic`을 사용했다. 성능 저하가 걱정이었지만 성능저하 없이 기대한대로 완벽하게 작동하였다.

pdf의 저자는 (최소한 이 Ray Tracer에서는) 가능한 한 쉽고 간단하게 만드는 것을 지향한다. 그 때문에 STL이나 최신 기능을 거의 사용하지 않고, 심지어 자원 관리의 기본인 메모리 해제도 해주지 않는다. 물론 나는 이것이 마음에 들지 않아서 최신 STL을 마음껏 사용해주었고 자원 관리에는 스마트 포인터(`unique_ptr`, `shared_ptr`)를 사용해주었다.

이 프로젝트는 짧고 간단했지만 상당히 인상깊었고 재미있었다. 아쉬운 구석이 남지만 일단은 여기에서 손을 떼기로 했다. 나중에 수학 및 그래픽 프로그래밍 공부를 한 뒤에 시간이 나면 리팩토링을 조금 진행해볼까 한다. 그리고 다음 학기에 진행할 자바 프로젝트로 이번에 만든 레이트레이서를 만들어볼까 한다.. ㅋ