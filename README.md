This is a small utility that I used to periodically backup data in my crouton-managed chroot to Google Drive.

## Installation

### Set up the backup script

The idea is to register an identity with which to communicate with Google Drive, perform [OAuth](https://developers.google.com/identity/protocols/oauth2/native-app) by hand under this identity to get a refresh token, create a file on Google Drive, and then provide the identity, token, and file ID to the backup script.

1.  [Create client credentials](https://console.developers.google.com/apis/credentials).

2.  Open the following URL in a browser, inserting the client ID obtained in the previous step:
    ```
    https://accounts.google.com/o/oauth2/v2/auth?scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive.file&response_type=code&redirect_uri=http%3A//127.0.0.1%3A9004&client_id=[insert client ID here]
    ```

    Extract the 'code' from the URL that the browser is redirected to. I.e.
    ```
    http://127.0.0.1:9004/?code=[copy this value]&scope=https://www.googleapis.com/auth/drive.file 
    ```

3.  Modify and run this command:
    ```
    curl -s \
      --data-urlencode "code=[insert code here]" \
      --data-urlencode "client_id=[insert client ID here]" \
      --data-urlencode "client_secret=[insert client secret here]" \
      --data-urlencode "redirect_uri=http://127.0.0.1:9004" \
      --data-urlencode "grant_type=authorization_code"
      https://oauth2.googleapis.com/token
    ```

    Extract the access token and refresh token from the response.
    ```
    {
      "access_token": [copy this value],
      "expires_in": 3599,
      "refresh_token": [copy this value too],
      "scope": "https://www.googleapis.com/auth/drive.file",
      "token_type": "Bearer"
    }
    ```

4.  Modify and run this command:
    ```
    echo stuff >> test.txt && \
    tar -cpzf - test.txt | \
    curl -v -s \
      -H "Authorization: Bearer [insert access token obtained in previous step]" \
      -H "Content-Type: application/gzip" \
      --data-binary "@-" \
      https://www.googleapis.com/upload/drive/v3/files?uploadType=media
    ```

    Extract the file ID from the response.
    ```
    {
     "kind": "drive#file",
     "id": [copy this value],
     "name": "Untitled",
     "mimeType": "application/gzip"
    }
    ```

    Open Google Drive web app or ChromeOS Files. There should be a file named "Untitled" in My Drive.
    From here on out, you can rename the file and move it anywhere at any time. Just be sure not to delete it.


5.  `backup` uses the `CLIENT_ID`, `CLIENT_SECRET`, `REFRESH_TOKEN`, and `FILE_ID` environment variables. You can create a script named `pre-backup` to set these (and run any other commands before backing up). `backup` will source this script before reading from any environment variables. See `pre-backup.sample`

6.  Modify the tar command in `backup` as needed. 

7.  `chmod +x backup`

### Set up daily backups

Create an anacron job to run `backup` daily.

```
1 0 heartbeat (date | cat && echo "anacron is running!") >> /home/alexn/cron.log
1 0 backup    cd /home/alexn/web-repos/backup_to_gdrive && ./backup >> /home/alexn/cron.log 2>&1
```

Optionally create a cron job that runs anacron every 10 minutes or so. This may be useful if your computer sleeps a lot and rarely reboots.
Anacron runs on boot, and at a set time each day as a cron job (on Ubuntu 16.04).
The 10-minute job ensures that a backup occurs even if the computer slept through today's cron job and didn't reboot.

```
*/10 * * * * root /usr/sbin/anacron -s && (date | cat && echo "starting anacron ...") >> /home/alexn/cron.log
```
