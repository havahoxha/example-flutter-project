pipeline {
    agent any
    
    environment {
        FLUTTER_VERSION = '3.3.0' // Specify the Flutter version you want to install
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh '''
                # Install wget if missing
                sudo apt-get update
                sudo apt-get install -y wget

                # Install Flutter SDK
                wget https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_${FLUTTER_VERSION}-stable.tar.xz
                tar xf flutter_linux_${FLUTTER_VERSION}-stable.tar.xz
                export PATH="$PATH:$WORKSPACE/flutter/bin"
                flutter doctor
                '''
            }
        }

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/havahoxha/example-flutter-project.git'
            }
        }

        stage('Increase Patch Version') {
            steps {
                script {
                    def mainDartPath = 'lib/main.dart'
                    def content = readFile(mainDartPath)
                    
                    // Find APP_VERSION in main.dart
                    def versionPattern = /APP_VERSION\s*=\s*['"](\d+)\.(\d+)\.(\d+)['"]/
                    def matcher = content =~ versionPattern

                    if (matcher) {
                        def major = matcher[0][1].toInteger()
                        def minor = matcher[0][2].toInteger()
                        def patch = matcher[0][3].toInteger() + 1

                        def newVersion = "${major}.${minor}.${patch}"

                        // Replace the APP_VERSION line with updated patch version
                        def updatedContent = content.replaceFirst(versionPattern, "APP_VERSION = '${newVersion}'")

                        // Write updated content back to the file
                        writeFile(file: mainDartPath, text: updatedContent)

                        echo "Updated APP_VERSION to ${newVersion}"
                    } else {
                        error "APP_VERSION not found in ${mainDartPath}"
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh '''
                export PATH="$PATH:$WORKSPACE/flutter/bin"
                flutter pub get
                flutter build linux
                '''
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'build/app/outputs/flutter-linux/*', allowEmptyArchive: true
        }
        success {
            echo 'Build completed successfully!'
        }
        failure {
            echo 'Build failed.'
        }
    }
}
