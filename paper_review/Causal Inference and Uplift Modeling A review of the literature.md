# Causal Inference

주로 CATE를 구하고 싶은 경우가 많다.

- $Y_i(1)$: person $i$’s outcome when he receives the active treatment
- $Y_i(0)$: person $i$’s outcome when he receives the control treatment

$$CATE: \tau (X_i) = E[Y_i (1)|X_i] - E[Y_i (0) | X_i]$$

observational data를 통해서 CATE를 구하기 위해서는 _Unconfoundedness Assumption_ (_Conditional Independence Assumption_)을 만족해야한다.

$$CIA: Y_i (1), Y_i (0) \perp T_i | X_i$$

# Uplift Modeling

uplift modeling은 결국 CATE를 estimate하는 것이다. 그래서 주로 CATE가 큰 유저군에 대해 treatment를 주는 접근을 많이한다. CATE를 추정하는 방법은 아래와 같이 3가지 정도가 있다.

## The Two-Model Approach

- $E[Y_i (1)|X_i], E[Y_i (0)|X_i]$를 각각 모델링하는 방법이다.
- 간단하지만 다른 방법에 비해 효과가 떨어진다.

## The Class Transformation Method

- binary outcome의 경우를 가정한다. (continuous의 경우는 아직 확실한 논문은 없는 것 같다)
- under CIA,

$$E[Y_i^* | X_i] = \tau(X_i)$$

아래의 과정을 통해서 위와 같은 결과를 얻을 수 있다.

- $e(X_i)=P(T_i=1|X_i)$는 propensity score

$$Y^*_i = Y_i \cdot \dfrac{T_i - e(X_i)}{e(X_i)(1-e(X_i))}$$

$$
\begin{align}
E[Y^*_i|X_i] &= E\big[Y_i \cdot \dfrac{T_i - e(X_i)}{e(X_i)(1-e(X_i))}|X_i\big] \\
&= E\big[Y_i T_i \cdot \dfrac{T_i - e(X_i)}{e(X_i)(1-e(X_i))} + Y_i (1-T_i) \cdot \dfrac{T_i - e(X_i)}{e(X_i)(1-e(X_i))}|X_i\big]\\
&= E\big[Y_i (1) \cdot \dfrac{T_i(1 - e(X_i))}{e(X_i)(1-e(X_i))} | X_i\big] - E\big[Y_i(0) \cdot \dfrac{(1-T_i)e(X_i)}{e(X_i)(1-e(X_i))}|X_i\big]\\
&= \dfrac{1}{e(X_i)} E[Y_i(1) \cdot T_i|X_i] - \dfrac{1}{1-e(X_i)} E[Y_i(0) \cdot (1-T_i)| X_i]\\
&= \dfrac{1}{e(X_i)} E[Y_i(1)|X_i] E[T_i|X_i] - \dfrac{1}{1-e(X_i)} E[Y_i(0)|X_i] E[(1-T_i)| X_i]\\
&= E[Y_i(1)|X_i] - E[Y_i(0)|X_i]\\
&= \tau(X_i)
\end{align}
$$

- 식 (1)에서 (2)로 넘어갈 때는 $Y_i = Y_i T_i + Y_i (1-T_i)$가 이용되었다.
- 식 (2)과 식 (3)은 각각 $T_i$이 0 또는 1인 경우를 비교하면 동일하다. 뒤의 수식과정에서 $e(X_i)$를 지우기 위한 과정이라고 이해 할 수도 있다.
- 실제 데이터를 사용한 예시는 [여기](https://github.com/minsoo9506/causality-study/blob/master/Causal_Inference_for_the_Brave_and_True_practice/20_Plug_and_Play_Estimators.ipynb)에서 확인할 수 있다.

## Modeling Uplift Directly

- 다양한 ML 모델을 이용하여 treatment effect를 추정하는 방법이다.
- 지금은 트리계열이 가장 많이 사용된다.
  - split criterion choice에 따라 다양한 모델들이 발표되었다.

# Evaluation

## Traditional Uplift Metrics

- observation에서 control, treatment outcome을 모두 확인할 수 없기 때문에 loss를 measure하기 어렵다.
- 그래서 대부분 uplift bins, uplift curves를 사용한다.
- 논문의 설명과 그림을 같이 보면 좋다.

## Metric Based on $Y^*$

위에서 transformation 방법론을 토대로 MSE loss function을 쓸 수 있을까?

$$MSE(Y^*_i, \hat{\tau}) = \sum_i^n \frac{1}{n}(Y_i^* - \hat{\tau}_i)^2$$

위의 식을 통해서 아래의 식을 approximation하는 것이 가능할까?

$$MSE(\tau_i, \hat{\tau}_i) = \sum_i^n \frac{1}{n}(\tau_i - \hat{\tau}_i)^2$$

해당 논문에서는 이에 대히 수식적으로 증명하고 있다.
