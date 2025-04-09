---
- name: Download Artifact from Nexus to Windows
  hosts: windows
  tasks:
    - name: Create directory for artifacts
      win_file:
        path: C:\Artifacts
        state: directory

    - name: Download artifact from Nexus
      win_get_url:
        url: http://nexus.local:8081/repository/myrepo/app-v1.0.0.zip
        dest: C:\Artifacts\app-v1.0.0.zip