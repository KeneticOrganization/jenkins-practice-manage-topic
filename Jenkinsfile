properties([
    parameters([
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HIDDEN_HTML', 
            description: '', 
            omitValueField: false, 
            name: 'ParamsAsENV',
            referencedParameters: 'ParamsAsENV',
            script: [
                $class: 'GroovyScript', 
                fallbackScript: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''return['LIST_TOPIC:ERROR']'''
                ], 
                script: [
                    classpath: [], 
                    sandbox: true, 
                    script: '''
                            return "<input type='checkbox' name='value' value='true'/> Use parameterized environment."
                            '''
                ]
            ]
        ],
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HIDDEN_HTML', 
            description: '', 
            omitValueField: false, 
            name: 'ENVIRONMENT_PARAMS',
            referencedParameters: 'ParamsAsENV',
            script: [
                $class: 'GroovyScript', 
                fallbackScript: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''return['LIST_TOPIC:ERROR']'''
                ], 
                script: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''
                        if (ParamsAsENV == 'true'){
                            return """
                                <table><tr>
                                <td><label>Rest API Endpoint : </label><input name='value' type='text' value=''></td>
                                <td><label>Cluster ID : </label><input name='value' type='text' value=''></td>
                                <td><label>Connection Type : </label>
                                <select name='value'>
                                    <option value='Cloud'>Confluent Cloud</option>
                                    <option value='Platform'>Confluent Platform</option>
                                </select></td>
                                </tr></table>
                            """
                        } else{
                            return "<label></label>"
                        }
                        '''
                ]
            ]
        ]
    ])
])
pipeline {
    agent any
    environment {
        CC_API_KEY = credentials('BASE64_API_KEY')
        CP_API_KEY = credentials('CP_BASE64_API_KEY')
    }
    parameters {
        string(name: 'TopicName', defaultValue: 'default-topic', description: 'Topic name(s) - use comma to separate multiple topics')
        string(name: 'Partitions', defaultValue: '6', description: 'Integer')
        choice(name: 'CleanupPolicy', choices: [
            'Compact', 'Delete', 'Compact & Delete'
            ], description: '')
        string(name: 'RetentionTime', defaultValue: '604800000', description: 'Milli seconds')
        string(name: 'RetentionSize', defaultValue: '-1', description: 'Bytes')
        string(name: 'MaxMessageBytes', defaultValue: '2097164', description: 'Bytes')
    }
    stages {
        stage('Setup Environment') {
            steps{
                script{
                    def UseParamsAsENV = "${ParamsAsENV}".split(',').collect { it.trim() }.findAll { it }
                    def env_params = "${ENVIRONMENT_PARAMS}".split(',').collect { it.trim() }.findAll { it }
                    
                    def props = null
                    if (UseParamsAsENV[0] != 'true') {
                        props = readProperties file: 'env.properties'
                    }
                    else if (env_params[2] != 'Cloud'){
                        env_params[0] = env_params[0].replaceAll(";", ",")
                        env_params[2] = env_params[2].replaceAll(";", ",")
                    }

                    if (UseParamsAsENV[0] == 'true'){
                        if (env_params[2] == 'Platform,KafkaTools') {
                            env.BOOTSTRAP_SERVER = env_params[0]
                            env.KAFKA_TOOLS_PATH = env_params[1]
                        }
                        else {
                            env.REST_ENDPOINT = env_params[0]
                            env.CLUSTER_ID = env_params[1]
                        }
                    } else  {
                        if (props.CONNECTION_TYPE == 'Platform,KafkaTools') {
                            env.BOOTSTRAP_SERVER = props.BOOTSTRAP_SERVER
                            env.KAFKA_TOOLS_PATH = props.KAFKA_TOOLS_PATH
                        }
                        else {
                            env.REST_ENDPOINT = props.REST_ENDPOINT
                            env.CLUSTER_ID = props.CLUSTER_ID
                        }
                    }
                    env.Auth = ""
                    env.Sort = "| jq '.'"
                    if(env_params[2] == 'Cloud' || props?.CONNECTION_TYPE == 'Cloud'){
                        env.REST_ENDPOINT = env.REST_ENDPOINT + '/kafka'
                        env.Auth = env.Auth + " -H \"Authorization: Basic \$CC_API_KEY\""
                    }
                    else if (env_params[2] == 'Platform,RestAPI' || props?.CONNECTION_TYPE == 'Platform,RestAPI'){
                        env.Auth = env.Auth + " -H \"Authorization: Basic \$CP_API_KEY\""
                    }

                    // Cleanup Policy setup
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
                    
                    // Store cleanup policy for use in topic creation
                    env.CLEANUP_POLICY = cleanPolicy
                    
                    echo """
Topic Name(s) : ${params.TopicName}
Partition : ${params.Partitions}
Cleanup Policy : ${cleanPolicy}
Retention Time (ms) : ${params.RetentionTime}
Retention Size (bytes) : ${params.RetentionSize}
Max Message Bytes (bytes) : ${params.MaxMessageBytes}
                    """
                }
            }
        }
        stage('Create Topics'){
            steps{
                script{
                    // Split topic names by comma and trim whitespace
                    def topicNames = params.TopicName.split(',').collect { it.trim() }.findAll { it }
                    def allResults = []
                    
                    echo "Found ${topicNames.size()} topic(s) to create: ${topicNames.join(', ')}"
                    
                    // Loop through each topic name
                    for (String topicName : topicNames) {
                        echo "Processing topic: ${topicName}"
                        
                        // Set up commands for current topic
                        def hasTopicCommand = ""
                        def createCommand = ""
                        
                        if (env.BOOTSTRAP_SERVER && env.KAFKA_TOOLS_PATH) {
                            // Platform KafkaTools approach
                            hasTopicCommand = "${env.KAFKA_TOOLS_PATH}/bin/kafka-topics.sh --bootstrap-server ${env.BOOTSTRAP_SERVER} --list --command-config ${env.KAFKA_TOOLS_PATH}/config/kafka-config.properties | grep -xq \"${topicName}\""
                            createCommand = """${env.KAFKA_TOOLS_PATH}/bin/kafka-topics.sh --bootstrap-server ${env.BOOTSTRAP_SERVER} --command-config ${env.KAFKA_TOOLS_PATH}/config/kafka-config.properties \
                                        --create \
                                        --topic ${topicName} \
                                        --partitions ${params.Partitions} \
                                        --replication-factor 1 \
                                        --config cleanup.policy=${env.CLEANUP_POLICY} \
                                        --config retention.ms=${params.RetentionTime} \
                                        --config retention.bytes=${params.RetentionSize} \
                                        --config max.message.bytes=${params.MaxMessageBytes}
                            """
                        } else {
                            // REST API approach
                            hasTopicCommand = "curl -s ${env.Auth} --request GET --url \"${env.REST_ENDPOINT}/v3/clusters/${env.CLUSTER_ID}/topics\" | grep -c \"\\\"topic_name\\\":\\\"${topicName}\\\"\""
                            createCommand = """
                            curl -s ${env.Auth} -H 'Content-Type: application/json' --request POST --url "${env.REST_ENDPOINT}/v3/clusters/${env.CLUSTER_ID}/topics" \
                                -d "{
                                    \\"topic_name\\":\\"${topicName}\\",
                                    \\"partitions_count\\":\\"${params.Partitions}\\",
                                    \\"configs\\": [
                                        { \\"name\\": \\"cleanup.policy\\", \\"value\\": \\"${env.CLEANUP_POLICY}\\" },
                                        { \\"name\\": \\"retention.ms\\", \\"value\\": ${params.RetentionTime} },
                                        { \\"name\\": \\"retention.bytes\\", \\"value\\": ${params.RetentionSize} },
                                        { \\"name\\": \\"max.message.bytes\\", \\"value\\": ${params.MaxMessageBytes} }
                                    ]
                                }"
                            """
                        }
                        
                        // Execute topic creation for current topic
                        def createResult = sh(
                            script: """
                                if ! ${hasTopicCommand} ; then
                                    echo "Creating topic: ${topicName}"
                                    ${createCommand}
                                    
                                    echo "Successfully created topic name \"${topicName}\"."
                                else
                                    echo "Already has topic name \"${topicName}\"."
                                fi
                            """,
                            returnStdout: true
                        ).trim()
                        
                        allResults.add("=== Topic: ${topicName} ===\n${createResult}")
                        echo "Completed processing topic: ${topicName}"
                    }
                    
                    // Write all results to file
                    def finalResult = allResults.join('\n\n')
                    writeFile file: 'create_result.txt', text: finalResult
                    archiveArtifacts artifacts: 'create_result.txt'
                    
                    echo "All topics processing completed. Results archived in create_result.txt"
                }
            }
        }
    }
}