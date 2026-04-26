Lab 1 Report: Gitea Installation with Docker \& Git LFS Support



1\. Environment Setup

\--------------------

OS: Windows 11

Docker Desktop: v4.66.1

Gitea: v1.26.1

Git: Latest



Docker Desktop was installed and verified running with "Engine running" status.



2\. Part (a): Install Gitea Using Docker

\---------------------------------------



Step 1: Create Project Directory

mkdir "$env:USERPROFILE\\gitea-docker"

cd "$env:USERPROFILE\\gitea-docker"



Step 2: Create docker-compose.yml

\---------------------------------

version: "3"



networks:

&#x20; gitea:

&#x20;   external: false



services:

&#x20; server:

&#x20;   image: docker.gitea.com/gitea:1.26.1

&#x20;   container\_name: gitea

&#x20;   environment:

&#x20;     - USER\_UID=1000

&#x20;     - USER\_GID=1000

&#x20;   restart: always

&#x20;   networks:

&#x20;     - gitea

&#x20;   volumes:

&#x20;     - ./gitea:/data

&#x20;     - /etc/timezone:/etc/timezone:ro

&#x20;     - /etc/localtime:/etc/localtime:ro

&#x20;   ports:

&#x20;     - "3000:3000"

&#x20;     - "222:22"



Step 3: Start Gitea

\-------------------

mkdir gitea

docker compose up -d



Step 4: Initialize via Web UI

\-----------------------------

Accessed http://localhost:3000

Configured SQLite3 database

Set administrator account: XueZhengyang

Completed installation wizard



Result: Gitea successfully deployed and accessible at http://localhost:3000



3\. Part (b): Does Gitea Support LFS? Prove It.

\----------------------------------------------



Verification Method 1: Check Configuration File

docker exec gitea cat /data/gitea/conf/app.ini | Select-String -Pattern "lfs"



Output:

\[lfs]

ENABLED = true

PATH = /data/git/lfs



Verification Method 2: Web Interface

Navigated to Admin Panel -> Configuration

Confirmed LFS module is enabled by default



Verification Method 3: Functional Test (see Part c)

Successfully tracked .bin files with git lfs track

Uploaded 1.1 GB file via LFS protocol



Conclusion: YES, Gitea fully supports Git LFS. The feature is enabled by default in the Docker image and verified through both configuration inspection and functional testing.



4\. Part (c): Create a Repo with a Large File (1GB+)

\---------------------------------------------------



Step 1: Create Repository via Web UI

Repository name: large-file-test

Initialized with README



Step 2: Local Git LFS Setup

\---------------------------

git clone http://localhost:3000/XueZhengyang/large-file-test.git

cd large-file-test

git lfs install

git lfs track "\*.bin"



Step 3: Generate 1GB Test File

\------------------------------

fsutil file createnew largefile.bin 1073741824

(Get-Item largefile.bin).Length / 1GB

Output: 1 (1GB confirmed)



Step 4: Commit and Push

\-----------------------

git add .gitattributes largefile.bin

git commit -m "Add 1GB test file via Git LFS"

git push origin main



Push Output:

Uploading LFS objects: 100% (1/1), 1.1 GB | 80 MB/s, done.

Enumerating objects: 5, done.

To http://localhost:3000/XueZhengyang/large-file-test.git

&#x20;  efc3136..e7fc9c8  main -> main



Step 5: Web Verification

\------------------------

File largefile.bin (1.1 GB) visible in repository

Stored via Git LFS protocol confirmed



5\. Part (d): Backup and Restore \[Optional]

\-------------------------------------------



Backup to External Drive/USB

\----------------------------

cd "$env:USERPROFILE\\gitea-docker"

docker compose down

cd "$env:USERPROFILE"

Compress-Archive -Path "gitea-docker\\gitea" -DestinationPath "gitea-backup.zip"

Copy-Item "gitea-backup.zip" "E:\\gitea-backup.zip"



Restore on Another Computer

\---------------------------

On new machine with Docker installed:

mkdir "$env:USERPROFILE\\gitea-docker"

cd "$env:USERPROFILE\\gitea-docker"

Copy-Item "E:\\gitea-backup.zip" .

Expand-Archive -Path "gitea-backup.zip" -DestinationPath "." -Force

Create docker-compose.yml (same as original)

docker compose up -d



Result: All repositories, user data, and LFS files fully restored on new machine.



6\. Part (e): Install Gitea Without Docker \[Optional]

\----------------------------------------------------



Alternative installation using native binary with MySQL database:



Install MySQL

brew install mysql



Create database

mysql -u root -e "CREATE DATABASE gitea;"

mysql -u root -e "CREATE USER 'gitea'@'localhost' IDENTIFIED BY 'password';"

mysql -u root -e "GRANT ALL PRIVILEGES ON gitea.\* TO 'gitea'@'localhost';"



Download and run Gitea binary

wget https://dl.gitea.com/gitea/1.26.1/gitea-1.26.1-linux-amd64

chmod +x gitea

./gitea web



7. Conclusion

\-------------



This lab successfully demonstrated:

1\. Docker deployment of Gitea with persistent storage

2\. Git LFS support verified through configuration and functional testing

3\. Large file handling (1GB+) using LFS protocol

4\. Backup/restore capability for data portability

5\. Alternative installation methods understood



Gitea proves to be a lightweight, self-hosted Git service with full LFS support suitable for teams requiring large file version control.



