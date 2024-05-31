pipeline {
    agent any

    environment {
        JMETER_HOME = "/opt/apache-jmeter-5.6.3"
        PATH = "${JMETER_HOME}/bin:${env.PATH}"
        COVERAGE_THRESHOLD_UNSTABLE = 80
        COVERAGE_THRESHOLD_STABLE = 90
    }

    stages{
        stage('Get Code'){
            agent { label 'linux' }
            steps{
                //Comprobar identidad de quien está ejecutando que.
                sh ''' 
                    whoami
                    hostname
                    hostname -i
                '''
                //limpiar el workspace
                cleanWs()
                // Obtener el código del repo
                git 'https://github.com/xHavckedx/helloworld'
                stash includes: 'test/**/*', name: 'tests'
                stash includes: 'app/**/*', name: 'code'
            }
        }
        stage('Parallel Stages') {
            parallel {
                stage('Unit'){
                    agent { label 'linux' }
                    steps{
                        //Comprobar identidad de quien está ejecutando que.
                        sh ''' 
                            whoami
                            hostname
                            hostname -i
                        '''
                        unstash 'tests'
                        unstash 'code'
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            sh '''
                                export PYTHONPATH=.
                                pytest --junitxml=result-unit.xml test/unit
                            '''
                            junit 'result*.xml'
                            echo 'Finish!!'   
                        }
                    }
                }
                stage('Rest'){
                    agent { label 'linux' }
                    steps{
                        //Comprobar identidad de quien está ejecutando que.
                        sh ''' 
                            whoami
                            hostname
                            hostname -i
                        '''
                        unstash 'tests'
                        unstash 'code'
                        sh '''
                            export PYTHONPATH=.
                            export FLASK_APP=app/api.py
                            flask run &
                            java -jar /wiremock-standalone-3.6.0.jar  --port 9090 --root-dir /root/workspace/2024-folder/pipeline/test/wiremock &
                            sleep 4
                            pytest --junitxml=result-rest.xml test/rest
                        '''
                        junit 'result*.xml'
                    }
                }
                stage('Static'){
                    agent { label 'linux' }
                    steps{
                        //Comprobar identidad de quien está ejecutando que.
                        sh ''' 
                            whoami
                            hostname
                            hostname -i
                        '''
                        unstash 'code'
                        sh '''
                            flake8 --exit-zero --format=pylint app >flake8.out
                        '''
                        recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold:8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
                        //flake8
                        // 8 o mas hallazgos etapa UNSTABLE
                        // 10 o mas hallazgos UNHEALTHY
                        // Da igual el resultado siempre debe continuar la ejecución.
                    }
                }
                stage('Security Test'){
                    agent { label 'linux' }
                    steps{
                        //Comprobar identidad de quien está ejecutando que.
                        sh ''' 
                            whoami
                            hostname
                            hostname -i
                        '''
                        unstash 'code'
                        sh '''
                            bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                        '''
                        recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold:2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                        //bandit
                        // Si encuentra 2 hallazgos UNSTABLE
                        // Si encuentra 4 hallazgos UNHEALTHY
                        // Da igual el resultado siempre debe continuar la ejecución
                    }
                }
                stage('Performance'){
                    agent { label 'linux' }
                    steps{
                        //Comprobar identidad de quien está ejecutando que.
                        sh ''' 
                            whoami
                            hostname
                            hostname -i
                        '''
                        unstash 'tests'
                        unstash 'code'
                        sh '''
                            export PYTHONPATH=.
                            export FLASK_APP=app/api.py
                            flask run &
                            jmeter -n -t test/jmeter/flask2.jmx -f -l flask.jtl
                        '''
                        perfReport sourceDataFiles: 'flask.jtl'
                        //jmeter
                        // la prueba tiene que incluir un test-plan 5 hilos realicen 40 llamadas  al micro suma y otros 40 a resta
                        // flask tiene que estar levantado y ejecutandose. Wiremock no es necesario
                        // test-plan tiene que ejecutarse en la etapa performance y llamar al plugin para visualizar los resultados
                        // indicar cual sería el valor de la linea 90 para el micro suma, así como captura gráfica en la que puede apreciarse este valor.
                    }
                }
                stage('Coverage'){
                    agent { label 'linux' }
                    steps{
                        //Comprobar identidad de quien está ejecutando que.
                        sh ''' 
                            whoami
                            hostname
                            hostname -i
                        '''
                        unstash 'code'
                        unstash 'tests'
                        sh '''
                            export PYTHONPATH=.
                            coverage run --branch --source=app --omit=app/init.py,app/api.py -m pytest test/unit
                            coverage xml -o coverage.xml
                        '''
                        cobertura coberturaReportFile:'coverage.xml', onlyStable: false, conditionalCoverageTargets: '95,90,70', lineCoverageTargets: '95,90,70'
                        //coverage
                        // si está entre 80 y 90 UNSTABLE
                        // si está por encima de 90 verde por debajo de 80 rojo
                        // sea cual sea el resultado hay que continuar la ejecución
                    }
                }
            }
        }
    }
    post {
        always {
            //limpiar el workspace
            cleanWs()
        }
    }
}