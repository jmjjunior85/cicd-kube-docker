
pipeline {
	agent any

	tools {
	  maven "MAVEN3.9"
	  jdk "JDK17"
	}

	environment {
      registry = "jmjjunior85/vprofileappdock"
      registryCredential = 'dockerhub'
	}

	stages {

	  stage('Build') {
	    steps {
	      sh 'mvn install -DskipTests'
	    }
	    post {
	      success {
	        echo "Archiving artifact"
	        archiveArtifacts artifacts: '**/*.war'
	      }
	    }
	  }

	  stage('Unit test') {
	    steps {
	      sh 'mvn test'
	    }
	  }

	  stage('Checkstyle Analysis') {
	    steps {
	      sh 'mvn checkstyle:checkstyle'
	    }
	  }

	  stage('Sonar Analysis') {
            environment {
                scannerHome = tool "sonarscanner"
            }
            steps {
                withSonarQubeEnv("sonarserver") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }


        stage('Build App Image') {
            steps {
                script {
                    dockerImage = docker.build( registry + ":V$BUILD_NUMBER", "./")
                }
            }
        }

        stage('Upload App Image') {
            steps {
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push("V$BUILD_NUMBER")
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Remove Container Images') {
        	steps {
        	  sh "docker rmi -f $registry:V$BUILD_NUMBER"
        	}
        }

        stage('Kubernetes Deploy') {
          agent {label 'KOPS'}
          steps {
            sh "sudo helm upgrade --install --force-replace vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
          }
        }


	}
}