CI/CD is an integral part of modern development practices. It involves automating the build and testing of code every time a team member makes changes. 
Continuous deployment follows continuous integration by deploying all changes to the production environment.

In this tutorial, I will guide you on how to set up a CI/CD pipeline using Jenkins, a popular automation server, and GitHub.

Prerequisites:
```
A running Jenkins server
GitHub account
Basic understanding of Git, Docker, and Jenkins
```
The setup:

1. Ensure you have a Jenkins server running. Jenkins can be set up on a local machine or on a remote server, depending on your requirements.

If you do not have Jenkins installed, you can find instructions on the official Jenkins documentation: https://www.jenkins.io/sigs/docs/.

2. Creating a GitHub Repository
Go to GitHub and create a new repository.

Clone this repository to your local machine.
```
git clone https://github.com/your_username/repository_name.git
```
3. Create the application
For demonstration purposes, I will create a simple Python application. In your cloned repository, create a file app.py:

  ```python
  def hello_world():
    return "Hello, world!"

  if __name__ == "__main__":
    print(hello_world())

  ```
4. Writing the Unit Test
Write a simple test for the Python app. Create a file unitest_app.py:

  ```python
 import app

def test_hello_world():
    assert app.hello_world() == "Hello, world!"
  ```

The test can be executed with ```python -m unittest unitest_app.py```.

5. Dockerfile
In the root directory of the project, create a Dockerfile:

```
FROM python:3.9

WORKDIR /app

COPY . /app

EXPOSE 5555

CMD ["python", "app.py"]
```

This Dockerfile creates a Docker image with Python 3.9, copies our code into the image, and runs it when a container is started.

6. Jenkins Pipeline Configuration
In the root directory of the project, create a Jenkinsfile:

```
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t python-tesing-app .'
            }
        }
        stage('Test') {
            steps {
                sh 'docker run --rm python-testing-app python -m unittest unitest_app.py'
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker run -d -p 80:80 python-testing-app'
            }
        }
    }
}
```
In this pipeline, we have three stages:

Build: Builds a Docker image with the application.
Test: Runs the unit tests in a Docker container.
Deploy: Deploys the application in a Docker container.

7. Set up the CI/CD Pipeline in Jenkins
In Jenkins, create a new item - choose a "Multibranch Pipeline" and name it.

In the "Branch Sources" section, select "Add source", choose "Git" and then provide the repository URL.

In the "Build Configuration" section, select "by Jenkinsfile" and make sure the script path is "Jenkinsfile" or the path to your Jenkinsfile if you've placed it elsewhere.

Save the configuration. Jenkins will scan your repository and build a job.

Every commit pushed to the repository will now trigger a new build in Jenkins. Jenkins will build a Docker image with the application, run the tests, and if they pass, it will deploy the application.

## Michael Sanchez
