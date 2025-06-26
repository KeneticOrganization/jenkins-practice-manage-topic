properties([
    parameters([
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HIDDEN_HTML', 
            description: '', 
            omitValueField: true, 
            name: 'ParamsAsENV',
            referencedParameters: '',
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
        API_KEY = credentials('BASE64_API_KEY')
    }
    /*
    parameters {
        string(name: 'REST_ENDPOINT', defaultValue: '', description: 'If you want to use env.properties then skip this.')
        string(name: 'CLUSTER_ID', defaultValue: '', description: 'If you want to use env.properties then skip this.')
    }
    */
    stages {
        stage('Setup Environment') {
            steps{
                script{
                    // echo 'https://pkc-ldvr1.asia-southeast1.gcp.confluent.cloud:443' //REST_ENDPOINT
                    // echo 'lkc-yjvgnk' //CLUSTER_ID
                    def UseParamsAsENV = "${ParamsAsENV}".split(',').collect { it.trim() }.findAll { it }
                    
                    if (UseParamsAsENV[0] == 'true'){
                        def env_params = "${ENVIRONMENT_PARAMS}".split(',').collect { it.trim() }.findAll { it }
                        env.REST_ENDPOINT = env_params[0]
                        env.CLUSTER_ID = env_params[1]
                        env.Auth = ""
                        env.Sort = "jq '.data"
                        if(env_params[2] == 'Cloud'){
                            env.REST_ENDPOINT = env.REST_ENDPOINT + '/kafka'
                            env.Auth = env.Auth + " -H \"Authorization: Basic \$API_KEY\""
                            echo env.Auth
                        }
                        else if (env_params[2] == 'Platform') {
                            env.Sort = env.Sort + " | map(select(.topic_name | startswith(\"_\") | not))"
                        }
                        env.Sort = env.Sort + "'"
                    } else  {
                        def props = readProperties file: 'env.properties'
                        env.REST_ENDPOINT = props.REST_ENDPOINT
                        env.CLUSTER_ID = props.CLUSTER_ID
                        env.Auth = ""
                        env.Sort = "jq '.data"
                        if(props.CONNECTION_TYPE == 'CLOUD'){
                            env.REST_ENDPOINT = env.REST_ENDPOINT + '/kafka'
                            env.Auth = env.Auth + " -H \"Authorization: Basic \$API_KEY\""
                            echo env.Auth
                        }
                        else if (props.CONNECTION_TYPE == 'PLATFORM') {
                            env.Sort = env.Sort + " | map(select(.topic_name | startswith(\"_\") | not))"
                        }
                        env.Sort = env.Sort + "'"
                    }
                }
            }
        }
        stage('List Topic'){
            steps{
                script{
                    def listResult = sh(
                        script: """
                            RESPONSE=\$(curl -s ${env.Auth} --request GET --url "${env.REST_ENDPOINT}/v3/clusters/${env.CLUSTER_ID}/topics")
                            echo "\$RESPONSE" | ${env.Sort}
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