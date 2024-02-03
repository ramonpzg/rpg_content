##  Engineering Synthetic Data

> "Learning is synthesizing seemingly divergent ideas and data.” ~ Terry Heick


![dogs](../images/hr_dogs.png)


## Table of Contents


1. Overview
2. Tools
3. Quick Intro
3. Real Data
4. Fake Data
5. Synthetic Data
6. Use Cases
7. Final Thoughts
8. Exercises


## 1. Overview


Employee churn refers to the rate at which employees leave a company within a specific time period. To predict churn,
companies use data such as employee demographics, job satisfaction surveys, performance metrics, tenure, and historical
turnover rates. Analyzing this data helps identify patterns and factors contributing to employee departures, enabling
organizations to implement strategies to retain their staff.

Now that you know what churn is, imagine that the HR department of the company we've been building comes to you and says,
"a lot of people a churning and I think it would be a good to see if we can predict when people might leave the company
in feature. After all, we have over 50,000 employees and we want to make sure we not only make them all happy, but also
replace them with the same quality when they take off for their next adventure.

That said, I want to request budget for a dedicated People Analytics person but I can't go to the VP of People and ask for more
budget without at least something useful information. Remi from the data analytics team offered to help with analyzing the data,
but said that it is quite messy at the moment. Could you please help us clean it, and, if possible, automate such cleaning pipeline
for future ad-hoc analysis and machine learning use cases?

One caveat: We only have a tiny sample of the real data. Since what we need first is a Proof of Concept, would you be
able to make it better?

What we are going to do is to have a look at the sample provided to us, enhance it with some fake data, and create some
cleaning pipelines that we might be able to automate later on. Let's get started! 😎


## 2. Tools


There are three main tools we'll be using for this section.

- [mimesis](https://mimesis.name/en/master/index.html) --> "Mimesis is a powerful data generator for
  Python that can produce a wide range of fake data in multiple languages. This tool is useful for populating
  testing databases, creating fake API endpoints, generating custom structures in JSON and XML files, and
  anonymizing production data, among other things. With Mimesis, developers can obtain realistic, randomized
  data easily to facilitate development and testing.
- [pandas](https://pandas.pydata.org/pandas-docs/stable/index.html) --> "pandas is a Python package
  providing fast, flexible, and expressive data structures designed to make working with “relational” or
  “labeled” data both easy and intuitive. It aims to be the fundamental high-level building block for doing
  practical, real-world data analysis in Python."
- [SDV]() --> Synthetic Data Vault


## 3. Quick Intro

### Fake Data


```python
from mimesis import Person, Datetime, Text, Generic
from mimesis.locales import Locale
from mimesis.enums import Gender
```


```python
person = Person(Locale.EN)
person??
```

    [0;31mType:[0m           Person
    [0;31mString form:[0m    Person <Locale.EN>
    [0;31mFile:[0m           ~/mambaforge/envs/ml_svcs_p5/lib/python3.11/site-packages/mimesis/providers/person.py
    [0;31mSource:[0m        
    [0;32mclass[0m [0mPerson[0m[0;34m([0m[0mBaseDataProvider[0m[0;34m)[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m    [0;34m"""Class for generating personal data."""[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0m__init__[0m[0;34m([0m[0mself[0m[0;34m,[0m [0;34m*[0m[0margs[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mAny[0m[0;34m,[0m [0;34m**[0m[0mkwargs[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mAny[0m[0;34m)[0m [0;34m->[0m [0;32mNone[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Initialize attributes.[0m
    [0;34m[0m
    [0;34m        :param locale: Current locale.[0m
    [0;34m        :param seed: Seed.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0msuper[0m[0;34m([0m[0;34m)[0m[0;34m.[0m[0m__init__[0m[0;34m([0m[0;34m*[0m[0margs[0m[0;34m,[0m [0;34m**[0m[0mkwargs[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0mself[0m[0;34m.[0m[0m_store[0m [0;34m=[0m [0;34m{[0m[0;34m[0m
    [0;34m[0m            [0;34m"age"[0m[0;34m:[0m [0;36m0[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0;34m}[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mclass[0m [0mMeta[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0mname[0m [0;34m=[0m [0;34m"person"[0m[0;34m[0m
    [0;34m[0m        [0mdatafile[0m [0;34m=[0m [0;34mf"{name}.json"[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mage[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mminimum[0m[0;34m:[0m [0mint[0m [0;34m=[0m [0;36m16[0m[0;34m,[0m [0mmaximum[0m[0;34m:[0m [0mint[0m [0;34m=[0m [0;36m66[0m[0;34m)[0m [0;34m->[0m [0mint[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a random integer value.[0m
    [0;34m[0m
    [0;34m        :param maximum: Maximum value of age.[0m
    [0;34m        :param minimum: Minimum value of age.[0m
    [0;34m        :return: Random integer.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            23.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mage[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mrandint[0m[0;34m([0m[0mminimum[0m[0;34m,[0m [0mmaximum[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0mself[0m[0;34m.[0m[0m_store[0m[0;34m[[0m[0;34m"age"[0m[0;34m][0m [0;34m=[0m [0mage[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mage[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mwork_experience[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mworking_start_age[0m[0;34m:[0m [0mint[0m [0;34m=[0m [0;36m22[0m[0;34m)[0m [0;34m->[0m [0mint[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a work experience.[0m
    [0;34m[0m
    [0;34m        :param working_start_age: Age then person start to work.[0m
    [0;34m        :return: Depend on previous generated age.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mage[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0m_store[0m[0;34m[[0m[0;34m"age"[0m[0;34m][0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0mage[0m [0;34m==[0m [0;36m0[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0mage[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mage[0m[0;34m([0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mmax[0m[0;34m([0m[0mage[0m [0;34m-[0m [0mworking_start_age[0m[0;34m,[0m [0;36m0[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mname[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mgender[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mOptional[0m[0;34m[[0m[0mGender[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a random name.[0m
    [0;34m[0m
    [0;34m        :param gender: Gender's enum object.[0m
    [0;34m        :return: Name.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            John.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mkey[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mvalidate_enum[0m[0;34m([0m[0mgender[0m[0;34m,[0m [0mGender[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0mnames[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"names"[0m[0;34m,[0m [0mkey[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mnames[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mfirst_name[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mgender[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mOptional[0m[0;34m[[0m[0mGender[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a random first name.[0m
    [0;34m[0m
    [0;34m        ..note: An alias for self.name().[0m
    [0;34m[0m
    [0;34m        :param gender: Gender's enum object.[0m
    [0;34m        :return: First name.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mname[0m[0;34m([0m[0mgender[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0msurname[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mgender[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mOptional[0m[0;34m[[0m[0mGender[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a random surname.[0m
    [0;34m[0m
    [0;34m        :param gender: Gender's enum object.[0m
    [0;34m        :return: Surname.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            Smith.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0msurnames[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mSequence[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"surnames"[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;31m# Surnames separated by gender.[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0misinstance[0m[0;34m([0m[0msurnames[0m[0;34m,[0m [0mdict[0m[0;34m)[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0mkey[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mvalidate_enum[0m[0;34m([0m[0mgender[0m[0;34m,[0m [0mGender[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m            [0msurnames[0m [0;34m=[0m [0msurnames[0m[0;34m[[0m[0mkey[0m[0;34m][0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0msurnames[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mlast_name[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mgender[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mOptional[0m[0;34m[[0m[0mGender[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a random last name.[0m
    [0;34m[0m
    [0;34m        ..note: An alias for self.surname().[0m
    [0;34m[0m
    [0;34m        :param gender: Gender's enum object.[0m
    [0;34m        :return: Last name.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0msurname[0m[0;34m([0m[0mgender[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mtitle[0m[0;34m([0m[0;34m[0m
    [0;34m[0m        [0mself[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0mgender[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mOptional[0m[0;34m[[0m[0mGender[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0mtitle_type[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mOptional[0m[0;34m[[0m[0mTitleType[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a random title for name.[0m
    [0;34m[0m
    [0;34m        You can generate random prefix or suffix[0m
    [0;34m        for name using this method.[0m
    [0;34m[0m
    [0;34m        :param gender: The gender.[0m
    [0;34m        :param title_type: TitleType enum object.[0m
    [0;34m        :return: The title.[0m
    [0;34m        :raises NonEnumerableError: if gender or title_type in incorrect format.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            PhD.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mgender_key[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mvalidate_enum[0m[0;34m([0m[0mgender[0m[0;34m,[0m [0mGender[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0mtitle_key[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mvalidate_enum[0m[0;34m([0m[0mtitle_type[0m[0;34m,[0m [0mTitleType[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0mtitles[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"title"[0m[0;34m,[0m [0mgender_key[0m[0;34m,[0m [0mtitle_key[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mtitles[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mfull_name[0m[0;34m([0m[0;34m[0m
    [0;34m[0m        [0mself[0m[0;34m,[0m [0mgender[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mOptional[0m[0;34m[[0m[0mGender[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m [0mreverse[0m[0;34m:[0m [0mbool[0m [0;34m=[0m [0;32mFalse[0m[0;34m[0m
    [0;34m[0m    [0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a random full name.[0m
    [0;34m[0m
    [0;34m        :param reverse: Return reversed full name.[0m
    [0;34m        :param gender: Gender's enum object.[0m
    [0;34m        :return: Full name.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            Johann Wolfgang.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mname[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mname[0m[0;34m([0m[0mgender[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0msurname[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0msurname[0m[0;34m([0m[0mgender[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0;34mf"{surname} {name}"[0m [0;32mif[0m [0mreverse[0m [0;32melse[0m [0;34mf"{name} {surname}"[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0musername[0m[0;34m([0m[0;34m[0m
    [0;34m[0m        [0mself[0m[0;34m,[0m [0mmask[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mOptional[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m [0mdrange[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mTuple[0m[0;34m[[0m[0mint[0m[0;34m,[0m [0mint[0m[0;34m][0m [0;34m=[0m [0;34m([0m[0;36m1800[0m[0;34m,[0m [0;36m2100[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m    [0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate username by mask.[0m
    [0;34m[0m
    [0;34m        Masks allow you to generate a variety of usernames.[0m
    [0;34m[0m
    [0;34m        - **C** stands for capitalized username.[0m
    [0;34m        - **U** stands for uppercase username.[0m
    [0;34m        - **l** stands for lowercase username.[0m
    [0;34m        - **d** stands for digits in the username.[0m
    [0;34m[0m
    [0;34m        You can also use symbols to separate the different parts[0m
    [0;34m        of the username: **.** **_** **-**[0m
    [0;34m[0m
    [0;34m        :param mask: Mask.[0m
    [0;34m        :param drange: Digits range.[0m
    [0;34m        :raises ValueError: If template is not supported.[0m
    [0;34m        :return: Username as string.[0m
    [0;34m[0m
    [0;34m        Example:[0m
    [0;34m            >>> username(mask='C_C_d')[0m
    [0;34m            Cotte_Article_1923[0m
    [0;34m            >>> username(mask='U.l.d')[0m
    [0;34m            ELKINS.wolverine.2013[0m
    [0;34m            >>> username(mask='l_l_d', drange=(1900, 2021))[0m
    [0;34m            plasmic_blockader_1907[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0mlen[0m[0;34m([0m[0mdrange[0m[0;34m)[0m [0;34m!=[0m [0;36m2[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0;32mraise[0m [0mValueError[0m[0;34m([0m[0;34m"The drange parameter must contain only two integers."[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0mmask[0m [0;32mis[0m [0;32mNone[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0mmask[0m [0;34m=[0m [0;34m"l_d"[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0mrequired_tags[0m [0;34m=[0m [0;34m"CUl"[0m[0;34m[0m
    [0;34m[0m        [0mtags[0m [0;34m=[0m [0mre[0m[0;34m.[0m[0mfindall[0m[0;34m([0m[0;34mr"[CUld.\-_]"[0m[0;34m,[0m [0mmask[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0;32mnot[0m [0many[0m[0;34m([0m[0mtag[0m [0;32min[0m [0mtags[0m [0;32mfor[0m [0mtag[0m [0;32min[0m [0mrequired_tags[0m[0;34m)[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0;32mraise[0m [0mValueError[0m[0;34m([0m[0;34m[0m
    [0;34m[0m                [0;34m"Username mask must contain at least one of these: (C, U, l)."[0m[0;34m[0m
    [0;34m[0m            [0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0mfinal_username[0m [0;34m=[0m [0;34m""[0m[0;34m[0m
    [0;34m[0m        [0;32mfor[0m [0mtag[0m [0;32min[0m [0mtags[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0musername[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mUSERNAMES[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m            [0;32mif[0m [0mtag[0m [0;34m==[0m [0;34m"C"[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m                [0mfinal_username[0m [0;34m+=[0m [0musername[0m[0;34m.[0m[0mcapitalize[0m[0;34m([0m[0;34m)[0m[0;34m[0m
    [0;34m[0m            [0;32mif[0m [0mtag[0m [0;34m==[0m [0;34m"U"[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m                [0mfinal_username[0m [0;34m+=[0m [0musername[0m[0;34m.[0m[0mupper[0m[0;34m([0m[0;34m)[0m[0;34m[0m
    [0;34m[0m            [0;32melif[0m [0mtag[0m [0;34m==[0m [0;34m"l"[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m                [0mfinal_username[0m [0;34m+=[0m [0musername[0m[0;34m.[0m[0mlower[0m[0;34m([0m[0;34m)[0m[0;34m[0m
    [0;34m[0m            [0;32melif[0m [0mtag[0m [0;34m==[0m [0;34m"d"[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m                [0mfinal_username[0m [0;34m+=[0m [0mstr[0m[0;34m([0m[0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mrandint[0m[0;34m([0m[0;34m*[0m[0mdrange[0m[0;34m)[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m            [0;32melif[0m [0mtag[0m [0;32min[0m [0;34m"-_."[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m                [0mfinal_username[0m [0;34m+=[0m [0mtag[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mfinal_username[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mpassword[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mlength[0m[0;34m:[0m [0mint[0m [0;34m=[0m [0;36m8[0m[0;34m,[0m [0mhashed[0m[0;34m:[0m [0mbool[0m [0;34m=[0m [0;32mFalse[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a password or hash of password.[0m
    [0;34m[0m
    [0;34m        :param length: Length of password.[0m
    [0;34m        :param hashed: SHA256 hash.[0m
    [0;34m        :return: Password or hash of password.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            k6dv2odff9#4h[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mcharacters[0m [0;34m=[0m [0mascii_letters[0m [0;34m+[0m [0mdigits[0m [0;34m+[0m [0mpunctuation[0m[0;34m[0m
    [0;34m[0m        [0mpassword[0m [0;34m=[0m [0;34m""[0m[0;34m.[0m[0mjoin[0m[0;34m([0m[0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoices[0m[0;34m([0m[0mcharacters[0m[0;34m,[0m [0mk[0m[0;34m=[0m[0mlength[0m[0;34m)[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0mhashed[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0msha256[0m [0;34m=[0m [0mhashlib[0m[0;34m.[0m[0msha256[0m[0;34m([0m[0;34m)[0m[0;34m[0m
    [0;34m[0m            [0msha256[0m[0;34m.[0m[0mupdate[0m[0;34m([0m[0mpassword[0m[0;34m.[0m[0mencode[0m[0;34m([0m[0;34m)[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m            [0;32mreturn[0m [0msha256[0m[0;34m.[0m[0mhexdigest[0m[0;34m([0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mpassword[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0memail[0m[0;34m([0m[0;34m[0m
    [0;34m[0m        [0mself[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0mdomains[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mOptional[0m[0;34m[[0m[0mt[0m[0;34m.[0m[0mSequence[0m[0;34m[[0m[0mstr[0m[0;34m][0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0munique[0m[0;34m:[0m [0mbool[0m [0;34m=[0m [0;32mFalse[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a random email.[0m
    [0;34m[0m
    [0;34m        :param domains: List of custom domains for emails.[0m
    [0;34m        :param unique: Makes email addresses unique.[0m
    [0;34m        :return: Email address.[0m
    [0;34m        :raises ValueError: if «unique» is True and the provider was seeded.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            foretime10@live.com[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0munique[0m [0;32mand[0m [0mself[0m[0;34m.[0m[0m_has_seed[0m[0;34m([0m[0;34m)[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0;32mraise[0m [0mValueError[0m[0;34m([0m[0;34m[0m
    [0;34m[0m                [0;34m"You cannot use «unique» parameter with the seeded provider"[0m[0;34m[0m
    [0;34m[0m            [0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0;32mnot[0m [0mdomains[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0mdomains[0m [0;34m=[0m [0mEMAIL_DOMAINS[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0mdomain[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mdomains[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0;32mnot[0m [0mdomain[0m[0;34m.[0m[0mstartswith[0m[0;34m([0m[0;34m"@"[0m[0;34m)[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0mdomain[0m [0;34m=[0m [0;34m"@"[0m [0;34m+[0m [0mdomain[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0munique[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0mname[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0m_randstr[0m[0;34m([0m[0munique[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32melse[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0mname[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0musername[0m[0;34m([0m[0mmask[0m[0;34m=[0m[0;34m"ld"[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0;34mf"{name}{domain}"[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mgender[0m[0;34m([0m[0mself[0m[0;34m,[0m [0miso5218[0m[0;34m:[0m [0mbool[0m [0;34m=[0m [0;32mFalse[0m[0;34m,[0m [0msymbol[0m[0;34m:[0m [0mbool[0m [0;34m=[0m [0;32mFalse[0m[0;34m)[0m [0;34m->[0m [0mt[0m[0;34m.[0m[0mUnion[0m[0;34m[[0m[0mstr[0m[0;34m,[0m [0mint[0m[0;34m][0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a random gender.[0m
    [0;34m[0m
    [0;34m        Get a random title of gender, code for the representation[0m
    [0;34m        of human sexes is an international standard that defines a[0m
    [0;34m        representation of human sexes through a language-neutral single-digit[0m
    [0;34m        code or symbol of gender.[0m
    [0;34m[0m
    [0;34m        :param iso5218:[0m
    [0;34m            Codes for the representation of human sexes is an international[0m
    [0;34m            standard (0 - not known, 1 - male, 2 - female, 9 - not applicable).[0m
    [0;34m        :param symbol: Symbol of gender.[0m
    [0;34m        :return: Title of gender.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            Male[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0miso5218[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0;34m[[0m[0;36m0[0m[0;34m,[0m [0;36m1[0m[0;34m,[0m [0;36m2[0m[0;34m,[0m [0;36m9[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0msymbol[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mGENDER_SYMBOLS[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0mgenders[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"gender"[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mgenders[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0msex[0m[0;34m([0m[0mself[0m[0;34m,[0m [0;34m*[0m[0margs[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mAny[0m[0;34m,[0m [0;34m**[0m[0mkwargs[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mAny[0m[0;34m)[0m [0;34m->[0m [0mt[0m[0;34m.[0m[0mUnion[0m[0;34m[[0m[0mstr[0m[0;34m,[0m [0mint[0m[0;34m][0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""An alias for method self.gender().[0m
    [0;34m[0m
    [0;34m        See docstrings of method self.gender() for details.[0m
    [0;34m[0m
    [0;34m        :param args: Positional arguments.[0m
    [0;34m        :param kwargs: Keyword arguments.[0m
    [0;34m        :return: Sex[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mgender[0m[0;34m([0m[0;34m*[0m[0margs[0m[0;34m,[0m [0;34m**[0m[0mkwargs[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mheight[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mminimum[0m[0;34m:[0m [0mfloat[0m [0;34m=[0m [0;36m1.5[0m[0;34m,[0m [0mmaximum[0m[0;34m:[0m [0mfloat[0m [0;34m=[0m [0;36m2.0[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a random height in meters.[0m
    [0;34m[0m
    [0;34m        :param minimum: Minimum value.[0m
    [0;34m        :param float maximum: Maximum value.[0m
    [0;34m        :return: Height.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            1.85.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mh[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0muniform[0m[0;34m([0m[0mminimum[0m[0;34m,[0m [0mmaximum[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0;34mf"{h:0.2f}"[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mweight[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mminimum[0m[0;34m:[0m [0mint[0m [0;34m=[0m [0;36m38[0m[0;34m,[0m [0mmaximum[0m[0;34m:[0m [0mint[0m [0;34m=[0m [0;36m90[0m[0;34m)[0m [0;34m->[0m [0mint[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a random weight in Kg.[0m
    [0;34m[0m
    [0;34m        :param minimum: min value[0m
    [0;34m        :param maximum: max value[0m
    [0;34m        :return: Weight.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            48.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mrandint[0m[0;34m([0m[0mminimum[0m[0;34m,[0m [0mmaximum[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mblood_type[0m[0;34m([0m[0mself[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a random blood type.[0m
    [0;34m[0m
    [0;34m        :return: Blood type (blood group).[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            A+[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mBLOOD_GROUPS[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0moccupation[0m[0;34m([0m[0mself[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a random job.[0m
    [0;34m[0m
    [0;34m        :return: The name of job.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            Programmer.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mjobs[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"occupation"[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mjobs[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mpolitical_views[0m[0;34m([0m[0mself[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a random political views.[0m
    [0;34m[0m
    [0;34m        :return: Political views.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            Liberal.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mviews[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"political_views"[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mviews[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mworldview[0m[0;34m([0m[0mself[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a random worldview.[0m
    [0;34m[0m
    [0;34m        :return: Worldview.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            Pantheism.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mviews[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"worldview"[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mviews[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mviews_on[0m[0;34m([0m[0mself[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a random views on.[0m
    [0;34m[0m
    [0;34m        :return: Views on.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            Negative.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mviews[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"views_on"[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mviews[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mnationality[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mgender[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mOptional[0m[0;34m[[0m[0mGender[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a random nationality.[0m
    [0;34m[0m
    [0;34m        :param gender: Gender.[0m
    [0;34m        :return: Nationality.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            Russian[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mnationalities[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"nationality"[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;31m# Separated by gender[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0misinstance[0m[0;34m([0m[0mnationalities[0m[0;34m,[0m [0mdict[0m[0;34m)[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0mkey[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mvalidate_enum[0m[0;34m([0m[0mgender[0m[0;34m,[0m [0mGender[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m            [0mnationalities[0m [0;34m=[0m [0mnationalities[0m[0;34m[[0m[0mkey[0m[0;34m][0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mnationalities[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0muniversity[0m[0;34m([0m[0mself[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a random university.[0m
    [0;34m[0m
    [0;34m        :return: University name.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            MIT.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0muniversities[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"university"[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0muniversities[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0macademic_degree[0m[0;34m([0m[0mself[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a random academic degree.[0m
    [0;34m[0m
    [0;34m        :return: Degree.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            Bachelor.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mdegrees[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"academic_degree"[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mdegrees[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mlanguage[0m[0;34m([0m[0mself[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Get a random language.[0m
    [0;34m[0m
    [0;34m        :return: Random language.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            Irish.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mlanguages[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"language"[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mlanguages[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mphone_number[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mmask[0m[0;34m:[0m [0mstr[0m [0;34m=[0m [0;34m""[0m[0;34m,[0m [0mplaceholder[0m[0;34m:[0m [0mstr[0m [0;34m=[0m [0;34m"#"[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a random phone number.[0m
    [0;34m[0m
    [0;34m        :param mask: Mask for formatting number.[0m
    [0;34m        :param placeholder: A placeholder for a mask (default is #).[0m
    [0;34m        :return: Phone number.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            +7-(963)-409-11-22.[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0;32mif[0m [0;32mnot[0m [0mmask[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m            [0mcode[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mCALLING_CODES[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m            [0mdefault[0m [0;34m=[0m [0;34mf"{code}-(###)-###-####"[0m[0;34m[0m
    [0;34m[0m            [0mmasks[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mextract[0m[0;34m([0m[0;34m[[0m[0;34m"telephone_fmt"[0m[0;34m][0m[0;34m,[0m [0mdefault[0m[0;34m=[0m[0;34m[[0m[0mdefault[0m[0;34m][0m[0;34m)[0m[0;34m[0m
    [0;34m[0m            [0mmask[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mchoice[0m[0;34m([0m[0mmasks[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mcustom_code[0m[0;34m([0m[0mmask[0m[0;34m=[0m[0mmask[0m[0;34m,[0m [0mdigit[0m[0;34m=[0m[0mplaceholder[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mtelephone[0m[0;34m([0m[0mself[0m[0;34m,[0m [0;34m*[0m[0margs[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mAny[0m[0;34m,[0m [0;34m**[0m[0mkwargs[0m[0;34m:[0m [0mt[0m[0;34m.[0m[0mAny[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mphone_number[0m[0;34m([0m[0;34m*[0m[0margs[0m[0;34m,[0m [0;34m**[0m[0mkwargs[0m[0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0midentifier[0m[0;34m([0m[0mself[0m[0;34m,[0m [0mmask[0m[0;34m:[0m [0mstr[0m [0;34m=[0m [0;34m"##-##/##"[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""Generate a random identifier by mask.[0m
    [0;34m[0m
    [0;34m        With this method you can generate any identifiers that[0m
    [0;34m        you need. Simply select the mask that you need.[0m
    [0;34m[0m
    [0;34m        :param mask:[0m
    [0;34m            The mask. Here ``@`` is a placeholder for characters and ``#`` is[0m
    [0;34m            placeholder for digits.[0m
    [0;34m        :return: An identifier.[0m
    [0;34m[0m
    [0;34m        :Example:[0m
    [0;34m            07-97/04[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mself[0m[0;34m.[0m[0mrandom[0m[0;34m.[0m[0mcustom_code[0m[0;34m([0m[0mmask[0m[0;34m=[0m[0mmask[0m[0;34m)[0m[0;34m[0m[0;34m[0m[0m
    [0;31mInit docstring:[0m
    Initialize attributes.
    
    :param locale: Current locale.
    :param seed: Seed.


```python
person.full_name(gender=Gender.FEMALE)
```




    'Johna Fox'




```python
person.full_name(gender=Gender.MALE)
```




    'Stephen Cortez'




```python
person = Person(Locale.UK)
datetime = Datetime(Locale.UK)
text = Text(Locale.UK)

print(f"Person: {person.full_name()}; Date-Time: {datetime.date()}; Text: {text.quote()};")
```

    Person: Юхимія Белонович; Date-Time: 2019-03-28; Text: Той, хто зневажливо ставиться до рідної мови, не може й сам викликати поваги до себе;



```python
generic = Generic(locale=Locale.EN)
generic.person.username()
```




    'maintained_1985'




```python
generic.datetime.date()
```




    datetime.date(2022, 11, 5)




```python
p_en = Person(Locale.EN)
p_sv = Person(Locale.SV)
p_en.full_name(), p_sv.full_name()
```




    ('Elden Brooks', 'Ærnfridh Bäck')




```python
g = Generic(locale=Locale.ES)
g.datetime.month(), g.code.imei(), g.food.fruit()
```




    ('Agosto', '358755056763888', 'Piña')




```python
from mimesis import Fieldset
```


```python
fs = Fieldset(i=100)
```


```python
fs('random.randint', a=1, b=100)
```




    [84,
     19,
     80,
     8,
     45,
     10,
     26,
     13,
     64,
     3,
     10,
     15,
     96,
     10,
     70,
     99,
     8,
     38,
     91,
     73,
     78,
     90,
     89,
     59,
     86,
     8,
     24,
     4,
     73,
     61,
     5,
     69,
     10,
     45,
     17,
     44,
     55,
     15,
     62,
     3,
     57,
     92,
     96,
     92,
     71,
     60,
     41,
     18,
     11,
     89,
     31,
     5,
     8,
     58,
     47,
     42,
     72,
     33,
     52,
     2,
     92,
     82,
     74,
     77,
     75,
     66,
     80,
     55,
     96,
     83,
     33,
     5,
     25,
     26,
     62,
     4,
     67,
     66,
     75,
     90,
     3,
     21,
     24,
     1,
     12,
     42,
     86,
     39,
     59,
     25,
     94,
     17,
     28,
     42,
     15,
     79,
     88,
     89,
     34,
     71]




```python
fs('email', key=lambda x: x.split('@')[0].replace('xxxx')
```

### Synthetic Data


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




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>guest_email</th>
      <th>has_rewards</th>
      <th>room_type</th>
      <th>amenities_fee</th>
      <th>checkin_date</th>
      <th>checkout_date</th>
      <th>room_rate</th>
      <th>billing_address</th>
      <th>credit_card_number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>michaelsanders@shaw.net</td>
      <td>False</td>
      <td>BASIC</td>
      <td>37.89</td>
      <td>27 Dec 2020</td>
      <td>29 Dec 2020</td>
      <td>131.23</td>
      <td>49380 Rivers Street\nSpencerville, AK 68265</td>
      <td>4075084747483975747</td>
    </tr>
    <tr>
      <th>1</th>
      <td>randy49@brown.biz</td>
      <td>False</td>
      <td>BASIC</td>
      <td>24.37</td>
      <td>30 Dec 2020</td>
      <td>02 Jan 2021</td>
      <td>114.43</td>
      <td>88394 Boyle Meadows\nConleyberg, TN 22063</td>
      <td>180072822063468</td>
    </tr>
    <tr>
      <th>2</th>
      <td>webermelissa@neal.com</td>
      <td>True</td>
      <td>DELUXE</td>
      <td>0.00</td>
      <td>17 Sep 2020</td>
      <td>18 Sep 2020</td>
      <td>368.33</td>
      <td>0323 Lisa Station Apt. 208\nPort Thomas, LA 82585</td>
      <td>38983476971380</td>
    </tr>
    <tr>
      <th>3</th>
      <td>gsims@terry.com</td>
      <td>False</td>
      <td>BASIC</td>
      <td>NaN</td>
      <td>28 Dec 2020</td>
      <td>31 Dec 2020</td>
      <td>115.61</td>
      <td>77 Massachusetts Ave\nCambridge, MA 02139</td>
      <td>4969551998845740</td>
    </tr>
    <tr>
      <th>4</th>
      <td>misty33@smith.biz</td>
      <td>False</td>
      <td>BASIC</td>
      <td>16.45</td>
      <td>05 Apr 2020</td>
      <td>NaN</td>
      <td>122.41</td>
      <td>1234 Corporate Drive\nBoston, MA 02116</td>
      <td>3558512986488983</td>
    </tr>
  </tbody>
</table>
</div>




```python
metadata.visualize()
```




    
![svg](10_intro_files/10_intro_28_0.svg)
    




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




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>guest_email</th>
      <th>has_rewards</th>
      <th>room_type</th>
      <th>amenities_fee</th>
      <th>checkin_date</th>
      <th>checkout_date</th>
      <th>room_rate</th>
      <th>billing_address</th>
      <th>credit_card_number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>dsullivan@example.net</td>
      <td>False</td>
      <td>BASIC</td>
      <td>9.752814</td>
      <td>31 Mar 2020</td>
      <td>19 Apr 2020</td>
      <td>151.871782</td>
      <td>90469 Karla Knolls Apt. 781\nSusanberg, NC 28401</td>
      <td>5161033759518983</td>
    </tr>
    <tr>
      <th>1</th>
      <td>steven59@example.org</td>
      <td>False</td>
      <td>BASIC</td>
      <td>NaN</td>
      <td>25 Jun 2020</td>
      <td>14 Aug 2020</td>
      <td>181.481346</td>
      <td>1080 Ashley Creek Apt. 622\nWest Amy, NM 25058</td>
      <td>4133047413145475690</td>
    </tr>
    <tr>
      <th>2</th>
      <td>brandon15@example.net</td>
      <td>False</td>
      <td>BASIC</td>
      <td>22.554106</td>
      <td>14 Apr 2020</td>
      <td>07 Apr 2020</td>
      <td>143.655477</td>
      <td>99923 Anderson Trace Suite 861\nNorth Haley, T...</td>
      <td>4977328103788</td>
    </tr>
    <tr>
      <th>3</th>
      <td>humphreyjennifer@example.net</td>
      <td>False</td>
      <td>BASIC</td>
      <td>24.324111</td>
      <td>25 May 2020</td>
      <td>11 Jun 2020</td>
      <td>178.353772</td>
      <td>9301 John Parkways\nThomasland, OH 61350</td>
      <td>3524946844839485</td>
    </tr>
    <tr>
      <th>4</th>
      <td>joshuabrown@example.net</td>
      <td>False</td>
      <td>BASIC</td>
      <td>20.523800</td>
      <td>13 Nov 2019</td>
      <td>25 Oct 2019</td>
      <td>183.572274</td>
      <td>126 George Tunnel\nDuranstad, MS 95176</td>
      <td>4446905799576890978</td>
    </tr>
  </tbody>
</table>
</div>




```python
sensitive_column_names = ['guest_email', 'billing_address', 'credit_card_number']
real_data[sensitive_column_names].head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>guest_email</th>
      <th>billing_address</th>
      <th>credit_card_number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>michaelsanders@shaw.net</td>
      <td>49380 Rivers Street\nSpencerville, AK 68265</td>
      <td>4075084747483975747</td>
    </tr>
    <tr>
      <th>1</th>
      <td>randy49@brown.biz</td>
      <td>88394 Boyle Meadows\nConleyberg, TN 22063</td>
      <td>180072822063468</td>
    </tr>
    <tr>
      <th>2</th>
      <td>webermelissa@neal.com</td>
      <td>0323 Lisa Station Apt. 208\nPort Thomas, LA 82585</td>
      <td>38983476971380</td>
    </tr>
  </tbody>
</table>
</div>




```python
synthetic_data[sensitive_column_names].head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>guest_email</th>
      <th>billing_address</th>
      <th>credit_card_number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>dsullivan@example.net</td>
      <td>90469 Karla Knolls Apt. 781\nSusanberg, NC 28401</td>
      <td>5161033759518983</td>
    </tr>
    <tr>
      <th>1</th>
      <td>steven59@example.org</td>
      <td>1080 Ashley Creek Apt. 622\nWest Amy, NM 25058</td>
      <td>4133047413145475690</td>
    </tr>
    <tr>
      <th>2</th>
      <td>brandon15@example.net</td>
      <td>99923 Anderson Trace Suite 861\nNorth Haley, T...</td>
      <td>4977328103788</td>
    </tr>
  </tbody>
</table>
</div>




```python
quality_report = evaluate_quality(real_data, synthetic_data, metadata)
```

    Generating report ...
    (1/2) Evaluating Column Shapes: : 100%|██████████| 9/9 [00:00<00:00, 657.91it/s]
    (2/2) Evaluating Column Pair Trends: : 100%|██████████| 36/36 [00:00<00:00, 199.28it/s]
    
    Overall Quality Score: 86.57%
    
    Properties:
    - Column Shapes: 91.67%
    - Column Pair Trends: 81.47%



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




    SingleTablePreset(name=FAST_ML)



### ML-Generated Data


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

    [0;31mSignature:[0m
    [0mtokenizer[0m[0;34m.[0m[0mencode[0m[0;34m([0m[0;34m[0m
    [0;34m[0m    [0mtext[0m[0;34m:[0m [0mUnion[0m[0;34m[[0m[0mstr[0m[0;34m,[0m [0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m[0;34m,[0m [0mList[0m[0;34m[[0m[0mint[0m[0;34m][0m[0;34m][0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0mtext_pair[0m[0;34m:[0m [0mUnion[0m[0;34m[[0m[0mstr[0m[0;34m,[0m [0mList[0m[0;34m[[0m[0mstr[0m[0;34m][0m[0;34m,[0m [0mList[0m[0;34m[[0m[0mint[0m[0;34m][0m[0;34m,[0m [0mNoneType[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0madd_special_tokens[0m[0;34m:[0m [0mbool[0m [0;34m=[0m [0;32mTrue[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0mpadding[0m[0;34m:[0m [0mUnion[0m[0;34m[[0m[0mbool[0m[0;34m,[0m [0mstr[0m[0;34m,[0m [0mtransformers[0m[0;34m.[0m[0mutils[0m[0;34m.[0m[0mgeneric[0m[0;34m.[0m[0mPaddingStrategy[0m[0;34m][0m [0;34m=[0m [0;32mFalse[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0mtruncation[0m[0;34m:[0m [0mUnion[0m[0;34m[[0m[0mbool[0m[0;34m,[0m [0mstr[0m[0;34m,[0m [0mtransformers[0m[0;34m.[0m[0mtokenization_utils_base[0m[0;34m.[0m[0mTruncationStrategy[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0mmax_length[0m[0;34m:[0m [0mOptional[0m[0;34m[[0m[0mint[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0mstride[0m[0;34m:[0m [0mint[0m [0;34m=[0m [0;36m0[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0mreturn_tensors[0m[0;34m:[0m [0mUnion[0m[0;34m[[0m[0mstr[0m[0;34m,[0m [0mtransformers[0m[0;34m.[0m[0mutils[0m[0;34m.[0m[0mgeneric[0m[0;34m.[0m[0mTensorType[0m[0;34m,[0m [0mNoneType[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0;34m**[0m[0mkwargs[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m[0;34m)[0m [0;34m->[0m [0mList[0m[0;34m[[0m[0mint[0m[0;34m][0m[0;34m[0m[0;34m[0m[0m
    [0;31mDocstring:[0m
    Converts a string to a sequence of ids (integer), using the tokenizer and vocabulary.
    
    Same as doing `self.convert_tokens_to_ids(self.tokenize(text))`.
    
    Args:
        text (`str`, `List[str]` or `List[int]`):
            The first sequence to be encoded. This can be a string, a list of strings (tokenized string using the
            `tokenize` method) or a list of integers (tokenized string ids using the `convert_tokens_to_ids`
            method).
        text_pair (`str`, `List[str]` or `List[int]`, *optional*):
            Optional second sequence to be encoded. This can be a string, a list of strings (tokenized string using
            the `tokenize` method) or a list of integers (tokenized string ids using the `convert_tokens_to_ids`
            method).
    
        add_special_tokens (`bool`, *optional*, defaults to `True`):
            Whether or not to add special tokens when encoding the sequences. This will use the underlying
            `PretrainedTokenizerBase.build_inputs_with_special_tokens` function, which defines which tokens are
            automatically added to the input ids. This is usefull if you want to add `bos` or `eos` tokens
            automatically.
        padding (`bool`, `str` or [`~utils.PaddingStrategy`], *optional*, defaults to `False`):
            Activates and controls padding. Accepts the following values:
    
            - `True` or `'longest'`: Pad to the longest sequence in the batch (or no padding if only a single
              sequence if provided).
            - `'max_length'`: Pad to a maximum length specified with the argument `max_length` or to the maximum
              acceptable input length for the model if that argument is not provided.
            - `False` or `'do_not_pad'` (default): No padding (i.e., can output a batch with sequences of different
              lengths).
        truncation (`bool`, `str` or [`~tokenization_utils_base.TruncationStrategy`], *optional*, defaults to `False`):
            Activates and controls truncation. Accepts the following values:
    
            - `True` or `'longest_first'`: Truncate to a maximum length specified with the argument `max_length` or
              to the maximum acceptable input length for the model if that argument is not provided. This will
              truncate token by token, removing a token from the longest sequence in the pair if a pair of
              sequences (or a batch of pairs) is provided.
            - `'only_first'`: Truncate to a maximum length specified with the argument `max_length` or to the
              maximum acceptable input length for the model if that argument is not provided. This will only
              truncate the first sequence of a pair if a pair of sequences (or a batch of pairs) is provided.
            - `'only_second'`: Truncate to a maximum length specified with the argument `max_length` or to the
              maximum acceptable input length for the model if that argument is not provided. This will only
              truncate the second sequence of a pair if a pair of sequences (or a batch of pairs) is provided.
            - `False` or `'do_not_truncate'` (default): No truncation (i.e., can output batch with sequence lengths
              greater than the model maximum admissible input size).
        max_length (`int`, *optional*):
            Controls the maximum length to use by one of the truncation/padding parameters.
    
            If left unset or set to `None`, this will use the predefined model maximum length if a maximum length
            is required by one of the truncation/padding parameters. If the model has no specific maximum input
            length (like XLNet) truncation/padding to a maximum length will be deactivated.
        stride (`int`, *optional*, defaults to 0):
            If set to a number along with `max_length`, the overflowing tokens returned when
            `return_overflowing_tokens=True` will contain some tokens from the end of the truncated sequence
            returned to provide some overlap between truncated and overflowing sequences. The value of this
            argument defines the number of overlapping tokens.
        is_split_into_words (`bool`, *optional*, defaults to `False`):
            Whether or not the input is already pre-tokenized (e.g., split into words). If set to `True`, the
            tokenizer assumes the input is already split into words (for instance, by splitting it on whitespace)
            which it will tokenize. This is useful for NER or token classification.
        pad_to_multiple_of (`int`, *optional*):
            If set will pad the sequence to a multiple of the provided value. Requires `padding` to be activated.
            This is especially useful to enable the use of Tensor Cores on NVIDIA hardware with compute capability
            `>= 7.5` (Volta).
        return_tensors (`str` or [`~utils.TensorType`], *optional*):
            If set, will return tensors instead of list of python integers. Acceptable values are:
    
            - `'tf'`: Return TensorFlow `tf.constant` objects.
            - `'pt'`: Return PyTorch `torch.Tensor` objects.
            - `'np'`: Return Numpy `np.ndarray` objects.
    
        **kwargs: Passed along to the `.tokenize()` method.
    
    Returns:
        `List[int]`, `torch.Tensor`, `tf.Tensor` or `np.ndarray`: The tokenized ids of the text.
    [0;31mSource:[0m   
        [0;34m@[0m[0madd_end_docstrings[0m[0;34m([0m[0;34m[0m
    [0;34m[0m        [0mENCODE_KWARGS_DOCSTRING[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0;34m"""[0m
    [0;34m            **kwargs: Passed along to the `.tokenize()` method.[0m
    [0;34m        """[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0;34m"""[0m
    [0;34m        Returns:[0m
    [0;34m            `List[int]`, `torch.Tensor`, `tf.Tensor` or `np.ndarray`: The tokenized ids of the text.[0m
    [0;34m        """[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0;34m)[0m[0;34m[0m
    [0;34m[0m    [0;32mdef[0m [0mencode[0m[0;34m([0m[0;34m[0m
    [0;34m[0m        [0mself[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0mtext[0m[0;34m:[0m [0mUnion[0m[0;34m[[0m[0mTextInput[0m[0;34m,[0m [0mPreTokenizedInput[0m[0;34m,[0m [0mEncodedInput[0m[0;34m][0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0mtext_pair[0m[0;34m:[0m [0mOptional[0m[0;34m[[0m[0mUnion[0m[0;34m[[0m[0mTextInput[0m[0;34m,[0m [0mPreTokenizedInput[0m[0;34m,[0m [0mEncodedInput[0m[0;34m][0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0madd_special_tokens[0m[0;34m:[0m [0mbool[0m [0;34m=[0m [0;32mTrue[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0mpadding[0m[0;34m:[0m [0mUnion[0m[0;34m[[0m[0mbool[0m[0;34m,[0m [0mstr[0m[0;34m,[0m [0mPaddingStrategy[0m[0;34m][0m [0;34m=[0m [0;32mFalse[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0mtruncation[0m[0;34m:[0m [0mUnion[0m[0;34m[[0m[0mbool[0m[0;34m,[0m [0mstr[0m[0;34m,[0m [0mTruncationStrategy[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0mmax_length[0m[0;34m:[0m [0mOptional[0m[0;34m[[0m[0mint[0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0mstride[0m[0;34m:[0m [0mint[0m [0;34m=[0m [0;36m0[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0mreturn_tensors[0m[0;34m:[0m [0mOptional[0m[0;34m[[0m[0mUnion[0m[0;34m[[0m[0mstr[0m[0;34m,[0m [0mTensorType[0m[0;34m][0m[0;34m][0m [0;34m=[0m [0;32mNone[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0;34m**[0m[0mkwargs[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m    [0;34m)[0m [0;34m->[0m [0mList[0m[0;34m[[0m[0mint[0m[0;34m][0m[0;34m:[0m[0;34m[0m
    [0;34m[0m        [0;34m"""[0m
    [0;34m        Converts a string to a sequence of ids (integer), using the tokenizer and vocabulary.[0m
    [0;34m[0m
    [0;34m        Same as doing `self.convert_tokens_to_ids(self.tokenize(text))`.[0m
    [0;34m[0m
    [0;34m        Args:[0m
    [0;34m            text (`str`, `List[str]` or `List[int]`):[0m
    [0;34m                The first sequence to be encoded. This can be a string, a list of strings (tokenized string using the[0m
    [0;34m                `tokenize` method) or a list of integers (tokenized string ids using the `convert_tokens_to_ids`[0m
    [0;34m                method).[0m
    [0;34m            text_pair (`str`, `List[str]` or `List[int]`, *optional*):[0m
    [0;34m                Optional second sequence to be encoded. This can be a string, a list of strings (tokenized string using[0m
    [0;34m                the `tokenize` method) or a list of integers (tokenized string ids using the `convert_tokens_to_ids`[0m
    [0;34m                method).[0m
    [0;34m        """[0m[0;34m[0m
    [0;34m[0m        [0mencoded_inputs[0m [0;34m=[0m [0mself[0m[0;34m.[0m[0mencode_plus[0m[0;34m([0m[0;34m[0m
    [0;34m[0m            [0mtext[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m            [0mtext_pair[0m[0;34m=[0m[0mtext_pair[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m            [0madd_special_tokens[0m[0;34m=[0m[0madd_special_tokens[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m            [0mpadding[0m[0;34m=[0m[0mpadding[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m            [0mtruncation[0m[0;34m=[0m[0mtruncation[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m            [0mmax_length[0m[0;34m=[0m[0mmax_length[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m            [0mstride[0m[0;34m=[0m[0mstride[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m            [0mreturn_tensors[0m[0;34m=[0m[0mreturn_tensors[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m            [0;34m**[0m[0mkwargs[0m[0;34m,[0m[0;34m[0m
    [0;34m[0m        [0;34m)[0m[0;34m[0m
    [0;34m[0m[0;34m[0m
    [0;34m[0m        [0;32mreturn[0m [0mencoded_inputs[0m[0;34m[[0m[0;34m"input_ids"[0m[0;34m][0m[0;34m[0m[0;34m[0m[0m
    [0;31mFile:[0m      ~/mambaforge/envs/ml_svcs_p5/lib/python3.11/site-packages/transformers/tokenization_utils_base.py
    [0;31mType:[0m      method


```python
inputs = tokenizer.encode("Translate to English: Je t’aime.", return_tensors="pt")
inputs
```




    tensor([[89349,   288,  5413,   267,  1170,   259,   270,   293, 29543,   260,
                 1]])




```python
inputs = tokenizer.encode("Translate to English: Je t’aime.", return_tensors="pt")
outputs = model.generate(inputs, max_new_tokens=100)
print(tokenizer.decode(outputs[0]))
```

    <pad> I love you.</s>


## 4. Real Data


For the "sample data" we were give, we'll use one of the many churn prediction datasets available on Kaggle, and
you can find this particular one [here](https://www.kaggle.com/datasets/ninopadilla13/employee-churn?select=employee_churn.csv).

It contains the following variables:

- `avg_monthly_hrs`: average monthly hours worked per employee
- `department`: department where employee works
- `filed_complaint`: whether the employee ever filed a complaint
- `last_evaluation`: last evaluation score by manager (float between 0.0 and 1.0)
- `n_projects`: number of projects the employee has worked on
- `recently_promoted`: whether the employee was recently promoted
- `salary`: the employee's salary as a categorical variable (low, medium, high)
- `satisfaction`: the employee's satisfaction as a categorical variable (float between 0.0 and 1.0)
- `status`: the employee's status as a categorical variable (Employed, Left). This would be used as the target
  class for a machine learning model.
- `tenure`: number of years the employee has worked with the company (int between 2 and 10)

We will be adding the following variables:

- `name`
- `last_name`
- `email`
- `address`
- `city`
- `state`
- `country`
- `latitude`
- `longitude`
- `postal_code`
- `number`

Let's get started evaluating the original dataset.



```python
from pathlib import Path
import pandas as pd
```


```python
path = Path().cwd().parent.joinpath('data')
raw_data = path.joinpath('employee_churn', 'real', 'employee_churn.csv')
print(path, '\n', raw_data)
```


```python
df_raw = pd.read_csv(raw_data)
df_raw.head()
```


```python
df_raw.shape
```

There seem to be quite a few missing values, let's see how many exactly.



```python
df_raw.isna().sum() / df_raw.shape[0] * 100.0
```

## 4. Fake Data


#### Average Monthly Hours (Worked)


Average monthly hours worked is a ver important proxy (in my opinion) of happiness at work. If I average 310 hours
of work per month, I would be doing about 70 hours per week, and that's not sustainable no matter how much one loves
its job. There are exceptions to this, of course, but, for the sake of running a successful Tech Business, let's say
we'd rather keep our employees at a normal 40 hours per week.

Let's see what the actual distribution of hours worked per month is in the small dataset provided to us.



```python
df_raw['avg_monthly_hrs'].describe()
```

It seems that about 25% of the employees in our sample do work about 10/day, and this should probably be flagged as a concern.

Let's start creating som fake data based on the characteristics of our sample. For this, we'll use the `Fieldset`
and the `Locale` classes from mimesis





```python
from mimesis import Fieldset
from mimesis.locales import Locale
```


```python
fs = Fieldset(locale=Locale.EN, i=1000, seed=42)
```


```python
amh = fs("random.randints", amount=1, a=155, b=310)
```


```python
amh_clean = pd.Series(amh).apply(lambda x: x[0]).tolist()
```


```python

```


```python
df_raw['department'].value_counts(normalize=True, dropna=False) * 100
```


```python
deps_list = df_raw['department'].dropna().unique().tolist()
deps_list
```


```python
from typing import Any
```


```python
def random_departments(random, department_names: list) -> Any:
    return random.choice(department_names)
```


```python
fs.register_field('department', random_departments)
```


```python
from mimesis.keys import maybe
```


```python
fs('department', department_names=deps_list, key=maybe('nan', probability=0.02))[:5]
```


```python
df_raw['department'].value_counts(normalize=True, dropna=False).to_dict()
```


```python
department = fs('random.weighted_choice', choices=df_raw['department'].value_counts(normalize=True, dropna=False).to_dict())
```


```python
df_raw['filed_complaint'].value_counts(dropna=False, normalize=True)
```


```python
complaint = fs('random.weighted_choice', choices={1: 0.15, 'nan': 0.85})
complaint
```


```python

```

#### Last Evaluation



```python
df_raw['last_evaluation'].describe()
```


```python
fs('random.uniform', a=0.35, b=1.0, precision=5)[:10]
```


```python
evaluation = fs('random.uniform', a=0.35, b=1.0, precision=5)
```

#### Number of Projects



```python
n_projects = df_raw['n_projects'].value_counts(normalize=True).to_dict()
n_projects
```


```python
projects = fs('random.weighted_choice', choices=n_projects)
projects
```

#### Recently Promoted



```python
promos = df_raw['recently_promoted'].value_counts(normalize=True, dropna=False).to_dict()
promos
```


```python
promotions = fs('random.weighted_choice', choices=promos)
```

#### Salary



```python
salaries_cat = df_raw['salary'].value_counts(normalize=True, dropna=False).to_dict()
salaries_cat
```


```python
salaries = fs('random.weighted_choice', choices=salaries_cat)
```

#### Satisfaction



```python
df_raw['satisfaction'].describe()
```


```python
satisfaction = fs('random.uniform', a=0.45, b=1.0, precision=4, key=maybe(0.10, probability=0.05))
```

#### Target Variable: Churn



```python
target = df_raw['status'].value_counts(normalize=True, dropna=False).to_dict()
target
```


```python
target_y = fs('random.weighted_choice', choices=target)
```

#### Tenure



```python
tenure = df_raw['tenure'].value_counts(normalize=True, dropna=True).to_dict()
tenure
```


```python
tenure_val = fs('random.weighted_choice', choices=tenure)
```

#### Additional Variables



```python
name        = fs('name')
last_name   = fs('last_name')
email       = fs('email', domains=['creativeagency.com'])
address     = fs('address')
city        = fs('city')
state       = fs('state')
country     = fs('default_country')
latitude    = fs('latitude')
longitude   = fs('longitude')
postal_code = fs('postal_code')
number      = fs('phone_number')
```

Time to build our dataframe



```python
fs = Fieldset(locale=Locale.EN, i=2000)

df = pd.DataFrame({
    "___": ____,
    "___": ____,
    ...
})

df.head()
```

## 5. Synthetic Data



```python
df_real = pd.read_csv(raw_data)
df_real.head()
```

### 3.2 Generation Process



```python
from sdv.datasets.demo import get_available_demos

get_available_demos(modality='single_table')
```


```python
from sdv.datasets.local import load_csvs
```


```python
raw_data.parent
```


```python
datasets = load_csvs(folder_name=raw_data.parent)
datasets.keys()
```


```python
house_table = datasets['employee_churn']
type(house_table)
```


```python
house_table.head()
```


```python
from sdv.metadata import SingleTableMetadata

metadata = SingleTableMetadata()
```


```python
metadata.detect_from_dataframe(data=house_table)
```


```python
python_dict = metadata.to_dict()
python_dict
```


```python
metadata.visualize(
    show_table_details='summarized',
    output_filepath='my_metadata.png'
)
```


```python
metadata.validate()
```


```python
df_raw['status'].apply(lambda x: 1 if x=='Left' else 0)
```


```python
metadata.update_column(
    column_name='new_target',
    sdtype='boolean') # categorical and numerical go the same way but can have computer_representation='Float'
```


```python
metadata.update_column(
    column_name='checkin_date',
    sdtype='datetime',
    datetime_format='%d %b %Y')
```


```python
metadata.update_column(
    column_name='billing_address',
    sdtype='address',
    pii=True
)
```


```python
metadata.set_primary_key(column_name='guest_email')
```


```python
metadata.save_to_json(filepath='my_metadata_v1.json')
```


```python
from sdv.metadata import SingleTableMetadata
import json

with open('my_metadata_v1.json', 'r') as f:
    

metadata_obj = SingleTableMetadata.load_from_dict(metadata_dict)
```


```python
from sdv.single_table import GaussianCopulaSynthesizer
```


```python
GaussianCopulaSynthesizer??
```


```python
synthesizer = GaussianCopulaSynthesizer(metadata)
synthesizer.
```


```python
synthesizer.fit(df_real)
```


```python
synthetic_data = synthesizer.sample(num_rows=2000)
synthetic_data.head()
```


```python
synthetic_data.avg_monthly_hrs.describe()
```


```python
pd.Series(amh_clean).describe()
```


```python
synthesizer.save(
    filepath='../models/my_synthesizer.pkl'
)
```


```python
from sdv.evaluation.single_table import evaluate_quality

quality_report = evaluate_quality(
    real_data=df_real,
    synthetic_data=synthetic_data,
    metadata=metadata
)
```


```python
synthetic_data.shape, df_real.shape
```


```python
quality_report.get_score()
```


```python
quality_report.get_properties()
```


```python
quality_report.get_details(property_name='Column Shapes')
```


```python
from sdv.evaluation.single_table import run_diagnostic

diagnostic_report = run_diagnostic(
    real_data=df_real,
    synthetic_data=synthetic_data,
    metadata=metadata
)
```


```python
diagnostic_report.get_results()
```


```python
diagnostic_report.get_properties()
```


```python
diagnostic_report.get_details(property_name='Coverage')
```


```python
from sdv.evaluation.single_table import get_column_plot

fig = get_column_plot(
    real_data=df_real,
    synthetic_data=synthetic_data,
    column_name='n_projects',
    metadata=metadata
)

fig.show()
```


```python
from sdv.evaluation.single_table import get_column_pair_plot

fig = get_column_pair_plot(
    real_data=df_real,
    synthetic_data=synthetic_data,
    column_names=['satisfaction', 'last_evaluation'],
    metadata=metadata)
    
fig.show()
```

## 6. Final Thoughts


Both fake data and synthetic data are useful techniques for bootstrapping analyses, creating data engineering pipelines
and building machine learning models.

Synthetic Data helps us anonymize data and still get the benefits of analyzing personally identifiable information.

Fake data can serve as an excellent placeholder when building applications that require that we have a populated database
in place for testing purposes.

