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
        CC_API_KEY = credentials('BASE64_API_KEY')
        CP_API_KEY = credentials('CP_BASE64_API_KEY')
    }
    parameters {
        string(name: 'TopicName', defaultValue: 'default-topic', description: 'String')
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
                            echo env.Auth
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
                            echo env.Auth
                        }
                        else if(props.CONNECTION_TYPE == 'Platform'){
                            env.Auth = env.Auth + " -H \"Authorization: Basic \$CP_API_KEY\""
                        }
                    }
                }
            }
        }
        stage('Describe Topic'){
            steps{
                script{
                    def describeResult = sh(
                            script:"""
                            if curl -s ${env.Auth} --request GET --url "${env.REST_ENDPOINT}/v3/clusters/${env.CLUSTER_ID}/topics" | grep -c "\\"topic_name\\":\\"${params.TopicName}\\"" ; then
                                RESPONSE=\$(curl ${env.Auth} --request GET --url "${env.REST_ENDPOINT}/v3/clusters/${env.CLUSTER_ID}/topics/${params.TopicName}")
                                echo "\$RESPONSE" | jq '.'
                            else
                                echo "Unknown topic \"${params.TopicName}\"."
                            fi
                        """,
                        returnStdout: true
                    ).trim()
                    
                    writeFile file: 'describe_result.txt', text: describeResult
                    archiveArtifacts artifacts: 'describe_result.txt'
                }
            }
        }
    }
}