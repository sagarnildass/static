pipeline {
     agent any
     stages {
         stage('Lint HTML') {
              steps {
                  sh 'tidy -q -e *.html'
              }
         }
         stage('Security Scan') {
              steps { 
                 aquaMicroscanner imageName: 'alpine:latest', notCompliesCmd: 'exit 1', onDisallowed: 'fail', outputFormat: 'html'
              }
         }     
         stage('Check S3 availability') {
              steps {
                  sh '''
                     response=$(curl -s -o /dev/null -w "%{http_code}\n" http://jenkins-static-bucket-sagarnil.s3-website.ap-south-1.amazonaws.com)
                     if [ "$response" != "200" ]
                     then
                      exit 1
                     fi
                  '''
              }
         }    
         stage('Upload to AWS') {
              steps {
                  withAWS(region:'ap-south-1') {
                  sh 'echo "Uploading content with AWS creds"'
                      s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file:'index.html', bucket:'jenkins-static-bucket-sagarnil')
                  }
              }
         }
         stage('Deliver for development to AWS') {
            when {
                branch 'Development'
            }
            steps {
                  withAWS(region:'ap-south-1') {
                  sh 'echo "Uploading content with AWS creds"'
                      s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file:'index_development.html', bucket:'jenkins-static-bucket-sagarnil-development')
                  }
              }
        }
        stage('Deploy for production to AWS') {
            when {
                branch 'Deployment'
            }
            steps {
                  withAWS(region:'ap-south-1') {
                  sh 'echo "Uploading content with AWS creds"'
                      s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file:'index_deployment.html', bucket:'jenkins-static-bucket-sagarnil-deployment')
                  }
              }
        }
     }
}
