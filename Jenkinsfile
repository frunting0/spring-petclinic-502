pipeline {
  agent any

  tools {
    maven "M3"
    jdk "JDK21"
  }
  
  stages {
    // Github에 있는 소스코드 가져오기
    stage('Git Clone') { 
      steps {
        git url: 'https://github.com/frunting0/spring-petclinic-502.git',
        branch: 'main'
      } 
    }
    // Maven을 이용한 build
    stage('Maven Build') {
      steps {
        sh 'mvn -Dmaven.test.failure.ignore=true clean package'
      }
    }

    
  }
}
