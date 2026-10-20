# 08 — Amazon VPC (Virtual Private Cloud)

## 1. O que é a VPC

A **VPC (Virtual Private Cloud)** é a sua **rede privada e isolada dentro da AWS**. É nela que você coloca seus recursos (instâncias EC2, bancos de dados, load balancers) e define **como eles se comunicam** entre si e com a internet.

- É como ter a **rede de um datacenter próprio**, só que virtual e criada em minutos.
- Uma VPC pertence a **uma região** e se estende por **todas as zonas de disponibilidade (AZs)** daquela região.
- Você controla tudo: **faixa de IPs**, **subnets**, **rotas**, **gateways** e **firewalls**.
- Criar uma VPC é **gratuito**. Alguns componentes são pagos (veja a [seção 6](#6-custos)).

> 💡 **Analogia: o condomínio fechado**
>
> | Componente | No condomínio |
> |---|---|
> | **VPC** | O **condomínio inteiro**, com muro em volta |
> | **Subnets** | Os **blocos** do condomínio |
> | **Tabela de rotas** | As **placas de trânsito**, que dizem para onde cada rua leva |
> | **Internet Gateway** | O **portão principal**, por onde se entra e se sai |
> | **NAT Gateway** | Uma **saída só para moradores**: eles saem, mas ninguém de fora entra por ali |
> | **Network ACL** | A **guarita de cada bloco** |
> | **Security Group** | A **porta de cada apartamento** |

---

## 2. Bloco CIDR (faixa de IPs da VPC)

Ao criar a VPC, você define a **faixa de IPs privados** que ela vai usar, no formato **CIDR** (veja a explicação completa em [10 — CIDR e Endereçamento IP](10-cidr-enderecamento-ip.md)). O passo a passo de criação está em [09 — VPC na Prática](09-aws-vpc-na-pratica.md).

- Tamanho permitido: de **`/16`** (65.536 IPs, o maior) até **`/28`** (16 IPs, o menor).
- Use faixas **privadas**:

| Faixa privada | Exemplo comum |
|---|---|
| `10.0.0.0/8` | `10.0.0.0/16` |
| `172.16.0.0/12` | `172.31.0.0/16` (usada pela VPC padrão) |
| `192.168.0.0/16` | `192.168.0.0/16` |

> ⚠️ **Planeje os IPs com cuidado**
>
> Se um dia você quiser **conectar duas VPCs** (ou a VPC com o escritório), as faixas **não podem se sobrepor**. Exemplo: não dá para conectar duas VPCs que usam `10.0.0.0/16`. Use faixas diferentes desde o início (ex.: `10.0.0.0/16` para produção e `10.1.0.0/16` para desenvolvimento).

> 💡 **Aprofundamento — a AWS reserva 5 IPs em cada subnet**
>
> Em uma subnet `10.0.1.0/24` (256 IPs), só **251 podem ser usados**:
>
> | IP | Reservado para |
> |---|---|
> | `10.0.1.0` | Endereço da rede |
> | `10.0.1.1` | Roteador da VPC |
> | `10.0.1.2` | Servidor DNS da AWS |
> | `10.0.1.3` | Uso futuro da AWS |
> | `10.0.1.255` | Broadcast |

---

## 3. Componentes da VPC

### 3.1. Subnets (sub-redes)
- São **divisões da VPC**. Cada subnet tem uma faixa de IPs **dentro** da faixa da VPC.
- Cada subnet fica em **uma única AZ**. Para ter alta disponibilidade, crie subnets em **pelo menos duas AZs**.

| Tipo | Característica | O que colocar nela |
|---|---|---|
| **Pública** | Tem uma rota para o **Internet Gateway**: acessível pela internet | Load balancers, NAT Gateway, servidores web expostos |
| **Privada** | **Não tem** rota para o Internet Gateway: **não é acessível** pela internet | Servidores de aplicação, bancos de dados, sistemas internos |

> ⚠️ O que torna uma subnet **pública ou privada** é a **tabela de rotas** dela, e não o nome. Uma subnet chamada "pública" sem rota para o Internet Gateway é, na prática, privada.

### 3.2. Tabela de rotas (Route Table)
- Define **para onde vai o tráfego** que sai de cada subnet.
- Toda tabela tem a rota **`local`**, que permite a comunicação entre **todos os recursos da VPC**.

**Exemplo:**

| Subnet | Destino | Alvo | Significado |
|---|---|---|---|
| **Pública** | `10.0.0.0/16` | `local` | Tráfego interno fica na VPC |
| | `0.0.0.0/0` | `igw-xxxx` (Internet Gateway) | Todo o resto vai para a internet |
| **Privada** | `10.0.0.0/16` | `local` | Tráfego interno fica na VPC |
| | `0.0.0.0/0` | `nat-xxxx` (NAT Gateway) | Saída para a internet pelo NAT (só saída) |
| **Banco de dados** | `10.0.0.0/16` | `local` | **Só tráfego interno**, sem nenhum acesso à internet |

### 3.3. Internet Gateway (IGW)
- Conecta a VPC à **internet**, nos **dois sentidos** (entrada e saída).
- **Um por VPC.** É gerenciado pela AWS, redundante e **gratuito**.
- Para uma instância ser acessada pela internet, ela precisa de **três coisas**:
  1. Estar em uma **subnet pública** (com rota para o IGW).
  2. Ter um **IP público** ou um **Elastic IP**.
  3. Ter o **Security Group** liberando a porta.

### 3.4. NAT Gateway
- Permite que instâncias em **subnets privadas** **acessem a internet** (para baixar atualizações ou chamar APIs externas), mas **impede que a internet inicie conexões** com elas.
- Fica em uma **subnet pública** e usa um **Elastic IP**.
- É **pago** (por hora e por GB processado).
- Fica em **uma única AZ**. Para alta disponibilidade, crie **um NAT Gateway por AZ**.

### 3.5. Elastic IP
- Um **IP público fixo**, que não muda quando a instância é desligada e ligada.
- Pode ser **movido** de uma instância para outra (útil em caso de falha).
- É **cobrado** por hora, assim como **todo IPv4 público** na AWS.

### 3.6. Security Groups e Network ACLs
Os dois firewalls da VPC (veja os detalhes em [05 — Segurança no EC2](05-aws-ec2-seguranca.md)):

| | **Security Group** | **Network ACL** |
|---|---|---|
| **Nível** | Instância | **Subnet** |
| **Estado** | Stateful | **Stateless** |
| **Regras** | Só permitir | **Permitir e negar** |
| **Padrão** | Entrada bloqueada, saída liberada | A NACL **padrão libera tudo**. Uma NACL **nova bloqueia tudo**. |

> 💡 **Aprofundamento — portas efêmeras na NACL**
>
> Como a NACL é **stateless**, a **resposta** também precisa ser liberada. As respostas usam **portas efêmeras** (de `1024` a `65535`). Se você liberar a entrada na porta 443 e esquecer de liberar a saída nas portas efêmeras, a conexão não funciona.

### 3.7. VPC Endpoints
Permitem acessar **serviços da AWS** (S3, DynamoDB, SQS...) **sem passar pela internet** e **sem NAT Gateway**.

| Tipo | Serviços | Como funciona | Custo |
|---|---|---|---|
| **Gateway Endpoint** | **Só S3 e DynamoDB** | Uma rota na tabela de rotas | **Gratuito** |
| **Interface Endpoint** (PrivateLink) | A maioria dos serviços da AWS | Uma interface de rede com IP privado dentro da subnet | Pago (por hora e por GB) |

> 💡 **Dica de economia:** se instâncias privadas acessam muito o S3, crie um **Gateway Endpoint**. O tráfego deixa de passar pelo NAT Gateway, que cobra por GB processado.

### 3.8. Conectando VPCs e redes externas

| Recurso | O que faz | Detalhe importante |
|---|---|---|
| **VPC Peering** | Conecta **duas VPCs** diretamente (mesma conta ou outra, mesma região ou outra) | **Não é transitivo**: se A conecta com B e B com C, **A não fala com C** |
| **Transit Gateway** | Um **hub central** que conecta **muitas VPCs** e redes locais | **É transitivo**. Ideal quando há muitas VPCs. |
| **Site-to-Site VPN** | Conecta o **escritório ou datacenter** à VPC por um **túnel criptografado pela internet** | Rápido de configurar. Depende da qualidade da internet. |
| **Direct Connect** | **Link físico dedicado** entre o datacenter e a AWS, **sem passar pela internet** | Mais estável e com mais banda. Leva semanas para instalar. |

### 3.9. VPC Flow Logs
- Registram **informações sobre o tráfego** de rede da VPC: IP de origem e destino, porta, protocolo e se foi **aceito (ACCEPT)** ou **rejeitado (REJECT)**.
- Podem ser enviados para o **CloudWatch Logs**, o **S3** ou o **Data Firehose**.
- **Quando usar:** descobrir **por que uma conexão não funciona**, investigar incidentes de segurança e auditoria.

---

## 4. VPC padrão vs VPC personalizada

| | **VPC padrão (default)** | **VPC personalizada** |
|---|---|---|
| **Quem cria** | A AWS, automaticamente, **em cada região** | Você |
| **Faixa de IPs** | `172.31.0.0/16` | Você escolhe |
| **Subnets** | Uma subnet **pública** em cada AZ | Você define (públicas e privadas) |
| **Internet Gateway** | Já vem conectado | Você cria e conecta |
| **IP público nas instâncias** | **Sim, automaticamente** | Você decide |
| **Quando usar** | **Estudos e testes rápidos** | **Produção** |

> ⚠️ Na VPC padrão, **tudo é público**. Em produção, crie uma **VPC personalizada**, com subnets privadas para a aplicação e para o banco de dados.

---

## 5. Arquitetura típica (3 camadas, 2 AZs)

```
Internet
   │
[ Internet Gateway ]
   │
VPC 10.0.0.0/16
├── AZ A
│   ├── Subnet pública  10.0.1.0/24   → Load Balancer, NAT Gateway
│   ├── Subnet privada  10.0.11.0/24  → EC2 da aplicação
│   └── Subnet banco    10.0.21.0/24  → Banco de dados (principal)
└── AZ B
    ├── Subnet pública  10.0.2.0/24   → Load Balancer, NAT Gateway
    ├── Subnet privada  10.0.12.0/24  → EC2 da aplicação
    └── Subnet banco    10.0.22.0/24  → Banco de dados (réplica)
```

**Caminho de um acesso do usuário:**
```
Usuário ──▶ Internet Gateway ──▶ Load Balancer (pública) ──▶ EC2 (privada) ──▶ Banco (subnet do banco)
```

**Caminho da instância baixando uma atualização:**
```
EC2 (privada) ──▶ NAT Gateway (pública) ──▶ Internet Gateway ──▶ Internet
```

**Por que assim?**
- **Só o load balancer fica exposto.** As instâncias e o banco ficam em subnets privadas, inacessíveis pela internet.
- **Duas AZs:** se uma AZ cair, a outra continua atendendo.
- **Security Groups encadeados** (load balancer → aplicação → banco), como vimos na anotação de segurança.

---

## 6. Custos

### 6.1. O que é gratuito e o que é pago

| Componente | Custo |
|---|---|
| VPC, subnets, tabelas de rotas | **Gratuito** |
| Internet Gateway | **Gratuito** (paga-se a transferência de dados) |
| Security Groups e Network ACLs | **Gratuito** |
| Gateway Endpoint (S3 e DynamoDB) | **Gratuito** |
| VPC Peering | **Sem custo por hora**. Paga-se a transferência de dados entre AZs ou regiões. |
| **NAT Gateway** | **Pago**: por hora + por GB processado |
| **IPv4 público** (inclusive Elastic IP) | **Pago**: por hora, por IP |
| **Interface Endpoint** | **Pago**: por hora, por AZ + por GB |
| **Transit Gateway** | **Pago**: por conexão (attachment) por hora + por GB |
| **Site-to-Site VPN** | **Pago**: por conexão por hora |
| **Flow Logs** | Paga-se o armazenamento dos logs (CloudWatch ou S3) |

### 6.2. Valores de referência (região us-east-1)

> Valores **aproximados**, para estudo. Variam por região e podem mudar. Consulte a [página oficial de preços da VPC](https://aws.amazon.com/vpc/pricing/). Um mês tem cerca de **730 horas**.

| Componente | Preço | Por mês (24/7) |
|---|---|---|
| **NAT Gateway** | ~US$ 0,045 por hora + ~US$ 0,045 por GB processado | **~US$ 32,85** + dados |
| **IPv4 público / Elastic IP** | ~US$ 0,005 por hora, por IP | **~US$ 3,65** por IP |
| **Interface Endpoint** | ~US$ 0,01 por hora, por AZ + ~US$ 0,01 por GB | **~US$ 7,30** por AZ + dados |
| **Transit Gateway** | ~US$ 0,05 por attachment por hora + ~US$ 0,02 por GB | **~US$ 36,50** por attachment + dados |
| **Site-to-Site VPN** | ~US$ 0,05 por conexão por hora | **~US$ 36,50** por conexão |
| **VPC Encryption Controls** (modo Monitor ou Enforce) | ~US$ 0,15 por hora, por VPC em uso | **~US$ 109,50** por VPC |
| **Transferência entre AZs** | ~US$ 0,01 por GB, em cada sentido | — |
| **Transferência na mesma AZ** (IP privado) | **Gratuita** | — |

### 6.3. Exemplos de cálculo

| Cenário | Cálculo | Total/mês |
|---|---|---|
| 1 NAT Gateway, 100 GB processados | (730 × 0,045) + (100 × 0,045) | **~US$ 37,35** |
| 2 NAT Gateways (alta disponibilidade), 100 GB cada | 2 × 37,35 | **~US$ 74,70** |
| 3 instâncias com IP público | 3 × 3,65 | **~US$ 10,95** |
| 1 TB de tráfego para o S3 **pelo NAT** | 1.024 × 0,045 | **~US$ 46,08** |
| 1 TB de tráfego para o S3 **pelo Gateway Endpoint** | — | **US$ 0** |

> **Lição:** o **NAT Gateway** costuma ser o **maior custo de rede** de uma VPC. Use **Gateway Endpoints** para S3 e DynamoDB e evite IPs públicos desnecessários. Instâncias privadas atrás de um load balancer não precisam de IP público.

---

## 7. Exemplos: "se eu preciso de X, uso Y"

| Se eu preciso... | Eu uso... | Por quê |
|---|---|---|
| Um **site acessível pela internet** | **Load balancer em subnet pública** + EC2 em subnet privada | Só o load balancer fica exposto |
| Um **servidor simples de teste** acessível direto | **Subnet pública** + **IP público** + Security Group | Rápido de montar. Não recomendado para produção. |
| Um **banco de dados** seguro | **Subnet privada** sem rota para a internet | Ninguém de fora consegue chegar até ele |
| Que uma **instância privada baixe atualizações** | **NAT Gateway** | Permite sair para a internet sem ficar exposta |
| Que uma **instância privada acesse o S3** sem custo de NAT | **Gateway Endpoint** | Gratuito e não passa pela internet |
| Acessar **SQS, Secrets Manager ou outro serviço** de forma privada | **Interface Endpoint** | Tráfego fica dentro da rede da AWS |
| Um **IP público que nunca muda** | **Elastic IP** | O IP continua o mesmo mesmo depois de desligar a instância |
| **Conectar 2 VPCs** | **VPC Peering** | Simples e sem custo por hora |
| **Conectar 10 ou mais VPCs** e o escritório | **Transit Gateway** | Hub central e transitivo. Evita dezenas de peerings. |
| Ligar o **escritório à AWS rapidamente** | **Site-to-Site VPN** | Configura em horas, usando a internet |
| Uma conexão **dedicada, estável e com muita banda** com o datacenter | **Direct Connect** | Link físico, sem passar pela internet |
| **Bloquear um IP malicioso** específico | **Network ACL** com regra de **deny** | Security Groups não têm regra de bloqueio |
| Descobrir **por que uma conexão está sendo bloqueada** | **VPC Flow Logs** | Mostram o que foi aceito e o que foi rejeitado |
| **Alta disponibilidade** | Subnets e NAT Gateways em **pelo menos 2 AZs** | Se uma AZ cair, a outra continua |
| **Separar produção e desenvolvimento** | **VPCs diferentes**, com faixas de IP diferentes | Isolamento total entre os ambientes |

---

## 8. Boas práticas

- **Não use a VPC padrão em produção.** Crie uma VPC personalizada.
- **Planeje as faixas de IP** para não se sobreporem com outras VPCs ou com o escritório.
- Coloque **aplicação e banco em subnets privadas**. Só exponha o que precisa (ex.: o load balancer).
- Use **pelo menos 2 AZs**.
- Use **Gateway Endpoints** para S3 e DynamoDB.
- Ative os **VPC Flow Logs** para auditoria e diagnóstico.
- Use o **Session Manager** em vez de abrir a porta 22 ou manter um "bastion host" (servidor de salto).

---

## 9. Links oficiais

- [O que é a Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [VPC padrão](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html)
- [Subnets](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [Tabelas de rotas](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [Gateway Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html)
- [AWS PrivateLink (Interface Endpoints)](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html)
- [VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
- [Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html)
- [Site-to-Site VPN](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html)
- [Direct Connect](https://aws.amazon.com/directconnect/)
- [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [Preços da VPC](https://aws.amazon.com/vpc/pricing/)

---

## 10. Revisão rápida

- **O que é uma VPC e qual é o escopo dela?**
  É uma rede privada e isolada dentro da AWS. Pertence a uma região e se estende por todas as AZs dela.
- **Qual é o maior e o menor bloco CIDR permitido em uma VPC?**
  O maior é `/16` (65.536 IPs) e o menor é `/28` (16 IPs).
- **Quantos IPs a AWS reserva em cada subnet?**
  5 IPs.
- **Uma subnet pode ficar em duas AZs?**
  Não. Cada subnet fica em uma única AZ.
- **O que define se uma subnet é pública ou privada?**
  A tabela de rotas: a subnet pública tem rota para o Internet Gateway.
- **O que uma instância precisa para ser acessada pela internet?**
  Estar em uma subnet pública, ter um IP público e ter o Security Group liberando a porta.
- **Qual é a diferença entre Internet Gateway e NAT Gateway?**
  O Internet Gateway permite entrada e saída. O NAT Gateway permite só a saída de instâncias privadas.
- **Onde o NAT Gateway precisa ficar?**
  Em uma subnet pública.
- **Quais serviços têm Gateway Endpoint, e quanto custa?**
  Só S3 e DynamoDB, e é gratuito.
- **Se a VPC A tem peering com a B e a B com a C, a A consegue falar com a C?**
  Não. O peering não é transitivo. Para isso, use um Transit Gateway.
- **Qual é a diferença entre Site-to-Site VPN e Direct Connect?**
  A VPN usa um túnel criptografado pela internet e é rápida de configurar. O Direct Connect é um link físico dedicado, sem passar pela internet.
- **Qual recurso mostra se o tráfego foi aceito ou rejeitado?**
  Os VPC Flow Logs.
- **Qual é geralmente o maior custo de rede de uma VPC?**
  O NAT Gateway.
- **Elastic IP é gratuito?**
  Não. Todo IPv4 público é cobrado por hora (~US$ 3,65 por mês por IP).
