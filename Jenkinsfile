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
                        '''return['MANAGE_SCHEMA:ERROR']'''
                ], 
                script: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''return["Create","Update","Get","List:selected","Delete"]'''
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
                        '''return['MANAGE_SCHEMA:ERROR']'''
                ], 
                $class: 'GroovyScript', 
                script: [
                    classpath: [], 
                    sandbox: true, 
                    script: 
                        '''
                        if (Action == 'List'){
                            return "<label>This action didn't need any options.</label>"
                        } else if (Action == 'Create') {
                            return """
                                <table>
                                <tr>
                                <img src="https://www.mfec.co.th/wp-content/uploads/2023/09/New-Logo-MFEC-More.-2023.jpg" style="width: 150px; height: auto; border: 2px solid #555; border-radius: 10px;">
                                <td><label>Subject Name</label><br><input name='value' type='text' value='test-subject'></td>
                                <td><label>Schema Name</label><br><input name='value' type='text' value='DefaultRecord'></td>
                                <td><label>Schema Namespace</label><br><input name='value' type='text' value='com.test'></td>
                                <td><label>Compat Level</label><br>
                                <select name='value'>
                                    <option value='BACKWARD' selected>BACKWARD</option>
                                    <option value='FORWARD'>FORWARD</option>
                                    <option value='FULL'>FULL</option>
                                    <option value='NONE'>NONE</option>
                                </select></td>
                                <td><label>Schema Type</label><br>
                                <select name='value'>
                                    <option value='AVRO' selected>AVRO</option>
                                    <option value='JSON'>JSON</option>
                                    <option value='PROTOBUF'>PROTOBUF</option>
                                </select></td>
                                </tr>
                                <tr>
                                <td colspan='6'><label>Schema Fields</label><br><textarea name='value' rows='8' cols='80' style='width: 100%;'>[
    {"name": "id", "type": "string"},
    {"name": "name", "type": "string"},
    {"name": "timestamp", "type": "long"}
]</textarea></td>
                                </tr>
                                </table>
                            """
                        } else if (Action == 'Update') {
                            return """
                                <table>
                                <tr>
                                <td><label>Subject Name</label><br><input name='value' type='text' value='test-subject'></td>
                                <td><label>Schema Name</label><br><input name='value' type='text' value='DefaultRecord'></td>
                                <td><label>Schema Namespace</label><br><input name='value' type='text' value='com.test'></td>
                                <td><label>Compat Level</label><br>
                                <select name='value'>
                                    <option value='BACKWARD' selected>BACKWARD</option>
                                    <option value='FORWARD'>FORWARD</option>
                                    <option value='FULL'>FULL</option>
                                    <option value='NONE'>NONE</option>
                                </select></td>
                                <td><label>Schema Type</label><br>
                                <select name='value'>
                                    <option value='AVRO' selected>AVRO</option>
                                    <option value='JSON'>JSON</option>
                                    <option value='PROTOBUF'>PROTOBUF</option>
                                </select></td>
                                </tr>
                                <tr>
                                <td colspan='5'><label>Schema Fields</label><br><textarea name='value' rows='8' cols='80' style='width: 100%;'>[
    {"name": "id", "type": "string"},
    {"name": "name", "type": "string"},
    {"name": "timestamp", "type": "long"},
    {"name": "value", "type": "int", "default": 0}
]</textarea></td>
                                </tr>
                                </table>
                            """
                        } else if (Action == 'MANAGE_SCHEMA:ERROR') {
                            return['MANAGE_SCHEMA:ERROR']
                        } else {
                            return """
                                <table><tr>
                                <td><label>Subject Name : </label><br><input name='value' type='text' value='test-subject'></td>
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
                        env.params_1 = props.SCHEMA_REGISTRY_URL.replaceAll(",", ";")
                    }
                    else{
                        env.params_1 = props.SCHEMA_REGISTRY_URL
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
                        message: "Type the subject name to confirm deletion: '${values[0]}'",
                        parameters: [
                            string(defaultValue: '', description: 'Re-type the subject name exactly to confirm', name: 'CONFIRM_NAME')
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
        stage('Schema Management') {
            parallel{
                stage('Create'){
                    when{
                        expression {return params.Action == 'Create'}
                    }
                    steps{
                        script{
                            def option = "${Option}"

                            def parts = []
                            def current = new StringBuilder()
                            int curlyBraces = 0
                            int squareBrackets = 0

                            for (int i = 0; i < option.length(); i++) {
                                char c = option.charAt(i)

                                if (c == '{') {
                                    curlyBraces++
                                    current.append(c)
                                } else if (c == '}') {
                                    curlyBraces--
                                    current.append(c)
                                } else if (c == '[') {
                                    squareBrackets++
                                    current.append(c)
                                } else if (c == ']') {
                                    squareBrackets--
                                    current.append(c)
                                } else if (c == ',' && curlyBraces == 0 && squareBrackets == 0) {
                                    // Split outside of both {} and []
                                    parts << current.toString().trim()
                                    current.setLength(0)
                                } else {
                                    current.append(c)
                                }
                            }

                            // Add last part
                            if (current.length() > 0) {
                                parts << current.toString().trim()
                            }

                            // Now categorize parts
                            def jsons = []
                            def values = []

                            parts.each { val ->
                                def trimmed = val.trim()
                                if ((trimmed.startsWith('{') && trimmed.endsWith('}')) || 
                                    (trimmed.startsWith('[') && trimmed.endsWith(']'))) {
                                    jsons << trimmed
                                } else {
                                    values << trimmed
                                }
                            }
                            
                            echo """
Subject Name : ${values[0]}
Schema Name : ${values[1]}
Schema Namespace : ${values[2]}
Schema Fields : ${jsons[0]}
Compatibility Level : ${values[3]}
Schema Type : ${values[4]}
                            """
                            
                            def createResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/create-schema', parameters: [
                                string(name: 'SubjectName', value: "${values[0]}"), 
                                string(name: 'SchemaName', value: "${values[1]}"), 
                                string(name: 'SchemaNamespace', value: "${values[2]}"), 
                                string(name: 'SchemaFields', value: "${values[3]}"), 
                                string(name: 'CompatibilityLevel', value: "${values[4]}"), 
                                string(name: 'SchemaType', value: "${values[5]}"),
                                string(name: 'ParamsAsENV', value: 'true,'),
                                string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${CONNECTION_TYPE},")
                            ]

                            copyArtifacts(projectName: createResult.projectName, selector: specific("${createResult.number}"), filter: 'schema_create_result.txt')

                            def output = readFile('schema_create_result.txt').trim()
                            echo "Creating output: ${output}"

                            generateJUnitXML('create-schema', output.contains('Success') || output.contains('created'), 'Create Schema', output)
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
                            
                            echo """
Subject Name : ${values[0]}
Schema Name : ${values[1]}
Schema Namespace : ${values[2]}
Schema Fields : ${values[3]}
Compatibility Level : ${values[4]}
Schema Type : ${values[5]}
                            """
                            
                            def updateResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/create-schema', parameters: [
                                string(name: 'SubjectName', value: "${values[0]}"), 
                                string(name: 'SchemaName', value: "${values[1]}"), 
                                string(name: 'SchemaNamespace', value: "${values[2]}"), 
                                string(name: 'SchemaFields', value: "${values[3]}"), 
                                string(name: 'CompatibilityLevel', value: "${values[4]}"), 
                                string(name: 'SchemaType', value: "${values[5]}"),
                                string(name: 'ParamsAsENV', value: 'true,'),
                                string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${CONNECTION_TYPE},")
                            ]

                            copyArtifacts(projectName: updateResult.projectName, selector: specific("${updateResult.number}"), filter: 'schema_create_result.txt')

                            def output = readFile('schema_create_result.txt').trim()
                            echo "Update output: ${output}"

                            generateJUnitXML('update-schema', output.contains('Success') || output.contains('updated'), 'Update Schema', output)
                        }
                    }
                }
                stage('Get'){
                    when{
                        expression {return params.Action == 'Get'}
                    }
                    steps{
                        script{
                            def option = "${Option}"
                            def values = option.split(',').collect { it.trim() }.findAll { it }
                            
                            echo """
Subject Name : ${values[0]}
                            """
                            
                            def getResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/get-schema', parameters: [
                                string(name: 'Subject', value: "${values[0]}"),
                                string(name: 'ParamsAsENV', value: 'true,'),
                                string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${CONNECTION_TYPE},")
                            ]

                            copyArtifacts(projectName: getResult.projectName, selector: specific("${getResult.number}"), filter: 'get_schema_result.txt')

                            def output = readFile('get_schema_result.txt').trim()
                            echo "Get output: ${output}"

                            generateJUnitXML('get-schema', output.contains("${values[0]}") || output.contains('1'), 'Get Schema', output)
                        }
                    }
                }
                stage('List'){
                    when{
                        expression {return params.Action == 'List'}
                    }
                    steps{
                        script{
                            def listResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/list-schema', parameters: [
                                string(name: 'ParamsAsENV', value: 'true,'),
                                string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${CONNECTION_TYPE},")
                            ]

                            copyArtifacts(projectName: listResult.projectName, selector: specific("${listResult.number}"), filter: 'list_result.txt')

                            def output = readFile('list_result.txt').trim()
                            echo "List output: ${output}"

                            generateJUnitXML('list-schema', !output.toLowerCase().contains('error'), 'List Schema', output)
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
Subject Name : ${values[0]}
                            """
                            if (env.confirmation){
                                def deleteResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/delete-schema', parameters: [
                                    string(name: 'Subject', value: "${values[0]}"),
                                    string(name: 'ParamsAsENV', value: 'true,'),
                                    string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${CONNECTION_TYPE},")
                                ]

                                copyArtifacts(projectName: deleteResult.projectName, selector: specific("${deleteResult.number}"), filter: 'delete_schema_result.txt')

                                def output = readFile('delete_schema_result.txt').trim()
                                echo "Delete output: ${output}"

                                generateJUnitXML('delete-schema', output.contains('Success') || output.contains('deleted'), 'Delete Schema', output)
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
            echo 'All schema management tests passed successfully!'
        }
        
        failure {
            echo 'Some schema management tests failed. Check the test results for details.'
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
<testsuite name="SchemaManagementTests" tests="1" failures="${passed ? 0 : 1}" errors="0" time="1.0">
    <testcase name="${testName}" classname="SchemaManagement" time="1.0">
        <system-out><![CDATA[${output}]]></system-out>${failureElement}
    </testcase>
</testsuite>"""
    
    // Create test-results directory if it doesn't exist
    sh 'mkdir -p test-results'
    
    // Write the XML file
    writeFile file: "test-results/${testName}.xml", text: xmlContent
}