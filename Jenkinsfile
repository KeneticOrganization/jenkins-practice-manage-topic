pipeline {
    agent any
    environment {
        REST_ENDPOINT = 'https://pkc-ldvr1.asia-southeast1.gcp.confluent.cloud:443'
        API_KEY = credentials('BASE64_API_KEY')
        CLUSTER_ID = 'lkc-yjvgnk'
    }
    parameters {
        string(name: 'TopicName', defaultValue: 'default-topic', description: 'String')
        choice(name: 'CleanupPolicy', choices: [
            'Compact', 'Delete'
            ], description: '')
        string(name: 'RetentionTime', defaultValue: '604800000', description: 'Milli seconds')
        string(name: 'RetentionSize', defaultValue: '-1', description: 'Bytes')
        string(name: 'MaxMessageBytes', defaultValue: '2097164', description: 'Bytes')
    }
    stages {
        stage('Update Topic'){
            steps{
                script{
                    def cleanPolicy = ""
                    if (params.CleanupPolicy == "Compact") {
                        cleanPolicy = "compact"
                    } 
                    else if (params.CleanupPolicy == "Delete"){
                        cleanPolicy = "delete"
                    }
                    echo """
Topic Name : ${params.TopicName}
Cleanup Policy : ${cleanPolicy}
Retention Time (ms) : ${params.RetentionTime}
Retention Size (bytes) : ${params.RetentionSize}
Max Message Bytes (bytes) : ${params.MaxMessageBytes}
                    """
                    def updateJson = """{
                        \\"${params.TopicName}\\": {
                        \\"retention.ms\\": ${params.RetentionTime},
                        \\"retention.bytes\\": ${params.RetentionSize},
                        \\"max.message.bytes\\": ${params.MaxMessageBytes},
                        \\"cleanup.policy\\": \\"${cleanPolicy}\\"
                        }
                    }"""

                    sh("""
                        echo "${updateJson}" | jq -r 'to_entries[] | "\\(.key) \\(.value | to_entries[] )"' | while read topic data; do
                            property=\$(echo \$data | jq -r '.key')
                            valueJson=\$(echo \$data | jq -r '.value')
                            
                            curl -H "Authorization: Basic \$API_KEY" -H 'Content-Type: application/json' --request PUT \\
                                --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics/\$topic/configs/\$property" \\
                                -d "{\\\"value\\\": \\\"\$valueJson\\\"}"
                        done
                    """)
                }
            }
        }
    }
}