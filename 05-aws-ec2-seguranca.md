# 05 — Segurança no EC2

## 1. Por que a segurança é crucial

A segurança no EC2 é essencial para garantir a **confidencialidade**, a **integridade** e a **disponibilidade** dos seus recursos de computação na nuvem. Esses três princípios formam a chamada **Tríade CID** (em inglês, **CIA Triad**).

| Princípio | O que significa | Como o EC2 ajuda |
|---|---|---|
| **Confidencialidade** | Só quem tem autorização acessa os dados | Security Groups, pares de chaves, IAM, criptografia |
| **Integridade** | Os dados não são alterados sem autorização | IAM (controla quem pode modificar), logs de auditoria |
| **Disponibilidade** | Os recursos estão acessíveis quando necessário | Regras de acesso bem configuradas, proteção contra ataques, backups (snapshots) |

Os **três principais itens** de segurança do EC2 são:

| Item | Protege o quê | Pergunta que responde |
|---|---|---|
| **Security Groups** | A **rede** (tráfego que entra e sai) | "De onde e por qual porta podem acessar a instância?" |
| **Pares de chaves** | O **login** no sistema operacional | "Quem pode entrar na máquina?" |
| **IAM** | As **ações na AWS** | "Quem pode criar, desligar ou excluir instâncias? O que a instância pode acessar na AWS?" |

> 💡 **Aprofundamento — Modelo de Responsabilidade Compartilhada**
>
> Na AWS, a segurança é **dividida** entre a AWS e o cliente:
>
> | **AWS**: segurança **DA** nuvem | **Você**: segurança **NA** nuvem |
> |---|---|
> | Datacenters e segurança física | Configuração dos **Security Groups** |
> | Hardware, rede física | Gestão dos **pares de chaves** |
> | Virtualização (hypervisor) | Permissões do **IAM** |
> | | **Atualizações do sistema operacional** (patches) |
> | | **Criptografia** dos dados (ex.: volumes EBS) |
> | | Segurança da **aplicação** |
>
> No EC2, como você controla o servidor inteiro, **a maior parte da segurança é responsabilidade sua**. Esse tema cai muito em prova.

---

## 2. Security Groups (grupos de segurança)

### 2.1. O que são
- Atuam como **firewalls virtuais** que controlam o **tráfego de entrada e de saída** das instâncias EC2.
- São fundamentais para definir **regras de acesso**, que determinam **quem pode e quem não pode** acessar as instâncias.

### 2.2. Comportamento padrão: nascem com o mínimo
Um Security Group novo vem assim:

| Direção | Padrão | O que significa |
|---|---|---|
| **Entrada (inbound)** | ❌ **Tudo bloqueado** | Ninguém consegue acessar a instância até você **liberar** |
| **Saída (outbound)** | ✅ **Tudo liberado** | A instância consegue acessar qualquer destino (ex.: baixar atualizações) |

> ⚠️ **Precisão na anotação original**
>
> O Security Group nasce com o mínimo **na entrada**: todo acesso precisa ser liberado manualmente. Já a **saída vem toda liberada** por padrão.

### 2.3. Regras de entrada (inbound)
Cada regra de entrada é formada por três critérios:

| Critério | Exemplo |
|---|---|
| **Protocolo** | TCP, UDP, ICMP (ping) ou todos |
| **Porta** | `22` (SSH), `443` (HTTPS) ou uma faixa (`8000-8100`) |
| **Origem** | De onde o tráfego pode vir (veja a tabela abaixo) |

**Opções de origem:**

| Opção | Exemplo | Quando usar |
|---|---|---|
| **Um IP específico** | `203.0.113.10/32` | Liberar só o seu IP (ex.: acesso SSH do seu computador) |
| **Uma faixa de IPs (range)** | `203.0.113.0/24` | Liberar a rede de um escritório inteiro |
| **Uma subnet** | `10.0.1.0/24` | Liberar só as máquinas de uma subnet da VPC |
| **Outro Security Group** | `sg-0abc123...` | Liberar qualquer instância que use aquele Security Group (veja o aprofundamento abaixo) |
| **Público (qualquer IP)** | `0.0.0.0/0` (IPv4) e `::/0` (IPv6) | Só para serviços **públicos**, como um site nas portas 80 e 443 |

> 💡 **Aprofundamento — entendendo a notação CIDR (`/32`, `/24`...)**
>
> O número depois da barra define **quantos IPs** a regra cobre. Quanto **maior** o número, **menor** a faixa.
>
> | CIDR | Quantidade de IPs | Exemplo |
> |---|---|---|
> | `/32` | **1 IP** | `203.0.113.10/32` → só esse IP |
> | `/24` | 256 IPs | `203.0.113.0/24` → de `.0` a `.255` |
> | `/16` | 65.536 IPs | `10.0.0.0/16` → uma VPC inteira |
> | `/0` | **Todos os IPs** | `0.0.0.0/0` → a internet inteira |
>
> Explicação completa, com cálculos e exercícios, em [10 — CIDR e Endereçamento IP](10-cidr-enderecamento-ip.md).

### 2.4. Regras de saída (outbound)
- Funcionam do **mesmo jeito** que as de entrada (protocolo, porta e **destino**).
- Por padrão, a saída é **toda liberada**.
- Em ambientes mais restritos, dá para **limitar a saída** (ex.: o servidor só pode acessar o banco de dados e a porta 443).

### 2.5. Características importantes

| Característica | Detalhe |
|---|---|
| **Stateful** (com estado) | Se uma **entrada** foi liberada, a **resposta** sai automaticamente, sem precisar de regra de saída. O contrário também vale. |
| **Só regras de permissão** | Só existem regras de **liberar** (allow). Não dá para criar uma regra de **bloquear** (deny): tudo o que não foi liberado já está bloqueado. |
| **Vários por instância** | Uma instância pode ter **vários Security Groups**, e as regras de todos se **somam**. |
| **Reutilizável** | Um Security Group pode ser usado em **várias instâncias**. |
| **Efeito imediato** | Mudanças nas regras valem **na hora**, sem reiniciar a instância. |
| **Escopo** | Pertence a uma **VPC** em uma **região**. |

### 2.6. Exemplo prático: servidor web

| Direção | Protocolo | Porta | Origem/Destino | Motivo |
|---|---|---|---|---|
| Entrada | TCP | 80 | `0.0.0.0/0` | Site público (HTTP) |
| Entrada | TCP | 443 | `0.0.0.0/0` | Site público (HTTPS) |
| Entrada | TCP | 22 | `SEU_IP/32` | SSH **só do seu IP** |
| Saída | Todos | Todas | `0.0.0.0/0` | Padrão (atualizações, APIs externas) |

### 2.7. Portas mais comuns

| Porta | Serviço |
|---|---|
| **22** | SSH (acesso remoto Linux) |
| **3389** | RDP (acesso remoto Windows) |
| **80** | HTTP |
| **443** | HTTPS |
| **3306** | MySQL / MariaDB |
| **5432** | PostgreSQL |
| **1433** | SQL Server |
| **6379** | Redis |
| **27017** | MongoDB |

### 2.8. Boas práticas
- **Nunca** deixe as portas **22 (SSH)** ou **3389 (RDP)** abertas para `0.0.0.0/0`. São alvo de ataques automatizados o tempo todo.
- **Libere só o necessário** (princípio do **menor privilégio**): só as portas e as origens que a aplicação realmente usa.
- **Bancos de dados nunca devem ser públicos.** Libere a porta do banco só para o Security Group da aplicação.
- Use **nomes e descrições claros** nos Security Groups e nas regras (ex.: `sg-web-producao`, "HTTPS público").
- **Revise as regras periodicamente** e remova as que não são mais usadas.

> 💡 **Aprofundamento — encadeamento de Security Groups (arquitetura em camadas)**
>
> Em vez de liberar IPs, você pode usar **um Security Group como origem de outro**:
>
> ```
>  Internet ──443──▶ [ sg-web ] ──8080──▶ [ sg-app ] ──5432──▶ [ sg-banco ]
>                  (origem: 0.0.0.0/0)  (origem: sg-web)    (origem: sg-app)
> ```
>
> - O **sg-app** só aceita tráfego de instâncias que estão no **sg-web**.
> - O **sg-banco** só aceita tráfego de instâncias que estão no **sg-app**.
>
> **Vantagem:** se novas instâncias forem criadas (ex.: Auto Scaling), elas já entram com o acesso certo, **sem precisar atualizar IPs**.

> 💡 **Aprofundamento — Security Group vs Network ACL (cai muito em prova)**
>
> A **Network ACL (NACL)** é outro firewall da AWS, que atua na **subnet**:
>
> | | **Security Group** | **Network ACL** |
> |---|---|---|
> | **Nível** | **Instância** | **Subnet** |
> | **Estado** | **Stateful**: a resposta volta automaticamente | **Stateless**: precisa liberar a entrada **e** a saída |
> | **Tipos de regra** | Só **permitir** (allow) | **Permitir e negar** (allow e deny) |
> | **Avaliação** | **Todas** as regras são avaliadas juntas | Em **ordem numérica**: a primeira que combinar vale |
> | **Uso típico** | Controle principal de acesso | Camada extra, ex.: **bloquear um IP malicioso** |

> 💡 **Dica de diagnóstico**
>
> - Conexão dá **timeout** (fica tentando e não conecta) → geralmente é o **Security Group** bloqueando.
> - Conexão dá **"connection refused"** (recusa na hora) → o Security Group liberou, mas a **aplicação** não está rodando ou não está ouvindo naquela porta.

---

## 3. Pares de chaves (Key Pairs)

### 3.1. O que são
Um par de chaves é formado por **duas chaves criptográficas** usadas para **acessar a instância com segurança**, sem senha:

| Chave | Onde fica | Função |
|---|---|---|
| **Chave pública** | **Na AWS e dentro da instância**. No Linux, fica no arquivo `~/.ssh/authorized_keys`. | Funciona como a **fechadura** |
| **Chave privada** | **Só com você**, em um arquivo `.pem` ou `.ppk` | Funciona como a **chave** que abre a fechadura |

> ⚠️ **Ponto mais importante**
>
> A AWS guarda **apenas a chave pública**. A **chave privada é baixada uma única vez**, no momento da criação. **A AWS não guarda uma cópia**: se você perder, ela **não pode ser recuperada**.

> ⚠️ **Ajuste na anotação original**
>
> A ideia de que a chave pública "tranca" e só a privada "abre" está certa. Tecnicamente:
> - **No Linux (SSH):** a chave privada não "descriptografa" algo. Ela é usada para **provar sua identidade**: o servidor envia um desafio, o seu computador **assina** com a chave privada, e o servidor confere a assinatura com a chave pública.
> - **No Windows:** aí sim, a chave privada é usada para **descriptografar a senha do Administrador**, que a AWS gera criptografada com a sua chave pública.

### 3.2. Tipos e formatos

| Item | Opções |
|---|---|
| **Tipo de chave** | **RSA** ou **ED25519** (mais moderna; não funciona em instâncias Windows) |
| **Formato** | **`.pem`** (OpenSSH: Linux, macOS, Windows 10+) ou **`.ppk`** (PuTTY) |

### 3.3. Como conectar (Linux)

```bash
ssh -i minha-chave.pem ec2-user@IP_PUBLICO_DA_INSTANCIA
```

O usuário padrão depende do sistema operacional:

| Sistema | Usuário |
|---|---|
| Amazon Linux / RHEL / SUSE | `ec2-user` |
| Ubuntu | `ubuntu` |
| Debian | `admin` |

### 3.4. Armazenamento seguro e permissões

**Onde guardar a chave privada:**
- Em um **diretório seguro** no seu computador, como `~/.ssh/`.
- Em um **gerenciador de senhas ou de segredos**: Bitwarden, 1Password, HashiCorp Vault ou **AWS Secrets Manager** (para equipes e automações).

**Permissões do arquivo:** é preciso **tirar o acesso de todos os outros usuários**, deixando só você com permissão de leitura.

**Linux / macOS:**
```bash
chmod 400 minha-chave.pem
```

**Windows (PowerShell):**
```powershell
icacls.exe minha-chave.pem /reset
icacls.exe minha-chave.pem /grant:r "$($env:USERNAME):(R)"
icacls.exe minha-chave.pem /inheritance:r
```

> ⚠️ **Se as permissões estiverem abertas demais, o SSH recusa a chave** com este erro:
> ```
> WARNING: UNPROTECTED PRIVATE KEY FILE!
> Permissions 0644 for 'minha-chave.pem' are too open.
> ```
> A solução é rodar o `chmod 400`.

> 💡 **Aprofundamento — o que o `chmod 400` faz**
>
> | Dígito | Para quem | `4` = só leitura, `0` = nenhum acesso |
> |---|---|---|
> | **4** | Dono do arquivo (você) | Pode **ler** |
> | **0** | Grupo | **Nenhum** acesso |
> | **0** | Outros usuários | **Nenhum** acesso |

### 3.5. Boas práticas
- **Nunca compartilhe** a chave privada por e-mail, chat ou WhatsApp.
- **Nunca envie** a chave para o **Git** ou o GitHub. Adicione `*.pem` ao `.gitignore`.
- Use **chaves diferentes** por ambiente (desenvolvimento, produção) e, se possível, **por pessoa**.
- **Troque as chaves periodicamente** e **remova** as chaves de quem saiu da equipe (apagando a linha correspondente no `authorized_keys`).

> 💡 **Aprofundamento — perdi a chave privada, e agora?**
>
> A AWS não consegue recuperar a chave, mas dá para recuperar o acesso:
> 1. **EC2 Instance Connect** ou **Session Manager** (veja abaixo), se estiverem configurados.
> 2. **Pelo volume EBS:** desligar a instância, desconectar o volume raiz, conectar em outra instância, adicionar uma **nova chave pública** no arquivo `authorized_keys` e devolver o volume para a instância original.

> 💡 **Aprofundamento — acessar sem chave SSH (recomendado pela AWS)**
>
> | Opção | Como funciona | Vantagem |
> |---|---|---|
> | **EC2 Instance Connect** | A AWS envia uma **chave temporária** para a instância, válida por **60 segundos** | Acesso controlado pelo **IAM**, sem guardar chave |
> | **Systems Manager Session Manager** | Abre um terminal pelo **console da AWS** ou pela CLI | **Não precisa abrir a porta 22** nem usar chave, e **registra tudo** o que foi feito (auditoria) |
>
> O **Session Manager** é considerado a **opção mais segura**, porque elimina a porta 22 aberta e o risco de vazamento de chave.

---

## 4. IAM no EC2

O **IAM (Identity and Access Management)** controla **quem pode fazer o quê** na AWS. No EC2, ele atua em duas frentes:

| Frente | Exemplo |
|---|---|
| **Quem pode gerenciar as instâncias** | O estagiário pode só **ver** as instâncias. O time de infraestrutura pode **criar, desligar e excluir**. |
| **O que a instância pode acessar** | A instância da aplicação pode **ler arquivos de um bucket S3**, mas não pode apagar nada. |

> ⚠️ **Boa prática essencial: use IAM Roles, nunca chaves de acesso dentro da instância**
>
> - ❌ **Errado:** colocar `AWS_ACCESS_KEY_ID` e `AWS_SECRET_ACCESS_KEY` em arquivos dentro do servidor. Se o servidor for invadido, as chaves vazam.
> - ✅ **Certo:** associar uma **IAM Role** à instância (via *instance profile*). A instância recebe **credenciais temporárias**, que a AWS **troca automaticamente**.

> 💡 **Aprofundamento — IMDSv2**
>
> As credenciais da IAM Role chegam à instância pelo **serviço de metadados (IMDS)**. Use sempre a **versão 2 (IMDSv2)**, que exige um token de sessão e protege contra ataques que tentam roubar essas credenciais (como o SSRF).

Veja o IAM em detalhes em [06 — IAM](06-aws-iam.md).

---

## 5. Checklist de segurança do EC2

- [ ] Security Groups com **só as portas necessárias** liberadas
- [ ] **SSH (22) e RDP (3389)** liberados só para IPs específicos, ou acesso pelo **Session Manager**
- [ ] **Banco de dados** acessível só pelo Security Group da aplicação
- [ ] Chave privada guardada em **local seguro**, com permissão **`chmod 400`**
- [ ] Chaves **fora do Git**
- [ ] **IAM Role** na instância em vez de chaves de acesso
- [ ] **IMDSv2** obrigatório
- [ ] Volumes EBS **criptografados** (veja [03 — EBS](03-aws-ebs.md))
- [ ] Sistema operacional **atualizado** (patches)

---

## 6. Revisão rápida

- **Quais são os três princípios da Tríade CID?**
  Confidencialidade, integridade e disponibilidade.
- **Quais são os três principais itens de segurança do EC2?**
  Security Groups, pares de chaves e IAM.
- **Qual é o comportamento padrão de um Security Group novo?**
  Entrada toda bloqueada e saída toda liberada.
- **Quais são os três critérios de uma regra de entrada?**
  Protocolo, porta e origem.
- **O que significa dizer que o Security Group é "stateful"?**
  Que, se a entrada foi liberada, a resposta sai automaticamente, sem precisar de regra de saída.
- **Dá para criar uma regra de bloqueio (deny) em um Security Group?**
  Não, só regras de permissão. Para bloquear um IP específico, use uma Network ACL.
- **O que significa a origem `0.0.0.0/0`? E `/32`?**
  `0.0.0.0/0` é qualquer IP (público). `/32` é um único IP.
- **Quem guarda a chave pública e quem guarda a chave privada?**
  A pública fica na AWS e dentro da instância. A privada fica só com você: a AWS não guarda cópia.
- **Qual comando corrige o erro "UNPROTECTED PRIVATE KEY FILE"?**
  `chmod 400 minha-chave.pem`
- **Qual é a forma mais segura de acessar uma instância sem abrir a porta 22?**
  O Systems Manager Session Manager.
- **Como dar permissão para uma instância acessar o S3 sem colocar chaves de acesso nela?**
  Associando uma IAM Role à instância.
- **A conexão com a instância dá timeout. Qual é o primeiro lugar para verificar?**
  O Security Group.
