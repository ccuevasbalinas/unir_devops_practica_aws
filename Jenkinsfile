pipeline{
    agent any
    stages{
        stage('Static Tests'){
            parallel{
                stage('Flake8'){
                    steps{  
                        catchError(buildResult:'UNSTABLE',stageResult:'FAILURE'){
                            sh '''
                                python -m flake8 --exit-zero --format=pylint src > flake8.out
                            '''
                            recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
                        }
                    }
                }
                stage('Bandit'){
                    steps{  
                        catchError(buildResult:'UNSTABLE',stageResult:'FAILURE'){
                            sh '''
                                bandit -r src -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}"    
                            '''
                            recordIssues tools: [pyLint(name: 'Flake8', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                        }
                    }     
                }
            }
        }
        stage('Deploy'){
            steps{
                script{

                    def already_deployed = sh(script: '''
                        aws cloudformation describe-stacks --stack-name todo-list-aws-staging --region us-east-1
                    ''', returnStatus: true)

                    if(already_deployed!=0){
                        sh'''
                            sam build
                            sam validate --config-file samconfig.toml --region us-east-1   
                            sam deploy --template-file template.yaml --stack-name todo-list-aws-staging --capabilities "CAPABILITY_IAM" --s3-bucket aws-sam-cli-managed-default-samclisourcebucket-nqyfysup146f --s3-prefix todo-list-aws --region us-east-1 --parameter-overrides Stage=\"staging\"  
                        '''
                    }
                }
            }
        }

        stage('Rest Test'){
            steps{
                script {
                    def apiUrl = sh(script: '''
                        aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" --output text
                    ''', returnStdout: true).trim()

                    env.BASE_URL = apiUrl

                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        pytest --junitxml=result-unit.xml test/integration/todoApiTest.py
                    '''

                    junit 'result*.xml'
                }
            }
        }
        stage('Promote') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'MyToken', variable: 'GIT_TOKEN')]) {
                        sh '''
                            git config --global user.name "ccuevasbalinas"
                            git config --global user.email "ccuevasbalinas@gmail.com"
                            git config -l
                            ls -lsa
                            git checkout master 
                            git pull origin master
                        '''
                            //           
                            //git merge -X oursorigin/develop --no-commit --no-ff 
                            //git add -u
                            //git commit -m "Merged develop into master"
                            //git push -u https://${GIT_TOKEN}@github.com/ccuevasbalinas/unir_devops_practica_aws.git master
                    }
                }
            }
        }

    }
}