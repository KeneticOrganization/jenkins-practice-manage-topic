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
        stage('Delete Topic'){
            steps{
                script{
                    echo """
Topic Name : ${params.TopicName}
                    """
                    sh ("""
                        if curl -H "Authorization: Basic \$API_KEY" --request GET --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics" | grep -c "${params.TopicName}" ; then
                            curl -H "Authorization: Basic \$API_KEY" --request DELETE --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics/${params.TopicName}"
                            
                            echo "Successfully deleted topic \\"${params.TopicName}\\"."
                        else
                            echo "Topic \\"${params.TopicName}\\" not found. Cannot delete."
                        fi
                    """)
                }
            }
        }
    }
}