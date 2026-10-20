# 06 — AWS IAM (Identity and Access Management)

## 1. O que é o IAM

O **IAM (Identity and Access Management)** é o **serviço central** da AWS para definir **quem pode fazer o quê** dentro da conta.

- Vale para o **EC2** e para **todos os outros serviços** da AWS (S3, RDS, Lambda...).
- É **global**: não depende de região. Um usuário criado no IAM vale em todas as regiões.
- É **gratuito**.

> 💡 **Aprofundamento — autenticação vs autorização**
>
> O IAM cuida de duas coisas diferentes:
>
> | | **Autenticação** | **Autorização** |
> |---|---|---|
> | **Pergunta** | **Quem é você?** | **O que você pode fazer?** |
> | **Como funciona** | Senha, chave de acesso, MFA | Políticas (permissões) |
> | **Exemplo** | Entrar no console com usuário, senha e MFA | Poder desligar instâncias, mas não excluir |

> ⚠️ **O usuário root**
>
> Quando a conta AWS é criada, nasce o **usuário root**, que é o **e-mail usado no cadastro**. Ele tem **acesso total e irrestrito** a tudo, e não dá para limitar as permissões dele.
>
> - **Não use o root no dia a dia.** Crie usuários ou acessos com as permissões necessárias.
> - **Ative o MFA no root.** A AWS passou a exigir isso na maioria das contas.
> - **Nunca crie chaves de acesso** (access keys) para o root.
> - Use o root só para tarefas que **exigem** ele, como mudar o plano de suporte ou fechar a conta.

---

## 2. Componentes do IAM

| Componente | O que é | Tipo de credencial |
|---|---|---|
| **Usuários** | Uma **pessoa** ou aplicação com acesso à conta | **Permanente** (senha, chaves de acesso) |
| **Grupos** | Um **conjunto de usuários** com as mesmas permissões | Não tem credencial: só organiza os usuários |
| **Funções (Roles)** | Uma **identidade temporária** que pode ser "assumida" por usuários ou serviços | **Temporária** (expira sozinha) |
| **Políticas** | **Documentos JSON** que definem as **permissões** | Não é identidade: é anexada às outras três |

> 💡 **Analogia: o prédio de uma empresa**
>
> - **Usuário** = um **funcionário** com crachá próprio.
> - **Grupo** = um **departamento** (Financeiro, TI). Todo mundo do departamento tem os mesmos acessos.
> - **Role** = um **crachá de visitante**: você pega na portaria, usa por um tempo e devolve.
> - **Política** = a **lista de portas** que cada crachá consegue abrir.

### 2.1. Usuários
- Representam **uma pessoa** (ou, em casos antigos, uma aplicação).
- Podem ter dois tipos de credencial:
  - **Senha**: para acessar o **console** (site da AWS).
  - **Chaves de acesso** (Access Key ID + Secret Access Key): para acessar pela **CLI**, por **SDKs** e pela **API**.
- Um usuário novo **nasce sem nenhuma permissão**: tudo precisa ser liberado por políticas.
- **Um usuário por pessoa**: nunca compartilhe usuários.

> 💡 **Aprofundamento — a recomendação atual da AWS para pessoas**
>
> Para **pessoas** em empresas, a AWS recomenda o **IAM Identity Center** (antigo AWS SSO) em vez de usuários IAM. Ele permite login único (SSO) com o e-mail corporativo (Google Workspace, Microsoft Entra ID) e usa **credenciais temporárias**. Os usuários IAM continuam válidos para estudo, contas pequenas e casos específicos.

### 2.2. Grupos
- São **perfis** que reúnem usuários com as mesmas necessidades. Exemplos: `Desenvolvedores`, `Administradores`, `Financeiro`, `Auditoria`.
- Você anexa a política **ao grupo**, e **todos os usuários** do grupo recebem as permissões.
- Um usuário pode estar em **vários grupos**, e as permissões se **somam**.
- Grupos **não podem conter outros grupos** (não existe grupo dentro de grupo).

> **Boa prática:** dê permissões **aos grupos**, e não diretamente aos usuários. Fica muito mais fácil de gerenciar: quando alguém entra ou sai da equipe, basta adicionar ou remover a pessoa do grupo.

### 2.3. Funções (Roles)
- São identidades de **caráter temporário**: quem assume uma role recebe **credenciais que expiram sozinhas** (por padrão em 1 hora, configurável até 12 horas).
- **Não têm senha nem chave de acesso permanente.**
- **Quem pode assumir uma role:**
  - **Serviços da AWS** (ex.: uma instância EC2 que precisa ler arquivos do S3)
  - **Usuários** da mesma conta ou de **outra conta AWS**
  - **Usuários externos**, que entram com login do Google, do Facebook ou da empresa (**federação**)

Uma role tem **duas políticas**:

| Política | Pergunta que responde |
|---|---|
| **Política de confiança** (*trust policy*) | **Quem pode assumir** esta role? |
| **Política de permissão** | **O que** esta role pode fazer? |

> ⚠️ **Precisão na anotação original: quando um serviço precisa acessar outro**
>
> Quando um serviço da AWS precisa acessar outro (ex.: o **EC2** precisa ler arquivos do **S3**), o correto é usar uma **role**, e **não criar um usuário** com chaves de acesso para ele.
>
> - ❌ Criar um usuário IAM e colocar as chaves de acesso dentro do servidor.
> - ✅ Criar uma **role**, associar à instância EC2 (via *instance profile*), e a AWS entrega **credenciais temporárias** que são trocadas automaticamente.
>
> Veja também [05 — Segurança no EC2](05-aws-ec2-seguranca.md#4-iam-no-ec2).

### 2.4. Políticas
- São **documentos JSON** que definem as **permissões**: quais **ações** são permitidas ou negadas e em quais **recursos**.
- São o **coração do controle de acesso** do IAM. Políticas bem pensadas são a base de uma conta segura.
- Diferente das roles, que são **temporárias**, as políticas são **permanentes**: ficam anexadas até alguém removê-las.

> ⚠️ **Precisão na anotação original: políticas são "imutáveis"?**
>
> Depende do tipo:
> - **Políticas gerenciadas pela AWS**: **você não pode editar**. Só a própria AWS atualiza.
> - **Políticas gerenciadas pelo cliente**: **podem ser editadas**, e o IAM guarda **até 5 versões**, o que permite voltar para uma versão anterior.
> - **Políticas inline**: podem ser editadas à vontade.

---

## 3. Políticas em detalhe

### 3.1. Estrutura de uma política JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PermitirVerInstancias",
      "Effect": "Allow",
      "Action": "ec2:DescribeInstances",
      "Resource": "*"
    }
  ]
}
```

| Elemento | O que é |
|---|---|
| **`Version`** | Versão da **linguagem de políticas**. Use sempre `"2012-10-17"`: é um valor fixo, não uma data para atualizar. |
| **`Statement`** | Lista de **regras**. Uma política pode ter várias. |
| **`Sid`** | Nome da regra (opcional), para facilitar a leitura. |
| **`Effect`** | **`Allow`** (permitir) ou **`Deny`** (negar). |
| **`Action`** | **Quais ações** da API, no formato `servico:Acao` (ex.: `ec2:StartInstances`). Aceita `*` (ex.: `ec2:Describe*`). |
| **`Resource`** | **Em quais recursos**, identificados pelo **ARN**. `*` significa todos. |
| **`Condition`** | **Condições** extras (opcional): por tag, IP de origem, horário, uso de MFA... |
| **`Principal`** | **Quem** recebe a permissão. Só aparece em políticas de confiança e em políticas de recurso. |

> 💡 **Aprofundamento — o que é um ARN**
>
> O **ARN (Amazon Resource Name)** é o "endereço" único de um recurso na AWS:
>
> ```
> arn:aws:ec2:sa-east-1:123456789012:instance/i-0abc123def456
>     │   │    │          │            └── recurso
>     │   │    │          └── ID da conta
>     │   │    └── região
>     │   └── serviço
>     └── partição (aws)
> ```

### 3.2. Políticas gerenciadas vs inline

| | **Gerenciada pela AWS** | **Gerenciada pelo cliente** | **Inline** |
|---|---|---|---|
| **Quem cria** | A AWS | Você | Você |
| **Reutilizável** | ✅ Em vários usuários, grupos e roles | ✅ Em vários usuários, grupos e roles | ❌ **Escrita diretamente** em **um** usuário, grupo ou role específico |
| **Pode editar** | ❌ | ✅ (com até 5 versões) | ✅ |
| **Se a identidade for excluída** | A política continua existindo | A política continua existindo | A política **é excluída junto** |
| **Exemplos** | `AdministratorAccess`, `ReadOnlyAccess`, `AmazonEC2ReadOnlyAccess` | `Politica-Devs-EC2-Dev` | Uma permissão exclusiva de uma role |
| **Quando usar** | Para começar rápido, em casos comuns | **Uso repetido**, com permissões sob medida | Quando a permissão deve valer **só para aquela identidade**, de forma independente |

> **Recomendação da AWS:** prefira as **políticas gerenciadas pelo cliente**. Elas são reutilizáveis, versionadas e fáceis de auditar. Use **inline** só quando a relação precisa ser estritamente 1 para 1.

> 💡 **Aprofundamento — como a AWS decide se permite ou nega (cai muito em prova)**
>
> 1. **Por padrão, tudo é negado** (negação implícita).
> 2. Se alguma política tiver um **`Allow`** para a ação, ela passa a ser **permitida**.
> 3. Se **qualquer** política tiver um **`Deny`** para a ação, ela é **negada**, mesmo que exista um `Allow`.
>
> **Regra de ouro: um `Deny` explícito sempre vence.**
>
> ```
> Existe Deny explícito? ── Sim ──▶ ❌ NEGADO
>          │
>         Não
>          │
> Existe Allow? ── Sim ──▶ ✅ PERMITIDO
>          │
>         Não
>          │
>          ▼
>    ❌ NEGADO (negação implícita)
> ```

> 💡 **Aprofundamento — outros tipos de política**
>
> | Tipo | Onde fica | Para que serve |
> |---|---|---|
> | **Baseada em identidade** | Anexada a usuário, grupo ou role | É o que vimos até aqui (gerenciadas e inline) |
> | **Baseada em recurso** | Anexada ao **recurso** (ex.: *bucket policy* do S3) | Define quem pode acessar **aquele recurso**, inclusive de outras contas |
> | **Permissions Boundary** | Anexada a usuário ou role | Define o **limite máximo** de permissões que a identidade pode ter |
> | **SCP** (Service Control Policy) | No **AWS Organizations** | Define o limite máximo de permissões de **contas inteiras** |

---

## 4. Segurança no nível de API (crucial para o EC2)

**Toda ação na AWS é uma chamada de API**, seja pelo console, pela CLI ou por código. O IAM verifica **cada chamada** antes de executar.

No EC2, isso controla **quem pode criar, excluir e atualizar** os recursos:

| Ação | Chamada de API |
|---|---|
| Criar uma instância | `ec2:RunInstances` |
| Ligar / desligar | `ec2:StartInstances` / `ec2:StopInstances` |
| Excluir uma instância | `ec2:TerminateInstances` |
| Ver as instâncias | `ec2:DescribeInstances` |
| Criar um Security Group | `ec2:CreateSecurityGroup` |
| Liberar uma porta no Security Group | `ec2:AuthorizeSecurityGroupIngress` |
| Criar um volume / snapshot | `ec2:CreateVolume` / `ec2:CreateSnapshot` |

### Exemplo: política para o time de desenvolvimento

O time pode **ver** todas as instâncias, **ligar, desligar e reiniciar** só as instâncias de desenvolvimento, e **nunca excluir** nenhuma:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VerTodasAsInstancias",
      "Effect": "Allow",
      "Action": "ec2:Describe*",
      "Resource": "*"
    },
    {
      "Sid": "LigarDesligarSoDev",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:RebootInstances"
      ],
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": { "aws:ResourceTag/Ambiente": "dev" }
      }
    },
    {
      "Sid": "NuncaExcluir",
      "Effect": "Deny",
      "Action": "ec2:TerminateInstances",
      "Resource": "*"
    }
  ]
}
```

- A regra 1 usa `"Resource": "*"` porque as ações `Describe` não permitem restringir por recurso.
- A regra 2 só funciona em instâncias com a **tag** `Ambiente = dev`.
- A regra 3 é um **`Deny` explícito**: mesmo que outra política permita, ninguém com esta política consegue excluir instâncias.

### Exemplo: role para o EC2 ler arquivos do S3

**Política de confiança** (quem pode assumir: o serviço EC2):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**Política de permissão** (o que pode fazer: ler um bucket específico):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::meu-bucket",
        "arn:aws:s3:::meu-bucket/*"
      ]
    }
  ]
}
```

> 💡 **Aprofundamento — CloudTrail: quem fez o quê?**
>
> O **AWS CloudTrail** registra **todas as chamadas de API** da conta: **quem** fez, **o quê**, **quando** e **de onde** (IP). Se uma instância for excluída, o CloudTrail mostra exatamente qual usuário ou role chamou o `TerminateInstances`. É a base da **auditoria** na AWS.

---

## 5. Boas práticas de segurança no IAM

### 5.1. Princípio do menor privilégio
- Dê **só as permissões necessárias** para a pessoa ou o serviço fazer o trabalho, **nada além**.
- Comece com o **mínimo** e vá liberando conforme a necessidade.
- **Evite `"Action": "*"` e `"Resource": "*"`** sempre que possível.
- Use **condições** (tags, IP, MFA) para restringir ainda mais.

### 5.2. Auditoria regular
Revise periodicamente quem tem acesso a quê e **remova o que não é usado**.

| Ferramenta | O que mostra |
|---|---|
| **Credential Report** | Lista **todos os usuários** e o status das credenciais: última vez que a senha foi usada, idade das chaves de acesso, se o MFA está ativo |
| **Último acesso** (*Last Accessed*) | **Quais serviços** cada identidade **realmente usou**, e quando. Ajuda a remover permissões que nunca foram usadas. |
| **IAM Access Analyzer** | Encontra **acessos externos** (recursos compartilhados com outras contas) e **permissões sem uso**. Também **gera políticas** com base no que foi realmente usado. |
| **CloudTrail** | Histórico de **todas as chamadas de API** |

### 5.3. Rotação de credenciais
- **Troque as chaves de acesso periodicamente** (ex.: a cada 90 dias).
- Cada usuário pode ter **até 2 chaves de acesso ativas**, justamente para permitir a troca sem interromper o serviço:
  1. Criar a chave nova.
  2. Atualizar a aplicação para usar a chave nova.
  3. Desativar a chave antiga e, depois de confirmar que nada parou, excluir.
- Configure uma **política de senhas** (tamanho mínimo, complexidade, expiração).
- **Melhor ainda:** use **roles** sempre que possível. As credenciais temporárias **expiram sozinhas** e não precisam de rotação manual.

### 5.4. Autenticação multifator (MFA)
- Além da senha, exige um **segundo fator**: algo que você **tem**.
- **Obrigatório no usuário root** e altamente recomendado para todos os usuários, principalmente administradores.

| Tipo de MFA | Exemplo |
|---|---|
| **App autenticador (virtual)** | Google Authenticator, Microsoft Authenticator, Authy |
| **Chave de segurança FIDO2 / Passkey** | YubiKey, passkey no celular ou no computador |
| **Token físico (TOTP)** | Dispositivo que gera códigos de 6 dígitos |

> 💡 **Aprofundamento — exigir MFA para ações críticas**
>
> Dá para criar uma política que **só permite excluir instâncias se o usuário entrou com MFA**, usando a condição `aws:MultiFactorAuthPresent`:
>
> ```json
> "Condition": {
>   "Bool": { "aws:MultiFactorAuthPresent": "true" }
> }
> ```

### 5.5. Outras boas práticas
- **Não use o usuário root** no dia a dia.
- **Um usuário por pessoa**: nunca compartilhe credenciais.
- Dê permissões **por grupo**, e não por usuário.
- Use **roles** para aplicações e serviços, nunca chaves de acesso fixas.
- **Nunca coloque chaves de acesso** no código ou no Git.

---

## 6. Revisão rápida

- **O que é o IAM e qual é o escopo dele?**
  É o serviço que define quem pode fazer o quê na AWS. É global (não depende de região) e gratuito.
- **Quais são os quatro componentes principais do IAM?**
  Usuários, grupos, funções (roles) e políticas.
- **Qual é a diferença entre autenticação e autorização?**
  Autenticação é provar quem você é. Autorização é definir o que você pode fazer.
- **Qual é a principal diferença entre usuário e role?**
  O usuário tem credenciais permanentes. A role fornece credenciais temporárias, que expiram sozinhas.
- **Uma instância EC2 precisa ler arquivos do S3. Qual é a forma correta de dar esse acesso?**
  Criar uma role com permissão de leitura no S3 e associar à instância.
- **Um grupo pode conter outro grupo?**
  Não.
- **Quais são os elementos principais de uma política JSON?**
  `Version`, `Statement`, `Effect`, `Action`, `Resource` e, opcionalmente, `Condition`.
- **Qual é a diferença entre política gerenciada e inline?**
  A gerenciada é reutilizável em várias identidades. A inline é escrita direto em uma única identidade e é excluída junto com ela.
- **Uma política dá `Allow` e outra dá `Deny` para a mesma ação. O que acontece?**
  A ação é negada: um `Deny` explícito sempre vence.
- **Um usuário IAM recém-criado tem alguma permissão?**
  Não. Tudo é negado por padrão até alguma política liberar.
- **Qual serviço registra quem chamou cada API na conta?**
  O AWS CloudTrail.
- **Quais são as quatro boas práticas de IAM da anotação?**
  Menor privilégio, auditoria regular, rotação de credenciais e MFA.
- **Por que cada usuário pode ter até 2 chaves de acesso?**
  Para permitir trocar a chave sem interromper o serviço.
