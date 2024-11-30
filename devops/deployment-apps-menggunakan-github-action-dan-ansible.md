## Tambahkan SSH Private Key sebagai Secret di GitHub:

- Pergi ke repository di GitHub.
- Klik pada tab `Settings`.
- Pilih `Secrets and variables` > `Actions`.
- Klik New repository secret.
- Tambahkan secret dengan nama seperti `SSH_PRIVATE_KEY` dan masukkan kunci
  privat SSH Anda.

## Buat Workflow YAML:

- Buat file di `.github/workflows/deploy.yml`

Berikut adalah contoh file `deploy.yml` yang menggunakan kunci SSH untuk
terhubung ke host remote dan menjalankan Ansible playbook:

```
name: Deploy using Ansible

on:
  push:
    branches:
      - main

jobs:
  install-ansible:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4  # Updated to Node.js 20 compatible version

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.x'

    - name: Install Ansible
      run: |
        python -m pip install --upgrade pip
        pip install ansible

  deploy:
    needs: install-ansible
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4  # Updated to Node.js 20 compatible version

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.x'

    - name: Start ssh-agent
      uses: webfactory/ssh-agent@v0.5.4
      with:
        ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

    - name: Test SSH connection
      run: ssh -o StrictHostKeyChecking=no your_user@your_remote_host hostname

    - name: Run Ansible Playbook
      run: ansible-playbook -i hosts playbook.yml
      env:
        ANSIBLE_HOST_KEY_CHECKING: 'False'
```

### Penjelasan:

1. jobs: Mendefinisikan dua pekerjaan (install-ansible dan deploy).

2. install-ansible: Pekerjaan pertama untuk menginstal Ansible.

   - runs-on: Menentukan runner yang akan digunakan (ubuntu-latest).
   - steps: Langkah-langkah untuk pekerjaan ini.
     - Checkout repository: Melakukan checkout repositori.
     - Set up Python: Mengatur lingkungan Python.
     - Install Ansible: Menginstal Ansible menggunakan pip.

3. deploy: Pekerjaan kedua untuk menjalankan playbook Ansible.

   - needs: install-ansible: Menentukan bahwa pekerjaan ini bergantung pada
     pekerjaan install-ansible. Pekerjaan ini akan dijalankan hanya jika
     install-ansible selesai dengan sukses.
   - runs-on: Menentukan runner yang akan digunakan (ubuntu-latest).
   - steps: Langkah-langkah untuk pekerjaan ini.
     - Checkout repository: Melakukan checkout repositori.
     - Set up Python: Mengatur lingkungan Python.
     - Run Ansible Playbook: Menjalankan ansible-playbook.

Dengan memisahkan instalasi Ansible dan eksekusi playbook ke dalam dua pekerjaan
yang berbeda, Anda memastikan bahwa Ansible sudah terinstal sebelum menjalankan
playbook.

## Contoh konfigurasi ansible

buat file `playbook.yml`

```
---

- name: Deploy latest main branch for temubisnis app
  hosts: production
  become: no
  vars_files:
    - var.yaml

  tasks:
  - name: Display srcDir variable
    debug:
      var: srcDir

  - name: pull latest main branch
    ansible.builtin.shell: |
      cd {{ srcDir }}
      git pull origin {{ branchOrTag }}

  - name: build new image
    ansible.builtin.shell: |
      cd {{ deployDir }}
      docker compose build app

  - name: up service
    ansible.builtin.shell: |
      cd {{ deployDir }}
      docker compose up -d

  - name: optimize app
    ansible.builtin.shell: |
      docker exec app php artisan optimize
      docker exec app php artisan filament:cached-components
      docker exec app php artisan icons:cache

  - name: run migration
    ansible.builtin.shell: |
      docker exec app php artisan migrate --force
```

buat file `var.yml`

```
---

# Variables
app_name: "temubisnis"

baseDir: "/root/{{ app_name }}"
repo: "https://github.com/jhonoryza/temubisnis.git"
branchOrTag: "main"

deployDir: "{{ baseDir }}/.docker"
srcDir: "{{ baseDir }}"
```

terakhir buat file `hosts`

```
[production]
xxx.xxx.xxx.xxx           ansible_user=root
```

ganti `xxx.xxx.xxx.xxx` dengan ip address server anda.

konfigurasi selesai, maka setiap kali ada push code ke github branch main github
action akan berjalan secara otomatis untuk menjalankan ansible ke server tujuan.
