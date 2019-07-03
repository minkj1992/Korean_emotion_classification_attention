# korean emotion classification with Attention
2018년 짧은 한국어 문장을 8가지 감정으로 분류하는 모델입니다.

![](./전체모델.jpg)

- `전체 순서`: `Konlpy` -> `word2vec` -> `Bi-lstm` -> `Attention` -> `8 emotion classify`

# 0. 도입부
일일이 sns 댓글을 뒤지지 않고도 손쉽게 주변인물들이 당신을 어떻게 생각하는지를 판단할 수 있다.

why? 사람과의 관계의 모호함, 문맥을 잃어버림. 기계는 다르다. 

필요한 이유는 기계가 감정을 파악할수 있다. 감정이 파악이 중요한 이유는 
# 1. emotion classifer


what:타인의 글속에 숨겨진 당신에 대한 감정을 판단해드립니다. EMOtion classifier


-------------------------------------------------
정보의 홍수
오늘날은 
우리는 현재 수많은 network를 가지며 살고 있습니다. 

거울사진
타인들이 나를 어떻게 생각할까 때로는 두려워합니다.

이재용사진, 기업들 이미지들
기업들 수많은 제품들또한 마찬가지 입니다.



혹시 수많은 글들 속에서 나에대한 감정을 판단해 줄 수 있다면 어떻까요?
좀 더 안전하게 행동을 할 수 있습니다.

Emotion classifier
타인들의 글속에 숨겨진 당신에 대한 감정을 판단해드립니다.





-----15초----
# 3.구현 방법

1) 논문 보여줌 무수한 논문들 스터디 5초
기본 지식이 없어서 적합한 모델을찾기 위하여 여러 연구 논문들을 참조하였고, 최종적으로 Attention 모델을 계획하였습니다.
2) 구현 방법 정리된 사진 과 순서 10초
전체 모델은 트위터 문장이 주어졌을경우 Embedding을 거쳐 벡터화 시켜준 데이터를 Lstm을 통하여 문장을 정방향 역방향으로 읽어 문맥을 파악해주고, 추가적으로 Attention score를 계산하여 중요한 단어들은 좀더 부각시켜줍니다. 최종적으로 1트위터당 11가지의 감정 발현 확률이 제공됩니다.

3) 영상을 통해서 실구현 장면 설명(3분)

## `embedding`
- 1.`Word Embedding layer`
한국어 단어들간의 관계를 학습시키기 위해, 네이버 영화 후기, 트위터를 무작위로 크롤링하여, 1기가의 텍스트 파일을 생성
이를 형태소 token 단위로 자른뒤, 벡터화 시켜주기 위하여 word2vec기법을 사용하여 300차원의 -1~1값을 가진 벡터들을 dict형식으로 생성.
최종적으로 각 형태소당 300차원 벡터형태의 사전이 완성되었습니다. 


- 2.`Preprocessing`

학습에 사용될 train validation test dataset을 준비하고 load시켜줍니다. 이 데이터들은 각 문장당 11가지의 감정 발현여부가 라벨링 되어있습니다. 
이후 앞서 생성한 사전을 load시켜주고, 이를 활용하여  각 문장들을 index화 시켜줍니다. 
이때 사전에 등재되지 않은 형태소들을 무작위로 300차원의 값을 할당해 줍니다. 이후 각 문장들을 100개의 토큰으로 길이를 padding 시켜줍니다.

최종적으로 학습모델의 input: 100개의 index로 구성된 문장들 약 30000개, 그리고 각 index들은 300차원의 word vector를 대표한다. shape(30000,100,300)

- 3. `Language Model`
100개의 token index들을 양방향 lstm에 넣어주어 문맥을 파악합니다. 
이때 각 문장당 300*100 크기의 matrix를 500차원(250*2)으로 정보를 압축시켜줍니다.

추가적으로 중요한 단어들을 계산해주기 위해서, 3겹의 tanh layer를 거친 후 softmax층에 넣어주어 attention score를 계산합니다.

이를 앞서 생성한 문맥 정보와 합하고 곱한 뒤, sigmoid층을 거쳐 0~1사이의 11가지 감정 값을 예측합니다.
이후 jaccard coefficient similarity를 통하여 loss를 구하고 adam optimizer를 사용하여 30번정도 학습을 진행합니다.

#index 문서들을 넣어주고, embedding layer를 거치면 한 문장당(1,100,300) matrix를   
#앞 뒤로 읽어주는 500차원(250*2)의 양방향LSTM에 넣어주어 정보를 압축시켜준다. 이때 한 문장에 들어있는 단어#(token)당 문장에서 얼마만큼의 주목도를 가지는지 평가하기 위해서
#각 단어당 500차원의 hidden state들을 tanh layer와 softmax층에 넣어주어 attention score를 계산한다.

#최종적으로 한 문장을 구성하는 100개의 token들에 대한 attention score를 합쳐주어 스칼라 값을 계산해주고, 이를 #(100,500) Bi_lstm hidden state에 곱한뒤, 100개의 token들을 합쳐주면
#한 문장당 attention score이 계산된  (500)차원의 벡터가 생성된다. 이러한 문장이 30000개 있으므로 (30000,500) #를 dense(11)인 신경망에 넣어주고 sigmoid 값을 구해주면 
#각 감정 라벨당 구현 정도가 0~1 사이의 값이 생성되고 이를 y정답값과 jaccard coefficient similarity를 구해주  #어 loss값으로 전달해주며 이를 adam optimizer에 전달해주어 backpropagation을 진행해 준다.

# 4.적용 영상
국가부도의 날 댓글 감정을 예측해보겠습니다. 높은 점수를 준 댓글을 예측해보니  기쁨과 약간의 분노로 예측을 하였습니다. 분노의 경우 댓글에서 영화 보는 내내 답답했다를 해석한 것 같습니다.

반대로 부정적인 댓글을 예측해보겠습니다. 임베딩 되지 못한 단어들이 다소 존재하지만, 결과적으로 분노와 혐오가 예측되었습니다.

# 5. 결론
장점: 단순히 감정 단어들의 횟수를 세는 방식이 아닌, 문맥에 따른 감정 판단을 하여, 어순에 따라서 감정값이 다르게 나온다.

단점: 100단어 이하의 짧은 문장은 100문장으로 패딩을 시켜서 임의의 값을 만들어 내므로 정확도가 떨어진다. 다만 장문의 경우에는 문맥을 판단할 확률이 높아진다.


학습데이터가 영문 트위터 번역 데이터인 관계로, 한국의 신조어들에 대한 감정 학습이 다소 적게 발현하였다. 

