pipeline{
    agent any
    stages{
        stage('Get Code'){
            steps{
                sh 'ls -lsa'
                sh '''curl -O https://raw.githubusercontent.com/ccuevasbalinas/todo-list-aws-config/production/samconfig.toml'''
                sh 'ls -lsa'
                sh 'cat samconfig.toml'
            }
        }
        stage('Deploy'){
            steps{
                script{
                    
                    def already_deployed = sh(script: '''
                        aws cloudformation describe-stacks --stack-name todo-list-aws-production --region us-east-1
                    ''', returnStatus: true)
                    
                    if(already_deployed!=0){
                        sh'''
                            sam build --template-file template.yaml 
                            sam validate --config-file samconfig.toml --region us-east-1     
                            sam deploy --template-file template.yaml --config-file samconfig.toml --config-env production
                        '''
                    }

                }
            }
        }
        stage('Rest Test'){
            steps{
                script {
                    def apiUrl = sh(script: '''
                        aws cloudformation describe-stacks --stack-name todo-list-aws-production --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" --output text
                    ''', returnStdout: true).trim()

                    env.BASE_URL = apiUrl

                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        pytest --junitxml=result-unit.xml -k "test_api_gettodo or test_api_listtodos" test/integration/todoApiTest.py
                    '''

                    junit 'result*.xml'
                }
            }
        }
    }
}