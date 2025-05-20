# azure_repositorio

# 🚀 Repositório de Estudos Microsoft Azure

## 📋 Índice
- [Introdução ao Azure](#introdução-ao-azure)
- [Criação de Máquinas Virtuais](#criação-de-máquinas-virtuais)
- [Configuração e Gerenciamento](#configuração-e-gerenciamento)
- [Segurança e Rede](#segurança-e-rede)
- [Monitoramento e Backup](#monitoramento-e-backup)
- [Dicas Práticas](#dicas-práticas)
- [Comandos Úteis](#comandos-úteis)
- [Troubleshooting](#troubleshooting)

---

## 🌟 Introdução ao Azure

### O que é Microsoft Azure?
O Microsoft Azure é uma plataforma de computação em nuvem que oferece serviços de infraestrutura como serviço (IaaS), plataforma como serviço (PaaS) e software como serviço (SaaS).

### Principais Vantagens
- **Escalabilidade**: Recursos podem ser aumentados ou diminuídos conforme demanda
- **Disponibilidade**: SLA de até 99.99% para VMs
- **Segurança**: Múltiplas camadas de proteção
- **Economia**: Modelo de pagamento por uso

### Conceitos Fundamentais
- **Resource Group**: Container lógico para recursos relacionados
- **Subscription**: Unidade de cobrança e limite administrativo
- **Region**: Localização geográfica dos data centers
- **Availability Zone**: Data centers fisicamente separados dentro de uma região

---

## 💻 Criação de Máquinas Virtuais

### Pré-requisitos
1. Conta Microsoft Azure ativa
2. Subscription com créditos disponíveis
3. Permissões adequadas (Contributor ou Owner)

### Processo Passo a Passo

#### 1. Acesso ao Portal Azure
```
1. Acesse portal.azure.com
2. Faça login com suas credenciais
3. Navegue até "Virtual machines" no menu principal
```

#### 2. Criação da VM

##### Configurações Básicas
- **Nome da VM**: Escolha um nome descritivo (ex: vm-web-prod-01)
- **Região**: Selecione próxima aos usuários finais
- **Imagem**: Windows Server, Ubuntu, CentOS, etc.
- **Tamanho**: Baseado na necessidade de CPU/RAM

##### Tamanhos Recomendados por Uso
| Uso | Tamanho Sugerido | vCPUs | RAM |
|-----|------------------|-------|-----|
| Desenvolvimento | B2s | 2 | 4GB |
| Aplicações Web | D2s_v3 | 2 | 8GB |
| Banco de Dados | E4s_v3 | 4 | 32GB |
| HPC | H16 | 16 | 112GB |

#### 3. Configuração de Rede
```
Virtual Network: Crie uma nova ou use existente
Subnet: Defina o range de IPs
Public IP: Necessário para acesso externo
Network Security Group: Firewall da VM
```

#### 4. Configuração de Armazenamento
- **OS Disk**: SSD Premium para melhor performance
- **Data Disks**: Adicione conforme necessidade
- **Disk Encryption**: Recomendado para dados sensíveis

### Script de Criação via Azure CLI
```bash
# Login no Azure
az login

# Criar Resource Group
az group create --name rg-lab-vm --location eastus

# Criar VM Linux
az vm create \
  --resource-group rg-lab-vm \
  --name vm-ubuntu-lab \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --size Standard_B2s

# Criar VM Windows
az vm create \
  --resource-group rg-lab-vm \
  --name vm-windows-lab \
  --image Win2022Datacenter \
  --admin-username azureuser \
  --admin-password MinhaSenh@123! \
  --size Standard_B2s
```

---

## ⚙️ Configuração e Gerenciamento

### Conexão à VM

#### Linux (SSH)
```bash
# Conectar via SSH
ssh azureuser@<public-ip>

# Conectar com chave específica
ssh -i ~/.ssh/id_rsa azureuser@<public-ip>
```

#### Windows (RDP)
1. Baixe o arquivo RDP do portal
2. Use Remote Desktop Connection
3. Ou conecte via PowerShell:
```powershell
mstsc /v:<public-ip>
```

### Configurações Pós-Criação

#### Atualizações do Sistema
```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y

# CentOS/RHEL
sudo yum update -y

# Windows (PowerShell)
Install-Module PSWindowsUpdate
Get-WUInstall -AcceptAll -AutoReboot
```

#### Configuração de Firewall
```bash
# Ubuntu - UFW
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443

# CentOS - firewalld
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### Redimensionamento de VM
```bash
# Listar tamanhos disponíveis
az vm list-sizes --location eastus

# Redimensionar VM
az vm resize --resource-group rg-lab-vm --name vm-ubuntu-lab --size Standard_D2s_v3
```

---

## 🔒 Segurança e Rede

### Network Security Groups (NSG)

#### Regras Básicas de Segurança
```bash
# Criar NSG
az network nsg create --resource-group rg-lab-vm --name nsg-web

# Adicionar regra SSH (apenas IP específico)
az network nsg rule create \
  --resource-group rg-lab-vm \
  --nsg-name nsg-web \
  --name AllowSSH \
  --priority 1000 \
  --source-address-prefixes "203.0.113.0/24" \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp

# Regra para HTTP
az network nsg rule create \
  --resource-group rg-lab-vm \
  --nsg-name nsg-web \
  --name AllowHTTP \
  --priority 1010 \
  --destination-port-ranges 80 \
  --access Allow \
  --protocol Tcp
```

### Autenticação e Acesso

#### Configuração de Chaves SSH
```bash
# Gerar chave SSH
ssh-keygen -t rsa -b 4096 -C "seuemail@exemplo.com"

# Copiar chave pública para VM
ssh-copy-id azureuser@<public-ip>
```

#### Azure Active Directory Integration
- Habilite login via Azure AD para gerenciamento centralizado
- Configure RBAC (Role-Based Access Control)
- Use Managed Identity para aplicações

### Criptografia
- **Disk Encryption**: Azure Disk Encryption (ADE)
- **Key Vault**: Armazenamento seguro de chaves
- **SSL/TLS**: Certificados gerenciados

---

## 📊 Monitoramento e Backup

### Azure Monitor

#### Métricas Importantes
- CPU Percentage
- Memory Usage
- Disk IOPS
- Network In/Out
- Available Memory

#### Configuração de Alertas
```bash
# Criar alerta para CPU alta
az monitor metrics alert create \
  --name "High CPU Alert" \
  --resource-group rg-lab-vm \
  --scopes /subscriptions/{sub-id}/resourceGroups/rg-lab-vm/providers/Microsoft.Compute/virtualMachines/vm-ubuntu-lab \
  --condition "avg Percentage CPU > 80" \
  --description "CPU usage is above 80%"
```

### Backup e Recovery

#### Azure Backup
```bash
# Criar Recovery Services Vault
az backup vault create \
  --resource-group rg-lab-vm \
  --name backup-vault-lab \
  --location eastus

# Habilitar backup para VM
az backup protection enable-for-vm \
  --resource-group rg-lab-vm \
  --vault-name backup-vault-lab \
  --vm vm-ubuntu-lab \
  --policy-name DefaultPolicy
```

#### Snapshots Manuais
```bash
# Criar snapshot do disco
az snapshot create \
  --resource-group rg-lab-vm \
  --name snapshot-vm-ubuntu-lab \
  --source vm-ubuntu-lab
```

---

## 💡 Dicas Práticas

### Otimização de Custos
1. **Auto-shutdown**: Configure desligamento automático para VMs de desenvolvimento
2. **Reserved Instances**: Para workloads previsíveis (economia até 72%)
3. **Spot Instances**: Para workloads tolerantes a interrupções (economia até 90%)
4. **Right-sizing**: Monitore uso e ajuste tamanho da VM

### Performance
1. **Premium SSD**: Use para aplicações críticas
2. **Accelerated Networking**: Habilite para melhor performance de rede
3. **Proximity Placement Groups**: Para comunicação entre VMs
4. **VM Extensions**: Para configuração automatizada

### Automação
```bash
# Script de inicialização para instalar Docker
#!/bin/bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker azureuser
sudo systemctl enable docker
sudo systemctl start docker
```

### Organização
- Use convenções de nomenclatura consistentes
- Organize recursos em Resource Groups lógicos
- Aplique tags para categorização e billing
- Documente arquiteturas e configurações

---

## 🔧 Comandos Úteis

### Azure CLI Essenciais
```bash
# Login e configuração
az login
az account set --subscription "nome-da-subscription"

# Listar recursos
az vm list --output table
az group list --output table

# Status da VM
az vm get-instance-view --resource-group rg-lab-vm --name vm-ubuntu-lab

# Start/Stop/Restart VM
az vm start --resource-group rg-lab-vm --name vm-ubuntu-lab
az vm stop --resource-group rg-lab-vm --name vm-ubuntu-lab
az vm restart --resource-group rg-lab-vm --name vm-ubuntu-lab

# Deallocate (para economizar custos)
az vm deallocate --resource-group rg-lab-vm --name vm-ubuntu-lab

# Informações de IP
az vm list-ip-addresses --resource-group rg-lab-vm --name vm-ubuntu-lab
```

### PowerShell para Azure
```powershell
# Conectar ao Azure
Connect-AzAccount

# Listar VMs
Get-AzVM

# Informações da VM
Get-AzVM -ResourceGroupName "rg-lab-vm" -Name "vm-windows-lab"

# Start/Stop VM
Start-AzVM -ResourceGroupName "rg-lab-vm" -Name "vm-windows-lab"
Stop-AzVM -ResourceGroupName "rg-lab-vm" -Name "vm-windows-lab" -Force
```

---

## 🚨 Troubleshooting

### Problemas Comuns

#### VM não inicia
1. Verificar quota de vCPUs na região
2. Validar configurações de rede
3. Checar logs de boot diagnostics

#### Não consegue conectar via SSH/RDP
1. Verificar NSG rules
2. Confirmar IP público atribuído
3. Validar credenciais de acesso
4. Testar conectividade de rede

#### Performance baixa
1. Verificar métricas de CPU/Memory
2. Analisar IOPS do disco
3. Revisar tamanho da VM
4. Verificar throttling de rede

### Logs e Diagnósticos
```bash
# Habilitar boot diagnostics
az vm boot-diagnostics enable \
  --resource-group rg-lab-vm \
  --name vm-ubuntu-lab

# Ver serial console output
az vm boot-diagnostics get-boot-log \
  --resource-group rg-lab-vm \
  --name vm-ubuntu-lab
```

### Recuperação de Acesso
```bash
# Reset de senha via CLI
az vm user update \
  --resource-group rg-lab-vm \
  --name vm-ubuntu-lab \
  --username azureuser \
  --password NovaSenha123!

# Adicionar nova chave SSH
az vm user update \
  --resource-group rg-lab-vm \
  --name vm-ubuntu-lab \
  --username azureuser \
  --ssh-key-value "$(cat ~/.ssh/id_rsa.pub)"
```

---

## 📚 Recursos Adicionais

### Documentação Oficial
- [Azure Virtual Machines Documentation](https://docs.microsoft.com/azure/virtual-machines/)
- [Azure CLI Reference](https://docs.microsoft.com/cli/azure/)
- [Azure PowerShell Documentation](https://docs.microsoft.com/powershell/azure/)

### Certificações Relacionadas
- AZ-104: Microsoft Azure Administrator
- AZ-305: Microsoft Azure Solutions Architect Expert
- AZ-900: Microsoft Azure Fundamentals

### Ferramentas Recomendadas
- Azure Storage Explorer
- Azure Data Studio
- Visual Studio Code com extensão Azure
- Terraform para Infrastructure as Code

---

## 📝 Checklist de Implementação

### Pré-deployment
- [ ] Definir requisitos de recurso
- [ ] Escolher região adequada
- [ ] Planejar arquitetura de rede
- [ ] Definir estratégia de backup
- [ ] Configurar monitoramento

### Durante deployment
- [ ] Validar configurações de rede
- [ ] Configurar grupos de segurança
- [ ] Implementar autenticação
- [ ] Testar conectividade
- [ ] Aplicar tags organizacionais

### Pós-deployment
- [ ] Configurar backup automático
- [ ] Implementar monitoramento
- [ ] Testar procedimentos de recovery
- [ ] Documentar configurações
- [ ] Treinar equipe de suporte

---

**🎯 Objetivo do Laboratório Concluído**: Este repositório serve como guia completo para criação, configuração e gerenciamento de máquinas virtuais no Microsoft Azure, fornecendo base sólida para implementações futuras e estudos contínuos da plataforma.
