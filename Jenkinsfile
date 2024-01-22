@Library('Jenkins_shared_Library')

def COLOR_MAP = [
    'FAILURE' : 'danger',
    'SUCCESS' : 'good'
]
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    parameters {
        choice(name: 'action', choices: 'create\ndelete', description: 'Select create or destroy.')
        string(name: 'DOCKER_HUB_USERNAME', defaultValue: 'kubegourav', description: 'Docker Hub Username')
        string(name: 'IMAGE_NAME', defaultValue: 'youtube', description: 'Docker Image Name')
    }
    
    stages{
        stage('clean workspace'){
            steps{
               cleanWorkspace()
            }
        }
        stage('checkout from Git'){
            steps{
                checkoutGit('https://github.com/viddhant1205/Youtube-clone-app.git', 'main')
            }
        }
     
        stage('sonarqube Analysis'){
        when { expression { params.action == 'create'}}    
             steps{
                 sonarqubeAnalysis()
            }
        }
        stage('sonarqube QualitGate'){
        when { expression { params.action == 'create'}}    
            steps{
                script{
                    def credentialsId = 'sonar-cred'
                    qualityGate(credentialsId)
                }
            }
        }
        stage('Npm'){
        when { expression { params.action == 'create'}}    
            steps{
                npmInstall()
            }
        }
        
        stage('OWASP FS SCAN') {
        when { expression { params.action == 'create'}}
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy file scan'){
        when { expression { params.action == 'create'}}
            steps{
                trivyFs()
            }
        }
        
        stage('Docker Build'){
        when { expression { params.action == 'create'}}    
            steps{
                script{
                   def dockerHubUsername = params.DOCKER_HUB_USERNAME
                   def imageName = params.IMAGE_NAME

                   dockerBuild(dockerHubUsername, imageName)
                }
            }
        }
        
        stage('Trivy iamge'){
        when { expression { params.action == 'create'}}    
            steps{
                trivyImage()
            }
        }
        
        stage('Run container'){
        when { expression { params.action == 'create'}}
            steps{
                runContainer()
            }
        }
        stage('Remove container'){
        when { expression { params.action == 'delete'}}
            steps{
                deleteContainer()
            }
        }
        
        
    }    
        
     post {
         always {
             echo 'Slack Notifications'
             slackSend (
                 channel: '#jenkins-youtube', 
                 color: COLOR_MAP[currentBuild.currentResult],
                 message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
               )
           }
       }
   }
