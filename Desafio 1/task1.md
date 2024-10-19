# Análise Técnica do Código Terraform

O arquivo **main.tf** provê uma infraestrutura básica na AWS utilizando o Terraform. Ele cria uma série de recursos, incluindo uma VPC, Subnet, Internet Gateway, Route Table, Security Group, uma chave SSH, e uma instância EC2. Aqui está a análise detalhada de cada componente:

---

### 1. **Provider AWS**
```bash
provider "aws" {
  region = "us-east-1"
}
```
Este bloco define o **provedor** AWS, especificando que todos os recursos serão criados na região **us-east-1** (Norte da Virgínia).

---

### 2. **Variáveis de Projeto e Candidato**
```bash
variable "projeto" {
  description = "Nome do projeto"
  type        = string
  default     = "VExpenses"
}

variable "candidato" {
  description = "Nome do candidato"
  type        = string
  default     = "SeuNome"
}
```
Essas variáveis permitem a customização de alguns valores, como o nome do projeto e o nome do candidato. Esses valores são utilizados para nomear os recursos que serão criados, garantindo um nome único e identificável.

---

### 3. **Criação de uma Chave Privada e Key Pair**
```bash
resource "tls_private_key" "ec2_key" {
  algorithm = "RSA"
  rsa_bits  = 2048
}

resource "aws_key_pair" "ec2_key_pair" {
  key_name   = "${var.projeto}-${var.candidato}-key"
  public_key = tls_private_key.ec2_key.public_key_openssh
}
```
- Um par de chaves SSH é gerado utilizando o algoritmo RSA com 2048 bits.
- O par de chaves gerado é registrado na AWS como uma **Key Pair** com o nome baseado nas variáveis `projeto` e `candidato`. Esta chave será usada para acessar a instância EC2.

---

### 4. **Criação da VPC**
```bash
resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "${var.projeto}-${var.candidato}-vpc"
  }
}
```
Uma VPC é criada com o bloco CIDR **10.0.0.0/16**. Ela permite a resolução de DNS (tanto para suporte quanto para hostnames) e é nomeada de acordo com as variáveis.

---

### 5. **Criação da Subnet**
```bash
resource "aws_subnet" "main_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "${var.projeto}-${var.candidato}-subnet"
  }
}
```
Uma **subnet** é criada dentro da VPC com o bloco CIDR **10.0.1.0/24**, alocada na **zona de disponibilidade** `us-east-1a`.

---

### 6. **Internet Gateway e Route Table**
```bash
resource "aws_internet_gateway" "main_igw" {
  vpc_id = aws_vpc.main_vpc.id

  tags = {
    Name = "${var.projeto}-${var.candidato}-igw"
  }
}

resource "aws_route_table" "main_route_table" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main_igw.id
  }

  tags = {
    Name = "${var.projeto}-${var.candidato}-route_table"
  }
}

resource "aws_route_table_association" "main_association" {
  subnet_id      = aws_subnet.main_subnet.id
  route_table_id = aws_route_table.main_route_table.id

  tags = {
    Name = "${var.projeto}-${var.candidato}-route_table_association"
  }
}
```
- Um **Internet Gateway** é criado e anexado à VPC.
- A **Tabela de Rotas** permite o tráfego para a internet via o Internet Gateway (`0.0.0.0/0`).
- A **associação** da tabela de rotas com a subnet garante que essa subnet tenha acesso à Internet.

---

### 7. **Security Group**
```bash
resource "aws_security_group" "main_sg" {
  name        = "${var.projeto}-${var.candidato}-sg"
  description = "Permitir SSH de qualquer lugar e todo o tráfego de saída"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    description      = "Allow SSH from anywhere"
    from_port        = 22
    to_port          = 22
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  egress {
    description      = "Allow all outbound traffic"
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "${var.projeto}-${var.candidato}-sg"
  }
}
```
- Um **Security Group** é configurado para permitir conexões SSH (porta 22) de qualquer lugar (`0.0.0.0/0`).
- Todo o tráfego de saída é permitido, sem restrições.

---

### 8. **Busca de AMI do Debian 12**
```bash
data "aws_ami" "debian12" {
  most_recent = true

  filter {
    name   = "name"
    values = ["debian-12-amd64-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["679593333241"]
}
```
O Terraform busca a **AMI** mais recente do **Debian 12** (versão amd64) usando filtros. Essa AMI será utilizada para lançar a instância EC2.

---

### 9. **Criação da Instância EC2**
```bash
resource "aws_instance" "debian_ec2" {
  ami             = data.aws_ami.debian12.id
  instance_type   = "t2.micro"
  subnet_id       = aws_subnet.main_subnet.id
  key_name        = aws_key_pair.ec2_key_pair.key_name
  security_groups = [aws_security_group.main_sg.name]

  associate_public_ip_address = true

  root_block_device {
    volume_size           = 20
    volume_type           = "gp2"
    delete_on_termination = true
  }

  user_data = <<-EOF
              #!/bin/bash
              apt-get update -y
              apt-get upgrade -y
              EOF

  tags = {
    Name = "${var.projeto}-${var.candidato}-ec2"
  }
}
```
Uma instância **EC2** é criada:
- AMI do **Debian 12**.
- Tipo de instância **t2.micro** (coberto pelo nível gratuito da AWS).
- Associada à **subnet** e com um **IP público**.
- A instância é configurada com um volume de 20GB (gp2).
- O script de inicialização atualiza e atualiza o sistema automaticamente ao iniciar.
  
---

### 10. **Outputs**
```bash
output "private_key" {
  description = "Chave privada para acessar a instância EC2"
  value       = tls_private_key.ec2_key.private_key_pem
  sensitive   = true
}

output "ec2_public_ip" {
  description = "Endereço IP público da instância EC2"
  value       = aws_instance.debian_ec2.public_ip
}
```
Os **outputs** fornecem:
- A **chave privada** gerada para acessar a instância.
- O **endereço IP público** da instância EC2.

---

### Observações:
1. **Segurança:**
   - O Security Group permite SSH de qualquer lugar, o que pode ser uma brecha de segurança. Uma solução mais segura seria restringir o acesso a um IP específico.
   
2. **Erro:**
   - A resource "aws_route_table_association" não suporta tags diretamente, ao rodar o main.tf, provavelmente teríamos um problema com a associação entre a subnet e a route-table.

