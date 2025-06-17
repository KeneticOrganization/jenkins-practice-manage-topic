pipeline {
    agent any
    environment {
        REST_ENDPOINT = 'https://pkc-ldvr1.asia-southeast1.gcp.confluent.cloud:443'
        API_KEY = credentials('BASE64_API_KEY')
        CLUSTER_ID = 'lkc-yjvgnk'
    }
    parameters {
        string(name: 'TopicName', defaultValue: 'default-topic', description: 'String')
    }
    stages {
        stage('Describe Topic'){
            steps{
                steps{
                    script{
                        echo """
Topic Name : ${params.TopicName}
                        """
                        sh ("""
                            if curl -H "Authorization: Basic \${API_KEY}" --request GET --url "\${REST_ENDPOINT}/kafka/v3/clusters/\${CLUSTER_ID}/topics" | grep -c "${params.TopicName}" ; then
                                RESPONSE=\$(curl -H "Authorization: Basic \${API_KEY}" --request GET --url "\${REST_ENDPOINT}/kafka/v3/clusters/\${CLUSTER_ID}/topics/${params.TopicName}")
                                echo "\$RESPONSE" | jq '.data'
                            else
                                echo "Unknown topic \"${params.TopicName}\"."
                            fi
                        """)
                    }
                }
            }
        }
    }
}