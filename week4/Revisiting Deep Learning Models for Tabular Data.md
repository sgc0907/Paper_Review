## 읽기 전에
이 요약은 철저히 개인적인 입장에서 작성되었으며, 해석에 틀린 부분이 있을 수 있습니다.

## 논문의 기여점
1. 다양한 task의 DL 모델들을 평가함.
2. ResNet이 Baseline이 될 수 있음을 확인.
3. FT-Transformer라는 새로운 솔루션 제안.
4. GBDT와 DL간에는 보편적으로 뛰어난 솔루션이 없음을 확인.

## 관련 연구
Tabular Data에 대한 머신 러닝 연구는 지속되어 왔음. <br>
지금까지는 DT의 앙상블이 보통 좋은 성능을 보임. GBDT(Gradient Boosting Decision Tree) - XGB, LGBM등 <br>
최근 DL모델은 3가지 방향으로 개발됨.

- Differentiable trees : 앙상블에서 영감을 얻어, end-to-end 방식으로 훈련하기 위해 Decision Fucntions를 Smooth시킴. 일부 GBDT를 능가하며, ResNet을 능가하지는 못 했음.
- Attention 기반 : 본 paper의 실험에서는 ResNet이 보다 우수한 성능을 보임. Transformer 아키텍쳐를 사용하는 FT-Transformer(이후에 제시됨)는 ResNet을 능가하는 경향을 보이기도.
- Multiplicative interactions의 명시적 모델링 : feature product를 MLP에 통합하는 다양한 방법이 있음. 본 paper의 실험에서는 성능이 좋지 못함.

이외에도 여러가지 모델이 있는데, 비교하고 좋은 성능을 보이는 것을 식별할 예정.

## Tabular Data를 위한 Model (비교할 모델)
- MLP
- ResNet : 원래 아키텍쳐보다 단순화된 것 사용.
- FT-Transformer : 새로 제안된 구조. 모든 feature를 embedding시키고, 이후 여러 개의 transformer layer stack을 적용. <br>
성능이 좋다고는 하는데, 리소스를 많이 잡아먹는다고 한다. <br>
MHSA(Multi-Head Self-Attention)의 quadratic complexity때문에 그렇다고 함. 이는 효율적인 근사치를 사용하여 완화할 수 있음.
- 다른 모델 : SNN, NODE, TabNet, GrowNet, DCN V2, AutoInt, XGBoost, CatBoost

## 실험
최대한 기본 모델로 적용. pretraining, Loss Function, data Augmentation등 일절 관여 안함. <br>
데이터셋은 11개의 공개 데이터셋 사용
- 전처리 : 모두 동일한 전처리. (quantile transformation, standardization, Epsilon데이터셋에서는 사용X)
- 튜닝 : 대부분 Bayesian 최적화. 
- 평가 : test set.
- 앙상블 : 15개의 단일 모델을 동일한 3개의 배타적인 그룹으로 나누고 예측 평균.
- NN : 분류-Cross Entropy, 회귀-MSE, TabNet, GrowNet은 Adam, 다른건 AdamW, 스케줄러 없음, 미리 정의된 batch size 사용. patience = 16
- Cat Features : XGB는 원핫인코딩, CatBoost는 알아서. 나머지는 임베딩 되는 방식.
- 15개 랜덤시드.

### 실험 결론
	- MLP는 여전히 좋은 안정성 확인 방법임
	- ResNet은 Baseline으로 적합. (Global하게 뛰어난 모델 없음)
	- FT-Transformer는 대부분의 task에서 좋은 성능.
	- Tuning은 MLP, ResNet과 같은 간단한 모델을 경쟁령 있게 만듬.

NODE가 좋을 때도 있지만, parameter수 많고, ResNet과 FT-Transformer와 유사하게 생김. <br>
ResNet과 FT-Transformer는 앙상블을 했을 때 더 많은 이점이 있음.

### DL과 GBDT를 비교 <br>
    GBDT는 기본적으로 앙상블이므로, DL도 앙상블시켜 비교를 진행.
    Default H.param : FT-Transformer가 2개 데이터셋 제외하고 다른 모델보다 높은 성능.
    Tuned H.param : 일부 데이터셋에서 FT-T보다 우세해질 때 있음(격차 큼). DL이 낫고 GBDT가 낫고 이런게 없다는것. GBDT는 class수가 많을 경우 부적절할 수 있음. 느리거나 성능이 안 좋거나.
	    결론 : DL, GBDT의 보편적 솔루션은 없음. GBDT가 우수한 데이터셋을 기준으로 DL 개발을 해야 함.
    GBDT가 우수한 데이터셋에서는 FT-T와 ResNet은 유사한 성능을 보임.

## 분석
### FT-T가 ResNet보다 나을 때는 언제? <br>
	ResNet-Friendly target : ResNet와 FT-T는 유사.
	GBDT-Friendly target : ResNet성능 뚝떨어짐, FT-T는 GBDT와 유사.
	--> GBDT>ResNet인 경우 성능 FT-T > ResNet
	--> GBDT<=ResNet인 경우 성능 FT-T ~~ ResNet

### Ablation study
	1. FT-T와 AutoInt(Feature Embedding후 Self-Attention적용)를 비교. 뛰어남.
	2. FT의 feature biases가 필요함을 확인했음.
	
### Attention maps로부터 feature importance 얻기
	IG와 유사하게 작동함을 확인함. Attention map을 이용하는 것이 cost대비 효율적일 수 있음.


## 결론
    Tabular data에서의 DL
    ResNet류의 아키텍쳐가 baseline으로 사용될 수 있음.
    FT-Transformer는 대부분 다른 DL 솔루션 능가
    GBDT는 여전히 일부 task에서 우위.

## 개인적인 생각?
- FT-T는 역시 학습에 오래 걸림.
- 저자의 주장대로라면 새로운 Baseline정도로 사용할 수 있지 않을까 싶음. 어차피 global하게 좋은 것은 아니니까.
- 데이터에 따라서 GBDT와 NN류의 모델 성능 차이가 보이면, 그 데이터의 특징이 뭔지 궁금하다.
    - H.param를 Tuning할 대 기준으로, CA, AD, YA를 사용했을 때 GBDT가 우수했다.
    - California Housing(CA) : 20640, num8, RMSE
    - Adult(AD) : 48842, num6, cat8, Acc, class2, cat이 있는 유일한 데이터셋
    - Yahoo(YA) : 709877, num699, RMSE
- Categorical Features를 가진 데이터가 AD밖에 없었던 것이 아쉬움. 허나 이것은 이후에 저자들이 총 2개의 Cat.Features가 포함덴 데이터셋을 포함해서 추가 실험했다고 함.
    - Real world Dataset은 Cat.Features도 충분히 사용하는 게 도움 될 때가 있으니까, Categorical Features가 포함된 데이터에 대한 더 많은 실험이 필요해 보임.
- Transformer 모델은 입력 시퀀스 순서에 영항을 받을 것 같은데? 그런 영향은 따로 없는건가 궁금함.






