pipeline {
  agent any

  tools {
    maven 'Maven 3.8.x+'
    jdk   'JDK 21'
  }

  environment {
    // Use snapshots if your version ends with -SNAPSHOT; use releases for non-SNAPSHOT versions.
    NEXUS_REPO_URL = 'http://nexus:8081/repository/maven-snapshots'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build + Deploy to Nexus (Maven)') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'NEXUS_CRED',
          usernameVariable: 'NUSER',
          passwordVariable: 'NPASS'
        )]) {
          sh '''
            set -eux

            # Build (skip tests to be fast)
            mvn -B -DskipTests clean package

            # Create a minimal Maven settings.xml that contains Nexus credentials
            cat > settings.xml <<EOT
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
          https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>nexus</id>
      <username>${NUSER}</username>
      <password>${NPASS}</password>
    </server>
  </servers>
</settings>
EOT

            # Deploy without modifying pom.xml:
            # -s settings.xml  -> use the creds above
            # -DaltDeploymentRepository -> point at your Nexus repo (snapshots or releases)
            mvn -B -DskipTests deploy \
              -s settings.xml \
              -DaltDeploymentRepository=nexus::default::${NEXUS_REPO_URL}

            # Show what we built
            ls -lh target || true
          '''
        }
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      echo 'Build + Deploy OK'
    }
    failure {
      echo 'Pipeline failed'
    }
  }
}
