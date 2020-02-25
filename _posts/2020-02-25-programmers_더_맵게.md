---
title: '프로그래머스 - 더 맵게'
layout: 
tags: []
category: Programmers
---
[https://programmers.co.kr/learn/courses/30/lessons/42626](https://programmers.co.kr/learn/courses/30/lessons/42626 "https://programmers.co.kr/learn/courses/30/lessons/42626")

&nbsp;

## 문제 설명
모든 음식을 스코빌 지수 K 이상으로 만들자.  
두 음식을 섞어서 맵게 만들 수 있다.  

**새 스코빌 = 가장 안매운거 + (두번째로 안매운거 * 2)**

섞어야 하는 최소 횟수를 리턴하자.

&nbsp;

## 제한 사항
- 1 <= 스코빌 길이 <= 1,000,000
- 0 <= K <= 1,000,000,000
- 0 <= 각 스코빌 원소 <= 1,000,000
- 불가능하면 -1을 리턴해야 한다.

&nbsp;

## 1. brute force - O(N^2) - 시간 초과

&nbsp;

간단한 방법으로는 문제에서 설명한 동작을 계속 수행하는 것이다.  
조건을 만족할 때 까지 가장 안 매운 음식 둘을 계속 섞어주면 된다.  
어떤 과정으로 수행해야 할 지 생각해보자.  

1. 스코빌 배열을 순회하면서, 가장 맵지 않은 두 음식을 찾는다.  
2. 그리고 가장 맵지 않은 음식이 K 이상인지 확인한다.  
	1. K 이상이면, 지금까지 섞은 횟수를 리턴한다.  
	2. 그렇지 않으면, 가장 맵지 않은 두 음식을 섞는다.  
3. 이 과정을 음식 하나가 남을 때 까지 반복한다.
4. 음식 하나가 남았을 때, 이 음식의 스코빌 지수를 확인한다.
	1. K 이상이면 지금까지 섞은 횟수를 리턴한다.
	2. 그렇지 않으면, -1을 리턴한다.

그러면 코드를 작성해보자.

&nbsp;

####solution_bruteforce.cpp
```cpp
#include <vector>

using namespace std;

int solution(vector<int> scoville, int K) {
    int answer = 0;
    int m1 = 0, m2 = 1, length = static_cast<int>(scoville.size());
    
    for (int i=0;i<length-1;++i)
    {
        m1 = 0; m2 = 1;
        if (scoville[m2] < scoville[m1])
            swap(m1, m2);
        
        for (int j=2;j<length-i;++j)
        {
            if (scoville[j] < scoville[m2])
            {
                m2 = j;
                if (scoville[m2] < scoville[m1])
                    swap(m1, m2);
            }
        }
        
        if (scoville[m1] >= K)
            return answer;
        
        scoville[m1] += 2*scoville[m2];
        swap(scoville[m2], scoville[length-i-1]);
        
        ++answer;
    }
    
    return (scoville[0] >= K) ? answer : -1;
}
```

&nbsp;

> 두 번째 작은 원소에 K를 집어넣고, 이 값을 끝의 원소와 swap 하는 식으로
> 섞은 음식을 삭제하는 동작을 구현하였다.  
> 연속된 메모리에서 특정 원소를 삭제하는 동작은 O(N)의 시간이 필요하다.  
> 따라서 최대한의 시간을 절약하기 위해 swap으로 삭제를 구현하였다.  

&nbsp;

이 방식으로 수행했을 때, n개의 음식에 대해 최악의 경우의 연산량을 생각해보자.  
최악의 경우는 1개의 음식만을 남기고 모든 음식을 섞었을 때이다.  

&nbsp;

------------

n개의 음식을 순회한 뒤, 2개의 음식을 섞는다.  
n-1개의 음식을 순회한 뒤, 2개의 음식을 섞는다.  
...  
3개의 음식을 순회한 뒤, 2개의 음식을 섞는다.  
나머지 2개의 음식을 섞는다.  
마지막 남은 음식의 스코빌 지수를 확인한다.   

------------

&nbsp;

위의 과정에서 순회한 값들을 한 번 더해보자.

`` n + (n-1) + (n-2) + ... + 3 + 2 ``

회 정도의 연산을 수행하게 된다.
위의 연산 결과는 약 n^2 정도가 된다.

음식 개수의 최대 크기는 1백만이고, 1백만의 제곱은 1조이다.  
반으로 나눠도 5천억이고, 이는 시간초과를 의미한다.  
따라서, brute force로는 풀 수 없다.

&nbsp;

## 2. heap 자료구조 사용하기 - O(NlogN)

&nbsp;

음식을 섞으면 섞을 수록 가장 맵지 않은 음식의 여부는 바뀔 수 있다.  
따라서 음식을 섞은 후의 가장 맵지 않은 음식 2개를 계속해서 찾아야 한다.  

예시로 주어진 입력 값, `` [1,2,3,9,10,12] `` 를 보자.  

여기서 가장 맵지 않은 음식의 스코빌 값은 ``1``이다.  
두번째로 맵지 않은 음식의 스코빌 값은 ``2``다.  
이 두 음식을 섞으면 다음의 스코빌 값을 갖는 음식이 만들어진다.  

	1 + 2*2 = 5

그렇다면 이 ``5``는 가장 맵지 않은 음식일까?  
그렇지 않다. 이 음식보다 덜 매운 음식, ``3``이 존재한다.  

``[3,5,9,10,12]``는 맞는 순서다.  
``[5,3,9,10,12]``는 틀린 순서다.  

새로 생성된 음식을 추가했을 때,  
스코빌 지수의 크기대로 맞춰서 들어가면 좋을 것 같다.

새로운 값을 넣은 뒤, 정렬하는 방법을 생각할 수도 있다.  
하지만, 그 방법은 brute force보다 효율이 더 떨어진다.  

빠르게 수행할 수 있는 정렬의 시간 복잡도는 O(NlogN)이다.  
그리고 우리는 이 행동을 최대 N회 해야한다.  
정렬을 계속 수행할 경우, 시간 복잡도는 O(N^2 logN)이 된다.  
이런 방법을 쓸 바엔 차라리 brute force로 푸는 편이 더 빠르다.  

이 문제의 카테고리가 뭔지 확인해보자. **힙(heap)**이다.  
힙이라는것이 무엇인지 한 번 찾아보자.

[https://ko.wikipedia.org/wiki/%ED%9E%99_(%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0)](https://ko.wikipedia.org/wiki/%ED%9E%99_(%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0))

위키백과에서는 힙에 대해 다음과 같이 설명하고 있다.

> 최댓값 및 최솟값을 찾아내는 연산을 빠르게 하기 위해 고안된 완전 이진트리

우리가 주목해야 할 사항은 값을 추가하거나 빼는데 소요되는 시간이다.  
다행스럽게도 해당 동작에 소요되는 시간은 **O(logN)**이다.  
원리가 궁금하다면 한 번 검색해보자.  

그러면 이 힙 자료구조를 어떻게 사용해야 할 까?

가장 먼저, 힙 자료구조에 현재 존재하는 모든 음식을 집어넣는다.  
이 과정에서 **O(NlogN)**의 시간이 소요된다.

그러면 힙을 이용해서 가장 작은 음식의 정보를 **O(logN)**에 가져올 수 있다.
맨 처음에 가져온 음식의 스코빌 수치가 K 이상이라면 섞은 횟수를 리턴한다.  
아니라면 하나를 더 가져온 뒤, 섞은 음식의 값을 힙에 다시 추가한다.  
이 과정을 힙에 음식이 단 하나 남아있을 때 까지 반복한다.  

이 경우 **O(logN)** 동작을 최대 N번까지 수행하게 되므로  
시간복잡도는 **O(NlogN)**이 된다.

**C++**에서는 **priority_queue**,  
**Java**에서는 **PriorityQueue**,
**Python**에서는 **heapq**를 이용할 수 있다.

> C++의 경우, set도 heap과 비슷한 형태로 쓸 수 있다.  
> set 역시 크기 순서대로 정렬을 해 주기 때문이다.  
> 하지만, set은 중복 원소를 허용하지 않는다.  
> 따라서, 이 문제에서 set을 써서 풀고 싶다면  
> 중복 원소를 허용해주는 multiset을 사용하도록 하자.  

> C++에도 배열이나 vector 등을 이용해서 heap을 만들 수 있다.  
> 하지만 같은 동작을 해도 더 많은 코드를 써야 한다.  
> 귀찮으니 여기선 그냥 priority_queue를 사용한다.  

&nbsp;

그러면 이 내용을 코드로 작성해보자.

&nbsp;

####solution_heap.cpp
```cpp
#include <vector>
#include <queue>

using namespace std;

int solution(vector<int> scoville, int K) {
    int answer = 0;
    priority_queue<int, vector<int>, greater<int>> pq;
    
    for (int v : scoville)
        pq.push(v);
    
    while(pq.size() > 1)
    {
        int v = pq.top();   pq.pop();        
        
        if (v >= K) return answer;
        
        v += 2 * pq.top();  pq.pop();
        pq.push(v);
        ++answer;
    }
    
    return (pq.top() >= K) ? answer : -1;
}
```

&nbsp;

> C++의 priority_queue는 기본적으로 가장 큰 수를 앞에 둔다.  
> 따라서 가장 작은 수롤 맨 앞에 두고 싶을 때에는  
> 위와 같이 greater<>를 사용하도록 하자.  

&nbsp;


