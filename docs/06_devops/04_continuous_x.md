# 04 Continous X

> The most powerful tool we have as developers is automation.” ~ Scott Hanselman

![continuous_x](https://miro.medium.com/max/4800/1*WsbCzcT3HWRJ4SspEl-qdg.jpeg)

**Source:** Found on a [Medium](https://devopsdiaries.williamtsoi.net/continuous-delivery-in-cartoons-d67bbd6b6954) post where the author credits Nhan Ngo for the creation of this image.

## Table of Contents

1. [Overview](#1.-Overview)
2. [Learning Outcomes](#2.-Learning-Outcomes)
3. [Tools](#3.-Tools)
4. [Version Control](#4.-Version-Control)
    - Git
    - dvc
5. [CI/CD Pipelines](#3.-CI/CD-Pipelines)
    - GitHub Actions
    - CML
6. [Summary](#6.-Summary)

## 1. Overview

![automation](https://imgs.xkcd.com/comics/automation_2x.png)


At the heart of DevOps lies automation. While the word rolls off the tongue quite easily, accomplishing it in the software engineering world can be quite challenging. To manage the complexity, engineers have developed a system called, Continuous Delivery.

In the words of renoun software engineering, Continuous Delivery is
> "Continuous Delivery is a software development discipline where you build software in such a way that the software can be released to production at any time." ~ Martin Fowler

You’re doing continuous delivery when:
- Your software is deployable throughout its lifecycle
- Your team prioritizes keeping the software deployable over working on new features
- Anybody can get fast, automated feedback on the production readiness of their systems any time somebody makes a change to them
- You can perform push-button deployments of any version of the software to any environment on demand

If you have heard of the terms Continuous Integration and Continuous Deployment (or CI/CD), the you have heard of the the process of Continuous Delivery since, at its core, Continuous Delivery is CI/CD plus more.

In this section of the workshop we will cover some aspects of Continuous Delivery within the data science sphere.

## 2. Learning Outcomes

Before we get started, let's go over the learning outcomes for this section of the workshop.

By the end of this lesson you will be able to,
1. Discuss what Continuous Delivery, Continuous Integration, and Continuous Deployment is.
2. Will be able to use git to track code more effectively
3. Use dvc to track data, models, and experiments.
4. Understand how to create CI/CD pipelines for your projects.

## 3. Tools

Here are the tools we will be using for the this section of the workshop.

- [Git](https://git-scm.com/): "Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency." 
- [DVC](https://dvc.org/): "Data Version Control is a data versioning, ML workflow automation, and experiment management tool that takes advantage of the existing software engineering toolset you're already familiar with (Git, your IDE, CI/CD, etc.). DVC helps data science and machine learning teams manage large datasets, make projects reproducible, and better collaborate."
- [GitHub Actions](https://github.com/features/actions): "GitHub Actions makes it easy to automate all your software workflows, now with world-class CI/CD. Build, test, and deploy your code right from GitHub. Make code reviews, branch management, and issue triaging work the way you want."
- [CML](https://cml.dev/): "Continuous Machine Learning (CML) is an open-source library for implementing continuous integration & delivery (CI/CD) in machine learning projects. Use it to automate parts of your development workflow, including model training and evaluation, comparing ML experiments across your project history, and monitoring changing datasets."

## 4. Version Control

Version control is at the heart of the DevOps culture, profession, and mentality. Hence, mastering git and dvc will allow you to exploit, and take advantage of, different aspects of the trade that be beneficial in any profession you might choose to move to in the future.

We will go over some examples that by no means you need to adopt. Instead, pick what works and what doesn't and keep take those learnings with you beyond this session.

Let's get started with git first.

### 4.1 Git

Regardless if the project you are working on is open source or closed source, setting up git the right way when a project starts can pay tremendous dividens down the line. That said, in this section we will walk through the set up and development of a project using the files in our `src` folder and a new one.

Let's get started.


```python
!mkdir -p ~/our_pyconza_proj/src
```


```python
!ls ~/our_pyconza_proj
```

Let's go over the next steps together.

1. Open a new terminal in your editor of choice and navigate to the new folder.
2. Activate the same environment you have been using for this session `conda activate our_environment`
3. Navigate to the new folder `cd ~/our_pyconza_proj`
4. Copy our src files into the new directory with `cp pycon_za22_data_devops/src/*.py ~/our_pyconza_proj`
5. Initialize a new git repository with `git init`
6. Navigate to GitHub, log in, and create a new repository by clicking on top right-hand corner.
7. Add a `README.md` file with `echo "# pycon-za-test" >> README.md`
8. Let's view what we have and start tracking our files `git status && git add .`
9. Let's commit our changes `git commit -m "Our first commit"`
10. Now let's push our changes to our remote repository 
    - First we need to add it with `git remote add origin https://github.com/ramonpzg/pycon-za-test.git` or with ssh `git remote add origin git@github.com:ramonpzg/pycon-za-test.git`
    - Then we can push our changes with `git push -u origin master`
    - Note: You might get an authentication error if this is the first time you are pushing changes to GitHub. To bypass this momentarily, go to right-hand corner and click on your profile icon and then go to,
        - `Settings` > `Developer settings` > `Personal access tokens` > `Generate new token`.
        - Then, for the purpose of this tutorial only, click on all of them and then generate the token.
        - Copy it somewhere safe as you might need it more than once.
        - Try pushing your changes again and use the code when prompted

Great now that we got set up with git to start tracking our code, let's get going with dvc to start tracking our data.

### 4.2 Data Version Control

Let's get started with dvc.

1. Let's first create a data folder with a "fake" remote_storage init `mkdir -p data/remote_storage`
2. Run `python src/get_data.py`
3. Let's initialise our dvc repository with `dvc init`
4. Add "fake remote" storage with `dvc remote add -d our_storage data/remote_storage`
5. Let's add our first file with `dvc add data/02_part/raw/SeoulBikeData.csv`
6. We can now start stracking a metadata about our data to version control it and manipulate where it goes. `git add data/02_part/raw/.gitignore data/02_part/raw/SeoulBikeData.csv.dvc`
7. Commit changes and push `git commit -m "the first dataset" && git push`
8. Let's track our data with dvc by using
    - `dvc status` # to check what's up
    - `dvc commit` # to commit all of the data in our repo
    - `dvc push` # to move the data to our not so remote repo for this demo
9. Now let's start creating our pipeline.
    1. Let's remove our data again with `rm -rf data/02_part` to see the full example
    2. `dvc run -n get_data -d src/get_data.py -o data/02_part/raw/SeoulBikeData.csv python src/get_data.py`
    3. Two files got created, one is `dvc.yml` and the other is `dvc.lock`. The former keeps track of the pipeline we started creating with `dvc run`, and the latter contains less human friendly quotes (hashes and other metadata)
    4. We can start tracking `git add data/02_part/raw/.gitignore dvc.lock dvc.yaml`
    5. Commit incrementally `git commit -m "initial steps of our pipeline"` and push `git push`
    6. Let's keep adding to our pipeline `dvc run -n prepare -d src/prepare_data.py -d data/02_part/raw/SeoulBikeData.csv -o data/02_part/interim/clean_data.parquet python src/prepare_data.py`
    7. Now we can use `dvc config core.autostage true` to stop adding each step individually.
    8. Another one `dvc run -n split_data -d src/split_data.py -d data/02_part/interim/clean_data.parquet -o data/02_part/processed/train.parquet -o data/02_part/processed/test.parquet python src/split_data.py`
    9. Another one `dvc run -n train -d src/train_model.py -d data/02_part/processed/train.parquet -o models/rf_model.pkl python src/train_model.py`
    10. Last one `dvc run -n evaluate -d src/evaluate_model.py -d models/rf_model.pkl -d data/02_part/processed/test.parquet -M reports/metrics.json python src/evaluate_model.py`
    11. We can evaluate what our dag of steps looks like with `dvc dag`.
    12. Now we can run it with `dvc repro`. If we make any changes to a step in the pipeline, the previous steps that were not changed will not get rerun as their ourput gets cached in memory.
    13. Lastly, let's keep track of everything `git add . && git commit -m "pipeline ready" && git push`

## 5. CI/CD Pipelines

![cicd](https://kp2.in.htwg-konstanz.de/git/help/ci/img/cicd_pipeline_infograph.png)

### 5.1 GitHub Actions

![gh_actions](https://svrooij.io/assets/images/github-actions-banner.png)

Explain what it is.


Create a `mypy` 

GitHub Actions wokflow.


```python
!mkdir -p ~/our_pyconza_proj/.github/workflows
```


```python
%%writefile ~/our_pyconza_proj/requirements.txt

pandas
scikit-learn
numpy
flake8
black
```


```python
%%writefile ~/our_pyconza_proj/.github/workflows/demo_pipe.yml

name: PyCon ZA Demo 2022

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.9", "3.10"]

    steps:
      - uses: actions/checkout@v3
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      - name: Lint with flake8
        run: |
          # stop the build if there are Python syntax errors or undefined names
          # flake8 ./src/get_data.py --count --select=E9,F63,F7,F82 --show-source --statistics
          # exit-zero treats all errors as warnings. The GitHub editor is 127 chars wide
          flake8 ./src/get_data.py --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
      - name: Reformat with black
        run: |
          black ./src/split_data.py
      - name: Check types with mypy
        run: |
          mypy ./src/prepare_data.py
```

Time to test our pipeline
1. Run `git status`
2. Add files `git add .`
3. Commit and push `git commit -m "added CI and requirements.txt" && git push`
4. Immediately go to the **Actions** tab in your repo and then click on **PyCon ZA Demo** > **your commit message** > **build 3.10**, and you will be able to evaluate what happened inside our workflows.

Congrats! You just ran your first pipeline 😎

### 5.2 Continuous Machine Learning

Now it is time to create a workflow for our machine learning models.

Steps
1. git checkout -b new_params_v1
2. Change Random Forests Parameters
3. dvc status
4. dvc repro
5. dvc status
6. dvc push
7. dvc metrics show
8. dvc metrics diff --show-md master
9. git add .
10. git commit -m "Testing New Params"
11. git push --set-upstream origin new_params_v1
12. git push
13. mkdir .github/workflows
14. Add

```yaml

name: bikes-pipeline-test # (1)
on: push # (2)
jobs: # (3)
  run: # (4)
    runs-on: [ubuntu-latest] # (5)
    container: docker://dvcorg/cml:0-dvc2-base1 # (6)
    steps: # (7)
      - uses: actions/checkout@v2 # (8)
      - name: cml_run # (9)
        env: # (10)
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }} # (11)
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }} # (12)
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }} # (13)
        run: | # (14)
          pip install -r requirements.txt # (15)
          
          dvc repro # (16)
          dvc push # (17)
          git fetch --prune # (18)

          echo "# CML Report" > report.md # (19)
          dvc metrics diff --show-md master >> report.md # (20)
          cml-send-comment report.md # (21)
```

1. The name of our CI/CD pipeline.
2. Indicates to GitHub that the pipeline should run every time we push changes to our repository.
3. Indicates steps and dependencies that should run on push.
4. Run what follows.
5. Use the latest Ubuntu operating system to test our code.
6. The environment will use a container with DVC and CML already installed in it.
7. Follow the next steps after the operating system and the environment have been set up.
8. This action checks-out your repository under $GITHUB_WORKSPACE, so your workflow can access it.
9. The name of our CML run.
10. What follows are environment secrets needed for the run.
11. Your GitHub access token needs to be accessible by your environment.
12. Your AWS ACCESS KEY ID needs to be accessible within the run in other to push the models to s3.
13. Your AWS SECRET ACCESS KEY needs to be accessible within the run in other to push the models to s3.
14. Run the following commands one after the other. The | is very important.
15. Install our dependencies. See this below.
16. Reproduce the pipeline using our dvc.lock and dvc.yaml.
17. Push the data (if it changed) and push the model to our remote repository.
18. Updates all remote branches.
19. Create a report markdown file.
20. Add the metrics in the master branch to our report. If this were a different branch, compare the results with those in master.
21. Send the report as an email/pull request.


```python
%%writefile ~/our_pyconza_proj/.github/workflows/cml.yaml

name: testing-dvc-repro
on: [push]
jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v2
      - uses: iterative/setup-cml@v1
        with:
          version: '0.18.0'
      - uses: iterative/setup-dvc@v1
      - name: train_model
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          pip install -r requirements.txt
          dvc dag
          dvc repro
          dvc push
          echo "# My Report" > report.md
          dvc metrics show --show-md >> report.md
          dvc dag >> report.md
          cml send-comment report.md
          cml comment create --publish report.md
```



## 7. Summary

1. Continuous Delivery is the overaching representation of Continuous Improvement and Continuous Deployment.
2. CI/CD pipelines are built using a declarative approach, effectively making their composition a very human readable one.
3. CML not only helps us targe different infrastructure to run models on, but it also helps us sent reports and charts to our email and to our pull requests.
4. Git is the main hammer of DevOps engineers and we should strive to integrate with all the code we develop.
