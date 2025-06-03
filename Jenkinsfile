pipeline {
    agent any  // Simplified agent declaration

    options {
        // More comprehensive build retention
        buildDiscarder logRotator(
            artifactDaysToKeepStr: '',  // Keep artifacts for 7 days
            artifactNumToKeepStr: '5',   // Keep last 5 artifacts
            daysToKeepStr: '5'         // Keep build records for 14 days
        )
        disableConcurrentBuilds()

        // Consider adding these common options:
        timeout(time: 30, unit: 'MINUTES')  // Prevent hung builds
        timestamps()                        // Add timestamps to logs
    }

    stages {
        stage('For all branches') {
            steps {
                sh "echo 'hello'"
            }
        }
        stage('For PR branches') {
            when {
                changeRequest()
            }
            steps {
                sh """
                    echo "I'm a ${GIT_BRANCH} PR branch with a ${CHANGE_ID} number. I will be available locally on port ${env.PLATFORM_PORT}"
                """
            }
        }
        stage('Start services') {
            when {
                changeRequest()
            }
            environment {
                PLATFORM_PORT = "${3000 + env.CHANGE_ID.toInteger()}"
            }
            steps {
                sh 'echo "starting services..."'
                sh """
                    HOST_PORT=${PLATFORM_PORT} docker compose up -d --build
                """
            }
        }
        stage('Create a dedicated caddy domain') {
            when {
                changeRequest()
            }
            environment {
                PLATFORM_PORT = "${3000 + env.CHANGE_ID.toInteger()}"
            }
            steps {
                script {
                    sh """
                        response=\$(curl -s -o /dev/null -w "%{http_code}" \
                        "http://localhost:2019/id/${CHANGE_ID}")

                        if [ "\$response" -eq 404 ]; then
                            echo "Creating new Caddy route..."
                            curl -X POST "http://localhost:2019/config/apps/http/servers/srv0/routes" \
                            -H "Content-Type: application/json" \
                            -d '{
                                "@id": "${CHANGE_ID}",
                                "match": [{"host": ["${GIT_BRANCH}.localhost"]}],
                                "handle": [{
                                "handler": "reverse_proxy",
                                "upstreams": [{"dial": ":${PLATFORM_PORT}"}]
                                }]
                            }'
                        else
                            echo "No changes to the existing caddy route"
                        fi
                    """
                }
            }
        }
    }
}

