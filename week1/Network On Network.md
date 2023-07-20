## 배경

* Tabular data는 현실 세계에서 고차원의 데이터로 많이 쓰임.
* 기존 연구들의 문제점
    * 각 field의 정보(intra-field information)를 고려하지 않음.
    * data에 상관 없이 Operation이 정해져 있음.
    * Operation간의 비선형적 관계를 고려하지 않음.

## Network On Network의 구조
* 전반적인 구조
    * Field-wise network
    * Across field network
    * Operation fusion network

### Field-wise network
- intra-field information을 추출하기 위함.
- 각 field는 DNN을 가짐.
- 각 field 안의 feature들은 embedding되어 field가 가진 DNN에 넣어짐.

### Across field network
- 서로 다른 field간의 상호작용을 학습하기 위함.
- 여러 Operation조합을 사람이 미리 설정(선형 모델, DNN, self-attention, FM & Bi-Interaction 등).
- data-driven된 방식으로 학습됨.

### Operation fusion network
- Operation 사이의 상호작용을 더 잘 포착하기 위함.
- 표현력이 좋은 DNN을 사용

### Auxiliary loss
- 매 DNN layer마다 Aux loss 적용.
- 정보 손실, Gradient 소실 방지.
- training때만 사용, inference할 때는 사용하지 않음.

## 실험
* aux loss를 통해 Training step을 많이 줄임 (1.67배 감소)
* 6개 real-world dataset을 통해 실험
    * ablation study 결과, 각 요소를 추가할 수록 성능 증가
    * baseline model과의 비교 결과, SOTA 성능을 확인

## 개인적으로 드는 생각
* 정확도 비교도 좋지만, training/inference time을 확인해보고 싶다. DNN 구조가 워낙 많이 쌓여 있기 때문.
* field-wise network 실험 결과로 2차원 투영한 plot을 보여주는데, 단 두개의 field만 비교한 것이 아쉬움.
* Baseline model과 NON을 비교할 때, AUC Score 등으로 비교를 진행하는데, 단 한번의 실험 결과를 기재한 것이 아쉬움.
    * 여러번 실험을 한 결과를 토대로 편차치를 확인해야 유의한 성능 변화인지 확인 가능할 듯.

## Q&A
* Aux loss를 매 layer마다 쌓으면 기존 loss의 영향이 줄어들지는 않는가?
    * 그렇다. 그래서 weight에 영향을 주는 것을 막기 위해 α라는 특정 상수 값을 곱해 이 정도를 조절.
    * 원본이 되는 GoogLeNet에서는 α=0.3으로 두고 진행.
    * NON 논문에서는 α값을 확인 불가능했음. layer마다 α의 값을 다르게 두고, Training 동안 점차 decay된다고 함.

* Across field network에서 Operation을 모델이 학습할 때 알아서 골라야 하는데, 방식이 궁금함
    * 논문에 기재되어 있지 않아 확인이 불가능
    * 기본적으로 DNN은 포함시키고, 나머지 요소들을 추가하는 방식으로 진행됨.
    * 다른 아키텍쳐에서는 기본적으로 모두 성능을 체크하고 제일 좋은 방향으로 선택됨.

* Field-wise network에서 Numerical 변수는 그대로 다시 DNN에 넣는데 의미가 있는지?
    * 없어보임. input이 하나고 output도 하나인데 여기서 어느 정보를 추출할 수 있을지 의문.
    * 구조 통일을 위해 넣은 것이 아닐지?
    * DNN이 비선형적 젼환을 수행하기도 하고, 해당 field의 패턴을 포착하게 될 지는 명확하게 밝혀진 부분이 없으므로, 실험적으로 밝혀야 될 것으로 보임.
