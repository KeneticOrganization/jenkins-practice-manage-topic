pipeline {
    agent any
    environment {
        REST_ENDPOINT = 'https://pkc-ldvr1.asia-southeast1.gcp.confluent.cloud:443'
        API_KEY = credentials('BASE64_API_KEY')
        CLUSTER_ID = 'lkc-yjvgnk'
    }
    stages {
        stage('Creating Topic Testing'){
            steps{
                script{
                    def createResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/create-topic-Jenkins', parameters: [
                        string(name: 'TopicName', value: 'test-topic'), 
                        string(name: 'Partitions', value: '6'), 
                        string(name: 'CleanupPolicy', value: 'Compact'), 
                        string(name: 'RetentionTime', value: '604800000'), 
                        string(name: 'RetentionSize', value: '-1'), 
                        string(name: 'MaxMessageBytes', value: '2097164')
                    ]

                    copyArtifacts(projectName: createResult.projectName, selector: specific("${createResult.number}"), filter: 'create_result.txt')

                    def output = readFile('create_result.txt').trim()
                    echo "Creating output: ${output}"
                }
            }
        }

        stage('List Topic Testing'){
            steps{
                script{
                    def listResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/list-topic'

                    copyArtifacts(projectName: listResult.projectName, selector: specific("${listResult.number}"), filter: 'list_result.txt')

                    def output = readFile('list_result.txt').trim()
                    echo "List output: ${output}"
                }
            }
        }

        stage('Describe Topic Testing'){
            steps{
                script{
                    def describeResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/describe-topic', parameters: [
                        string(name: 'TopicName', value: 'test-topic')
                    ]

                    copyArtifacts(projectName: describeResult.projectName, selector: specific("${describeResult.number}"), filter: 'describe_result.txt')

                    def output = readFile('describe_result.txt').trim()
                    echo "Describe output: ${output}"
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
                        string(name: 'MaxMessageBytes', value: '2097164')
                    ]

                    copyArtifacts(projectName: updateResult.projectName, selector: specific("${updateResult.number}"), filter: 'update_result.txt')

                    def output = readFile('update_result.txt').trim()
                    echo "Update output: ${output}"
                }
            }
        }
        
        stage('Delete Topic Testing'){
            steps{
                script{
                    def deleteResult = build job: 'Jenkins Practice/jenkins-practice-manage-topic/delete-topic', parameters: [
                        string(name: 'TopicName', value: 'test-topic')
                    ]

                    copyArtifacts(projectName: deleteResult.projectName, selector: specific("${deleteResult.number}"), filter: 'delete_result.txt')

                    def output = readFile('delete_result.txt').trim()
                    echo "Delete output: ${output}"
                }
            }
        }
    }
}