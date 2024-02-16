pipeline {
    agent any
    options {
        // This is required if you want to clean before build
        skipDefaultCheckout(true)
    }
    stages {
        stage('Get Code') {
            steps {
                // Clean before build
                cleanWs()
                // Obtener código del repo de la rama develop
                git branch: 'master', url: 'https://github.com/hernancastel/todo-list-aws.git'
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy --stack-name todo-list-aws --resolve-s3 --region us-east-1 --no-confirm-changeset --no-fail-on-empty-changeset --force-upload --config-env production
                '''
            }
        }
        stage('Rest Test') {
            environment {
                BASE_URL = """${sh(
                        returnStdout: true,
                        script: 'sam list endpoints --stack-name todo-list-aws --region us-east-1 --output json | jq -r \'.[] | select(.LogicalResourceId=="ServerlessRestApi").CloudEndpoint | .[0]\' | tr -d \'\n\t\''
                    )}""" 
            }
            steps{
                sh '''
                    echo $BASE_URL
                    pytest --junitxml=result-rest-production.xml -k "not deletetodo and not updatetodo and not addtodo" -s ${WORKSPACE}/test/integration/todoApiTest.py
                '''
                junit 'result-rest-production.xml'
            }
        }
    }
}
