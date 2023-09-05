pipeline {
    
    agent {
        label 'agent01'
    }

    triggers {
        cron '0 10 * * 5'
    }
    
    environment {
        SENSU_PSQL = credentials('psql-sensu-crawler')
        default_elastic = credentials('DB-default-elastic')
        k8s_portscan = credentials('DB-kubernetes-portscan')
        production_staging = credentials('DB-production-staging')
    }

    stages {

        stage('Remove metrics older than 3 months - default') {
            steps{
                    sh '''
                        timestamp=$(date +%s)
                        timestampMinus3months="$((timestamp - 7776000))"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'DELETE FROM $default_elastic_USR WHERE time_stamp < $timestampMinus3months;'"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'VACUUM FULL $default_elastic_USR;'"
                    '''
            }
        }
        stage('Remove metrics older than 3 months - elasticsearch-prod') {
            steps{
                    sh '''
                        timestamp=$(date +%s)
                        timestampMinus3months="$((timestamp - 7776000))"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'DELETE FROM $default_elastic_PSW WHERE time_stamp < $timestampMinus3months;'"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'VACUUM FULL $default_elastic_PSW;'"
                    '''
            }
        }
        stage('Remove metrics older than 3 months - kubernetes-prod') {
            steps{
                    sh '''
                        timestamp=$(date +%s)
                        timestampMinus3months="$((timestamp - 7776000))"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'DELETE FROM $k8s_portscan_USR WHERE time_stamp < $timestampMinus3months;'"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'VACUUM FULL $k8s_portscan_USR;'"
                    '''
            }
        }
        stage('Remove metrics older than 3 months - port-scan') {
            steps{
                    sh '''
                        timestamp=$(date +%s)
                        timestampMinus3months="$((timestamp - 7776000))"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'DELETE FROM $k8s_portscan_PSW WHERE time_stamp < $timestampMinus3months;'"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'VACUUM FULL $k8s_portscan_PSW;'"
                    '''
            }
        }
        stage('Remove metrics older than 3 months - production') {
            steps{
                    sh '''
                        timestamp=$(date +%s)
                        timestampMinus3months="$((timestamp - 7776000))"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'DELETE FROM $production_staging_USR WHERE time_stamp < $timestampMinus3months;'"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'VACUUM FULL $production_staging_USR;'"
                    '''
            }
        }
        stage('Remove metrics older than 3 months - staging') {
            steps{
                    sh '''
                        timestamp=$(date +%s)
                        timestampMinus3months="$((timestamp - 7776000))"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'DELETE FROM $production_staging_PSW WHERE time_stamp < $timestampMinus3months;'"
                        ssh anyuser@slowik1.com "$SENSU_PSQL_PSW psql -U $SENSU_PSQL_USR -h localhost -p 5432 -c 'VACUUM FULL $production_staging_PSW;'"
                    '''
            }
        } 
    }
    post {
    success {
        mattermostSend(color: 'good', icon: "https://jenkins.io/images/logos/jenkins/jenkins.png", message: "${env.JOB_NAME}", text: "Retention-policy job for your server finished successfully! Check ${BUILD_URL}", channel: 'log' , endpoint: 'https://chat.slowik1.com/hooks/4oiuar539ej9h')
        }
    failure {
        mattermostSend(color: 'danger', icon: "https://jenkins.io/images/logos/jenkins/jenkins.png", message: "${env.JOB_NAME}", text: "Retention-policy job for your server job failed! Check ${BUILD_URL}", channel: 'log' , endpoint: 'https://chat.slowik1.com/hooks/4oiuar559ej9h')     
        }
    }
}
