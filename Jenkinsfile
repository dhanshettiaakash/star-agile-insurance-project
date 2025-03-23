node {
    
    def mavenHome
    def mavenCMD
    def dockerCMD
    def tagName
    
    stage('Prepare Environment') {
        echo 'Initializing all the variables'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        dockerCMD = '/usr/bin/docker' // Correct Docker path
        tagName = "3.0"
    }

    stage('Git Code Checkout') {
        try {
            echo 'Checking out the code from Git repository'
            git 'https://github.com/dhanshettiaakash/star-agile-insurance-project'
        } catch (Exception e) {
            echo 'Exception occurred in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
            The Jenkins job ${JOB_NAME} has failed. Please check it immediately by clicking on the below link.
            ${BUILD_URL}''', subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} failed', to: 'shubham@gmail.com'
            error 'Git checkout failed'
        }
    }

    stage('Build the Application') {
        echo "Cleaning, Compiling, Testing, and Packaging the application"
        sh "${mavenCMD} clean package"        
    }

    stage('Publish Test Reports') {
        publishHTML([
            allowMissing: false, 
            alwaysLinkToLastBuild: false, 
            keepAll: false, 
            reportDir: 'target/surefire-reports', 
            reportFiles: 'index.html', 
            reportName: 'HTML Report'
        ])
    }

    stage('Containerize the Application') {
        echo 'Creating Docker image'
        sh "${dockerCMD} build -t aakki2503/insure-me:${tagName} ."
    }

    stage('Push to DockerHub') {
        echo 'Pushing the Docker image to DockerHub'
        withCredentials([usernamePassword(credentialsId: 'dock-password', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            sh "${dockerCMD} login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
            sh "${dockerCMD} push aakki2503/insure-me:${tagName}"
        }
    }

    stage('Run Docker Container on Port 8081') {
        echo 'Running Docker container on port 8081'
        sh "${dockerCMD} run -d -p 8081:8080 aakki2503/insure-me:${tagName}"
    }

    stage('Configure and Deploy to the Test Server') {
        echo 'Deploying using Ansible'
        ansiblePlaybook become: true, credentialsId: 'ansible-key', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
    }
} 

