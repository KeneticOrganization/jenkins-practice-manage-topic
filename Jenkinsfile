pipeline {
    agent any
    environment {
        REST_ENDPOINT = 'https://pkc-ldvr1.asia-southeast1.gcp.confluent.cloud:443'
        API_KEY = credentials('BASE64_API_KEY')
        CLUSTER_ID = 'lkc-yjvgnk'
    }
    parameters {
        string(name: 'TopicName', defaultValue: 'default-topic', description: 'String')
        string(name: 'Partitions', defaultValue: '6', description: 'Integer')
        choice(name: 'CleanupPolicy', choices: [
            'Compact', 'Delete', 'Compact & Delete'
            ], description: '')
        string(name: 'RetentionTime', defaultValue: '604800000', description: 'Milli seconds')
        string(name: 'RetentionSize', defaultValue: '-1', description: 'Bytes')
        string(name: 'MaxMessageBytes', defaultValue: '2097164', description: 'Bytes')
    }
    stages {
        stage('Create Topic'){
            steps{
                script{
                    def cleanPolicy = ""
                    if (params.CleanupPolicy == "Compact") {
                        cleanPolicy = "compact"
                    } 
                    else if (params.CleanupPolicy == "Delete"){
                        cleanPolicy = "delete"
                    }
                    else if (params.CleanupPolicy == "Compact & Delete") {
                        cleanPolicy = "compact,delete"
                    }
                    sh("""
                        if ! curl -H "Authorization: Basic \$API_KEY" --request GET --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics" | grep -c "${values[0]}" ; then
                            curl -H "Authorization: Basic \$API_KEY" -H 'Content-Type: application/json' --request POST --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics" \
                            -d "{
                                \\"topic_name\\":\\"${params.TopicName}\\",
                                \\"partitions_count\\":\\"${params.Partitions}\\",
                                \\"configs\\": [
                                    { \\"name\\": \\"cleanup.policy\\", \\"value\\": \\"${cleanPolicy}\\" },
                                    { \\"name\\": \\"retention.ms\\", \\"value\\": ${RetentionTime} },
                                    { \\"name\\": \\"retention.bytes\\", \\"value\\": ${RetentionSize} },
                                    { \\"name\\": \\"max.message.bytes\\", \\"value\\": ${MaxMessageBytes} }
                                ]
                            }"
                            
                            echo "Successful created topic name \"${params.TopicName}\"."
                        else
                            echo "Already has topic name \"${params.TopicName}\"."
                        fi
                    """)
                }
            }
        }
    }
}