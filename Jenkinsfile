pipeline {
    agent any

    parameters {        
        string(name: 'dockerHubUserName', defaultValue: 'userName', description: 'UserName for docker hub')
        //password(name: 'dockerHubPassword', defaultValue: 'SECRET', description: 'password for dockerhub')
        
        
        string(name: 'dockerImageNameApp', defaultValue: 'ImageName', description: 'image name for spp')
        string(name: 'dockerVersionApp', defaultValue: '1.0', description: 'docker image version for app')

        string(name: 'dockerImageNamePresent', defaultValue: 'ImageName', description: 'image name for presentation')
        string(name: 'dockerVersionPresent', defaultValue: '1.0', description: 'docker image version for presentation')
        
        string(name: 'gitHubURL', defaultValue: 'https://github.com/ebrahim0000p0/depi-final-project.git', description: ' git hub url for your repo for checkout')
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
        choice(name: 'action', choices: ['apply', 'destroy'], description: 'Select the action to perform')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION    = 'us-west-2'
        GITHUB_TOKEN          = credentials('github-token')
        AppDockerFile = './Web-And-App/application-tier'
        PresentDockerFile ='./Web-And-App/presentation-tier'

        AppImageName=''
        PresentImageName=''

    }


    stages {
        stage('Checkout') {
                    steps {
                        git branch: 'main', url: '${gitHubURL}'
                    }
                }    

        stage('Login Docker') {
            steps {
                    // sh 'docker login -u ${dockerHubUserName} -p ${dockerHubPassword}'  
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                                sh "echo $PASS | docker login -u $USER --password-stdin " 
                            }            
            }
        }

        //dockers for app tier
        stage('Build Docker Image For App Tier') {
            steps {
                sh 'docker build -t ${dockerImageNameApp}:${dockerVersionApp} ${AppDockerFile}'
            }
        }
 
        stage('Push Docker App Image') {
            steps {
                sh 'docker tag ${dockerImageNameApp}:${dockerVersionApp} ${dockerHubUserName}/${dockerImageNameApp}:${dockerVersionApp} && docker push ${dockerHubUserName}/${dockerImageNameApp}:${dockerVersionApp}'
                script {
                        AppImageName="${dockerHubUserName}/${dockerImageNameApp}:${dockerVersionApp}"
                        //this to print image  name the will be used in userdata file
                        echo "image name : ${AppImageName}"
                    }
            }
        }
        //dockers for presentaion tier
        stage('Build Docker Image For presentation Tier') {
                steps {
                    sh 'docker build -t ${dockerImageNamePresent}:${dockerVersionPresent} ${PresentDockerFile}'
                }
        }
       
        stage('Push Docker Presentation Image') {
            steps {
                sh 'docker tag ${dockerImageNamePresent}:${dockerVersionPresent} ${dockerHubUserName}/${dockerImageNamePresent}:${dockerVersionPresent} && docker push ${dockerHubUserName}/${dockerImageNamePresent}:${dockerVersionPresent}'
                script {
                        PresentImageName="${dockerHubUserName}/${dockerImageNamePresent}:${dockerVersionPresent}"
                        //this to print image  name the will be used in userdata file
                        echo "image name : ${PresentImageName}"
                    }
            }
        }
   
        stage('Terraform init') {
            steps {
                sh 'terraform init -upgrade '
            }
        }
        stage('Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
                sh 'terraform show -no-color tfplan > tfplan.txt'
            }
        }
        stage('Apply / Destroy') {
            steps {
                script {
                    if (params.action == 'apply') {
                        if (!params.autoApprove) {
                            def plan = readFile 'tfplan.txt'
                            input message: "Do you want to apply the plan?",
                            parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                        }

                        sh 'terraform ${action} -input=false tfplan'
                    } else if (params.action == 'destroy') {
                        sh 'terraform ${action} --auto-approve'
                    } else {
                        error "Invalid action selected. Please choose either 'apply' or 'destroy'."
                    }
                }
            }
        }

    }
}
