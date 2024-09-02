pipeline {
    agent {
        kubernetes {
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: podman
                image: quay.io/podman/stable:latest
                securityContext:
                  privileged: true
                command:
                - sleep
                args:
                - infinity
                volumeMounts:
                - name: podman-graph-storage
                  mountPath: /var/lib/containers

              - name: snyk
                image: snyk/snyk:docker
                command:
                - sleep
                args:
                - infinity
                volumeMounts:
                - name: podman-graph-storage
                  mountPath: /var/lib/containers
              
              volumes:
              - name: podman-graph-storage
                emptyDir: {}
            """
        }
    }

    environment {
        SNYK_TOKEN = credentials('snyk-api-token')  // Reference the Jenkins secret
        SNYK_ORG_ID = 'b250d181-3d61-47e9-8bfb-aa1375a534cc'  // Your Snyk organization ID
        SNYK_PROJECT_NAME = 'java-application2_local_test'  // Your Snyk project name
    }

    stages {
        stage('Check Podman') {
            steps {
                container('podman') {
                    sh 'podman --version'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x ./gradlew'
                sh './gradlew clean build'
            }
        }

        stage('Podman Build') {
            steps {
                container('podman') {
                    sh 'podman build -t daundkarash/java-application2_local .'
                    sh 'podman save -o /var/lib/containers/java-application2_local.tar daundkarash/java-application2_local'
                }
            }
        }

        stage('Load Image') {
            steps {
                container('podman') {
                    sh 'podman load -i /var/lib/containers/java-application2_local.tar'
                }
            }
        }

        stage('Verify Image') {
            steps {
                container('podman') {
                    sh 'podman images | grep daundkarash/java-application2_local'
                }
            }
        }

        stage('Snyk Container Scan') {
            steps {
                container('snyk') {
                    script {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh 'snyk auth $SNYK_TOKEN'  // Authenticate with Snyk 
                            sh 'DEBUG=*snyk* snyk monitor --all-projects --org=b250d181-3d61-47e9-8bfb-aa1375a534cc'
                            // sh 'snyk container test docker-archive:/var/lib/containers/java-application2_local.tar --file=Dockerfile --json --debug > snyk_scan_results.json'
                        }
                    }
                }
            }
        }

    //   stage('Publish Snyk Results') {
    // steps {
    //     container('snyk') {
    //         script {
    //             // Install curl if not present
    //             sh 'apk add --no-cache curl'

    //             // Read and sanitize the Snyk results file
    //             def snykResults = readFile('snyk_scan_results.json').trim()

    //             // Save sanitized results to a temporary file
    //             writeFile file: 'snyk_temp_results.json', text: snykResults

    //             // Use the temporary file with curl to avoid issues with large data
    //             sh """
    //             curl -X POST \\
    //                 -H "Authorization: token ${SNYK_TOKEN}" \\
    //                 -H "Content-Type: application/json" \\
    //                 -d @snyk_temp_results.json \\
    //                 https://snyk.io/api/v1/org/${SNYK_ORG_ID}/projects
    //             """
    //         }
    //     }
    // }
// }







        // stage('Archive Snyk Results') {
        //     steps {
        //         archiveArtifacts artifacts: 'snyk_scan_results.json', allowEmptyArchive: true
        //     }
        // }
    }
}
