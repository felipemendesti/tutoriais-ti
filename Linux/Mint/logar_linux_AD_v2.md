# Ingressar o Linux Mint em um domínio Active Directory

Este documento reúne dois métodos para integrar uma estação Linux Mint a um domínio Active Directory/UCS:

1. **CID** — ferramenta gráfica para ingresso no domínio.
2. **Realmd + SSSD** — método padrão usando ferramentas nativas do Linux.

> **Atenção:** adapte os nomes de domínio, usuários e senhas ao seu ambiente. Os exemplos abaixo usam `sixcore.intranet`.

## 1. Método CID

### 1.1. Instalar o CID

Execute os comandos no terminal:

```bash
sudo add-apt-repository -y ppa:emoraes25/cid
sudo apt update
sudo apt install -y cid cid-gtk
```

### 1.2. Abrir a ferramenta

```bash
sudo cid-gtk
```

Preencha os campos solicitados:

| Campo | Descrição |
|---|---|
| `hostname` | Nome da estação Linux |
| `domain` | Nome do domínio, por exemplo `sixcore.intranet` |
| `user` | Usuário administrador do UCS |
| `Password` | Senha do usuário administrador do UCS |

Siga as instruções exibidas pela ferramenta para concluir a integração.

## 2. Método padrão com Realmd e SSSD

### 2.1. Instalar os pacotes necessários

```bash
sudo apt update && apt upgrade -y
sudo apt install -y realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin
```

### 2.2. Descobrir o domínio

Substitua `seudominio.local` pelo domínio do seu ambiente:

```bash
realm discover seudominio.local
```

### 2.3. Ingressar no domínio

No exemplo abaixo, o domínio é `sixcore.intranet`. O comando solicitará a senha do usuário informado:

```bash
sudo realm join -U administrator sixcore.intranet
```

### 2.4. Habilitar a criação automática do diretório pessoal

```bash
sudo pam-auth-update --enable mkhomedir
```

### 2.5. Ajustar a configuração do SSSD

Abra o arquivo de configuração:

```bash
sudo nano /etc/sssd/sssd.conf
```

Na seção apropriada, configure:

```ini
use_fully_qualified_names = False
```

> **Importante:** se a opção já existir, ajuste seu valor em vez de criar uma entrada duplicada. Preserve as demais configurações do arquivo.

### 2.6. Configurar a tela de login do LightDM

Abra ou crie o arquivo:

```bash
sudo nano /etc/lightdm/lightdm.conf.d/90-linuxmint.conf
```

Adicione:

```ini
[SeatDefaults]
greeter-show-manual-login=true
greeter-hide-users=true
allowguest=false
```

### 2.7. Reiniciar a estação

```bash
sudo reboot
```

## 3. Fazer login com uma conta do domínio

Na tela de login do Linux Mint, informe as credenciais da conta do Active Directory. Dependendo da configuração do sistema, use um dos formatos aceitos:

- `usuario@seudominio.local`
- `DOMINIO\usuario`
- Apenas `usuario`, se a configuração do SSSD permitir nomes sem o domínio.

Use a senha da conta do domínio.

## 4. Verificar a integração

Para consultar os domínios configurados e verificar se a estação reconhece a associação, execute:

```bash
realm list
```

Se houver problemas de autenticação, confira o estado do SSSD e os registros do sistema antes de repetir o ingresso no domínio.

## Referências rápidas

- **Ferramenta gráfica:** `cid-gtk`
- **Descoberta do domínio:** `realm discover`
- **Ingresso no domínio:** `realm join`
- **Verificação:** `realm list`
