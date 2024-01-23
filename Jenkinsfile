pipeline {
    agent{
       label 'slave1'
     }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockeruser')
        registry = "siriuspallavi/book-store"
    }
    stages{
        stage('Docker Build') {
            steps {
                sh 'docker build -t siriuspallavi/book-store .'
              }
          }
        stage('Docker Login & Push') {
            steps {
		sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push siriuspallavi/book-store'
              }
          }
        stage('Remove Docker Images') {
            steps {
                sh 'docker rmi -f siriuspallavi/book-store'
              }
          }
        stage('Docker Network') {
            steps {
                script {
                    def existingNetwork = sh(script: "docker network ls --filter name=booknetwork --format '{{.Name}}'", returnStdout: true).trim()
                    if (!existingNetwork.contains('booknetwork')) {
                        sh 'docker network create -d bridge booknetwork'
                    } else {
                        echo 'booknetwork already exists.'
                    }
                }
            }
        }
       stage('Creating Database') {
            steps {
                script {
                    def existingMysqlContainer = sh(script: "docker ps -aqf name=mysql", returnStdout: true).trim()
                    if (existingMysqlContainer) {
                        echo 'MySQL container already exists.'
                    } else {
                        sh 'docker run -dt --name mysql --network booknetwork -p 3306:3306 -e MYSQL_ROOT_PASSWORD=Qwerty@123 mysql'
                    }
                }
            }
        }
        stage('Running the docker container') {
            steps {
                script {
                    def existingBookContainer = sh(script: "docker ps -aqf name=book", returnStdout: true).trim()
                    if (existingBookContainer) {
                        sh "docker rm -f ${existingBookContainer}"
                    }
                    sh 'docker run -dt --name book --network booknetwork -p 80:80 -e DB_SERVERNAME=mysql -e DB_USERNAME=root -e DB_PASSWORD=Qwerty@123 -e DB_NAME=mkbook siriuspallavi/book-store'
                }
            }
        }
	stage('Slack Notification') {
	    steps {
	        slackSend(
	            channel: 'jenkins-syncup',
	            color: '439FE0',
	            message: "Job '${env.JOB_NAME}' build #${env.BUILD_NUMBER} - Your book store application deployed successfully!",
	            teamDomain: 'siriuslync',
	            tokenCredentialId: 'slackuser',
	            username: 'slackuser'
	        )
	    }
	}

    }
}
