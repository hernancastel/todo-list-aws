pipeline {
    agent any
    options {
        // This is required if you want to clean before build
        skipDefaultCheckout(true)
    }
    stages {
        stage ('Get Code') {
            steps {
                // Clean before build
                cleanWs()
                // Obtener código del repo de la rama develop
                git branch: 'develop', url:'https://github.com/hernancastel/todo-list-aws.git'
            }
        }
        
        stage('Static test') {
            parallel{
                stage('flake8'){
                    steps {
                        sh '''
                            flake8 --exit-zero --format=pylint src > flake8.out
                        '''
                        recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates:[[threshold:10, type: 'TOTAL', unstable: false],[threshold: 12, type: 'TOTAL', unstable: false]]
                	}
                }
                stage('bandit') {
                    steps {
                	    sh '''
                			bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                		'''
                		recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates:[[threshold:1, type: 'TOTAL', unstable: false],[threshold: 2, type: 'TOTAL', unstable: false]]
                	}
                }
            }
        }
         
        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam deploy --stack-name todo-list-aws --resolve-s3 --region us-east-1 --no-confirm-changeset --no-fail-on-empty-changeset --force-upload --config-env staging

                '''
            }
        }
         
        stage('Rest') {
            environment {
                BASE_URL = """${sh(
                        returnStdout: true,
                        script: 'sam list endpoints --stack-name todo-list-aws --region us-east-1 --output json | jq -r \'.[] | select(.LogicalResourceId=="ServerlessRestApi").CloudEndpoint | .[0]\' | tr -d \'\n\t\''
                    )}""" 
            }
            steps {
                sh '''
                    pytest --junitxml=result-rest.xml -s ${WORKSPACE}/test/integration/todoApiTest.py
                '''
                junit 'result-rest.xml'
            }
        }
        
         stage('Promote') {
            environment {
                GIT_ACCESS_TOKEN = credentials('GIT_ACCESS_TOKEN')
            }
            steps {
                sh '''
                    git remote set-url origin https://x-access-token:$GIT_ACCESS_TOKEN@github.com/hernancastel/todo-list-aws.git
                    git checkout master
                    git merge develop
                    git push origin master
                '''
            }
        }
    }
}
