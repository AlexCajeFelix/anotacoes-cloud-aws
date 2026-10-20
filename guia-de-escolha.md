# Guia de Escolha: "se eu preciso de X, eu uso Y"

Exemplos práticos de **qual opção escolher** em cada categoria. Os detalhes estão nas anotações de cada assunto (links em cada seção).

---

## 1. Famílias de instância EC2

Detalhes em [02 — EC2](02-aws-ec2.md).

| Se eu preciso de... | Eu pego... | Por quê |
|---|---|---|
| Um **blog pessoal** ou site com pouco acesso | **T** (ex.: `t4g.small`) | Barata, e o burst segura os picos de acesso |
| Um **ambiente de desenvolvimento ou teste** | **T** (ex.: `t3.medium`) | Fica parada a maior parte do tempo e acumula créditos de CPU |
| Uma **API ou backend** de uma empresa média | **M** (ex.: `m7g.large`) | Equilíbrio entre CPU e memória, sem depender de créditos |
| Um **servidor de jogos multiplayer** | **C** (ex.: `c7g.xlarge`) | Muito processamento por segundo |
| **Converter vídeos** (transcodificação) ou processamento em lote | **C** | Tarefa que usa muita CPU |
| Um **banco de dados relacional grande** ou um **cache Redis** | **R** (ex.: `r7g.2xlarge`) | Muita memória RAM |
| Rodar o **SAP HANA** ou um banco em memória gigante | **X** | Memória extrema (terabytes) |
| Um **banco NoSQL** com altíssimo IOPS (Cassandra, MongoDB) | **I** (ex.: `i4i.xlarge`) | Disco NVMe local muito rápido |
| Um **cluster Hadoop** com muito disco | **D** | Muito armazenamento HDD barato |
| **Treinar um modelo de IA** | **P** (GPU) ou **Trn** (Trainium) | Processamento paralelo massivo |
| Rodar um **chatbot ou modelo de IA já treinado** (inferência) | **Inf** (Inferentia) ou **G** | Inferência com custo menor |
| **Renderização 3D** ou streaming de jogos | **G** | GPU voltada para gráficos |
| Um software com **licença cobrada por núcleo** | **z** (alta frequência) | Menos núcleos, cada um mais rápido: paga menos licença |
| O **melhor custo-benefício** em geral | Instâncias com **`g`** (Graviton) | Processador ARM da AWS, mais barato para o mesmo desempenho |

---

## 2. Modelos de preço do EC2

Detalhes em [02 — EC2](02-aws-ec2.md#4-modelos-de-preço-como-pagar-pelo-ec2).

| Se eu preciso de... | Eu pego... | Por quê |
|---|---|---|
| Um **projeto de 2 semanas** ou uma prova de conceito | **On-Demand** | Sem compromisso: desliga quando terminar |
| Um **site de produção** ligado 24/7 por anos | **Compute Savings Plans** | Até 66% de desconto, com liberdade para trocar de família e de região |
| Um **banco de dados** que nunca vai mudar de família nem de região | **EC2 Instance Savings Plans** ou **Reserved Instance Standard** | Até 72% de desconto |
| **Processar 10 mil imagens à noite**, podendo reiniciar se parar | **Spot** | Até 90% de desconto. Se for interrompido, é só continuar depois. |
| **Rodar testes de CI/CD** | **Spot** | Tarefas curtas que toleram interrupção |
| **Garantir capacidade** na Black Friday | **Capacity Reservation** | Garante que vai ter servidor disponível na AZ |
| Usar uma **licença própria** do Windows Server ou Oracle por socket | **Dedicated Host** | Servidor físico inteiro, com visibilidade de sockets e núcleos |
| **Hardware isolado** por exigência de compliance, sem gerenciar o servidor | **Dedicated Instance** | Hardware exclusivo da conta |
| Uma **aplicação com base fixa e picos** | **Savings Plans** para a base + **On-Demand** para os picos | Desconto no que é previsível, flexibilidade no resto |

---

## 3. Volumes EBS e Instance Store

Detalhes em [03 — EBS](03-aws-ebs.md) e [04 — Instance Store](04-aws-instance-store.md).

| Se eu preciso de... | Eu pego... | Por quê |
|---|---|---|
| O **disco do sistema operacional** de qualquer instância | **gp3** | Padrão, barato, com 3.000 IOPS de base |
| Um **MySQL ou PostgreSQL** de um e-commerce médio | **gp3** (comprando IOPS extra, se precisar) | Bom desempenho sem o custo do io2 |
| Um **Oracle ou SAP crítico**, com latência abaixo de 1 ms | **io2** | IOPS alto, constante e durabilidade de 99,999% |
| **O mesmo disco em várias instâncias** ao mesmo tempo | **io2** (Multi-Attach) | Só io1 e io2 suportam |
| Mais de **80.000 IOPS** em um volume | **io2** | Chega a 256.000 IOPS |
| **Processar logs** ou um data warehouse com leitura sequencial | **st1** | Throughput alto, preço baixo |
| Guardar dados **quase nunca lidos** em disco, pelo menor preço | **sc1** | O volume EBS mais barato |
| Um sistema antigo usando **gp2** | **Migrar para gp3** | ~20% mais barato e com desempenho independente do tamanho |
| **Cache ou arquivos temporários** ultrarrápidos | **Instance Store** | Disco local, mais rápido e incluído no preço da instância |

---

## 4. Tipo de armazenamento

Detalhes em [03 — EBS](03-aws-ebs.md#11-ebs-vs-outros-armazenamentos-da-aws) e [07 — S3](07-aws-s3.md).

| Se eu preciso de... | Eu pego... | Por quê |
|---|---|---|
| Um **disco para o banco de dados** de uma instância | **EBS** | Armazenamento em bloco, baixa latência |
| Uma **pasta compartilhada** entre 10 servidores web | **EFS** | Vários servidores acessando os mesmos arquivos ao mesmo tempo |
| Guardar as **fotos dos usuários** de um app | **S3** | Escala ilimitada, acesso por URL |
| **Cache** ou dados temporários de processamento | **Instance Store** | Rápido e sem custo extra |
| **Backup de longo prazo** | **S3 Glacier** | Muito barato e com durabilidade de 11 noves |
| Hospedar um **site estático** | **S3 + CloudFront** | Sem servidor, com HTTPS e entrega rápida |

---

## 5. Classes de armazenamento do S3

Detalhes em [07 — S3](07-aws-s3.md#7-classes-de-armazenamento).

| Se eu preciso guardar... | Eu pego... | Por quê |
|---|---|---|
| **Imagens e vídeos de um site** (acessados toda hora) | **S3 Standard** | Acesso frequente, sem taxa de recuperação |
| Um **data lake** sem saber com que frequência os dados serão lidos | **S3 Intelligent-Tiering** | Move sozinho para a camada mais barata |
| **Backups do mês passado**, usados só se algo der errado | **S3 Standard-IA** | Mais barato para guardar, acesso imediato quando precisar |
| Uma **cópia secundária de backup** ou **miniaturas** que podem ser recriadas | **S3 One Zone-IA** | 20% mais barato que o Standard-IA. Se perder, dá para recriar. |
| **Exames médicos antigos** que o médico pode pedir a qualquer momento | **S3 Glacier Instant Retrieval** | Barato para arquivar e acesso em milissegundos |
| **Fotos de anos anteriores** de um app, raramente pedidas, podendo esperar horas | **S3 Glacier Flexible Retrieval** | Muito barato. Recuperação em minutos ou horas. |
| **Documentos fiscais** que a lei manda guardar por anos | **S3 Glacier Deep Archive** | O mais barato de todos (~US$ 1 por TB/mês) |
| **Dados de treino de IA**, lidos milhões de vezes com latência mínima | **S3 Express One Zone** | A classe mais rápida do S3 |
| **Logs** muito usados no início e quase nada depois | **Regra de ciclo de vida**: Standard → Standard-IA → Glacier → Deep Archive | Cada fase na classe mais barata possível |
| Receber **uploads de usuários de outros continentes** | **Transfer Acceleration** | Os dados entram pela edge location mais próxima |
| **Proteger os backups contra ransomware** | **Object Lock** + **Versionamento** | Ninguém consegue apagar ou alterar durante o período definido |
| Dar um **link temporário** para alguém baixar um arquivo privado | **URL pré-assinada** | Acesso por tempo limitado, sem abrir o bucket |

---

## 6. Segurança e IAM

Detalhes em [05 — Segurança no EC2](05-aws-ec2-seguranca.md) e [06 — IAM](06-aws-iam.md).

| Se eu preciso... | Eu uso... | Por quê |
|---|---|---|
| Deixar um **site público** | **Security Group** liberando 80 e 443 para `0.0.0.0/0` | Só as portas do site ficam abertas |
| Acessar a instância por **SSH só do meu computador** | **Security Group** com a porta 22 liberada para `MEU_IP/32` | Ninguém mais consegue tentar entrar |
| Acessar a instância **sem abrir a porta 22** | **Session Manager** | Sem porta aberta, sem chave e com registro de tudo |
| Que o **banco só aceite conexões da aplicação** | **Security Group** do banco tendo **o Security Group da aplicação como origem** | Não depende de IPs |
| **Bloquear um IP malicioso** | **Network ACL** com regra de **deny** | Security Group não tem regra de bloqueio |
| Que uma **instância EC2 leia o S3** | **IAM Role** | Credenciais temporárias, sem chaves no servidor |
| Dar as **mesmas permissões a 10 desenvolvedores** | **Grupo do IAM** + **política gerenciada pelo cliente** | Gerencia tudo em um lugar só |
| Uma permissão **exclusiva de uma única role** | **Política inline** | Fica presa àquela identidade |
| Que os funcionários entrem com o **e-mail corporativo** | **IAM Identity Center** | Login único (SSO) com credenciais temporárias |
| **Impedir que alguém exclua instâncias**, mesmo com outras permissões | **`Deny` explícito** em uma política | O `Deny` sempre vence |
| Saber **quem excluiu uma instância** | **CloudTrail** | Registra todas as chamadas de API |
| **Compartilhar um bucket** com outra conta AWS | **Bucket policy** | Política baseada em recurso |
| **Criptografar os discos** | **Criptografia do EBS** (KMS), ativada por padrão na região | Transparente e sem custo adicional |
| Ter **controle e auditoria** sobre as chaves de criptografia do S3 | **SSE-KMS** | Cada uso da chave fica registrado |

---

## 7. Rede (VPC)

Detalhes em [08 — VPC](08-aws-vpc.md#7-exemplos-se-eu-preciso-de-x-uso-y).

| Se eu preciso... | Eu uso... | Por quê |
|---|---|---|
| Um **site acessível pela internet** | **Load balancer em subnet pública** + EC2 em subnet privada | Só o load balancer fica exposto |
| Um **banco de dados** seguro | **Subnet privada** sem rota para a internet | Inacessível de fora |
| Que uma **instância privada baixe atualizações** | **NAT Gateway** | Sai para a internet sem ficar exposta |
| Que uma **instância privada acesse o S3** sem pagar NAT | **Gateway Endpoint** | Gratuito |
| Um **IP público fixo** | **Elastic IP** | Não muda ao desligar a instância |
| **Conectar 2 VPCs** | **VPC Peering** | Simples, sem custo por hora |
| **Conectar muitas VPCs** e o escritório | **Transit Gateway** | Hub central e transitivo |
| Ligar o **escritório à AWS** rapidamente | **Site-to-Site VPN** | Túnel criptografado pela internet |
| Uma conexão **dedicada e estável** com o datacenter | **Direct Connect** | Link físico, sem internet |
| Descobrir **por que o tráfego está sendo bloqueado** | **VPC Flow Logs** | Mostram o que foi aceito e o que foi rejeitado |
| Que uma **subnet acesse a internet** | **Internet Gateway anexado** + rota `0.0.0.0/0 → igw` na tabela dela + **IP público** | São os três requisitos juntos (veja [09 — VPC na Prática](09-aws-vpc-na-pratica.md)) |
| Que **subnets novas nasçam privadas** | Deixar a **tabela principal sem rota para a internet** | Subnet sem associação usa a tabela principal |
| Um **firewall para instâncias públicas** | Security Group com 80 e 443 de `0.0.0.0/0` | Só as portas do site ficam abertas |
| Um **firewall para instâncias privadas** | Security Group com **outro Security Group como origem** | Só a camada de cima consegue chegar |
| Fazer **ping** entre instâncias | Regra **ICMP** no Security Group | O ping é bloqueado por padrão |

---

## 8. Tamanho de rede (CIDR)

Detalhes em [10 — CIDR e Endereçamento IP](10-cidr-enderecamento-ip.md).

| Se eu preciso... | Eu uso... | Por quê |
|---|---|---|
| Liberar **um único IP** no Security Group | **`/32`** (ex.: `200.150.10.5/32`) | Só aquele endereço |
| Liberar **qualquer IP** | **`0.0.0.0/0`** | A internet inteira |
| Liberar **qualquer instância da VPC** | O **CIDR da VPC** (ex.: `10.0.0.0/16`) | Todos os IPs internos |
| Uma **VPC de produção** com espaço para crescer | **`/16`** | A maior permitida: 65.536 IPs |
| Uma **subnet comum**, com poucos servidores | **`/24`** | 251 IPs utilizáveis |
| Pelo menos **500 IPs** em uma subnet | **`/23`** | 507 IPs utilizáveis |
| Uma subnet para **Kubernetes** ou um Auto Scaling grande | **`/20`** ou maior | 4.091 IPs utilizáveis |
| A **menor subnet possível** | **`/28`** | 11 IPs utilizáveis |
| **Separar ambientes** sem conflito de IPs | Produção `10.0.0.0/16`, homologação `10.1.0.0/16`, desenvolvimento `10.2.0.0/16` | Faixas que não se sobrepõem permitem conectar as VPCs depois |

---

## 9. Juntando tudo: exemplos de arquitetura completa

### 9.1. Blog pessoal
| Categoria | Escolha |
|---|---|
| Instância | `t4g.small` (família **T**) |
| Preço | **On-Demand** no começo. Se for ficar no ar por anos, **Savings Plans**. |
| Disco | **gp3** de 20 GB |
| Rede | **Subnet pública** com IP público |
| Segurança | Security Group: 80 e 443 para `0.0.0.0/0`. SSH só pelo **Session Manager**. |
| Imagens | **S3 Standard** + **CloudFront** |

> Se o blog for **só HTML estático** (sem WordPress, por exemplo), nem precisa de EC2: **S3 + CloudFront** resolve, mais barato e sem servidor para manter.

### 9.2. Loja virtual (e-commerce) de porte médio
| Categoria | Escolha |
|---|---|
| Rede | **VPC personalizada** com subnets públicas, privadas e de banco em **2 AZs** |
| Entrada | **Load balancer** nas subnets públicas |
| Aplicação | Instâncias **M** (`m7g.large`) nas subnets privadas |
| Preço | **Compute Savings Plans** para a base + **On-Demand** para os picos |
| Banco de dados | Instância **R** com volume **gp3** (ou **io2**, se for crítico), em subnet privada |
| Saída para a internet | **1 NAT Gateway por AZ** |
| Acesso ao S3 | **Gateway Endpoint** |
| Fotos dos produtos | **S3 Standard** + **CloudFront** |
| Notas fiscais antigas | **Ciclo de vida** → **Glacier Deep Archive** |
| Segurança | **Security Groups encadeados**, **IAM Roles** nas instâncias, **Session Manager**, **EBS criptografado** |

### 9.3. Processamento de dados noturno (big data)
| Categoria | Escolha |
|---|---|
| Instâncias | **C** ou **M** em **Spot** |
| Arquivos temporários | **Instance Store** (instâncias com `d`) |
| Dados brutos | **S3 Intelligent-Tiering** (data lake) |
| Consultas | **Athena** direto no S3 |
| Dados antigos | **Ciclo de vida** → **Glacier Flexible Retrieval** |

### 9.4. Backup corporativo com exigência legal
| Categoria | Escolha |
|---|---|
| Backups recentes | **S3 Standard-IA** |
| Backups antigos | **Ciclo de vida** → **Glacier Deep Archive** |
| Proteção | **Versionamento** + **Object Lock** (modo compliance) + **MFA Delete** |
| Recuperação de desastres | **Replicação para outra região** (CRR) |
| Criptografia | **SSE-KMS** |
| Snapshots dos servidores | **EBS Snapshots** automatizados pelo **Data Lifecycle Manager**, com os antigos no **Snapshot Archive** |

### 9.5. Treinamento de modelo de IA
| Categoria | Escolha |
|---|---|
| Instâncias | **P** (GPU) ou **Trn** (Trainium) |
| Preço | **Spot** salvando o progresso (*checkpoints*) no S3, ou **On-Demand** se não puder parar |
| Dados de treino | **S3 Express One Zone** (máximo desempenho) ou **S3 Standard** |
| Modelos treinados | **S3 Standard** com **versionamento** |
| Servir o modelo (inferência) | Instâncias **Inf** ou **G** |
