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
        REST_ENDPOINT = 'https://pkc-ldvr1.asia-southeast1.gcp.confluent.cloud:443'
        API_KEY = credentials('BASE64_API_KEY')
        CLUSTER_ID = 'lkc-yjvgnk'
    }
    parameters {
        string(name: 'TopicName', defaultValue: 'default-topic', description: 'String')
    }
    stages {
        stage('Delete Topic'){
            steps{
                script{
                    echo """
Topic Name : ${params.TopicName}
                    """
                    def deleteResult = sh (
                        script: """
                            if curl -H "Authorization: Basic \$API_KEY" --request GET --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics" | grep -c "\\"topic_name\\":\\"${params.TopicName}\\"" ; then
                                curl -H "Authorization: Basic \$API_KEY" --request DELETE --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics/${params.TopicName}"
                                
                                echo "Successfully deleted topic \\"${params.TopicName}\\"."
                            else
                                echo "Topic \\"${params.TopicName}\\" not found. Cannot delete."
                            fi
                        """,
                        returnStdout: true
                    ).trim()
                    
                    writeFile file: 'delete_result.txt', text: deleteResult
                    archiveArtifacts artifacts: 'delete_result.txt'
                }
            }
        }
    }
}