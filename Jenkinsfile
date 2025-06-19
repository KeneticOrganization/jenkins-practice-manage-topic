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
                                    <option value='Compact'>Compact</option>
                                    <option value='Compact & Delete'>Compact & Delete</option>
                                    <option value='Delete' selected>Delete</option>
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
                                    <option value='Compact'>Compact</option>
                                    <option value='Delete' selected>Delete</option>
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
                            def createResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/create-topic-Jenkins', parameters: [
                                string(name: 'TopicName', value: "${values[0]}"), 
                                string(name: 'Partitions', value: "${values[1]}"), 
                                string(name: 'CleanupPolicy', value: "${values[2]}"), 
                                string(name: 'RetentionTime', value: "${values[3]}"), 
                                string(name: 'RetentionSize', value: "${values[4]}"), 
                                string(name: 'MaxMessageBytes', value: "${values[5]}")
                            ]

                            copyArtifacts(projectName: createResult.projectName, selector: specific("${createResult.number}"), filter: 'create_result.txt')

                            def output = readFile('create_result.txt').trim()
                            echo "Creating output: ${output}"

                            generateJUnitXML('create-topic', output.contains('Success') || output.contains('created'), 'Create Topic', output)
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
                            def updateResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/update-topic', parameters: [
                                string(name: 'TopicName', value: "${values[0]}"), 
                                string(name: 'CleanupPolicy', value: "${values[1]}"), 
                                string(name: 'RetentionTime', value: "${values[2]}"), 
                                string(name: 'RetentionSize', value: "${values[3]}"), 
                                string(name: 'MaxMessageBytes', value: "${values[4]}")
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
                        expression {return params.Action == 'Describe'}
                    }
                    steps{
                        script{
                            def option = "${Option}"
                            def values = option.split(',').collect { it.trim() }.findAll { it }
                            echo """
Topic Name : ${values[0]}
                            """
                            def describeResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/describe-topic', parameters: [
                                string(name: 'TopicName', value: "${values[0]}")
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
                        expression {return params.Action == 'List'}
                    }
                    steps{
                        script{
                            def listResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/list-topic'

                            copyArtifacts(projectName: listResult.projectName, selector: specific("${listResult.number}"), filter: 'list_result.txt')

                            def output = readFile('list_result.txt').trim()
                            echo "List output: ${output}"

                            generateJUnitXML('list-topic', output.contains("${values[0]}"), 'List Topic', output)
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
                                def deleteResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/delete-topic', parameters: [
                                    string(name: 'TopicName', value: "${values[0]}")
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