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
    // SSH를 이용한 실행 파일 전송 및 실행
    stage('SSH Publish') {
      steps {
        sshPublisher(publishers: [sshPublisherDesc(configName: 'target',
        transfers: [sshTransfer(cleanRemote: false,
        excludes: '',
        execCommand: '''fuser -k 8080/tcp
        export BUILD_ID=PetClinic
        
        nohup java -jar /home/ubuntu/spring-petclinic-4.0.0-SNAPSHOT.jar >> nohup.out 2>&1 &''',
        execTimeout: 120000,
        flatten: false,
        makeEmptyDirs: false,
        noDefaultExcludes: false,
        patternSeparator: '[, ]+',
        remoteDirectory: '',
        remoteDirectorySDF: false,
        removePrefix: 'target',
        sourceFiles: 'target/spring-petclinic-4.0.0-SNAPSHOT.jar')],
        usePromotionTimestamp: false,
        useWorkspaceInPromotion: false,
        verbose: false)])
      }
    }
    
  }
}
