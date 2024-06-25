pipeline{
    agent any
    stages{
        stage('Deploy'){
            steps{
                script{
                    
                    def already_deployed = sh(script: '''
                        aws cloudformation describe-stacks --stack-name todo-list-aws-production --region us-east-1
                    ''', returnStatus: true)

                    if(already_deployed!=0){
                        sh'''
                            sam build
                            sam validate --config-file samconfig.toml --region us-east-1     
                        '''
                                        
                        //def changes = sh script:'''
                            //sam deploy --template-file template.yaml --stack-name todo-list-aws-production --capabilities "CAPABILITY_IAM" --s3-bucket aws-sam-cli-managed-default-samclisourcebucket-nqyfysup146f --s3-prefix todo-list-aws --region us-east-1 --parameter-overrides Stage=\"production\"
                        //''',returnStatus:true

                        //if(changes == 0){
                            sh '''
                                sam deploy --template-file template.yaml --stack-name todo-list-aws-production --capabilities "CAPABILITY_IAM" --s3-bucket aws-sam-cli-managed-default-samclisourcebucket-nqyfysup146f --s3-prefix todo-list-aws --region us-east-1 --parameter-overrides Stage=\"production\"
                            '''
                        
                        //}
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