properties([
    parameters([
        [$class: 'ChoiceParameter', 
            choiceType: 'PT_SINGLE_SELECT', 
            description: 'What do you want to do?', 
            filterLength: 1, 
            filterable: false, 
            name: 'Action',
            script: [
                $class: 'GroovyScript', 
                fallbackScript: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''return['MANAGE_TOPIC:ERROR']'''
                ], 
                script: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''return["Create","Update","Describe","List:selected","Delete"]'''
                ]
            ]
        ], 
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HTML', 
            description: '', 
            name: 'Option', 
            omitValueField: false, 
            referencedParameters: 'Action',
            script: [
                $class: 'GroovyScript', 
                fallbackScript: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''return['MANAGE_TOPIC:ERROR']'''
                ], 
                $class: 'GroovyScript', 
                script: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''
                        if (Action == 'List'){
                            return "<label>This action didn't need any opions.</label>"
                        } else if (Action == 'Create') {
                            return """
                                <table><tr>
                                <td><label>Topic Name</label><input name='value' type='text' value='default-topic'></td>
                                <td><label>Partitions</label><input name='value' type='number' value='6'></td>
                                <td><label>Cleanup Policy</label>
                                <select name='value'>
                                    <option value='compact'>Compact</option>
                                    <option value='compact,delete'>Compact & Delete</option>
                                    <option value='delete' selected>Delete</option>
                                </select></td>
                                <td><label>Retention Time (ms)</label><input name='value' type='number' value='604800000'></td>
                                <td><label>Retention Size (bytes)</label><input name='value' type='number' value='-1'></td>
                                <td><label>Max Message Bytes (bytes)</label><input name='value' type='number' value='2097164'></td>
                                </tr></table>
                            """
                        } else if (Action == 'Update') {
                            return """
                                <table><tr>
                                <td><label>Topic Name</label><input name='value' type='text' value='default-topic'></td>
                                <td><label>Cleanup Policy</label>
                                <select name='value'>
                                    <option value='compact'>Compact</option>
                                    <option value='compact,delete'>Compact & Delete</option>
                                    <option value='delete' selected>Delete</option>
                                </select></td>
                                <td><label>Retention Time (ms)</label><input name='value' type='number' value='604800000'></td>
                                <td><label>Retention Size (bytes)</label><input name='value' type='number' value='-1'></td>
                                <td><label>Max Message Bytes (bytes)</label><input name='value' type='number' value='2097164'></td>
                                </tr></table>
                            """
                        } else if (Action == 'MANAGE_TOPIC:ERROR') {
                            return['MANAGE_TOPIC:ERROR']
                        } else {
                            return """
                                <table><tr>
                                <td><label>Topic Name : </label><input name='value' type='text' value='default-topic'></td>
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
        REST_ENDPOINT = 'https://pkc-ldvr1.asia-southeast1.gcp.confluent.cloud:443' // Replace with your actual REST endpoint
        API_KEY = credentials('BASE64_API_KEY')
        CLUSTER_ID = 'lkc-yjvgnk' // replace with your cluster 
    }
    stages {
        stage('Confirmation'){
            when{
                expression {return params.Action == 'Delete'}
            }
            agent none
            steps{
                script {
                    def option = "${Option}"
                    def values = option.split(',').collect { it.trim() }.findAll { it }
                    
                    def CONFIRM_NAME = input(
                        message: "Type the topic name to confirm deletion: '${values[0]}'",
                        parameters: [
                            string(defaultValue: '', description: 'Re-type the topic name exactly to confirm', name: 'CONFIRM_NAME')
                        ],
                        ok: "Confirm",
                        cancel: "Cancel"
                    )
                    
                    def confirmation = true
        
                    if (CONFIRM_NAME != values[0]) {
                        confirmation = false
                    }
                    
                    env.confirmation = confirmation
                }
            }
        }
        stage('Topic') {
            parallel{
                stage('Create'){
                    when{
                        expression {return params.Action == 'Create'}
                    }
                    steps{
                        script{
                            def option = "${Option}"
                            def values = option.split(',').collect { it.trim() }.findAll { it }
                            if (values[3] == 'delete') {
                                values[2] = "${values[2]},${values[3]}"
                                
                                values[3] = values[4]
                                values[4] = values[5]
                                values[5] = values[6]
                                
                                values = values.take(6)
                            }
                            echo """
Topic Name : ${values[0]}
Partition : ${values[1]}
Cleanup Policy : ${values[2]}
Retention Time (ms) : ${values[3]}
Retention Size (bytes) : ${values[4]}
Max Message Bytes (bytes) : ${values[5]}
                            """
                            sh("""
                                if ! curl -H "Authorization: Basic \$API_KEY" --request GET --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics" | grep -c "${values[0]}" ; then
                                    curl -H "Authorization: Basic \$API_KEY" -H 'Content-Type: application/json' --request POST --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics" \
                                    -d "{
                                        \\"topic_name\\":\\"${values[0]}\\",
                                        \\"partitions_count\\":\\"${values[1]}\\",
                                        \\"configs\\": [
                                            { \\"name\\": \\"cleanup.policy\\", \\"value\\": \\"${values[2]}\\" },
                                            { \\"name\\": \\"retention.ms\\", \\"value\\": ${values[3]} },
                                            { \\"name\\": \\"retention.bytes\\", \\"value\\": ${values[4]} },
                                            { \\"name\\": \\"max.message.bytes\\", \\"value\\": ${values[5]} }
                                        ]
                                    }"
                                    
                                    echo "Successful created topic name \"${values[0]}\"."
                                else
                                    echo "Already has topic name \"${values[0]}\"."
                                fi
                            """)
                        }
                    }
                }
                stage('Update'){
                    when{
                        expression {return params.Action == 'Update'}
                    }
                    steps{
                        script{
                            def option = "${Option}"
                            def values = option.split(',').collect { it.trim() }.findAll { it }
                            if (values[2] == 'delete') {
                                values[1] = "${values[1]},${values[2]}"
                                
                                values[2] = values[3]
                                values[3] = values[4]
                                values[4] = values[5]
                                
                                values = values.take(5)
                            }
                            echo """
Topic Name : ${values[0]}
Cleanup Policy : ${values[1]}
Retention Time (ms) : ${values[2]}
Retention Size (bytes) : ${values[3]}
Max Message Bytes (bytes) : ${values[4]}
                            """
                            def updateJson = """{
                                \\"${values[0]}\\": {
                                \\"retention.ms\\": ${values[2]},
                                \\"retention.bytes\\": ${values[3]},
                                \\"max.message.bytes\\": ${values[4]},
                                \\"cleanup.policy\\": \\"${values[1]}\\"
                                }
                            }"""

                            sh("""
                                echo "${updateJson}" | jq -r 'to_entries[] | "\\(.key) \\(.value | to_entries[] )"' | while read topic data; do
                                    property=\$(echo \$data | jq -r '.key')
                                    valueJson=\$(echo \$data | jq -r '.value')
                                    
                                    curl -H "Authorization: Basic \$API_KEY" -H 'Content-Type: application/json' --request PUT \\
                                        --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics/\$topic/configs/\$property" \\
                                        -d "{\\\"value\\\": \\\"\$valueJson\\\"}"
                                done
                            """)
                        }
                    }
                }
                stage('Describe'){
                    when{
                        expression {return params.Action == 'Describe'}
                    }
                    steps{
                        script{
                            def option = "${Option}"
                            def values = option.split(',').collect { it.trim() }.findAll { it }
                            echo """
Topic Name : ${values[0]}
                            """
                            sh ("""
                                if curl -H "Authorization: Basic \${API_KEY}" --request GET --url "\${REST_ENDPOINT}/kafka/v3/clusters/\${CLUSTER_ID}/topics" | grep -c "${values[0]}" ; then
                                    RESPONSE=\$(curl -H "Authorization: Basic \${API_KEY}" --request GET --url "\${REST_ENDPOINT}/kafka/v3/clusters/\${CLUSTER_ID}/topics/${values[0]}")
                                    echo "\$RESPONSE" | jq '.data'
                                else
                                    echo "Unknown topic \"\$TOPIC_NAME\"."
                                fi
                            """)
                        }
                    }
                }
                stage('List'){
                    when{
                        expression {return params.Action == 'List'}
                    }
                    steps{
                        script{
                            def responseJson = sh(
                                script: '''
                                    RESPONSE=$(curl -s -H "Authorization: Basic $API_KEY" --request GET --url "$REST_ENDPOINT/kafka/v3/clusters/$CLUSTER_ID/topics")
                                    echo "$RESPONSE" | jq '.data'
                                ''',
                                returnStdout: true
                            ).trim()
                            
                            echo "${responseJson}"
                        }
                    }
                }
                stage('Delete'){
                    when{
                        expression {return params.Action == 'Delete'}
                    }
                    steps{
                        script{
                            def option = "${Option}"
                            def values = option.split(',').collect { it.trim() }.findAll { it }
                            echo env.confirmation
                            echo """
Topic Name : ${values[0]}
                            """
                            if (env.confirmation){
                                sh ("""
                                    if curl -H "Authorization: Basic \$API_KEY" --request GET --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics" | grep -c "${values[0]}" ; then
                                        curl -H "Authorization: Basic \$API_KEY" --request DELETE --url "\$REST_ENDPOINT/kafka/v3/clusters/\$CLUSTER_ID/topics/${values[0]}"
                                        
                                        echo "Successfully deleted topic \\"${values[0]}\\"."
                                    else
                                        echo "Topic \\"${values[0]}\\" not found. Cannot delete."
                                    fi
                                """)
                            }
                        }
                    }
                }
            }
        }
    }
}