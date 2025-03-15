
# MySQL Backup & S3 Upload 🚀

This project automates the process of backing up a MySQL database, encrypting the backup, and uploading it to an AWS S3 bucket. Perfect for DevOps engineers and system administrators who want a simple and automated backup process! 🔒☁️

## Features ✨
- **Automates MySQL backup**: Dumps the database into a `.sql` file.
- **Encrypts the backup**: Uses AES-256 encryption to secure the backup.
- **Uploads to AWS S3**: Easily store your encrypted backups in the cloud. ☁️
- **Environment variables**: Credentials stored securely in your environment variables. 🔐

## Prerequisites 🛠️
Before running this playbook, ensure the following are set up:
- **MySQL Database**: Ensure you have a running MySQL or MariaDB database that you want to back up.
- **AWS S3 Bucket**: Create an S3 bucket in your AWS account for storing backups.
- **Ansible**: Install Ansible on your local machine or remote server.
- **AWS CLI**: Ensure AWS CLI is installed and configured.

## Environment Variables 📜
Set the following environment variables with your credentials:
- `MYSQL_HOST`: The host of your MySQL database.
- `MYSQL_USER`: The MySQL username.
- `MYSQL_PASSWORD`: The MySQL password.
- `AWS_S3_BUCKET`: The name of your S3 bucket.
- `AWS_ACCESS_KEY_ID`: Your AWS access key.
- `AWS_SECRET_ACCESS_KEY`: Your AWS secret key.
- `ENCRYPTION_PASSWORD`: Password for encrypting the backup file.

## How to Run 🏃‍♂️
1. Clone the repository or create a new YAML file for the playbook.
2. Set the required environment variables.
3. Run the playbook using Ansible:

```bash
ansible-playbook /path/to/backup_mysql_to_s3.yml
```

## Example Playbook 🎯
Here's an example of the playbook you will be running:

```yaml
---
- name: Backup MySQL Database and Store in S3
  hosts: localhost
  vars:
    MYSQL_HOST: "{{ lookup('env', 'MYSQL_HOST') }}"
    MYSQL_USER: "{{ lookup('env', 'MYSQL_USER') }}"
    MYSQL_PASSWORD: "{{ lookup('env', 'MYSQL_PASSWORD') }}"
    AWS_S3_BUCKET: "{{ lookup('env', 'AWS_S3_BUCKET') }}"
    AWS_ACCESS_KEY_ID: "{{ lookup('env', 'AWS_ACCESS_KEY_ID') }}"
    AWS_SECRET_ACCESS_KEY: "{{ lookup('env', 'AWS_SECRET_ACCESS_KEY') }}"
    BACKUP_DIR: "/tmp/mysql_backups"
  
  tasks:
    - name: Ensure backup directory exists
      file:
        path: "{{ BACKUP_DIR }}"
        state: directory

    - name: Export MySQL database to a .sql file
      shell: >
        mysqldump -h {{ MYSQL_HOST }} -u {{ MYSQL_USER }} -p{{ MYSQL_PASSWORD }} my_database > {{ BACKUP_DIR }}/backup.sql
      args:
        creates: "{{ BACKUP_DIR }}/backup.sql"

    - name: Encrypt backup file
      command: >
        openssl aes-256-cbc -salt -in {{ BACKUP_DIR }}/backup.sql -out {{ BACKUP_DIR }}/backup.sql.enc -pass pass:{{ lookup('env', 'ENCRYPTION_PASSWORD') }}

    - name: Upload encrypted backup to S3
      aws_s3:
        bucket: "{{ AWS_S3_BUCKET }}"
        object: "backups/backup_{{ ansible_date_time.iso8601 }}.sql.enc"
        src: "{{ BACKUP_DIR }}/backup.sql.enc"
        mode: put
        aws_access_key: "{{ AWS_ACCESS_KEY_ID }}"
        aws_secret_key: "{{ AWS_SECRET_ACCESS_KEY }}"

    - name: Clean up local backup files
      file:
        path: "{{ item }}"
        state: absent
      with_items:
        - "{{ BACKUP_DIR }}/backup.sql"
        - "{{ BACKUP_DIR }}/backup.sql.enc"
```

## License 📄
GNU public v2.0 - Feel free to use, modify, and distribute with attribution!

## Contributing 🤝
If you have ideas for improving this project or fixing any issues, feel free to open an issue or submit a pull request! 😊

---

Happy Backing Up! 🚀
