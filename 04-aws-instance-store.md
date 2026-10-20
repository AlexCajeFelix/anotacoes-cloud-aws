# 04 — AWS Instance Store

## 1. O que é o Instance Store

O **Instance Store** é um tipo de **armazenamento em bloco efêmero (temporário) e de alto desempenho**. Os discos ficam **fisicamente conectados ao servidor** onde a instância EC2 está rodando.

- **Efêmero:** os dados **só existem enquanto a instância está rodando** naquele servidor.
- **Alto desempenho:** como o disco está **na própria máquina**, sem passar pela rede, ele é **muito mais rápido** do que o EBS.

> 💡 **Analogia**
>
> - O **Instance Store** é como um **quadro branco** na sala de reunião: é rápido de usar, mas **é apagado quando você sai da sala**.
> - O **EBS** é como um **caderno** que você leva com você: é um pouco mais lento, mas **os dados ficam guardados**.

---

## 2. Quando os dados são perdidos

| Ação na instância | O que acontece com os dados do Instance Store |
|---|---|
| **Reboot** (reiniciar) | ✅ Os dados **permanecem** |
| **Stop** (desligar) | ❌ Os dados são **perdidos** |
| **Hibernate** (hibernar) | ❌ Os dados são **perdidos** |
| **Terminate** (excluir) | ❌ Os dados são **perdidos** |
| **Falha do disco ou do servidor físico** | ❌ Os dados são **perdidos** |

> ⚠️ **Por que os dados somem ao desligar?**
>
> Quando a instância é desligada e ligada de novo, a AWS pode colocá-la em **outro servidor físico**. O disco do Instance Store fica no servidor antigo e é **apagado** por segurança. Só no **reboot** a instância continua no mesmo servidor.

---

## 3. Características principais

| Característica | Detalhe |
|---|---|
| **Desempenho** | **Altíssimo**: baixa latência e IOPS muito alto. Em instâncias otimizadas para armazenamento (família **I**), pode chegar a **milhões de IOPS**. |
| **Custo** | **Já está incluído no preço da instância**. Não há cobrança separada. |
| **Tamanho** | **Fixo**, definido pelo tipo de instância. **Não dá para aumentar.** |
| **Conexão** | **Não dá para desconectar** e conectar em outra instância. |
| **Snapshots** | **Não tem snapshot.** O backup é responsabilidade sua, copiando os dados para o EBS ou o S3. |
| **Disponibilidade** | Só existe em **alguns tipos de instância** (veja a seção 4). |
| **Configuração** | Precisa ser definido **ao criar a instância**. Não dá para adicionar depois. |
| **Criptografia** | Nos discos **NVMe**, os dados são criptografados automaticamente por hardware (**AES-256**), e as chaves são destruídas quando a instância é desligada ou excluída. |

---

## 4. Quais instâncias têm Instance Store

O Instance Store **não está disponível em todas as instâncias**. Ele aparece principalmente em:

| Onde aparece | Exemplos | Tipo de disco |
|---|---|---|
| Instâncias com a letra **`d`** no nome | `m5d.large`, `c6gd.xlarge`, `r6id.2xlarge` | SSD NVMe |
| Família **I** (otimizada para armazenamento, foco em IOPS) | `i3.large`, `i4i.xlarge` | SSD NVMe |
| Família **D** (otimizada para armazenamento, foco em capacidade) | `d3.xlarge` | HDD |

> 💡 Lembra da [02 — EC2](02-aws-ec2.md)? Lá vimos que a letra **`d`** no nome da instância significa **"disco local NVMe"**, que é justamente o Instance Store.

---

## 5. Quando usar

O Instance Store é ideal para dados **temporários** ou que **já existem em outro lugar**:

- **Cache** (dados que podem ser recriados a qualquer momento)
- **Buffers** e filas temporárias
- **Arquivos temporários** de processamento (*scratch*)
- **Dados replicados entre várias instâncias**, como em clusters de **Cassandra**, **Hadoop (HDFS)** ou **Kafka**. Se uma instância perder os dados, as outras ainda têm cópia.

**Não use para:**
- Dados que **não podem ser perdidos** (dados de clientes, banco de dados sem réplica)
- Qualquer coisa que **precisa sobreviver** a um desligamento da instância

> 💡 **Aprofundamento — padrão comum em produção**
>
> Muitas empresas usam **os dois juntos**:
> - **EBS** para o sistema operacional e os dados importantes (persistentes).
> - **Instance Store** para cache e arquivos temporários (velocidade).
>
> Assim, você tem **segurança** onde precisa e **desempenho máximo** onde os dados podem ser recriados.

---

## 6. Instance Store vs EBS

| | **Instance Store** | **EBS** |
|---|---|---|
| **Onde fica** | **Fisicamente** no servidor da instância | Na **rede**, conectado à instância |
| **Persistência** | ❌ **Efêmero** (perde os dados ao desligar) | ✅ **Persistente** |
| **Desempenho** | **Altíssimo** (sem rede no caminho) | Alto |
| **Custo** | **Incluído** no preço da instância | Cobrado à parte (por GB e por desempenho) |
| **Tamanho** | Fixo pelo tipo de instância | Escolhido por você e pode aumentar |
| **Snapshots** | ❌ Não tem | ✅ Tem |
| **Mover para outra instância** | ❌ Não dá | ✅ Dá (na mesma AZ) |
| **Uso típico** | Cache, dados temporários, dados replicados | Banco de dados, boot, dados importantes |

Veja os detalhes do EBS em [03 — EBS](03-aws-ebs.md).

---

## 7. Revisão rápida

- **O que significa dizer que o Instance Store é "efêmero"?**
  Que os dados são temporários: só existem enquanto a instância está rodando naquele servidor físico.
- **O que acontece com os dados do Instance Store em um reboot? E em um stop?**
  No reboot, os dados permanecem. No stop, são perdidos.
- **Por que o Instance Store é mais rápido que o EBS?**
  Porque o disco fica fisicamente no servidor da instância, sem passar pela rede.
- **Quanto custa o Instance Store?**
  Nada a mais: já está incluído no preço da instância.
- **É possível fazer snapshot de um Instance Store?**
  Não. O backup precisa ser feito manualmente, copiando os dados para o EBS ou o S3.
- **Como identificar pelo nome se uma instância tem Instance Store?**
  Pela letra `d` no nome (ex.: `m5d.large`) ou pelas famílias I e D.
- **Dê dois exemplos de uso ideal para o Instance Store.**
  Cache e dados replicados entre várias instâncias (ex.: cluster Cassandra ou Hadoop).
