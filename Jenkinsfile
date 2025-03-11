import groovy.json.JsonSlurper

pipeline {
    agent any

    environment {
        // These values come from job parameters defined via the properties tag
        ENV_VARS = "${params.ENV_VARS}"
        SSH_CREDENTIALS_ID = "${params.SSH_CREDENTIALS_ID}"
        SERVER_B = "${params.SERVER_B}"
        PROJECT_NAME = "${params.PROJECT_NAME}"
    }

    stages {

        stage('Create .env File') {
            steps {
                script {
                    // Initialize an empty map for environment variables
                    def envVars = [:]
                    
                    // If ENV_VARS parameter is provided, parse it as JSON
                    if (ENV_VARS?.trim()) {
                        def jsonSlurper = new JsonSlurper()
                        // Convert the LazyMap to a plain HashMap to avoid serialization issues
                        envVars = new HashMap(jsonSlurper.parseText(ENV_VARS))
                    } else {
                        echo "No dynamic environment variables provided."
                    }
                    
                    // Build the content for the .env file
                    def envContent = ""
                    envVars.each { key, value ->
                        envContent += "${key}=${value}\n"
                    }
                    
                    // Write the content to a .env file in the workspace
                    writeFile file: '.env', text: envContent
                    echo "Generated .env file:\n${envContent}"
                }
            }
        }

        stage('Fetch Data from Database') {
            steps {
                script {

                    // echo "SSH Credentials ID: ${SSH_CREDENTIALS_ID} and SERVER_B: ${SERVER_B}"
                    echo "SERVER_B: ${SERVER_B}"

                    sh '''
                        echo "Displaying contents of .env file:"
                        cat .env
                    '''

                    // Example: simulate fetching data from the database
                    echo "Fetched data: 111"

                    // Write or update the data.json file with the fetched data
                    writeFile file: 'data.json', text: 'kljhgfdsfghjkljhghj'
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent(credentials: ["${SSH_CREDENTIALS_ID}"]) {
                    script {
                        // Copy the contents of the "dist" folder to the target deployment directory on Server B
                        sh """
                            scp -r dist/* ${SERVER_B}:/var/www/${PROJECT_NAME}
                        """
                        // Optionally, reload the web server on Server B (e.g., Nginx) to serve the new version
                        sh """
                            ssh ${SERVER_B} 'sudo systemctl reload nginx.service'
                        """
                    }
                }
 
                // sshPublisher(
                //     publishers: [
                //         sshPublisherDesc(
                //             configName: "${SERVER_B}",
                //             transfers: [
                //                 sshTransfer(
                //                     sourceFiles: 'dist/**/*',
                //                     remoteDirectory: "/var/www/${PROJECT_NAME}",
                //                     removePrefix: 'dist'
                //                 )
                //             ],
                //             usePromotionTimestamp: false,
                //             verbose: true
                //         )
                //     ]
                // )
            }
        }

    }

    post {
        success {
            echo 'Deployment succeeded.'
            // Add notification steps, e.g., email or Slack notifications
        }
        failure {
            echo 'Deployment failed.'
            // Add notification steps
        }
        // always {
        //     // Clean up any temporary files if necessary
        //     echo 'Cleaning up workspace...'
        //     deleteDir() // Deletes all files in the workspace
        // }
    }
}
