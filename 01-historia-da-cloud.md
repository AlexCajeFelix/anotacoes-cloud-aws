# 01 — Origem e História da Cloud Computing

## 1. Antes da cloud: o modelo on-premises

Antes da computação em nuvem, **toda a infraestrutura era física e própria** (modelo *on-premises*, "no local"). Para colocar um software no ar, a empresa precisava comprar e manter:

- Servidores, storage (armazenamento) e equipamentos de rede
- Espaço físico (datacenter ou sala de servidores)
- Energia, refrigeração, segurança física
- Equipe para instalar, manter e atualizar tudo

### Principais problemas

| Problema | O que acontecia |
|---|---|
| **Custo inicial gigantesco** | Era preciso investir muito dinheiro antes mesmo de o software gerar receita. |
| **Planejamento antecipado** | Era preciso estimar e declarar de antemão quanto se ia gastar e de quanta capacidade se precisaria. |
| **Superdimensionamento** | Comprava-se mais do que o necessário, e o hardware ficava ocioso (dinheiro parado). |
| **Subdimensionamento** | Comprava-se menos do que o necessário, e o sistema caía nos picos de acesso. |
| **Lentidão** | Comprar, entregar e instalar um servidor podia levar semanas ou meses. |

> 💡 **Aprofundamento — CAPEX vs OPEX**
>
> Esta é uma das mudanças mais importantes trazidas pela cloud, e aparece muito em provas de certificação.
>
> | | **CAPEX** (Capital Expenditure) | **OPEX** (Operational Expenditure) |
> |---|---|---|
> | O que é | Investimento inicial em bens físicos | Gasto contínuo com a operação |
> | Exemplo | Comprar 10 servidores | Pagar por hora de uso de servidores na nuvem |
> | Modelo | On-premises | Cloud |
> | Pagamento | Adiantado, valor alto | Sob demanda (*pay-as-you-go*) |
>
> Com a cloud, a empresa **troca CAPEX por OPEX**: em vez de comprar a infraestrutura, ela "aluga" e paga só pelo que usa.

---

## 2. Linha do tempo

| Ano | Marco |
|---|---|
| **1999** | **Salesforce** é fundada e populariza a ideia de software entregue pela internet (SaaS). |
| **2006** | **AWS** lança o **S3** (armazenamento) e o **EC2** (servidores virtuais), o marco inicial da cloud moderna (IaaS). |
| **2006** | O termo **"cloud computing"** é popularizado por Eric Schmidt, então CEO do Google. |
| **2008** | **Google App Engine** é lançado: plataforma para hospedar aplicações sem gerenciar servidores (PaaS). |
| **2008** | A Microsoft **anuncia** o Windows Azure. |
| **2010** | **Windows Azure** fica disponível comercialmente, mais estruturado e organizado. |
| **2011** | O **NIST** (instituto de padrões dos EUA) publica a definição oficial de cloud computing. |
| **2014** | O Windows Azure é renomeado para **Microsoft Azure**. |

> ⚠️ **Ajustes na anotação original**
>
> - "Google App" → o nome correto é **Google App Engine** (2008).
> - A **AWS (2006)** veio **antes** do Google e da Microsoft e é considerada a pioneira da cloud como conhecemos hoje. Vale incluí-la na linha do tempo.

---

## 3. Fatores que impulsionaram a cloud

1. **Avanço tecnológico do hardware**
   A capacidade de processamento e de armazenamento aumentou muito, e o custo por GB e por processador caiu.

2. **Mais banda de internet**
   Conexões mais rápidas e baratas tornaram viável acessar servidores remotos como se fossem locais.

3. **Globalização**
   As empresas passaram a atender clientes no mundo todo e precisavam de infraestrutura em várias regiões, sem construir datacenters em cada país.

4. **Crescimento do Big Data**
   O volume de dados explodiu, e armazenar e processar tudo isso em infraestrutura própria ficou caro e difícil de escalar.

5. **Democratização da tecnologia**
   Startups e pequenas empresas passaram a ter acesso à mesma infraestrutura das grandes, pagando só pelo que usam.

> 💡 **Aprofundamento — dois fatores que valem acrescentar**
>
> 6. **Virtualização**: é a **tecnologia-base da cloud**. Ela permite dividir um servidor físico em várias máquinas virtuais (VMs) isoladas. Sem ela, o provedor não conseguiria "fatiar" e alugar a infraestrutura para vários clientes ao mesmo tempo.
>
> 7. **Economia de escala**: provedores como AWS, Azure e Google compram hardware e energia em quantidades enormes, o que reduz o custo por unidade. Esse ganho chega ao cliente em forma de preços menores do que ele teria montando a própria infraestrutura.

---

## 4. Revisão rápida

- **Qual era o principal problema do modelo on-premises?**
  Alto custo inicial (CAPEX) e necessidade de prever a capacidade com antecedência.
- **Qual provedor é considerado o pioneiro da cloud moderna, e em que ano?**
  A AWS, em 2006, com o S3 e o EC2.
- **Qual tecnologia é a base que tornou a cloud possível?**
  A virtualização.
