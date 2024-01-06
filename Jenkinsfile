pipeline {
    agent any
    environment{
        credential = 'github-appserver'
        server = 'team3@103.127.96.218'
        directory = '/home/team3/literature-frontend'
        branch = 'main'
        service = 'frontend'
        image = 'rakhafe/frontend-team3'
    }
    stages {
        stage('Pull code dari repository'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    docker compose down
                    cd ${directory}
                    git pull origin ${branch}
                    exit
                    EOF'''
                }
            }
        }
        stage('Building application'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    cd ${directory}
                    docker build -t ${image}:${BUILD_NUMBER} .
                    exit
                    EOF'''
                }
            }
        }
        stage('Testing application'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    cd ${directory}
                    docker run --name test_fe -p 3000:3000 -d ${image}:${BUILD_NUMBER}
                    wget --spider localhost:3000
                    docker stop test_fe
                    docker rm test_fe
                    exit
                    EOF'''
                }
            }
        }
        stage('Deploy aplikasi on top docker'){
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF
                    sed -i '32c\\    image: ${image}:${BUILD_NUMBER}' docker-compose.yaml
                    docker compose up -d 
                    cd ${directory}
                    exit
                    EOF'''
                }
            }
        }
        stage('Push image to docker hub'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    cd ${directory}
                    docker push ${image}:${BUILD_NUMBER}
                    exit
                    EOF'''
                }
            }
        }
        stage('send notification to discord'){
            steps {
                discordSend description: "frontend-team3 notify", footer: "team3 notify frontend", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: "https://discord.com/api/webhooks/1192097833042579536/qInbfRCW9G8UdivrNlhqeO-AvSijRIdWYbTaGKgKx8sSDR8bCHllZrZ8dPQR3HwKKjau"
            }
        }
    }
}
