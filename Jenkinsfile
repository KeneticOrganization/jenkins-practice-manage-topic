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
                        '''return['DELETE_TOPIC:ERROR']'''
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
                        '''return['DELETE_TOPIC:ERROR']'''
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
                    
                    echo """
Topic Name(s) : ${params.TopicName}
                    """
                }
            }
        }
        stage('Delete Topics'){
            steps{
                script{
                    // Split topic names by comma and trim whitespace
                    def topicNames = params.TopicName.split(',').collect { it.trim() }.findAll { it }
                    def allResults = []
                    
                    echo "Found ${topicNames.size()} topic(s) to delete: ${topicNames.join(', ')}"
                    
                    // Loop through each topic name
                    for (String topicName : topicNames) {
                        echo "Processing topic: ${topicName}"
                        
                        // Set up commands for current topic
                        def hasTopicCommand = ""
                        def deleteCommand = ""
                        
                        if (env.BOOTSTRAP_SERVER && env.KAFKA_TOOLS_PATH) {
                            // Platform KafkaTools approach
                            hasTopicCommand = "${env.KAFKA_TOOLS_PATH}/bin/kafka-topics.sh --bootstrap-server ${env.BOOTSTRAP_SERVER} --list --command-config ${env.KAFKA_TOOLS_PATH}/config/kafka-config.properties | grep -xq \"${topicName}\""
                            deleteCommand = "${env.KAFKA_TOOLS_PATH}/bin/kafka-topics.sh --bootstrap-server ${env.BOOTSTRAP_SERVER} --delete --topic ${topicName} --command-config ${env.KAFKA_TOOLS_PATH}/config/kafka-config.properties"
                        } else {
                            // REST API approach
                            hasTopicCommand = "curl -s ${env.Auth} --request GET --url \"${env.REST_ENDPOINT}/v3/clusters/${env.CLUSTER_ID}/topics\" | grep -c \"\\\"topic_name\\\":\\\"${topicName}\\\"\""
                            deleteCommand = "curl -s ${env.Auth} --request DELETE --url \"${env.REST_ENDPOINT}/v3/clusters/${env.CLUSTER_ID}/topics/${topicName}\""
                        }
                        
                        // Execute topic deletion for current topic
                        def deleteResult = sh(
                            script: """
                                if ${hasTopicCommand} ; then
                                    echo "Deleting topic: ${topicName}"
                                    ${deleteCommand}
                                    
                                    echo "Successfully deleted topic \\"${topicName}\\"."
                                else
                                    echo "Topic \\"${topicName}\\" not found. Cannot delete."
                                fi
                            """,
                            returnStdout: true
                        ).trim()
                        
                        allResults.add("=== Topic: ${topicName} ===\n${deleteResult}")
                        echo "Completed processing topic: ${topicName}"
                    }
                    
                    // Write all results to file
                    def finalResult = allResults.join('\n\n')
                    writeFile file: 'delete_result.txt', text: finalResult
                    archiveArtifacts artifacts: 'delete_result.txt'
                    
                    echo "All topics processing completed. Results archived in delete_result.txt"
                }
            }
        }
    }
}