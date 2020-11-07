pipeline{
    agent any
    environment{
        MYSQL_DATABASE_HOST = "database-42.cbanmzptkrzf.us-east-1.rds.amazonaws.com"
        MYSQL_DATABASE_PASSWORD = "Clarusway"
        MYSQL_DATABASE_USER = "admin"
        MYSQL_DATABASE_DB = "phonebook"
        MYSQL_DATABASE_PORT = 3306
        PATH="/usr/local/bin/:${env.PATH}"
    }

    stages{
        stage("compile"){
           agent{
               docker{
                   image 'python:alpine'
               }
           }
           steps{
               withEnv(["HOME=${env.WORKSPACE}"]) {
                    sh 'pip install -r requirements.txt'
                    sh 'python -m py_compile src/*.py'
                }
           }
        } 
       
        stage('test'){
            agent {
                docker {
                    image 'python:alpine'
                }
            }
            steps {
                withEnv(["HOME=${env.WORKSPACE}"]) {
                    sh 'python -m pytest -v --junit-xml results.xml src/appTest.py'
                }
            }
            post {
                always {
                    junit 'results.xml'
                }
            }
        }   

        stage('build'){
            agent any
            steps{
                sh "docker build -t matt/jenkins-handson ."
                sh "docker tag matt/jenkins-handson:latest  046402772087.dkr.ecr.us-east-1.amazonaws.com/matt/jenkins-handson:latest"
            }
        }

        stage('push'){
            agent any
            steps{
                sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 046402772087.dkr.ecr.us-east-1.amazonaws.com"
                sh "docker push 046402772087.dkr.ecr.us-east-1.amazonaws.com/matt/jenkins-handson:latest"
            }
        }
        stage('compose'){
            agent any
            steps{
                sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 046402772087.dkr.ecr.us-east-1.amazonaws.com"
                sh "docker-compose up -d"
            }
        }
        stage('app-check'){
            agent any
            steps{
                // add jenkins to sudoers
                sh '''
                    #!/bin/sh
                    running=$(sudo lsof -i:80) || true
                    if [ "$running" != '' ]
                    then
                        docker-compose down
                        exist="$(aws eks list-clusters | grep my-cluster)" || true
                        if [ "$exist" == '' ]
                        then
                        eksctl create cluster \
                            --name my-cluster \
                            --version 1.17 \
                            --region us-east-1 \
                            --nodegroup-name my-nodes \
                            --node-type t2.small \
                            --nodes 1 \
                            --nodes-min 1 \
                            --nodes-max 2 \
                            --ssh-access \
                            --ssh-public-key  mattsKey \
                            --managed
                        else
                            echo no need to create cluster...
                        fi
                    else 
                    echo app is not running...
                    fi
                '''
            }
        }
    }
}