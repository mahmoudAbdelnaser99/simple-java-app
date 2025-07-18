pipeline {
    agent { label 'aws-agent' }

    stages {
        stage('build') {
            steps {
                sh 'docker build -t java-app .'
            }
        }

        stage('push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    passwordVariable: 'Password',
                    usernameVariable: 'Username'
                )]) {
                    sh '''
                        docker login --username $Username --password $Password
                        docker tag java-app $Username/java-app
                        docker push $Username/java-app
                    '''
                }
            }
        }

        stage('deploy') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY'),
                    string(credentialsId: 'aws-session-token', variable: 'AWS_SESSION_TOKEN')
                ]) {
                    sh '''
                        export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                        export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                        export AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN

                        aws eks update-kubeconfig --region us-east-1 --name eks
                        kubectl apply -f ./k8s/deployment.yaml
                    '''
                }
            }
        }
    }
}
