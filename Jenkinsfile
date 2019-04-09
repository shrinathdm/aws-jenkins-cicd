pipeline {
  agent {
    docker {
      image 'node:10-alpine'
      args '-p 20001-20100:3000'
    }
  }
  environment {
    CI = 'true'
    HOME = '.'
    npm_config_cache = 'npm-cache'
  }
  stages {
    stage('Install Packages') {
      steps {
        sh 'npm install'
      }
    }
    stage('Cache clean'){
       steps{
	 sh 'npm cache verify'
      }
    }
    stage('Test and Build') {
      parallel {
	    stage('Create Build Artifacts') {
          steps {
            sh 'npm run build'
          }
        }
        stage('Run Tests') {
          steps {
            sh 'npm run test'
          }
        }
      }
    }
    stage('Deployment') {
      parallel {
        stage('Staging') {
          when {
            branch 'staging'
          }
          steps {
            withAWS(region:'us-east-1',credentials:'Jenkins-AWS') {
              s3Delete(bucket: 'jenkis-cicd', path:'**/*')
              s3Upload(bucket: 'jenkis-cicd', workingDir:'build', includePathPattern:'**/*');
            }
            mail(subject: 'Staging Build', body: 'New Deployment to Staging', to: 'shrinath.mangalore2@harman.com')
          }
        }
        stage('Production') {
          when {
            branch 'master'
          }
          steps {
            withAWS(region:'us-east-1',credentials:'Jenkins-AWS') {
              s3Delete(bucket: 'jenkis-cicd', path:'**/*')
              s3Upload(bucket: 'jenkis-cicd', workingDir:'build', includePathPattern:'**/*');;
            }
            mail(subject: 'Production Build', body: 'New Deployment to Production', to: 'shrinath.mangalore2@harman.com')
          }
        }
      }
    }
  }
}
