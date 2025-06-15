pipeline {
    agent any

    environment {
        SERVER_PORT = "${3000 + (env.CHANGE_ID ? env.CHANGE_ID.toInteger() : 0)}"
        NATS_PORT = "${6000 + (env.CHANGE_ID ? env.CHANGE_ID.toInteger() : 0)}"
    }

    options {
        buildDiscarder logRotator(
            artifactDaysToKeepStr: '',
            artifactNumToKeepStr: '5',
            daysToKeepStr: '5'
        )
        disableConcurrentBuilds()

        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }

    stages {
        stage('For non-PR branches') {
            when {
                not { changeRequest() }
            }
            steps {
                echo 'A new non-PR branch was created'
            }
        }
        stage('Start services') {
            when {
                changeRequest()
            }
            steps {
                echo 'starting services...'
                sh """
                    COMPOSE_PROJECT_NAME="project-pr-${CHANGE_ID}" SERVER_PORT=${SERVER_PORT} NATS_PORT=${NATS_PORT} docker compose up -d --build
                """
            }
        }
        stage('Create a dedicated caddy domain') {
            when {
                changeRequest()
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
                                "upstreams": [{"dial": ":${SERVER_PORT}"}]
                                }]
                            }'
                        else
                            echo "No changes to the existing caddy route"
                        fi
                    """
                }
                echo "The app is running on https://${GIT_BRANCH}.localhost"
            }
        }
    }
}

