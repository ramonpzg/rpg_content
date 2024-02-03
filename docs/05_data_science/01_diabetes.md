# XAI in Healthcare

## Table of Contents

1. Overview
2. Q&As
3. Dependencies
4. Data
5. Model Training
6. Model Evaluation
7. Model Interpretation
8. Summary
9. Exercises

## 1. Overview

Diabetes is a chronic, metabolic disease characterized by elevated levels 
of blood glucose (or blood sugar), which leads over time to serious damage to the 
heart, blood vessels, eyes, kidneys and nerves. There are different kinds of diabetes 
diseases and, sadly, a lot of people can be living with at least one of them without knowing it.

The issue is so pronounced that the International Diabetes Federation said

> The IDF Diabetes Atlas (2021) reports that 10.5% of the adult population (20-79 years) 
has diabetes, with almost half unaware that they are living with the condition. By 2045, 
IDF projections show that 1 in 8 adults, approximately 783 million, will be living with 
diabetes, an increase of 46%.

That said, imagine we could turn to machine learning to help us detect who might be diabetic 
and not know it, or who could be closer to becoming one? Wouldn't that be not only ideal, but also 
desirable by everyone? I think it is but it is important to keep in mind that a simple message 
with "Hey, just wanted to tell that you have diabetes!" won't suffice. Context and a good explanation 
will go a long way and can also provide the patient with useful information for his/her/their loved 
ones, should some of the variables causing the illness were to be highly hereditary.

With this overview out of the way, let's examine some questions our patients may want to know.

## 2. Q&As

When talking to a professional that used an algorithm to tell me if I am sick or not, I would 
most certainly ask the following (at the very least).

1. Why did I get flagged as diabetic or potentially becoming one in the near future?
2. What led to this conclusion?
3. What should I do next?

Some potential answers "I" would like to hear if my doctor was telling me that I have diabetes 
would be:
1. Your level of glucose was extremely high and this, and other variables, contributed towards 
flagging you as having diabetes. If you had a fasting blood glucose test, a level between 70 
and 100 mg/dL (3.9 and 5.6 mmol/L) is considered normal. Yours was at 130.
2. The combination of high glucose and blood preassure, your age, and your skin thickness, increased 
the likelihood of you having diabetes when compare to other patients similar to you.
3. We would need to first have you take a few tests to figure out if you this is a false positive or 
if you indeed have diabetes and, if so, of which kind (type I or type II). In addition, we would 
be giving you guidelines and steps to take to bring your glucose to normal levels, and we will also 
ask you to come again soon for more regular check ups.

More or less that would make me feel a bit more positive about the situation. Nonetheless, there are 
many factors leading to diabetes that would take too long to go over so let's get started with our 
example. 😎

## 3. Dependencies

Here are the packages we will be using in this notebook.

- `scikit-learn`
- `pandas`
- `joblib`
- `matplotlib`
- `alibi`
- `statsmodels`
- `mlserver`


```python
!pip install scikit-learn pandas joblib matplotlib alibi numpy rich mlserver
```

## 4. Data

The dataset we will be using can be downloaded from kaggle 
[here](https://www.kaggle.com/datasets/akshaydattatraykhare/diabetes-dataset). It is 
an abreviated version from the **National Institute of Diabetes and Digestive and Kidney 
Diseases**. This dataset contains a very specific information such as all femaile, over 21, 
and from a particular region of India, but it can serve as a starting point for a much 
larger project with many more variables.

Why it matters? Machine learning is great at finding patterns in the data, and we should use 
this tools, with the appropriate measures in place, as best we can to enhance human life. A lot 
of illnesses become fatal ones due to late detection, therefore, if there is a way to help people 
of all backgrounds while keeping their information and rights safe and intact, we should be doing 
what we can to push the needle forward.

Description or dictionary of variables.
- `Pregnancies` - number of pregnancies.
- `Glucose` - glucose level.
- `BloodPressure` - blood pressure.
- `SkinThickness`
- `Insulin` - level of insulin in the blood.
- `BMI` - body mass index.
- `DiabetesPedigreeFunction`
- `Age`
- `Outcome` - target variable.

Let's get started evaluating loading and evaluating our data.


```python
from sklearn.model_selection import train_test_split
from rich import print
import pandas as pd
```


```python
df = pd.read_csv('data/diabetes/diabetes.csv')
df.head()
```


```python
df.shape
```


```python
y = df['Outcome']
X = df.drop(['Outcome'], axis=1).copy()
```


```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.25, random_state=9)
```

## 5. Model Training

For this section, we will use a logistic regression because of its high degree of interpretability 
and ease of use via the packages described two sections back.

If you have never used a logistic regression before, you can think of it as a classification 
algorithm used to predict a binary outcome (e.g. 1 or 0, white or black, cat or not cat, etc).

For a quick example, suppose you are an instructor that wants to predict if a student will 
pass an exam (a binary, yes/no outcome) based on hours studied, previous grades, and, 
potentially, other variables. Your process might look as follows.

1. Convert the output to a probability value between 0-1. This will represent the chance of passing.

2. Use a linear model to combine the input features and calculate a 'score'. 
    
    $score = Intercept + HoursStudied * \beta_1 + PreviousGrades * \beta_2$

3. Step 3: Convert this score to a probability using the logistic function:
    
    $probability = \frac{1}{(1 + e^{(-score)})}$
    This squashes the score to between 0 and 1.

4. If probability > 0.5, predict the student will pass. Otherwise predict they will fail. The 
probabilities provide a measure of confidence in the binary predictions.

While the above example is a shorter version of the process, my hope is that it gives you an intuition 
for how the method works in practice. 

Now, let's train our model.


```python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
from sklearn.linear_model import LogisticRegression
import matplotlib.pyplot as plt
import seaborn as sns
import joblib

sns.set(rc={'figure.figsize':(11.7,8.27)})
```

Feel free to experiment with the parameters below.


```python
lr_cls = LogisticRegression(random_state=0, max_iter=500, verbose=0)
```


```python
lr_cls.fit(X_train, y_train)
```


```python
lr_cls.coef_
```

If you don't have the path below, you can create with the following command in the termianl.

```sh
mkdir -p models/diabetes/
```


```python
model_path = 'models/diabetes/lr_cls_diabetes.pkl'
```


```python
joblib.dump(lr_cls, model_path)
```


```python
!ls models/diabetes
```


```python
lr_cls = joblib.load(model_path)
```

Let's do a quick sanity check before we move on to thoroughly evaluating our model. For this, we will 
pick a random sample from the test dataset.


```python
x = X_test.sample(1)
y = y_test[x.index[0]]
print(y)
x
```


```python
lr_cls.classes_
```


```python
lr_cls.predict_proba(x)
```


```python
y_pred = lr_cls.predict(X_test)
```


```python
cm = confusion_matrix(y_test, y_pred)
```


```python
title = 'Confusion matrix for Logistic Regression'
disp = ConfusionMatrixDisplay.from_estimator(
    lr_cls, X_test, y_test, 
    display_labels=['Not Diabetic', 'Diabetic'],
    cmap=plt.cm.Blues, normalize=None
)
disp.ax_.set_title(title);
```

## 6. Model Evaluation

The first method we will explore is called [Kernel SHAP](https://shap-lrjball.readthedocs.io/en/latest/generated/shap.KernelExplainer.html#:~:text=Kernel%20SHAP%20is%20a%20method,modelfunction%20or%20iml.Model).

> It is a model interpretation method that explains individual predictions by determining 
the contribution of each feature. It uses concepts from game theory and local surrogate models 
to quantify feature importance.

Here's an analogy to understand Kernel SHAP:

Imagine a fruit smoothie prediction model. The ingredients are bananas, strawberries, yogurt, 
and ice. The model predicts how sweet the smoothie will taste.

To explain an individual prediction, Kernel SHAP is like asking:

"How much did each ingredient contribute to the overall sweetness?" 

It determines the SHAP value, or impact, of each feature by comparing smoothies with and without 
that ingredient. Bananas may get a high positive SHAP value because they make smoothies much 
sweeter. Ice may have a negative SHAP value since it dilutes the sweetness. By summing the SHAP 
values for all features, Kernel SHAP explains the total predicted sweetness. It reveals why 
the model predicted that particular level of sweetness given those ingredients.

Like this, Kernel SHAP attributes the prediction of any complex model to each input feature. The 
analogy helps convey how it quantifies each feature's contribution, like ingredients in a recipe. This 
makes model behavior more interpretable.

You can learn more about Kernel Shap [here](https://docs.seldon.io/projects/alibi/en/stable/examples/kernel_shap_wine_intro.html).

Let's get started implementing `KernelShap`.


```python
from alibi.explainers import KernelShap
```


```python
explainer = KernelShap(lr_cls.predict_proba, task='classification')
explainer
```

Explainers in Alibi work in the same fashion as estimators in sklearn, that is, they follow the 
`.fit()` and `.predict()` way of doing things so if you are familiar with sklearn, this step will 
feel familiar to you.


```python
explainer.fit(X_train)
```

Once we finish creating an explainer, the object we get back gives us a lot of useful information like the one above.

Note that, running an explainer in a large batch of data can be quite compute intensive (depending on the 
explainer of course), so it is good practice to save your models once your code finishes creating them. Let's 
save ours, load it and test it again.


```python
explainer_path = 'models/diabetes/lr_cls_explainer.pkl'
```


```python
joblib.dump(explainer, explainer_path)
```


```python
explainer = joblib.load(explainer_path)
```


```python
x = X_test.sample(1)
y = y_test[x.index].iloc[0]
print(y)
x
```


```python
features = X_train.columns.to_list()
features
```

As you might have noticed in the metadata returned to us once we trained our model, `KernelShap` is both 
local and global, which means that it can be applied to one or many samples at a time. Let's try it on our 
random sample from above.


```python
result = explainer.explain(x)
```


```python
result.shap_values[0]
```


```python
explainer.predictor(x)
```

What we're interested in is the `shap_values` returned by our explainer. Let's see what these look like.


```python
result.shap_values
```


```python
def plot_importance(feat_imp, feat_names, class_idx):
    df = pd.DataFrame(data=feat_imp, columns=feat_names).sort_values(by=0, axis='columns')
    feat_imp, feat_names = df.values[0], df.columns
    fig, ax = plt.subplots(figsize=(10, 5))
    y_pos = np.arange(len(feat_imp))
    ax.barh(y_pos, feat_imp)
    ax.set_yticks(y_pos)
    ax.set_yticklabels(feat_names, fontsize=15)
    ax.invert_yaxis()
    ax.set_xlabel(f'Feature effects for class {class_idx}', fontsize=15)
    return ax, fig
```


```python
import numpy as np
```


```python
plot_importance(result.shap_values[1], features, 'Diabetes');
```


```python
import shap
```


```python
result = explainer.explain(X_train[:100])
```


```python
shap.summary_plot(result.shap_values[0], X_train[:100], features);
```

A positive SHAP value means the feature pushed the output higher. Negative means it pushed the output lower.

It is important to note that, if we train the explainer on a large amount of data (with some compute expenses), the 
explainer would have learned enough about the model globally to locally explain the interactions for new cases.

## 7. Model Interpretation

While the explainer method we choose, Kernel Shap, to explain our model is considered a black-box method, 
because we choose a logistic regresion as our class, we can also interrogate each of the coefficients of our 
model and interpret further the result we got.

To do this, we will make use of `statsmodels` to fit a model again because it has a very nice summary table.


```python
# !pip install 'alibi[shap]'
!pip install 'alibi[ray]'
```


```python
import statsmodels.api as sm
```


```python
log_model = sm.Logit(y_train, sm.add_constant(X_train))
log_result = log_model.fit()
```


```python
print(log_result.summary2())
```

In the table above we can examine not only the coefficients of each parameter, but also the standard deviation and 
AIC and BIC values of our model.

Because the coefficients are the logarithms of the odds (i.e. the probability of a positive case over 
the probability of a negative case), we can convert them back into exponentials to get a better sense of 
what each value means.


```python
np.exp(log_result.params).sort_values(ascending=False)
```

What do the odds mean for a diabetic patient? It means that the odds of having Diabetes increase by a factor of 2.95 for each additional unit of DiabetesPedigreeFunction, or by 1.14 for every new pregnancy, provided every other feature stays unchanged.

We need more context for this, and that can be achieved with the standard deviation.


```python
coefs = log_result.params.drop(labels=['const'])
stdv = np.std(X_train, 0)
abs(coefs * stdv).sort_values(ascending=False)
```

The preceding table can be interpreted as an approximation of risk factors from high to low 
according to the model. It is also a model-specific feature importance method, and a global one 
at that (as it was gather from a group of samples). It tells us how far away from the mean each of 
these values are.

## 8. Summary

1. Explainable AI (XAI) is a set of techniques and methods that enable machine learning models 
to provide clear, understandable, and human-interpretable explanations for their predictions, 
helping users develop trust and make informed decisions based on machine learning outputs.

2. Kernel SHAP is a specific XAI method that attributes the impact of each input feature on a 
model's prediction by using a game theoretic framework, providing insights into how features 
influence the model's outcomes.

3. Logistic regression is a statistical model commonly used in machine learning for binary 
classification tasks. It estimates the probability of a binary outcome by fitting a linear 
function to input features and applying a logistic (sigmoid) transformation.

4. XAI can be used to enhance transparency, accountability, and trust in AI systems, 
especially in critical domains like healthcare, where decisions based on AI can have significant 
real-world consequences. XAI helps uncover the black-box nature of complex models and makes 
their reasoning accessible to human experts.

5. Machine Learning can assist in diagnosing diseases in deceased individuals by analyzing medical data, 
potentially improving the accuracy and efficiency of diagnoses.

## Bonus: Serving our Models

To serve models and explainers together, you can run a server with both models using `mlserver`. 
To do so, run the following command.

```sh
python servers/diabetes/cls_diabetes_service.py
```

You can test that your server is working with the following commands.


```python
from mlserver.codecs import NumpyCodec
import requests
```


```python
x.values, y
```


```python
inf_request = {
    'inputs': [
        NumpyCodec.encode_input(name='payload', payload=x.values).dict()
    ]
}
print(inf_request)
```

Change the name from **classifier** to **explainer** and back to change the endpoint you 
are hitting.


```python
model = 'diabetes_classifier'
endpoint = f"http://0.0.0.0:8080/v2/models/{model}/infer"
r = requests.post(endpoint, json=inf_request)
r.json()
```

## 10. Exercises

### Build an Explainer on Hospital Readmission Data

- Load the hospital [readmission dataset](https://www.kaggle.com/datasets/dubradave/hospital-readmissions). No 
need to use all features. 10 features including age, medications, vitals, etc., would suffice.
- Train a logistic regression model to predict readmission.
- Create a Kernel Shap explainer using Alibi, or use another method from a different Alibi like AnchorTabular 
or CounterFactual, or from another library.
- Write a short narrative describing the result. 
- Compare the explanations if you choose more than one model - which features do they highlight? How do they differ? 
and so on.
