# Jenkins Pipeline for Firebird Backup

This repository contains a [Jenkins](https://www.jenkins.io/) pipeline script designed for automating the backup of a Firebird database. 
The pipeline performs the following tasks:

1. **Preparation**: Sets up the backup environment, including directories and file paths.
2. **Backup**: Executes a database backup using `gbak`.
3. **Compression**: Compresses the backup file using `7-Zip`.
4. **Upload**: Uploads the compressed backup file to an Amazon S3 bucket.
5. **Confirmation**: Asks for user confirmation to preserve temporary files, with timeout.
6. **Cleanup**: Deletes temporary backup files if the user does not confirm to preserve them.

The script is configured to run on Windows and may require adjustments for other operating systems.

![Jenkins pipeline for Firebird backup](https://github.com/daniel-pereira-guimaraes/jenkins-firebird-backup/blob/main/jenkins.png)

## Prerequisites

- **Jenkins**: Install and run Jenkins. For installation instructions, visit the [Jenkins website](https://www.jenkins.io/doc/book/installing/).

- **Firebird installation**: Ensure that [Firebird](https://firebirdsql.org/en/firebird-3-0/) is installed to use `gbak.exe`.

- **7-Zip installation**: Install [7-Zip](https://www.7-zip.org/) to compress backup files.

- **S3 bucket and AWS CLI**:
   - **Create an S3 bucket** for backup storage.
   - **Install and configure AWS CLI** to interact with S3.
   - **IMPORTANT!** Using S3 may incur costs based on storage and data transfer. Please review AWS pricing details.
         
## Configure environment variables in Jenkins

- Go to **Manage Jenkins** > **Configure System**.
- In **Global properties**, check **Environment variables** and add:
  - **BACKUP_BUCKET**: S3 bucket name.
  - **BACKUP_DIR** (optional): Local directory for temporary backup files (default: `c:\backup`).
  - **BACKUP_NAME** (optional): Base name for backup files (default: `backup`).
  - **GBAK_PATH**: Path to `gbak.exe` (e.g., `C:\Program Files\Firebird\Firebird_3_0\bin\gbak.exe`).
  - **SEVEN_ZIP_PATH**: Path to `7z.exe` (e.g., `C:\Program Files\7-Zip\7z.exe`).

## Configure credentials in Jenkins

- Go to **Manage Jenkins** > **Manage Credentials** and add:
  - **fb-user**: Type **Username with password**.
  - **fb-password**: Type **Username with password**.
  - **fb-database**: Type **Secret text**. Provide the connection string in the format `host/port:path-to-database`, for example, `localhost/3050:c:\directory\database.fdb`.

## Create a new pipeline

- Open Jenkins and click **New Item**.
- Name the project and select **Pipeline**. Click **OK**.

## Configure the pipeline

- In **Pipeline**, select **Pipeline script from SCM**.
- Choose **Git** as the **SCM**.
- Provide the **Repository URL**:
  - e.g., `https://github.com/daniel-pereira-guimaraes/jenkins-firebird-backup/`
- Add credentials if needed (required for private repository only)
- Set `*/main` as the **Branch**.
- Specify `Jenkinsfile` as the **Script Path** (assuming it is in the root directory).
- Click `Save` button.

## Running the backup pipeline

To execute the backup using the newly configured pipeline, follow these steps:

- **Navigate to Jenkins Dashboard**: Go to your Jenkins instance in a web browser.
- **Select your pipeline**: Click on the pipeline job you created or configured.
- **Run the pipeline**: Click on the **"Build Now"** button to start the pipeline.
- **Monitor the execution**: Watch the pipeline progress and view logs in the console output.

The pipeline will perform the backup, compression, and upload tasks as defined in the script.

## License

- This project is licensed under the [MIT License](https://opensource.org/licenses/MIT). You are free to use, modify, and distribute the code as long as you include the original copyright notice and license in your documentation or credits.
