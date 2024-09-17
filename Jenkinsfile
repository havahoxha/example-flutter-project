pipeline {
    agent any
    
    environment {
        FLUTTER_VERSION = '3.24.3' // Specify the Flutter version you want to install
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/havahoxha/example-flutter-project.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                # Install apt dependencies
                apt-get update
                apt-get install -y wget xz-utils clang cmake ninja-build pkg-config libgtk-3-dev

                # Install Flutter SDK
                wget -q https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_${FLUTTER_VERSION}-stable.tar.xz
                tar xf flutter_linux_${FLUTTER_VERSION}-stable.tar.xz
                export PATH="$PATH:$WORKSPACE/flutter/bin"
                git config --global --add safe.directory '*'
                flutter doctor
                '''
            }
        }

   
        stage('Increase Patch Version') {
            steps {
                script {
                    def mainDartPath = 'lib/main.dart'
                    echo "Reading ${mainDartPath}"
                    def content = readFile(mainDartPath)
                    
                    def newVersion = updateVersion(content)

                    // Replace the APP_VERSION line with the updated patch version
                    echo "Updating APP_VERSION to ${newVersion}"

                    def updatedContent = content.replaceFirst(/APP_VERSION\s*=\s*['"](\d+)\.(\d+)\.(\d+)['"]/, "APP_VERSION = '${newVersion}'")
                    echo "Updated content of ${mainDartPath}"

                    try {
                        echo "Attempting to write to file: ${mainDartPath}"
                        writeFile(file: mainDartPath, text: updatedContent)

                        echo "Updated APP_VERSION to ${newVersion}"
                    } catch (Exception e) {
                        // Log the error message
                        echo "Error while writing to file: ${e.getMessage()}"
                        error("Failed to write version to file: ${mainDartPath}")
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
        success {
            echo 'Build completed successfully!'
        }
        failure {
            echo 'Build failed.'
        }
    }
}

@NonCPS
def updateVersion(content) {
    def versionPattern = /APP_VERSION\s*=\s*['"](\d+)\.(\d+)\.(\d+)['"]/
    def matcher = content =~ versionPattern

    if (matcher) {
        def major = matcher[0][1].toInteger()
        def minor = matcher[0][2].toInteger()
        def patch = matcher[0][3].toInteger() + 1

        return "${major}.${minor}.${patch}"
    } else {
        error "APP_VERSION not found in the file."
    }
}
