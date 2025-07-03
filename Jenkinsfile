properties([
    parameters([
        [$class: 'ChoiceParameter', 
            choiceType: 'PT_SINGLE_SELECT', 
            description: 'What do you want to do?', 
            filterLength: 1, 
            filterable: false, 
            name: 'TopicAction',
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
            name: 'Amount', 
            omitValueField: false, 
            referencedParameters: 'TopicAction',
            script: [
                $class: 'GroovyScript', 
                fallbackScript: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''return['MANAGE_TOPIC:ERROR CODE 1']'''
                ], 
                $class: 'GroovyScript', 
                script: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''
                        if (TopicAction == 'List'){
                            return """
                            <label>This action didn't have amount of topic setting.</label>
                            """
                        } else if (TopicAction == 'Create' || TopicAction == 'Update' || TopicAction == 'Delete') {
                            return """
                                <div style="margin-bottom: 15px;">
                                    <input type="number" name="value" min="1" value="1">
                                </div>
                            """
                        } else if (TopicAction == 'MANAGE_TOPIC:ERROR') {
                            return['MANAGE_TOPIC:ERROR CODE 0']
                        } else {
                            return """
                                <label>This action didn't have amount of topic setting.</label>
                            """
                        }
                        '''
                ]
            ]
        ], 
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HTML', 
            description: '', 
            name: 'Option', 
            omitValueField: false, 
            referencedParameters: 'TopicAction,Amount',
            script: [
                $class: 'GroovyScript', 
                fallbackScript: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''return['MANAGE_TOPIC:ERROR CODE 2']'''
                ], 
                $class: 'GroovyScript', 
                script: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''
                        def values = Amount?.split(',')?.collect { it.trim() }?.findAll { it } ?: ['1']
                        def count = values[0].isInteger() ? values[0].toInteger() : 1
                        if (TopicAction == 'Create') {
                            def html = ""
                            for (int i = 0; i < count; i++) {
                                html += """
                                    <div style="margin-bottom: 10px;">
                                        <label for="option_${i}">Topic Name ${i + 1}:</label>
                                        <input type="text" id="option_${i}" name="value" value="topic-${i + 1}" style="width: 300px;" />
                                    </div>
                                """
                            }
                            html += """
                                <table><tr>
                                <td><label>Partitions</label><input name='value' type='number' value='6'></td>
                                <td><label>Cleanup Policy</label>
                                <select name='value'>
                                    <option value='Compact'>Compact</option>
                                    <option value='Compact & Delete'>Compact & Delete</option>
                                    <option value='Delete' selected>Delete</option>
                                </select></td>
                                <td><label>Retention Time (ms)</label><input name='value' type='number' value='604800000'></td>
                                <td><label>Retention Size (bytes)</label><input name='value' type='number' value='-1'></td>
                                <td><label>Max Message Bytes (bytes)</label><input name='value' type='number' value='2097164'></td>
                                </tr></table>
                            """
                            return html
                        } else if (TopicAction == 'Update') {
                            def html = ""
                            for (int i = 0; i < count; i++) {
                                html += """
                                    <div style="margin-bottom: 10px;">
                                        <label for="option_${i}">Topic Name ${i + 1}:</label>
                                        <input type="text" id="option_${i}" name="value" value="topic-${i + 1}" style="width: 300px;" />
                                    </div>
                                """
                            }
                            html += """
                                <table><tr>
                                <td><label>Cleanup Policy</label>
                                <select name='value'>
                                    <option value='Compact'>Compact</option>
                                    <option value='Compact & Delete'>Compact & Delete</option>
                                    <option value='Delete' selected>Delete</option>
                                </select></td>
                                <td><label>Retention Time (ms)</label><input name='value' type='number' value='604800000'></td>
                                <td><label>Retention Size (bytes)</label><input name='value' type='number' value='-1'></td>
                                <td><label>Max Message Bytes (bytes)</label><input name='value' type='number' value='2097164'></td>
                                </tr></table>
                            """
                            return html
                        } else if (TopicAction == 'Delete') {
                            def html = ""
                            for (int i = 0; i < count; i++) {
                                html += """
                                    <div style="margin-bottom: 10px;">
                                        <label for="option_${i}">Topic Name ${i + 1}:</label>
                                        <input type="text" id="option_${i}" name="value" value="topic-${i + 1}" style="width: 300px;" />
                                    </div>
                                """
                            }
                            return html
                        } else {
                            return """
                                <label>This action didn't need any options.</label>
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
    stages {
        stage('Setup Environment') {
            steps{
                script{
                    def props = readProperties file: 'env.properties'
                    if(props.CONNECTION_TYPE == 'Platform,KafkaTools'){
                        env.params_1 = props.BOOTSTRAP_SERVER.replaceAll(",", ";")
                        env.params_2 = props.KAFKA_TOOLS_PATH
                    }
                    else{
                        env.params_1 = props.REST_ENDPOINT
                        env.params_2 = props.CLUSTER_ID
                    }
                    env.CONNECTION_TYPE = props.CONNECTION_TYPE.replaceAll(",", ";")
                }
            }
        }
        stage('Confirmation'){
            when{
                expression {return params.TopicAction == 'Delete'}
            }
            agent none
            steps{
                script {
                    def values = "${Option}".split(',').collect { it.trim() }.findAll { it }

                    def countStr = "${Amount}".split(',').collect { it.trim() }.findAll { it }
                    def count = countStr[0].isInteger() ? countStr[0].toInteger() : 1

                    if (values.size() < count) {
                        error "Number of topic names (${values.size()}) is less than expected (${count})"
                    }

                    def confirmation = true
                    for (int i = 0; i < count; i++) {
                        def topicName = values[i]
                        def CONFIRM_NAME = input(
                            message: "Type the topic name to confirm deletion: '${topicName}'",
                            parameters: [
                                string(defaultValue: '', description: "Re-type '${topicName}' exactly to confirm", name: 'CONFIRM_NAME')
                            ],
                            ok: "Confirm",
                            cancel: "Cancel"
                        )

                        if (CONFIRM_NAME != topicName) {
                            confirmation = false
                            break
                        }
                    }

                    env.confirmation = confirmation
                }
            }
        }
        stage('Topic') {
            parallel{
                stage('Create'){
                    when{
                        expression {return params.TopicAction == 'Create'}
                    }
                    steps{
                        script{
                            def values = "${Option}".split(',').collect { it.trim() }.findAll { it }

                            def countStr = "${Amount}".split(',').collect { it.trim() }.findAll { it }
                            def count = countStr[0].isInteger() ? countStr[0].toInteger() : 1

                            //If count = 2 then 
                            // 0 is Topic Name
                            // 1 is Topic Name
                            // 2 is Partition
                            // 3 is CleanupPolicy
                            // 4 is RetentionTime
                            // 5 is RetentionSize
                            // 6 is MaxMessageBytes

                            if (values[count + 2] == 'delete') {
                                values[count + 1] = "${values[count + 1]},${values[count + 2]}"
                                
                                values[count + 2] = values[count + 3]
                                values[count + 3] = values[count + 4]
                                values[count + 4] = values[count + 5]
                                
                                values = values.take(count + 5)
                            }
                            def output = ""
                            def multipleTopicName = values[0..(count-1)].join(',')

                            def createResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/create-topic-multiple', parameters: [
                                string(name: 'TopicName', value: "${multipleTopicName}"), 
                                string(name: 'Partitions', value: "${values[count]}"), 
                                string(name: 'CleanupPolicy', value: "${values[count+1]}"), 
                                string(name: 'RetentionTime', value: "${values[count+2]}"), 
                                string(name: 'RetentionSize', value: "${values[count+3]}"), 
                                string(name: 'MaxMessageBytes', value: "${values[count+4]}"),
                                string(name: 'ParamsAsENV', value: 'true,'),
                                string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${params_2},${CONNECTION_TYPE},")
                            ]

                            copyArtifacts(projectName: createResult.projectName, selector: specific("${createResult.number}"), filter: 'create_result.txt')

                            def outputFile = readFile('create_result.txt').trim()
                            output += "${outputFile}\n"
                            echo "Creating output: ${outputFile}"

                            generateJUnitXML('create-topic', output.contains('Success') || output.contains('created'), 'Create Topic', output)
                        }
                    }
                }
                stage('Update'){
                    when{
                        expression {return params.TopicAction == 'Update'}
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
                            def updateResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/update-topic', parameters: [
                                string(name: 'TopicName', value: "${values[0]}"), 
                                string(name: 'CleanupPolicy', value: "${values[1]}"), 
                                string(name: 'RetentionTime', value: "${values[2]}"), 
                                string(name: 'RetentionSize', value: "${values[3]}"), 
                                string(name: 'MaxMessageBytes', value: "${values[4]}"),
                                string(name: 'ParamsAsENV', value: 'true,'),
                                string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${params_2},${CONNECTION_TYPE},")
                            ]

                            copyArtifacts(projectName: updateResult.projectName, selector: specific("${updateResult.number}"), filter: 'update_result.txt')

                            def output = readFile('update_result.txt').trim()
                            echo "Update output: ${output}"

                            generateJUnitXML('update-topic', output.contains('Success') || output.contains('updated'), 'Update Topic', output)
                        }
                    }
                }
                stage('Describe'){
                    when{
                        expression {return params.TopicAction == 'Describe'}
                    }
                    steps{
                        script{
                            def option = "${Option}"
                            def values = option.split(',').collect { it.trim() }.findAll { it }
                            echo """
Topic Name : ${values[0]}
                            """
                            def describeResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/describe-topic', parameters: [
                                string(name: 'TopicName', value: "${values[0]}"),
                                string(name: 'ParamsAsENV', value: 'true,'),
                                string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${params_2},${CONNECTION_TYPE},")
                            ]

                            copyArtifacts(projectName: describeResult.projectName, selector: specific("${describeResult.number}"), filter: 'describe_result.txt')

                            def output = readFile('describe_result.txt').trim()
                            echo "Describe output: ${output}"

                            generateJUnitXML('describe-topic', output.contains("${values[0]}"), 'Describe Topic', output)
                        }
                    }
                }
                stage('List'){
                    when{
                        expression {return params.TopicAction == 'List'}
                    }
                    steps{
                        script{
                            def listResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/list-topic', parameters: [
                                string(name: 'ParamsAsENV', value: 'true,'),
                                string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${params_2},${CONNECTION_TYPE},")
                            ]

                            copyArtifacts(projectName: listResult.projectName, selector: specific("${listResult.number}"), filter: 'list_result.txt')

                            def output = readFile('list_result.txt').trim()
                            echo "List output: ${output}"

                            generateJUnitXML('list-topic', !output.toLowerCase().contains('error'), 'List Topic', output)
                        }
                    }
                }
                stage('Delete'){
                    when{
                        expression {return params.TopicAction == 'Delete'}
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
                                def deleteResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/delete-topic', parameters: [
                                    string(name: 'TopicName', value: "${values[0]}"),
                                    string(name: 'ParamsAsENV', value: 'true,'),
                                    string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${params_2},${CONNECTION_TYPE},")
                                ]

                                copyArtifacts(projectName: deleteResult.projectName, selector: specific("${deleteResult.number}"), filter: 'delete_result.txt')

                                def output = readFile('delete_result.txt').trim()
                                echo "Delete output: ${output}"

                                generateJUnitXML('delete-topic', output.contains('Success') || output.contains('deleted'), 'Delete Topic', output)
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            junit testResults: 'test-results/*.xml', 
                  keepLongStdio: true,
                  allowEmptyResults: false
            
            archiveArtifacts artifacts: 'test-results/*.xml', allowEmptyArchive: true
            
            archiveArtifacts artifacts: '*_result.txt', allowEmptyArchive: true
        }
        
        success {
            echo 'All topic management tests passed successfully!'
        }
        
        failure {
            echo 'Some topic management tests failed. Check the test results for details.'
        }
    }
}

def generateJUnitXML(testName, passed, displayName, output) {
    def status = passed ? 'passed' : 'failed'
    def failureElement = passed ? '' : """
        <failure message="Test failed" type="AssertionError">
            <![CDATA[${output}]]>
        </failure>"""
    
    def xmlContent = """<?xml version="1.0" encoding="UTF-8"?>
<testsuite name="TopicManagementTests" tests="1" failures="${passed ? 0 : 1}" errors="0" time="1.0">
    <testcase name="${testName}" classname="TopicManagement" time="1.0">
        <system-out><![CDATA[${output}]]></system-out>${failureElement}
    </testcase>
</testsuite>"""
    
    // Create test-results directory if it doesn't exist
    sh 'mkdir -p test-results'
    
    // Write the XML file
    writeFile file: "test-results/${testName}.xml", text: xmlContent
}