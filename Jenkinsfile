pipeline {
    agent any
    environment {
        REST_ENDPOINT = 'https://pkc-ldvr1.asia-southeast1.gcp.confluent.cloud:443'
        API_KEY = credentials('BASE64_API_KEY')
        CLUSTER_ID = 'lkc-yjvgnk'
    }
    stages {
        stage('List Topic'){
            steps{
                script{
                    def listResult = sh(
                        script: '''
                            RESPONSE=$(curl -s -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics")
                            echo "$RESPONSE" | jq '.data'
                        ''',
                        returnStdout: true
                    ).trim()
                    
                    echo "${listResult}"
                    writeFile file: 'list_result.txt', text: listResult
                    archiveArtifacts artifacts: 'list_result.txt'
                }
            }
        }
    }
}