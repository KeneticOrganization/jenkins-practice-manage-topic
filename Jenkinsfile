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
    parameters {
        string(name: 'TopicName', defaultValue: 'default-topic', description: 'String')
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
        stage('Create Topic'){
            steps{
                script{
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
                    echo """
Topic Name : ${params.TopicName}
Partition : ${params.Partitions}
Cleanup Policy : ${cleanPolicy}
Retention Time (ms) : ${params.RetentionTime}
Retention Size (bytes) : ${params.RetentionSize}
Max Message Bytes (bytes) : ${params.MaxMessageBytes}
                    """
                    def createResult = sh(
                        script: """
                            if ! ${KAFKA_TOOLS_PATH}/bin/kafka-topics.sh --bootstrap-server ${BOOTSTRAP_SERVER} --list | grep -xq "${params.TopicName}" ; then
                                ${KAFKA_TOOLS_PATH}/bin/kafka-topics.sh --bootstrap-server ${BOOTSTRAP_SERVER} \
                                    --create \
                                    --topic ${params.TopicName} \
                                    --partitions ${params.Partitions} \
                                    --replication-factor 1

                                ${KAFKA_TOOLS_PATH}/bin/kafka-configs.sh --bootstrap-server ${BOOTSTRAP_SERVER} \
                                    --entity-type topics \
                                    --entity-name ${params.TopicName} \
                                    --alter \
                                    --add-config cleanup.policy=${cleanPolicy},retention.ms=${params.RetentionTime},retention.bytes=${params.RetentionSize},max.message.bytes=${params.MaxMessageBytes}
                                
                                echo "\nSuccessful created topic name \"${params.TopicName}\"."
                            else
                                echo "Already has topic name \"${params.TopicName}\"."
                            fi
                        """,
                        returnStdout: true
                    ).trim()

                    writeFile file: 'create_result.txt', text: createResult
                    archiveArtifacts artifacts: 'create_result.txt'
                }
            }
        }
    }
}