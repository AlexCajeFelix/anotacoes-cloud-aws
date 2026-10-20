# 02 — AWS EC2: Tipos de Instância e Modelos de Preço

## 1. O que é o EC2

**EC2 (Elastic Compute Cloud)** é o serviço da AWS que fornece **servidores virtuais** (chamados de **instâncias**) sob demanda. Ele foi lançado em 2006 e é um dos pilares da cloud moderna (veja [01 — História](01-historia-da-cloud.md)).

O EC2 oferece **várias famílias de instância**, das mais simples (família **T**) às mais potentes. Cada família é otimizada para um tipo de uso.

---

## 2. Como ler o nome de uma instância

```
   m 7 g . xlarge
   │ │ │     │
   │ │ │     └── Tamanho (quanto de CPU e memória)
   │ │ └──────── Atributo (aqui: g = processador AWS Graviton)
   │ └────────── Geração (quanto maior, mais nova)
   └──────────── Família (para que ela é otimizada)
```

**Tamanhos** (do menor para o maior):
`nano` → `micro` → `small` → `medium` → `large` → `xlarge` → `2xlarge` → ... → `48xlarge` → `metal`

A partir de `large`, cada tamanho **dobra** a CPU e a memória do anterior.

> 💡 **Aprofundamento — atributos (letras depois da geração)**
>
> | Letra | Significado |
> |---|---|
> | `g` | Processador **AWS Graviton** (ARM). Costuma ter o melhor custo-benefício. |
> | `a` | Processador **AMD** |
> | `i` | Processador **Intel** |
> | `d` | Inclui **disco local NVMe** (armazenamento rápido na própria máquina, o [Instance Store](04-aws-instance-store.md)) |
> | `n` | **Rede** de alta performance |
> | `e` | **Memória ou armazenamento extra** |
> | `flex` | Versão mais barata, para quem não usa 100% da CPU o tempo todo |

---

## 3. Famílias de instância

### Uso geral (General Purpose)

| Família | Característica | Quando usar |
|---|---|---|
| **T** | **Burstable** (com "turbo"): desempenho base baixo, mas consegue picos de CPU quando precisa. A mais "de boa" e barata. | Sites pequenos, ambientes de teste e desenvolvimento, aplicações com pouco tráfego |
| **M** | **Equilibrada**: um pouco acima da T, com CPU, memória e rede balanceadas. Processamento leve a médio. | Servidores de aplicação, backends, bancos de dados pequenos e médios |

> 💡 **Aprofundamento — como funciona a família T (créditos de CPU)**
>
> - Enquanto a instância usa **menos** CPU do que a sua linha de base, ela **acumula créditos**.
> - Quando precisa de **mais** CPU (um pico de acesso, por exemplo), ela **gasta os créditos** e roda acima da linha de base.
> - No modo **Unlimited** (padrão nas T3 e T4g), ela pode passar do limite quando os créditos acabam, mas **cobra a mais** por isso.
>
> ⚠️ Se a sua aplicação usa CPU alta **o tempo todo**, a família T sai cara ou fica lenta. Nesse caso, use a **M** ou a **C**.

### Otimizada para computação (Compute Optimized)

| Família | Característica | Quando usar |
|---|---|---|
| **C** | **Muito poder de processamento** (CPU forte) em relação à memória | Processamento em lote (batch), servidores web de alto desempenho, servidores de jogos, codificação de vídeo, cálculos científicos |

### Otimizada para memória (Memory Optimized)

| Família | Característica | Quando usar |
|---|---|---|
| **R** | Muita **RAM** (R de RAM) | Bancos de dados relacionais grandes, cache (Redis, Memcached) |
| **X** | Memória **extrema** | SAP HANA, bancos de dados em memória gigantes |
| **z** | CPU de **alta frequência** com bastante memória | Softwares com licença cobrada por núcleo (core) |

### Otimizada para armazenamento (Storage Optimized)

| Família | Característica | Quando usar |
|---|---|---|
| **I** | Disco SSD NVMe local com **altíssimo IOPS** | Bancos NoSQL (Cassandra, MongoDB), bancos de dados transacionais |
| **D** | Muito armazenamento em **HDD** (denso) | Data warehouse, sistemas de arquivos distribuídos (Hadoop) |

### Computação acelerada (Accelerated Computing)

| Família | Característica | Quando usar |
|---|---|---|
| **P** | **GPU** de alto desempenho | **Treinamento** de modelos de IA e machine learning |
| **G** | **GPU** para gráficos | Renderização 3D, streaming de jogos, inferência de IA |
| **Inf** | Chip **AWS Inferentia** | **Inferência** de IA (rodar o modelo já treinado) com baixo custo |
| **Trn** | Chip **AWS Trainium** | **Treinamento** de IA com custo menor que GPU |
| **F** | **FPGA** (hardware programável) | Aplicações muito específicas: genômica, análise financeira |

> 💡 **Aprofundamento — a proporção CPU : memória**
>
> Comparando o mesmo tamanho (`large`, com 2 vCPUs) em famílias diferentes:
>
> | Instância | vCPU | Memória | Proporção |
> |---|---|---|---|
> | `c5.large` | 2 | 4 GiB | **1 : 2** (foco em CPU) |
> | `m5.large` | 2 | 8 GiB | **1 : 4** (equilibrada) |
> | `r5.large` | 2 | 16 GiB | **1 : 8** (foco em memória) |
>
> **Regra prática:** a C tem mais CPU por GB de memória, a M é equilibrada e a R tem mais memória por CPU.

### Dica para decorar

| Letra | Associação |
|---|---|
| **T** | **T**urbo (burstable) |
| **M** | **M**eio-termo (equilibrada) |
| **C** | **C**omputação |
| **R** | **R**AM |
| **X** | e**X**tra memória |
| **I** | **I**OPS |
| **D** | **D**enso (muito disco) |
| **P** / **G** | **P**rocessamento paralelo / **G**ráficos (GPU) |

---

## 4. Modelos de preço (como pagar pelo EC2)

### On-Demand (sob demanda)
- Paga **por segundo** de uso (mínimo de 60 segundos) para Linux e Windows. Alguns sistemas, como certas distribuições comerciais, são cobrados por hora.
- **Sem compromisso**: liga e desliga quando quiser.
- É o modelo **mais caro** por hora.
- **Quando usar:** testes, cargas imprevisíveis, projetos curtos, quando você ainda não sabe quanto vai usar.

### Savings Plans (planos de economia)
- Você se **compromete a gastar um valor fixo por hora** (ex.: US$ 10/hora) durante **1 ou 3 anos**.
- **Desconto de até 72%** em relação ao On-Demand.
- Dois tipos principais:

| Tipo | Desconto | Flexibilidade |
|---|---|---|
| **Compute Savings Plans** | Até **66%** | **Máxima**: vale para qualquer família, tamanho, região e sistema operacional, e também cobre **Fargate** e **Lambda** |
| **EC2 Instance Savings Plans** | Até **72%** | **Menor**: fica preso a uma família em uma região (ex.: M7g em São Paulo), mas pode trocar o tamanho |

- **Quando usar:** cargas estáveis e previsíveis. É o modelo **recomendado pela AWS** hoje no lugar das Reserved Instances.

### Reserved Instances (instâncias reservadas)
- Você **reserva uma instância específica** por **1 ou 3 anos**.
- **Desconto de até 72%**.
- **Standard**: maior desconto, mas não permite trocar a família.
- **Convertible**: desconto menor, mas permite trocar a família ou o sistema operacional durante o contrato.
- **Formas de pagamento:** *All Upfront* (tudo adiantado, maior desconto), *Partial Upfront* (parte adiantada) ou *No Upfront* (nada adiantado, menor desconto).
- **Quando usar:** servidores que ficam ligados 24/7 por anos (ex.: banco de dados de produção).

### Spot Instances
- Usa a **capacidade ociosa** dos datacenters da AWS.
- **Desconto de até 90%**: é o modelo **mais barato**.
- ⚠️ **A AWS pode tomar a instância de volta** a qualquer momento, com apenas **2 minutos de aviso**.
- **Quando usar:** tarefas que **toleram interrupção**: processamento em lote, renderização, big data, testes de CI/CD.
- **Não usar:** bancos de dados, sistemas críticos ou qualquer coisa que não pode parar.

> 💡 **Aprofundamento — outros modelos que caem em prova**
>
> | Modelo | O que é | Quando usar |
> |---|---|---|
> | **Dedicated Host** | Um **servidor físico inteiro só seu**, com visibilidade de sockets e núcleos. O mais caro. | Licenças cobradas por núcleo ou socket (BYOL: *Bring Your Own License*, como Oracle e Windows Server) e exigências de compliance |
> | **Dedicated Instance** | Roda em hardware exclusivo da sua conta, mas sem controle sobre o servidor físico | Isolamento físico por compliance, sem precisar gerenciar o host |
> | **Capacity Reservation** | **Garante capacidade** em uma zona de disponibilidade específica. **Não dá desconto sozinho**: você paga o preço On-Demand mesmo sem usar. Pode ser combinado com Savings Plans para ter desconto. | Eventos críticos em que você **não pode ficar sem servidor** (ex.: Black Friday) |

### Resumo comparativo

| Modelo | Desconto vs On-Demand | Compromisso | Pode ser interrompido? |
|---|---|---|---|
| **On-Demand** | — | Nenhum | Não |
| **Savings Plans** | Até 72% | 1 ou 3 anos (US$/hora) | Não |
| **Reserved Instances** | Até 72% | 1 ou 3 anos (instância) | Não |
| **Spot** | **Até 90%** | Nenhum | **Sim (aviso de 2 min)** |
| **Dedicated Host** | Mais caro | Opcional | Não |

> 💡 **Aprofundamento — analogia do hotel**
>
> - **On-Demand**: chegar no hotel e pagar a diária cheia.
> - **Savings Plans**: fechar um pacote "gasto R$ X por mês em hospedagem" em qualquer hotel da rede.
> - **Reserved**: reservar o mesmo quarto por 1 ou 3 anos com desconto.
> - **Spot**: pegar um quarto vazio bem barato, mas o hotel pode te tirar se aparecer alguém pagando o preço cheio.
> - **Dedicated Host**: alugar o prédio inteiro.

---

## 5. Como escolher (na prática)

1. **Comece com On-Demand** para entender o padrão de uso da aplicação.
2. Identificou uma **base de uso estável**? Cubra essa base com **Savings Plans**.
3. Tem tarefas que **podem ser interrompidas**? Use **Spot**.
4. Os picos imprevisíveis continuam em **On-Demand**.

> **Estratégia comum em empresas:** Savings Plans para a base + Spot para tarefas flexíveis + On-Demand para os picos.

---

## 6. Revisão rápida

- **Qual família é burstable e funciona com créditos de CPU?**
  A família T.
- **Qual família escolher para processamento pesado? E para muita memória RAM?**
  A C para processamento e a R para memória.
- **Qual é o modelo de preço mais barato e qual é o risco dele?**
  O Spot, com até 90% de desconto. O risco é a AWS interromper a instância com 2 minutos de aviso.
- **Qual a diferença entre Compute Savings Plans e EC2 Instance Savings Plans?**
  O Compute é mais flexível (qualquer família, região, Fargate e Lambda) e dá até 66% de desconto. O EC2 Instance fica preso a uma família em uma região e dá até 72%.
- **Qual modelo usar para uma licença de software cobrada por núcleo físico?**
  O Dedicated Host.
