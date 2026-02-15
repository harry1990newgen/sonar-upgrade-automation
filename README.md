# sonar-upgrade-automation (Developer Edition)
Automate sonarqube upgrade process using ansible
This project automates the SonarQube upgrade process using Ansible.
It handles service management, backup, extraction, configuration updates, and permission corrections to ensure a smooth upgrade.


Directory structure followed as below :

sonar-upgrade-automation/
                  └── automation-sonar/
                    └── tasks/
                       ├── main.yml
                       ├── permission.yml
                       ├── extract.yml
                       ├── rename2.yml
                       ├── start-service.yml
                       ├── stop-service.yml
                       ├── append-sonar.yml
                       ├── system-service2.yml
                       └── tarback.yml
                       ├── backup.yml

-----------------------------------------------------

Flow of tasks :

Step1: Stop the services of postgresql+sonarqube.
       (stop-service.yml)

Step2: Take backup of sonarqube working directory (/opt/sonarqube/) and place the archived(tar) to /tmp/sonarqube-{{current_date}}.
       (tarback.yml)

Step3: Download zip file from official sonarqube website - This step is manual as developer edition not accessible publically , however for community edition hasnt such limitations.
        
Step4: Copy /tmp/{{latest version .zip}} to /opt/sonarqube/.
       (extract.yml)

Step5: Rename the latest version directory as per Sonarqube-MAJOR:MINOR:PATCH format.
       (rename2.yml)

Step6: Replace older version with new version in system-service file.
       (system-service2.yml)

Step7: Edit sonar.properties with jdbc connection string or credentials.
       (append-sonar.yml)

Step8: Correct permissions+ownership of sonar directories or files.
       (permission.yml)

Step9: Start sonarqube_postgresql services.
       (start-service.yml)

---------------------------------------------------------




