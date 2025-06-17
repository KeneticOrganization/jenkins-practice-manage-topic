pipeline {
    agent any
    environment {
        REST_ENDPOINT = 'https://pkc-ldvr1.asia-southeast1.gcp.confluent.cloud:443'
        API_KEY = credentials('BASE64_API_KEY')
        CLUSTER_ID = 'lkc-yjvgnk'
    }
    stages {
        stage('Creating Topic Testing'){
            steps{
                script{
                    build job: 'Jenkins Practice/jenkins-practice-manage-topic/create-topic-Jenkins', parameters: [
                        string(name: 'TopicName', value: 'test-topic'), 
                        string(name: 'Partitions', value: '6'), 
                        string(name: 'CleanupPolicy', value: 'Compact'), 
                        string(name: 'RetentionTime', value: '604800000'), 
                        string(name: 'RetentionSize', value: '-1'), 
                        string(name: 'MaxMessageBytes', value: '2097164')
                    ]
                }
            }
        }

        stage('Describe Topic Testing'){
            steps{
                script{
                    build job: 'Jenkins Practice/jenkins-practice-manage-topic/describe-topic', parameters: [
                        string(name: 'TopicName', value: 'test-topic')
                    ]
                }
            }
        }
    }
}