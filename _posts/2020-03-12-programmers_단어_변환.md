---
title: '프로그래머스 - 단어 변환'
layout: 
tags: []
category: Programmers
---
[https://programmers.co.kr/learn/courses/30/lessons/43163](https://programmers.co.kr/learn/courses/30/lessons/43163 "https://programmers.co.kr/learn/courses/30/lessons/43163")

&nbsp;

## 문제 설명
두 개의 단어 ``begin``, ``target``과 단어의 집합 ``words``가 있다.  
다음 규칙을 이용하여 ``begin``에서 ``target``으로 변환하는 가장 짧은 과정을 찾자.

```
1. 한 번에 한 개의 알파벳만 바꿀 수 있다.
2. words에 있는 단어로만 변환할 수 있다.
```

최소 몇 번을 변환해야 ``begin``이 ``target``이 되는지를 리턴하자.  

만약 변환할 수 없으면, ``0``을 리턴하자.

&nbsp;

## 제한 사항
- 단어는 알파벳 **소문자**로만 이루어져 있다.
- 3 <= ``len(words[i])`` <= 10
- ``len(words[i])`` == ``len(words[j])``
- ``begin`` != ``target``

&nbsp;

## 1. BFS - O(N^2)

주어진 단어를 특정한 규칙을 이용하여 목표로 하는 단어를
만들 수 있는 가장 짧은 단계를 리턴해야하는 문제이다.
주어진 예제를 한번 살펴보자.

	["hot", "dot", "dog", "lot", "log", "cog"]
	begin : "hit", target : "cog"

시작 단어인 ``begin``은 ``"hit"``이고, 목표로 하는 단어인 ``target``은
``"cog"``다. 그럼 ``"hit"``에서 변화할 수 있는 단어는 무엇이 있을까?

``"hot"``으로만 변화할 수 있다. 가운데의 ``'o'`` 한글자만 차이난다.
나머지 단어들은 모두 2~3글자의 차이가 나서 변화할 수 없다.

``"hot"``또한 반대로 ``"hit"``으로 변화할 수 있을 것이다.
다만 다시 ``"hit"``로 돌아가봤자 변화 횟수만 낭비하므로 굳이 돌아갈 필요는 없다.

그러면 ``"hot"``에서 변화할 수 있는 단어는 무엇이 있을까?
``"dot"``과 ``"lot"``으로 변화할 수 있다. 둘 다 앞글지 하나만 다르다.  
나머지 변환은 2~3글자의 차이가 나거나, 이미 이전에 변화한 적이 있는 글자라
굳이 다시 변화하는 것은 낭비다.

그리고 ``"dot"``과 ``"lot"``은 각각 다음과 같이 변화할 수 있다.
여기서 같이 변화를 했거나, 이미 변화를 해 본 단어는 생략한다.  

``"dot"`` -> ``"dog"``
``"lot"`` -> ``"log"``

그리고 ``"dog"``와 ``"log"``는 둘 다 목표 단어인 ``"cog"``로
변화할 수 있다. 그러면 ``"cog"``로 변화하기 위한 최단 변화는 다음과 같이
두 가지 방법이 존재한다.

	"hit" -> "hot" -> "dot" -> "dog" -> "cog"
	"hit" -> "hot" -> "lot" -> "log" -> "cog"

&nbsp;

가만 생각해보면, 단어들 사이에는 변화 가능성이라는 관계가 존재한다.
이를 그림으로 표현할 수 있을 것 같다.

![Graph](/assets/images/programmers/p20/graph.png "Graph")

이런 식으로 그래프를 그릴 수 있다는 것을 알았다면?  
각각의 단어는 ``정점``으로 표현하고, 변화의 가능성을 ``간선``으로 표현하면,
주어진 단어들을 토대로 그래프를 만들고, 탐색을 수행해서 최단거리를
알아낼 수 있을 것이다.

그래프로 만드는 것은 간단하다. 각각의 단어가 들어있는 인덱스를
정점의 번호로 삼고, 모든 단어에 대해 변화 가능성을 확인하여
간선을 만들면 되는 것이다.  


&nbsp;

#### solution.cpp
```cpp
#include <string>
#include <vector>
#include <queue>

using namespace std;

int solution(string begin, string target, vector<string> words) {
    words.push_back(begin);
    
    int l = words.size(), wl = words[0].size(), t = -1;
    
    vector<vector<int>> g(words.size(), vector<int>());
    
    for (int i=0;i<l;++i)
    {
        if (words[i] == target) t = i;
        for (int j=i+1;j<l;++j)
        {
            int d = 0;
            for (int v=0;v<wl;++v)
                if (words[i][v] != words[j][v]) ++d;
            if (1 == d)
            {
                g[i].push_back(j);
                g[j].push_back(i);
            }
        }
    }
    
    if (-1 == t)    return 0;
    
    queue<int> qu;
    vector<bool> dirt(l,false);
    dirt[l-1] = true;
    qu.push(l-1);
    
    int ans = 1, s = 1;
    while(!qu.empty())
    {
        int v = qu.front(); qu.pop();
        for (int i : g[v])
        {
            if (t == i) return ans;
            if (!dirt[i])
            {
                qu.push(i); dirt[i] = true;
            }
        }
        
        if (0 == --s)
        {
            ++ans;
            s = static_cast<int>(qu.size());
        }
    }
    
    return 0;
}
```

&nbsp;

해외 풀이 사이트인 **LeetCode**에도 동일한 문제가 있다.  
[Word Ladder](https://leetcode.com/problems/word-ladder/ "Word Ladder")  
혹시라도 본인의 코드가 오답을 리턴하여 테스트케이스가 알고싶은 경우,
이 곳에서 테스트를 해 보자.  
다만 leetcode에서 요구하는 리턴 값은 변환 리스트의 길이이므로,
프로그래머스에서 요구하는 답에 1을 더해줘야 한다.