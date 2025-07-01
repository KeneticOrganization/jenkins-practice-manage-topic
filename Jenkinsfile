properties([
    parameters([
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HTML', 
            description: '', 
            omitValueField: false, 
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
                            return "<input type='checkbox' name='value' value='true'/> Use parameterized environment."
                            '''
                ]
            ]
        ],
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HTML', 
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
                                <td><label>Bootstrap Server : </label><input name='value' type='text' value=''></td>
                                <td><label>Kafka Tools Path : </label><input name='value' type='text' value=''></td>
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
                        env.BOOTSTRAP_SERVER = env_params[2]
                        env.KAFKA_TOOLS_PATH = env_params[3]
                    } else  {
                        def props = readProperties file: 'env.properties'
                        env.REST_ENDPOINT = props.REST_ENDPOINT
                        env.CLUSTER_ID = props.CLUSTER_ID
                        env.BOOTSTRAP_SERVER = props.BOOTSTRAP_SERVER
                        env.KAFKA_TOOLS_PATH = props.KAFKA_TOOLS_PATH
                    }
                }
            }
        }
        stage('List Topic'){
            steps{
                script{
                    def listResult = sh(
                        script: """
                            ${KAFKA_TOOLS_PATH}/bin/kafka-topics.sh --bootstrap-server ${BOOTSTRAP_SERVER} --list --command-config /opt/kafka/config/kafka-config.properties | grep -v '^_'
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