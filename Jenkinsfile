pipeline{
    agent any
    tools{
        // jdk 'jdk17'
        // nodejs 'node16'
        ansible 'ansible'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'master', url: 'https://github.com/sankar0812/swiggy-app-github-actions.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=swiggy-app \
                    -Dsonar.projectKey=swiggy-app '''
                }
            }
        }
        // stage("quality gate"){
        //    steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
        //         }
        //     } 
        // }
        // stage('Install Dependencies') {
        //     steps {
        //         sh "npm install"
        //     }
        // }
        // stage('OWASP FS SCAN') {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        // stage('TRIVY FS SCAN') {
        //     steps {
        //         sh "trivy fs . > trivyfs.txt"
        //     }
        // }
        stage("Docker Build & Push"){
            steps{
                script{
                      withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                         sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                         sh "docker build -t ideauxhub/swiggy-app:${env.BUILD_NUMBER} ."
                         sh "docker push ideauxhub/swiggy-app:${env.BUILD_NUMBER}"
                      }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image ideauxhub/swiggy-app:${env.BUILD_NUMBER} > trivyimage.txt" 
            }
        }
        stage('Deploy with Ansible') {
           steps {
               script {
                  ansiblePlaybook credentialsId: 'ubuntu-server', 
                      disableHostKeyChecking: true, installation: 'ansible', 
                      inventory: '/etc/ansible/hosts', playbook: '/etc/ansible/deploy.yml', vaultTmpPath: '',
                      extraVars: [ build_number: "${env.BUILD_NUMBER}" ]
               }
           }
        }
        // stage('Deploy with Ansible') {
        //     steps {
        //         script {
        //             ansiblePlaybook(
        //                 playbook: '/etc/ansible/deploy.yml',
        //                 inventory: '/etc/ansible/hosts',
        //                 extras: '-e "app_name=netflix"'
        //             )
        //         }
        //     }
        // }
        // stage('Deploy to container'){
        //     steps{
        //         sh 'docker run -d -p 8081:80 sankar0812/netflix:latest'
        //     }
        // }
    //     stage('Deploy to kubernets'){
    //         steps{
    //             script{
    //                 dir('Kubernetes') {
    //                     withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
    //                             sh 'kubectl apply -f deployment.yml'
    //                             sh 'kubectl apply -f service.yml'
    //                     }   
    //                 }
    //             }
    //         }
    //     }

     }
//     post {
//      always {
//         emailext attachLog: true,
//             subject: "'${currentBuild.result}'",
//             body: "Project: ${env.JOB_NAME}<br/>" +
//                 "Build Number: ${env.BUILD_NUMBER}<br/>" +
//                 "URL: ${env.BUILD_URL}<br/>",
//             to: 'iambatmanthegoat@gmail.com',                                #change mail here
//             attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
//         }
//     }
 }
