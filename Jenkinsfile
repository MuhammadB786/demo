pipeline {
  agent any

  tools {
    maven 'Maven 3.8.x+'
    jdk   'JDK 21'
  }

  environment {
    NEXUS_URL = 'http://nexus:8081'
    NEXUS_REPO = 'maven-releases'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build (Jar)') {
      steps {
        sh '''
          mvn -B -DskipTests clean package
          ls -lh target
        '''
      }
    }

    stage('Determine Maven Coordinates') {
      steps {
        sh '''
          GID=$(mvn -q -Dexec.executable=echo -Dexec.args='${project.groupId}' --non-recursive org.codehaus.mojo:exec-maven-plugin:3.1.0:exec)
          AID=$(mvn -q -Dexec.executable=echo -Dexec.args='${project.artifactId}' --non-recursive org.codehaus.mojo:exec-maven-plugin:3.1.0:exec)
          VER=$(mvn -q -Dexec.executable=echo -Dexec.args='${project.version}'   --non-recursive org.codehaus.mojo:exec-maven-plugin:3.1.0:exec)
          echo "GAV: $GID:$AID:$VER" > gav.txt
          printf "GID=%s\nAID=%s\nVER=%s\n" "$GID" "$AID" "$VER" > coords.env
        '''
        script { echo "Resolved " + readFile('gav.txt').trim() }
      }
    }

    stage('Upload to Nexus') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'NEXUS_CRED', usernameVariable: 'NUSER', passwordVariable: 'NPASS')]) {
          sh '''
            . coords.env
            FILE=$(ls target/*.jar | head -n1)
            echo "Uploading $FILE to $NEXUS_URL repository $NEXUS_REPO as $GID:$AID:$VER"
            curl -f -u "$NUSER:$NPASS" \
              -H "accept: application/json" \
              -H "Content-Type: multipart/form-data" \
              -F "maven2.groupId=$GID" \
              -F "maven2.artifactId=$AID" \
              -F "maven2.version=$VER" \
              -F "maven2.generate-pom=true" \
              -F "maven2.asset1=@${FILE}" \
              -F "maven2.asset1.extension=jar" \
              "$NEXUS_URL/service/rest/v1/components?repository=$NEXUS_REPO"
          '''
        }
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      echo 'Build + Upload OK'
    }
    failure {
      echo 'Pipeline failed'
    }
  }
}
