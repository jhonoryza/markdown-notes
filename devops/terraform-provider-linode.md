# Spesifikasi :

Ubuntu 24.04 LTS
Singapore, SG
Nanode 1 GB $5/month
Buat 1 user 
Install docker di dalamnya

## Struktur Direktori

```
.
├── main.tf
├── variables.tf
└── terraform.tfvars
```

## Langkah 1: Buat File Konfigurasi Terraform

`main.tf`

```
provider "linode" {
  token = var.linode_token
}

resource "linode_instance" "web" {
  label     = "ubuntu-24-04-instance"
  region    = "ap-southeast"
  type      = "g6-nanode-1"
  image     = "linode/ubuntu24.04"
  authorized_keys = [
    var.ssh_public_key
  ]

  tags = ["docker-host"]
}

resource "linode_instance_ssh_key" "ssh_key" {
  instance_id = linode_instance.web.id
  label       = "fajar"
  ssh_key     = var.ssh_public_key
}

resource "null_resource" "docker_install" {
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common",
      "curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -",
      "sudo add-apt-repository \"deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable\"",
      "sudo apt-get update",
      "sudo apt-get install -y docker-ce docker-ce-cli containerd.io",
      "sudo usermod -aG docker fajar"
    ]
  }

  depends_on = [
    linode_instance.web
  ]
}
```

`variables.tf`

```
variable "linode_token" {
  type = string
}

variable "ssh_public_key" {
  type = string
}

```

`terraform.tfvars`

```
linode_token = "your-linode-api-token"
ssh_public_key = "your-ssh-public-key"
```

### Penjelasan Konfigurasi

- Provider Linode: Mengonfigurasi provider Linode dengan menggunakan token API yang disediakan.
- Resource linode_instance: Mendefinisikan sebuah instance Linode dengan label "ubuntu-24-04-instance", menggunakan image Ubuntu 24.04 LTS di region Asia Tenggara (ap-southeast) dengan spesifikasi g6-nanode-1.
- Resource linode_instance_ssh_key: Menambahkan kunci SSH untuk user "fajar" ke instance Linode.
- Resource null_resource.docker_install: Menggunakan provisioner remote-exec untuk menginstal Docker di instance, serta menambahkan user "fajar" ke grup Docker.

## Langkah 2: Menjalankan Terraform

### Inisialisasi Terraform:

```
terraform init
```

### Periksa Rencana:

```
terraform plan
```

### Terapkan Konfigurasi:

```
terraform apply
```

Terraform akan menanyakan konfirmasi sebelum menerapkan perubahan. Gunakan -auto-approve untuk menjalankannya tanpa interaksi.

### Catatan Tambahan

- Pastikan Anda memiliki pengetahuan yang memadai tentang keamanan saat menangani kunci API Linode dan pengelolaan kunci SSH.
- Perhatikan bahwa penggunaan null_resource dengan provisioner remote-exec memungkinkan instalasi Docker dan konfigurasi tambahan. Ini cocok untuk keperluan pengujian dan pengembangan, namun untuk lingkungan produksi, pertimbangkan penggunaan modul Terraform yang lebih terstruktur dan aman.
- Terraform akan menyimpan state infrastruktur dalam file terraform.tfstate. Pastikan untuk memanage state file dengan aman dan mempertimbangkan penggunaan backend remote seperti AWS S3 atau Terraform Cloud.

Dengan menggunakan konfigurasi ini, Anda akan dapat memprovision instance Linode dengan spesifikasi yang diinginkan dan menginstall Docker di dalamnya, serta menambahkan user "fajar" ke grup Docker untuk memulai penggunaan instance tersebut.
