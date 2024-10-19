# Análise Técnica do Código Terraform

O código Terraform fornecido realiza a criação de uma infraestrutura básica na AWS, que inclui os seguintes recursos:

- **VPC**: Uma VPC é criada com um bloco CIDR de `10.0.0.0/16`.
- **Subnet**: Uma subnet é criada dentro da VPC com um bloco CIDR de `10.0.1.0/24` na zona de disponibilidade `us-east-1a`.
- **Internet Gateway**: Um gateway de internet é associado à VPC, permitindo que a subnet tenha acesso à internet.
- **Route Table**: Uma tabela de rotas é criada para direcionar o tráfego da subnet para o gateway de internet.
- **Security Group**: Um grupo de segurança é configurado para permitir conexões SSH (porta 22) de um IP específico, e permite todo o tráfego de saída.
- **Key Pair**: Uma chave RSA é gerada para acesso SSH à instância EC2.
- **Instância EC2**: Uma instância EC2 é criada usando a imagem mais recente do Debian 12, configurada para instalação automática do Nginx.

# Modificações e Melhorias do Código Terraform

## Melhorias de Segurança
- O acesso SSH foi restrito para um IP específico, aumentando a segurança da instância.
  
## Automação da Instalação do Nginx
- A instância EC2 agora instala e inicia o servidor Nginx automaticamente através do script `user_data`.

---

# Instruções de Uso

1. **Pré-requisitos**: Certifique-se de ter o Terraform e as credenciais da AWS configuradas em sua máquina.
  
2. **Inicialização**:
   - Navegue até o diretório onde o arquivo `main.tf` está localizado.
   - Execute o comando:
     ```bash
     terraform init
     ```

3. **Planejamento**:
   - Execute o comando:
     ```bash
     terraform plan
     ```

4. **Aplicação**:
   - Execute o comando:
     ```bash
     terraform apply
     ```
   - Confirme a aplicação quando solicitado.

5. **Acesso à Instância**:
   - Utilize a chave privada gerada para acessar a instância EC2 via SSH.

## Nota
- Lembre-se de substituir `SEU_ENDERECO_IP_AQUI` pelo seu endereço IP atual para acessar a instância.
