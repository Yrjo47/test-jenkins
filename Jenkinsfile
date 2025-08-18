pipeline {
    agent any

    environment {
        SERVER_PORT = "${3000 + (env.CHANGE_ID ? env.CHANGE_ID.toInteger() : 0)}"
        NATS_PORT = "${6000 + (env.CHANGE_ID ? env.CHANGE_ID.toInteger() : 0)}"
    }

    options {
        buildDiscarder logRotator(
            artifactDaysToKeepStr: '5',
            artifactNumToKeepStr: '',
            daysToKeepStr: '5'
        )
        disableConcurrentBuilds()

        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }

    stages {
        stage('Start services') {
            when {
                changeRequest()
            }
            steps {
                echo "Trying to start services for ${GIT_BRANCH}..."
                sh """
                    export COMPOSE_PROJECT_NAME="project-pr-${CHANGE_ID}" SERVER_PORT=${SERVER_PORT} NATS_PORT=${NATS_PORT}
                    docker compose run -rm --no-deps --build platform_hh_worker
                    docker compose run -rm --no-deps --build platform_server
                    docker compose run -rm --no-deps --build platform_llm_worker
                """
            }
        }
        stage('Create a dedicated caddy domain') {
            when {
                changeRequest()
            }
            steps {
                echo "Trying to create a dedicated domain for ${GIT_BRANCH}..."
                script {
                    sh """
                        response=\$(curl -s -o /dev/null -w "%{http_code}" \
                        "http://localhost:2019/id/platform-${GIT_BRANCH}")

                        if [ "\$response" -eq 404 ]; then
                            echo "Creating new Caddy route..."
                            curl -X POST "http://localhost:2019/config/apps/http/servers/srv0/routes" \
                            -H "Content-Type: application/json" \
                            -d '{
                                "@id": "platform-${GIT_BRANCH}",
                                "match": [{"host": ["platform-${GIT_BRANCH}.localhost"]}],
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
                echo "The app is running on https://platform-${GIT_BRANCH}.localhost"
            }
        }
    }
}

