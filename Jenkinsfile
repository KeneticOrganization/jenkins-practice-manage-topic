properties([
    parameters([
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HTML', 
            description: '', 
            filterLength: 1, 
            filterable: false, 
            name: 'ParamsAsENV',
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
                        return "<input name="value" alt="Use parameterize to be environment." json="Use parameterize to be environment." type="checkbox" class=" ">"
                        '''
                ]
            ]
        ],
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HTML', 
            description: '', 
            filterLength: 1, 
            filterable: false, 
            name: 'ENVIRONMENT_PARAMS',
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
                        if (ParamsAsENV == 'List'){
                            return "<label></label>"
                        } else{
                            return """
                                <table><tr>
                                <td><label>Rest API Endpoint</label><input name='value' type='text' value=''></td>
                                <td><label>Cluster ID</label><input name='value' type='number' value=''></td>
                                </tr></table>
                            """
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
                    echo 'https://pkc-ldvr1.asia-southeast1.gcp.confluent.cloud:443' //REST_ENDPOINT
                    echo 'lkc-yjvgnk' //CLUSTER_ID
                    
                    echo "${ENVIRONMENT_PARAMS}"

                }
            }
        }
        stage('List Topic'){
            steps{
                script{
                    def listResult = sh(
                        script: '''
                            RESPONSE=$(curl -s -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics")
                            echo "$RESPONSE" | jq '.data'
                        ''',
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