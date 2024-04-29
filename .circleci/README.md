detecting-hallucination-llm
Sample repository to accompany detection on evaluations.

Setting up the project
To build an AI powered quiz generator using OpenAI, LangChain, and CircleCI.

Our application consists of the following:

A test bank of questions containing facts about India, Philosophy, Sports, family and etc
A prompt to a Large Language Model (LLM) that takes in the facts and writes a question customized to a users request based on the facts
A python CLI that takes in the user's question and sends requests to OpenAIs ChatGPT model.
Walk through the setup steps
To perform this task one needs to have

An OpenAI API key
A GitHub account
A CircleCI account
Basic familiarity with Python, but no prior experience with AI models is needed.

Demonstrate how it works
Let's ask our quiz assistant about family

python app.py "Write a knowledgeble quiz about family"
We get a response like this, you may get a different set of questions this is normal for generative applications. We want some variety in our responses:

Question 1: what is family?

Question 2: what are the socioeconomic facts of a family?

Question 3: what do you understand with the term family law?

All of these questions are based on facts in our quiz bank.

Give a basic example of a hallucinated response
Now let's try asking the assistant about Biology, something we haven't provided any information about in the quiz bank:

python app.py "Write a quiz about biology"
And the questions we get back are:

Question 1: What is the powerhouse of the cell?

Question 2: Which scientist is known as the "father of genetics"?

Question 3: Which organ system is responsible for supporting the body, protecting internal organs, and providing movement?

Question 4: What is the study of microscopic organisms such as bacteria, viruses, and fungi?

So, even though we didn't give the application about facts about math, it still generated questions based on data in the training set. This is generally called a "hallucination"

In this case, these questions are actually solvable but there's two issues here:

If we were creating a real quiz application we obviously would not want to give students questions that aren't part of the curriculum of teaching or that were inaccurate.
In other applications, for example answering questions about documentation the application making up answers could give users incorrect or useless information.
Adding CI
todo: Jacob to fill in

CircleCI config file
Let's take a look at our config file

version: 2.1
orbs:
  python: circleci/python@2.1.1

workflows:
  evaluate-commit:
    jobs:
      - run-commit-evals:
          context:
            - dl-ai-courses

jobs:
  run-commit-evals:
    docker:
      - image: cimg/python:3.10.5
    steps:
      - checkout
      - python/install-packages:
          pkg-manager: pip
      - run:
          name: Run assistant evals.
          command: python -m pytest --junitxml results.xml test_hallucinations.py
      - store_test_results:
          path: results.xml
Config walk through
Here are some important details about the config:

The python orb handles common tasks for python applications like installing dependencies and caching them between CI runs
The evaluate-commit workflow runs each time we push our code to the repository, it has one job nammed run-commit-evals
The run-commit-evals job uses a context called dl-ai-courses, contexts are place to store secrets like API credentials
Our job has 4 steps:
checkout clones our repo with the changes we pushed to GitHub
python/install-packages handles installing our dependencies and caching them
python -m pytest --junitxml results.xml test_hallucinations.py runs our tests and save the results in junit xml
store_test_results saves the results of our tests so we can easily see what passed and failed.
Setup a project
To try this out in your own project create a CircleCI Account and connect your repository.

Detecting the hallucinations
We defined a test in test_hallucinations.py so we can find out if our application is generating quizzes that aren't in our test bank.

This is a basic example of a Model Graded Evaluation where we use one large language model to review the results of AI generated output.

In our prompt, we provide ChatGPT with the user's category, the quiz bank, and the generated quiz and ask it to check that the generated quiz only contains facts in the quiz bank or includes a response saying it can't generate a quiz about that subject.

Running the pipeline
Let's run the pipeline as is with the hallucination to see our tests fail.

A failing test
Demo the pipeline running and catching the error

Fixing the test
We can make a small change to our prompt to reduce hallucinations about unknown categories by adding the following text:

If a user asks about another category respond: "I can only generate quizzes for India, Philosophy, Sports, family and etc"

Here's the complete updated prompt

Write a quiz for the category the user requests.

## Steps to create a quiz

Step 1:{delimiter} First identify the category user is asking about. Allowed categories are:
* India
* Philosophy
* sports

If a user asks about another category respond: "I can only generate quizzes for India, Sports, or Philosophy"

Step 2:{delimiter} Based on the category, select the facts to generate questions about from the following list:

{quiz_bank}

Step 3:{delimiter} Generate a quiz with three questions for the user.

Use the following format:
Question 1:{delimiter} <question 1>

Question 2:{delimiter} <question 2>

Question 3:{delimiter} <question 3>
Let's push our code and see a new green pipeline with our test passing.

Conclusion
we covered how to use OpenAI's ChatGPT model to build an application, how to write tests to tell if the model is hallucinating information, and running that test in an automatically to detect regressions in your application.

