---
title: '프로그래머스 - 기능개발'
layout: 
tags: []
category: Programmers
---
[https://programmers.co.kr/learn/courses/30/lessons/42586](https://programmers.co.kr/learn/courses/30/lessons/42586 "https://programmers.co.kr/learn/courses/30/lessons/42586")

&nbsp;

## 문제 설명

어느 개발자 집단이 기능 개선 작업을 수행중이다.
각 기능은 진도가 100%일 때 서비스에 반영할 수 있다.

또한, 각 기능의 개발속도는 모두 다르기 때문에, 뒤에 있는 기능이
앞에 있는 기능보다 먼저 개발될 수 있다.
그리고 **뒤의 기능은 앞의 기능이 배포되어야 배포될 수 있다**.

배포되어야 하는 순서대로 작업의 진도가 적힌 정수 배열
``progresses``와 각 작업의 개발 속도가 적힌 정수 배열 ``speeds``가
주어질 때, 각 배포마다 몇 개의 기능이 배포되는지를 리턴하자.

&nbsp;

## 제한 사항

- ``len(progresses)``, ``len(speeds)`` <= 100
- ``progresses[i]``, ``speeds[i]`` <= 100
- 배포는 하루에 한 번만 할 수 있다.
	- 95% 완료된 작업의 작업 속도가 하루에 4%라면,
	배포는 2일 뒤에 이루어진다.

&nbsp;

## 1. Queue - O(N)

문제에서 주여진 예제를 살펴보자.

	progresses = [93,30,55], speeds = [1,30,5]

맨 처음부터 끝까지의 작업을 1,2,3번으로 정하자.

그러면 1번 작업은 며칠동안 작업해야 완료가 될까?  
93% 완료되어 있고, 하루에 1%씩 작업이 이루어진다.
따라서 ``7일``동안 작업하면 작업이 완료가 된다.

그리고 2번 작업과 3번 작업도 위의 과정을 거치면 각각
``3일``과 ``9일``이 걸린다는 것을 알 수 있다.

위의 예시에서 가장 빨리 완료된 작업은 2번 작업이지만,
2번 작업은 1번 작업이 끝나야 배포가 가능하다.
**뒤의 기능은 앞의 기능이 배포되어야 배포될 수 있다**라는 조건이
있기 때문이다.  
그리고 이는 2번 작업은 1번 작업이 완료가 되는 대로 배포가 될 것이라는
것을 의미하기도 한다.

따라서, 1번 작업이 완료되고 배포될 때, 2번 작업도 같이 배포가 되므로
예시의 정답 중 첫 번째 인덱스에는 ``2``가 들어가게 된다.
1번 작업과 2번 작업이 함깨 배포된다는 것을 의미한다.  
그리고 3번 작업은 1번 작업보다 이틀 더 걸리므로, 이틀 뒤에 배포를
진행하게 된다. 그리고 3번 작업이 마지막 작업이므로,
3번 작업 하나만 배포가 되므로 마지막 인덱스에는 ``1``이 들어가게 된다.

&nbsp;

그러면 어떻게 해야 할 지 대충 보이기 시작했다.
맨앞에 있는 작업을 배포하려고 할 때, 뒤에 남아있는 작업들을 살펴보자.
만약 뒤에 있는 작업이 현재 배포하려는 작업보다 **짧게 걸리는 작업**이라면, **같이 배포하는 작업 목록에 추가**하면 된다.  
만약 뒤에 있는 작업이 현재 배포하려는 작업보다 **길게 걸리는 작업**이라면, **지금까지 확인한 작업들을 배포**하고, 이 작업을 맨앞에 있는 작업으로 만들어준다. 그리고 위의 과정을 모든 작업에 대해 확인할 때 까지 반복한다.

&nbsp;

이 문제의 카테고리는 **큐(Queue)**다. 한번 큐를 써서 풀어보자.
그러면 큐에는 무엇을 넣어야 할까?

며칠 후에 작업이 완료되는지에 대한 정보를 큐에 작업 번호 순서대로
집어넣으면 될 것이다. 여기서는 큐에 작업이 완료되기까지 걸리는
날짜를 집어넣도록 하자.

&nbsp;

작업이 완료될때까지 걸리는 시간은 다음과 같이 구할 수 있다.

	ceil( (100 - progress[i]) / speeds[i] )

``ceil``은 올림을 의미한다. ``ceil(3.2)``는 4가 되고,
``ceil(3.0)``은 3이 된다.

C++에도 ceil 함수가 있다. 형변환을 써야 함에 주의하자.
여기서는 나머지 연산으로 소수점이 있는지 확인할 것이다.

위의 내용을 코드로 표현하면 다음과 같다.

#### solution_queue.cpp
```cpp
#include <vector>
#include <queue>

using namespace std;

vector<int> solution(vector<int> progresses, vector<int> speeds) {
	queue<int> qu;

	for (int i = 0; i < static_cast<int>(speeds.size()); ++i)
		qu.push((100 - progresses[i]) / speeds[i] +
		(((100 - progresses[i]) % speeds[i]) ? 1 : 0));

	vector<int> answer;
	int days = qu.front(), releases = 0;

	while (!qu.empty())
	{
		if (days < qu.front())
		{
			days = qu.front();
			answer.push_back(releases);
			releases = 0;
		}
		else
		{
			++releases;
			qu.pop();
		}
	}

	answer.push_back(releases);
	return answer;
}
```

&nbsp;

위의 코드로도 물론 통과가 된다.
하지만 이 문제는 큐 자료구조를 굳이 새로 만들지 않아도 풀 수 있다.

큐 자료구조 대신, ``progresses``배열을 처음부터 끝까지 순회할 때
사용하는 변수를 하나 만들어서, ``progressess``배열을 큐 처럼
사용할 수 있게 만들면 된다. 여기서는 변수의 이름을 ``idx``라고 짓고,
맨 처음에는 ``0``부터 시작한다고 하자.

``pop()``은 ``idx`` 값을 1 증가키시면 된다.  
``front()``는 ``progresses[idx]``로 확인할 수 있다.  
``empty()``는 ``idx == len(progresses)``로 확인할 수 있다.

그러면 큐를 사용하지 않고 인덱스로 구현해보자.

&nbsp;

#### solution_without_std_queue.cpp
```cpp
#include <vector>

using namespace std;

vector<int> solution(vector<int> progresses, vector<int> speeds) {
    int idx = 0, size = static_cast<int>(progresses.size());
    vector<int> answer;
    
    while(idx != size)
    {
        int from = idx, date = (100-progresses[idx]) / speeds[idx] +
            (((100-progresses[idx]) % speeds[idx]) ? 1 : 0);
        
        while(idx != size-1)
        {
            if (progresses[idx+1] + date * speeds[idx+1] >= 100)
                ++idx;
            else
                break;
        }
        
        answer.push_back(idx - from + 1);
        ++idx;
    }
    
    return answer;
}
```

&nbsp;

큐는 **추상적 자료형(ADT : Abstract Data Type)**이다.
무엇으로 구현하든, 원하는 동작만 잘 동작하게 하면 된다.
현재 주어진 자료들을 이용하여 특정 자료구조 형을 구현할 수 있는지
한번쯤은 생각해보자.

&nbsp;