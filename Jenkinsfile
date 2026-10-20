pipeline {
  agent any

  tools {
    maven "M3"
    jdk "JDK21"
  }
  environment {
    DOCKERHUB_CRED = credentials('dockerCredentials')
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
    // Docker Images 생성
    stage('Docker Image Create'){
      steps {
        sh '''
          docker build -t spring-petclinic:${BUILD_NUMBER} .
          docker tag spring-petclinic:${BUILD_NUMBER} frunting0/spring-petclinic:latest
        '''
      }
    }
    
    // Docker Image Push
    stage('Docker Image Push'){
      steps {
        sh '''
          echo ${DOCKERHUB_CRED_PSW} | docker login -u ${DOCKERHUB_CRED_USR} --password-stdin
          docker push frunting0/spring-petclinic:latest
        '''
      }
    }

    
    // SSH를 이용한 실행 파일 전송 및 실행
    stage('SSH Publish') {
      steps {
        sshPublisher(publishers: [sshPublisherDesc(configName: 'k8s-master',   //  publish over ssh에 지정한 이름 쓰기 
        transfers: [sshTransfer(cleanRemote: false,
        excludes: '',
        execCommand: '''
	set -ex
        #docker rm -f $(docker ps -aq)
        #docker rmi -f $(docker image)
        #docker run -itd -p 80:8080 --name=spring-petclinic frunting0/spring-petclinic:latest

	# ---- root home에 있는 yml 을 일반 home 으로 복사 (권한 문제 방지)----------
	echo "---디렉터리 생성---"
	mkdir -p /home/k8s
	echo "---디렉터리 생성 확인---"
	cp -av /root/spring-petclinic-502/k8s/*.yaml /home/k8s

	kubectl apply -f /home/k8s/

	echo "컨테이너를 Jenkins 파이프라인으로 kubernetes 배포 완료"
        ''',
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
    // Docker Image 삭제
    stage("Docker Image Remove") {
      steps {
        sh '''
        docker rmi -f spring-petclinic:${BUILD_NUMBER}
        docker rmi -f frunting0/spring-petclinic:latest
        '''
      } 
    }
  }
}
