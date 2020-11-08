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

        stage('get-keypair'){
            agent any
            steps{
                sh '''
                    if [ -f "mattsJenkinsKey3_public.pem" ]
                    then 
                        echo "file exists..."
                    else
                        aws ec2 create-key-pair \
                          --region us-east-2 \
                          --key-name mattsJenkinsKey3.pem \
                          --query KeyMaterial \
                          --output text > mattsJenkinsKey3.pem

                        chmod 400 mattsJenkinsKey3.pem

                        ssh-keygen -y -f mattsJenkinsKey3.pem >> mattsJenkinsKey3_public.pem
                    fi
                '''                
            }
        }

        stage('app-check_&_cluster-create'){
            agent any
            steps{
                // dont forget to add jenkins to sudoers ----> jenkins ALL=(ALL) NOPASSWD: ALL
                sh '''
                    #!/bin/sh
                    echo $HOME
                    running=$(sudo lsof -i:80) || true
                    if [ "$running" != '' ]
                    then
                        docker-compose down
                        exist="$(aws eks list-clusters | grep my-cluster)" || true
                        if [ "$exist" == '' ]
                        then
                            eksctl create cluster \
                                --name matts-cluster \
                                --version 1.18 \
                                --region us-east-1 \
                                --nodegroup-name my-nodes \
                                --node-type t2.small \
                                --nodes 1 \
                                --nodes-min 1 \
                                --nodes-max 2 \
                                --ssh-access \
                                --ssh-public-key  mattsJenkinsKey3_public.pem \
                                --managed
                        else
                            echo 'no need to create cluster...'
                        fi
                    else
                        echo 'app is not running with docker-compose up -d'
                    fi
                '''
            }
        }

        stage('create-ebs'){
            agent any
            steps{
                sh '''
                    VolumeId=$(aws ec2 describe-volumes --filters Name=tag:Name,Values="k8s-python-mysql-app" | grep VolumeId |cut -d '"' -f 4| head -n 1)  || true
                    if [ "$VolumeId" == '' ]
                    then
                        aws ec2 create-volume \
                            --availability-zone us-east-1a \
                            --volume-type gp2 \
                            --size 10 \
                            --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=k8s-python-mysql-app}]'
                        export EBS_VOLUME_ID=$VolumeId
                    fi
                    echo $EBS_VOLUME_ID
                '''
            }
        }

        stage('Up&Running?'){
            agent any
            steps{
                sh "kubectl apply -f k8s"                
            }
        }
    }
}