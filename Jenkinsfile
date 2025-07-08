properties([
    parameters([
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HIDDEN_HTML', 
            description: '', 
            omitValueField: false, 
            name: 'ParamsAsENV',
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
                    script: '''
                            return "<input type='checkbox' name='value' value='true'/> Use parameterized environment."
                            '''
                ]
            ]
        ],
        [$class: 'DynamicReferenceParameter', 
            choiceType: 'ET_FORMATTED_HIDDEN_HTML', 
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
                                <td><label>Connection Type : </label>
                                <select name='value'>
                                    <option value='Cloud'>Confluent Cloud</option>
                                    <option value='Platform'>Confluent Platform</option>
                                </select></td>
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
        CC_API_KEY = credentials('BASE64_SCHEMA_API_KEY')
        CP_API_KEY = credentials('CP_BASE64_SCHEMA_API_KEY')
    }
    parameters {
        string(name: 'SubjectName', defaultValue: 'default-subject', description: 'Schema subject name')
        string(name: 'SchemaName', defaultValue: 'DefaultRecord', description: 'Schema record name')
        string(name: 'SchemaNamespace', defaultValue: 'com.example', description: 'Schema namespace')
        text(name: 'SchemaFields', defaultValue: '''[
    {"name": "id", "type": "string"},
    {"name": "name", "type": "string"},
    {"name": "timestamp", "type": "long"}
]''', description: 'JSON array of schema fields')
        choice(name: 'CompatibilityLevel', choices: [
            'BACKWARD', 'FORWARD', 'FULL', 'BACKWARD_TRANSITIVE', 'FORWARD_TRANSITIVE', 'FULL_TRANSITIVE', 'NONE'
            ], description: 'Schema compatibility level')
        choice(name: 'SchemaType', choices: [
            'AVRO', 'JSON', 'PROTOBUF'
            ], description: 'Schema type')
    }
    stages {
        stage('Setup Environment') {
            steps{
                script{
                    def UseParamsAsENV = "${ParamsAsENV}".split(',').collect { it.trim() }.findAll { it }
                    def env_params = "${ENVIRONMENT_PARAMS}".split(',').collect { it.trim() }.findAll { it }
                    
                    def props = null
                    if (UseParamsAsENV[0] != 'true') {
                        props = readProperties file: 'env.properties'
                    }
                    else if (env_params[2] != 'Cloud'){
                        env_params[0] = env_params[0].replaceAll(";", ",")
                        env_params[2] = env_params[2].replaceAll(";", ",")
                    }

                    if (UseParamsAsENV[0] == 'true'){
                        if (env_params[2] == 'Platform,KafkaTools') {
                            env.SCHEMA_REGISTRY_URL = env_params[0]
                            env.KAFKA_TOOLS_PATH = env_params[1]
                        }
                        else {
                            env.SCHEMA_REGISTRY_URL = env_params[0]
                            env.CLUSTER_ID = env_params[1]
                        }
                    } else  {
                        if (props.CONNECTION_TYPE == 'Platform,KafkaTools') {
                            env.SCHEMA_REGISTRY_URL = props.SCHEMA_REGISTRY_URL
                            env.KAFKA_TOOLS_PATH = props.KAFKA_TOOLS_PATH
                        }
                        else {
                            env.SCHEMA_REGISTRY_URL = props.SCHEMA_REGISTRY_URL
                            env.CLUSTER_ID = props.CLUSTER_ID
                        }
                    }
                    
                    env.Auth = ""
                    env.Sort = "| jq '.'"
                    
                    if(env_params[2] == 'Cloud' || props?.CONNECTION_TYPE == 'Cloud'){
                        env.Auth = env.Auth + " -H \"Authorization: Basic \$CC_API_KEY\""
                    }
                    else if (env_params[2] == 'Platform,RestAPI' || props?.CONNECTION_TYPE == 'Platform,RestAPI'){
                        env.Auth = env.Auth + " -H \"Authorization: Basic \$CP_API_KEY\""
                    }

                    // Create Schema Part
                    def schemaFields = params.SchemaFields.replaceAll(/\s+/, ' ').trim()
                    def avroSchema = """
                    {
                        "type": "record",
                        "name": "${params.SchemaName}",
                        "namespace": "${params.SchemaNamespace}",
                        "fields": ${schemaFields}
                    }
                    """.replaceAll(/\s+/, ' ').trim()
                    
                    def jsonSchema = """
                    {
                        "type": "object",
                        "properties": {
                            "id": {"type": "string"},
                            "name": {"type": "string"},
                            "timestamp": {"type": "number"}
                        },
                        "required": [
                            "id", "name", "timestamp"
                        ]
                    }
                    """.replaceAll(/\s+/, ' ').trim()
                    
                    def selectedSchema = ""
                    if (params.SchemaType == "AVRO") {
                        selectedSchema = avroSchema
                    } else if (params.SchemaType == "JSON") {
                        selectedSchema = jsonSchema
                    } else {
                        selectedSchema = avroSchema // Default to AVRO
                    }
                    
                    // Escape the schema for JSON
                    def escapedSchema = selectedSchema.replaceAll('"', '\\\\\"')
                    
                    echo """
Subject Name : ${params.SubjectName}
Schema Name : ${params.SchemaName}
Schema Namespace : ${params.SchemaNamespace}
Schema Type : ${params.SchemaType}
Compatibility Level : ${params.CompatibilityLevel}
Schema Fields : ${schemaFields}
                    """
                    
                    env.HasSchema = "curl -s ${env.Auth} --request GET --url \"${env.SCHEMA_REGISTRY_URL}/subjects/${params.SubjectName}/versions\" | grep -c \"\\\"version\\\"\""
                    
                    env.SetCompatibilityCommand = """
                    curl -s ${env.Auth} -H 'Content-Type: application/vnd.schemaregistry.v1+json' --request PUT --url "${env.SCHEMA_REGISTRY_URL}/config/${params.SubjectName}" \
                        -d "{
                            \\"compatibility\\": \\"${params.CompatibilityLevel}\\"
                        }"
                    """
                    
                    env.CreateSchemaCommand = """
                    curl -s ${env.Auth} -H 'Content-Type: application/vnd.schemaregistry.v1+json' --request POST --url "${env.SCHEMA_REGISTRY_URL}/subjects/${params.SubjectName}/versions" \
                        -d "{
                            \\"schema\\": \\"${escapedSchema}\\"
                        }"
                    """
                    
                    env.CheckCompatibilityCommand = """
                    curl -s ${env.Auth} -H 'Content-Type: application/vnd.schemaregistry.v1+json' --request POST --url "${env.SCHEMA_REGISTRY_URL}/compatibility/subjects/${params.SubjectName}/versions/latest" \
                        -d "{
                            \\"schema\\": \\"${escapedSchema}\\"
                        }"
                    """
                    
                    if (env_params[2] == 'Platform,KafkaTools' || props?.CONNECTION_TYPE == 'Platform,KafkaTools'){
                        env.Sort = ""
                        // For platform tools, you might need to create schema files and use confluent CLI
                        env.HasSchema = "test -f /tmp/${params.SubjectName}.avsc"
                        env.CreateSchemaCommand = """
                        echo '${selectedSchema}' > /tmp/${params.SubjectName}.avsc && \
                        echo "Schema file created at /tmp/${params.SubjectName}.avsc"
                        """
                    }
                }
            }
        }
        stage('Create Schema'){
            steps{
                script{
                    def createResult = sh(
                        script: """
                            echo "Checking if schema exists for subject: ${params.SubjectName}"
                            
                            if ! ${env.HasSchema} >/dev/null 2>&1; then
                                echo "Setting compatibility level to ${params.CompatibilityLevel}"
                                ${env.SetCompatibilityCommand}
                                
                                echo "Creating new schema for subject: ${params.SubjectName}"
                                ${env.CreateSchemaCommand}
                                
                                echo "Successfully created schema for subject \\\"${params.SubjectName}\\\"."
                            else
                                echo "Schema already exists for subject \\\"${params.SubjectName}\\\". Checking compatibility..."
                                
                                compatibilityResult=\$(${env.CheckCompatibilityCommand})
                                echo "Compatibility check result: \$compatibilityResult"
                                
                                if echo "\$compatibilityResult" | grep -q '"is_compatible":true'; then
                                    echo "Schema is compatible. Creating new version..."
                                    ${env.CreateSchemaCommand}
                                    echo "Successfully created new version of schema for subject \\\"${params.SubjectName}\\\"."
                                else
                                    echo "Schema is not compatible with existing versions."
                                    echo "Compatibility check failed: \$compatibilityResult"
                                    exit 1
                                fi
                            fi
                        """,
                        returnStdout: true
                    ).trim()

                    writeFile file: 'schema_create_result.txt', text: createResult
                    archiveArtifacts artifacts: 'schema_create_result.txt'
                }
            }
        }
    }
}