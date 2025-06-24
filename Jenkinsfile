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
                    env.REST_ENDPOINT = props.REST_ENDPOINT
                    env.CLUSTER_ID = props.CLUSTER_ID
                    env.CONNECTION_TYPE = props.CONNECTION_TYPE
                }
            }
        }
        
        stage('Creating Topic Testing'){
            steps{
                script{
                    def createResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/create-topic-Jenkins', parameters: [
                        string(name: 'TopicName', value: 'test-topic'), 
                        string(name: 'Partitions', value: '6'), 
                        string(name: 'CleanupPolicy', value: 'Compact'), 
                        string(name: 'RetentionTime', value: '604800000'), 
                        string(name: 'RetentionSize', value: '-1'), 
                        string(name: 'MaxMessageBytes', value: '2097164'),
                        string(name: 'ParamsAsENV', value: 'true,'),
                        string(name: 'ENVIRONMENT_PARAMS', value: "${REST_ENDPOINT},${CLUSTER_ID},${CONNECTION_TYPE},")
                    ]

                    copyArtifacts(projectName: createResult.projectName, selector: specific("${createResult.number}"), filter: 'create_result.txt')

                    def output = readFile('create_result.txt').trim()
                    echo "Creating output: ${output}"
                    
                    // Generate JUnit XML for create topic test
                    generateJUnitXML('create-topic-test', output.contains('Success') || output.contains('created'), 'Create Topic Test', output)
                }
            }
        }

        stage('List Topic Testing'){
            steps{
                script{
                    def listResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/list-topic', parameters: [
                        string(name: 'ParamsAsENV', value: 'true,'),
                        string(name: 'ENVIRONMENT_PARAMS', value: "${REST_ENDPOINT},${CLUSTER_ID},${CONNECTION_TYPE},")
                    ]

                    copyArtifacts(projectName: listResult.projectName, selector: specific("${listResult.number}"), filter: 'list_result.txt')

                    def output = readFile('list_result.txt').trim()
                    echo "List output: ${output}"
                    
                    // Generate JUnit XML for list topic test
                    generateJUnitXML('list-topic-test', output.contains('test-topic'), 'List Topic Test', output)
                }
            }
        }

        stage('Describe Topic Testing'){
            steps{
                script{
                    def describeResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/describe-topic', parameters: [
                        string(name: 'TopicName', value: 'test-topic'),
                        string(name: 'ParamsAsENV', value: 'true,'),
                        string(name: 'ENVIRONMENT_PARAMS', value: "${REST_ENDPOINT},${CLUSTER_ID},${CONNECTION_TYPE},")
                    ]

                    copyArtifacts(projectName: describeResult.projectName, selector: specific("${describeResult.number}"), filter: 'describe_result.txt')

                    def output = readFile('describe_result.txt').trim()
                    echo "Describe output: ${output}"
                    
                    // Generate JUnit XML for describe topic test
                    generateJUnitXML('describe-topic-test', output.contains('test-topic'), 'Describe Topic Test', output)
                }
            }
        }
        
        stage('Update Topic Testing'){
            steps{
                script{
                    def updateResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/update-topic', parameters: [
                        string(name: 'TopicName', value: 'test-topic'), 
                        string(name: 'CleanupPolicy', value: 'Delete'), 
                        string(name: 'RetentionTime', value: '259200000'), 
                        string(name: 'RetentionSize', value: '-1'), 
                        string(name: 'MaxMessageBytes', value: '2097164'),
                        string(name: 'ParamsAsENV', value: 'true,'),
                        string(name: 'ENVIRONMENT_PARAMS', value: "${REST_ENDPOINT},${CLUSTER_ID},${CONNECTION_TYPE},")
                    ]

                    copyArtifacts(projectName: updateResult.projectName, selector: specific("${updateResult.number}"), filter: 'update_result.txt')

                    def output = readFile('update_result.txt').trim()
                    echo "Update output: ${output}"
                    
                    // Generate JUnit XML for update topic test
                    generateJUnitXML('update-topic-test', output.contains('Success') || output.contains('updated'), 'Update Topic Test', output)
                }
            }
        }
        
        stage('Delete Topic Testing'){
            steps{
                script{
                    def deleteResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/delete-topic', parameters: [
                        string(name: 'TopicName', value: 'test-topic'),
                        string(name: 'ParamsAsENV', value: 'true,'),
                        string(name: 'ENVIRONMENT_PARAMS', value: "${REST_ENDPOINT},${CLUSTER_ID},${CONNECTION_TYPE},")
                    ]

                    copyArtifacts(projectName: deleteResult.projectName, selector: specific("${deleteResult.number}"), filter: 'delete_result.txt')

                    def output = readFile('delete_result.txt').trim()
                    echo "Delete output: ${output}"
                    
                    // Generate JUnit XML for delete topic test
                    generateJUnitXML('delete-topic-test', output.contains('Success') || output.contains('deleted'), 'Delete Topic Test', output)
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