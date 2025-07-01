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
                        '''return['UPDATE_TOPIC:ERROR']'''
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
                        '''return['UPDATE_TOPIC:ERROR']'''
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
        string(name: 'TopicName', defaultValue: 'default-topic', description: 'String')
        choice(name: 'CleanupPolicy', choices: [
            'Compact', 'Delete'
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
                    echo UseParamsAsENV[0]
                    
                    if (UseParamsAsENV[0] == 'true'){
                        def env_params = "${ENVIRONMENT_PARAMS}".split(',').collect { it.trim() }.findAll { it }
                        env.REST_ENDPOINT = env_params[0]
                        env.CLUSTER_ID = env_params[1]
                        env.Auth = ""
                        if(env_params[2] == 'Cloud'){
                            env.REST_ENDPOINT = env.REST_ENDPOINT + '/kafka'
                            env.Auth = env.Auth + " -H \"Authorization: Basic \$CC_API_KEY\""
                        }
                        else if(env_params[2] == 'Platform'){
                            env.Auth = env.Auth + " -H \"Authorization: Basic \$CP_API_KEY\""
                        }
                    } else  {
                        def props = readProperties file: 'env.properties'
                        env.REST_ENDPOINT = props.REST_ENDPOINT
                        env.CLUSTER_ID = props.CLUSTER_ID
                        env.Auth = ""
                        if(props.CONNECTION_TYPE == 'Cloud'){
                            env.REST_ENDPOINT = env.REST_ENDPOINT + '/kafka'
                            env.Auth = env.Auth + " -H \"Authorization: Basic \$CC_API_KEY\""
                        }
                        else if(props.CONNECTION_TYPE == 'Platform'){
                            env.Auth = env.Auth + " -H \"Authorization: Basic \$CP_API_KEY\""
                        }
                    }
                }
            }
        }
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

                    def updateResult = sh(
                        script: """
                            # First check if topic exists
                            if curl -s ${Auth} --request GET --url "${REST_ENDPOINT}/v3/clusters/${CLUSTER_ID}/topics" | grep -c "\\"topic_name\\":\\"${params.TopicName}\\"" ; then
                                echo "${updateJson}" | jq -r 'to_entries[] | "\\(.key) \\(.value | to_entries[] )"' | while read topic data; do
                                    property=\$(echo \$data | jq -r '.key')
                                    valueJson=\$(echo \$data | jq -r '.value')
                                    
                                    curl -s ${Auth} -H 'Content-Type: application/json' --request PUT \\
                                        --url "${REST_ENDPOINT}/v3/clusters/${CLUSTER_ID}/topics/\$topic/configs/\$property" \\
                                        -d "{\\\"value\\\": \\\"\$valueJson\\\"}"
                                done
                                echo "Successfully update topic '${params.TopicName}'"
                            else
                                echo "Topic '${params.TopicName}' not found. Cannot update."
                            fi
                        """,
                        returnStdout: true
                    ).trim()
                    
                    echo "${updateResult}"
                    writeFile file: 'update_result.txt', text: updateResult
                    archiveArtifacts artifacts: 'update_result.txt'
                }
            }
        }
    }
}