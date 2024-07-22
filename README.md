
# Google Drive Backup Script

This repository contains a Python script that automates the process of backing up files to a Google Drive folder. The script uses the Google Drive API to upload files from a specified local directory to a designated folder on Google Drive.

## Prerequisites

Before you can use this script, you need to have the following:

1. **Python 3.6+** installed on your machine.
2. **Pip** (Python package installer).
3. A **Google Cloud Platform** project with the Google Drive API enabled.
4. OAuth 2.0 credentials (a `credentials.json` file) from the Google Cloud Platform.

## Setup

1. **Clone the repository**:
    ```bash
    git clone https://github.com/yourusername/drive-backup-script.git
    cd drive-backup-script
    ```

2. **Install the required libraries**:
    ```bash
    pip install --upgrade google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client
    ```

3. **Set up Google Drive API credentials**:
    - Go to the [Google Cloud Console](https://console.cloud.google.com/).
    - Create a new project (or select an existing one).
    - Enable the Google Drive API for your project.
    - Create OAuth 2.0 credentials and download the `credentials.json` file.
    - Place the `credentials.json` file in the root directory of this repository.

## Usage

1. **Run the script**:
    ```bash
    python backitup.py
    ```

    The script will perform the following steps:
    
    - Check if the `token.json` file exists. If it does, it loads the stored credentials. If not, it initiates the OAuth 2.0 flow to obtain new credentials and save them to `token.json`.
    - Create a folder named `BackupFolderPC` on Google Drive if it doesn't already exist.
    - Upload all files from the specified local directory (`Random Folder`) to the `BackupFolderPC` folder on Google Drive.
    - Print the name of each file that has been successfully backed up.

## Script Explanation

### Imports

The script imports several libraries for authentication and interaction with the Google Drive API:

```python
import os
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
from googleapiclient.errors import HttpError
from googleapiclient.http import MediaFileUpload
```

### Constants

The `SCOPES` variable defines the scope of access required:

```python
SCOPES = ["https://www.googleapis.com/auth/drive"]
```

### Authentication

The script handles authentication by checking for existing credentials and refreshing or obtaining new ones as needed:

```python
creds = None
if os.path.exists("token.json"):
    creds = Credentials.from_authorized_user_file("token.json", SCOPES)

if not creds or not creds.valid:
    if creds and creds.expired and creds.refresh_token:
        creds.refresh(Request())
    else:
        flow = InstalledAppFlow.from_client_secrets_file("credentials.json", SCOPES)
        creds = flow.run_local_server(port=0)
    
    with open('token.json', 'w') as token:
        token.write(creds.to_json())
```

### Google Drive API Interaction

The script interacts with the Google Drive API to create a folder and upload files:

```python
try:
    service = build("drive", "v3", credentials=creds)

    response = service.files().list(
        q="name='BackupFolderPC' and mimeType='application/vnd.google-apps.folder'",
        spaces='drive'
    ).execute()

    if not response['files']:
        file_metadata = {
            "name": "BackupFolderPC",
            "mimeType": "application/vnd.google-apps.folder"
        }
        file = service.files().create(body=file_metadata, fields="id").execute()
        folder_id = file.get('id')
    else:
        folder_id = response['files'][0]['id']

    for file in os.listdir(r'C:\Users\91701\OneDrive\Desktop\Python Development\File Backup System\Random Folder'):
        file_metadata = {
            "name": file,
            "parents": [folder_id]
        }
        
        media = MediaFileUpload(f"C:\\Users\\91701\\OneDrive\\Desktop\\Python Development\\File Backup System\\Random Folder/{file}")
        upload_file = service.files().create(body=file_metadata, media_body=media, fields="id").execute()
        
        print("Backed up file: " + file)

except HttpError as e:
    print("Error: " + str(e))
```

### Error Handling

Any errors encountered during the API interactions are caught and printed:

```python
except HttpError as e:
    print("Error: " + str(e))
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [Google Drive API](https://developers.google.com/drive)
- [Google API Python Client](https://github.com/googleapis/google-api-python-client)

.

