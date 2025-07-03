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
                            return "<label>This action didn't need any options.</label>"
                        } else if (TopicAction == 'Create') {
                            return """
                                <div style="width: 630px; margin-bottom: 15px;">
                                    <img src="https://www.mfec.co.th/wp-content/uploads/2023/09/New-Logo-MFEC-More.-2023.jpg" style="max-width: 100%; height: auto;">
                                </div>
                                
                                <div style="margin-bottom: 15px;">
                                    <label style="font-weight: bold; color: #333;">Number of Topics to Create:</label>
                                    <input type="number" id="topicCount" min="1" value="1" onchange="generateTopicInputs()" style="padding: 5px; border: 1px solid #ccc; border-radius: 3px; width: 100px; margin-left: 10px;">
                                </div>
                                
                                <div id="topicInputsContainer"></div>
                                
                                <script>
                                    function generateTopicInputs() {
                                        const count = parseInt(document.getElementById('topicCount').value);
                                        const container = document.getElementById('topicInputsContainer');
                                        
                                        let html = '';
                                        for (let i = 1; i <= count; i++) {
                                            html += '<div style="border: 1px solid #ddd; padding: 15px; margin-bottom: 15px; border-radius: 5px; background: #fafafa;">';
                                            html += '<h4 style="margin: 0 0 15px 0; color: #333; border-bottom: 1px solid #ddd; padding-bottom: 5px;">Topic ' + i + '</h4>';
                                            html += '<table style="width: 100%; border-collapse: collapse;"><tr>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Topic Name</label><input name="value" type="text" value="default-topic-' + i + '" style="width: 120px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"></td>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Partitions</label><input name="value" type="number" value="6" style="width: 80px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"></td>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Cleanup Policy</label><select name="value" style="width: 130px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"><option value="Compact">Compact</option><option value="Compact & Delete">Compact & Delete</option><option value="Delete" selected>Delete</option></select></td>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Retention Time (ms)</label><input name="value" type="number" value="604800000" style="width: 120px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"></td>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Retention Size (bytes)</label><input name="value" type="number" value="-1" style="width: 120px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"></td>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Max Message Bytes</label><input name="value" type="number" value="2097164" style="width: 120px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"></td>';
                                            html += '</tr></table></div>';
                                        }
                                        container.innerHTML = html;
                                    }
                                    
                                    // Initialize with one topic
                                    generateTopicInputs();
                                </script>
                            """
                        } else if (TopicAction == 'Update') {
                            return """
                                <div style="width: 630px; margin-bottom: 15px;">
                                    <img src="https://www.mfec.co.th/wp-content/uploads/2023/09/New-Logo-MFEC-More.-2023.jpg" style="max-width: 100%; height: auto;">
                                </div>
                                
                                <div style="margin-bottom: 15px;">
                                    <label style="font-weight: bold; color: #333;">Number of Topics to Update:</label>
                                    <input type="number" id="topicCountUpdate" min="1" value="1" onchange="generateUpdateInputs()" style="padding: 5px; border: 1px solid #ccc; border-radius: 3px; width: 100px; margin-left: 10px;">
                                </div>
                                
                                <div id="topicUpdateContainer"></div>
                                
                                <script>
                                    function generateUpdateInputs() {
                                        const count = parseInt(document.getElementById('topicCountUpdate').value);
                                        const container = document.getElementById('topicUpdateContainer');
                                        
                                        let html = '';
                                        for (let i = 1; i <= count; i++) {
                                            html += '<div style="border: 1px solid #ddd; padding: 15px; margin-bottom: 15px; border-radius: 5px; background: #fafafa;">';
                                            html += '<h4 style="margin: 0 0 15px 0; color: #333; border-bottom: 1px solid #ddd; padding-bottom: 5px;">Topic ' + i + ' Update</h4>';
                                            html += '<table style="width: 100%; border-collapse: collapse;"><tr>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Topic Name</label><input name="value" type="text" value="default-topic-' + i + '" style="width: 120px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"></td>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Cleanup Policy</label><select name="value" style="width: 130px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"><option value="Compact">Compact</option><option value="Delete" selected>Delete</option></select></td>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Retention Time (ms)</label><input name="value" type="number" value="604800000" style="width: 120px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"></td>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Retention Size (bytes)</label><input name="value" type="number" value="-1" style="width: 120px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"></td>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Max Message Bytes</label><input name="value" type="number" value="2097164" style="width: 120px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"></td>';
                                            html += '</tr></table></div>';
                                        }
                                        container.innerHTML = html;
                                    }
                                    
                                    // Initialize with one topic
                                    generateUpdateInputs();
                                </script>
                            """
                        } else if (TopicAction == 'Delete') {
                            return """
                                <div style="width: 630px; margin-bottom: 15px;">
                                    <img src="https://www.mfec.co.th/wp-content/uploads/2023/09/New-Logo-MFEC-More.-2023.jpg" style="max-width: 100%; height: auto;">
                                </div>
                                
                                <div style="margin-bottom: 15px;">
                                    <label style="font-weight: bold; color: #333;">Number of Topics to Delete:</label>
                                    <input type="number" id="topicCountDelete" min="1" value="1" onchange="generateDeleteInputs()" style="padding: 5px; border: 1px solid #ccc; border-radius: 3px; width: 100px; margin-left: 10px;">
                                </div>
                                
                                <div id="topicDeleteContainer"></div>
                                
                                <script>
                                    function generateDeleteInputs() {
                                        const count = parseInt(document.getElementById('topicCountDelete').value);
                                        const container = document.getElementById('topicDeleteContainer');
                                        
                                        let html = '';
                                        for (let i = 1; i <= count; i++) {
                                            html += '<div style="border: 1px solid #ddd; padding: 15px; margin-bottom: 15px; border-radius: 5px; background: #fafafa;">';
                                            html += '<h4 style="margin: 0 0 15px 0; color: #333; border-bottom: 1px solid #ddd; padding-bottom: 5px;">Topic ' + i + ' to Delete</h4>';
                                            html += '<table style="width: 100%; border-collapse: collapse;"><tr>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Topic Name</label><input name="value" type="text" value="default-topic-' + i + '" style="width: 200px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"></td>';
                                            html += '</tr></table></div>';
                                        }
                                        container.innerHTML = html;
                                    }
                                    
                                    // Initialize with one topic
                                    generateDeleteInputs();
                                </script>
                            """
                        } else if (TopicAction == 'Describe') {
                            return """
                                <div style="width: 630px; margin-bottom: 15px;">
                                    <img src="https://www.mfec.co.th/wp-content/uploads/2023/09/New-Logo-MFEC-More.-2023.jpg" style="max-width: 100%; height: auto;">
                                </div>
                                
                                <div style="margin-bottom: 15px;">
                                    <label style="font-weight: bold; color: #333;">Number of Topics to Describe:</label>
                                    <input type="number" id="topicCountDescribe" min="1" value="1" onchange="generateDescribeInputs()" style="padding: 5px; border: 1px solid #ccc; border-radius: 3px; width: 100px; margin-left: 10px;">
                                </div>
                                
                                <div id="topicDescribeContainer"></div>
                                
                                <script>
                                    function generateDescribeInputs() {
                                        const count = parseInt(document.getElementById('topicCountDescribe').value);
                                        const container = document.getElementById('topicDescribeContainer');
                                        
                                        let html = '';
                                        for (let i = 1; i <= count; i++) {
                                            html += '<div style="border: 1px solid #ddd; padding: 15px; margin-bottom: 15px; border-radius: 5px; background: #fafafa;">';
                                            html += '<h4 style="margin: 0 0 15px 0; color: #333; border-bottom: 1px solid #ddd; padding-bottom: 5px;">Topic ' + i + ' to Describe</h4>';
                                            html += '<table style="width: 100%; border-collapse: collapse;"><tr>';
                                            html += '<td style="padding: 5px; vertical-align: top;"><label style="font-weight: bold;">Topic Name</label><input name="value" type="text" value="default-topic-' + i + '" style="width: 200px; padding: 5px; border: 1px solid #ccc; border-radius: 3px; margin-top: 3px;"></td>';
                                            html += '</tr></table></div>';
                                        }
                                        container.innerHTML = html;
                                    }
                                    
                                    // Initialize with one topic
                                    generateDescribeInputs();
                                </script>
                            """
                        } else if (TopicAction == 'MANAGE_TOPIC:ERROR') {
                            return['MANAGE_TOPIC:ERROR CODE 2']
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
        API_KEY = credentials('BASE64_API_KEY')
    }
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
                                string(name: 'MaxMessageBytes', value: "${values[5]}"),
                                string(name: 'ParamsAsENV', value: 'true,'),
                                string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${params_2},${CONNECTION_TYPE},")
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
                        expression {return params.Action == 'List'}
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