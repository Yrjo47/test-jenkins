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
        stage('Hello') {
            steps {
                echo 'hello'
            }
            
            // Optional post section
            post {
                always {
                    echo "Stage completed"
                }
            }
        }
    }
}
