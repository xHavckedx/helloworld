pipeline {
    agent{
        label 'linux'
    } 
    
    stages{
        stage('Get Code'){
            steps{
                // Obtener el código del repo
                git branch: 'feature_fix_racecond', url: 'https://github.com/xHavckedx/helloworld'
            }
        }
        stage('Build'){
            steps{
                echo 'Eyyy, esto es Python. No hay que compilar nada!!'
                echo WORKSPACE
                sh 'ls -la'
            }
        }
        stage('Unit'){
            steps{
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    sh '''
                        export PYTHONPATH=.
                        pytest --junitxml=result-unit.xml test/unit
                    '''   
                }
            }
        }
        stage('Rest'){
            steps{
                sh '''
                    export PYTHONPATH=.
                    export FLASK_APP=app/api.py
                    flask run &
                    java -jar /wiremock-standalone-3.5.4.jar  --port 9090 --root-dir /var/jenkins_home/workspace/P24/test1/test/wiremock &
                    sleep 2
                    pytest --junitxml=result-rest.xml test/rest
                '''
            }
        }
        stage('Results'){
            steps{
                junit 'result*.xml'
                echo 'Finish!!'
            }
        }
    }
}