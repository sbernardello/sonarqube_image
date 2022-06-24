def podlabel = "jenkins-${UUID.randomUUID().toString()}"

pipeline { 
    agent { 
        kubernetes {
            label podlabel
            yaml """
kind: Pod
metadata:
  name: jenkins-slave
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: docker-registry-config
        mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
    - name: docker-registry-config
      configMap:
        name: docker-registry-config
"""
        }  
}
    stages {
        stage('Checkout'){
            
        }
        stage('Build') {
        environment {
            PATH = "/busybox:/kaniko:$PATH"
        }
        steps {
            
            container(name: 'kaniko', shell: '/busybox/sh') {

                sh """#!/busybox/sh
            
                echo " \n *** Building sonarqube image *** \n"

                /kaniko/executor -c `pwd` -f `pwd`/Dockerfile.arm64 --cleanup --destination=mobiletechnologies/sonarqube:arm64
        
            """
            }
        }
    
    }
   
  }
}
