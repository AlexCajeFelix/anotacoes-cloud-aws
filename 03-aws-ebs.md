# 03 — AWS EBS (Elastic Block Store)

## 1. O que é o EBS

O **EBS (Elastic Block Store)** é o serviço de **armazenamento em bloco** da AWS usado pelas instâncias EC2. Ele funciona como os **discos rígidos (HD/SSD)** dos computadores tradicionais: a instância enxerga o volume EBS como um disco, que pode ser formatado e usado normalmente.

> ⚠️ **Ajustes na anotação original**
>
> - "Banco de dados rígidos" → o correto é **discos rígidos**. O EBS é um disco virtual, não um banco de dados.
> - "SC2" → **EC2**.

> 💡 **Analogia**
>
> O EBS é como um **HD externo conectado pela rede** ao servidor. Dá para desconectar o volume de uma instância e conectar em outra, desde que as duas estejam na **mesma zona de disponibilidade**.

---

## 2. Características principais

### Persistência
- Os dados do EBS **continuam existindo mesmo que a instância EC2 seja desligada (stop)**.
- O ciclo de vida do volume é **independente** do ciclo de vida da instância.

> ⚠️ **Detalhe importante (cai muito em prova)**
>
> | Ação na instância | O que acontece com o EBS |
> |---|---|
> | **Reboot** (reiniciar) | Os dados permanecem |
> | **Stop** (desligar) | Os dados permanecem |
> | **Terminate** (excluir a instância) | O **volume raiz (root) é apagado por padrão**. Os volumes adicionais **são mantidos** por padrão. |
>
> Esse comportamento é controlado pela opção **`DeleteOnTermination`**. Se você precisa manter o volume raiz depois de excluir a instância, desmarque essa opção.

### Vinculado a uma zona de disponibilidade (AZ)
- Cada volume EBS existe em **uma única AZ** (ex.: `sa-east-1a`).
- Ele só pode ser conectado a instâncias **da mesma AZ**.
- Para levar os dados para outra AZ ou região, use um **snapshot** (veja a [seção 6](#6-snapshots-backups-do-ebs)).

### Conexão com as instâncias
- Normalmente, **um volume é conectado a uma instância por vez**.
- Uma instância pode ter **vários volumes** conectados.
- **Exceção:** os volumes **io1 e io2** suportam **Multi-Attach**, que conecta o mesmo volume a até 16 instâncias da mesma AZ.

### Replicação
- O EBS faz **replicação automática** dentro da própria AZ, para proteger contra falha de hardware.

---

## 3. Métricas de desempenho: IOPS, Throughput e Latência

Para escolher o volume certo, é preciso entender as **três medidas de desempenho** de um disco.

### IOPS (Input/Output Operations Per Second)
- **Operações de entrada e saída por segundo.**
- Mede **quantas** leituras e escritas o disco consegue fazer **por segundo**, sem importar o tamanho de cada uma.
- Importa para cargas com **muitos acessos pequenos e aleatórios**, como **bancos de dados**.
- Exemplo: um volume de 3.000 IOPS faz até 3.000 leituras ou escritas por segundo.

### Throughput (vazão)
- Mede a **quantidade de dados** que pode ser **lida e escrita por segundo**.
- É medido em **MiB/s** (megabytes por segundo).
- Importa para **arquivos grandes lidos em sequência**, como logs, big data e vídeo.

### Latência
- É o **tempo necessário** para **completar uma operação** de leitura ou escrita.
- É medida em **milissegundos (ms)**. Quanto **menor**, melhor.
- **SSD:** latência baixa (de menos de 1 ms até poucos ms). **HDD:** latência mais alta.

> 💡 **Analogia da estrada**
>
> - **IOPS** = **quantos carros** passam por segundo.
> - **Throughput** = **quanta carga** (em toneladas) passa por segundo.
> - **Latência** = **quanto tempo cada carro leva** para chegar ao destino.
>
> Muitos carros pequenos (alto IOPS) é o padrão dos **bancos de dados**. Poucos caminhões grandes (alto throughput) é o padrão de **big data**.

> 💡 **Aprofundamento — a relação entre IOPS e throughput**
>
> **Throughput = IOPS × tamanho de cada operação**
>
> - 3.000 IOPS × 16 KiB por operação ≈ **47 MiB/s**
> - 500 IOPS × 1 MiB por operação = **500 MiB/s**
>
> Por isso um HDD pode ter **IOPS baixo** e mesmo assim **throughput alto**: ele faz poucas operações, mas cada uma lê um bloco grande.

> ⚠️ **A instância também tem limite**
>
> Cada tipo de instância EC2 tem um **limite máximo de IOPS e throughput para o EBS**. Não adianta colocar um volume io2 de 256.000 IOPS em uma instância pequena: o desempenho vai ficar limitado pela instância. Confira o limite do tipo de instância na documentação.

---

## 4. Tipos de volume

O EBS tem duas categorias:
- **SSD:** foco em **IOPS** e baixa latência (operações pequenas e aleatórias).
- **HDD:** foco em **throughput** e baixo custo (leitura sequencial de grandes volumes).

### 4.1. SSD

#### General Purpose SSD (gp2 e gp3)
- **Equilíbrio entre preço e desempenho.**
- Atendem a uma **ampla gama de cargas de trabalho**.
- **Quando usar:** volumes de boot, aplicações web, ambientes de desenvolvimento e teste, desktops virtuais, bancos de dados pequenos e médios.
- O **gp3** é o padrão recomendado hoje. O **gp2** é a geração anterior, e o ideal é migrar para o gp3.

**Valores do gp2:**

| Item | Valor |
|---|---|
| Tamanho | 1 GiB – 16 TiB |
| IOPS | **3 IOPS por GiB** (mínimo 100, máximo **16.000**) |
| Burst | Volumes **menores que 1 TiB** podem ter picos de até **3.000 IOPS**, usando créditos |
| Throughput máximo | 250 MiB/s |
| Latência | Poucos milissegundos |

> O IOPS do gp2 **depende do tamanho**: um volume de 100 GiB tem 300 IOPS de base. Para chegar aos 16.000 IOPS, o volume precisa ter cerca de **5.334 GiB**, mesmo que você não precise de todo esse espaço.

**Valores do gp3:**

| Item | Valor |
|---|---|
| Tamanho | 1 GiB – **64 TiB** |
| IOPS de base (incluído no preço) | **3.000 IOPS**, em **qualquer tamanho** |
| Throughput de base (incluído no preço) | **125 MiB/s** |
| IOPS máximo (pago à parte) | **80.000** (até 500 IOPS por GiB) |
| Throughput máximo (pago à parte) | **2.000 MiB/s** |
| Latência | Poucos milissegundos |

> No gp3, o desempenho **não depende do tamanho**. Um volume de 10 GiB já tem 3.000 IOPS, e você compra IOPS ou throughput extra **separadamente**, só se precisar. Ele é cerca de **20% mais barato por GB** do que o gp2.

#### Provisioned IOPS SSD (io1 e io2)
- Feitos para **cargas de trabalho intensivas de I/O** (muita leitura e escrita), que são as **mais pesadas**.
- Você **define (provisiona) exatamente quantos IOPS** o volume deve entregar, e o desempenho fica **constante**.
- **Regra clássica:** quando a aplicação precisa de **mais de 16.000 IOPS**, use io1 ou io2.
- **Quando usar:** **bancos de dados críticos e grandes** (Oracle, SQL Server, SAP HANA), aplicações sensíveis à latência.
- Suportam **Multi-Attach** (mesmo volume em várias instâncias).
- O **io2** é a evolução do io1: mais durável e com mais desempenho pelo mesmo preço.

**Valores do io1 e do io2:**

| Item | io1 | io2 Block Express |
|---|---|---|
| Tamanho | 4 GiB – 16 TiB | 4 GiB – **64 TiB** |
| IOPS máximo | 64.000 (até 50 IOPS por GiB) | **256.000** (até 1.000 IOPS por GiB) |
| Throughput máximo | 1.000 MiB/s | **4.000 MiB/s** |
| Latência | Poucos milissegundos | **Menos de 1 milissegundo** |
| Durabilidade | 99,8% – 99,9% | **99,999%** |

> ⚠️ **Atualização importante sobre o limite de 16.000 IOPS**
>
> A regra "mais de 16.000 IOPS → io1/io2" vem do limite **antigo** do gp2 e do gp3, e ainda aparece em muitos cursos e simulados.
>
> Desde **setembro de 2025**, o **gp3 suporta até 80.000 IOPS**, 2.000 MiB/s e 64 TiB. O **gp2 continua limitado a 16.000 IOPS**.
>
> **Então, por que ainda usar io2?**
> - Precisa de **mais de 80.000 IOPS** (o io2 chega a 256.000).
> - Precisa de **latência abaixo de 1 milissegundo**, constante.
> - Precisa de **durabilidade de 99,999%** (o gp3 tem 99,8% a 99,9%).
> - Precisa de **Multi-Attach**.
>
> **Na prova:** se a questão falar em "mais de 16.000 IOPS", a resposta esperada costuma ser **io1/io2**, porque os simulados geralmente seguem a regra antiga.

### 4.2. HDD

#### Throughput Optimized HDD (st1)
- **Otimizado para throughput**: ler e gravar **grandes quantidades de dados em sequência**.
- **Quando usar:** **big data**, **data warehouse**, processamento de logs, ETL, streaming.

#### Cold HDD (sc1)
- O HDD **"frio"**, para dados **acessados com pouca frequência**.
- É o **mais barato** de todos os volumes EBS.
- **Quando usar:** dados guardados que quase não são lidos, quando o **menor custo** é o mais importante.

**Valores do st1 e do sc1:**

| Item | st1 | sc1 |
|---|---|---|
| Tamanho | 125 GiB – 16 TiB | 125 GiB – 16 TiB |
| Throughput máximo | **500 MiB/s** | 250 MiB/s |
| IOPS máximo (operações de 1 MiB) | 500 | 250 |
| Throughput de base | 40 MiB/s por TiB | 12 MiB/s por TiB |
| Burst | 250 MiB/s por TiB | 80 MiB/s por TiB |

> ⚠️ **Ajuste na anotação original**
>
> **Big data e data warehouse** são casos de uso do **st1** (Throughput Optimized), e **não do sc1**. O **sc1** é para dados **acessados com pouca frequência**.

> ⚠️ Volumes **HDD (st1 e sc1)**:
> - **não podem ser usados como volume de boot** (disco do sistema operacional);
> - **não suportam Multi-Attach**.

### 4.3. Resumo de todos os tipos

| Tipo | Categoria | Foco | IOPS máx. | Throughput máx. | Boot? | Uso típico |
|---|---|---|---|---|---|---|
| **gp3** | SSD | Equilíbrio preço/desempenho | 80.000 | 2.000 MiB/s | ✅ | **Padrão para quase tudo** |
| **gp2** | SSD | Equilíbrio (geração anterior) | 16.000 | 250 MiB/s | ✅ | Sistemas antigos |
| **io2** | SSD | IOPS alto e constante | 256.000 | 4.000 MiB/s | ✅ | Bancos de dados críticos |
| **io1** | SSD | IOPS alto (geração anterior) | 64.000 | 1.000 MiB/s | ✅ | Bancos de dados críticos (sistemas antigos) |
| **st1** | HDD | Throughput | 500 | 500 MiB/s | ❌ | Big data, data warehouse, logs |
| **sc1** | HDD | Menor custo | 250 | 250 MiB/s | ❌ | Dados pouco acessados |

---

## 5. Redimensionamento (Elastic Volumes)

- É a **capacidade de mudar o volume sem perder os dados** e **sem desligar a instância**.
- O que dá para mudar:
  - **Aumentar o tamanho**
  - **Trocar o tipo** (ex.: gp2 → gp3)
  - **Ajustar IOPS e throughput**

> ⚠️ **Pontos de atenção**
>
> - Só dá para **aumentar** o tamanho, **nunca diminuir**. Para reduzir, é preciso criar um volume menor e copiar os dados.
> - Depois de aumentar o volume, é preciso **expandir o sistema de arquivos** dentro do sistema operacional para usar o espaço novo:
>   - **Linux:** `growpart` e depois `resize2fs` (ext4) ou `xfs_growfs` (xfs)
>   - **Windows:** Gerenciamento de Disco → Estender Volume
> - Existe um **limite de quantas modificações** podem ser feitas no mesmo volume em um período. Planeje as mudanças em vez de fazer várias seguidas.

---

## 6. Snapshots (backups do EBS)

- Um **snapshot** é uma **cópia de segurança** do volume em um determinado momento.
- Ele fica armazenado no **S3**, gerenciado pela própria AWS (não aparece nos seus buckets).
- É **incremental**: depois do primeiro, cada novo snapshot salva **só o que mudou**, o que economiza espaço e dinheiro.
- O snapshot guarda **só os blocos com dados**: um volume de 100 GB com 20 GB ocupados gera um primeiro snapshot de cerca de 20 GB.

**Para que serve:**
- **Backup** e recuperação de dados.
- **Mover um volume para outra AZ**: criar um snapshot e depois criar um volume novo a partir dele na AZ desejada.
- **Copiar para outra região** (útil para recuperação de desastres).
- **Criar AMIs** (imagens de servidor para lançar novas instâncias).

> 💡 **Aprofundamento — recursos extras de snapshot**
>
> | Recurso | O que faz |
> |---|---|
> | **Snapshot Archive** | Move snapshots antigos para um nível de armazenamento até **75% mais barato**. A restauração leva de 24 a 72 horas. |
> | **Recycle Bin** (lixeira) | Permite **recuperar snapshots apagados por engano** dentro de um prazo definido por você. |
> | **Fast Snapshot Restore** | Faz o volume criado a partir do snapshot já sair com **desempenho máximo**, sem tempo de "aquecimento". É cobrado à parte. |
> | **Data Lifecycle Manager (DLM)** | **Automatiza** a criação e a exclusão de snapshots (ex.: um backup por dia, mantendo os últimos 7). |

---

## 7. Criptografia

- O EBS pode criptografar os volumes usando o **AWS KMS** (Key Management Service), com o algoritmo **AES-256**.
- Quando o volume é criptografado, ficam protegidos:
  - os **dados armazenados** no volume;
  - os **dados trafegando** entre a instância e o volume;
  - **todos os snapshots** criados a partir dele;
  - **todos os volumes** criados a partir desses snapshots.
- É **transparente**: a aplicação não precisa mudar nada, e o impacto no desempenho é **mínimo**.
- A criptografia do EBS **não tem custo adicional**. Só há custo se você usar uma chave KMS própria.

> 💡 **Aprofundamento — criptografia na prática**
>
> - **Criptografia por padrão:** existe uma configuração, **por região**, que faz **todo volume novo** já ser criado criptografado. É uma boa prática ativar.
> - **Tipos de chave:**
>   - **Chave gerenciada pela AWS** (`aws/ebs`): a mais simples, já vem pronta.
>   - **Chave gerenciada pelo cliente (CMK):** você controla quem pode usar a chave. **É obrigatória** se você quiser **compartilhar um snapshot criptografado com outra conta AWS**.
> - **Como criptografar um volume que já existe sem criptografia:**
>   1. Crie um **snapshot** do volume.
>   2. Crie um **volume novo** a partir do snapshot, **marcando a opção de criptografia** (ou copie o snapshot com criptografia ativada).
>   3. **Troque** o volume antigo pelo novo na instância.
> - Não é possível **remover a criptografia** de um volume criptografado diretamente.

---

## 8. Monitoramento com o CloudWatch

> ⚠️ Interpretei o trecho "volumes no caso é o CloudWatch" como **monitoramento dos volumes pelo CloudWatch**. Se você quis dizer outra coisa, me avise.

O EBS envia métricas para o **Amazon CloudWatch** automaticamente, **sem custo**, a cada 1 minuto.

| Métrica | O que mostra |
|---|---|
| `VolumeReadOps` / `VolumeWriteOps` | Quantidade de leituras e escritas. Dividindo pelo número de segundos do período, você tem o **IOPS**. |
| `VolumeReadBytes` / `VolumeWriteBytes` | Quantidade de dados lidos e escritos. Dividindo pelo número de segundos, você tem o **throughput**. |
| `VolumeTotalReadTime` / `VolumeTotalWriteTime` | Tempo total gasto nas operações. Dividindo pelo número de operações, você tem a **latência média**. |
| `VolumeQueueLength` | Operações **esperando na fila**. Fila alta e constante indica que o volume está **no limite**. |
| `BurstBalance` | Porcentagem de **créditos de burst** restantes (gp2, st1 e sc1). Se chegar a **0%**, o desempenho cai para a linha de base. |
| `VolumeIdleTime` | Tempo em que o volume ficou **sem nenhuma operação**. |

> 💡 **Aprofundamento — alarmes úteis**
>
> - **`BurstBalance` abaixo de 20%** → o volume gp2 está prestes a perder desempenho. Considere migrar para o gp3.
> - **`VolumeQueueLength` alto por muito tempo** → o volume precisa de mais IOPS, ou a instância está no limite dela.
> - **`VolumeIdleTime` alto por dias** → o volume pode estar sem uso e gerando custo à toa.

---

## 9. Custos

Você **paga de acordo com a sua arquitetura**, ou seja, pelo que **provisiona**:

| O que é cobrado | Detalhe |
|---|---|
| **Armazenamento** | Por **GB provisionado por mês**, e **não pelo que usa**. Um volume de 100 GB com 10 GB ocupados é cobrado como 100 GB. |
| **IOPS provisionado** | No **io1 e io2**, todo IOPS é cobrado. No **gp3**, só o que passar de 3.000. |
| **Throughput provisionado** | No **gp3**, só o que passar de 125 MiB/s. |
| **Snapshots** | Por **GB armazenado por mês**. Como são incrementais, você paga só pelo que mudou. |
| **Cópia entre regiões** | Copiar snapshots para outra região tem **custo de transferência de dados**. |

> 💡 **Aprofundamento — preços de referência (região us-east-1)**
>
> Valores **aproximados**, só para comparação. Eles variam por região (São Paulo é mais cara) e podem mudar. Consulte sempre a [página oficial de preços](https://aws.amazon.com/ebs/pricing/).
>
> | Tipo | Preço por GB/mês | Extra |
> |---|---|---|
> | **gp3** | ~US$ 0,08 | ~US$ 0,005 por IOPS acima de 3.000; ~US$ 0,04 por MiB/s acima de 125 |
> | **gp2** | ~US$ 0,10 | — |
> | **io1 / io2** | ~US$ 0,125 | ~US$ 0,065 por IOPS provisionado (o io2 fica mais barato em volumes muito grandes) |
> | **st1** | ~US$ 0,045 | — |
> | **sc1** | ~US$ 0,015 | — |
> | **Snapshot (padrão)** | ~US$ 0,05 | — |
> | **Snapshot (Archive)** | ~US$ 0,0125 | Cobrança extra para restaurar |
>
> **Exemplo de cálculo (volume de 100 GB):**
>
> | Configuração | Cálculo | Total/mês |
> |---|---|---|
> | gp2 | 100 × 0,10 | **~US$ 10** |
> | gp3 (padrão: 3.000 IOPS) | 100 × 0,08 | **~US$ 8** |
> | io2 com 10.000 IOPS | (100 × 0,125) + (10.000 × 0,065) | **~US$ 662** |
>
> **Lição:** no io1 e io2, o que pesa é o **IOPS provisionado**, não o tamanho. Só use quando realmente precisar.

> 💡 **Aprofundamento — dicas para economizar**
>
> - **Migre gp2 para gp3**: economia de cerca de 20%, sem desligar a instância (Elastic Volumes).
> - **Apague volumes "órfãos"**: volumes que não estão conectados a nenhuma instância continuam sendo cobrados.
> - **Gerencie os snapshots**: apague os antigos, use o **Snapshot Archive** para os que precisa guardar por muito tempo e automatize com o **DLM**.
> - **Não superdimensione**: como dá para aumentar depois, comece com o tamanho necessário.

---

## 10. Uso ideal

O EBS é indicado para:

- **Bancos de dados** (MySQL, PostgreSQL, Oracle, SQL Server, MongoDB)
- **Sistemas de arquivos** de uma instância
- **Volume de boot** (disco do sistema operacional) das instâncias EC2
- Aplicações que precisam **armazenar e acessar grandes volumes de dados em bloco**, com baixa latência

> 💡 **Aprofundamento — o que é armazenamento "em bloco"?**
>
> No armazenamento em bloco, os dados são divididos em **blocos de tamanho fixo**, cada um com um endereço. O sistema operacional enxerga isso como um **disco "cru"**, formata com um sistema de arquivos (ext4, xfs, NTFS) e pode **ler ou alterar um pedaço específico** de um arquivo sem reescrever o arquivo inteiro.
>
> Por isso o armazenamento em bloco tem **baixa latência** e é ideal para bancos de dados, que fazem muitas alterações pequenas o tempo todo.

---

## 11. EBS vs outros armazenamentos da AWS

> 💡 **Aprofundamento — esta comparação cai muito em prova**
>
> | | **EBS** | **Instance Store** | **EFS** | **S3** |
> |---|---|---|---|---|
> | **Tipo** | Bloco | Bloco | Arquivo (NFS) | Objeto |
> | **Onde fica** | Na rede, conectado à instância | **Fisicamente** no servidor da instância | Na rede, compartilhado | Serviço independente, acessado por API ou HTTP |
> | **Persistência** | ✅ Persiste ao desligar | ❌ **Perde os dados** ao desligar ou excluir a instância | ✅ Persiste | ✅ Persiste |
> | **Compartilhamento** | 1 instância (exceto Multi-Attach) | Só aquela instância | **Várias instâncias ao mesmo tempo**, em várias AZs | Acessível de qualquer lugar |
> | **Escopo** | 1 AZ | 1 instância | Região (várias AZs) | Região |
> | **Desempenho** | Alto | **Altíssimo** | Médio a alto | Varia |
> | **Uso típico** | Bancos de dados, boot | Cache, buffers, dados temporários | Pastas compartilhadas, CMS, conteúdo web | Backups, arquivos estáticos, data lake |
>
> Veja os detalhes do Instance Store em [04 — Instance Store](04-aws-instance-store.md) e do S3 em [07 — S3](07-aws-s3.md).

---

## 12. Revisão rápida

- **O que acontece com o volume raiz do EBS quando a instância é excluída (terminate)?**
  Ele é apagado por padrão, por causa da opção `DeleteOnTermination`. Os volumes adicionais são mantidos.
- **É possível conectar um volume EBS da AZ `us-east-1a` em uma instância da `us-east-1b`?**
  Não diretamente. É preciso criar um snapshot e depois um volume novo na outra AZ.
- **Qual é a diferença entre IOPS, throughput e latência?**
  IOPS é quantas operações o disco faz por segundo. Throughput é quantos dados passam por segundo (MiB/s). Latência é o tempo que cada operação leva para ser concluída.
- **Qual tipo de volume é o padrão recomendado para a maioria dos casos?**
  O gp3.
- **Qual a diferença de IOPS entre o gp2 e o gp3?**
  No gp2, o IOPS depende do tamanho (3 IOPS por GiB, até 16.000). No gp3, todo volume tem 3.000 IOPS de base, e dá para comprar mais, até 80.000.
- **Qual tipo usar para um banco de dados crítico com altíssimo IOPS?**
  O io2.
- **Pela regra clássica, a partir de quantos IOPS é preciso usar io1 ou io2? E o que mudou?**
  A partir de 16.000 IOPS. Desde setembro de 2025, o gp3 chega a 80.000 IOPS, mas o io2 continua sendo a escolha para latência abaixo de 1 ms, durabilidade de 99,999% e Multi-Attach.
- **Qual volume usar para um data warehouse? E para dados quase nunca acessados?**
  O st1 para data warehouse e o sc1 para dados pouco acessados.
- **Volumes HDD podem ser usados como volume de boot?**
  Não. Só os volumes SSD (gp2, gp3, io1 e io2).
- **Depois de aumentar um volume EBS, o espaço novo já aparece no sistema operacional?**
  Não. É preciso expandir o sistema de arquivos (ex.: `growpart` e `resize2fs` no Linux).
- **Como criptografar um volume EBS que já existe sem criptografia?**
  Criar um snapshot, criar um volume novo criptografado a partir dele e trocar o volume na instância.
- **Qual métrica do CloudWatch avisa que um volume gp2 está ficando sem créditos de burst?**
  A `BurstBalance`.
- **Os snapshots são completos ou incrementais?**
  Incrementais: depois do primeiro, cada snapshot guarda só o que mudou.
