---
layout: post
title: Sample post
tags: [ReinforcementLearning, machineLearning]
excerpt_separator: <!--more-->
---

딥 강화학습 쉽게 이해하기
이 글은 머신러닝 블로그 nervanasys 저자 Tambet의 허락을 받아 원문 DEMYSTIFYING DEEP REINFORCEMENT LEARNING의 딥러닝 글을 번역한 것입니다. 원문도 꼭 읽어보셨으면 합니다.

저자는 DEMYSTIFYING DEEP REINFORCEMENT LEARNING 의 글이 더 정확하므로 여기를 참고하라고 알려왔습니다. 저도 번역한 후에 이 곳을 다시참조해서 업데이트 하였습니다.

이 글은 딥 강화학습 시리즈의 첫 번째 글입니다. NEON 딥러닝 툴킷을 사용하여 실제 향상을 하는 것은 두 번째 글 Deep Reinforcement Learning with Neon로 이어집니다.

2년 전 오늘 (2015년 12월 19일 이 글이 쓰임), 런던의 작은 회사인 딥마인드가 선지적인 논문 딥 강화학습으로 아타리 게임하기라는 논문을 논문 사이트에 내놓았습니다. 이 논문에서는 딥마인드가 스크린 픽셀들과 게임점수가 높을 때 주는 리워드만을 주면서 어떻게 컴퓨터가 아타리 2600게임을 학습할 수 있는지 보여주었습니다. 게임도 다를 뿐더러 모든 게임들의 목표가 매우 달랐고, 이를 디자인하기가 어려웠기에 매우 인상적인 결과였습니다. 각각의 게임마다 변화를 주지않고 동일한 모델의 아키텍쳐로 7가지의 다른 게임을 실행했는 데, 3가지는 알고리즘이 사람들보다 오히려 더 나은 결과를 보였습니다!

이 사건은 일반적 AI로 나아가기위한 첫 단추를 꿰메었다고 여겨집니다. 즉, AI가 체스를 게임하는 것 같은 제한된 영역에 국한되지 않고, 다양한 환경에서 살아남을 수 있다는 것입니다.

당연하게도 딥마인드는 구글에 즉각 인수되었고 딥러닝 연구에 있어 가장 앞서나가고 있다는 것에는 의심의 여지가 없습니다. 2015년 2월에는 딥 강화학습을 통한 인간 수준의 컨트롤하기라는 딥마인드의 논문이 과학계에서 가장 저명한 잡지중 하나인 네이처지의 커버를 장식했었습니다. 이 논문에서는 같은 모델로 49가지 다른 게임을 수행했고 절반 이상에서 사람 이상의 능력을 보여주었습니다.

지도학습이나 비지도학습을 통한 딥 모델들을 업계에서 광범위적으로 사용하고 있고, 아직까지도 딥 강화학습은 약간의 미스터리로 남아있긴 합니다. 이 포스트에서 이런 기술적인 부분들을 설명한 후에 뒷 부분에서 어떤 논리로 동작하는 지 이해해 볼 것 입니다. 이 글을 읽어주셨으면 하는 분들은 뉴럴 네트워크에 관련한 머신러닝에 어느정도 배경이 있지만, 아직까지 딥 강화학습을 파고들 시간은 없었던 분들입니다.

역주: 뉴럴네트워크에 대해서 잘 모른다면 제 블로그의 다른 글부터 한 번 읽어보세요. 제가 생각하기에 최고의 듀토리얼들을 번역해서 올려두었습니다.
뉴럴 네트워크 짜는 법 배우기
11줄의 파이썬 코드로 뉴럴 네트워크를 만들어보자
로드맵은 다음과 같습니다.

딥 강화학습에서 가장 힘든 장벽은 무엇인지? 신뢰할당문제(credit assignment problem)와 탐색-이용(exploration-exploitation) 딜레마에 관해서 다룹니다.
수학적으로 딥 강화학습을 어떻게 표현할 수 있는지? 마르코프 결정과정(Markov Decision Process)을 정의하고 강화학습에 관한 추론을 위해 마르코프 결정과정을 사용 해봅니다.
장기적인 관점에서 어떻게 바라볼 것인지? 다음 섹션에서 가장 기본이 되는 알고리즘인 “차감된 미래 리워드”(discounted future reward)에 관해 정의해 볼 것입니다.
어떻게 미래의 리워드 대해 추정하거나 대략적인 근사값을 만들어낼 수 있는지? 간단한 테이블-베이스 Q-러닝 알고리즘을 정의하고 이해해 볼 것입니다.
상태 공간이 너무 크면 어떻게 할 것인지? 여기서 어떻게 Q-table 알고리즘들이 (딥)뉴럴네트워크로 대체될 수 있는 지 알아 볼 것입니다.
실제로 동작키위해서는 무엇이 필요할까요? 반복경험 테크닉들을 논하고, 뉴럴네트워크를 통해 학습을 안정화 해 봅니다.
아직 안 끝났나요? 마지막으로 탐색-이용(exploration-exploitation)의 간단한 해결책들을 고려 해 봅니다.
강화학습
벽돌깨기를 생각해봅시다. 이 게임은 화면 아래에 있는 바를 움직여서 공을 튕겨낸 후에 화면 상단의 절반을 차지하고 있는 벽돌을 전부 없애는 게임입니다. 벽돌을 하나씩 깰 때마다 벽돌은 사라지고 점수는 올라갈 것입니다. 다시 말해 리워드를 얻는 겁니다.

아타리 벽돌깨기 게임. 이미지 출처 : 딥마인드

뉴럴네트워크에게 이 게임을 가르친다고 생각해봅시다. 네트워크에 입력하는 값들은 화면 이미지일 것이고, 결과는 세 가지 행동으로 이루어집니다. 왼쪽, 오른쪽, 발사(처음 공을 띄울때). 이 문제는 분류문제로 생각 할 수 있습니다. 결정해야하는 각각의 게임의 화면에서, 당신은 왼쪽, 오른쪽, 혹은 공을 발사시키는 행동을 할 것입니다. 쉬워 보이죠? 물론 트레이닝 예시들이 필요하고, 이 예시들은 정말 많이 필요합니다. 물론 여러분들이 나가서 전문 게이머들이 게임하는 장면들을 녹화할 수도 있겠지만, 우리가 진짜 배우려고 하는 것들은 이런 것들이 아닙니다. 각각의 화면에서 어디로 움직여야 할 지 누군가에게 수백만번 말하라고 할 필요는 없습니다. 잘 하고 있는지 가끔 피드백을 받고 그것으로 우리가 모든 것들을 알아낼 수 있습니다.

이것이 바로 강화학습이 풀고자 하는 문제입니다. 강화학습은 지도학습과 비지도학습 사이 어딘가에 위치하고 있습니다. 지도학습은 각각의 트레이닝 예시를 위한 목표 라벨이 있는 반면 비지도 학습은 라벨이 아예 없는데, 강화학습은 뜨문 뜨문히, 시간이 지연되는 리워드 라벨들을 가지고 있습니다. 이러한 리워드만을 바탕으로 각각의 상황에서 에이전트들은 어떻게 행동해야할 지 학습해야합니다.

몇 번을 부닥치며 실행하는 이 아이디어는 꽤 직관적입니다. 예를 들면, 벽돌을 부수고 점수로 리워드를 얻는 벽돌 게임에서는 보통 (작대기를 움직이는) 행동을 하지 않고 점수를 얻기 직전에만 행동하면 됩니다. 모든 힘든 일이 끝나고 나면, 작대기 위치를 적절한 위치에 두어 공을 튕겨내기만 하면 됩니다. 이것을 신뢰 할당 문제라 부릅니다. 다시 말하면 이전의 행동들이 점수를 얻고 어떤 규모인지 믿을 수 있어야 합니다.

여러분이 리워드의 특정 점수만을 얻는다는 원칙을 세워 놓았다고 해봅시다. 그렇다면 더 많은 리워드를 얻을 수 있음에도 불구하고 과연 이 원칙을 고수해야 할까요? 위의 벽돌깨기 게임에서는 화면 왼쪽 끝에 작대기를 옮겨놓고 기다리는 간단한 원칙을 생각해 볼 수 있겠네요. 게임이 시작되면, 공은 오른쪽보다는 왼쪽으로 올 가능성이 좀 더 높기 때문에 죽기 직전에 10포인트를 쉽게 얻을 수 있을 것입니다. 그러면 여기에서 그냥 만족을 할 지 아니면 좀 더 많은 것을 원할지 고민해야합니다. 이를 탐색-이용 딜레마(explore-exploit dilemma) 라 일컫습니다. 다시 말하면 이미 알고 있는 방법을 고수할 것인지(exploit) 아니면 가능한 다른 방법들을 찾아 볼 것인지(explore) 더 나은 방법을 택해야합니다.

배우는 방법은 (보통 모든 동물이 그렇겠지만) 강화학습에서는 매우 중요한 모델입니다. 부모님의 칭찬, 학교를 졸업하고, 직장에서 받는 연봉. 이런 것들이 리워드의 예시라고 할 수 있습니다. 신뢰 할당 문제와 탐색-이용 딜레마(explore-exploit dilemma)는 우리가 매일 비즈니스와 여러 관계들에서 부닺히는 문제들입니다. 이 문제를 공부하는 가장 중요한 이유이기도 하고, 새로운 접근을 시도하기 위한 멋진 놀기 좋은 모양의 게임이기도 합니다.

마르코프 의사결정 모델
그렇다면 문제는 어떻게 강화학습 문제를 공식화해서 그것으로 추론할 수 있느냐는 것입니다. 마르코프 의사 결정 과정을 사용하는 것이 가장 일반적인 방법입니다.

여러분이 에이전트라 생각하고, 어떤 환경(예를 들면, 벽돌게임) 에 처해져있다고 생각 해봅시다. 환경은 특정한 상태안에 있습니다(바의 위치, 공의 방향과 위치, 모든 벽돌의 존재 유무 등). 에이전트는 환경안에서 특정한 행동을 수행합니다(바를 왼쪽이나 오른쪽으로 옮기는 것). 때로는 이 행동들은 결과로 리워드를 얻습니다(점수가 올라가는 것). 행동들은 환경을 바꾸고, 에이전트가 또 다른 행동을 만들어내는 새로운 상태로 이끌어줍니다. 이런 행동들을 선택하는 방법을 지침이라 합니다. 보통 환경은 확률적으로 만들어지는데, 이는 다음 상태가 어느정도는 무작위적이라는 것입니다.(예를 들면, 공을 놓쳐버리고 난 후에는 새로운 공이 발사되죠. 이 공이 어느 방향으로는 모르죠.)

Figure2

상태와 행동들의 모임으로 구성되어있고, 하나에서 또 다른 하나로 이전되는 규칙은 마르코프 의사결정과정을 만듭니다. 유한한 상태, 행동, 리워드들로 이 과정의 한 에피소드(예를 들면, 게임 한 턴)를 만듭니다.

여기에는 상태를 대신하는 , 행동을 대신하는 , 행동을 수행한 결과에 따른 리워드 가 있습니다. 에피소드는 종료 상태 (예를 들면, “게임 오버”가 스크린에 뜨면)가 되면 끝이 납니다. 마르코프 의사 결정 과정은 다음 상태 의 확률은 오직 현재의 상태 와 현재의 행동 에만 영향을 받고, 이전의 상태와 행동에는 영향을 받지 않는 마르코프 가정을 기반으로 합니다.

차감된 미래의 리워드(Discounted Future Reward)
(역주: 차감된 미래의 리워드란 단어는 매우 어색하지만, 적당한 단어를 찾기가 쉽지않고, 번역사례가 있어 이 단어로 대신했습니다. 적절한 단어가 생각나신 분께서는 댓글로 추천 해주시면 감사합니다.)

오랜 시간동안 좋은 수행능력이 나오기 위해서 당장의 리워드를 포함해서, 미래에 얻을 리워드도 반영할 수 있어야합니다. 이것을 어떻게 할 수 있을까요?

주어진 마르코프 과정이 한 번 실행된 것을 보면, 한 에피소드의 ‘리워드 합계’를 쉽게 계산할 수 있습니다.

one Episode

이에 따르면, 시간 시점 에서 얻는 미래의 총 리워드는, 아래와 같은 식으로 표현할 수 있을 것입니다.

futuer reward T

그러나 환경은 확률적이기 때문에, 같은 리워드를 받아서 다음 번에 같은 행동을 수행했다고 해서 절대로 이를 확신할 수 없습니다. 미래에 대해 파고들면 들수록, 더 나뉘어지게 됩니다. 이런 이유로 일반적으로 미래의 리워드를 차감하여 사용합니다.

위의 0과 1사이의 차감 변수입니다. 리워드가 미래로 가면 갈수록, 적게 반영합니다. 밑에서 볼수 있듯, 시간 시점 t에서 차감된 미래는 시간시점 t+1의 시점에서 동일하게 표현할 수도 있습니다.

만약 차감변수 라 둔다면, 우리 지침은 바로 앞의 미래만 생각하고 즉각적인 리워드만 반영합니다. 만약 즉각적인 리워드와 미래의 리워드 사이에 균형을 찾길 원한다면, 차감변수를 처럼 설정해야합니다. 만약 환경이 절대적이고, 같은 행동이 언제나 동일한 리워드를 제공한다면, 차감변수 로 설정해야합니다.

에이전트에 가장 좋은 지침은 (차감된) 미래 리워드를 극대화시켜서 행동을 택하게 하는 것일 것입니다.

Q-러닝
Q-러닝에서는 함수 가 각 지점에서 계속 최적값을 찾으면서 상태 에서 행동 를 수행할 때 차감된 미래의 리워드(discounted future reward)를 나타내는 것이라고 정의합니다.

를 “상태 안에서 행동 를 수행한 후에 게임이 끝난 시점에서의 최고 점수”라고 생각하면 됩니다. 주어진 상태에서 특정 행동의 “품질(quality)”을 표현하기에 이것을 Q-함수라고 부릅니다.

이 정의는 꽤 애매모호하게 들릴지 모릅니다. 미래의 행동과 리워드가 아니라 현재의 상태와 행동만 알고 있다면, 어떻게 게임 끌날 때의 점수를 예측할 수 있을까요? 사실 할 수 없죠. 그렇지만 함수의 존재등으로 이론적으로는 구성해 볼 수는 있습니다. 눈을 감고 다섯 번만 반복 해보세요. “는 존재한다…는 존재한다…” 뭔가 느껴지시나요?

만약 아직까지도 확신이 서지않는다면, 그 함수가 무엇을 내포하는 지 생각 해보세요. 여러분이 상태 안에 있고, 행동 와 둘 중에 어떤 행동을 선택해야할 지 숙고하고 있다고 생각 해 보세요. 게임이 끝날 때에 가장 높은 점수를 얻을 수 있는 행동을 선택하고 싶다고 해 봅시다. 그리고 마법의 Q-함수를 가지고 있다면 답은 정말 간단해집니다. 최고 높은 Q-값과 함께 행동을 취하면 됩니다!

이 는 각각의 상태에서 어떤 행동을 선택하는 규칙인 지침을 의미합니다.

자, 그러면 어떻게 Q-함수를 만들 수 있을까요? 그러면 한 번만 틀어서 로 생각해봅시다. 바로 전 문단 ‘차감된 미래의 리워드’에서처럼, 상태 와 행동 에서의 Q-값을 다음 상태인 관점에서 표현할 수 있습니다.

이 식을 벨멘 방정식(Bellman equation)이라고 부릅니다. 생각 해 볼 수 있듯, 즉각적인 리워드와 다음 상태에서의 미래의 리워드 최댓값를 합하면 꽤 논리적으로 이 상태의 미래의 리워드 최댓값이 나옵니다.

Q-러닝의 요지는 벨멘 방정식을 이용해서 반복적으로 Q-함수를 근사시킬 수 있는 것입니다. 표에서 상태(states)들을 행에, 행동(actions)들을 열에 두면, 가장 간단하게 Q-함수를 향상시킬 수 있습니다. Q-러닝 알고리즘의 골자는 아래와 같이 매우 간단합니다.

initialize Q[numstates,numactions] arbitrarily
observe initial state s
repeat
    select and carry out an action a
    observe reward r and new state s'
    Q[s,a] = Q[s,a] + α(r + γmaxa' Q[s',a'] - Q[s,a])
    s = s'
until terminated

Q[상태 갯수, 행동 갯수] 를 임의적으로 초기화시킨다. 
초기 상태s를 조사한다. 
반복
  행동 a를 선택하고 수행한다. 
  리워드 r를 관찰하고 새로운 상태 s'로 만든다.
  Q[s,a] = Q[s,a] + α(r + γmaxa' Q[s',a'] - Q[s,a])
  s = s' (상태를 s'로 바꾼다)
끝날 때까지 반복
알고리즘의 (알파)는 알고리즘 내에서 이루어진 이전의 Q-값과 새로 제시된 Q-값의 차이를 컨트롤 하는 학습률 입니다. 특히, 이면, 두 는 취소되고 벨맨 방정식과 완전히 동일한 방법으로 업데이트 시킵니다.

은 근삿값이고 학습의 매우 초기 단계에서 만든 를 사용하여 업데이트를 하기 때문에 완전히 틀린 값이 나올 것입니다. 그러나 여기서 볼 수 있듯이 반복을 할 때마다 조금씩 조금씩 더 정확해 집니다. 그래서 충분한 시간동안 수행한다면 Q-함수는 수렴할 것이고, 진짜 Q-값을 대신할 수 있을 것입니다.

딥 Q 네트워크
환경이 벽돌깨기라면 그 상태로 바(패들)의 위치, 공의 위치와 방향, 각각의 벽돌의 존재유무를 들 수 있을 것입니다. 하지만 직관적으로 보면 대표하는 것은 게임 상황일 것입니다. 이런 것들을 좀 더 일반적으로 만들어서, 모든 게임에 적용가능할까요? 확실한 방법은 스크린 픽셀들을 선택하는 것입니다. 공의 속도와 방향을 제외하고는 게임 상황내의 적절한 모든 정보들을 내포합니다. 연이은 두 화면들은 공의 속도와 방향 또한 모두 포함할 것입니다.

딥마인드 논문의 방법과 같은 방법으로 게임스크린을 전처리 한다면, 마지막 네 개의 스크린 이미지만 가지고, 이를 로 크기를 변환하고, 그레이레벨로 구성된 그레이 스케일로 변환합니다. 그러면 개의 가능한한 게임 상태를 가집니다. 이 의미는 가상의 Q-표에서 행을 가진다는 것입니다. 즉, 우리가 알고 있는 우주의 원자 개수보다 더 많은 것이죠! 어떤 분은 많은 픽셀 조합(상태가 되는)들은 절대 일어나지 않는다고 말씀할 수도 있겠죠. 즉, 우리는 방문한 상태들만을 포함하여 뜨문 뜨문 표를 만들어 대신할 수 있을 것입니다. Q-테이블을 수렴시키려면 우주 시간이상이 걸리기 때문에 대부분의 상태는 매우 드물게 방문 될 것입니다. 이상적으로는 우리가 한 번도 보지못한 곳의 Q-값도 좋은 예측값으로 가지고 있어야합니다.

딥러닝이 들어와야할 단계가 바로 이 시점입니다. 특히 뉴럴 네트워크는 탄탄히 구성된 데이터를 위한 좋은 특징들과 함께라면 매우 좋은 결과를 냅니다. 상태(4개의 게임 화면들)와 Q-값를 따르는 입력값과 결과값으로 구성된 행동들을 수행하는 뉴럴네트워크로 Q-함수를 대신할 수 있습니다. 각각의 가능한 행동을 위한 Q-값들의 인풋과 아웃풋으로 몇몇 게임화면들을 대안으로 사용할 수 있습니다. 이 접근방법은 Q-값 업데이트를 수행하던지, 가장 높은 Q-값과 함께하는 행동을 선택할 수 있는 이점이 있습니다. 우리는 네트워크를 이용한 진행경로에서는 단 한 방향으로만 수행할 수 있고, 즉시 행동이 가능하도록 Q-값 모두를 가지게 됩니다.

Figure3

딥마인드의 네트워크 아키텍쳐는 아래와 같습니다.

딥마인드 아키텍쳐

완전히 연결된 레이어 두 개가 뒤이어오는 convolution 레이어 3개를 가지고 있는 전통적인 convolutional 뉴럴 네트워크입니다. 보통은 당기는 레이어들이 없는 객체 인식 네트워크들은 많이 알고 있을 것입니다. 당기는 레이어들은 형태의 불변성을 가져다 주기 때문에 꼭 한 번 생각 해보셔야합니다. 다시 말하면, 네트워크는 이미지 안의 객체의 위치에는 무감각합니다. 이미지넷과 같이 분류시키는 것에는 확실하게 반응합니다. 그러나 게임에서 공의 위치는 잠재적인 리워드를 결정짓는 것에 중대한 정보이기 때문에, 이 정보를 버릴 수는 없죠!

네트워크의 입력값은 네 개의 84x84그레이 스케일의 게임화면입니다. 네트워크의 결과값은 가능한 행동 (아타리(벽돌깨기)의 경우 18개) 각각의 Q-값들입니다. Q-값들은 회귀 조건을 만들고 간단한 제곱근 오차(square error loss)에 최적화되는 그 어떤 실제 변수도 될 수 있습니다.

L = target-prediction

주어진 바뀐 값 으로, Q-테이블은 이전의 알고리즘의 규칙들을 아래와 같은 방식으로 대체되고 업데이트 합니다.

모든 행동들을 위해 예측된 Q-값들을 가질 수 있도록 현재 상태 를 위해 전향적으로 통과시킵니다(feedforward pass).
다음 상태 을 위해 전향적으로 통과시키고, 총체척인 네트워크 결과값의 최댓값 를 계산합니다.
(2단계에서 계산된 최댓값)를 향하는 행동 을 위해 Q-값의 대상을 설정합니다. 남은 다른 행동들을 위해서, 원래 1단계의 반환된 것과 같이 Q-값들을 설정하여 그 결과들의 에러를 0으로 만들어 줍니다.
역전파법을 활용하여 웨이트를 업데이트해줍니다.
재현 경험하기(Experience Replay)
이제 우리는 Q-러닝을 이용해서 각각의 상태에서 미래의 리워드를 추정하는 방법과 convolutional 뉴럴 네트워크를 이용해서 Q-함수를 근사시키는 방법에 대한 아이디어가 생겼습니다. 그러나 비선형적인 함수를 사용해서 Q-값들을 예측하면 매우 안정적이지 않습니다. 이것을 수렴시킬 수많은 요령들이 있습니다. 그리고 GPU하나로 돌린다면 거의 1주일이나 소요될 정도로 오랜 시간이 걸리는 일입니다.

가장 중요한 요령은 재현을 경험시키는 것입니다. 게임을 하는 동안 모든 경험들 은 재현 메모리에 저장됩니다. 네트워크를 훈련시키는 동안, 가장 최근의 보다는 재현 메모리의 무작위적인 샘플들이 사용될 것입니다. 이 방법은 차후의 훈련 예시들의 유사성을 무너뜨리거나 네트워크를 극솟점으로 이동시켜 줍니다. 또한 재현을 경험시키는 것은 간단하게 디버깅하고 알고리즘을 테스트할 수 있는 일반적인 지도학습과 비슷한 훈련 업무로 만들어 줍니다. 실제로는 사람이 게임을 플레이하는 모든 경험들을 수집하여 이들로 네트워크를 훈련시킬 수도 있습니다.

탐색-이용(Exploration-Exploitation)
Q-러닝은 신뢰할당문제를 해결하려고 합니다. 진짜 리워드를 얻게하는 중요한 결정 포인트에 다다를 때까지 게속 리워드를 전파합니다. 아직까지 우리는 탐색-이용 딜레마(exploration-exploitation dillema)에 대해 다루지 못했습니다.

첫 번째로 관찰할 수 있듯, Q-테이블과 Q-네트워크가 무작위로 초기화되면, 그에 따른 예측 또한 랜덤적으로 초기화됩니다. 가장 높은 Q-값을 선택하면 행동은 아마 무작위적이고, 에이전트는 대충 “탐색”을 수행할 것입니다. Q-함수가 수렴하면 더 많은 거듭된 Q-값들을 반환하고, 탐색의 양은 줄어들게 됩니다. 누군가 말한 것처럼, 이 Q-러닝은 알고리즘의 일부분으로써 탐색을 만들게 됩니다. 그렇지만 이 탐색은 “(욕심)greedy”이 많아서, 찾은 방법 중에 첫번째 효율적인 방법에 안착합니다.

위 문제를 간단하고 효율적으로 고치기 위해서는 무작위적인 행동을 선택하는 확률 ε를 이용하는 ε-greedy exploration을 사용합니다. 그 외에는, 가장 높은 Q-값을 이용해서 “greedy” 행동을 수행합니다. 사실 딥마인드 시스템에서는 시간이 지나면서 ε를 1에서 0.1로 만들어줍니다. 즉, 시스템 시작시점에서는 상태 공간을 최대화시켜 검색하도록 완전히 무작위적으로 만들어 준 후, 고정된 탐색률에 안착시키는 방법을 사용합니다.

Deep Q-러닝 알고리즘
재현을 경험하며 최종으로 만든 Q-learning 알고리즘은 다음과 같습니다.

initialize replay memory D
initialize action-value function Q with random weights
observe initial state s
repeat
    select an action a
        with probability ε select a random action
        otherwise select a = argmaxa’Q(s,a’)
    carry out action a
    observe reward r and new state s’
    store experience <s, a, r, s’> in replay memory D

    sample random transitions <ss, aa, rr, ss’> from replay memory D
    calculate target for each minibatch transition
        if ss’ is terminal state then tt = rr
        otherwise tt = rr + γmaxa’Q(ss’, aa’)
    train the Q network using (tt - Q(ss, aa))^2 as loss

    s = s'
until terminated

Q[상태 갯수, 행동 갯수] 를 임의적으로 초기화시킨다. 
초기 상태s를 조사한다. 
반복
  행동 a를 선택하고 수행한다. 
    무작위적인 행동 a를 선택하는 확률 ε를 사용한다. 
    다른 것들은 select a = argmaxa’Q(s,a’)
  행동 a를 끄집어낸다.
  리워드 r를 관찰하고 새로운 상태 s'로 만든다.
  재현 매모리 D에 경험 <s, a, r, s’> 을 저장한다.

  무작위적인 샘플 <ss, aa, rr, ss'>를 재현 메모리 D로부터 끄집어낸다.
  각각의 미니 배치 계산을 위한 목표를 계산한다.
    만약 ss' 이 끝나는 상태이면 tt = rr
    그렇지 않으면 tt = rr + γmaxa’Q(ss’, aa’)
  게임에서 질 때까지 (tt - Q(ss, aa))^2 을 이용해서 Q 네트워크를 트레이닝 시킨다.
  s = s' (상태를 s'로 바꾼다)
끝날 때까지 반복
사실 딥마인드가 이를 실제로 작동시키면서 타켓 네트워크, 에러 클리핑, 리워드 클리핑 등 여러 요령들을 더 사용하지만, 듀토리얼 범위에서는 벗어나는 내용들입니다.

이 알고리즘의 가장 멋진 부분은 어떤 것이든지 배울 수 있다는 것입니다. 한 번 생각 해보세요. Q-함수가 무작위적으로 초기화되었기 떄문에, 결과값은 정말 말도 안되죠. 그리고 우리는 이 말도 안되는 값을(다음 상태의 최대 Q-값)를 네트워크의 목표로서 사용하며, 몇 안되는 리워드들로 드문드문 조금씩 모읍니다. 이상한 소리로 들릴지 모르지만, 이렇게 해서 정말 전체에서 의미있는 것들로 배울까요? 실제로는 정말 그렇게 됩니다.

마지막 노트
Q-러닝이 처음 소개된 이후, 많은 향상기법이 만들어졌습니다. Double Q-learning, Prioritized Experience Replay, Dueling Network Architechture, extension to continuous action space 같은 것들이 유명한 것들이겠지요. 가장 최근의 향상 동향을 확인하시려면, NIPS 2015 deep reinforcement learning workshop과 ICLR 2016(제목에 “reinforcement”로 검색해보세요). 그러나, 딥 Q-러닝은 구글이 특허를 가지고 있다는 것은 명심하시기 바랍니다.

세간에서 말하듯 어떤 의미로는 인공지능은 아직 생각도 못했을 수도 있습니다. 어떻게 동작하는 지 알게되면 더 이상 똑똑해 보이지 않기 마련입니다. 그렇지만 딥 Q-네트워크는 여전히 제게 놀랍습니다. 새로운 게임에 대해 생각하며 리워드에 대해 그 자신이 직접 경험하는 것을 보면 야생동물을 관찰하는 것만 같습니다.

참조
초안을 보고 코멘트와 의견을 남겨준 Ardi Tampuu, Tanel Pärnamaa, Jaan Aru, Ilya Kuzovkin, Arjun Bansal and Urs Köster 에게 특히 감사를 전합니다.


