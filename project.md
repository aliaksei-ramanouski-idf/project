Project
----
Project's reporter: *Ramanouski Aliaksei*

Group number: *m-sa2-12-20*

Description of application for deployment
----
|node-todo|
| :------------ |
|[link](https://github.com/manlyalex/node-todo)|
It's project written in Java Script programming language using Angular framework + MongoDB.
![Sheme](https://github.com/manlyalex/project/blob/master/sheme-03.jpg)

Technologies which were used in project
----
Orchestration: Jenkins, git.

Automation tools: Ansible.

####CI description:

Push(Developer) -> Run pipeline -> Git clone(Jenkins) -> Testing process -> Create new version -> Publish -> Slack notification

####CD description:

Git clone (new version or manually selected version) -> Npm install -> Artifacts copying and systemd restart -> Slack notification

```jenkinsfile
pipeline {
    agent any

    stages {
        stage('clone repo') {
            steps{
                deleteDir()
                git branch: 'master',
                url: 'https://github.com/manlyalex/node-todo'
            }
        }
        stage('ls') {
            steps{
                sh 'ls -l'
            }
        }
        stage('npm install') {
            steps{
                sh 'npm install'
            }
        }
        stage ('test lint') {
            steps{
                sh 'npm run lint'
            }
        }
        stage('config') {
            steps{
                sh '''
                  git config --global user.name $USER_NAME
                  git config --global user.email $USER_EMAIL
                  git status --porcelain
                '''
            }
        }
        stage ('New version'){
            steps {
                script {
                    newVersion = sh(script: 'npm version patch -m "Bumped version %s."', returnStdout: true).trim()
                }
            echo "${newVersion}"
            } 
        }
        stage('Publish') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'git_hub', usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {
                    sh('git push https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@github.com/manlyalex/node-todo.git master --tags')
                }
            }
        }
        stage ('Start_Deploy') {
            steps {
                build job: '03.npm_Deploy', parameters: [
                string(name: 'VERSION_TO_DEPLOY', value: "${newVersion}")
                ]
            }
        }
    }
    post {
    success {
      slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) ${newVersion}")
        }
    failure {
      slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }
}
```

```jenkinsfile
pipeline {
    agent any

    stages {
        stage('clone repo') {
            steps{
                deleteDir()
                git branch: 'master',
                url: 'https://github.com/manlyalex/node-todo'
            }
        }
        stage('deploy app with version') {
            steps{
                sh '''
                #!bin/bash
                if [ $VERSION_TO_DEPLOY != "" ]; then
                git checkout tags/$VERSION_TO_DEPLOY -b jenkins-branch
                fi
                '''
            }
        }
        stage('npm install') {
            steps{
                sh 'npm install'
            }
        }
        
        stage ("SSH copy app and restart server") {
          steps{
            withCredentials([sshUserPrivateKey(credentialsId: 'host-246', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'username')]) {
              script{
                    def remote = [:]
                    remote.name = "root"
                    remote.identityFile = "~/.ssh/id_rsa"
                    remote.host = "192.168.100.251"
                    remote.allowAnyHosts = true
                    remote.user = "$username"
                    
                    sshCommand remote: remote, command: "systemctl stop node_api"
                    sshRemove remote: remote, path: '/home/user/prod/*'
                    sshPut remote: remote, from: '.', into: '/home/user/prod'
                    sshCommand remote: remote, command: "systemctl start node_api"
                    }    
                }
            }
        }
    }
    post {
    success {
      slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    failure {
      slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }
}
```

