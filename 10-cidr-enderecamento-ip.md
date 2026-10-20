# 10 — CIDR e Endereçamento IP

Entender CIDR é essencial para criar **VPCs**, **subnets** e **regras de Security Group** sem errar. Esta anotação vai do básico até o cálculo de faixas, com exercícios no final.

---

## 1. O endereço IPv4

Um endereço IPv4 tem **32 bits**, divididos em **4 partes de 8 bits**, chamadas **octetos**. Cada octeto vai de **0 a 255**.

```
   10   .    0     .    1     .   25
00001010.00000000.00000001.00011001
└ 8 bits┘└ 8 bits┘└ 8 bits┘└ 8 bits┘   = 32 bits
```

### Como converter um octeto para binário
Cada posição do octeto tem um valor. Some os valores das posições que têm `1`:

| Posição | 1ª | 2ª | 3ª | 4ª | 5ª | 6ª | 7ª | 8ª |
|---|---|---|---|---|---|---|---|---|
| **Valor** | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
| **25 em binário** | 0 | 0 | 0 | 1 | 1 | 0 | 0 | 1 |

25 = 16 + 8 + 1 → `00011001`

> Você **não precisa** converter para binário no dia a dia. Os truques da [seção 4](#4-truques-para-calcular-de-cabeça) resolvem quase tudo. Mas entender o binário mostra **por que** os números funcionam assim.

---

## 2. O que é CIDR

**CIDR** (*Classless Inter-Domain Routing*) é a forma de escrever uma **faixa de IPs**: um endereço seguido de **`/` e um número**, chamado **prefixo**.

```
10.0.0.0/16
         └── prefixo: quantos bits estão FIXOS
```

- O **prefixo** diz quantos bits, da esquerda para a direita, são **fixos** (a parte da **rede**).
- Os bits que sobram são **livres** (a parte dos **hosts**, ou seja, das máquinas).
- **Quantidade de IPs = 2^(32 − prefixo)**

**Exemplo com `10.0.0.0/16`:**
```
10       .0        .x        .x
00001010.00000000.????????.????????
└──── 16 bits fixos ────┘└─ 16 bits livres ─┘

2^16 = 65.536 IPs  →  de 10.0.0.0 até 10.0.255.255
```

**Exemplo com `10.0.1.0/24`:**
```
10       .0        .1        .x
00001010.00000000.00000001.????????
└──────── 24 bits fixos ────────┘└ 8 livres ┘

2^8 = 256 IPs  →  de 10.0.1.0 até 10.0.1.255
```

> **Regra principal:** quanto **maior** o prefixo, **menor** a faixa.

---

## 3. Tabela de referência

| Prefixo | Total de IPs | Utilizáveis em uma subnet AWS (−5) | Máscara equivalente | Observação |
|---|---|---|---|---|
| `/0` | Todos | — | `0.0.0.0` | A internet inteira (`0.0.0.0/0`) |
| `/8` | 16.777.216 | — | `255.0.0.0` | Grande demais para uma VPC |
| **`/16`** | **65.536** | 65.531 | `255.255.0.0` | **Maior VPC permitida** |
| `/17` | 32.768 | 32.763 | `255.255.128.0` | |
| `/18` | 16.384 | 16.379 | `255.255.192.0` | |
| `/19` | 8.192 | 8.187 | `255.255.224.0` | |
| **`/20`** | **4.096** | 4.091 | `255.255.240.0` | Subnets grandes (Kubernetes, Auto Scaling) |
| `/21` | 2.048 | 2.043 | `255.255.248.0` | |
| `/22` | 1.024 | 1.019 | `255.255.252.0` | |
| `/23` | 512 | 507 | `255.255.254.0` | |
| **`/24`** | **256** | **251** | `255.255.255.0` | **Subnet mais comum** |
| `/25` | 128 | 123 | `255.255.255.128` | |
| `/26` | 64 | 59 | `255.255.255.192` | |
| `/27` | 32 | 27 | `255.255.255.224` | |
| **`/28`** | **16** | **11** | `255.255.255.240` | **Menor VPC ou subnet permitida** |
| **`/32`** | **1** | — | `255.255.255.255` | **Um único IP**: usado em regras de Security Group |

> Lembre-se: a AWS **reserva 5 IPs** em cada subnet: os 4 primeiros e o último (veja [08 — VPC](08-aws-vpc.md#2-bloco-cidr-faixa-de-ips-da-vpc)).

> 💡 **Máscara de sub-rede** é outra forma de escrever a mesma coisa: `/24` = `255.255.255.0`. Ela aparece em configurações de rede mais antigas e em provas.

---

## 4. Truques para calcular de cabeça

### Truque 1: prefixos múltiplos de 8
| Prefixo | O que fica livre | Exemplo |
|---|---|---|
| `/8` | Os **3 últimos** octetos | `10.0.0.0/8` → `10.x.x.x` |
| `/16` | Os **2 últimos** octetos | `10.0.0.0/16` → `10.0.x.x` |
| `/24` | O **último** octeto | `10.0.1.0/24` → `10.0.1.x` |
| `/32` | **Nada** | `10.0.1.5/32` → só `10.0.1.5` |

### Truque 2: cada +1 no prefixo corta pela metade
```
/24 = 256  →  /25 = 128  →  /26 = 64  →  /27 = 32  →  /28 = 16
/16 = 65.536  →  /17 = 32.768  →  /18 = 16.384  →  /19 = 8.192  →  /20 = 4.096
```
E **cada −1 dobra**: `/23` = 512, `/22` = 1.024.

### Truque 3: o "tamanho do bloco" no octeto que muda
Quando o prefixo **não é múltiplo de 8**, um octeto fica "dividido". Para saber de quanto em quanto ele anda:

**Tamanho do bloco = 2^(próximo múltiplo de 8 − prefixo)**

| Prefixo | Octeto que muda | Cálculo | Anda de... em... |
|---|---|---|---|
| `/20` | 3º | 2^(24 − 20) = 2^4 | **16 em 16** |
| `/22` | 3º | 2^(24 − 22) = 2^2 | **4 em 4** |
| `/23` | 3º | 2^(24 − 23) = 2^1 | **2 em 2** |
| `/26` | 4º | 2^(32 − 26) = 2^6 | **64 em 64** |
| `/27` | 4º | 2^(32 − 27) = 2^5 | **32 em 32** |
| `/28` | 4º | 2^(32 − 28) = 2^4 | **16 em 16** |

---

## 5. Calculando o início e o fim de uma faixa

**Passo a passo:**
1. Descubra **qual octeto muda** e o **tamanho do bloco** (Truque 3).
2. O bloco começa no número do octeto e vai até **número + bloco − 1**.
3. Os octetos **depois** dele vão de **0 a 255**.

**Exemplo 1: `10.0.16.0/20`**
- Octeto que muda: o 3º, de 16 em 16.
- 3º octeto: de **16** até 16 + 16 − 1 = **31**.
- Faixa: **`10.0.16.0` até `10.0.31.255`** (4.096 IPs).

```
10.0.16.0 = 00001010.00000000.0001|0000.00000000
                                └ 20 bits fixos ┘└ 12 livres ┘
3º octeto: de 0001 0000 (16) até 0001 1111 (31)
```

**Exemplo 2: `192.168.1.64/26`**
- Octeto que muda: o 4º, de 64 em 64.
- 4º octeto: de **64** até 64 + 64 − 1 = **127**.
- Faixa: **`192.168.1.64` até `192.168.1.127`** (64 IPs).

**Exemplo 3: `172.31.32.0/20`** (uma subnet da VPC padrão)
- 3º octeto, de 16 em 16: de **32** até **47**.
- Faixa: **`172.31.32.0` até `172.31.47.255`**.

### Alinhamento: o bloco precisa começar no lugar certo
O número do octeto que muda precisa ser **múltiplo do tamanho do bloco**:

| CIDR | Válido? | Por quê |
|---|---|---|
| `10.0.16.0/20` | ✅ | 16 é múltiplo de 16 |
| `10.0.8.0/20` | ❌ | 8 não é múltiplo de 16. O correto seria `10.0.0.0/20` ou `10.0.16.0/20`. |
| `10.0.10.0/23` | ✅ | 10 é múltiplo de 2 |
| `10.0.3.0/23` | ❌ | 3 não é múltiplo de 2. O correto seria `10.0.2.0/23`. |
| `192.168.1.64/26` | ✅ | 64 é múltiplo de 64 |
| `192.168.1.100/26` | ❌ | 100 não é múltiplo de 64. O correto seria `192.168.1.64/26`. |

---

## 6. Dividindo uma VPC em subnets

**Quantas subnets cabem = 2^(prefixo da subnet − prefixo da VPC)**

**VPC `10.0.0.0/16` dividida em:**

| Tamanho da subnet | Quantas cabem | IPs em cada | Subnets |
|---|---|---|---|
| `/18` | 2^(18−16) = **4** | 16.384 | `10.0.0.0/18`, `10.0.64.0/18`, `10.0.128.0/18`, `10.0.192.0/18` |
| `/20` | 2^(20−16) = **16** | 4.096 | `10.0.0.0/20`, `10.0.16.0/20`, `10.0.32.0/20`, ..., `10.0.240.0/20` |
| `/24` | 2^(24−16) = **256** | 256 | `10.0.0.0/24`, `10.0.1.0/24`, `10.0.2.0/24`, ..., `10.0.255.0/24` |

> Não é obrigatório usar subnets do mesmo tamanho. Dá para ter subnets `/20` para a aplicação e `/24` para o banco, desde que **não se sobreponham**.

---

## 7. Sobreposição de faixas

Duas faixas se **sobrepõem** quando têm **algum IP em comum**. Isso impede criar subnets na mesma VPC e impede conectar VPCs (peering, VPN).

| Faixa A | Faixa B | Sobrepõem? | Por quê |
|---|---|---|---|
| `10.0.0.0/16` | `10.1.0.0/16` | ❌ Não | `10.0.x.x` e `10.1.x.x` são faixas diferentes |
| `10.0.0.0/16` | `10.0.200.0/24` | ✅ Sim | A `/24` está **dentro** da `/16` |
| `10.0.0.0/16` | `10.0.128.0/17` | ✅ Sim | A `/17` é a **metade de cima** da `/16` |
| `10.0.1.0/24` | `10.0.2.0/24` | ❌ Não | Terceiro octeto diferente |
| `10.0.0.0/23` | `10.0.1.0/24` | ✅ Sim | A `/23` vai de `10.0.0.0` a `10.0.1.255` |

---

## 8. Planejando os IPs de uma VPC

### Separar ambientes e redes
Use uma faixa diferente para cada ambiente, para nunca dar conflito:

| Rede | Faixa |
|---|---|
| Produção | `10.0.0.0/16` |
| Homologação | `10.1.0.0/16` |
| Desenvolvimento | `10.2.0.0/16` |
| Escritório / datacenter | `192.168.0.0/16` ou `10.100.0.0/16` |

### Dar significado aos números
Use o **terceiro octeto** para indicar o **tipo** e a **AZ** da subnet:

| Subnet | AZ a | AZ b | AZ c |
|---|---|---|---|
| **Pública** | `10.0.1.0/24` | `10.0.2.0/24` | `10.0.3.0/24` |
| **Privada** (aplicação) | `10.0.11.0/24` | `10.0.12.0/24` | `10.0.13.0/24` |
| **Banco de dados** | `10.0.21.0/24` | `10.0.22.0/24` | `10.0.23.0/24` |

Vendo o IP `10.0.12.47`, você já sabe que é uma instância **privada** na **AZ b**.

### Dicas de planejamento
- Use **`/16` na VPC**: sobra espaço para crescer.
- Use **`/24`** para subnets comuns. Use **`/20` ou maiores** para **Kubernetes (EKS)** ou grandes grupos de Auto Scaling, porque cada pod ou instância consome um IP.
- **Deixe faixas livres** entre os grupos de subnets para crescer depois.
- **Não dá para mudar o CIDR de uma subnet** depois de criada. Na VPC, dá para **adicionar** blocos CIDR secundários, mas não alterar o principal.

---

## 9. CIDR nas regras de Security Group

| Origem na regra | Significado |
|---|---|
| `203.0.113.10/32` | **Só** esse IP |
| `203.0.113.0/24` | Os 256 IPs de `203.0.113.0` a `203.0.113.255` (ex.: a rede de um escritório) |
| `10.0.1.0/24` | Só as instâncias da subnet pública da AZ a |
| `10.0.0.0/16` | **Qualquer instância da VPC** |
| `0.0.0.0/0` | **Qualquer IP** da internet |

> ⚠️ Um erro comum é usar `/24` em vez de `/32` ao liberar "só o meu IP". Isso libera **256 endereços**, e não apenas o seu.

---

## 10. Ferramentas para conferir

**No Linux, com o `ipcalc`:**
```bash
ipcalc 10.0.16.0/20
```

**Com Python (já vem instalado na maioria dos sistemas):**
```bash
python3 -c "import ipaddress as i; n=i.ip_network('10.0.16.0/20'); print(n[0], '-', n[-1], '|', n.num_addresses, 'IPs')"
# 10.0.16.0 - 10.0.31.255 | 4096 IPs
```

**Verificar se duas faixas se sobrepõem:**
```bash
python3 -c "import ipaddress as i; print(i.ip_network('10.0.0.0/16').overlaps(i.ip_network('10.0.200.0/24')))"
# True
```

O **console da AWS** também **sugere** blocos livres na hora de criar a subnet e mostra quantos IPs cada um tem.

---

> 💡 **Aprofundamento — e o IPv6?**
>
> - O IPv6 tem **128 bits** (ex.: `2600:1f18:abcd:1200::/56`), com uma quantidade praticamente infinita de endereços.
> - Na AWS, a VPC recebe um bloco **`/56`**, e cada subnet recebe um **`/64`**.
> - Os blocos IPv6 fornecidos pela AWS são **gratuitos**.
> - **Todo IPv6 é público** (não existe IPv6 "privado" como o `10.x.x.x`). Para que instâncias **só saiam** para a internet, sem receber conexões, usa-se um **Egress-Only Internet Gateway**, que é o equivalente ao NAT para IPv6.

---

## 11. Exercícios (com respostas)

Tente resolver antes de olhar a resposta.

1. **Quantos IPs tem um `/22`? Quantos são utilizáveis em uma subnet da AWS?**
   1.024 IPs, dos quais 1.019 são utilizáveis.
2. **Qual é a faixa de `10.0.48.0/20`?**
   O 3º octeto anda de 16 em 16: de `10.0.48.0` até `10.0.63.255`.
3. **Qual é a faixa de `10.0.40.0/21`?**
   O 3º octeto anda de 8 em 8 (2^(24−21)): de `10.0.40.0` até `10.0.47.255`.
4. **`10.0.10.0/23` é um CIDR válido? E `10.0.3.0/23`?**
   `10.0.10.0/23` é válido (10 é múltiplo de 2). `10.0.3.0/23` não é: o correto seria `10.0.2.0/23`.
5. **Divida `10.0.0.0/16` em 4 subnets iguais.**
   Use `/18`: `10.0.0.0/18`, `10.0.64.0/18`, `10.0.128.0/18` e `10.0.192.0/18`.
6. **Você precisa de uma subnet com pelo menos 500 IPs utilizáveis na AWS. Qual prefixo usar?**
   `/23` (507 utilizáveis). O `/24` tem só 251.
7. **`10.0.0.0/16` e `10.0.128.0/17` se sobrepõem?**
   Sim. A `/17` está dentro da `/16`.
8. **O IP `192.168.1.130` pertence a `192.168.1.128/26`?**
   Sim. O 4º octeto anda de 64 em 64, então a faixa vai de `.128` a `.191`.
9. **Qual é a máscara equivalente ao `/20`?**
   `255.255.240.0`.
10. **Qual é a menor e a maior VPC que dá para criar na AWS?**
    A menor é `/28` (16 IPs) e a maior é `/16` (65.536 IPs).
11. **Para liberar SSH só para o IP `200.150.10.5`, o que colocar na origem da regra?**
    `200.150.10.5/32`.
12. **Quantas subnets `/24` cabem em uma VPC `/20`?**
    2^(24−20) = 16 subnets.
