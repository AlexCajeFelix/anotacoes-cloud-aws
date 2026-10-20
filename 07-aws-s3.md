# 07 — Amazon S3 (Simple Storage Service)

## 1. O que é o S3

O **Amazon S3 (Simple Storage Service)** é o serviço de **armazenamento de objetos** da AWS. Ele guarda qualquer tipo de arquivo (fotos, vídeos, backups, logs, documentos) e permite acessá-los de qualquer lugar pela internet ou pela rede da AWS.

- É **um dos serviços mais antigos da AWS**: foi lançado em **março de 2006**, como o **primeiro serviço** da AWS disponível para o público (veja [01 — História](01-historia-da-cloud.md)).
- Tem **altíssima escalabilidade, desempenho, segurança e durabilidade**.
- É **muito versátil**: aparece em praticamente todos os projetos, em vários momentos (backup, site, análise de dados, data lake...).

> 💡 **Aprofundamento — armazenamento de objeto vs bloco vs arquivo**
>
> | | **Objeto (S3)** | **Bloco (EBS)** | **Arquivo (EFS)** |
> |---|---|---|---|
> | **Como guarda** | Cada arquivo é um **objeto completo** (dados + metadados + chave) | Dados divididos em **blocos** | Em **pastas e arquivos**, como no seu computador |
> | **Como acessa** | Por **API / HTTP** (URL) | Conectado a **uma instância**, como um disco | Montado como **pasta compartilhada** (NFS) |
> | **Alterar um arquivo** | Precisa **enviar o objeto inteiro de novo** | Altera **só o bloco** que mudou | Altera parte do arquivo |
> | **Escala** | **Praticamente ilimitada** | Até 64 TiB por volume | Petabytes |
>
> Veja também [03 — EBS](03-aws-ebs.md).

---

## 2. Principais características

### Escalabilidade
- O armazenamento total é **praticamente ilimitado**: dá para guardar **petabytes ou mais** sem precisar planejar capacidade.
- Você **não reserva espaço**: paga só pelo que guarda.

### Durabilidade: 99,999999999% ("11 noves")
- **Durabilidade** é a garantia de que o dado **não vai ser perdido**.
- O S3 é projetado para **99,999999999%** de durabilidade, o famoso **"11 noves"**.
- Para conseguir isso, o S3 guarda cópias do dado em **pelo menos 3 zonas de disponibilidade** (exceto nas classes One Zone).

> 💡 **O que "11 noves" significa na prática**
>
> Se você guardar **10 milhões de objetos** no S3, a expectativa é **perder 1 objeto a cada 10 mil anos**, em média.

### Disponibilidade
- **Disponibilidade** é a garantia de que o dado **está acessível** quando você precisa.
- O **S3 Standard** é projetado para **99,99%** de disponibilidade, e o **SLA** (o que a AWS garante em contrato) é de **99,9%**.

> ⚠️ **Ajustes na anotação original**
>
> - A durabilidade correta é **99,999999999%**: são **11 noves** no total (2 antes da vírgula e 9 depois). Na anotação ficou com um 9 a mais.
> - **Durabilidade ≠ disponibilidade.** Durabilidade é **não perder** o dado. Disponibilidade é **conseguir acessar** o dado agora.
> - Sobre os 99,9%: esse é o valor **garantido em contrato (SLA)**. O S3 Standard é **projetado** para 99,99%.

### Desempenho
- **Baixa latência** (milissegundos) nas classes de acesso frequente.
- Alta velocidade de transferência, que pode ser aumentada com **multipart upload** e **Transfer Acceleration** (veja a [seção 5](#5-velocidade-de-transferência)).

### Segurança
- **Privado por padrão.**
- **Criptografia em repouso** (dados guardados) e **em trânsito** (dados viajando pela rede).
- Veja a [seção 4](#4-segurança).

### Consistência
> 💡 **Aprofundamento:** desde dezembro de 2020, o S3 tem **consistência forte**. Logo depois de gravar ou apagar um objeto, qualquer leitura já enxerga a versão nova.

---

## 3. Buckets, objetos e chaves

```
Bucket: minha-empresa-fotos
│
├── Chave: fotos/2025/ferias.jpg       → Objeto (o arquivo + metadados)
├── Chave: fotos/2025/natal.jpg        → Objeto
└── Chave: documentos/contrato.pdf     → Objeto
```

### 3.1. Bucket
- É o **"contêiner"** onde os objetos ficam guardados. Funciona como uma pasta principal.
- É criado em **uma região** específica (ex.: `sa-east-1`, São Paulo).
- **Regras do nome:**
  - Precisa ser **único no mundo inteiro**, entre **todas** as contas AWS.
  - De **3 a 63 caracteres**: letras **minúsculas**, números, hífens (`-`) e pontos (`.`).
  - Precisa começar e terminar com letra ou número.
- Cada conta pode ter **até 10.000 buckets** por padrão.

### 3.2. Objeto
Cada objeto é formado por:

| Parte | O que é |
|---|---|
| **Chave (key)** | O **nome único** do objeto dentro do bucket |
| **Dados (value)** | O conteúdo do arquivo |
| **Metadados** | Informações sobre o objeto (tipo de arquivo, data, tags personalizadas) |
| **ID de versão** | Identifica cada versão do objeto, se o versionamento estiver ativo |

**Tamanho de cada objeto:** de **0 bytes até 50 TB**.

> ⚠️ **Ajuste e atualização**
>
> - O objeto **não precisa ter** 5 TB: esse era o **tamanho máximo**. Ele pode ter qualquer tamanho até o limite.
> - Em **dezembro de 2025**, a AWS **aumentou o limite de 5 TB para 50 TB** por objeto. Muitos cursos e simulados ainda falam em **5 TB**, então na prova a resposta esperada pode ser essa.

> 💡 **Aprofundamento — limites de envio**
>
> | Forma de envio | Limite |
> |---|---|
> | **Upload simples** (um único PUT) | Até **5 GB** |
> | **Multipart upload** | **Recomendado** acima de **100 MB** e **obrigatório** acima de **5 GB**. Divide o arquivo em até 10.000 partes. |
> | **Download em uma única requisição** (GET) | Até **5 TB**. Para objetos maiores, baixe por partes. |

### 3.3. Chave (key)
- É o **identificador único** do objeto dentro do bucket, como um **ID** ou um "caminho".
- Bucket + chave identificam **qualquer objeto** de forma única:
  ```
  https://minha-empresa-fotos.s3.sa-east-1.amazonaws.com/fotos/2025/ferias.jpg
          └──── bucket ────┘    └─ região ─┘           └──── chave ────┘
  ```

> 💡 **Aprofundamento — o S3 não tem pastas de verdade**
>
> A estrutura do S3 é **plana**. O que parece uma pasta (`fotos/2025/`) é só **parte do nome da chave**, chamada de **prefixo**. O console mostra como pastas só para facilitar a navegação.

---

## 4. Segurança

### 4.1. Configurações padrão
Todo bucket novo já nasce **seguro**:
- **Privado**: só o dono da conta tem acesso.
- **Block Public Access ativado**: impede que o bucket ou os objetos fiquem públicos por engano.
- **ACLs desativadas**: o acesso é controlado só por políticas (o modelo mais simples e seguro).
- **Criptografia ativada**: todo objeto novo é criptografado automaticamente.

### 4.2. Criptografia em repouso (dados guardados)

| Tipo | Quem gerencia a chave | Quando usar |
|---|---|---|
| **SSE-S3** | A **AWS**, de forma automática. É o **padrão** e não tem custo. | A maioria dos casos |
| **SSE-KMS** | O **AWS KMS**: você controla quem usa a chave, e cada uso fica registrado no CloudTrail | Quando precisa de **controle e auditoria** das chaves |
| **DSSE-KMS** | KMS, com **duas camadas** de criptografia | Exigências rígidas de compliance |
| **SSE-C** | **Você** envia a chave em cada requisição. A AWS não guarda a chave. | Quando a chave precisa ficar fora da AWS |
| **Criptografia no cliente** | **Você** criptografa **antes de enviar** | Quando nem a AWS pode ver o conteúdo |

### 4.3. Criptografia em trânsito (dados viajando)
- O S3 aceita conexões por **HTTPS (TLS)**, que criptografa os dados durante a transferência.
- Dá para **obrigar o uso de HTTPS** com uma **bucket policy**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ExigirHTTPS",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::meu-bucket",
        "arn:aws:s3:::meu-bucket/*"
      ],
      "Condition": {
        "Bool": { "aws:SecureTransport": "false" }
      }
    }
  ]
}
```

### 4.4. Controle de acesso

| Forma | Como funciona |
|---|---|
| **Políticas do IAM** | Definem o que **usuários e roles** podem fazer no S3 (veja [06 — IAM](06-aws-iam.md)) |
| **Bucket policy** | Política **anexada ao bucket** (política baseada em recurso). Define **quem** pode acessar **aquele bucket**, inclusive de outras contas. |
| **URL pré-assinada** (*presigned URL*) | Um **link temporário** que permite baixar ou enviar um objeto privado sem dar acesso ao bucket inteiro. Ex.: link de download que expira em 1 hora. |
| **VPC Endpoint** | Permite que instâncias da VPC acessem o S3 **sem passar pela internet** |

---

## 5. Velocidade de transferência

A transferência no S3 já é rápida, e existem formas de acelerar ainda mais.

### 5.1. Multipart upload
- Divide arquivos grandes em **partes enviadas em paralelo**.
- Se uma parte falhar, só ela é reenviada, e não o arquivo inteiro.

### 5.2. S3 Transfer Acceleration
- **Acelera envios e downloads de longa distância** (ex.: usuários no Brasil enviando arquivos para um bucket nos EUA).
- **Como funciona:** os dados entram na rede da AWS pelo **ponto de presença (edge location) do CloudFront mais próximo** do usuário e seguem pela **rede privada da AWS**, que é mais rápida e estável que a internet pública.
- Usa um endereço especial: `meu-bucket.s3-accelerate.amazonaws.com`.
- **Custo extra** por GB transferido, mas a AWS **só cobra se a transferência realmente ficar mais rápida**.

```
Sem Acceleration:  Usuário (Brasil) ──── internet pública (longa) ────▶ Bucket (EUA)

Com Acceleration:  Usuário (Brasil) ──▶ Edge location (São Paulo) ════ rede da AWS ════▶ Bucket (EUA)
```

> 💡 **Aprofundamento — desempenho por prefixo**
>
> O S3 suporta pelo menos **3.500 requisições de escrita** (PUT/COPY/POST/DELETE) e **5.500 de leitura** (GET/HEAD) **por segundo, por prefixo**. Para aplicações com altíssimo volume, dá para **espalhar os objetos em vários prefixos** e multiplicar esse limite.

---

## 6. Casos de uso

| Uso | Como o S3 ajuda |
|---|---|
| **Backup e armazenamento de dados** | Guarda cópias de segurança com durabilidade de 11 noves. As classes Glacier deixam o arquivamento muito barato. |
| **Hospedagem de sites estáticos** | Serve sites feitos só de HTML, CSS, JavaScript e imagens, **sem servidor** (veja o aprofundamento abaixo). |
| **Big data e analytics** | Guarda grandes volumes de dados que são analisados com **Athena** (SQL direto no S3), **EMR** (Spark, Hadoop) e **Redshift**. |
| **Distribuição de conteúdo** | Junto com o **CloudFront** (a CDN da AWS), entrega imagens, vídeos e downloads rapidamente para usuários do mundo todo. |
| **Data lake** | Repositório central com **todos os dados da empresa** (estruturados ou não), no formato original, para análise posterior. |
| **Amazon Redshift** | O data warehouse da AWS **carrega dados do S3** (comando `COPY`) e também **consulta dados direto no S3**, sem carregar (**Redshift Spectrum**). |

> ⚠️ Interpretei "distribuição contínua" como **distribuição de conteúdo** (com CloudFront). Se era outra coisa, me avise.
> E "red shift" é o **Amazon Redshift**.

> 💡 **Aprofundamento — hospedando um site estático no S3**
>
> 1. Criar o bucket e enviar os arquivos do site (`index.html`, CSS, imagens).
> 2. Ativar **Static website hosting** e definir a página inicial (`index.html`) e a de erro (`error.html`).
> 3. O site fica disponível em um endereço como `http://meu-bucket.s3-website-sa-east-1.amazonaws.com`.
>
> **Limitações:**
> - Só funciona com **HTTP**. Para ter **HTTPS** e domínio próprio, use o **CloudFront** na frente do bucket.
> - Só serve **conteúdo estático**. Não roda código no servidor (PHP, Node.js, Python).
>
> **Forma recomendada:** **CloudFront + S3 com OAC** (Origin Access Control). O bucket continua **privado**, e só o CloudFront consegue ler os arquivos.

---

## 7. Classes de armazenamento

Cada classe tem um **equilíbrio diferente entre preço de armazenamento, preço de acesso e velocidade de recuperação**. A regra geral: **quanto menos você acessa, mais barato fica guardar, e mais caro (e mais lento) fica recuperar.**

### S3 Standard
- Para dados de **acesso frequente**.
- **Baixa latência e alta velocidade.**
- Sem taxa de recuperação e sem tempo mínimo de armazenamento.
- **Quando usar:** sites, aplicações, conteúdo para usuários, análise de dados ativa.

### S3 Intelligent-Tiering
- Para dados com **padrão de acesso desconhecido ou que muda**.
- O S3 **monitora o uso de cada objeto** e **move automaticamente** para a camada mais barata:

| Camada | Quando o objeto vai para ela | Economia aproximada vs Standard |
|---|---|---|
| **Frequent Access** | Padrão (ou quando o objeto é acessado de novo) | — |
| **Infrequent Access** | Após **30 dias** sem acesso | ~40% |
| **Archive Instant Access** | Após **90 dias** sem acesso | ~68% |
| **Archive Access** *(opcional)* | Após 90 dias ou mais (você configura) | Igual ao Glacier Flexible |
| **Deep Archive Access** *(opcional)* | Após 180 dias ou mais (você configura) | Igual ao Glacier Deep Archive |

- **Sem taxa de recuperação**: se o objeto for acessado, volta para a camada frequente sem custo extra.
- Cobra uma **pequena taxa de monitoramento** por objeto.
- Objetos **menores que 128 KB** não são monitorados e ficam sempre na camada frequente.
- **Quando usar:** quando você **não sabe** com que frequência os dados vão ser acessados.

### S3 Standard-IA (Infrequent Access)
- Para dados **acessados com pouca frequência**, mas que precisam de **acesso rápido** quando necessário.
- Armazenamento **mais barato** que o Standard, mas **cobra por GB recuperado**.
- **Tempo mínimo de 30 dias** e **tamanho mínimo cobrado de 128 KB** por objeto.
- **Quando usar:** backups recentes, recuperação de desastres, arquivos acessados cerca de uma vez por mês.

### S3 One Zone-IA
- Igual ao Standard-IA, mas guarda os dados em **apenas uma zona de disponibilidade**.
- **Cerca de 20% mais barato** que o Standard-IA.
- ⚠️ Se aquela zona tiver um desastre, **os dados podem ser perdidos**.
- **Quando usar:** dados **fáceis de recriar** ou **cópias secundárias** de backups.

### S3 Glacier Instant Retrieval
- Para dados de **arquivo** acessados **raramente** (cerca de uma vez por trimestre), mas que precisam ser recuperados **na hora** (milissegundos).
- **Tempo mínimo de 90 dias.**
- **Quando usar:** imagens médicas, arquivos de notícias, conteúdo antigo que ainda pode ser pedido a qualquer momento.

### S3 Glacier Flexible Retrieval
- Para **arquivamento** de dados acessados **1 ou 2 vezes por ano**, em que **esperar minutos ou horas** não é problema.
- **Tempo mínimo de 90 dias.**
- Opções de recuperação:

| Opção | Tempo | Custo |
|---|---|---|
| **Expedited** (urgente) | 1 a 5 minutos | O mais caro |
| **Standard** | 3 a 5 horas | Intermediário |
| **Bulk** (em lote) | 5 a 12 horas | **Gratuito** |

- **Quando usar:** backups antigos, arquivos de mídia, dados que raramente são consultados.

### S3 Glacier Deep Archive
- A **classe mais barata** de todo o S3 (cerca de **US$ 1 por TB por mês**).
- Para dados que **quase nunca** são acessados e precisam ser **guardados por anos**.
- **Tempo mínimo de 180 dias.**
- Recuperação: **até 12 horas** (Standard) ou **até 48 horas** (Bulk).
- **Quando usar:** **compliance** e exigências legais (ex.: guardar registros financeiros ou de saúde por 7 a 10 anos), substituto de fitas de backup.

> 💡 **Aprofundamento — S3 Express One Zone**
>
> Existe também a classe **S3 Express One Zone**, a **mais rápida** do S3: latência de **milissegundos de um dígito**, até **10 vezes mais rápida** que o Standard. Ela fica em **uma única zona** e usa um tipo especial de bucket (*directory bucket*). É usada em **machine learning** e **análises** que precisam de altíssimo desempenho. O armazenamento é **mais caro** por GB, mas as requisições são mais baratas.

### Resumo das classes

| Classe | Frequência de acesso | AZs | Tempo mínimo | Recuperação | Taxa de recuperação |
|---|---|---|---|---|---|
| **Standard** | Frequente | ≥ 3 | — | Milissegundos | Não |
| **Intelligent-Tiering** | Desconhecida ou variável | ≥ 3 | — | Milissegundos* | Não |
| **Standard-IA** | ~1 vez por mês | ≥ 3 | 30 dias | Milissegundos | Sim |
| **One Zone-IA** | ~1 vez por mês (dados recriáveis) | **1** | 30 dias | Milissegundos | Sim |
| **Glacier Instant Retrieval** | ~1 vez por trimestre | ≥ 3 | 90 dias | Milissegundos | Sim |
| **Glacier Flexible Retrieval** | 1–2 vezes por ano | ≥ 3 | 90 dias | Minutos a horas | Sim (Bulk gratuito) |
| **Glacier Deep Archive** | Quase nunca | ≥ 3 | 180 dias | 12 a 48 horas | Sim |

\* Nas camadas opcionais de arquivamento do Intelligent-Tiering, a recuperação leva horas.

> ⚠️ **Tempo mínimo de armazenamento:** se você **apagar ou mover** um objeto **antes** do tempo mínimo, paga **como se ele tivesse ficado o período todo**. Ex.: apagar um objeto do Deep Archive depois de 30 dias cobra os 180 dias.

> 💡 **Analogia: onde guardar as coisas de casa**
>
> - **Standard** = a **mesa de trabalho**: tudo à mão.
> - **Standard-IA** = o **armário do quarto**: perto, mas você abre de vez em quando.
> - **One Zone-IA** = um armário em **um único cômodo**: se esse cômodo pegar fogo, você perde.
> - **Glacier Instant Retrieval** = o **depósito do prédio**: pouco usado, mas você pega na hora.
> - **Glacier Flexible Retrieval** = um **guarda-móveis na cidade**: precisa agendar a retirada (horas).
> - **Glacier Deep Archive** = um **guarda-móveis em outra cidade**: o mais barato, mas leva 1 a 2 dias para buscar.
> - **Intelligent-Tiering** = um **organizador** que move as coisas sozinho conforme você usa.

### Ciclo de vida (Lifecycle): movendo os dados automaticamente

As **regras de ciclo de vida** movem os objetos entre as classes, ou apagam, **automaticamente** com base na idade:

```
Dia 0            Dia 30            Dia 90                 Dia 365              7 anos
  │                │                 │                      │                    │
Standard ──▶ Standard-IA ──▶ Glacier Flexible ──▶ Glacier Deep Archive ──▶ Excluir
```

- Exemplo real: **logs de uma aplicação**. São muito consultados na primeira semana, pouco consultados depois de um mês e só precisam ser guardados por exigência legal depois de um ano.
- O objeto precisa ficar **pelo menos 30 dias** no Standard antes de ir para o Standard-IA ou para o One Zone-IA.

---

## 8. Custos

O custo do S3 depende da **sua estratégia**: **quanto você guarda**, **em qual classe**, **quantas requisições faz** e **quanto dado você tira** do S3.

### 8.1. O que é cobrado

| Item | Detalhe |
|---|---|
| **Armazenamento** | Por **GB por mês**, conforme a **classe** e o **volume total** (quanto mais você guarda, menor o preço por GB) |
| **Requisições** | Cada operação é cobrada por **lote de 1.000**: PUT, COPY, POST, LIST, GET... |
| **Recuperação** | Por **GB recuperado** nas classes IA e Glacier |
| **Transferência de dados** | **Entrada é gratuita**. **Saída para a internet** é cobrada por GB. |
| **Gerenciamento** | Monitoramento do Intelligent-Tiering, Storage Lens avançado, inventário |
| **Recursos extras** | Transfer Acceleration, replicação entre regiões |

### 8.2. Valores de referência (região us-east-1)

> Valores **aproximados**, para estudo e comparação. Eles **variam por região** (São Paulo é mais cara) e podem mudar. Consulte sempre a [página oficial de preços](https://aws.amazon.com/s3/pricing/) e simule na [AWS Pricing Calculator](https://calculator.aws/).

**Armazenamento: S3 Standard, com preço por faixa de volume:**

| Faixa de volume (por mês) | Preço por GB |
|---|---|
| **Primeiros 50 TB** | ~US$ 0,023 |
| **Próximos 450 TB** (de 50 a 500 TB) | ~US$ 0,022 |
| **Acima de 500 TB** | ~US$ 0,021 |

**Armazenamento: outras classes:**

| Classe | Preço por GB/mês | Recuperação por GB |
|---|---|---|
| **Standard** | ~US$ 0,023 | — |
| **Intelligent-Tiering** | De ~US$ 0,023 a ~US$ 0,004, conforme a camada, mais ~US$ 0,0025 por 1.000 objetos monitorados | — |
| **Standard-IA** | ~US$ 0,0125 | ~US$ 0,01 |
| **One Zone-IA** | ~US$ 0,01 | ~US$ 0,01 |
| **Glacier Instant Retrieval** | ~US$ 0,004 | ~US$ 0,03 |
| **Glacier Flexible Retrieval** | ~US$ 0,0036 | Expedited ~US$ 0,03 · Standard ~US$ 0,01 · Bulk grátis |
| **Glacier Deep Archive** | ~US$ 0,00099 | Standard ~US$ 0,02 · Bulk ~US$ 0,0025 |

**Requisições (S3 Standard):**

| Tipo de requisição | Preço |
|---|---|
| **PUT, COPY, POST, LIST** (escrita e listagem) | ~US$ 0,005 por 1.000 |
| **GET, SELECT** e outras leituras | ~US$ 0,0004 por 1.000 |
| **DELETE e CANCEL** | **Gratuito** |

> Nas classes IA e Glacier, as requisições são **mais caras** que no Standard. As **transições de ciclo de vida** também são cobradas como requisições.

**Transferência de dados:**

| Tipo | Preço |
|---|---|
| **Entrada** (internet → S3) | **Gratuita** |
| **S3 → EC2 na mesma região** | **Gratuita** |
| **S3 → CloudFront** | **Gratuita** |
| **Saída para a internet**: primeiros **100 GB/mês** (somando todos os serviços da AWS) | **Gratuita** |
| **Saída para a internet**: até 10 TB/mês | ~US$ 0,09 por GB |
| **Saída para a internet**: próximos 40 TB/mês | ~US$ 0,085 por GB |
| **Saída para a internet**: próximos 100 TB/mês | ~US$ 0,07 por GB |
| **Saída para a internet**: acima de 150 TB/mês | ~US$ 0,05 por GB |
| **Para outra região da AWS** | ~US$ 0,02 por GB |
| **Transfer Acceleration** (adicional) | ~US$ 0,04 a ~US$ 0,08 por GB |

### 8.3. Exemplos de cálculo

**Guardar 1 TB (1.024 GB) por mês em cada classe:**

| Classe | Cálculo | Total/mês |
|---|---|---|
| Standard | 1.024 × 0,023 | **~US$ 23,55** |
| Standard-IA | 1.024 × 0,0125 | **~US$ 12,80** |
| One Zone-IA | 1.024 × 0,01 | **~US$ 10,24** |
| Glacier Instant Retrieval | 1.024 × 0,004 | **~US$ 4,10** |
| Glacier Flexible Retrieval | 1.024 × 0,0036 | **~US$ 3,69** |
| Glacier Deep Archive | 1.024 × 0,00099 | **~US$ 1,01** |

**Preço por faixa: guardar 60 TB no Standard:**

| Faixa | Cálculo | Valor |
|---|---|---|
| Primeiros 50 TB (51.200 GB) | 51.200 × 0,023 | US$ 1.177,60 |
| Próximos 10 TB (10.240 GB) | 10.240 × 0,022 | US$ 225,28 |
| **Total** | | **~US$ 1.402,88** |

**Requisições e transferência:**

| Operação | Cálculo | Total |
|---|---|---|
| Enviar 1 milhão de arquivos (PUT) | 1.000 lotes × 0,005 | **~US$ 5,00** |
| Ler 10 milhões de vezes (GET) | 10.000 lotes × 0,0004 | **~US$ 4,00** |
| Baixar 500 GB para a internet | (500 − 100 grátis) × 0,09 | **~US$ 36,00** |

> **Lição:** em muitos projetos, a **transferência de saída** e as **requisições** custam **mais do que o armazenamento**. Vale olhar o padrão de uso, e não só o volume guardado.

### 8.4. Estratégias para economizar
- **Escolha a classe certa** para cada tipo de dado.
- Use **regras de ciclo de vida** para mover os dados antigos para classes mais baratas e **apagar** o que não precisa mais.
- Use o **Intelligent-Tiering** quando não souber o padrão de acesso.
- Use o **CloudFront** na frente do S3: a transferência S3 → CloudFront é gratuita, e o cache reduz as requisições.
- **Apague uploads multipart incompletos** (partes de envios que falharam continuam sendo cobradas). Dá para fazer isso com uma regra de ciclo de vida.
- Se o **versionamento** estiver ativo, configure a exclusão das **versões antigas**.
- **Evite objetos muito pequenos** nas classes IA e Glacier, por causa do tamanho mínimo cobrado.
- Use o **Storage Class Analysis** e o **S3 Storage Lens** para descobrir quais dados podem ir para classes mais baratas.

---

## 9. Outros recursos importantes

> 💡 **Aprofundamento**
>
> | Recurso | O que faz |
> |---|---|
> | **Versionamento** | Guarda **todas as versões** de um objeto. Protege contra exclusão ou sobrescrita acidental. |
> | **MFA Delete** | Exige **MFA** para apagar versões ou desativar o versionamento. |
> | **Object Lock** | Impede que objetos sejam **apagados ou alterados** por um período (modelo **WORM**: *write once, read many*). Usado para compliance e proteção contra ransomware. |
> | **Replicação** | Copia objetos automaticamente para **outro bucket**, na mesma região (**SRR**) ou em outra região (**CRR**). |
> | **Notificações de eventos** | Dispara uma ação (ex.: uma função **Lambda**) quando um objeto é criado ou apagado. Ex.: gerar uma miniatura sempre que uma foto é enviada. |
> | **S3 Storage Lens** | Painel com visão geral do uso e do custo de todos os buckets. |

---

## 10. Links oficiais

**Geral**
- [Amazon S3: página do produto](https://aws.amazon.com/s3/)
- [Guia do usuário do S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [Perguntas frequentes (FAQ)](https://aws.amazon.com/s3/faqs/)

**Buckets e objetos**
- [Regras de nomes de bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)
- [Visão geral de objetos](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingObjects.html)
- [Multipart upload](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
- [Anúncio: objetos de até 50 TB (dez/2025)](https://aws.amazon.com/about-aws/whats-new/2025/12/amazon-s3-maximum-object-size-50-tb/)

**Classes de armazenamento**
- [Visão geral das classes](https://aws.amazon.com/s3/storage-classes/)
- [Documentação das classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)
- [S3 Intelligent-Tiering](https://aws.amazon.com/s3/storage-classes/intelligent-tiering/)
- [S3 Glacier (Instant, Flexible e Deep Archive)](https://aws.amazon.com/s3/storage-classes/glacier/)
- [S3 Express One Zone](https://aws.amazon.com/s3/storage-classes/express-one-zone/)
- [Regras de ciclo de vida](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)
- [Storage Class Analysis](https://docs.aws.amazon.com/AmazonS3/latest/userguide/analytics-storage-class.html)

**Segurança**
- [Criptografia no S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html)
- [Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [Versionamento](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
- [Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)
- [Replicação](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)

**Desempenho e transferência**
- [S3 Transfer Acceleration](https://aws.amazon.com/s3/transfer-acceleration/)
- [Documentação do Transfer Acceleration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/transfer-acceleration.html)
- [Boas práticas de desempenho](https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html)

**Casos de uso**
- [Hospedagem de site estático](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [Amazon CloudFront](https://aws.amazon.com/cloudfront/)
- [Amazon Athena](https://aws.amazon.com/athena/)
- [Redshift Spectrum](https://docs.aws.amazon.com/redshift/latest/dg/c-using-spectrum.html)

**Custos**
- [Preços do S3](https://aws.amazon.com/s3/pricing/)
- [AWS Pricing Calculator](https://calculator.aws/)

---

## 11. Revisão rápida

- **Que tipo de armazenamento é o S3?**
  Armazenamento de objetos.
- **Qual é a durabilidade do S3? E a disponibilidade do S3 Standard?**
  Durabilidade de 99,999999999% (11 noves). Disponibilidade projetada de 99,99%, com SLA de 99,9%.
- **Qual é a diferença entre durabilidade e disponibilidade?**
  Durabilidade é não perder o dado. Disponibilidade é conseguir acessar o dado quando precisa.
- **O nome de um bucket precisa ser único onde?**
  No mundo inteiro, entre todas as contas AWS.
- **Qual é o tamanho máximo de um objeto?**
  50 TB desde dezembro de 2025. Muitos materiais ainda citam 5 TB.
- **A partir de qual tamanho o multipart upload é obrigatório?**
  Acima de 5 GB. É recomendado a partir de 100 MB.
- **O que é a chave (key) de um objeto?**
  O identificador único do objeto dentro do bucket, como um caminho (ex.: `fotos/2025/ferias.jpg`).
- **Como o Transfer Acceleration deixa a transferência mais rápida?**
  Os dados entram pela edge location mais próxima e seguem pela rede privada da AWS.
- **Qual classe usar quando o padrão de acesso é desconhecido?**
  O S3 Intelligent-Tiering.
- **Qual é a classe mais barata? E qual o tempo de recuperação dela?**
  O Glacier Deep Archive, com recuperação de até 12 horas (Standard) ou 48 horas (Bulk).
- **Qual é o risco do One Zone-IA?**
  Os dados ficam em uma única AZ e podem ser perdidos se ela tiver um desastre.
- **Qual classe de arquivamento recupera os dados em milissegundos?**
  O Glacier Instant Retrieval.
- **O que acontece se apagar um objeto do Standard-IA depois de 10 dias?**
  Você paga os 30 dias mínimos mesmo assim.
- **Quanto custa enviar dados para o S3? E tirar dados para a internet?**
  Enviar (entrada) é gratuito. A saída para a internet tem 100 GB grátis por mês e depois cerca de US$ 0,09 por GB.
- **Como obrigar o uso de HTTPS em um bucket?**
  Com uma bucket policy que nega acesso quando `aws:SecureTransport` é `false`.
- **Qual recurso move objetos entre classes automaticamente com base na idade?**
  As regras de ciclo de vida (Lifecycle).
