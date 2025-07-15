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
                    def oldFile = 'test-results/describe-schema-test.xml'
                    if (fileExists(oldFile)) {
                        echo "Deleting old artifact: ${oldFile}"
                        sh "rm -f ${oldFile}"
                    }
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
        
        stage('Creating Schema Testing'){
            steps{
                script{
                    def createResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/create-schema', parameters: [
                        string(name: 'SubjectName', value: 'test-subject'), 
                        string(name: 'SchemaName', value: 'DefaultRecord'), 
                        string(name: 'SchemaNamespace', value: 'com.test'), 
                        string(name: 'SchemaFields', value: '''[
    {"name": "id", "type": "string"},
    {"name": "name", "type": "string"},
    {"name": "timestamp", "type": "long"}
]'''), 
                        string(name: 'CompatibilityLevel', value: 'BACKWARD'), 
                        string(name: 'SchemaType', value: 'AVRO'),
                        string(name: 'ParamsAsENV', value: 'true,'),
                        string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${CONNECTION_TYPE},")
                    ]

                    copyArtifacts(projectName: createResult.projectName, selector: specific("${createResult.number}"), filter: 'schema_create_result.txt')

                    def output = readFile('schema_create_result.txt').trim()
                    echo "Creating output: ${output}"
                    
                    // Generate JUnit XML for create schema test
                    generateJUnitXML('create-schema-test', output.contains('Success') || output.contains('created'), 'Create Schema Test', output)
                }
            }
        }

        stage('List Schema Testing'){
            steps{
                script{
                    def listResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/list-schema', parameters: [
                        string(name: 'ParamsAsENV', value: 'true,'),
                        string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${CONNECTION_TYPE},")
                    ]

                    copyArtifacts(projectName: listResult.projectName, selector: specific("${listResult.number}"), filter: 'list_result.txt')

                    def output = readFile('list_result.txt').trim()
                    echo "List output: ${output}"
                    
                    // Generate JUnit XML for list schema test
                    generateJUnitXML('list-schema-test', output.contains('test-subject'), 'List Schema Test', output)
                }
            }
        }

        stage('Get Schema Testing'){
            steps{
                script{
                    def getSchemaResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/get-schema', parameters: [
                        string(name: 'Subject', value: 'test-subject'),
                        string(name: 'ParamsAsENV', value: 'true,'),
                        string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${CONNECTION_TYPE},")
                    ]

                    copyArtifacts(projectName: getSchemaResult.projectName, selector: specific("${getSchemaResult.number}"), filter: 'get_schema_result.txt')

                    def output = readFile('get_schema_result.txt').trim()
                    echo "Get Schema output: ${output}"
                    
                    // Generate JUnit XML for describe topic test
                    generateJUnitXML('get-schema-test', output.contains('1'), 'Get Schema Test', output)
                }
            }
        }
        
        stage('Update Schema Testing'){
            steps{
                script{
                    def updateResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/create-schema', parameters: [
                        string(name: 'SubjectName', value: 'test-subject'), 
                        string(name: 'SchemaName', value: 'DefaultRecord'), 
                        string(name: 'SchemaNamespace', value: 'com.test'), 
                        string(name: 'SchemaFields', value: '''[
    {"name": "id", "type": "string"},
    {"name": "name", "type": "string"},
    {"name": "timestamp", "type": "long"},
    {"name": "value", "type": "int", "default": 0}
]'''), 
                        string(name: 'CompatibilityLevel', value: 'BACKWARD'), 
                        string(name: 'SchemaType', value: 'AVRO'),
                        string(name: 'ParamsAsENV', value: 'true,'),
                        string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${CONNECTION_TYPE},")
                    ]

                    copyArtifacts(projectName: updateResult.projectName, selector: specific("${updateResult.number}"), filter: 'schema_create_result.txt')

                    def output = readFile('schema_create_result.txt').trim()
                    echo "Update output: ${output}"
                    
                    // Generate JUnit XML for update topic test
                    generateJUnitXML('update-schema-test', output.contains('Success') || output.contains('updated'), 'Update Schema Test', output)
                }
            }
        }
        
        stage('Delete Schema Testing'){
            steps{
                script{
                    def deleteResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/delete-schema', parameters: [
                        string(name: 'Subject', value: 'test-subject'),
                        string(name: 'ParamsAsENV', value: 'true,'),
                        string(name: 'ENVIRONMENT_PARAMS', value: "${params_1},${CONNECTION_TYPE},")
                    ]

                    copyArtifacts(projectName: deleteResult.projectName, selector: specific("${deleteResult.number}"), filter: 'delete_schema_result.txt')

                    def output = readFile('delete_schema_result.txt').trim()
                    echo "Delete output: ${output}"
                    
                    // Generate JUnit XML for delete topic test
                    generateJUnitXML('delete-schema-test', output.contains('Success') || output.contains('deleted'), 'Delete Schema Test', output)
                }
            }
        }
    }
    
    post {
        always {
            // Publish JUnit test results
            junit testResults: 'test-results/*.xml', 
                  keepLongStdio: true,
                  allowEmptyResults: false
            
            // Archive the test result files
            archiveArtifacts artifacts: 'test-results/*.xml', allowEmptyArchive: true
            
            // Archive the original result files
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

// Helper function to generate JUnit XML format
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