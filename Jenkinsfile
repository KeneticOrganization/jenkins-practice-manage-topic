properties([
    parameters([
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HIDDEN_HTML', 
            description: '', 
            omitValueField: true, 
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
                            return "<input type='checkbox' name='value' value='true'>"
                            '''
                ]
            ]
        ],
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HIDDEN_HTML', 
            description: '', 
            omitValueField: true, 
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
        CC_SCHEMA_API_KEY = credentials('BASE64_SCHEMA_API_KEY')
        CP_SCHEMA_API_KEY = credentials('CP_BASE64_SCHEMA_API_KEY')
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
                        env_params[1] = env_params[1].replaceAll(";", ",")
                    }

                    if (UseParamsAsENV[0] == 'true'){
                        if (env_params[1] == 'Platform,KafkaTools') {
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
                    env.Sort = "| jq ."
                    if(env_params[1] == 'Cloud' || props?.CONNECTION_TYPE == 'Cloud'){
                        env.Auth = env.Auth + " -H \"Authorization: Basic \$CC_SCHEMA_API_KEY\""
                        echo env.Auth
                    }
                    else if (env_params[1] == 'Platform,RestAPI' || props?.CONNECTION_TYPE == 'Platform,RestAPI'){
                        env.Auth = env.Auth + " -H \"Authorization: Basic \$CP_SCHEMA_API_KEY\""
                    }
                    env.Command = "curl -s ${env.Auth} --request GET \"${env.REST_ENDPOINT}/schemas\""
                    if (env_params[1] == 'Platform,KafkaTools' || props?.CONNECTION_TYPE == 'Platform,KafkaTools'){
                        env.Command = "${KAFKA_TOOLS_PATH}/bin/kafka-topics.sh --bootstrap-server ${BOOTSTRAP_SERVER} --list --command-config ${KAFKA_TOOLS_PATH}/config/kafka-config.properties"
                    }
                }
            }
        }
        stage('List Topic'){
            steps{
                script{
                    def listResult = sh(
                        script: """
                            RESPONSE=\$(${env.Command})
                            echo "\$RESPONSE ${env.Sort}"
                        """,
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