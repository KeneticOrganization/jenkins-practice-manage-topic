pipeline {
    agent any
    environment {
        REST_ENDPOINT = 'https://pkc-ldvr1.asia-southeast1.gcp.confluent.cloud:443' // Replace with your actual REST endpoint
        API_KEY = credentials('BASE64_API_KEY')
        CLUSTER_ID = 'lkc-yjvgnk' // replace with your cluster 
    }
    parameters {
        string(name: 'topic-name', defaultValue: 'default-name', description: 'Name of a topic that will create or delete in kafka.')
      
        choice(name: 'create-or-delete-topic', choices: ['Create', 'Delete', 'None'], description: 'Choose to create or delete topic.')
      
        choice(name: 'describe-topic', choices: ['Before', 'After', 'Both', 'None'], description: 'Choose when to show details of a specific Kafka topic: before, after, or both before and after creation/deletion.')
      
        choice(name: 'list-topic', choices: ['Before', 'After', 'Both', 'None'], description: 'Choose when to show details of a specific Kafka topic: before, after, or both before and after creation/deletion.')
            
        choice(name: 'config-topic', choices: ['Before', 'After', 'Both', 'None'], description: 'Choose when to show details of a specific Kafka topic: before, after, or both before and after creation/deletion.')
      
        text(name: 'update-topic', defaultValue: '''{
  "default-name": {
    "retention.ms": 259200000,
    "cleanup.policy": "delete"
  }
}''', description: 'JSON config to update multiple Kafka topics.')
    }
    stages {
        stage('Set Parameters') {
            steps {
                script {
                    env.TOPIC_NAME = params['topic-name']
                    env.CREATE_DELETE = params['create-or-delete-topic']
                    env.DESCRIBE = params['describe-topic']
                    env.LIST = params['list-topic']
                    env.CONFIG = params['config-topic']
                    env.UPDATE = params['update-topic']
                }
            }
        }
        
        stage('Start'){
            steps {
                sh '''
                    { set +x; } 2>/dev/null
                    if [ $LIST = "Before" ] || [ $LIST = "Both" ]; then
                        { RESPONSE=$(curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics"); } 2>/dev/null
                        
                        { DATA=$(echo "$RESPONSE" | jq '.data'); } 2>/dev/null
                        if [ "$DATA" = "[]" ]; then
                          echo "none"
                        else
                          echo "$DATA"
                        fi
                    fi
                    
                    if [ $DESCRIBE = "Before" ] || [ $DESCRIBE = "Both" ]; then
                        if curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics" 2>/dev/null | grep -c "$TOPIC_NAME" 2>/dev/null ; then
                            { RESPONSE=$(curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics/$TOPIC_NAME"); } 2>/dev/null
                            echo $RESPONSE | jq '.'
                        else
                            echo "Unknown topic \"$TOPIC_NAME\"."
                        fi
                    fi
                    
                    if [ $CONFIG = "Before" ] || [ $CONFIG = "Both" ]; then
                        if curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics" 2>/dev/null | grep -c "$TOPIC_NAME" 2>/dev/null ; then
                            echo $UPDATE | jq -r 'to_entries[] | "\\(.key) \\(.value | to_entries[] )"' | while read topic data; do
                                property=$(echo $data | jq -r '.key')
                                { RESPONSE=$(curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics/$TOPIC_NAME/configs/$property"); } 2>/dev/null
                                echo $RESPONSE | jq '.'
                            done
                        else
                            echo "Unknown topic \"$TOPIC_NAME\"."
                        fi
                    fi
                '''
            }
        }
        
        stage('Creating/Deleting Topic'){
            steps {
                sh '''
                    { set +x; } 2>/dev/null
                    if [ $CREATE_DELETE = "Create" ]; then
                        if ! curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics" 2>/dev/null | grep -c "$TOPIC_NAME" 2>/dev/null ; then
                            curl -H "Authorization: Basic $API_KEY" -H 'Content-Type: application/json' --request POST --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics" -d "{\\"topic_name\\":\\"$TOPIC_NAME\\"}" 2>/dev/null
                            
                            echo "Successful created topic name \"$TOPIC_NAME\""
                        else
                            echo "Already has topic name \"$TOPIC_NAME\"."
                        fi
                    elif [ $CREATE_DELETE = "Delete" ]; then
                        if curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics" 2>/dev/null | grep -c "$TOPIC_NAME" 2>/dev/null ; then
                            curl -H "Authorization: Basic $API_KEY" -H 'Content-Type: application/json' --request DELETE  --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics/$TOPIC_NAME" 2>/dev/null
                            
                            echo "Successful deleted topic name \"$TOPIC_NAME\""
                        else
                            echo "Unknown topic \"$TOPIC_NAME\"."
                        fi
                    fi
                '''
            }
        }
        
        stage('Update Topic') {
            steps {
                sh '''
                    { set +x; } 2>/dev/null
                    echo $UPDATE | jq -r 'to_entries[] | "\\(.key) \\(.value | to_entries[] )"' | while read topic data; do
                        property=$(echo $data | jq -r '.key')
                        valueJson=$(echo $data | jq -r '.value')
                        
                        curl -H "Authorization: Basic $API_KEY" -H 'Content-Type: application/json' --request PUT \
                        --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics/$topic/configs/$property" \
                        -d "{\\"value\\": \\"$valueJson\\"}"
                    done
                '''
            }
        }
        
        stage('Final'){
            steps {
                sh '''
                    { set +x; } 2>/dev/null
                    if [ $LIST = "After" ] || [ $LIST = "Both" ]; then
                        { RESPONSE=$(curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics"); } 2>/dev/null
                        
                        { DATA=$(echo "$RESPONSE" | jq '.data'); } 2>/dev/null
                        if [ "$DATA" = "[]" ]; then
                          echo "none"
                        else
                          echo "$DATA"
                        fi
                    fi
                    
                    if [ $DESCRIBE = "After" ] || [ $DESCRIBE = "Both" ]; then
                        if curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics" 2>/dev/null | grep -c "$TOPIC_NAME" 2>/dev/null ; then
                            { RESPONSE=$(curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics/$TOPIC_NAME"); } 2>/dev/null
                            echo $RESPONSE | jq '.'
                        else
                            echo "Unknown topic \"$TOPIC_NAME\"."
                        fi
                    fi
                    
                    if [ $CONFIG = "After" ] || [ $CONFIG = "Both" ]; then
                        if curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics" 2>/dev/null | grep -c "$TOPIC_NAME" 2>/dev/null ; then
                            { RESPONSE=$(curl -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics/$TOPIC_NAME"/configs/); } 2>/dev/null
                            echo $RESPONSE | jq '.data'
                        else
                            echo "Unknown topic \"$TOPIC_NAME\"."
                        fi
                    fi
                '''
            }
        }
    }
}
