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
        // stage("compile"){
        //    agent{
        //        docker{
        //            image 'python:alpine'
        //        }
        //    }
        //    steps{
        //        withEnv(["HOME=${env.WORKSPACE}"]) {
        //             sh 'pip install -r requirements.txt'
        //             sh 'python -m py_compile src/*.py'
        //         }
        //    }
        // } 
       
        // stage('test'){
        //     agent {
        //         docker {
        //             image 'python:alpine'
        //         }
        //     }
        //     steps {
        //         withEnv(["HOME=${env.WORKSPACE}"]) {
        //             sh 'python -m pytest -v --junit-xml results.xml src/appTest.py'
        //         }
        //     }
        //     post {
        //         always {
        //             junit 'results.xml'
        //         }
        //     }
        // }   

        // stage('build'){
        //     agent any
        //     steps{
        //         sh "docker build -t matt/jenkins-handson ."
        //         sh "docker tag matt/jenkins-handson:latest  046402772087.dkr.ecr.us-east-1.amazonaws.com/matt/jenkins-handson:latest"
        //     }
        // }

        // stage('push'){
        //     agent any
        //     steps{
        //         sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 046402772087.dkr.ecr.us-east-1.amazonaws.com"
        //         sh "docker push 046402772087.dkr.ecr.us-east-1.amazonaws.com/matt/jenkins-handson:latest"
        //     }
        // }
        // stage('compose'){
        //     agent any
        //     steps{
        //         sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 046402772087.dkr.ecr.us-east-1.amazonaws.com"
        //         sh "docker-compose up -d"
        //     }
        // }
        stage('app-check'){
            agent any
            steps{
                sh '''
                    echo hello world...
                    sudo lsof -i:80
                    if [ $? != 0 ]
                    then
                    echo working...
                    else 
                    echo not working...
                    fi
                '''
                // sh "lsof -i:80"
                // sh " if[[ $? -eq 0 ]] "
                // sh "   then"
                // sh "echo 'working...'"
                // sh "else"
                // sh "echo 'not working...'"
                // sh " fi" 
            }
        }
    }
}