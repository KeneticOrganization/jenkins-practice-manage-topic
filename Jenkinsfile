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
                        '''return['DESCRIBE_TOPIC:ERROR']'''
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
                        '''return['DESCRIBE_TOPIC:ERROR']'''
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
        CC_API_KEY = credentials('BASE64_SCHEMA_API_KEY')
        CP_API_KEY = credentials('CP_BASE64_SCHEMA_API_KEY')
    }
    parameters {
        string(name: 'SchemaID', defaultValue: '100003', description: 'Integer')
        string(name: 'SchemaVersion', defaultValue: 'latest', description: 'Schema Version (e.g., 1, 2, or latest)')
        string(name: 'Subject', defaultValue: 'default-subject', description: 'Schema Subject (required for versioned request)')
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
                            env.SCHEMA_REGISTRY_URL = env_params[0]
                            env.KAFKA_TOOLS_PATH = env_params[1]
                        }
                        else {
                            env.SCHEMA_REGISTRY_URL = env_params[0]
                            env.CLUSTER_ID = env_params[1]
                        }
                    } else  {
                        if (props.CONNECTION_TYPE == 'Platform,KafkaTools') {
                            env.SCHEMA_REGISTRY_URL = props.SCHEMA_REGISTRY_URL
                            env.KAFKA_TOOLS_PATH = props.KAFKA_TOOLS_PATH
                        }
                        else {
                            env.SCHEMA_REGISTRY_URL = props.SCHEMA_REGISTRY_URL
                            env.CLUSTER_ID = props.CLUSTER_ID
                        }
                    }
                    env.Auth = ""
                    env.Sort = "| jq -r '.schema' | jq ."
                    if(env_params[2] == 'Cloud' || props?.CONNECTION_TYPE == 'Cloud'){
                        env.Auth = env.Auth + " -H \"Authorization: Basic \$CC_API_KEY\""
                    }
                    else if (env_params[2] == 'Platform,RestAPI' || props?.CONNECTION_TYPE == 'Platform,RestAPI'){
                        env.Auth = env.Auth + " -H \"Authorization: Basic \$CP_API_KEY\""
                    }
                    env.HasTopic = "curl -s ${env.Auth} --request GET --url \"${env.SCHEMA_REGISTRY_URL}/schemas\" | grep -c \"\\\"id\\\":${params.SchemaID}\""
                    if (params.SchemaVersion?.trim() && params.SchemaVersion != 'latest') {
                        // Use specific version (e.g., 1, 2, etc.)
                        env.Command = "curl -s ${env.Auth} --request GET --url \"${env.SCHEMA_REGISTRY_URL}/subjects/${params.Subject}/versions/${params.SchemaVersion}\""
                    } else {
                        // Use latest version
                        env.Command = "curl -s ${env.Auth} --request GET --url \"${env.SCHEMA_REGISTRY_URL}/subjects/${params.Subject}/versions/latest\""
                    }
                    if (env_params[2] == 'Platform,KafkaTools' || props?.CONNECTION_TYPE == 'Platform,KafkaTools'){
                        env.Sort = ""
                        env.HasTopic = "${KAFKA_TOOLS_PATH}/bin/kafka-topics.sh --bootstrap-server ${BOOTSTRAP_SERVER} --list --command-config ${KAFKA_TOOLS_PATH}/config/kafka-config.properties | grep -xq \"${params.TopicName}\""
                        env.Command = "${KAFKA_TOOLS_PATH}/bin/kafka-topics.sh --bootstrap-server ${BOOTSTRAP_SERVER} --describe --topic ${params.SchemaID} --command-config ${KAFKA_TOOLS_PATH}/config/kafka-config.properties"
                    }
                }
            }
        }
        stage('Get Schema'){
            steps{
                script{
                    def getSchemaResult = sh(
                            script:"""
                            if ${env.HasTopic} ; then
                                RESPONSE=\$(${env.Command})
                                echo "\$RESPONSE" ${env.Sort}
                            else
                                echo "Unknown Schema ID \"${params.SchemaID}\"."
                            fi
                        """,
                        returnStdout: true
                    ).trim()

                    echo getSchemaResult
                    
                    writeFile file: 'get_schema_result.txt', text: getSchemaResult
                    archiveArtifacts artifacts: 'get_schema_result.txt'
                }
            }
        }
    }
}