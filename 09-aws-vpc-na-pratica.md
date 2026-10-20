# 09 — VPC na Prática: Criando a Rede Passo a Passo

Passo a passo para criar uma VPC **manualmente** no console da AWS, com subnets públicas e privadas, Internet Gateway, tabelas de rotas e Security Groups. A teoria de cada componente está em [08 — VPC](08-aws-vpc.md), e a explicação de CIDR está em [10 — CIDR e Endereçamento IP](10-cidr-enderecamento-ip.md).

> Os nomes dos botões aparecem em **português** e, entre parênteses, em *inglês*, porque o console pode estar em qualquer um dos dois idiomas.

---

## O que vamos montar

```
VPC "vpc-estudos"  10.0.0.0/16
├── AZ a
│   ├── subnet-publica-a   10.0.1.0/24   → rt-publica  (tem rota para a internet)
│   └── subnet-privada-a   10.0.11.0/24  → rt-privada  (só rota local)
└── AZ b
    ├── subnet-publica-b   10.0.2.0/24   → rt-publica
    └── subnet-privada-b   10.0.12.0/24  → rt-privada

igw-estudos (Internet Gateway) anexado à VPC

sg-publico  → firewall das instâncias públicas  (80 e 443 da internet)
sg-privado  → firewall das instâncias privadas  (só aceita tráfego do sg-publico)
```

---

## Passo 0 — Escolher a região e ver as zonas de disponibilidade

- No canto superior direito do console, escolha a **região** (ex.: `us-east-1` ou `sa-east-1`).
- Veja **quantas AZs** a região tem. Elas aparecem na lista na hora de criar a subnet, e também no painel do EC2.
- Exemplos: `sa-east-1` (São Paulo) tem **3 AZs** (`sa-east-1a`, `b` e `c`). `us-east-1` (Norte da Virgínia) tem **6**.

> 💡 **Aprofundamento — nome da AZ vs ID da AZ**
>
> O nome `us-east-1a` **pode apontar para datacenters diferentes em contas diferentes**: a AWS embaralha os nomes para distribuir a carga. Para saber a AZ **física**, use o **ID da AZ** (ex.: `use1-az1`), que é igual em todas as contas. Isso importa quando você precisa alinhar recursos entre contas diferentes.

---

## Passo 1 — Criar a VPC

**Caminho:** console da VPC → **Suas VPCs** (*Your VPCs*) → **Criar VPC** (*Create VPC*)

| Campo | O que escolher | Explicação |
|---|---|---|
| **Recursos a serem criados** (*Resources to create*) | **Somente VPC** (*VPC only*) | Cria só a VPC. Vamos criar o resto manualmente para aprender. |
| **Tag de nome** (*Name tag*) | `vpc-estudos` | **Opcional**. A AWS identifica a VPC pelo **ID** (ex.: `vpc-0a1b2c3d4e5f`), e o nome é só uma tag para facilitar. |
| **Bloco CIDR IPv4** (*IPv4 CIDR block*) | **Entrada manual** → `10.0.0.0/16` | A faixa de IPs privados da VPC (65.536 IPs) |
| **Bloco CIDR IPv6** (*IPv6 CIDR block*) | **Nenhum bloco CIDR IPv6** (*No IPv6 CIDR block*) | Mantemos sem IPv6 para simplificar |
| **Locação** (*Tenancy*) | **Padrão** (*Default*) | **Dedicado** coloca as instâncias em hardware exclusivo e é **bem mais caro** |
| **Controle de criptografia da VPC** (*VPC encryption control*) | **Deixar desativado** | Recurso **pago**: ~US$ 0,15 por hora por VPC em uso (veja o aviso abaixo) |
| **Tags** | `Projeto = estudos`, `Ambiente = dev` | Permitem **filtrar** os recursos (veja o aprofundamento) |

> ⚠️ **Ajuste na anotação original: o que é caro?**
>
> - **IPv6 não é caro**: os blocos IPv6 da AWS são **gratuitos**. Deixamos sem IPv6 só para **simplificar** o aprendizado.
> - O que é **caro** é a **locação Dedicada** (*Dedicated tenancy*): todas as instâncias da VPC rodariam em hardware exclusivo, com preço maior e taxa extra. Por isso, mantenha a **locação Padrão**.
> - O que **tem custo** no endereçamento é o **IPv4 público** (cerca de US$ 3,65 por mês por IP).

> ⚠️ **Controle de criptografia da VPC (*VPC Encryption Controls*): deixe desativado nos estudos**
>
> - **O que é:** um recurso que **audita e obriga** a criptografia **em trânsito** de todo o tráfego dentro da VPC.
>   - **Modo Monitor:** mostra qual tráfego está criptografado e quais recursos permitem tráfego sem criptografia.
>   - **Modo Enforce:** **impede** criar ou conectar recursos que permitam tráfego sem criptografia.
> - **Custo:** é pago desde março de 2026. A AWS cobra um valor **fixo por hora** para cada VPC **em uso** (com alguma interface de rede) que tenha o controle ativado. No exemplo da AWS para us-east-1, são **~US$ 0,15 por hora ≈ US$ 110 por mês**, mais do que todo o crédito de um plano gratuito.
> - **Quando usar:** empresas com exigências de compliance (saúde, financeiro, governo) que precisam **provar** que todo o tráfego interno é criptografado.
> - Mesmo sem esse recurso, você pode (e deve) criptografar o tráfego da **aplicação** com **HTTPS/TLS**.
>
> Fontes: [anúncio do recurso](https://aws.amazon.com/about-aws/whats-new/2025/11/aws-vpc-encryption-controls/), [anúncio do preço](https://aws.amazon.com/about-aws/whats-new/2026/03/vpc-encryption-controls-pricing/), [documentação](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-encryption-controls.html).

> 💡 **Aprofundamento — para que servem as tags**
>
> | Uso | Exemplo |
> |---|---|
> | **Filtrar** no console | Mostrar só os recursos com `Projeto = estudos` |
> | **Separar custos** | Ver na fatura quanto o `Projeto = loja` gastou no mês (*cost allocation tags*) |
> | **Permissões no IAM** | Deixar o time de dev mexer só em recursos com `Ambiente = dev` (veja o exemplo em [06 — IAM](06-aws-iam.md#4-segurança-no-nível-de-api-crucial-para-o-ec2)) |
> | **Automação** | Desligar à noite todas as instâncias com `Desligar = sim` |
>
> **Boa prática:** defina um padrão de tags e use **em todos os recursos** (ex.: `Name`, `Projeto`, `Ambiente`, `Dono`).

> 💡 **Aprofundamento — a opção "VPC e muito mais"**
>
> A opção **VPC e muito mais** (*VPC and more*) cria **tudo de uma vez**: VPC, subnets públicas e privadas em várias AZs, tabelas de rotas, Internet Gateway e, se você quiser, NAT Gateway e endpoint do S3. Ela mostra até um **diagrama** do que vai ser criado. É ótima para o dia a dia, mas para aprender vale fazer manualmente.

### O que a AWS cria automaticamente junto com a VPC

| Recurso | Detalhe |
|---|---|
| **Tabela de rotas principal** (*main route table*) | Só com a rota `local` |
| **Network ACL padrão** | **Libera** toda a entrada e a saída |
| **Security Group padrão** | Libera tráfego **entre os membros do próprio grupo** |

> ⚠️ Uma VPC **criada por você nasce sem subnets** e **sem Internet Gateway**. Já a **VPC padrão** (*default*) vem com uma subnet pública em cada AZ e com Internet Gateway.

> 💡 **Dica:** depois de criar, vá em **Ações → Editar configurações da VPC** (*Actions → Edit VPC settings*) e ative **Habilitar nomes de host DNS** (*Enable DNS hostnames*). Assim, as instâncias com IP público também recebem um **nome DNS público**. Em VPCs criadas manualmente, essa opção vem **desativada**.

> 💡 **Aprofundamento — as duas configurações de DNS da VPC**
>
> | Configuração | O que faz | Padrão em VPC criada manualmente |
> |---|---|---|
> | **Resolução de DNS** (*DNS resolution*) | Liga o **servidor DNS da AWS** dentro da VPC, que traduz nomes em IPs (ex.: `google.com` → IP). Ele fica no endereço **base da VPC + 2** (`10.0.0.2`), um dos IPs reservados. | ✅ Ativada |
> | **Nomes de host DNS** (*DNS hostnames*) | Dá **nomes DNS** às instâncias. Quem tem IP público ganha um **nome público**. | ❌ Desativada |
>
> **Exemplos de nomes (us-east-1):**
> - **Privado:** `ip-10-0-1-25.ec2.internal`, que só funciona dentro da VPC.
> - **Público:** `ec2-54-210-10-20.compute-1.amazonaws.com`, que funciona de qualquer lugar.
>
> **Por que ativar:** além do nome público, vários recursos **exigem as duas opções ativadas**: os **Interface Endpoints** com DNS privado, as **zonas privadas do Route 53**, a montagem do **EFS** pelo nome e o **RDS** com acesso público.
>
> **Por que vem desativada:** a AWS não documenta um motivo oficial. O entendimento comum é que ela mantém o padrão antigo por **compatibilidade** e deixa a decisão com você, porque muitas empresas usam o **próprio servidor DNS** e não querem nomes gerados pela AWS. A **VPC padrão** e o assistente **"VPC e muito mais"** já vêm com as duas ativadas.

---

## Passo 2 — Criar as subnets

**Caminho:** **Sub-redes** (*Subnets*) → **Criar sub-rede** (*Create subnet*)

1. Em **ID da VPC** (*VPC ID*), selecione a **sua VPC** (`vpc-estudos`). Cuidado para não escolher a VPC padrão.
2. Para cada subnet, preencha:

| Nome da sub-rede | Zona de disponibilidade | Bloco CIDR IPv4 da sub-rede |
|---|---|---|
| `subnet-publica-a` | `us-east-1a` | `10.0.1.0/24` |
| `subnet-publica-b` | `us-east-1b` | `10.0.2.0/24` |
| `subnet-privada-a` | `us-east-1a` | `10.0.11.0/24` |
| `subnet-privada-b` | `us-east-1b` | `10.0.12.0/24` |

3. Clique em **Adicionar nova sub-rede** (*Add new subnet*) para criar **várias de uma vez**, em **AZs diferentes**.
4. O console **sugere** um bloco CIDR automaticamente. Você pode aceitar ou digitar o seu.

**Regras do CIDR da subnet:**
- Precisa estar **dentro** da faixa da VPC (`10.0.x.x`).
- **Não pode se sobrepor** a outra subnet da mesma VPC.
- Tamanho entre **`/16` e `/28`**.
- **Não dá para mudar depois.** Se errar, é preciso excluir e criar de novo.

> 💡 **Por que esse padrão de IPs?** O **terceiro número** indica o tipo e a AZ: `1` e `2` são as públicas (AZs a e b), `11` e `12` são as privadas (AZs a e b). Olhando o IP de uma instância, você já sabe onde ela está. Mais detalhes em [10 — CIDR](10-cidr-enderecamento-ip.md#8-planejando-os-ips-de-uma-vpc).

---

## Passo 3 — Criar e anexar o Internet Gateway

**Caminho:** **Gateways da Internet** (*Internet gateways*) → **Criar gateway da Internet** (*Create internet gateway*)

1. Nome: `igw-estudos`.
2. Depois de criar, ele fica com o status **Detached** (desanexado).
3. Vá em **Ações → Associar à VPC** (*Actions → Attach to VPC*) e escolha `vpc-estudos`.

> ⚠️ **Não esqueça de anexar!** Um Internet Gateway criado e **não anexado** não faz nada. Cada VPC pode ter **apenas um** Internet Gateway.

---

## Passo 4 — Criar as tabelas de rotas e associar as subnets

### 4.1. Entendendo o ponto de partida
- Quando a VPC é criada, ela **já vem com uma tabela de rotas principal**, que tem **só a rota `local`**.
- Toda subnet que **não estiver associada** a nenhuma tabela usa **automaticamente a tabela principal**.
- Com só a rota `local`, os recursos **só conhecem os destinos internos** da VPC: **ninguém tem acesso à internet**. A rede é **privada por padrão**.

### 4.2. Criar duas tabelas: uma pública e uma privada

**Caminho:** **Tabelas de rotas** (*Route tables*) → **Criar tabela de rotas** (*Create route table*)

| Nome | VPC | Uso |
|---|---|---|
| `rt-publica` | `vpc-estudos` | Subnets públicas |
| `rt-privada` | `vpc-estudos` | Subnets privadas |

> 💡 Dê um nome também para a **tabela principal**, como `rt-principal-nao-usar`, para não confundir.

### 4.3. Associar as subnets a cada tabela

Selecione a tabela → aba **Associações de sub-rede** (*Subnet associations*) → **Editar associações de sub-rede** (*Edit subnet associations*):

| Tabela | Subnets associadas |
|---|---|
| `rt-publica` | `subnet-publica-a`, `subnet-publica-b` |
| `rt-privada` | `subnet-privada-a`, `subnet-privada-b` |

- **Cada subnet** fica associada a **uma única tabela**.
- **Uma tabela** pode ter **várias subnets**.

### 4.4. Ensinar a tabela pública o caminho para a internet

Selecione `rt-publica` → aba **Rotas** (*Routes*) → **Editar rotas** (*Edit routes*) → **Adicionar rota** (*Add route*):

| Destino (*Destination*) | Alvo (*Target*) |
|---|---|
| `10.0.0.0/16` | `local` (já existe) |
| **`0.0.0.0/0`** | **Gateway da Internet** → `igw-estudos` |

- `0.0.0.0/0` significa **"qualquer destino que não seja da VPC"**, ou seja, a internet.
- A `rt-privada` **não recebe essa rota**: fica só com a `local`.

> ⚠️ **Ajuste na anotação original: o que as tabelas separadas realmente protegem**
>
> - A rota **`local` sempre existe** e **não pode ser removida**. Por isso, **todas as subnets da VPC conseguem se comunicar entre si**, pública com privada inclusive, **não importa a tabela de rotas**.
> - O **perigo de usar uma tabela só** é outro: se você colocar a rota para o Internet Gateway na **mesma tabela** usada pelas subnets privadas, **elas viram públicas** e ficam expostas à internet. Esse é o motivo de ter **duas tabelas**.
> - Para **controlar o que a subnet pública pode acessar na privada**, use **Security Groups** e **Network ACLs** (veja [05 — Segurança no EC2](05-aws-ec2-seguranca.md)).

> 💡 **Boa prática:** **nunca coloque a rota para o Internet Gateway na tabela principal**. Assim, qualquer subnet nova que for criada e esquecida sem associação nasce **privada**, e não exposta.

---

## Passo 5 — Habilitar o IP público automático nas subnets públicas

Ter a rota para a internet **não basta**: a instância também precisa de um **IP público**.

**Caminho:** **Sub-redes** → selecione `subnet-publica-a` → **Ações → Editar configurações da sub-rede** (*Actions → Edit subnet settings*) → marque **Habilitar endereço IPv4 público de atribuição automática** (*Enable auto-assign public IPv4 address*).

Repita para `subnet-publica-b`.

- A partir daí, toda instância criada nessa subnet **recebe um IP público automaticamente**.
- **Não faça isso nas subnets privadas.**
- Lembre-se: **cada IPv4 público custa** cerca de US$ 3,65 por mês enquanto estiver em uso.

> 💡 Dá para escolher o IP público **instância por instância**, na hora de criar a instância, em vez de ativar para a subnet inteira.

---

## Passo 6 — Criar os Security Groups

O Security Group é o **firewall** de cada instância, com **regras de entrada** (*inbound*) e **regras de saída** (*outbound*). A teoria completa está em [05 — Segurança no EC2](05-aws-ec2-seguranca.md#2-security-groups-grupos-de-segurança).

### 6.1. O Security Group padrão (`default`)
Toda VPC já nasce com um Security Group chamado **`default`**:

| Direção | Regra padrão |
|---|---|
| **Entrada** | Libera **todo o tráfego** vindo de instâncias que estão **no próprio grupo `default`** |
| **Saída** | Libera **tudo** |

- Ele **não pode ser excluído**.
- Se você criar uma instância e não escolher nenhum grupo, ela entra no `default`.

> ⚠️ **Boa prática:** **não use o grupo `default`**. Crie grupos próprios, com nomes claros e só as regras necessárias. Muitas empresas até **removem as regras** do `default` para que ninguém o use por engano.

### 6.2. Como criar um Security Group

**Caminho:** console da VPC → **Grupos de segurança** (*Security groups*) → **Criar grupo de segurança** (*Create security group*). O mesmo menu existe no console do EC2.

| Campo | O que preencher | Detalhe |
|---|---|---|
| **Nome** (*Security group name*) | `sg-publico` | **Obrigatório** e **não pode ser alterado depois** |
| **Descrição** (*Description*) | "Firewall das instâncias públicas" | **Obrigatória** e **não pode ser alterada depois** |
| **VPC** | `vpc-estudos` | ⚠️ Confira! O grupo **só funciona** nas instâncias **desta VPC** |
| **Regras de entrada** | Veja abaixo | Começa **vazia**: tudo bloqueado |
| **Regras de saída** | Veja abaixo | Começa **liberando tudo** |
| **Tags** | `Projeto = estudos` | Opcional |

**Campos de cada regra:**

| Campo | O que é | Exemplos |
|---|---|---|
| **Tipo** (*Type*) | Atalho que já preenche o protocolo e a porta | `SSH`, `HTTP`, `HTTPS`, `MySQL/Aurora`, `TCP personalizado`, `Todo o tráfego`, `Todo o ICMP - IPv4` |
| **Protocolo** (*Protocol*) | TCP, UDP ou ICMP | Preenchido automaticamente pelo tipo |
| **Intervalo de portas** (*Port range*) | Uma porta ou uma faixa | `22`, `443`, `8000-8100` |
| **Origem** (entrada) / **Destino** (saída) | De onde vem ou para onde vai o tráfego | Veja a tabela abaixo |
| **Descrição** | Para que serve a regra | "HTTPS público", "SSH do escritório" |

**Opções de origem no console:**

| Opção | O que preenche | Quando usar |
|---|---|---|
| **Qualquer lugar-IPv4** (*Anywhere-IPv4*) | `0.0.0.0/0` | Serviços **públicos** (80, 443) |
| **Qualquer lugar-IPv6** (*Anywhere-IPv6*) | `::/0` | O mesmo, para IPv6 |
| **Meu IP** (*My IP*) | O seu IP atual, com `/32` | Acesso de administração (SSH, RDP) |
| **Personalizado** (*Custom*) | Uma **faixa de IPs** (ex.: `10.0.0.0/16`) ou **outro Security Group** (`sg-...`) | Faixas internas, subnets ou outro grupo |

### 6.3. Grupo público: `sg-publico`

Para as instâncias das **subnets públicas** (ex.: servidor web).

**Regras de entrada:**

| Tipo | Protocolo | Porta | Origem | Descrição |
|---|---|---|---|---|
| HTTP | TCP | 80 | `0.0.0.0/0` (Qualquer lugar) | Site público |
| HTTPS | TCP | 443 | `0.0.0.0/0` (Qualquer lugar) | Site público com HTTPS |
| SSH | TCP | 22 | **Meu IP** (ex.: `203.0.113.10/32`) | Administração, só do meu computador |

**Regras de saída:**

| Tipo | Protocolo | Porta | Destino | Descrição |
|---|---|---|---|---|
| Todo o tráfego | Todos | Todas | `0.0.0.0/0` | Padrão: atualizações e acesso às instâncias privadas |

### 6.4. Grupo privado: `sg-privado`

Para as instâncias das **subnets privadas** (ex.: aplicação ou banco de dados).

**Regras de entrada:**

| Tipo | Protocolo | Porta | Origem | Descrição |
|---|---|---|---|---|
| TCP personalizado | TCP | 8080 | **`sg-publico`** | Só a camada pública chama a aplicação |
| SSH | TCP | 22 | **`sg-publico`** | SSH só a partir de uma instância pública (servidor de salto) |
| Todo o ICMP - IPv4 | ICMP | Todas | `10.0.0.0/16` | Permite **ping** de dentro da VPC (útil para testes) |

**Regras de saída:**

| Tipo | Protocolo | Porta | Destino | Descrição |
|---|---|---|---|---|
| Todo o tráfego | Todos | Todas | `0.0.0.0/0` | Padrão. Só sai para a internet se houver NAT Gateway. |

> 💡 **Por que usar `sg-publico` como origem, e não um IP?**
>
> A regra passa a valer para **qualquer instância que estiver no `sg-publico`**, não importa o IP dela. Se você criar mais servidores públicos amanhã, eles **já têm acesso** à camada privada, sem precisar editar nada.

> ⚠️ **Liberar SSH do grupo público para o privado não é uma vulnerabilidade?**
>
> É um **risco calculado**, conhecido como padrão **bastion host** (servidor de salto). O risco real é: se a máquina pública for invadida, o atacante pode tentar **pular** dela para a privada (**movimento lateral**).
>
> **Comparando as opções de origem:**
>
> | Origem da regra SSH na privada | Risco |
> |---|---|
> | `0.0.0.0/0` | ❌ Péssimo: qualquer IP (só não é explorável hoje porque a privada não tem rota para a internet) |
> | `10.0.0.0/16` | ⚠️ Qualquer máquina da VPC pode tentar SSH |
> | IP da máquina pública (`/32`) | Mesmo risco do grupo, e quebra quando o IP muda |
> | **`sg-publico`** | ✅ **O mais restrito** entre as opções de rede: só as máquinas do grupo, e só na porta 22 |
>
> **Como reduzir o risco:**
> - O **bastion só aceita SSH do seu IP** (`/32`).
> - **Nunca copie a chave `.pem` para o bastion.** Use o `ssh -J` (*ProxyJump*): a chave fica só no seu computador.
> - Libere **só a porta necessária** (22), nunca "Todo o tráfego".
> - Mantenha o bastion **atualizado**, com o mínimo de software, e **desligado** quando não estiver em uso.
>
> **Em produção, as alternativas mais seguras eliminam o SSH:**
> - **Systems Manager Session Manager:** sem porta 22 e sem chave. A instância privada precisa de saída para a AWS (NAT ou Interface Endpoints, ambos pagos).
> - **EC2 Instance Connect Endpoint:** permite SSH em instâncias **privadas** sem bastion e sem IP público, **sem custo adicional**.
> - Na prática, a camada pública costuma ser um **load balancer**, e a privada só aceita a **porta da aplicação** (ex.: 8080) vinda do grupo do load balancer, **sem SSH**.

> 💡 **Variação para banco de dados:** crie um `sg-banco` com entrada **3306 (MySQL)** ou **5432 (PostgreSQL)** tendo **`sg-privado` como origem**. Assim, o banco só aceita conexões da aplicação, formando a cadeia **público → privado → banco**.

### 6.5. Pontos de atenção

- **Ping não funciona por padrão.** O ping usa o protocolo **ICMP**, que precisa ser liberado com uma regra própria.
- **"Meu IP" pode mudar.** Se a sua internet de casa troca de IP, você perde o acesso SSH. É só editar a regra e escolher **Meu IP** de novo.
- **As mudanças valem na hora**, sem reiniciar a instância.
- **Uma instância pode ter vários Security Groups**, e as regras se somam.
- Se você usar o **Session Manager**, nem precisa da regra de SSH (porta 22).

> 💡 **Aprofundamento — camadas de proteção da VPC**
>
> | Camada | Pergunta que responde |
> |---|---|
> | **Tabela de rotas** | **Existe caminho** até o destino? |
> | **Network ACL** | A **subnet** deixa esse tráfego passar? |
> | **Security Group** | A **instância** aceita esse tráfego? |
>
> O tráfego precisa passar pelas **três**. Essa ideia de várias barreiras se chama **defesa em profundidade**. Por enquanto, a **Network ACL padrão** (que libera tudo) atende aos estudos.

---

## Passo 7 (opcional) — Internet de saída para as subnets privadas

Se as instâncias privadas precisarem **baixar atualizações**:

1. Crie um **NAT Gateway** em uma **subnet pública**, com um **Elastic IP**.
2. Na `rt-privada`, adicione a rota `0.0.0.0/0` → **NAT Gateway**.

> ⚠️ O NAT Gateway **custa cerca de US$ 33 por mês** mesmo parado. Em estudos, **exclua assim que terminar**. Veja [08 — VPC](08-aws-vpc.md#34-nat-gateway).

---

## Passo 8 — Pronto para criar as máquinas

Com tudo configurado, ao criar uma instância EC2 é só escolher a **VPC**, a **subnet** e o **Security Group**:

| Subnet | Security Group | Resultado |
|---|---|---|
| `subnet-publica-a` | `sg-publico` | Recebe IP público, **acessa a internet** e pode ser acessada nas portas 80 e 443 (e na 22, só do seu IP) |
| `subnet-privada-a` | `sg-privado` | **Só IP privado**. Não acessa a internet nem é acessível de fora. Só aceita tráfego vindo das instâncias do `sg-publico`. |

### Checklist: o que uma instância precisa para acessar a internet

- [ ] **Internet Gateway** criado e **anexado** à VPC
- [ ] Subnet associada a uma tabela com a rota **`0.0.0.0/0` → Internet Gateway**
- [ ] Instância com **IP público** (ou Elastic IP)
- [ ] **Security Group** liberando as portas necessárias
- [ ] **Network ACL** liberando entrada e saída (a padrão já libera tudo)

---

## Entendendo o roteamento

### Sentido do roteamento: a tabela de rotas é um GPS
Para cada pacote que sai de uma subnet, a tabela responde: **"para chegar a esse destino, mande por qual caminho?"**

- **Destino**: para onde o tráfego quer ir (uma faixa de IPs).
- **Alvo**: por onde ele deve sair (local, Internet Gateway, NAT, peering...).

### Quando duas rotas servem, vence a mais específica
Se mais de uma rota combinar com o destino, a AWS escolhe a de **maior prefixo** (a faixa **menor e mais específica**). Isso se chama *longest prefix match*.

**Exemplo, com a `rt-publica`:**

| Pacote indo para... | Rotas que combinam | Rota escolhida | Resultado |
|---|---|---|---|
| `10.0.11.25` (instância privada) | `10.0.0.0/16` e `0.0.0.0/0` | `10.0.0.0/16` (mais específica) | Fica **dentro da VPC** |
| `142.250.79.14` (um site) | Só `0.0.0.0/0` | `0.0.0.0/0` | Vai para a **internet** pelo IGW |

### Modelos de tabela de rotas

| Modelo | Rotas | Uso |
|---|---|---|
| **Pública** | `local` + `0.0.0.0/0 → igw-` | Load balancers, servidores expostos, NAT Gateway |
| **Privada com saída** | `local` + `0.0.0.0/0 → nat-` | Aplicações que precisam baixar atualizações |
| **Isolada** | Só `local` | Bancos de dados, dados sensíveis |
| **Com endpoint do S3** | `local` + `pl-xxxx (S3) → vpce-` | Acessar o S3 sem internet e sem NAT |
| **Com peering** | `local` + `10.1.0.0/16 → pcx-` | Conversar com outra VPC |
| **Com VPN / Direct Connect** | `local` + `192.168.0.0/16 → vgw-` | Conversar com o escritório ou datacenter |
| **Com Transit Gateway** | `local` + `10.0.0.0/8 → tgw-` | Conversar com várias VPCs por um hub |

**Exemplo de uma tabela privada mais completa:**

| Destino | Alvo | Significado |
|---|---|---|
| `10.0.0.0/16` | `local` | Tudo da própria VPC fica dentro dela |
| `10.1.0.0/16` | `pcx-0abc...` | A VPC de desenvolvimento, via peering |
| `pl-63a5400a` (S3) | `vpce-0def...` | O S3, via Gateway Endpoint |
| `0.0.0.0/0` | `nat-0123...` | Todo o resto sai pelo NAT Gateway |

### Prefixos dos IDs na AWS

| Prefixo | Recurso |
|---|---|
| `vpc-` | VPC |
| `subnet-` | Subnet |
| `rtb-` | Tabela de rotas |
| `igw-` | Internet Gateway |
| `nat-` | NAT Gateway |
| `eipalloc-` | Elastic IP |
| `sg-` | Security Group |
| `acl-` | Network ACL |
| `eni-` | Interface de rede |
| `vpce-` | VPC Endpoint |
| `pl-` | Prefix list (lista de faixas de IP, ex.: as do S3) |
| `pcx-` | VPC Peering |
| `tgw-` | Transit Gateway |
| `vgw-` | Virtual Private Gateway (VPN) |

---

## Problemas comuns

| Problema | Causa provável | Solução |
|---|---|---|
| Instância na subnet pública **sem internet** | Sem IP público, IGW não anexado, rota faltando ou Security Group bloqueando | Conferir o checklist do Passo 8 |
| **Ping não responde** | O Security Group não libera **ICMP** | Adicionar a regra **Todo o ICMP - IPv4** com a origem certa |
| **SSH parou de funcionar** de um dia para o outro | O seu IP de casa mudou, e a regra **Meu IP** ficou com o IP antigo | Editar a regra e escolher **Meu IP** de novo |
| **Security Group não aparece** na hora de criar a instância | O grupo foi criado em **outra VPC** | Criar o grupo na VPC certa |
| **Erro ao criar a subnet** | CIDR **fora** da faixa da VPC ou **sobreposto** a outra subnet | Escolher um bloco livre dentro da VPC |
| **Subnet privada ficou exposta** | Rota para o IGW colocada na **tabela principal** ou na tabela errada | Tirar a rota e revisar as associações |
| Instância **sem nome DNS público** | *DNS hostnames* desativado na VPC | **Editar configurações da VPC** → ativar |
| **Não consigo excluir a VPC** | Ainda existem instâncias, NAT Gateways ou interfaces de rede dentro dela | Excluir esses recursos primeiro |
| **Conta veio mais cara** depois dos estudos | NAT Gateway ou IPs públicos esquecidos | Excluir o NAT e liberar os Elastic IPs |

> 💡 **Para limpar tudo depois dos estudos:**
> 1. Encerrar as instâncias.
> 2. Excluir os NAT Gateways.
> 3. Liberar os Elastic IPs.
> 4. Excluir a VPC. O console remove junto as subnets, as tabelas de rotas, o Internet Gateway, as Network ACLs e os Security Groups.

---

## Aprofundamento: a mesma VPC pela linha de comando (AWS CLI)

Para quando você for praticar. Cada comando devolve um ID (ex.: `vpc-...`), que é usado no comando seguinte.

```bash
# 1. VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=vpc-estudos}]'

# 2. Subnets (repita para as outras)
aws ec2 create-subnet --vpc-id vpc-XXXX --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=subnet-publica-a}]'

# 3. Internet Gateway (criar e anexar)
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id igw-XXXX --vpc-id vpc-XXXX

# 4. Tabela de rotas pública, rota para a internet e associação
aws ec2 create-route-table --vpc-id vpc-XXXX
aws ec2 create-route --route-table-id rtb-XXXX \
  --destination-cidr-block 0.0.0.0/0 --gateway-id igw-XXXX
aws ec2 associate-route-table --route-table-id rtb-XXXX --subnet-id subnet-XXXX

# 5. IP público automático na subnet pública
aws ec2 modify-subnet-attribute --subnet-id subnet-XXXX --map-public-ip-on-launch
```

---

## Revisão rápida

- **Na criação da VPC, qual é o campo obrigatório mais importante?**
  O bloco CIDR IPv4 (ex.: `10.0.0.0/16`). O nome é opcional, porque a VPC é identificada pelo ID.
- **Por que manter a locação "Padrão"?**
  Porque a locação "Dedicada" coloca as instâncias em hardware exclusivo e é bem mais cara.
- **IPv6 na VPC tem custo?**
  Não. O que tem custo é o IPv4 público.
- **Uma VPC criada manualmente já vem com subnets e Internet Gateway?**
  Não. Só a VPC padrão vem com eles.
- **O que a AWS cria automaticamente junto com uma VPC nova?**
  A tabela de rotas principal, a Network ACL padrão e o Security Group padrão.
- **O que acontece com uma subnet que não foi associada a nenhuma tabela de rotas?**
  Ela usa a tabela de rotas principal.
- **Quais são os passos para uma subnet ter acesso à internet?**
  Criar e anexar o Internet Gateway, adicionar a rota `0.0.0.0/0` para ele na tabela da subnet e dar IP público às instâncias.
- **Separar as tabelas de rotas impede que a subnet pública acesse a privada?**
  Não. A rota `local` sempre permite a comunicação dentro da VPC. As tabelas separadas evitam que a subnet privada fique exposta à internet. Para controlar o tráfego entre subnets, use Security Groups e Network ACLs.
- **Por que não colocar a rota para o Internet Gateway na tabela principal?**
  Porque toda subnet sem associação usaria essa tabela e nasceria pública.
- **O pacote vai para `10.0.11.25`, e a tabela tem `10.0.0.0/16 → local` e `0.0.0.0/0 → igw`. Qual rota é usada?**
  A `10.0.0.0/16`, porque é a mais específica.
- **O que significa o destino `0.0.0.0/0` em uma rota?**
  Qualquer destino que não combine com uma rota mais específica. Na prática, a internet.
- **Como vêm as regras do Security Group `default`?**
  A entrada libera só o tráfego de instâncias do próprio grupo, e a saída libera tudo. Ele não pode ser excluído.
- **Quais campos de um Security Group não podem ser alterados depois de criado?**
  O nome e a descrição.
- **Quais são os campos de uma regra do Security Group?**
  Tipo, protocolo, intervalo de portas, origem (ou destino, na saída) e descrição.
- **Por que usar outro Security Group como origem em vez de um IP?**
  Porque a regra passa a valer para qualquer instância daquele grupo, mesmo as criadas depois, sem depender de IPs.
- **O ping para uma instância não responde. O que verificar no Security Group?**
  Se existe uma regra liberando o protocolo ICMP.
- **Quais são as três camadas que o tráfego precisa atravessar na VPC?**
  A tabela de rotas, a Network ACL e o Security Group.
