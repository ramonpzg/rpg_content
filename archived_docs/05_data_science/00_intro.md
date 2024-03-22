# Introduction

## Table of Contents

1. Overview
2. Fake Data
    - What is it?
    - Why is it useful?
    - How do we create it?
3. Synthetic Data
    - What is it?
    - Why is it useful?
    - How do we create it?
4. Generative Data
    - What is it?
    - Why is it useful?
    - How do we create it?
5. Final Thoughts
6. Exercises

## 1. Overview

Data plays a pivotal role in numerous fields, but obtaining real-world data can be challenging due 
to privacy concerns, limited access, or high costs. To address these issues, various data generation 
techniques have emerged. This notebook provides insights into three key methods: Synthetic Data, Fake 
Data, and Data Generated from Generative Machine Learning Models.

In the following sections, we'll dive deeper into the three kinds of data generation techniques.

## 2. Fake Data

### What is Fake Data?

Fake data refers to any data that is artificially generated rather than collected from real-world sources. It is made up data.

### Why is it useful?

Fake data can be useful for a variety of purposes including Testing systems or applications without needing 
real data, populating databases when real data is not available, training machine learning models as a stand-in 
for real data, and anything that requires dummy data for placeholders.

The main advantage of using fake data is that it protects privacy since no real personal information is 
exposed. It also provides flexibility since the data can be generated on demand.

### How do we create it?

To start using fake data, we need to identify the types of data needed (names, addresses, images etc.) and 
find a fake data generator that suits our needs. For our use case, we'll use `mimesis`, "an effective Python 
library for populating databases, creating intricate JSON/XML files, anonymizing productive service data, and 
generating high-quality Pandas dataframes."

Let's walk through a few examples using mimesis.


```python
from mimesis import Person, Datetime, Text, Generic
from mimesis.locales import Locale
from mimesis.enums import Gender
```


```python
person = Person(Locale.EN)
person??
```


```python
person.full_name(gender=Gender.FEMALE)
```


```python
person.full_name(gender=Gender.MALE)
```


```python
person = Person(Locale.UK)
datetime = Datetime(Locale.UK)
text = Text(Locale.UK)

print(f"Person: {person.full_name()}; Date-Time: {datetime.date()}; Text: {text.quote()};")
```


```python
generic = Generic(locale=Locale.EN)
generic.person.username()
```


```python
generic.datetime.date()
```


```python
p_en = Person(Locale.EN)
p_sv = Person(Locale.SV)
p_en.full_name(), p_sv.full_name()
```


```python
g = Generic(locale=Locale.ES)
g.datetime.month(), g.code.imei(), g.food.fruit()
```


```python
from mimesis import Fieldset
```


```python
fs = Fieldset(i=100)
```


```python
fs('email', key=lambda x: x.split('@')[0].replace('xxxx')
```

## 3. Synthetic Data

### What is Synthetic Data?

Synthetic data is artificially generated to mimic real-world data. It is based on statistical analysis of real data.


### Why is it useful?

Synthetic data can be useful for augmenting small real datasets for training machine learning models, running 
simulations when real data is not available or very minimal, testing systems and models without exposing 
sensitive real data, sharing datasets while preserving privacy.

The key advantage of synthetic data is it looks and behaves like real data but does not contain private 
information. It also lets you generate large datasets even with a small sample of real data.

### How do we create it?

To start using synthetic data:
- Collect a sample of real representative data. 
- Analyze statistics like distributions, correlations, etc.
- Use a synthetic data generator with those stats to simulate distribution of your variables.
- Validate that the synthetic data matches real data.

Iterate by tweaking the data generation process until the synthetic data is realistic.

In essence, synthetic data provides a blueprint of real data without the bricks and mortar of actual 
details. Like a movie set, it creates the illusion of reality from analysis of the real thing. Synthetic 
data lets you mimic and scale real-world data more safely.

Let's walk through a few examples using the synthetic data vault (`svd`) Python package.




```python
from sdv.lite import SingleTablePreset
from sdv.datasets.demo import download_demo
from sdv.evaluation.single_table import get_column_plot, evaluate_quality, get_column_pair_plot
```


```python
real_data, metadata = download_demo(modality='single_table', dataset_name='fake_hotel_guests')
```


```python
real_data.head()
```


```python
metadata.visualize()
```


```python
synthesizer = SingleTablePreset(metadata, name='FAST_ML')
```


```python
synthesizer.fit(data=real_data)
```


```python
synthetic_data = synthesizer.sample(num_rows=500)
synthetic_data.head()
```


```python
sensitive_column_names = ['guest_email', 'billing_address', 'credit_card_number']
real_data[sensitive_column_names].head(3)
```


```python
synthetic_data[sensitive_column_names].head(3)
```


```python
quality_report = evaluate_quality(real_data, synthetic_data, metadata)
```


```python
quality_report.get_visualization('Column Shapes')
```


```python
fig = get_column_plot(real_data=real_data, synthetic_data=synthetic_data, column_name='amenities_fee', metadata=metadata)
fig.show()
```


```python
fig = get_column_pair_plot(real_data=real_data, synthetic_data=synthetic_data, column_names=['checkin_date', 'checkout_date'], metadata=metadata)
fig.show()
```


```python
synthesizer.save('my_synthesizer.pkl')
synthesizer = SingleTablePreset.load('my_synthesizer.pkl')
synthesizer
```

## 4. Generative Data

### What is Synthetic Data?

Generative models learn patterns from training data in order to generate new synthetic samples.

Two popular kinds of models are diffusion and transformers. Diffusion models add noise to data 
and learn to gradually denoise it. Transformers predict sequence data using attention mechanisms.

### Why is it useful?

Generative models are useful for simple to high quality creative assets, synthetic training data (images, 
text, etc), or for data augmentation when we have limited samples.

### How do we create it?

To start with generative models:
- Pick a pre-trained model from the Hugging Face model hub, PyTorch Hub, or any other hub available.
- Play with it via the `transformers` or the `diffusers` library.
- Find a dataset that interests you.
- Fine-tune the model on your dataset.

In essence, generative models are like artists studying the world and learning to mimic it. After 
observing enough data, they can synthesize new examples that capture the essence of their training. We 
guide them from imitation towards imagination.


```python
from transformers import AutoModelForSeq2SeqLM, AutoTokenizer
```


```python
checkpoint = "bigscience/mt0-small"

tokenizer = AutoTokenizer.from_pretrained(checkpoint)
model = AutoModelForSeq2SeqLM.from_pretrained(checkpoint)
```


```python
tokenizer.encode??
```


```python
inputs = tokenizer.encode("Translate to English: Je t’aime.", return_tensors="pt")
inputs
```


```python
inputs = tokenizer.encode("Translate to English: Je t’aime.", return_tensors="pt")
outputs = model.generate(inputs, max_new_tokens=100)
print(tokenizer.decode(outputs[0]))
```

## 5. Final Thoughts

Understanding the benefits and drawbacks of synthetic data, fake data, and data generated by machine learning 
models is crucial in diverse fields. Whether for privacy-preserving research, testing applications, or 
creative content generation, these techniques offer valuable solutions to data-related challenges, fostering 
innovation and problem-solving in various domains, but they need to be used with care.


To recap:

Synthetic data refers to artificially generated data designed to mimic real-world data patterns and statistical properties without containing actual observations.
- **Advantages:**
  - **Privacy Preservation:** Ideal for scenarios where real data must be kept confidential.
  - **Data Augmentation:** Useful for expanding datasets, especially in machine learning applications.
- **Techniques:**
  - **Statistical Modeling:** Generating data following statistical properties of real datasets.
  - **Generative Adversarial Networks (GANs):** Complex neural networks capable of generating highly realistic data samples.
- **Use Case:** Synthetic healthcare records for medical research without revealing sensitive patient information.

Fake data is entirely fabricated information without any basis in reality.
- **Characteristics:**
  - **Randomness:** Generated using algorithms or random processes.
  - **No Correlation:** Lacks any connection to real-world entities or events.
- **Applications:**
  - **Testing and Development:** Ideal for software testing scenarios where real data is unnecessary.
  - **Mock Scenarios:** Creating fictional scenarios for storytelling or simulations.
- **Example:** Dummy names, addresses, and phone numbers created for testing purposes.

Data generated by advanced machine learning models, like Generative Adversarial Networks (GANs) or Transformer-based models, trained on real datasets to generate new, realistic data samples.
- **Key Features:**
  - **Complexity:** Captures intricate patterns, context, and nuances from the training data.
  - **High Realism:** Produces data remarkably similar to real-world observations.
- **Use Cases:**
  - **Content Creation:** Generating realistic text, images, or multimedia content.
  - **Data Imputation:** Filling missing values in real datasets with plausible synthetic data.
- **Example:** Text passages generated by GPT-3 language models based on given prompts.

## 6. Exercise

Exercise 1 - Generate Fake Names

The mimesis Person() class can generate fake names. Import mimesis and initialize a person object:


```python
from mimesis import ____

person = ____()
```

Generate a random first name:


```python
first_name = person.____()
```

Generate a random last name:


```python
last_name = person.____() 
```

Combine them into a full name and print the result.


```python
full_name = first_name + " " + ______
print("Generated name:", ______)
```

Exercise 2 - Generate Fake Addresses

The address class generates fake addresses. Import and initialize it:


```python
from mimesis import _____

address = ______()
```

Generate a random street name: 


```python
street = ______.______()
```

Generate a city name:


```python
city = ______.______() 
```

Combine into a full address:


```python
full_address = ______ + ", " + ______ + ", " + address.______()
print("Generated address:") 
print(full_address)
```
