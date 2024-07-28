pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }
    
    stages {
        stage('Prepare') {
            steps {
              script {
                echo 'Preparing to start backup'
                def backupDir = env.BACKUP_DIR ?: "c:\\backup"
                def backupName = env.BACKUP_NAME ?: "backup"
                def timeStamp = new Date().format('yyyy-MM-dd_HH-mm-ss')
                def backupFileName = "${backupName}_${timeStamp}.fbk"
                def compressedFileName = "${backupFileName}.7Z"
                
                env.BACKUP_PATH = "${backupDir}\\${backupFileName}"
                env.COMPRESSED_BACKUP_PATH = "${backupDir}\\${compressedFileName}"
                env.BACKUP_S3_PATH = "s3://${BACKUP_BUCKET}/jenkins-study/${compressedFileName}"
                
                bat "if not exist ${backupDir} mkdir ${backupDir}"
              }
              checkout scm
            }
        }
        stage('Backup') {
            steps {
              withCredentials([
                string(credentialsId: 'fb-user', variable: 'FB_USER'),
                string(credentialsId: 'fb-password', variable: 'FB_PASSWORD'),
                string(credentialsId: 'fb-database', variable: 'FB_DATABASE')
              ]) {            
                script {
                  echo 'Creating database backup'
                  def gbakCommand = "\"${GBAK_PATH}\" -b -user ${FB_USER} -password ${FB_PASSWORD} ${FB_DATABASE} ${BACKUP_PATH}"
                  def result = bat(script: gbakCommand, returnStatus: true)
                  if (result != 0) {
                    error "Backup failed"
                  }
                }
              }                
            }
        }
        stage('Compress') {
            steps {
              script {
                echo 'Compressing backup'
                def compressCommand = "\"${SEVEN_ZIP_PATH}\" a ${COMPRESSED_BACKUP_PATH} ${BACKUP_PATH}"
                def result = bat(script: compressCommand, returnStatus: true)
                if (result != 0) {
                  error "Compress failed"
                }
              }
            }
        }
        stage('Upload') {
            steps {
              script {
                echo 'Uploading backup to S3'
                def uploadCommand = "aws s3 cp ${COMPRESSED_BACKUP_PATH} ${BACKUP_S3_PATH}"
                def result = bat(script: uploadCommand, returnStatus: true)
                if (result != 0) {
                  error "Upload failed"
                }
              }
            }
        }
        stage('Confirmation') {
          steps {
            script {
              try {
                timeout(time: 30, unit: 'SECONDS') {
                  input message: 'Preserve temporary files?', ok: 'Yes'
                }
                env.DELETE_TEMP_FILES = 'false'
              } catch(err) {
                env.DELETE_TEMP_FILES = 'true'
              }
            }
          }
        }
        stage('Cleanup') {
          when {
            expression { env.DELETE_TEMP_FILES == 'true' }
          }
          steps {
            script {
              echo 'Deleting temporary files'
              bat "del ${BACKUP_PATH}"
              bat "del ${COMPRESSED_BACKUP_PATH}"
            }
          }
        }
    }
}
