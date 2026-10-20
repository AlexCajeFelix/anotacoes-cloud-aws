# Anotações de Cloud Computing

Anotações de estudo organizadas por assunto.

**Legenda**
- 💡 **Aprofundamento**: conteúdo complementar adicionado à anotação original.
- ⚠️ **Ajuste**: correção ou precisão em relação à anotação original.

## Índice

1. [Origem e História da Cloud Computing](01-historia-da-cloud.md): on-premises, CAPEX vs OPEX, linha do tempo e fatores que impulsionaram a cloud.
2. [AWS EC2: Tipos de Instância e Modelos de Preço](02-aws-ec2.md): famílias (T, M, C, R...), como ler o nome da instância, On-Demand, Savings Plans, Reserved e Spot.
3. [AWS EBS (Elastic Block Store)](03-aws-ebs.md): persistência, IOPS, throughput e latência, tipos de volume (gp2, gp3, io1, io2, st1, sc1) com valores, redimensionamento, snapshots, criptografia, monitoramento com CloudWatch e custos.
4. [AWS Instance Store](04-aws-instance-store.md): armazenamento efêmero de alto desempenho, quando os dados são perdidos, quais instâncias têm e comparação com o EBS.
5. [Segurança no EC2](05-aws-ec2-seguranca.md): Tríade CID, responsabilidade compartilhada, Security Groups, pares de chaves, IAM Roles e checklist de segurança.
6. [AWS IAM](06-aws-iam.md): usuário root, usuários, grupos, roles, políticas JSON (gerenciadas e inline), lógica de avaliação, segurança no nível de API e boas práticas.
7. [Amazon S3](07-aws-s3.md): durabilidade, buckets, objetos e chaves, segurança e criptografia, Transfer Acceleration, casos de uso, classes de armazenamento, ciclo de vida, custos com valores e links oficiais.
8. [Amazon VPC](08-aws-vpc.md): CIDR, subnets públicas e privadas, tabelas de rotas, Internet Gateway, NAT Gateway, endpoints, peering, Transit Gateway, VPN, Direct Connect, custos com valores e arquitetura típica.
9. [VPC na Prática](09-aws-vpc-na-pratica.md): passo a passo no console (VPC, subnets, Internet Gateway, tabelas de rotas, IP público e Security Groups), modelos de roteamento e problemas comuns.
10. [CIDR e Endereçamento IP](10-cidr-enderecamento-ip.md): IPv4 em binário, prefixos, tabela de referência, cálculos de faixa, divisão de subnets, sobreposição, planejamento e exercícios.

## Guias

- [Guia de Escolha: "se eu preciso de X, eu uso Y"](guia-de-escolha.md): exemplos práticos de escolha em todas as categorias (famílias EC2, modelos de preço, volumes EBS, classes do S3, segurança, IAM e VPC) e arquiteturas completas.
