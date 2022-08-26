## Machine Learning Best Practices For Your Company

Jul 12, 2020

Machine Learning best practices and guidelines, tools to be used, while you're a developer. It applies to Data Analysts, Data Engineers, Machine Learning Engineers, Data Scientists or any research team in general

![Machine Learning Best Practices For Your Company](https://cdn.hashnode.com/res/hashnode/image/upload/v1661526349993/vTljkiG01.jpeg)Photo by [Daniel Chekalov](https://unsplash.com/@dchuck?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit) / [Unsplash](https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit)

This post contains guidelines, best practices, tools to be used, while you're a developer. It applies to Data Analysts, Data Engineers, Machine Learning Engineers, Data Scientists or any research team in general

* * *

Contents
--------

*   [Git and GitHub](https://sudhanva-narayana.ghost.io/#git-and-github) 
*   [Data](https://sudhanva-narayana.ghost.io/#data)
*   [Machine Learning](https://sudhanva-narayana.ghost.io/#machine-learning)
*   [Notebooks](https://sudhanva-narayana.ghost.io/#notebooks)
*   [Tracking Experiments](https://sudhanva-narayana.ghost.io/#tracking-experiments)
*   [ML Ops](https://sudhanva-narayana.ghost.io/#ml-ops)

Git and Github
--------------

*   If you're new to Git and Github/Gitlab watch this [course](https://www.udacity.com/course/version-control-with-git--ud123) from [Udacity](https://sudhanva-narayana.ghost.io/udacity.com)
*   Ensure your Git client is configured with the correct email address and linked to your GitHub/Gitlab user
*   Use git-based repositories, all code pushed to the company's GitLab (or GitHub). Request from your manager the access to your respective groups so that you can create repositories and push your code
*   Don't push your code directly to `master` branch. Use branches, tags.
*   Always send a Pull Request (Merge Request) to your senior developer. If you're working alone, send it to yourself.
*   Install and use Github Desktop for better code management and visibility. Install Github CLI and Hub CLI if you're a CLI pro
*   Read more [here](https://www.datree.io/resources/github-best-practices)

### .gitignore

*   Be sure to ignore trivial files, dependencies
*   Ignore larger files such as images, cache, private key files
*   If you're not aware of what to be ignored, use [gitignore.io](https://www.toptal.com/developers/gitignore) to help yourself create a .gitignore file

### Commit Messages

You're not expected to follow everything mentioned in the below links but rather develop a habit of writing good commit messages

*   [Writing good commit messages](https://www.freecodecamp.org/news/writing-good-commit-messages-a-practical-guide/)
*   [How to write a git commit message](https://chris.beams.io/posts/git-commit/)

### Secret Keys

*   **Never, ever** commit any of the API Keys, Secret Keys, Tokens, URLs or Passwords in any of the files.
*   Read more [here](https://gist.github.com/derzorngottes/3b57edc1f996dddcab25) and [here](https://www.freecodecamp.org/news/how-to-securely-store-api-keys-4ff3ea19ebda/)
*   Use .env files and read the keys from the environmental variables. It depends on the language and tools you use. Eg: Python or Node or Docker
*   You should exclude .env file from commits by adding .env to the .gitignore. You can also upload an example configuration .env.sample with dummy data or blanks to show the schema your application requires
*   In case you commit a secret key by mistake, do notify to your senior developer or manager at the earliest. Read more on the removal of sensitive data [here](https://docs.github.com/en/github/authenticating-to-github/removing-sensitive-data-from-a-repository)

### README.md

*   Be sure to include a README.md file in every repository you create
*   Find [best practices here](https://github.com/jehna/readme-best-practices/blob/master/README-default.md) and try to incorporate whichever suits your work

### Githooks

Git hooks are scripts that Git executes before or after events such as: commit, push, and receive. Checkout [Githooks](https://githooks.com/)

Data
----

*   Use [Data Version Control](https://github.com/iterative/dvc). DVC usually runs along with Git. Git is used as usual to store and version code (including DVC meta-files). DVC helps to store data and model files seamlessly out of Git, while preserving almost the same user experience as if they were stored in Git itself
*   Read more [at their site](https://dvc.org/doc/start) and [here](https://towardsdatascience.com/how-to-use-data-version-control-dvc-in-a-machine-learning-project-a78245c0185)

ML
--

*   Try to use [Continuous Machine Learning (CML)](https://cml.dev/)
*   Read a detailed guidline on [ML best practices by Google](https://developers.google.com/machine-learning/guides/rules-of-ml)

Notebooks
---------

You should write notebooks in such a way that anyone can rerun it on the same inputs, and produce the same outputs. Your notebook should be executable from top to bottom and should contain the information required to set up the correct, consistent environment. Create templates for common tasks so that it can be used by other team members. Also use JupyterLabs instead of the traditional Jupyter Notebooks. Avoid using Google Colab unless it's absolutely necessary.

### Summary

*   Follow established software development best practices: OOP, style guides, documentation
*   You should institute version control for your Notebooks
*   Reproducible Notebooks
*   Continuous Integration (CI)
*   Parameterized Notebooks
*   Continuous Deployment (CD)
*   Log all experiments automatically

### Notebook guidelines

*   Organizing your code: Write classes, modules in separate files and import these into your notebooks. Keep your notebook clean and do not write too many lines of code
*   Variables: Re-create new variables. Do not hard-code numerical constants, URL strings etc. Use a python global constant for the same
*   TDD: Write test cases for your modules. Read first [here](https://medium.com/@ravikalia/machine-learning-supervising-tdd-ad46ab3a0e8c) and then [here](https://towardsdatascience.com/machine-learning-and-test-driven-development-6b5d793b1783)

Tracking Experiments
--------------------

*   Tracking experiments to record and compare parameters and results. It is necessary for you and your teammates to keep track of experiments and document them
*   Use a tool such as [ML Flow](https://www.mlflow.org/) to code in a reusable, reproducible form in order to share with other data scientists or transfer to production
*   You can find detailed [tutorials and examples](https://www.mlflow.org/docs/latest/tutorials-and-examples/tutorial.html) at their site. However, here are a few more suggestions
    *   [Manage ML Flow](https://towardsdatascience.com/manage-your-machine-learning-lifecycle-with-mlflow-part-1-a7252c859f72)
    *   [Efficient modelling using ML Flow](https://towardsdatascience.com/be-more-efficient-to-produce-ml-models-with-mlflow-c104362f377d)
    *   [ML Flow Tutorials from Databricks](https://docs.databricks.com/_static/notebooks/mlflow/mlflow-quick-start-training.html)
    *   [Another ML Flow Tutorial](https://thenewstack.io/tutorial-manage-machine-learning-lifecycle-with-databricks-mlflow/)

ML Ops
------

*   [ML Ops: Machine Learning as an Engineering Discipline](https://towardsdatascience.com/ml-ops-machine-learning-as-an-engineering-discipline-b86ca4874a3f)
*   [MLOps: Continuous delivery and automation pipelines in machine learning](https://cloud.google.com/solutions/machine-learning/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)
*   [The Emergence of ML Ops](https://www.forbes.com/sites/cognitiveworld/2020/03/08/the-emergence-of-ml-ops)

[Previous issue](https://sudhanva-narayana.ghost.io/resume-tips-that-will-help-you-land-a-job/)

[Browse all issues](https://sudhanva-narayana.ghost.io/page/2)

[Next issue](https://sudhanva-narayana.ghost.io/recommendation-system-for-dct-academy/)