# AWS Infrastructure Builder — Diário de Construção

> De um HTML que desenhava ícones a uma plataforma que **projeta, valida, estima, importa e ensina** arquitetura AWS.

Este documento conta como o projeto foi construído — não só o que ele faz, mas *por que* cada decisão foi tomada. Leia como um diário de engenharia: cada fase resolveu um problema real e preparou a próxima.


<!-- !(https://imgur.com/0SheFpg) -->


| | |
|---|---|
| **35** serviços modelados | **~100** regras de validação |
| **6** templates prontos | **3** importadores de código |
| **5** desafios guiados | **2** partes (front + backend opcional) |

---

## A ideia central

Todo projeto de canvas começa igual: você arrasta ícones e desenha caixinhas ligadas por setas. Este projeto começou assim também. A pergunta que guiou tudo depois foi outra: **e se a ferramenta entendesse o que você está desenhando?**

> A virada não foi adicionar mais ícones. Foi transformar o desenho em um modelo que a máquina consegue raciocinar em cima.

A partir daí, cada camada se apoiou na anterior. Modelar os recursos com propriedades reais deu ao motor de validação algo concreto para inspecionar. A validação alimentou o placar de boas práticas. As propriedades permitiram estimar custo. E, quando tudo isso já existia, sobrou uma base rica o bastante para o app começar a **ensinar** — e para uma IA conversar sobre o diagrama com conhecimento de causa.

---

## Como o projeto evoluiu

**Fase 01 · Fundação — o motor de validação que já existia.**
O ponto de partida já tinha um motor de regras que sabia dizer "Athena precisa de S3". O problema: ele falava em bordas vermelhas, não em respostas. *Todo o resto foi construído extraindo valor desse motor, não substituindo-o.*

**Fase 02 · Apresentação — Well-Architected e recomendações.**
Cada regra ganhou um pilar do AWS Well-Architected Framework, virando um placar. Um painel passou a agregar o que faltava como checklist. *O conhecimento já estava nas regras; faltava apresentá-lo como resposta às perguntas do usuário.*

**Fase 03 · Geradores — do diagrama para código real.**
Terraform e AWS CLI reescritos para ler a config real de cada nó, não templates fixos. *Um gerador que ignora suas escolhas é só um template.*

**Fase 04 · Biblioteca — templates e importadores.**
Seis arquiteturas de referência com um clique. E o app passou a importar JSON, CloudFormation e Terraform. *Importar Terraform fecha o ciclo: o app gera E lê código.*

**Fase 05 · Modelagem — deixar de desenhar, passar a modelar.** ⭐
A virada de categoria. EC2, RDS, S3, Lambda, VPC e ALB ganharam dezenas de propriedades reais, com campos condicionais. *Sem propriedades ricas, a validação só vê "qual serviço existe"; com elas, vê como cada recurso está configurado.*

**Fase 06 · O diferencial — validação baseada em propriedades.** ⭐
Diagnósticos ricos: **Crítico · Impacto · Correção · Referência.** "RDS em subnet pública → exposição da camada de dados → mova para privada → Well-Architected." *A diferença entre um alerta e um parecer de revisor sênior.*

**Fase 07 · Inteligência — recomendação por padrões.** ⭐
Desenhe CloudFront → ALB → EC2 → RDS e o app sugere WAF, ACM, Route 53, Auto Scaling, Secrets Manager, CloudWatch, Backup, Systems Manager. *Regras determinísticas primeiro: rápidas, previsíveis, offline.*

**Fase 08 · Refino — autosave e custo.**
O diagrama sobrevive ao fechar o navegador. A estimativa de custo lê a config real (Spot desconta, Multi-AZ dobra, NAT multiplica por AZ). *Honestidade: custo é ordem de grandeza, e serviços por uso não recebem número inventado.*

**Fase 09 · Ensinar — a tríade didática.** ⭐
Cartões (o que é / quando usar / quando não), tooltips de conceito, e desafios que o validador corrige ao vivo. *O conteúdo técnico já estava nas regras; faltava apresentá-lo como aprendizado.*

**Fase 10 · Full-stack — o Arquiteto de IA + backend.** ⭐
Um chat que enxerga o seu diagrama real. Para isso, um backend mínimo cuja única razão de existir é guardar a chave da API fora do navegador. *Um backend que esconde um segredo é o servidor mínimo defensável. Sem credenciais AWS, sem risco.*

**Fase 11 · Polimento — aviso inteligente de serviço isolado.**
Um WAF sozinho não faz nada (avisa); um EC2 provavelmente esqueceu de conectar (aviso suave); uma VPC ou S3 podem estar sós (silêncio). *Um assistente que grita a cada clique é ignorado.*

---

## Arquitetura em duas partes

### Front-end — 1 arquivo HTML
Todo o núcleo roda 100% no navegador, sem servidor. Abra o HTML e use. Estado salvo no `localStorage`.
- Canvas, modelagem e validação
- Recomendações, custo, Well-Architected
- Geradores e importadores de código
- Cartões, tooltips e desafios didáticos

### Back-end — Node/Express, **opcional**
Existe por um único motivo: dar acesso à IA sem expor a chave da API no navegador. **Nada de credenciais AWS.**
- Proxy Express para o modelo de IA
- A chave nunca chega ao cliente; erros nunca vazam detalhes
- Se estiver fora do ar, o app segue inteiro (degradação graciosa)

---

## Sete capacidades, uma base

Cada uma se apoia na modelagem rica dos recursos — o alicerce comum.

1. **Modela, não desenha** — propriedades reais, agrupadas, com campos condicionais.
2. **Valida com parecer** — Crítico / Impacto / Correção / Referência.
3. **Recomenda o que falta** — analisa o padrão de conexões.
4. **Estima o custo** — lê a config real; honesto sobre ser aproximação.
5. **Gera e importa código** — ciclo diagrama ↔ código fechado.
6. **Ensina AWS** — cartões, tooltips e desafios.
7. **Conversa com IA** — arquiteto que enxerga o diagrama, via backend seguro.

---

## A camada de ensino: ensina, explica e cobra

Não são três features soltas — é um ciclo:

**📖 Cartões** → ensinam o serviço inteiro (o que é, analogia, quando usar/não usar, pegadinhas)
**💡 Tooltips** → ensinam o conceito por trás de cada propriedade, no momento em que você mexe nela
**🎯 Desafios** → "monte algo que sobrevive à queda de uma AZ", e o validador corrige ao vivo

Um aluno pode ler o cartão do RDS, entender Multi-AZ no tooltip, e provar que aprendeu completando o desafio de resiliência.

---

## Limitações conscientes

Escolhas, não descuidos:

- **Custo é ordem de grandeza**, não cotação — preços aproximados e datados.
- **Importador de Terraform é pragmático** — cobre `aws_*` comuns; não é parser HCL completo.
- **Autosave é local** ao navegador — para sincronizar, use export/import JSON.
- **Sem integração AWS ao vivo** — decisão deliberada; o backend só faz proxy da IA.
- **Desafios checam a estrutura** montada, não soluções criativas fora dos objetivos.

---

## Como usar

**Front-end** — abra `AWS_Infra_Builder.html` no navegador. Tudo funciona offline.

**Back-end** (opcional, habilita o Arquiteto de IA):

```bash
cd backend
npm install
cp .env.example .env      # e coloque sua ANTHROPIC_API_KEY
npm start                 # sobe em localhost:8787
```

Volte ao app e abra **🧠 Arquiteto IA**.

---

**Stack** — Front: HTML/CSS/JS puro, sem build. Back: Node, Express, SDK da Anthropic.
