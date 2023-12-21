# django-todo
A simple todo app built with django

![image](https://github.com/Fir3eye/pr_02_django_todo/assets/93431222/6f314f3f-ac9d-44c7-b5f1-0ded73d0e680)


### Setup
To get this repository, run the following command inside your git enabled terminal
```bash
$ git clone https://github.com/Fir3eye/pr_02_django_todo.git
```
You will need django to be installed in you computer to run this app. Head over to https://www.djangoproject.com/download/ for the download guide

Once you have downloaded django, go to the cloned repo directory and run the following command

```bash
$ python manage.py makemigrations
```


This will create all the migrations file (database migrations) required to run this App.

Now, to apply this migrations run the following command
```bash
$ python manage.py migrate
```

One last step and then our todo App will be live. We need to create an admin user to run this App. On the terminal, type the following command and provide username, password and email for the admin user
```bash
$ python manage.py createsuperuser
```

That was pretty simple, right? Now let's make the App live. We just need to start the server now and then we can start using our simple todo App. Start the server by following command

```bash
$ python manage.py runserver
```

Once the server is hosted, head over to http://127.0.0.1:8000/todos for the App.

Cheers and Happy Coding :)


# Host website to another node use CI/CD Pipeline
```
pipeline {
    agent any

    environment {
        SSH_CREDENTIALS = credentials('ssh-agent')
        REMOTE_SERVER = 'ubuntu@remote-ip'
        REMOTE_PATH = '/var/www/html'
        LOCAL_PATH = './root/pr_02_django_todo'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    sh 'sudo apt install apache2'
                }
            }
        }

        stage('Deploy to Node') {
            steps {
                script {
                    // Copy files to the remote node using SSH
                    sshagent(credentials: [SSH_CREDENTIALS]) {
                        sh "scp -r ${LOCAL_PATH} ${REMOTE_SERVER}:${REMOTE_PATH}"
                    }
                }
            }
        }

        stage('Restart Web Server') {
            steps {
                script {
                    // Example: Restarting Apache web server
                    sshagent(credentials: [SSH_CREDENTIALS]) {
                        sh "ssh ${REMOTE_SERVER} 'sudo systemctl restart apache2'"
                    }
                }
            }
        }
    }
}


```

--------

## Second 

```
pipeline{
    agent any

    environment {
        APP_NAME = "im"
        RELEASE = "latest"
        DOCKER_USER = "fir3eye"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}" 

        SSH_CREDENTIALS = credentials('ssh-agent')
        REMOTE_SERVER = 'user@remote-node'
        REMOTE_PATH = '/var/www/html'
        LOCAL_PATH = './var/lib/jenkins/workspace/test-job'

    }
    stages{
        stage("Clean WorkSpace"){
            steps {
                cleanWs()
            }
        }
        stage("Checkout SCM"){
            steps {
                git branch: 'develop', credentialsId: 'github', url: 'https://github.com/Fir3eye/pr_02_django_todo.git'
            }
        }
        stage("Build Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        sh "docker images --format '{{.Repository}}:{{.Tag}}' | grep ${IMAGE_NAME} | grep -v ${RELEASE}-${BUILD_NUMBER} | grep -v latest | xargs -I {} docker rmi {} || true" 
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }
            }
        }
        stage("Push Docker Image"){
            steps{
                script {
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        } 
        stage ('Deploy to Container') {
            steps {
                sh 'docker run -d --name pet1 -p 8082:8000 fir3eye/im:latest'
            }
        }
    }
}
```
