---
title: '프로그래머스 - 완주하지 못한 선수'
layout: 
tags: []
category: Programmers
---
[https://programmers.co.kr/learn/courses/30/lessons/42576](https://programmers.co.kr/learn/courses/30/lessons/42576 "https://programmers.co.kr/learn/courses/30/lessons/42576")
&nbsp;
## 문제 설명
마라톤 선수 중 단 한명이 완주를 하지 못했다. 누구인지 찾자.  
participant, completion 배열을 통해 정보를 확인할 수 있다.
&nbsp;
## 제한 사항
- 명단의 크기 n은 1 <= n <= 100,000 이다.
- len(completion) = len(participant) - 1
 - 완주자 수는 참가자 수 보다 1명 적다.
- **동명이인**이 존재할 수 있다.

&nbsp;
&nbsp;
## 1. 정렬을 이용하기 - O(NlogN)
&nbsp;
주어진 예제의 2번 입력을 보면서 생각해보자.
&nbsp;

	participant : ["marina", "josipa", "nikola", "vinko", "filipa"]
	completion  : ["josipa", "filipa", "marina", "nikola"]

&nbsp;
참가자와 완주자의 명단이 불규칙적으로 섞여있다. 이 문자열 배열들을 정렬하면 사전순으로 정렬한 형태가 된다. 보기 편하게 한 번 참가자 명단과 완주자 명단을 정렬해보자.
&nbsp;

	participant : ['filipa', 'josipa', 'marina', 'nikola', 'vinko']
	completion  : ['filipa', 'josipa', 'marina', 'nikola']

&nbsp;

참가자와 완주자가 뭔가 딱 맞아 떨어지는 것 처럼 보인다.  
맨 끝의 ``vinko``를 제외하면 말이다.

만약 완주자가 ``mardina``가 아니라 ``vinko`` 였다면 어땠을까?
&nbsp;

	participant : ['filipa', 'josipa', 'marina', 'nikola', 'vinko']
	completion  : ['filipa', 'josipa', 'nikola', 'vinko']

&nbsp;

``fillipa`` 와 ``josipa`` 까지는 맞다.,  
하지만, 참가자의 3번째 항목은 ``marina`` 이나,  
완주자의 3번째 항목은 ``nikola``다.  

만약 참가자 목록에 ``marina``가 있었다면  
사전순 정렬로 인해 ``nikola``보다  
``marina``가 먼저 나왔을 것이다.  

하지만 여기서 ``marina``가 나오지 않았다는 것은  
완주자 목록에는 ``marina``가 없다는 의미이다.  
  
따라서 참가자 명단에는 있지만, 완주자 명단에는 없는 사람은  
``marina``가 된다.

실제 문제상에서의 예제에는 참가자 명단과 완주자 명단의 모든 항목이 일치하므로,  
참가자 목록 맨 마지막에 있는 사람, ``vinko``가  완주하지 못한 사람이다.  
&nbsp;  
&nbsp;  
따라서, 다음과 같은 코드를 작성하면 답을 얻을 수 있을 것이다.

1. **정렬**한다
2. 두 배열의 같은 index를 비교하며, 다른 이름이 나오면
   **참가자 쪽의 이름을 리턴**한다.
3. 끝까지 순회해도 나오지 않았다면, **참가자 명단의 맨 끝의 이름을 리턴**한다.
&nbsp;

```cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

string solution(vector<string> participant, vector<string> completion) {
    sort(participant.begin(), participant.end());
    sort(completion.begin(), completion.end());
    
    for (int i=0; i<static_cast<int>(completion.size()); ++i)
    {
        if (participant[i] != completion[i])
            return participant[i];
    }
    
    return participant.back();
}
```

&nbsp;
&nbsp;
## 2. HashMap 사용하기 - O(kN) - k는 문자열 길이
&nbsp;
이 문제의 카테고리는 해시다. 해시를 써서 풀어보라고 한 만큼, 해시를 써서 풀어보자.  
**C++**의 **unordered_map**, **Java**의 **HashMap**, **Python**의 **Dictionary**를 사용할 수 있다.  
HashMap의 Insert(), Find()의 시간복잡도는 **O(1)**이다.

> 물론 항상 O(1)인 것은 아니다.  
> key의 해시 함수에 따라 O(1) 커질 수도 있다.  
> 문자열의 경우, 해시 성능은 O(k) (k는 문자열 길이) 이다.  
> 해시함수의 성능이 나쁘다면 해시마다 충돌이 일어나  
> **최악의 경우 O(N)**까지 치솟을 수 있다.  

완주한 사람의 명단을 HashMap에 집어 넣고,  
참가자 명단을 통해 찾아나가면  
**O(kN)** 시간 복잡도로 해결할 수 있다.  
반대로 해도 문제될 것은 없다.  

**동명이인**이 존재할 수 있다는 점을 감안하여,  
key는 **사람의 이름**, value는 **해당 이름을 가진 사람의 수**로 정해두자.  
완주자 명단을 순회하며 HashMap에 집어넣고, 그 수를 value에 기록하자.  
&nbsp;

| josipa | filipa | marina | nikola |
| :---: | :---: | :---: | :---: | 
| 1 | 1 | 1 | 1 |

&nbsp;
&nbsp;

참가자 명단을 순회하며  
HashMap에 해당 이름을 가진 참가자 명단이  
몇 명인지 확인하자.  
완주자가 남아 있으면 1을 빼 주고,  
없으면 이 참가자가 완주하지 못한 사람이다.  
&nbsp;  
한 번 실제로 순회해보자.  

&nbsp;

------------


&nbsp;

`` ["marina", "josipa", "nikola", "vinko", "filipa"] ``

&nbsp;
``marina``는 1명 있다. ``marina``의 수를 1 빼 주자.
&nbsp;

| josipa | filipa | marina | nikola |
| :---: | :---: | :---: | :---: | 
| 1 | 1 | 0 | 1 |

&nbsp;
``josipa``는 1명 있다. ``josipa``의 수를 1 빼주자.
&nbsp;

| josipa | filipa | marina | nikola |
| :---: | :---: | :---: | :---: | 
| 0 | 1 | 0 | 1 |

&nbsp;
``nikola``는 1명 있다. ``nikola``의 수를 1 빼주자.
&nbsp;

| josipa | filipa | marina | nikola |
| :---: | :---: | :---: | :---: | 
| 0 | 1 | 0 | 0 |

&nbsp;
``vinko``는 없다. 따라서 완주하지 못한 사람은 ``vinko``다.
&nbsp;

------------


&nbsp;
그러면 이 내용을 코드로 작성해보자.
&nbsp;
```cpp
#include <string>
#include <vector>
#include <unordered_map>

using namespace std;

string solution(vector<string> participant, vector<string> completion) {
    unordered_map<string, int> runners;
    
    for (string& str : completion)
        ++runners[str];
    
    for (string& str : participant)
        if (--runners[str] < 0) return str;
    
    return string("");
}
```

&nbsp;
> C++의 map의 [] 연산자는, 해당 key가 없으면
> 알아서 insert를 해 주므로 위와 같은 코드가 가능하다.  
> insert를 할 때, value는 해당 타입의 기본 생성자를 호출한다.  
> 여기서는 value의 타입이 int이므로, __int()__의 결과인  
> 0이 들어간다.

&nbsp;
정렬을 써도 통과할 수 있고, 해시를 써도 통과할 수 있다.  
하지만 이 문제 카테고리가 해시인 만큼,  
해시 자료구조를 쓰는 연습을 해 보도록 하자.