---
layout: post
title:  Hidden Markov Model
category: Text Mining 
tag: HMMs
description: Simple is a beautiful but functional jekyll theme. The font-type setting looks really good when writers use CJK mixed with English.
---

이번 글에선 **은닉마코프모델(Hidden Markov Models, HMMs)**을 다루어 보도록 하겠습니다. 순차적인 데이터를 다루는 데 강점을 지녀 개체명 인식, 포스태깅 등 단어의 연쇄로 나타나는 언어구조 처리에 과거 많은 주목을 받았던 기법입니다. 이 글은 고려대 정보통신대학원 정순영 교수님의 강의를 기반으로 작성합니다.

## Introduction
은닉마코프 모델은 Sequence 모델입니다.

## 마코프 체인
은닉마코프모델은 마코프 체인(Markov chain)을 전제로 한 모델입니다. 마코프 체인이란 마코프 성질(Markov Property)을 지닌 이산확률과정(discrete-time stochastic process)을 가리킵니다. 마코프 체인은 러시아 수학자 마코프가 1913년경에 러시아어 문헌에 나오는 글자들의 순서에 관한 모델을 구축하기 위해 제안된 개념입니다. 

한 상태(state)의 확률은 단지 그 이전 상태에만 의존한다는 것이 마코프 체인의 핵심입니다. 즉 한 상태에서 다른 상태로의 전이(transition)는 그동안 상태 전이에 대한 긴 이력(history)을 필요로 하지 않고 바로 직전 상태에서의 전이로 추정할 수 있다는 이야기입니다. 마코프 체인은 아래와 같이 도식화됩니다.

$$P({ q }_{ i }|{ q }_{ 1 },...,{ q }_{ i-1 })=P({ q }_{ i }|{ q }_{ i-1 })$$

날씨를 마코프 체인으로 모델링한 예시는 다음 그림과 같습니다. 아래 마코프 체인에서 각 노드는 상태, 엣지는 전이를 가리킵니다. 엣지 위에 작게 써 있는 $a_{ij}​$는 $i​$번째 상태에서 $j​$번째 상태로 전이할 확률을 나타냅니다. 각 노드별로 전이확률의 합은 1입니다(예컨대 $a_{01}+a_{02}+a_{03}=1​$) 상태에는 일반적인 종류의 상태(예컨대 HOT, COLD, WARM) 이외에 시작(start)과 끝(end) 상태도 있습니다. 

<a href="https://imgur.com/iCPKPWz"><img src="https://i.imgur.com/iCPKPWz.png" width="400px" title="source: imgur.com" /></a>





## 은닉 마코프 모델

은닉마코프모델은 각 상태가 마코프체인을 따르되 은닉(hidden)되어 있다고 가정합니다. 예컨대 당신이 100년 전 기후를 연구하는 학자인데, 주어진 정보는 당시 아이스크림 소비 기록뿐이라고 칩시다. 이 정보만으로 당시 날씨가 더웠는지, 추웠는지, 따뜻했는지를 알고 싶은 겁니다. 우리는 아이스크림 소비 기록의 연쇄를 관찰할 수 있지만, 해당 날짜의 날씨가 무엇인지는 직접적으로 관측하기 어렵습니다. 은닉마코프모델은 이처럼 관측치 뒤에 은닉되어 있는 상태(state)를 추정하고자 합니다. 날씨를 예시로 은닉마코프모델을 도식화한 그림은 다음과 같습니다.



<a href="https://imgur.com/lEMDGBC"><img src="https://i.imgur.com/lEMDGBC.png" width="500px" title="source: imgur.com" /></a>



위 그림에서 $B_1$은 날씨가 더울 때 아이스크림을 1개 소비할 확률이 0.2, 2개 내지 3개 먹을 확률은 각각 0.4라는 걸 나타냅니다. $B_1$은 날씨가 더울 때 조건부확률이므로 HOT이라는 은닉상태와 연관이 있습니다. $B$는 **방출확률(emission probablity)**이라고도 불립니다. 은닉된 상태로부터 관측치가 튀어나올 확률이라는 의미에서 이런 이름이 붙은 것 같습니다.





## Likelihood

우선 우도(likelihood)부터 계산해 보겠습니다. 우도는 모델 $λ$가 주어졌을 때 관측치 $O$가 나타날 확률 $p(O$\|$λ)$을 가리킵니다. 바꿔 말해 모델 $λ$이 관측치 하나를 뽑았는데 그 관측치가 $O$일 확률입니다. 이렇게 관측된 $O$가 아이스크림 [3개, 1개, 3개]라고 칩시다. 그럼 모델 $λ$가 위의 그림이라고 할 때 이 $O$가 뽑힐 확률은 얼마일까요? 이걸 계산해 보자는 겁니다. 아래 그림을 봅시다.



<a href="https://imgur.com/syZWL5E"><img src="https://i.imgur.com/syZWL5E.png" width="300px" title="source: imgur.com" /></a>



두번째 날짜를 중심으로 보겠습니다. 모델 $λ$를 보면 날씨가 더울 때(hot) 아이스크림을 1개 먹을 확률은 0.2입니다. 그런데 두번째 날이 전날에 이어 계속 더울 확률은 0.6이므로 이를 곱해주어야 둘째 날의 상태확률를 계산할 수 있습니다. 여기에서 마코프 체인을 따른다고 가정하므로 상태확률을 계산할 때는 직전 상태만을 고려합니다. 위 그림을 식으로 나타내면 다음과 같습니다.



$$
\begin{align*}
P(3\quad 1\quad 3,hot\quad hot\quad cold)=&P(hot|start)\times P(hot|hot)\times P(cold|hot)\\ &\times P(3|hot)\times P(1|hot)\times P(3|cold)\\=&0.8\times0.6\times0.3\\&\times0.4\times0.2\times0.1
\end{align*}
$$


각 날짜별로 날씨가 더울 수도 있고 추울 수도 있습니다. 따라서 $2^3$가지의 경우의 수가 존재합니다. 아래 표와 같습니다.

| 상태1  | 상태2  | 상태3  |
| :--: | :--: | :--: |
| cold | cold | cold |
| cold | cold | hot  |
| cold | hot  | cold |
| hot  | cold | cold |
| hot  | hot  | cold |
| cold | hot  | hot  |
| hot  | cold | hot  |
| hot  | hot  | hot  |

따라서 관측치 [3, 1, 3]에 대한 최종적인 우도는 다음과 같이 구합니다.



$$
\begin{align*}
P(3\quad 1\quad 3)=&P(3\quad 1\quad 3,cold\quad cold\quad cold)+\\ &P(3\quad 1\quad 3,cold\quad cold\quad hot)+...\\&P(3\quad 1\quad 3,hot\quad hot\quad hot)
\end{align*}
$$






## Notation

여기에서 notation을 잠깐 정리하고 넘어가겠습니다. 다음과 같습니다. 저 또한 정리 용도로 남겨두는 것이니 헷갈릴 때만 다시 보시고 스킵하셔도 무방할 것 같습니다.

- $Q=\{q_0, q_1, q_2, ..., q_n, q_F\}$ : 상태(state)의 집합(set). $q_0$은 시작상태, $q_F$는 종료상태, $n$은 상태의 개수를 나타낸다.
- $A$ : 전이확률 행렬($n×n$). $a_{ij}$는 $i$번째 상태에서 $j$번째 상태로 전이할 확률($Σ_{j=1}^{n}a_{ij}=1$).
- $B=b_i(o_t)$ : $i$번째 상태에서 $t$번째 관측치 $o_t$가 나타날 방출확률. 
- $O=[o_1,o_2,...,o_t,...,o_T]$ : 길이가 $T$인 관측치의 시퀀스.
- $α_t(j)=P(o_1,o_2,...,o_t,q_t=j$\|$λ)$ : 모델 $λ$가 주어졌을 때 $j$번째 상태와 $o_1,...,o_t$가 나타날 확률. 전방확률(Forward Probability)
- $β_t(j)=P(o_{t+1},o_{t+2},...,o_T,q_t=j$\|$λ)$ : 모델 $λ$가 주어졌을 때 $j$번째 상태와 $o_{t+1},...,o_T$가 나타날 확률. 후방확률(Backward Probability)







## Compute Likelihood : Forward Algorithm

지금까지 은닉마코프모델의 우도(방출확률)를 계산하는 예시를 보여드렸습니다. 이제 이걸 전체 데이터에 대해 확대해 봅시다. 그런데 여기 문제가 하나 있습니다. 예시에서도 살펴봤듯 계산해야 할 경우의 수가 정말 많다는 겁니다. 가령 $N$개의 은닉상태가 있고 관측치 길이가 $T$라면 우도 계산시 고려해야 할 가짓수가 $N^T$개나 됩니다. 이러한 비효율성을 완화하기 위해 다이내믹 프로그래밍(dynamic programming) 기법을 씁니다. 다이내믹 프로그래밍은 중복되는 계산을 저장해 두었다가 푸는 것이 핵심 원리입니다. 다음 그림을 보겠습니다.



<a href="https://imgur.com/UcXttLx"><img src="https://i.imgur.com/UcXttLx.png" title="source: imgur.com" /></a>



예컨대 아이스크림 3개($o_1$)와 1개($o_2$)가 연속으로 관측됐고 두 번째 시점($t=2$)의 날씨가 추웠을($q_1$) 확률은 $α_2(1)$입니다. 마찬가지로 아이스크림 3개($o_1$)가 관측됐고 첫 번째 시점($t=1$)의 날씨가 추웠을($q_1$) 확률은 $α_1(1)$입니다. 또한 아이스크림 3개($o_1$)가 관측됐고 첫 번째 시점($t=1$)의 날씨가 더웠을($q_2$) 확률은 $α_1(2)$입니다. 각각을 구하는 식은 다음과 같습니다.



$$
\begin{align*}
{ \alpha  }_{ 1 }(1)=&P(cold|start)\times P(3|cold)\\ { \alpha  }_{ 1 }(2)=&P(hot|start)\times P(3|hot)\\ { \alpha  }_{ 2 }(1)=&{ \alpha  }_{ 1 }(1)\times P(cold|cold)\times P(1|cold)\\ &+{ \alpha  }_{ 1 }(2)\times P(cold|hot)\times P(1|cold)
\end{align*}
$$



Forward Algorithm의 핵심 아이디어는 이렇습니다. 중복되는 계산은 그 결과를 어딘가에 저장해 두었다가 필요할 때마다 불러서 쓰자는 겁니다. 위 그림과 수식을 보시다시피 $α_2(1)$를 구할 때 직전 단계의 계산 결과인 $α_1(1)$, $α_1(2)$을 활용하게 됩니다. 이해를 돕기 위한 예시여서 지금은 계산량 감소가 도드라져 보이지는 않지만 데이터가 조금만 커져도 그 효율성은 명백해집니다. $j$번째 상태에서 $o_1,...,o_t$가 나타날 전방확률 $α$는 다음과 같이 정의됩니다.



$$
{ \alpha  }_{ t }(j)=\sum _{ i=1 }^{ n }{ { \alpha  }_{ t-1 }(i)\times { a }_{ ij } } \times { b }_{ j }({ o }_{ t })
$$



$α_t(j)$는 $j$번째 상태와 $t$개의 관측치 시퀀스가 나타날 수 있는, 가능한 경우의 수가 모두 고려되어 합쳐진 확률입니다. 예컨대 위 그림에서 $α_3(2)$의 경우 세번째 시점에 HOT이 되려는데 가능한 경로는 [H, H, H], [H, C, H], [C, H, H], [C, C, H]입니다. 따라서 전방확률을 관측치 시퀀스 끝까지 계산하면 앞서 계산한 우도와 동치가 됩니다.



$$
\begin{align*}
P(O|\lambda )=&P\left( { o }_{ 1 },{ o }_{ 2 },...,{ o }_{ T }|\lambda  \right) \\ =&P\left( { o }_{ 1 },{ o }_{ 2 },...,{ o }_{ T },{ q }_{ t }={ q }_{ F }|\lambda  \right) ={ \alpha  }_{ T }\left( { q }_{ F } \right) 
\end{align*}
$$






## Decoding : Viterbi Algorithm

우리의 두 번째 관심은 모델 $λ$과 관측치 시퀀스 $O$가 주어졌을 때 가장 확률이 높은 은닉상태의 시퀀스 $Q$를 찾는 것입니다. 이를 디코딩(decoding)이라고 합니다. 포스태깅 문제로 예를 들면 단어의 연쇄를 가지고 품사 태그의 시퀀스를 찾는 것입니다. 우리가 은닉마코프모델을 만드려는 근본 목적에 닿아 있는 문제가 됩니다. 은닉마코프모델의 디코딩 과정엔 **비터비 알고리즘(Viterbi Algorithm)**이 주로 쓰입니다. 

비터비 알고리즘의 계산 대상인 비터비 확률(Viterbi Probability) $v$는 다음과 같이 정의됩니다. $v_t(j)$는 $t$번째 시점의 $j$번째 은닉상태의 비터비 확률을 가리킵니다.



$$
{ v }_{ t }(j)=\max _{ i } ^{n}{ \left[ { v }_{ t-1 }(i)\times { a }_{ ij }\times { b }_{ j }({ o }_{ t }) \right]  }
$$


자세히 보시면 Forward Algoritm에서 구하는 전방확률 $α$와 디코딩 과정에서 구하는 비터비 확률 $v$를 계산하는 과정이 거의 유사한 것을 확인할 수 있습니다. Forward Algorithm은 각 상태에서의 $α$를 구하기 위해 가능한 모든 경우의 수를 고려해 그 확률들을 더해줬다면(sum), 디코딩은 그 확률들 가운데 최대값(max)에 관심이 있습니다. 디코딩 과정을 설명한 예시 그림은 다음과 같습니다.



<a href="https://imgur.com/MXxxdo7"><img src="https://i.imgur.com/MXxxdo7.png" title="source: imgur.com" /></a>



각 상태에서의 비터비 확률 $v$를 구하는 식은 다음과 같습니다. 전방확률을 계산하는 과정과 비교해서 보면 그 공통점(각 상태에서의 전이확률과 방출확률 간 누적 곱)과 차이점(sum vs max)을 분명하게 알 수 있습니다. 비터비 확률 역시 직전 단계의 계산 결과를 활용하는 다이내믹 프로그래밍 기법을 씁니다.



$$
\begin{align*}
v_{ 1 }(1)=&\max { \left[ P(cold|start)\times P(3|cold) \right]  } \\ =&P(cold|start)\times P(3|cold)\\ { v }_{ 1 }(2)=&\max { \left[ P(hot|start)\times P(3|hot) \right]  } \\ =&P(hot|start)\times P(3|hot)\\ { v }_{ 2 }(1)=&\max { \left[ { v }_{ 1 }(2)\times P(cold|hot)\times P(1|cold),\\ { v }_{ 1 }(1)\times P(cold|cold)\times P(1|cold) \right]  }
\end{align*}
$$



Forward Algorithm과 비터비 알고리즘 사이에 가장 큰 차이점은 비터비에 역추적(backtracking) 과정이 있다는 점입니다. 디코딩의 목적은 비터비 확률이 얼마인지보다 최적 상태열이 무엇인지에 관심이 있으므로 당연한 이치입니다. 위 그림에서 파란색 점선으로 된 역방향 화살표가 바로 역추적을 나타내고 있습니다. 

예컨대 2번째 시점 2번째 상태 $q_2$(=HOT)의 backtrace $b_{t_2}(2)$는 $q_2$입니다. $q_2$를 거쳐서 온 우도값(0.32×0.12)이 $q_1$보다 크기 때문입니다. 2번째 시점의 1번째 상태 $q_1$(=COLD)의 backtrace $b_{t_2}(1)$는 $q_2$입니다. 최적상태열은 이렇게 구한 backtrace들이 리스트에 저장된 결과입니다. 예컨대 위 그림에서 아이스크림 [3개, 1개]가 관측됐을 때 가장 확률이 높은 은닉상태의 시퀀스는 [HOT, COLD]가 되는 것입니다.

$t$번째 시점 $j$번째 상태의 backtrace는 다음과 같이 정의됩니다.



$$
{ b }_{ { t }_{ t } }(j)=arg\max _{ i=1 }^n{ \left[ { v }_{ t-1 }(i)\times { a }_{ ij }\times { b }_{ j }({ o }_{ t }) \right]  }
$$





## 모델 학습을 위한 사전 지식

은닉마코프모델의 본격적인 학습에 앞서 학습 과정을 유도하는 데 필요한 용어와 개념 몇 가지 살펴보도록 하겠습니다.




### 전방확률과 후방확률

은닉마코프모델의 파라메터 학습을 위해서는 후방확률 $β$ 개념을 먼저 짚고 넘어가야 합니다. 전방확률 $α$와 반대 방향으로 계산한 것이 후방확률입니다. 그 식은 각각 다음과 같습니다.



$$
{ \alpha  }_{ t }(j)=\sum _{ i=1 }^{ n }{ { \alpha  }_{ t-1 }(i)\times { a }_{ ij } } \times { b }_{ j }({ o }_{ t })\\ { \beta  }_{ t }(i)=\sum _{ j=1 }^{ n }{ { a }_{ ij } } \times { b }_{ j }({ o }_{ t+1 })\times { \beta  }_{ t+1 }(j)
$$



우선 전방확률 $α$ 먼저 보겠습니다. $α_3(4)$는 다음과 같이 구합니다.



<a href="https://imgur.com/mbBaTch"><img src="https://i.imgur.com/mbBaTch.png" title="source: imgur.com" /></a>


$$
{ \alpha  }_{ 3 }(4)=\sum _{ i=1 }^{ 4 }{ { \alpha  }_{ 2 }(i)\times { a }_{ i4 } } \times { b }_{ 4 }({ o }_{ 3 })
$$



이번엔 후방확률 $β$를 보겠습니다. 위와 같은 위치의 후방확률 $β_3(4)$는 다음과 같이 구합니다.



<a href="https://imgur.com/bP9BdJy"><img src="https://i.imgur.com/bP9BdJy.png" title="source: imgur.com" /></a>


$$
{ \beta  }_{ 3 }(4)=\sum _{ j=1 }^{ 4 }{ { a }_{ 4j } } \times { b }_{ j }({ o }_{ 4 })\times { \beta  }_{ 4 }(j)
$$


따라서 $α_3(4)$와 $β_3(4)$를 곱하면 3번째 시점에 4번째 상태일 확률이라는 의미를 가지게 됩니다. 바꿔 말해 $α_3(4)×β_3(4)$는 3번째 시점에 4번째 상태를 지나는 모든 경로에 해당하는 확률의 합을 가리킨다는 겁니다. 이를 도식적으로 나타내면 다음과 같습니다.



<a href="https://imgur.com/3SQDk3b"><img src="https://i.imgur.com/3SQDk3b.png" width="400px" title="source: imgur.com" /></a>



$$
{ \alpha  }_{ t }\left( j \right) \times { \beta  }_{ t }\left( j \right) =P\left( { q }_{ t }=j,O|\lambda  \right)
$$



따라서 특정 시점 $t$의 전방확률과 후방확률을 곱한 값을 모든 상태에 대해 더해 주면 앞서 계산한 우도와 동치가 됩니다. 후방확률을 관측치 시퀀스 맨 끝부터 처음까지 계산하면 이 또한 앞서 계산한 우도와 같습니다.



$$
\begin{align*}
P(O|\lambda )=&P\left( { o }_{ 1 },{ o }_{ 2 },...,{ o }_{ T }|\lambda  \right) \\ =&P\left( { o }_{ 1 },{ o }_{ 2 },...,{ o }_{ T },{ q }_{ t }={ q }_{ 0 }|\lambda  \right) =\beta _{ 0 }\left( { q }_{ 0 } \right) \\ =&\sum _{ s=1 }^{ n }{ \alpha _{ t }\left( s \right) \times \beta _{ t }\left( s \right)  } 
\end{align*}
$$



### 베이즈 정리

베이즈 정리에 의해 다음과 같은 식이 성립합니다.



$$
\begin{align*}
P\left( X|Y,Z \right) =&\frac { P(X,Y,Z) }{ P(Y,Z) } \\ =&\frac { P(X,Y)/P(Z) }{ P(Y,Z)/P(Z) } \\ =&\frac { P(X,Y|Z) }{ P(Y|Z) } 
\end{align*}
$$





### 방출확률 업데이트와 γ

방출확률 $b$를 업데이트하기 위해 $γ$ 개념을 살펴보도록 하겠습니다. $t$시점에 $j$번째 상태일 확률 $γ_t(j)$는 다음과 같이 정의됩니다. 이는 이미 정의된 식과 베이즈 정리에 의해 다음과 같이 다시 쓸 수 있습니다.



$$
\begin{align*}
{ \gamma  }_{ t }\left( j \right) =&P\left( { q }_{ t }=j|O,\lambda  \right) \\ =&\frac { P\left( { q }_{ t }=j,O|\lambda  \right)  }{ P\left( O|\lambda  \right)  } \\ =&\frac { { \alpha  }_{ t }\left( j \right) \times { \beta  }_{ t }\left( j \right)  }{ \sum _{ s=1 }^{ n }{ \alpha _{ t }\left( s \right) \times \beta _{ t }\left( s \right)  }  } 
\end{align*}
$$



$j$번째 상태에서 관측치 $v_k$가 나타날 방출확률 $\hat{b}_j(v_k)$는 다음과 같이 정의됩니다.



$$
{ \hat { b }  }_{ j }\left( { v }_{ k } \right) =\frac { \sum _{ t=1,s.t.{ o }_{ t }={ v }_{ k } }^{ T }{ { \gamma  }_{ t }\left( j \right)  }  }{ \sum _{ t=1 }^{ T }{ { \gamma  }_{ t }\left( j \right)  }  }
$$



위 식의 의미를 해석하면 이렇습니다. 방출확률 $\hat{b}_j(v_k)$의 분모는 $j$번째 상태가 나타날 확률입니다. 분자는 $j$번째 상태이면서 그 때 관측치($o_t$)가 $v_k$일 확률입니다($v_k$가 나타나지 않는 $γ$는 0으로 무시). 분모와 분자 모두에 시그마가 적용된 이유는 방출확률은 시점 $t$와는 무관한 값이기 때문입니다. 어떤 시점이든 $j$번째 상태가 나타날 확률 $γ$가 존재하므로 $T$개 관측치 시퀀스 전체에 걸쳐 모든 시점에 대해 $γ$를 더해주는 것입니다.





### 전이확률 업데이트와 ξ

전이확률 $a$를 업데이트하기 위해 $ξ$ 개념을 살펴보도록 하겠습니다. $t$시점에 $i$번째 상태이고 $t+1$시점에 $j$번째 상태일 확률 $ξ$는 다음과 같이 정의됩니다. 이 또한 베이즈 정리에 의해 다음과 같이 다시 쓸 수 있습니다.



$$
\begin{align*}
{ \xi  }_{ t }\left( i,j \right) =&P\left( { q }_{ t }=i,{ q }_{ t+1 }=j|O,\lambda  \right) \\ =&\frac { P\left( { q }_{ t }=i,{ q }_{ t+1 }=j,O|\lambda  \right)  }{ P\left( O|\lambda  \right)  }
\end{align*}
$$



위 식에서 분자 부분을 이해하기 위해 다음 그림을 보겠습니다. $t$시점에 $i$번째 상태이고 $t+1$시점에 $j$번째 상태인 경우를 도식화한 것입니다.





<a href="https://imgur.com/0QxMZTa"><img src="https://i.imgur.com/0QxMZTa.png" width="500px" title="source: imgur.com" /></a>



지금까지의 정의로 볼 때 $α_t(i)$는 $i$번째 상태 좌측의 모든 path에 해당하는 확률의 합입니다. $β_{t+1}(j)$는 $j$번째 상태 우측의 모든 path에 해당하는 확률의 합입니다. 그런데 이 두 가지 곱만으로는 $i$번째 상태와 $j$번째 상태를 이어주는 path가 존재하지 않습니다. 이를 연결해 주어야 합니다. $i$번째 상태에서 $j$번째 상태로 전이할 확률 $a_{ij}$, $j$번째 상태에서 관측치 $o_{t+1}$를 관측할 방출확률 $b_j(o_{t+1})$까지 곱해주어야 한다는 이야기입니다. 

따라서 $t$시점에 $i$번째 상태이고 $t+1$시점에 $j$번째 상태일 확률 $ξ$은 다음과 같이 다시 쓸 수 있습니다.


$$
\begin{align*}
{ \xi  }_{ t }\left( i,j \right) =&\frac { P\left( { q }_{ t }=i,{ q }_{ t+1 }=j,O|\lambda  \right)  }{ P\left( O|\lambda  \right)  } \\ =&\frac { { \alpha  }_{ t }\times { a }_{ ij }\times { b }_{ j }\left( { o }_{ t+1 } \right) \times { \beta  }_{ t+1 }\left( j \right)  }{ \sum _{ s=1 }^{ n }{ \alpha _{ t }\left( s \right) \times \beta _{ t }\left( s \right)  }  }
\end{align*}
$$


$i$번째 상태에서 $j$번째 상태로 전이할 확률 $\hat{a}_{ij}$는 다음과 같이 정의됩니다.


$$
\hat { a } _{ ij }=\frac { \sum _{ t=1 }^{ T-1 }{ { \xi  }_{ t }\left( i,j \right)  }  }{ \sum _{ t=1 }^{ T-1 }{ \sum _{ k=1 }^{ N }{ { \xi  }_{ t }\left( i,k \right)  }  }  } 
$$


위 식의 의미는 이렇습니다. 분모는 $i$번째 상태에서 전이할 수 있는 모든 path들의 확률들을 더한 값입니다. 분자는 $i$번째 상태에서 $j$번째 상태로 전이할 확률을 가리킵니다. 분모와 분자 모두에 $Σ_{t=1}^{T-1}$가 적용된 이유는 방출확률은 시점 $t$와는 무관한 값이기 때문입니다. 어떤 시점이든 $i$번째 상태에서 다른 상태로 전이할 확률 $ξ_t$가 존재하므로 관측치 시퀀스 전체에 걸쳐 모든 시점에 대해 $ξ_t$를 더해주는 것입니다. 다만 시퀀스 마지막 $T$번째 시점에선 종료상태로 전이될 확률이 1이 되므로 $t$에 대해 1에서 $T-1$까지 더해줍니다. 

위 식을 그림으로 도식화하면 아래와 같습니다. 위 식 분모는 굵은 적색 점선 화살표, 분자는 검은색 화살표입니다.



<a href="https://imgur.com/G0FUlgo"><img src="https://i.imgur.com/G0FUlgo.png" title="source: imgur.com" /></a>





## Training : EM Algorithm

은닉마코프모델의 파라메터는 전이확률 $A$와 방출확률 $B$입니다. 그런데 이 두 파라메터를 동시에 추정하기는 어렵습니다. 지금까지 설명한 날씨 예제를 기준으로 하면, 우리가 관측가능한 것은 아이스크림의 개수뿐이고 궁극적으로 알고 싶은 날씨는 숨겨져 있습니다. 따라서 HOT에서 COLD로 전이할 확률은 물론 날씨가 더울 때 아이스크림을 1개 먹을 방출확률 따위를 모두 한번에 알 수 없습니다. 이럴 때 주로 사용되는 것이 EM 알고리즘입니다. 은닉마코프모델에서는 이를 '바움-웰치 알고리즘' 또는 'Forward, Backward Algorithm'이라고도 부릅니다.




### E-step

모델 파라메터 $λ$, 즉 전이확률 $A$, 방출확률 $B$를 고정시킨 상태에서 관측치 $O$를 바탕으로 전방확률 $α$와 후방확률 $β$를 업데이트합니다. 이 $α$와 $β$를 바탕으로 $γ$와 $ξ$를 각각 계산합니다.




### M-step

E-step에서 구한 $γ$와 $ζ$를 바탕으로 모델 파라메터 $A$, $B$를 업데이트합니다. 



### pesudo code

EM 알고리즘의 의사코드는 다음과 같습니다. 단 아래 코드에서 $γ,ξ$의 분모는 표기가 다를 뿐 해당 관측치의 우도를 나타냅니다.



<a href="https://imgur.com/ukU14ub"><img src="https://i.imgur.com/ukU14ub.png" width="600px" title="source: imgur.com" /></a>


