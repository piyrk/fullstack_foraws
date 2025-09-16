pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
    }

   environment {
    BACKEND_DIR = 'car-rental-system-backend-master/car-rental-system-backend-master'
    FRONTEND_DIR = 'car-rental-system-frontend-master/car-rental-system-frontend-master'
    TOMCAT_URL = 'http://98.88.199.38:9090/manager/text'
    TOMCAT_USER = 'admin'
    TOMCAT_PASS = 'admin'
    BACKEND_WAR = 'springapp1.war'
    FRONTEND_WAR = 'frontapp1.war'
}


    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/piyrk/fullstack_foraws.git', branch: 'main'
            }
        }

        stage('Build Frontend (Vite)') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    script {
                        def nodeHome = tool name: 'NODE_HOME', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
                        env.PATH = "${nodeHome}/bin:${env.PATH}"
                    }
                    sh 'npm install'
                    sh 'npm start'
                }
            }
        }

        stage('Package Frontend as WAR') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    sh """
                        mkdir -p frontapp1_war/WEB-INF
                        cp -r dist/* frontapp1_war/
                        jar -cvf ../../${FRONTEND_WAR} -C frontapp1_war .
                    """
                }
            }
        }

        stage('Build Backend (Spring Boot WAR)') {
            steps {
                dir("${env.BACKEND_DIR}") {
                    sh 'mvn clean package'
                    sh "cp target/*.war ../../${BACKEND_WAR}"
                }
            }
        }

        stage('Deploy Backend to Tomcat (/springapp1)') {
            steps {
                script {
                    sh """
                        curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \\
                          --upload-file ${BACKEND_WAR} \\
                          "${TOMCAT_URL}/deploy?path=/springapp1&update=true"
                    """
                }
            }
        }

        stage('Deploy Frontend to Tomcat (/frontapp1)') {
            steps {
                script {
                    sh """
                        curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \\
                          --upload-file ${FRONTEND_WAR} \\
                          "${TOMCAT_URL}/deploy?path=/frontapp1&update=true"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Backend deployed: http://98.88.199.38:9090/springapp1"
            echo "✅ Frontend deployed: http://98.88.199.38:9090/frontapp1"
        }
        failure {
            echo "❌ Build or deployment failed"
        }
    }
}
